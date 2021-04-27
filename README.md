# Stupic - A FE code convention set
> Looks like stupid, right? It comes from an ancient Chinese saying: 'Great wisdom looks stupid. Great skill seems clumsy.'.  
> It's also a combination of several files types of the core code convention [`Folder as Component`](#folder-as-component) of this convention set, which are `Style`, `Type`, `hoo(U)k`, `Props-renderer`, `Index` and `Constant`.


## Table of Contents

- Folder as Component
- Naming
- Project Dependency

## Folder as Component 

As [answered by the React official](https://reactjs.org/docs/faq-structure.html), grouping by features/routes and by file types are both very helpful. What I refine one step further here is to treat a folder containing different types of files for a specific feature as a component. After all, component is the first-class citizen in React world.

### Classic HTML/CSS/JS files

When we construct a ui interface, we have to consider the **html dom tree**, the **css dom tree** and the **js business logic**. When us engineers write code for these 3 different things, we think in different mode. So I don't like the idea of messing them up together

What `React` brings to us is that it creates a mapper from data in js part to html dom in the html part. Unfortunately, `react` didn't handle the other parts for us at the beginning. `stupic`(I personally) choose `styled-components` as the `css-in-js` solution for the css part, you can choose your own and replace the file format accordingly. And after `React` support `hooks`, we have a great tool to handle logic, too.

### Component.tsx

A normal `React` component can be written inside such a file alone and it's also the original format of a `stupic` component. Everything starts from this file and will be assembled here. A typical `React` component with `styled-components` and `typescript` could look like this:

```ts
import React, { useState } from 'react';
import styled from 'styled-components';

const StyledComponent = styled.div`
  // some css fragments
`;

interface Props {
  // some ts definitions
}

interface State {
  // some ts definitions
}

const meta = {
  // some constant meta data
}

const initializeState = (props: Props): State => {};

export const Component: FC<Props> = (props) => {
  const [state, setState] = useState(() => initializeState(props));
  // some other hooks

  return (
    <StyledComponent>
      {/* some react vdom fragments */}
    </StyledComponent>
  );
}
```

If the component keeps small, the component is not very hard to maintain.

> However, what if it doesn't?

### Component/styled.Component.ts

When a `React` component has multiple styled components, the code above the core component could be so long that it affects the understanding for the core `html dom tree` and `js logic` parts. That's when we extract all styled components out to a `Component/styled.Component.ts` file and `import * as S` to use:

```ts
// Component/styled.Component.ts
export const Component = style.div`

`;

export const Header = styled.div`

`;

export const Content = styled.div`

`;

export const Footer = styled.div`

`;

// Component/Component.tsx
import * as S from './styled.Component';

<S.Component>
  <S.Header>
    {/* some header code */}
  </S.Header>
  <S.Content>
    {/* some content code */}
  </S.Content>
  <S.Footer>
    {/* some footer code */}
  </S.Footer>
</S.Component>
```

You might feel strange about the `import * as S` convention, that's another rule we will discuss later. The main point here is to split these styled components into another file to keep the core `html dom tree` part clean and simple.

> Again, `styled-components` is an opinioned choice, you can replace it with other css solutions like `less/scss` or `styled-jsx` as you want. Just replace the file name with `Component.less`, `style.Component.ts`. The key is to keep the representiveness for css dom tree.

### Component/useComponent.ts

And when the logic part gets too complex,

### Component/types.Component.ts

### Component/const.Component.ts

### Component/Component.ts

### Component/index.ts


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
ArticleHeader/index.tsx
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
# write component in this file to save one file
ArticleCard/index.tsx
ArticleCard/Header.tsx
QuestionCard/index.tsx
QuestionCard/Header.tsx
```

And in files:

```ts
// Good
const ArticleCard = () => <div>...</div>;
const useArticleCard = () => {};
const buildArticleFromFormValues = () => {};

const ArticleHeader = () => <div>...</div>;
const QuestionCard = () => <div>...</div>;
const QuestionHeader = () => <div>...</div>;

// Bad
const ArticleCard = () => <div>...</div>;
const useArticles = () => {};
const formatFormValues = () => {};
const Header = () => <div>...</div>;
const QuestionCard = () => <div>...</div>;
const Header = () => <div>...</div>;


// Good in ArticleCard/index.ts
export { ArticleHeader } from './ArticleHeader';

// Bad in ArticleCard/index.ts
export { Header } from './ArticleHeader';
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
