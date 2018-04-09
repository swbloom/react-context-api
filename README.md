# What is the Context API?
In a typical React app, data is passed top-down via props, like this:

```javascript
class App extends React.Component {
  render() {
    return <Toolbar theme="dark" />
  }
}

const Toolbar = ({ theme }) => (
  <div>
    <ThemedButton theme={props.theme} />
  </div>
)

const ThemedButton = ({ theme }) => (
  <Button theme={props.theme} />
)
```

`Button` needs to know the theme, so it needs to be passed through `Toolbar` for `Button` to get access to it. This process, passing components down from a higher parent down through components that don't even necessarily need it, so they can get to components lower down the chain, is called 'prop drilling'.

This process is known as 'prop drilling', and it's kind of a pain in the ass.

# Now, with context!

```javascript
const ThemeContext = React.createContext();

class App extends React.Component {
  render() {
    return (
      <ThemeContext.Provider value="dark">
        <Toolbar />
      </ThemeContext>
    )
  }
}

const Toolbar = () => (
  <div>
    <ThemedButton />
  </div>
)

const ThemedButton = () => (
  <ThemeContext.Consumer>
    {theme => <Button theme={theme}> />}
  </ThemeContext>
)
```

By taking advantage of the `consumer/provider` components offered to us by `createContext`, we can 'skip' passing `theme` through props and access it directly in `Toolbar`!

# Breaking it down

The Context API can be broken down into three steps:


## createContext
The `createContext` method has this signature:

```javascript
React.createContext(defaultValue);
```

It returns a `{ Provider, Consumer }` pair. You can use it like this:

```javascript
const { Provider, Consumer } = React.createContext()

// ...

<Provider>
<Consumer>
```

or like this:

```javascript
const Context = React.createContext()

// ...

<Context.Provider>
<Context.Consumer>
```

the `createContext` method also accepts a `defaultValue`. We'll talk about what this does in a bit.

## The Provider

The `Provider` looks like this:
```javascript
<Provider value={/* some value */}>
```

It's a React component that you can pass some value in to as a prop. Whenever that value changes, any of the Provider's `Consumer` pairs will be notified of the updated value.

A single `Provider` can have many consumers, and many consumers only ever belong to one Provider.

## Consume

The `Consumer` looks like this:

```javascript
<Consumer>
  {value => <div>{value}</div>}
</Consumer>
```

The `Consumer` receives whatever value was passed into its `<Provider>`. 

Whenever the `Provider`'s value changes, the `Consumer` re-renders and receives the new value (as a `render prop`!)

## Looking at it all together:

## Is Context the Redux killer?
As Redux maintainer Mark Erikson states in his blog post <a href="http://blog.isquaredsoftware.com/2018/03/redux-not-dead-yet/">Redux - Not Dead Yet!</a>:
```
Context is great for passing down props without needing intermediary components. If that's all you're using Redux for, then you can safely use context instead.

Context does NOT give you Redux DevTools, Redux middleware, the ability to trace state updates, and all kinds of other goodies Redux offers you
```
Here's Dan Abramov's take:
```
Context is an advanced feature and is subject to change. In some cases its conveniences outweigh its downsides so some libraries like React Redux and React Router choose to rely on it despite the experimental nature.

The important part here is the word libraries. If context changes its behavior, we as library authors will need to adjust. However, as long as the library doesn’t ask you to directly use the context API, you as the user shouldn’t have to worry about changes to it.

React Redux uses context internally but it doesn’t expose this fact in the public API. So you should feel much safer using context via React Redux than directly because if it changes, the burden of updating the code will be on React Redux and not you.
```

## Should you use context in your production applications as opposed to Redux?
Probably not. Or if you do, use it for relatively static application wide settings (locale and theme are the two that immediately come to mind).

## So when should you use context?
Context is actually a really great thing to introduce to people who are relatively new to React and want to take the next step in thinking about state management within their application. It's a nice gentle introduction to pub/sub and flux architecture.

## Additional Reading
- <a href="https://reactjs.org/docs/context.html">Official React Context Documentation</a>
- <a href="https://medium.com/dailyjs/reacts-%EF%B8%8F-new-context-api-70c9fe01596b">React's new Context API
- <a href="http://blog.isquaredsoftware.com/2018/03/redux-not-dead-yet/">Redux - Not Dead Yet!</a>
- <a href="https://medium.freecodecamp.org/replacing-redux-with-the-new-react-context-api-8f5d01a00e8c">Replacing Redux with the new React Context API</a>
- <a href="https://medium.freecodecamp.org/replacing-redux-with-the-new-react-context-api-8f5d01a00e8c">react-waterfall</a> - a minimal state management library built on top of React's Context API
