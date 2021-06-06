# Stupic - A FE code convention set
> Looks like stupid, aha? It comes from an ancient Chinese saying: 'Great wisdom looks stupid. Great skill seems clumsy.'.  
> It's also a combination of several files types of the core code convention [`Folder as Component`](./docs/folder-as-component) of this convention set, which are `Style`, `Type`, `hoo(U)k`, `Props-renderer`, `Index` and `Constant`.


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
- [Styling](#styling)
  - [Order css properties in 3 levels of parent-self-children](#order-css-properties-in-3-levels-of-parent-self-children)
  - [Use Component name as prefix for public class names](#use-component-name-as-prefix-for-public-class-names)
- [Api](#api)
  - [Use Functions/Hooks to fetch data](#use-functionshooks-to-fetch-data)
  - [Use one param as api function input](#use-one-param-as-api-function-input)
- [Project Management](#project-management)
  - [Use separate library to mange resource types](#use-separate-library-to-mange-resource-types)
  - [Use separate libraries to manage api functions for different backends](#use-separate-libraries-to-manage-api-functions-for-different-backends)
  - [Use absolute path for other components](#use-absolute-path-for-other-components)
  - [Use relative path inside folder components](#use-relative-path-inside-folder-components)
  - [Use enginner name as prefix of feature/fix branches](#use-enginner-name-as-prefix-of-featurefix-branches)

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

```

```ts
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

```

```ts
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

```

```ts
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

```

```ts
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

```

```ts
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
  const res1 = useSomeHook();
  const res2 = useAnotherHook();

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


```

```ts
// Bad
const Component = (props) => {
  const res1 = useSomeHook();
  const res2 = useAnotherHook();
  
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

```

```ts
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

```

```ts
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

```

```ts
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

```

```ts
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

```

```ts
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

```

```ts
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

```

```ts
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

```

```ts
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

```

```ts
// Bad
const Article = () => <div>...</div>;
const ArticleWrapper = () => <div>...</div>;
const ArticleComponent = () => <div>...</div>;
const Component = () => <div>...</div>;
```

For more shapes, you can refer [material-ui components](https://material-ui.com/components/grid/).

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
const ArticleCard = () => <div>...</div>;
const useArticleCard = () => {};
const buildArticleFromFormValues = () => {};

const ArticleHeader = () => <div>...</div>;
const QuestionCard = () => <div>...</div>;
const QuestionHeader = () => <div>...</div>;

```

```ts
// Bad
const Card = () => <div>...</div>;
const useArticles = () => {};
const formatFormValues = () => {};
const Header = () => <div>...</div>;
// QuestionCard
const Card = () => <div>...</div>;
const Header = () => <div>...</div>;
```

```ts
// Good in ArticleCard/index.ts
export { ArticleHeader } from './ArticleHeader';
export { Props as ArticleCardProps } from './types.ArticleCardProps';

```

```ts
// Bad in ArticleCard/index.ts
export { Header } from './ArticleHeader';
export { Props } from './types.ArticleCardProps';
```

### Shortcuts as words

Although we don't recommend using shortcuts only for shorter names, but when we use them, use them as words:

```ts
// Good
import GraphqlJson from 'graphql-type-json';

const buildTocOfCdnUrls = () => {};
const getHtmlFromServer = () => {};
const getHtmlsFromServer = () => {};
```

```ts
// Bad
// From official code example of graphql-type-json
import GraphQLJSON from 'graphql-type-json';

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

```

```ts
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
article/ArticleGrid/useArticleCard.tsx
// Except for react components
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

## Styling

### Order css properties in 3 levels of parent-self-children

When we write a lot of css properties, they could be very hard to read. A small tip is to separate them into 3 sections. 1. css that pair with parent 2. css that control itself 3. css that controls its children

```less
/* Good */
.grand-parent {
  // Control self
  width: 100%;

  // Control children
  position: relative;

  .parent {
    // Pair with parent's parent
    position: absolute;
    top: 0;
    left: 0;
    right: 0;

    // Control self
    color: red;
    background: yellow;

    // Control children
    display: flex;

    .child {
      // Pair with parent
      flex: 1;

      // Control self
      width: 33%;
    }
  }
}
```


```less
/* Bad */
.grand-parent {
  position: relative;
  width: 100%;

  .parent {
    display: flex;
    position: absolute;
    top: 0;
    color: red;
    left: 0;
    right: 0;
    background: yellow;

    .child {
      width: 33%;
      flex: 1;
    }
  }
}
```

### Use Component name as prefix for public class names

```ts
// Good
const ComponentA = styled.div`
  .ComponentA-public-class-name {

  }

  .inner-class-name {

  }
`;

const ComponentB = styled.div`
  .ComponentA-public-class-name {
    // override some css
  }
`;

```

```ts
// Bad
const ComponentA = styled.div`
  .public-class-name {

  }
`;

const ComponentB = styled.div`
  .public-class-name {
    // override some css
  }

  .inner-class-name {
    // override some css
  }
`;

```

## Api

### Use Functions/Hooks to fetch data

Api calls are side effects. Manage them with a promise or a meta object and we don't need to know which kind of backend we're using. No matter it's graphql, or fetch or axios or any other libraries.

```ts
// Good
const getArticles: Promise<Response<Article[]>> = async (params: ArticlesQueryParams) => {  
  // Build options for api caller
  const options = buildApiOptions(params);

  // Call api with fetch or other api clients. e.g. graphql or axios
  return fetch(options);
}

const useArticles = async (params: ArticlesQueryParams): {
  loading: boolean;
  error: Error;
  response: Response<Article[]>;
  data: Article[];
} => {
  const [loading, setLoading] = useState(false);
  const [response, setResponse] = useState([]);
  const [error, setError] = useState(null);

  const callApi = async () => {
    setLoading(true);
    try {
      const response = await getArticles(params);

      setResponse(response);
    } catch(e) {
      setError(e);
    }
    setLoading(false);
  }

  useEffect(() => {
    callApi();
  }, [callApi]);

  return {
    loading,
    error,
    response,
    data: response.path.to.data,
  }
}

// or with hook libraries
const useArticles = (params: ArticlesQueryParams) => {
  // Build options for api caller
  const options = buildApiOptions(params);

  return useQuery<Response<Article>, Options>(options);
}

// Get type guarded articles in business code with one line code.
import { useArticles } from '@scope/xxx-api';

const Component = () => {
  const { data: articles } = useArticles(params);

}
```

```ts
// Bad
// Import many stuff 
import { Article, ArticleQuery, ArticleResponse } from '@scope/xxx-api';
import { useQuery } from '@apollo/client';
// import { useAxios } from 'axios';

const Component = () => {
  // Write all the states and api calls inside component code. e.g.
  const { loading, data, error } = useQuery<Response<Article>, Options>(options);
}
```

### Use one param as api function input

One param makes refactoring easier and middleware(params builder) easier.

```diff
// Good

interface ArticleQueryParams {
  id: string,
+ withAuthor?: boolean,  
}

const useArticle = (params: ArticleQueryParams) => {

};

const articleParamsBuilder = (formValues: FormValues): ArticleQueryParams => {
  return {
    id: formValues.id,
+   withAuthor: formValues.withAuthor,
  }
}

const Component = () => {
  const { data: article } = useArticle(articleParamsBuilder(formValues));
}

```

```diff
// Bad

const useArticle = (
  id: string,
+ withAuthor?: boolean 
) => {
  
};

-const articleParamsBuilder = (formValues: FormValues): string => {
-  return formValues.id;

+const articleParamsBuilder = (formValues: FormValues): [string, boolean] => {
  return [formValues.id, formValues.withAuthor]
}

const Component = () => {
- const { data: article } = useArticle(articleParamsBuilder(formValues));
+ const { data: article } = useArticle(...articleParamsBuilder(formValues));
}
```


## Project Management

### Use separate library to mange resource types

```ts
// Good
// @scope/xxx-types/src/resources/articles.ts
interface Article {

}

// @scope/fe-app/src/components/ArticlePage.ts
import type { Article } from '@scope/xxx-types';

// @scope/be-app/src/app/article/resolvers/article.resolver.ts
import type { Article } from '@scope/xxx-types';

```

```ts
// Bad
// @scope/fe-app/src/types/graphql/articles
interface FrontendArticle {

}

// @scope/fe-app/src/components/ArticlePage.ts
import type { FrontendArticle } from '../types/graphql/articles';

// @scope/be-app/src/app/article/graphql-types/articles.ts
interface BackendArticle {

}

// @scope/be-app/src/app/article/resolvers/article.resolver.ts
import type { BackendArticle } from '../graphql-types/articles';

```

### Use separate libraries to manage api functions for different backends

```ts
// Good
// @scope/xxx-api/src/api-hooks/useArticles.ts
const useArticles = () => {}

// @scope/fe-app-a/src/components/ArticleListPage.ts
import { useArticles } from '@scope/xxx-api';

// @scope/fe-app-b/src/components/ArticleManagementPage.ts
import { useArticles } from '@scope/xxx-api';
```

```ts
// Bad
// @scope/fe-app-a/src/api-hooks/useArticles.ts
const useArticles = () => {}

// @scope/fe-app-a/src/components/ArticleListPage.ts
import { useArticles } from '../api-hooks/useArticles';

// @scope/fe-app-b/src/api-hooks/useArticles.ts
const useArticles = () => {}

// @scope/fe-app-b/src/components/ArticleManagementPage.ts
import { useArticles } from '../api-hooks/useArticles';
```

### Use absolute path for other components

Absolute path means no long or wrong `../`s. Also it would be easier to make code work after copying and pasting in another file.

```ts
// Good
// Component/Component.ts
import { Button } from '@/components/Button';
import { Typography } from '~components/Typography';
import { Typography } from '@my-org/my-lib/lib/Typography';
import { Icon } from '@my-org/other-lib';

```

```ts
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

```

```ts
// Bad
// Component/Component.ts
import { Props } from '@/components/Component/types.Component';
import { useComponent } from '@/components/Component/useComponent';
```

### Use enginner name as prefix of feature/fix branches

Managing branches in monorepo could be very hard, name-prefixed branches would make your life easier.

```bash
# Good

$ git branch

  feng/add-component-generator
* feng/fix-custom-env-preview-mode
  feng/fix-image-responsive-bug
  feng/fix-library-babelrc
  feng/fix-section-children-box-shadow
  feng/fix-ssr-className-mismatch-bug
  feng/fix-ssr-className-mismatch-bug-experiment
  feng/fix-ssr-className-mismatch-bug2
  feng/fix-video-url-of-review
  feng/implement-reviews-with-contentful
  feng/responsive-container
```

```bash
# Bad

$ git branch -r 

  origin/tesseract-example
  origin/test
  origin/testa
  origin/testinglib
  origin/tmp-test
  origin/tonygjerry-patch-1
  origin/trav-telematics-limit
  origin/travelers--retention-fix-attempt-3-using-proxy
  origin/tsconfig-correct
  origin/ui-expert-qna
  origin/ui-seo-insurance-e2e
  origin/update-prequal
  origin/update-usdl-regex
  origin/webhook-basic
  origin/wip
  origin/workaround_user_prompt_as_expect_error
  origin/wq-confirm-driver
  origin/wq-estimate-quoting-loader
  origin/wq-state-min-coverage

```

