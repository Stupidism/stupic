# Stupic - A FE code convention set
> Looks like stupid, aha? It comes from an ancient Chinese saying: 'Great wisdom looks stupid. Great skill seems clumsy.'.  
> It's also a combination of several files types of the core code convention [`Folder as Component`](#folder-as-component) of this convention set, which are `Style`, `Type`, `hoo(U)k`, `Props-renderer`, `Index` and `Constant`.


## Table of contents

  - [Folder as component](#folder-as-component)
    - [S.tyled components are different](#styled-components-are-different)
    - [Use hook to handle logic](#use-hook-to-handle-logic)
    - [Types should not cut off code](#types-should-not-cut-off-code)
    - [Constants can be long and sensitive to react components](#constants-can-be-long-and-sensitive-to-react-components)
    - [Props Renderer Component is the core](#props-renderer-component-is-the-core)
    - [Index file re-exports public components & variables](#index-file-re-exports-public-components--variables)
  - [Naming](#naming)
    - [Function/Component name as file name](#functioncomponent-name-as-file-name)
    - [Resource plus Shape as component name](#resource-plus-shape-as-component-name)
    - [Prefer long&consistant names](#prefer-longconsistant-names)
    - [Shortcuts as words](#shortcuts-as-words)
    - [Regularize irregular plural nones](#regularize-irregular-plural-nones)
    - [camelCase for leaf file name](#camelcase-for-leaf-file-name)
    - [dashed-case for namespace folder name](#dashed-case-for-namespace-folder-name)
  - [Project Management](#project-management)
    - [Use absolute path for other components](#use-absolute-path-for-other-components)
    - [Use relative path inside folder components](#use-relative-path-inside-folder-components)

## Folder as component

Click to read more about the thoughts behind [`folder-as-component`](./docs/folder-as-component).

### S.tyled components are different

- Separate file to separate code for html dom tree and css dom tree

```ts
// Good
// styled.Component.ts
export const Component = styled.div`

`;

export const SubComponent = styled.div`

`;

// Component.ts
import * as S from './styled.Component';
<S.Component>
  <S.SubComponent />
</S.Component>

// Bad
// Component.ts
const SubComponent = styled.div`

`;

const Component = styled(({ className }) => {
  return (
    <div className={className}>
      <SubComponent />
    </div>
  )
})`

`;

```

- Consistant root component name with the component, no need to make up names.

```ts
// Good
// Component.ts
const Component = () => (
  <S.Component>
  </S.Component>
);

// Bad
// Component.ts
const Component = () => (
  <S.ComponentContainer>
  </S.ComponentContainer>
);
```

- Clear difference between styled and other components.

```ts
// Good
// styled.Component.ts
export const Component = styled.div`

`;

export const Header = styled.div`

`;

export const Content = styled.div`

`;

export const Footer = styled.div`

`;

// Component.ts
import * as S from './styled.Component';

<S.Component>
  <S.Header>
    <OtherComponentA />
  </S.Header>
  <S.Content>
    <OtherComponentB />
  </S.Content>
  <S.Footer>
    <OtherComponentC />
  </S.Footer>
</S.Component>

// Bad
// Component.ts
import { ComponentContainer, Header, Content, Footer } from './styled.Component';

export const Component = () => (
  <ComponentContainer>
    <Header>
      <OtherComponentA />
    </Header>
    <Content>
      <OtherComponentB />
    </Content>
    <Footer>
      <OtherComponentC />
    </Footer>
  </ComponentContainer>
)
```

- No need to add a styled-components for every dom level.
```ts
// Good
// styled.Component.ts
export const Component = styled.div`
  .Component-header {
    
  }

  .override-other-component {

  }
`;

// Component.ts
import * as S from './styled.Component';

<S.Component>
  <div className="Component-header">header</div>
  <OtherComponent className="override-other-component" />
</S.Component>

// Bad
// styled.Component.ts
export const Component = styled.div`

`;

export const Header = style.div`

`

export const StyledOtherComponent = styled(OtherComponent)`

`;

// Component.ts
<S.Component>
  <S.Header>header</S.Header>
  <S.StyledOtherComponent />
</s.Component>
```

- Add individual styled-component for reusable, complex or portal components.

```ts
// Good
// styled.Component.ts
export const Component = styled.div`
`;

export const ReusableComponent = styled.div`

`;

export const ComplexComponent = styled.div`
  /* Many complex css. */
`;

export const PortalComponent = styled.div`

`;

// Component.ts
import * as S from './styled.Component';

<S.Component>
  <S.ReusableComponent />
  <S.ComplexComponent />

  <Modal>
    <S.PortalComponent />
  </Modal>
</S.Component>

// Bad
// styled.Component.ts
export const Component = styled.div`
  .reusable-component {

  }

  .complex-component {

  }

  .portal-component {

  }
`;

// Component.ts
<S.Component>
  <div className="reusable-component" />
  <div className="complex-component" />

  <Modal>
    {/* Only works when Modal in not implemented by React Portal */}
    <PortalComponent className="portal-component" />
  </Modal>
</S.Component>

// OtherParentComponent.ts
<S.OtherParentComponent>
  <S.Component>
    {/* Use className to reuse the style */}
    <div className="reusable-component" />
  </S.Component>>
</S.OtherParentComponent>
```

### Use hook to handle logic

- Use a separate hook file when changing states can be merged

```ts
// Good
// useComponent.ts
const useComponent = (props) => {
  const res1 = useApi1();
  const res2 = useApi2();

  const loading = res1.loading || res2.loading;
  const error = res1.error || res2.error;
  const data = useMemo(() => ({
    data1: res1.data,
    data2: res2.data,
  }), [res1, res2]);

  return {
    loading,
    error,
    data,
  };
}

// Component.ts
const Component = (props) => {
  const { loading, error, data } = useComponent(props);

  return (
    <S.Component>
      // Render with loading, error and data
    </S.Component>
  );
}


// Bad
const Component = (props) => {
  const res1 = useApi1();
  const res2 = useApi2();
  
  const loading = res1.loading || res2.loading;
  const error = res1.error || res2.error;
  const data = useMemo(() => ({
    data1: res1.data,
    data2: res2.data,
  }), [res1, res2]);

  return (
    <S.Component>
      // Render with loading, error and data
    </S.Component>
  );
}
```

- Don't use a separate hook file when hooks are very simple

```ts
// Good
// Component.ts
const Component = () => {
  const [open, setOpen] = useState(false);

  return (
    <S.Component>
      <button onClick={() => setOpen(!open)}>toggle open</button>
    </S.Component>
  );
}

// Bad
// useComponent.ts
const useComponent = () => {
  const [open, setOpen] = useState(false);

  const toggleOpen = useCallback(() => {
    setOpen(!open);
  }, [open]);

  return {
    open,
    toggleOpen,
  }
}

// Component.ts
const Component = () => {
  const { open, toggleOpen } = useComponent();

  return (
    <S.Component>
      <button onClick={toggleOpen}>toggle open</button>
    </S.Component>
  );
}
```

### Types should not cut off code

- Extract out complex types to keep the main code consistant and meaningful 

```ts
// Good
// types.Component.ts
export interface Props {
  id?: number,
}

export interface HookProps {
  loading: boolean;
  data: {
    id: number;
    name: string;
  }
}

// Component.ts
import { Props, HookProps } from './types.Component';

const ComponentRenderer: React.FC<Props & HookProps> = React.memo((props) => {
  
});

// useComponent.ts
const useComponent = (props: Props): HookProps => {

}

// Bad
interface Props {
  id?: number,
}

const ComponentRenderer: React.FC<Props & {
  loading: boolean;
  data: {
    id: number;
    name: string;
  }
}> = React.memo((props) => {
  
});

// useComponent.ts
const useComponent = (props: Props) => {
  return {
    id,
    data,
  }
}
```

- Typerify input(props), output(hook props, html dome tree) and side effects(user input and api call), leave the rest auto inferred.

```ts
// Good
// types.Component.ts

// input of component and hook
interface Props {}
// output of hook, input of renderer
interface HookProps {
  onClick: () => void;
}
// user input side effect
interface FormValues {}
// api call side effect
interface Resource {}
interface ApiData<T> {}
interface ApiError<T> {}

const apiCall = async (values: FormValues): Resource => {
  
}

<Form
  onSubmit={async (values: FormValues) => {
    // Got a resource with type Resource.
    const resource = await apiCall(values);
  }}
>

// Bad
const a: number = 3;

const apiCall = async (values) => {};

<Form
  onSubmit={async (values) => {
    // Force the resource to be type Resource
    const resource: Resource = (await apiCall(values)) as Resource;
  }}
>

interface UseComponent {
  (props: Props) => {
    onClick: () => void;
  }
}

interface OnClick {
  (e: Event) => void;
}

```

### Constants can be long and sensitive to react components

- Declare constants outside component if possible.
```ts
// Good
// Object reference not changed
const pageMeta = {

}

const Component = () => (
  <SomeComponent pageMeta={pageMeta}>
);

// Bad
const Component = () => {
  // New object reference every time when Component renders.
  const pageMeta = {

  };

  return (
    <SomeComponent pageMeta={pageMeta}>
  );
};
```

- Declare constants in another file to share them with other files in the component folder.
```ts
// Good
// const.Component.ts
export BUTTON_WIDTH = 200;
export PAGE_SIZE = 20;

// Component.ts
import { BUTTON_WIDTH, PAGE_SIZE } from './const.Component';

<S.Button>Fixed Width</S.Button>
{/* Align with button */}
<Box width={BUTTON_WIDTH}></Box>
<Pagination pageSize={PAGE_SIZE} />

// styled.Component.ts
import { BUTTON_WIDTH } from './const.Component';

const Button = styled.button`
  width: ${BUTTON_WIDTH}px;
`;

// useComponent.ts
import { PAGE_SIZE } from './const.Component';
const res = useApi({ pageSize: PAGE_SIZE });

// Bad
// Component.ts
<S.Button>Fixed Width</S.Button>
{/* Align with button */}
<Box width={200}></Box>
<Pagination pageSize={20} />

// styled.Component.ts
const Button = styled.button`
  width: 200px;
`;

// useComponent.ts
const res = useApi({ pageSize: 20 });
```

- Functions are first-class components in JS world

```ts
// Good
// const.Component.ts
const createPageMeta = (title: string) => {
  return {
    title: `meta for ${title}`,
  }
}

// Component.ts
const Component = ({ title }) => {
  const pageMeta = useMemo(() => createPageMeta(title), [title]);

  return (
    <SomeComponent pageMeta={pageMeta}>
  );
}

// Bad
// Component.ts
const Component = ({ title }) => {
  // This part can be very long and not really related to the current dom structure.
  const pageMeta = useMemo(() => ({
    title: `meta for ${title}`,
  }), [title]);

  return (
    <SomeComponent pageMeta={pageMeta}>
  );
};
```

### Props Renderer Component is the core

- Use a separate renderer when render logic has branches before hooks
```ts
// Good
const ComponentRenderer = ({ data }) => {
  const state = useMemo(() => {

  }, [data]);

  return (
    <S.Component>
    </S.Component>
  );
}

const Component = (props) => {
  const { error, loading, data } = useApi();

  if (error) {
    return 'Error!';
  }

  if (loading) {
    return 'Loading...';
  }

  return <ComponentRenderer {...props} data={data} />;
}

// Bad
const Component = (props) => {
  const { error, loading, data } = useApi();
  
  const state = useMemo(() => {

  }, [data]);

  if (error) {
    return 'Error';
  }

  if (loading) {
    return 'Loading...';
  }

  // Can't write it here because of the hook order limitation
  // const state = useMemo(() => {
  // }, [data]);

  return (
    <S.Component>
    </S.Component>
  );
}
```


### Index file re-exports public components & variables

```ts
// Good
// Component/index.ts
export { Header as StyledComponentHeader } from './styled.Component';
export { useComponent } from './useComponent';
export { buildResourceAFromResourceB } from './const.Component';
export { ComponentRenderer } from './Component';
export type { Props as ComponentProps } from './types.Component';

// OtherParentComponent.ts
import { StyledComponentHeader, buildResourceAFromResourceB, ComponentRenderer, useComponent, ComponentProps } from 'Component';

<S.OtherParentComponent>
  <StyledComponentHeader>
  </StyledComponentHeader>
</S.OtherParentComponent>

// Bad
// OtherParentComponent.ts
import * as ComopnentS from 'Component/styled.Component';
import { buildResourceAFromResourceB } from 'Component/const.Component';
import { ComponentRenderer } from 'Component/Component';
import { useComponent } from 'Component/useComponent';
import type { Props as ComponentProps } from 'Component/types.Component';

interface Props extends ComponentProps {

}

<S.OtherParentComponent>
  <ComopnentS.Header>
  </ComopnentS.Header>
</S.OtherParentComponent>
```


## Naming

### Function/Component name as file name

Use long specific names for components and functions instead of fuzzy names:

```bash
# Good
ArticleHeader/ArticleHeader.tsx
useBreakpoints.ts
withDataTable.tsx
parseLink.ts 
mediaUtils.ts

# Bad
ArticleHeader/index.tsx # write component code in index.tsx
article-components.tsx
hooks.ts
hocs.tsx
utils.ts
constants.ts
```

### Resource plus Shape as component name

Use long specific names combining business resource and shape type instead of fuzzy names:

```ts
// Good
const ArticleCard = () => <div>...</div>;
const ArticlePage = () => <div>...</div>;
const ArticleBox = () => <div>...</div>;
const ArticleList = () => <div>...</div>;
const ArticleTable = () => <div>...</div>;

// Bad
const Article = () => <div>...</div>;
const ArticleWrapper = () => <div>...</div>;
const ArticleComponent = () => <div>...</div>;
```

### Prefer long&consistant names

Long names normally mean clarity and consistancy can improve readibility.

```bash
# Good
ArticleCard/index.ts
ArticleCard/ArticleCard.tsx
ArticleCard/ArticleHeader.tsx
QuestionCard/index.ts
QuestionCard/QuestionCard.tsx
QuestionCard/QuestionHeader.tsx

# Bad
ArticleCard/index.tsx
ArticleCard/Card.tsx
ArticleCard/Header.tsx
QuestionCard/index.tsx
QuestionCard/Card.tsx
QuestionCard/Header.tsx
```

And in files:

```ts
// Good
// ArticleCard
const Card = () => <div>...</div>;
const useArticleCard = () => {};
const buildArticleFromFormValues = () => {};

const ArticleHeader = () => <div>...</div>;
const Card = () => <div>...</div>;
const QuestionHeader = () => <div>...</div>;

// Bad
const Card = () => <div>...</div>;
const useArticles = () => {};
const formatFormValues = () => {};
const Header = () => <div>...</div>;
// QuestionCard
const Card = () => <div>...</div>;
const Header = () => <div>...</div>;


// Good in ArticleCard/index.ts
export { ArticleHeader } from './ArticleHeader';
export { Props as ArticleCardProps } from './types.ArticleCardProps';

// Bad in ArticleCard/index.ts
export { Header } from './ArticleHeader';
export { Props } from './types.ArticleCardProps';
```

### Shortcuts as words

Although we don't recommend using shortcuts only for shorter names, but when we use them, use them as words:

```ts
// Good
const buildTocOfCdnUrls = () => {};
const getHtmlFromServer = () => {};
const getHtmlsFromServer = () => {};

// Bad
const buildToCofCDNURLs = () => {};
const buildToC_of_CDN_URLs = () => {};
const getHTMLFromServer = () => {};
const getHTMLsFromServer = () => {};
```

### Regularize irregular plural nones

Not all engineers are native English speakers, and we can understand if they are in the wrong foramt as long as it has a necessary `s` or `es` suffix. Sometimes, right plural nouns can even cause misunderstandings.

```ts
// Good
const articleIndexs = [];
const pageSchemas = [];
const metaDatas = [];
const currencys = [];
const fishes = [];
const analysises = [];

// Bad
const articleIndices = [];
const pageSchema = [];
const metaData = [];
const currencies = [];
const fish = [];
const analyses = [];

// Workaround
const articleIndexList = [];
const pageSchemaList = [];
const metaDataList = [];
const currencyList = [];
const fishList = [];
const analysisList = [];
```

### camelCase for leaf file name

Use camelCase for leaf files(except using `ClassCase` for Components, as React official recommended):

```bash
# Good
article/ArticleGrid/ArticleCard.tsx

# Bad
article/article-card/article-card.tsx
// Remember we consider folder as component
article/article-card/ArticleCard.tsx

```

### dashed-case for namespace folder name

Normally, we strongly recommend against using namespaces, but when you need one,

```bash
# Good
article-components/ArticleGrid/ArticleCard.tsx
article-utils/groupArticlesByTag.ts
article-types/relatedArticles.ts

# Bad
articleComponents/ArticleGrid/ArticleCard.tsx
// Remember that we consider folder as component, component folder is not a namespace here
article-components/article-card/ArticleCard.tsx

```

## Project Management

### Use absolute path for other components

Absolute path means no long or wrong `../`s. Also it would be easier to make code work after copying and pasting in another file.

```ts
// Good
// Component/Component.ts
import { Button } from '@/components/Button';
import { Typography } from '~components/Typography';
import { Typography } from '@my-org/my-lib/lib/Typography';
import { Icon } from '@my-org/other-lib';

// Bad
// Component/Component.ts
import { Button } from '../Button';
import { Typography } from '../Typography';
```

### Use relative path inside folder components
```ts
// Good
// Component/Component.ts
import { Props } from './types.Component';
import { useComponent } from './useComponent';

// Bad
// Component/Component.ts
import { Props } from '@/components/Component/types.Component';
import { useComponent } from '@/components/Component/useComponent';
```


