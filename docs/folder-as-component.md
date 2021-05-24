# Folder as Component 

As [answered by the React official](https://reactjs.org/docs/faq-structure.html), grouping by features/routes and by file types are both very helpful. What I refine one step further here is to treat a folder containing different types of files for a specific feature as a component. After all, component is the first-class citizen in the React world.

Before I introduct what's `Folder as Component`, let's have a look at the classic model of Front-end world.

- [Folder as Component](#folder-as-component)
    - [Classic HTML/CSS/JS Files](#classic-htmlcssjs-files)
    - [Normal React Component File](#normal-react-component-file)
    - [Style-heavy Components](#style-heavy-components)
      - [Component/styled.Component.ts](#componentstyledcomponentts)
      - [Logic-heavy Components](#logic-heavy-components)
      - [Component/useComponent.ts](#componentusecomponentts)
    - [Type-heavy components](#type-heavy-components)
      - [Component/types.Component.ts](#componenttypescomponentts)
    - [Const-heavy Components](#const-heavy-components)
      - [Component/const.Component.ts](#componentconstcomponentts)
    - [Component/Component.ts](#componentcomponentts)
    - [Component/index.ts](#componentindexts)


### Classic HTML/CSS/JS Files

When we construct a ui interface, we have to consider the **html dom tree**, the **css dom tree** and the **js business logic**. When us engineers write code for these 3 different things, we think in different mode. So I don't like the idea of messing them up together.

What `React` brings to us is that it creates a mapper from data in js part to html dom in the html part. Unfortunately, `react` didn't handle the other parts for us at the beginning. After `React` support `hooks`, we have a great tool to handle business logic and build the data we use in the html part, too. `stupic`(I personally) choose `styled-components` as the `css-in-js` solution for the css part, you can choose your own and replace the file format accordingly.

Let's dive into the component folder then.

### Normal React Component File

A normal `React` component can be written inside such a file alone and it's also the original format of a `stupic` component. Everything starts from this file and will be assembled here. A typical `React` component with `styled-components` and `typescript` could look like this:

```ts
import React, { useState } from 'react';
import styled from 'styled-components';

const StyledComponent = styled.div`
  // Some css fragments
`;

// Exported for reuse
export interface ComponentProps {
  // Some ts definitions
}

interface State {
  // Some ts definitions
}

const meta = {
  // Some constant meta data
}

const initializeState = (props: ComponentProps): State => {};

// Exported component 
export const Component: FC<ComponentProps> = (props) => {
  const [state, setState] = useState(() => initializeState(props));
  // Some other hooks

  return (
    <StyledComponent>
      {/* Some react vdom fragments */}
    </StyledComponent>
  );
}
```

When the component keeps growing, it is very hard to maintain.

If you agree with me that nobody can stop components from growing larger, we can discuss more upon this consensus then. Let's see what kinds of components would be too large to maintain:

### Style-heavy Components

When a `React` component has multiple styled components, the code above the core component could be so long that it affects the understanding for the core `html dom tree` and `js logic` parts. 

1. Hard to tell styled components and other components apart, they are all messed up together, even when someone choose to add a `Styled` prefix for styled components.

2. Feel awful everytime you have to name the root styled component. Which one is best? `ComponentContainer`, `ComponentWrapper`, `ComponentRoot` or `StyledComponent`?

3. Hard to export some of them for reuse because of the naming problem.

```ts
// Component.tsx
const ComponentContainer = styled.div`

`;

const Header = styled.div`

`;

const Content = styled.div`

`;

const Footer = styled.div`

`;

const Compoennt = () => (
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

// or with Styled prefix, still messed up
<StyledComponent>
  <StyledHeader>
    <OtherComponentA />
  </StyledHeader>
  <StyledContent>
    <OtherComponentB />
  </StyledContent>
  <StyledFooter>
    <OtherComponentC />
  </StyledFooter>
</StyledComponent>
```

#### Component/styled.Component.ts
That's when we extract all styled components out to a `Component/styled.Component.ts` file and `import * as S` to use:

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

And for export part, rename in the entry file as you want. It doesn't affect your usage inside the component folder.

```ts
// Component/index.ts
export { Component } from './Component';
export { Componsnt as StyledComponent } from './styled.Component';
```

You might feel strange about the `import * as S` convention, that's [another rule we will discuss later](#import-as-s). The main point here is to split these styled components into another file to keep the core `html dom tree` part clean and simple.

> Again, `styled-components` is an opinioned choice, you can replace it with other css solutions like `less/scss` or `styled-jsx` as you want. Just replace the file name with `Component.less`, `style.Component.ts`. The key is to keep the representiveness for css dom tree.


#### Logic-heavy Components

And when the logic part gets too complex, you might also feel hard to manage those state and callbacks.

1. Unnecessary initial render when state is not ready.

2. Some state only used for render must be enhanced before branch render because of the limitation of hooks

3. Unnecessary re-render caused by api hooks

4. Hard to reuse when I want to write a new component with the same logic but different UI(component variant), or a new component with the same UI but different logic(api upgrade).

5. Hard to write unit test with all the api hooks and side effects embedded.

6. Last but most important, hard to read and understand.

```ts
// Component.ts
interface State {

}

interface Props {

}

interface StateInReducer {

}

interface ReducerAction {

}

const reducer = (state: StateInReducer, action: ReducerAction): StateInReducer => {
  // could be very long
}

const Component = (props) => {
  // useContext inside
  const [router] = useRouter();
  // useState
  const [state, setState] = useState<State>();
  // useMemo
  const stateOnlyForRender = useMemo(() => {

  }, [state]);

  // useReducer
  const [stateInReducer, dispatch] = useReducer<StateInReducer>(reducer, state);

  // useReduer inside, api hooks like useApollo, useApi
  const { data: resources, loading, error } = useResources(state);

  // useEffect
  useEffect(() => {
    setTimeout(() => setState(/* some delayed initial state */));
  }, []);

  if (error) {
    return null;
  }

  if (loading) {
    return 'Loading...';
  }

  // Can't write hook after branch code here!
  // const stateOnlyForRender = useMemo(() => {

  // }, [state]);

  // The whole vdom tree re-renders even when all the states are not changed.
  return (
    <div>
      {/* use state, stateOnlyForRender, stateInReducer, resources somehow to render sth */}
      <button onClick={() => setState()}>change state</button>
      <button onClick={() => dispatch('action1')}>dispatch action 1 </button>
      <button onClick={() => dispatch('action2')}>dispatch action 2</button>
    </div>
  )
}
```

#### Component/useComponent.ts

Again, let's extract all the logics into another hook file and see what happens:

```ts
// Component/useComponent.ts
export interface ComponentProps {

}

export interface ComponentHookProps {

}

// This hook is reusable when another component wants the same logic but different UI.
// props => hookProps, no vdom tree, no event simulating, easier the unit test.
export useComponent = (props: ComponentProps): ComponentHookProps => {
  // useContext inside
  const [router] = useRouter();
  // useState
  const [state, setState] = useState<State>();

  // useReducer
  const [stateInReducer, dispatch] = useReducer<StateInReducer>(reducer, state);

  // useReduer inside, api hooks like useApollo, useApi
  const { data: resources, loading, error } = useResources(state);

  // useEffect
  useEffect(() => {
    setTimeout(() => setState(/* some delayed initial state */));
  }, []);

  return {
    state,
    setState,
    stateInReducer,
    dispatch,
    resources,
    loading,
    error,
  }
}

// Component/Component.ts

import { ComponentProps, ComponentHookProps, useComponent } from './useComponent';

// This renderer is reusable when another component wants the same UI but different logic.
// props => dom true, no side effects inside, easy to unit test.
const ComponentRenderer: React.FC<ComponentProps & ComponentHookProps> = React.memo((props) => {
  const { state, error, loading, setState, dispatch, resources } = props;
  // render only state can be written after the render branch
  const stateOnlyForRender = useMemo(() => {

  }, [state]);

  return (
    <div>
      {/* use state, stateOnlyForRender, stateInReducer, resources somehow to render sth */}
      <button onClick={() => setState()}>change state</button>
      <button onClick={() => dispatch('action1')}>dispatch action 1 </button>
      <button onClick={() => dispatch('action2')}>dispatch action 2</button>
    </div>
  )
})

const Component: React.FC<ComponentProps> = (props) => {
  const hookProps = useComponent(props);
  const { error, loading } = hookProps;

  // Moved meta data for render flow control down and keep the core render code on the top.
  if (error) {
    return null;
  }

  if (loading) {
    return 'Loading...';
  }

  // ComponentRenderer part would not be re-rendered if props and hookProps are not changed.
  return <ComponentRenderer {...props} {...hookProps} />;
}

```

### Type-heavy components

You must have noticed, each one type above has extra type requirements. We have several problems now:

1. The definitions of types could be very long and disturbing coders' attention.

2. Reuse among different types could be hard to manage.

3. Common props used by styled components and Component itself must be defined in styled file if we don't want duplication.

```ts
// Component/styled.Component.ts

interface StyledProps {

}

interface HeaderStyledProps {

}

export Component = styled.div<StyledProps>`

`;

export HeaderStyledProps = styled.div<HeaderStyledProps>`

`;

// Component/useComponent.ts
import { StyledProps, HeaderStyledProps } from './styled.Component';

// Some props for the Component is written apartly in styled.Component.
export interface Props extends Pick<StyledProps, 'size'> {

}

// Some hook props for the renderer is written apartly in styled.Component.
export interface HookProps extends Pick<HeaderStyledProps, 'fixedHeader'> {

}

export useComponent = (props: Props): HookProps => {

}


// Component/Component.ts
import { Props, HookProps, useComponent } from './useComponent';
import * as S from './styled.Component';


const Component: React.FC<Props & HookProps> = (props) => {
}
```

#### Component/types.Component.ts

Introduce a separate file to manage types inside a component is very natural now.

```ts
// Component/types.Component.ts
export interface Props {
  // All props for Component written together.

}

// Common props for styled components can be picked from Props.
export interface StyledProps extends Pick<Props, 'size'> {
  // Special props for styled components written here
}

export interface HookProps {
  // Additional props added by hook written together.
}

export interface HeaderStyledProps extends Pick<HookProps, 'fixedHeader'> {

}

// Component/styled.Component.ts
import { StyledProps, HeaderStyledProps } from './types.Component';

export Component = styled.div<StyledProps>`

`;

export HeaderStyledProps = styled.div<HeaderStyledProps>`

`;

// Component/useComponent.ts
import { Props, HookProps } from './types.Component';


export useComponent = (props: Props): HookProps => {

}

// Component/Component.ts
import { Props, HookProps } from './types.Component';
import { useComponent } from './useComponent';
import * as S from './styled.Component';

const Component: React.FC<Props & HookProps> = (props) => {
}
```

### Const-heavy Components

What's const-heavy? Is that even a word? You must have this doubt when you see it. Let me explain. Imagine you have a form component using a 3rd-party form library to collect the form values from inputs for you. You need to transfer the form values into api values to submit or transfer reversely to show filled fields. You have a static array of a certain type to render a drop down list for the user to select. The code might look like this with the above file types:

```ts
// Component/types.Component.ts
export interface Resource {

}

export interface ApiValues {
  resourceId: number;
}

export interface FormValues {
  resource: Resource;
}

export interface Props {}

export interface HookProps {}

// Component/useComponent.ts
import { FormValues, ApiValues } from './types.Component';

const useComponent = () => {
  const buildApiValuesFromFormValues = (formValues: FormValues): ApiValues => {}

  const buildFormValuesFromApiValues = (apiValues: ApiValues): FormValues => {}

  const { data, loading, error } = useApi();
  const initialValues = buildFormValuesFromApiValues(data);

  const onSubmit = async (values: FormValues) => {
    await callApi(buildApiValuesFromFormValues(values));
  }

  return {
    initialValues,
    loading,
    error,
    onSubmit,
    // ...
  }
}

// Component/Component.ts
import { Form } from 'some-3rd-party-form-library';

import { Resource } from './types.Component';

const resources: Resource = [
  {
    id: 1,
    name: 'name 1',
  },
  // duplicate the same structure for 10 times
];

const ComponentRenderer = ({ initialValues, onSubmit }) => {

  return (
    <S.Component>
      <Form initialValues={initialValues}>
        <select>
          {resources.map(resource => (
            <option key={resource.id} value={resource.id}>{resource.name}</option>
          ))}
        </select>
        <button onClick={onSubmit}>Submit</button>
      </Form>
    </S.Component>
  )
}

const Component: React.FC<ComponentProps> = (props) => {
  const hookProps = useComponent(props);
  const { error, loading } = hookProps;

  if (error) {
    return null;
  }

  if (loading) {
    return 'Loading...';
  }

  return <ComponentRenderer {...props} {...hookProps} />;
}

```

#### Component/const.Component.ts

You can see the `resources` constant doesn't change a lot. Also the two converter function don't change a lot because the logic is between api values(structure defined by backend) and form values(structure defined by designer). But the problem is 

1. These variables and functions could be very simple and very long. They can be too long to affect the readibility.

2. The functions can be very hard to unit test if they locate inside the hook function and use other variables unexpectedly.

You might feel putting them in a `utils` file is more commonly practiced, but remember [functions are the first-class objects in the JS world](https://www.developintelligence.com/blog/2016/10/javascript-functions-as-first-class-objects/).

```ts

// Component/types.Component.ts
export interface Resource {

}

export interface ApiValues {
  resourceId: number;
}

export interface FormValues {
  resource: Resource;
}

export interface Props {}

export interface HookProps {}
// const.Component.ts
import { Resource } from './types.Component';

export const resources: Resource = [
  {
    id: 1,
    name: 'name 1',
  },
  // duplicate the same structure for 10 times
];

export const buildApiValuesFromFormValues = (formValues: FormValues): ApiValues => {}

export const buildFormValuesFromApiValues = (apiValues: ApiValues): FormValues => {}

// useComponent.ts
import { FormValues } from './types.Component';
import { 
  buildApiValuesFromFormValues, 
  buildFormValuesFromApiValues, 
} from './const.Component';

const useComponent = () => {
  const { data, loading, error } = useApi();
  const initialValues = buildFormValuesFromApiValues(data);

  const onSubmit = async (values: FormValues) => {
    await callApi(buildApiValuesFromFormValues(values));
  }

  return {
    initialValues,
    loading,
    error,
    onSubmit,
    // ...
  }
}

// Component/Component.ts
import { Form } from 'some-3rd-party-form-library';

import { resources } from './const.Component';

const ComponentRenderer = ({ initialValues, onSubmit }) => {

  return (
    <S.Component>
      <Form initialValues={initialValues}>
        <select>
          {resources.map(resource => (
            <option key={resource.id} value={resource.id}>{resource.name}</option>
          ))}
        </select>
        <button onClick={onSubmit}>Submit</button>
      </Form>
    </S.Component>
  )
}

const Component: React.FC<ComponentProps> = (props) => {
  const hookProps = useComponent(props);
  const { error, loading } = hookProps;

  if (error) {
    return null;
  }

  if (loading) {
    return 'Loading...';
  }

  return <ComponentRenderer {...props} {...hookProps} />;
}
```

What else can be identified as constants? Almost everything you think don't belong to or just belong to or too long to be put together with `html dom tree`, `css dom tree` and `js business logic`, e.g.

```ts
// some reusable jsx fragments
const icon = (
  <svg>
    <path>
    </path>
  </svg>
);

// some jsx fragments as enhanced strings
const resources = [
  {
    id: 1,
    name: <span>Long name with <strong>strong part</strong></span>,
  },
];

// some page meta data used in the header
const pageMeta = {}

// some mapping data from one type to another, like an object version of converter functions above
const resourceById: Record<number, Resource> = {
  1: {
    id: 1,
    name: <span>Long name with <strong>strong part</strong></span>,
  }
}

// some static options for 3rd-party libraries
const options = {}

// some static configs for different variants of the component
const fontSizeByVariant = {};

// some util functions 
const groupResourceByCityName = (resources: Resource[]): Record<string, Resource[]> => {};

// some graphql queries
const apiValueQuery = gql`

`;
```

### Component/Component.ts

After `style`, `hook`, `type` and `const` are extracted out separatedly, let's see what does the core `Component` file looks like now.

```ts
// Component/Component.ts
import * as S from './styled.Component';
import { useComponent } from './useComponent';
import { Props, HookProps } from './types.Component';
import { someMetaData } from './const.Component';

// Core vdom structure is enhanced
const ComponentRenderer: React.FC<Props & HookProps> = React.memo((props) => {
  // some render-only state variables/hooks

  return (
    <S.Component>
      {/*  */}
    </S.Component>
  );
});

const Component: React.FC<Props> = (props) => {
  const hookProps = useComponent(props);

  // Non-core branch render logic is deferred
  if (hookProps.someSpecialCondition) {
    return 'sth else';
  }

  return <ComponentRenderer {...props} {...hookProps} />;
}
```

### Component/index.ts

As you can see we exported almost everything in the separated files, but it's obvious we don't want other components to reuse them all. That's why I want to introduce the `entry file` here:

```ts
// Component/index.ts
export { Component } from './Component';

// Export other things if necessary.
// export { useComponent } from './useComponent';
// export type { Props as ComponentProps } from './types.Component';

```

If you are familiar with object-oriented programing language, you would find this is just the `public-protected-private` mode.

- `public`: Exported in index.ts
- `protected`: Not exported in index.ts but exported to reuse inside the component.
- `private`: Not exported at all
