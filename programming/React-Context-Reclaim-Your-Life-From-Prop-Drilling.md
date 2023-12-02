---
slug: programming/React-Context-Reclaim-Your-Life-From-Prop-Drilling
created_at: 2023-11-18 10:01
is_template: "0"
featured_image: img/React-Context-Reclaim-Your-Life-From-Prop-Drilling/featured-image.png
title: "React Context: Reclaim Your Life From Prop Drilling Now!"
---
No- it's not what backstage production crews do to make set pieces for your favourite musical. Prop drilling is a problem that basically every React developer has encountered, or WILL encounter, at least once in their career. In this post, I'll go through what prop drilling is, and how we can mitigate it using [React Context](https://reactjs.org/docs/context.html). We'll also touch on some drawbacks of Context, and some caveats to know when using it.

> [!info] This post has supplementary code! Check it out on [GitHub](https://github.com/nitsudgo/blog-react-context-demo)!

# What is "Prop Drilling"?

If this is your first time hearing about the concept of prop drilling, here's a quick summary of what it is and why developers tend to avoid it.

> Anyone who has worked in React would have faced this and if not then will face it definitely. Prop drilling is basically a situation when the same data is being sent at almost every level due to requirements in the final level.
> <br>
> [GeeksForGeeks.org](https://www.geeksforgeeks.org/what-is-prop-drilling-and-how-to-avoid-it/)

Essentially, **prop drilling is when the same data is inefficiently* passed down through many layers of components as props.** The data "drills" through the components the same way a drill, uh, _drills_ through wood. It is a manifestation of the need to pass information from one component down to its (deeply) nested descendant components.

<sub>I specify "inefficiently" because [it is possible](https://blog.logrocket.com/solving-prop-drilling-react-apps/), [through component composition](https://javascript.plainenglish.io/composition-in-react-f02afe24bc46), to pass props through component trees without prop drilling. This is outside the scope of this article, but I highly suggest taking a look at the approach!</sub>

## Example

Consider the code listing below- it shows us what prop drilling looks like. You might even recognize the prop drilling pattern in your own projects:

```jsx
function ParentComponent () {
    let [state, setState] = useState(0);
    return (
        <ChildComponent state={state} setState={setState} />;
    );
}

function ChildComponent ({state, setState}) {
    return (
        <>
            The value of state in ChildComponent is {state}!
            <GrandChildComponent state={state} setState={setState} />
            <GrandChildComponent state={state} setState={setState} />
        </>
    );
}

function GrandChildComponent ({state, setState}) {
    return (
        <>
            <GreatGrandChildComponent state={state} setState={setState} />
        </>
    );
}

function GreatGrandChildComponent ({state, setState}) {
    const addOne = () => {
        setState(x => x + 1);
    };

    return (
        <div>
            The value of state in GreatGrandChildComponent is {state}!
            <button onClick={addOne}>Add One</button>
        </div>
    );
}
```

Although it's a very simple (and untested) example, it illustrates the concept, and drawbacks, of prop drilling quite nicely. Let's appreciate the following pain points:

- **There's a lot of unused data flying around:** some components may not need to use `state` or `setState`, but their children _do_. As a result, we end up having to pass it all down the _entire_ tree anyway. For example:
    - We only need to use `setState` in `GreatGrandChildComponent`, but to get it there, all of its ancestor components need to receive `setState` as props so they can pass it downwards.
    - `GrandChildComponent` doesn't actually use the value of `state` at all, unlike `ChildComponent` and `GreatGrandChildComponent`, which render the value as `{state}`. However, it still needs to pass the props on for its descendants.
- **There's a lot of repetition:** since we need to accept and pass the same props on at every level, what happens if we change the type/format of `state`, or even just its variable na
    - We need to make sure that our changes are accounted for in all of the descendants down the tree. This includes components that don't use the data and just need to pass it through!

<div style="width:100%;height:0;padding-bottom:56%;position:relative;"><iframe src="https://giphy.com/embed/3NtEfcZmzePW6Ps7Gn" width="100%" height="100%" style="position:absolute" frameBorder="0" class="giphy-embed" allowFullScreen></iframe></div><p><a href="https://giphy.com/gifs/heyduggee-3NtEfcZmzePW6Ps7Gn">via GIPHY</a></p>

It's no wonder why we should avoid excessive prop drilling! As simple projects grow and become more complex, this issue can slowly start creeping into the codebase. If left unchecked, this can result in an unsightly, unmaintainable quagmire of code nobody wants to touch.

Fortunately for us, there's one possible way around it- and that's with React's Context API!

# What is React Context?

The [React Context API](https://reactjs.org/docs/context.html) is a feature that allows a component to access data regardless of where it is in a component tree. 

<br>
<br>

Well... sort of....

You can _almost_ think of it like React State, except instead of being localized within a single component and shared via props, it's localized and accessible by any component within a _React Context Provider_.

## React Context Workflow

The code listing below shows how Context works. In practice, you would probably structure everything a bit better and move context-related things to a separate file to keep things tidy.

```jsx
// 1. Import stuff we need
import { useState, useContext, createContext } from 'react';

// 2. Create a context object
const SomeContext = createContext({state: 0, setState: () => {}});


function App () {
    // 3. Initialise state to manage the context value
    let [state, setState] = useState(0);
    return (
        // 4. Wrap consumers with the context provider
        // 5. Pass in the context value
        <SomeContext.Provider value={{ state, setState }}>
            <ChildComponent />
        </SomeContext.Provider>
    );
};


function ChildComponent () {
    // 6. Consume context in the children of the provider
    const { state, setState } = useContext(SomeContext);
    return (
        // 7. Use context values like normal
        <button onClick={() => setState(x => x + 1)}>{state}</button>
    );
};
```

Following the commented annotations in further detail, the Context API workflow is as follows:

- **Import some React functions to set up the Context infrastructure:**
    - **useState:** We will use React state to manage getting and setting the value of the Context.
    - **createContext:** We need this function to create our actual Context object.
    - **useContext:** We need this hook when we want to _consume_ (i.e. use) our Context values.
- **Create the Context object.**
    - This represents our Context. We consume this to access its data, and use its Provider to define the localised scope where its data is consumable.
    - The object argument passed in represents the default value of the Context object.
- **Initialise state to manage getting/setting the context value.**
    - In this example, the value is a simple integer. It can actually be anything, much like a typical React state value.
- **Wrap the Context consumers with the Context Provider.**
    - Any component that is _not_ a child of a particular Context's Provider will _not_ be able to consume the Context!
    - For this example, we are wrapping the Provider at the top level of the app. The location of your Provider wrapping will depend on your component tree and what you are using context for. As an example, if you are using Context to manage a complex form, you would probably want to wrap the form's top-level component in the Provider so that all children of the form can access the form state.
- **Pass the Context getter/setter into the Context Provider.**
    - In this case, we pass in `state` (getter) and `setState` (setter).
    - These will be accessible to consumers of the Context.
- **Consume the Context in any child of the Context Provider.**
    - Pass the Context object into the `useContext` hook, and use object de-structuring to get the data out.
- **Use the data like normal.**

# How does Context Solve Prop Drilling?

As you can probably guess, Context helps us get around prop drilling by making data directly available to a component. If components can access the data they need directly, there is no need to pass props down to their level!

Here's the prop drilling example from the previous section re-written to use Context instead. Let's see how Context helps us remove prop drilling entirely:

```jsx
function ParentComponent () {
    let [state, setState] = useState(0);
    return (
        <SomeContext.Provider value={{ state, setState }}>
            <ChildComponent state={state} setState={setState} />
        </SomeContext.Provider>
    );
}

function ChildComponent () {
    const { state } = useContext(SomeContext);
    return (
        <>
            The value of state in ChildComponent is {state}!
            <GrandChildComponent state={state} setState={setState} />
            <GrandChildComponent state={state} setState={setState} />
        </>
    );
}

function GrandChildComponent () {
    const { state } = useContext(SomeContext);
    return (
        <>
            <GreatGrandChildComponent state={state} setState={setState} />
        </>
    );
}

function GreatGrandChildComponent () {
    const { state, setState } = useContext(SomeContext);
    const addOne = () => {
        setState(x => x + 1);
    };

    return (
        <div>
            The value of state in GreatGrandChildComponent is {state}!
            <button onClick={addOne}>Add One</button>
        </div>
    );
}
```

<br>

> [!tip] If you're itching to play with a working example, don't forget about the supplementary code on [GitHub](https://github.com/nitsudgo/blog-react-context-demo)!


# When would I NOT use Context?

After reading about how amazing Context can be for solving our prop drilling problem, you might be asking: "why wouldn't I use Context for everything?". Well, let's look at two big cons of Context (lol):

## Context makes components less reusable

When a component consumes data from context, we lose the ability to easily reuse that component for other things. For example, let's say we have a styled component that is meant to display some data:

```jsx
function StyledComponentNoContext ({ data }) {
    return (
        <div className="sexy-styled-component">
            {JSON.stringify(data)}
        </div>
    )
}

function StyledComponentYesContext () {
    const { data } = useContext(SomeContext);
    return (
        <div className="sexy-styled-component">
            {JSON.stringify(data)}
        </div>
    )
}
```

When receiving the data via props, the component is readily reusable because we can pass in different props if we want something else rendered. However, using Context to get our data, the component is basically hardcoded to receive data from a particular Context object. We've lost the ability to easily reuse our component!

## Context consumers are always re-rendered when the context changes

Any component that consumes Context will _always_ trigger a re-render of itself and its children if the context data changes. This issue will only really be tangible for larger, more complex component trees where rendering is expensive. Even so, I still think it's good to know that it happens and address it, even in smaller projects.

Imagine you have a very complex page or form that handles a lot of data, and you pass the data to the relevant components using a JSON object in Context. If _anything_ changes within the Context's JSON value, _all_ of the consumers of that Context i.e. anything that has called `useContext(<the context>)` will re-render. This can cause devastating performance hits to your application ☠️

```jsx
// If the value of data in SomeContext changes...
// ... ALL of the context consumers will re-render...
// ... even if they didn't use the values that changed!

function SomeExpensiveComponentA () {
    const { data } = useContext(SomeContext);
    render (
        // Some expensive rendering operation
    )
}

function SomeExpensiveComponentB () {
    const { data } = useContext(SomeContext);
    render (
        // Some expensive rendering operation
    )
}

function SomeExpensiveComponentC () {
    const { data } = useContext(SomeContext);
    render (
        // Some expensive rendering operation
    )
}
```

As of the writing of this post, there is still no native, straightforward way to directly opt-out of this behavior. However, there are other ways to prevent the unnecessary rendering! I will be covering this topic, and how to address it in a future article!

# Wrapping Up

We've seen that Context is a useful feature in React that we can use to mitigate cases of prop drilling. It works by providing our components the data they need no matter where they are in the component tree, as long as they are children of the Context Provider. We also took a look into the pros and cons of using Context, and how situational their use can be.

At the end of the day, React Context is no silver bullet. It's just another tool in your belt. It's up to you to decide on your approach and what is best for your projects. Now that you know, go build something with it!

<div style="width:100%;height:0;padding-bottom:75%;position:relative;"><iframe src="https://giphy.com/embed/vzO0Vc8b2VBLi" width="100%" height="100%" style="position:absolute" frameBorder="0" class="giphy-embed" allowFullScreen></iframe></div><a href="https://giphy.com/gifs/vzO0Vc8b2VBLi">via GIPHY</a>

**Watch this space for my next post, where I'll talk about addressing Context's gratuitous re-rendering with React Memo**!