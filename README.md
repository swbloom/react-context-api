# What is the Context API?
In a typical React app, data is passed top-down via props, like this:

```javascript
class App extends React.Component {
  state = {
    theme: "dark"
  }
  render() {
    return <Toolbar theme={this.state.theme} />
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

`Button` wants to know the theme, so it needs to be passed down from `App`, through `Toolbar`, and into `ThemedButton` for the `Button` to finally get access to it. This process, passing components down from a higher parent down through components that don't even necessarily need it, so they can get to components lower down the chain, is called 'prop drilling'.

It's kind of a pain in the ass. Usually the way this is solved is through some kind of 3rd party state management library like Redux or Mobx.

# Now, with context!

```javascript
const ThemeContext = React.createContext();

class App extends React.Component {
  render() {
    return (
      <ThemeContext.Provider value="dark">
        <Toolbar />
      </ThemeContext.Provider>
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
  </ThemeContext.Consumer>
)
```

By taking advantage of the `consumer/provider` components offered to us by `createContext`, we can 'skip' passing `theme` through props and access it directly in `Toolbar`!

# Breaking it down

The Context API can be broken down into three basic pieces:


## The createContext method
The `createContext` method has this signature:

```javascript
React.createContext(defaultValue);
```

It returns an object which holds a `{ Provider, Consumer }` pair. You can destructure it like this:

```javascript
const { Provider, Consumer } = React.createContext()

// which gives you access to a <Provider> component
<Provider></Provder>

// and a <Consumer> component
<Consumer></Consumer>
```

or like this in the 'compound component' style:

```javascript
const Context = React.createContext()

// now we have
<Context.Provider></Context.Provider>

// and
<Context.Consumer></Context.Consumer>
```

## The Provider

The `Provider` looks like this:
```javascript
<Provider value={/* some value */}>
```

It's a React component that you can pass some value in to as a prop. Whenever that value changes, any of the Provider's `Consumer` pairs will be notified of the updated value.

A single `Provider` can have many consumers.

## Consumer

The `Consumer` looks like this:

```javascript
<Consumer>
  {value => <div>{value}</div>}
</Consumer>
```

The `Consumer` receives whatever value was passed into its `<Provider>` pair.

Whenever the `Provider`'s value changes, the `Consumer` re-renders and receives the new value (as a `render prop`!)

## Looking at it all together:
Let's imagine we want different parts of our application to know which language our app should currently be rendering in:

```javascript
// create our Provider and Consumer pair, store them in LanguageContext
const LanguageContext = React.createContext()

// first, we wrap the highest level component we want to know about our language in our Provider
const App = () => (
  <div>
    <LanguageContext.Provider value={'english'}>
      <Header />
    </LanguageContext>
  </div>
)

// just a plain ol' react component, nothing to see here
const Header = () => (
  <header>
    <CurrentLanguage />
  </header>
)

// We wrap our child component that wants to know about the parent's value in our Consumer
const CurrentLanguage = () => (
  <LanguageContext.Consumer>
    {val => <div>Current Language: {val}</div>} {/* english */}
  </LanguageContext.Consumer>
)
```

## Accessing Context in Lifecycle Methods
Context can be accessed in lifecycle methods through a higher order function that returns your consumed component, like this:

```javascript
class Button extends React.Component {
  componentDidMount() {
    // ThemeContext value is this.props.theme
  }

  componentDidUpdate(prevProps, prevState) {
    // Previous ThemeContext value is prevProps.theme
    // New ThemeContext value is this.props.theme
  }

  render() {
    const {theme, children} = this.props;
    return (
      <button className={theme ? 'dark' : 'light'}>
        {children}
      </button>
    );
  }
}

export default props => (
  <ThemeContext.Consumer>
    {theme => <Button {...props} theme={theme} />}
  </ThemeContext.Consumer>
);
```

## Some Gotchas
Context uses reference identity to determine when to re-render, which means this:
```javascript
class App extends React.Component {
  render() {
    return (
      <Provider value={{something: 'something'}}>
        <Toolbar />
      </Provider>
    );
  }
}
```

will cause all child consumers to re-render (since a new object literal gets passed in every time the App component renders, regardless of if the value of `value` has changed.

To solve this, simply lift the value into `App`'s state:

```javascript
class App extends React.Component {
  constructor(props) {
    super(props);
    this.state = {
      value: {something: 'something'},
    };
  }

  render() {
    return (
      <Provider value={this.state.value}>
        <Toolbar />
      </Provider>
    );
  }
}
```

## Is Context the Redux killer?
As Redux maintainer Mark Erikson states in his blog post <a href="http://blog.isquaredsoftware.com/2018/03/redux-not-dead-yet/">Redux - Not Dead Yet!</a>:

"Context is great for passing down props without needing intermediary components. If that's all you're using Redux for, then you can safely use context instead.

Context does NOT give you Redux DevTools, Redux middleware, the ability to trace state updates, and all kinds of other goodies Redux offers you"

Here's Dan Abramov's take:

"Context is an advanced feature and is subject to change. In some cases its conveniences outweigh its downsides so some libraries like React Redux and React Router choose to rely on it despite the experimental nature.

The important part here is the word libraries. If context changes its behavior, we as library authors will need to adjust. However, as long as the library doesn’t ask you to directly use the context API, you as the user shouldn’t have to worry about changes to it.

React Redux uses context internally but it doesn’t expose this fact in the public API. So you should feel much safer using context via React Redux than directly because if it changes, the burden of updating the code will be on React Redux and not you."

## Should you use context in your production applications as opposed to Redux?
Probably not. Or if you do, use it for relatively static application wide settings (locale and theme are the two that immediately come to mind).

## So when should you use context?
There are a few uses cases where you might want to use context:
- You have a value you want to share in a lot of different parents of your application and don't want to clutter up your app through prop drilling
- You're relatively new to the app ecosystem and want to take advantage of shared component state, or want to take the first steps towards a flux/redux style approach to state management in your application.
- You're a library maintainer and you want to build a state management solution similar to something like `redux`.

## Additional Reading
- <a href="https://reactjs.org/docs/context.html">Official React Context Documentation</a>
- <a href="https://medium.com/dailyjs/reacts-%EF%B8%8F-new-context-api-70c9fe01596b">React's new Context API
- <a href="http://blog.isquaredsoftware.com/2018/03/redux-not-dead-yet/">Redux - Not Dead Yet!</a>
- <a href="https://medium.freecodecamp.org/replacing-redux-with-the-new-react-context-api-8f5d01a00e8c">Replacing Redux with the new React Context API</a>
- <a href="https://medium.freecodecamp.org/replacing-redux-with-the-new-react-context-api-8f5d01a00e8c">react-waterfall</a> - a minimal state management library built on top of React's Context API
