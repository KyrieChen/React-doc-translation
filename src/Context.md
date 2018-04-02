## Context
Context 提供了一种方法来在组件树上传递数据，这种方法不需要手动的在每个层级传递数据下去。
在典型的React应用，数据是通过props来自上而下得传递的（父组件到子组件），但是某些类型的props，例如 locale preference 和 UI theme，在一个应用中很多组件都需要这些props，这就导致了难以处理。Context提供了一种不需要显示地在组件树的每一层级都传递props的在组件间共享value的方法。

### When to Use Context
Context是被设计用来分享data的，可以看做是react组件树的“global”变量，比如目前已经通过验证的用户，主题或者是语言偏好。举个例子，在下面的代码中，我们手动地通过一个"theme"prop来给按钮组件加上样式。
```javascript
function ThemedButton(props) {
  return <Button theme={props.theme} />;
}

// An intermediate component
function Toolbar(props) {
  // The Toolbar component must take an extra theme prop
  // and pass it to the ThemedButton
  return (
    <div>
      <ThemedButton theme={props.theme} />
    </div>
  );
}

class App extends React.Component {
  render() {
    return <Toolbar theme="dark" />;
  }
}
```
使用Context，我们可以避免在中间元素传递prop。
```javascript
// Create a theme context, defaulting to light theme
const ThemeContext = React.createContext('light');

function ThemedButton(props) {
  // The ThemedButton receives the theme from context
  return (
    <ThemeContext.Consumer>
      {theme => <Button {...props} theme={theme} />}
    </ThemeContext.Consumer>
  );
}

// An intermediate component
function Toolbar(props) {
  return (
    <div>
      <ThemedButton />
    </div>
  );
}

class App extends React.Component {
  render() {
    return (
      <ThemeContext.Provider value="dark">
        <Toolbar />
      </ThemeContext.Provider>
    );
  }
}
```
> 注意：不要仅用Context来避免在一些层级传递prop下去。坚持在许多层级的许多组件都需要相同的data需要的情况下使用。

### API
#### React.createContext
```javascript
const {Provider, Consumer} = React.createContext(defaultValue);
```
创建一对`{ Provider, Consumer }`。接收一个可选参数`defaultValue`，用来在没有Provider祖先时传递给Consumers使用。

#### Provider
```javascript
<Provider value={/* some value */}>
```
一个允许Consumers订阅context改变的组件。接收一个`value`属性用来传给以来其Consumers子节点。一个Provider能连接多个Consumers。Provider能被嵌套来更深的覆盖value在一棵组件树内。

#### Consumer
```javascript
<Consumer>
  {value => /* render something based on the context value */}
</Consumer>
```
一个订阅context改变的React组件，要求用一个函数作为孩子节点。这个函数接受目前的context值和返回一个React节点。每当Provider的值变化，所有consumers都会重新渲染(re-render)。通过`Object.is()`来比较新值和旧值来决定是否是改变。（当传入对象作为`value`会造成一些问题：见Caveats）
> 注意：更多的关于这一块的信息，见[render props](https://reactjs.org/docs/render-props.html).

### Examples
#### Dynamic Context
一个更复杂的例子关于动态的theme值。
##### theme-context.js
```javascript
export const themes = {
  light: {
    foreground: '#ffffff',
    background: '#222222',
  },
  dark: {
    foreground: '#000000',
    background: '#eeeeee',
  },
};

export const ThemeContext = React.createContext(
  themes.dark // 默认值
);
```
##### themed-button.js
```javascript
import {ThemeContext} from './theme-context';

function ThemedButton(props) {
  return (
    <ThemeContext.Consumer>
      {theme => (
        <button
          {...props}
          style={{backgroundColor: theme.background}}
        />
      )}
    </ThemeContext.Consumer>
  );
}

export default ThemedButton;
```
##### app.js
```javascript
import {ThemeContext, themes} from './theme-context';
import ThemedButton from './button';

// An intermediate component that uses the ThemedButton
// 一个起媒介作用的组件使用了ThemedButton
function Toolbar(props) {
  return (
    <ThemedButton onClick={props.changeTheme}>
      Change Theme
    </ThemedButton>
  );
}

class App extends React.Component {
  constructor(props) {
    super(props);
    this.state = {
      theme: themes.light,
    };

    this.toggleTheme = () => {
      this.setState(state => ({
        theme:
          state.theme === themes.dark
            ? themes.light
            : themes.dark,
      }));
    };
  }

  render() {
    // The ThemedButton button inside the ThemeProvider
    // uses the theme from state while the one outside uses
    // the default dark theme
    return (
      <Page>
        <ThemeContext.Provider value={this.state.theme}>
          <Toolbar changeTheme={this.toggleTheme} />
        </ThemeContext.Provider>
        <Section>
          <ThemedButton />
        </Section>
      </Page>
    );
  }
}

ReactDOM.render(<App />, document.root);
```
#### Consuming Multiple Contexts
为了保持context快速的重新渲染(re-render)，React需要使每一个context消费者为一个独立的节点在节点树里。
```javascript
// Theme context, default to light theme
const ThemeContext = React.createContext('light');

// Signed-in user context
const UserContext = React.createContext();

// An intermediate component that depends on both contexts
function Toolbar(props) {
  return (
    <ThemeContext.Consumer>
      {theme => (
        <UserContext.Consumer>
          {user => (
            <ProfilePage user={user} theme={theme} />
          )}
        </UserContext.Consumer>
      )}
    </ThemeContext.Consumer>
  );
}

class App extends React.Component {
  render() {
    const {signedInUser, theme} = this.props;

    // App component that provides initial context values
    return (
      <ThemeContext.Provider value={theme}>
        <UserContext.Provider value={signedInUser}>
          <Toolbar />
        </UserContext.Provider>
      </ThemeContext.Provider>
    );
  }
}
```
如果两个或者多个context值(value)经常一起使用，你就可能需要考虑创建你自己渲染属性组件，以提供两者。

#### Accessing Context in Lifecycle Methods
从生命周期方法中的context取得值(value)是一个相对多件的使用案例。不需要为每个生命周期方法都添加context，你只需要把它作为属性传递，然后像一个普通属性一样使用它。
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
#### Consuming Context with a HOC
一些类型的contexts会被许多组件消费(consume)，例如主题和局部化。使用`<Context.Consumer>`元素显式包装每个依赖可能非常繁琐。高阶组件(HOC)会在这方面帮助我们。
例如，一个按钮组件可能像这样消费(consume)一个主题(theme)context：
```javascript
const ThemeContext = React.createContext('light');

function ThemedButton(props) {
  return (
    <ThemeContext.Consumer>
      {theme => <button className={theme} {...props} />}
    </ThemeContext.Consumer>
  );
}
```
对于几个组件来说这是没问题的，但是如果我们想在很多地方使用这个主题context会怎么样？我们会创建一个高阶组件叫做`withTheme`:
```javascript
const ThemeContext = React.createContext('light');

// This function takes a component...
export function withTheme(Component) {
  // ...and returns another component...
  return function ThemedComponent(props) {
    // ... and renders the wrapped component with the context theme!
    // Notice that we pass through any additional props as well
    // 注意我们也要传递所有其他的属性下去
    return (
      <ThemeContext.Consumer>
        {theme => <Component {...props} theme={theme} />}
      </ThemeContext.Consumer>
    );
  };
}
```
现在，任何依赖于主题contextd的组件可以很容易的通过我们创建的`withTheme`函数订阅。
```javascript
function Button({theme, ...rest}) {
  return <button className={theme} {...rest} />;
}

const ThemedButton = withTheme(Button);
```
#### Forwarding Refs to Context Consumers
渲染属性API的一个问题是refs不会自动地传递到包裹组件。为了解决这个问题，使用`React.forwardRef`:
##### fancy-button.js
```javascript
class FancyButton extends React.Component {
  focus() {
    // ...
  }

  // ...
}

// Use context to pass the current "theme" to FancyButton.
// Use forwardRef to pass refs to FancyButton as well.
export default React.forwardRef((props, ref) => (
  <ThemeContext.Consumer>
    {theme => (
      <FancyButton {...props} theme={theme} ref={ref} />
    )}
  </ThemeContext.Consumer>
));
```
##### app.js
```javascript
import FancyButton from './fancy-button';

const ref = React.createRef();

// Our ref will point to the FancyButton component,
// And not the ThemeContext.Consumer that wraps it.
// This means we can call FancyButton methods like ref.current.focus()
<FancyButton ref={ref} onClick={handleClick}>
  Click me!
</FancyButton>;
```
### Caveats(警告)
因为context使用了引用标识去决定何时重新渲染(re-render)。这会造成一些陷阱：当提供者父元素重新渲染的时候会无意中触发消费者渲染。例如,下面的代码会让所有消费者在每次Providerc重新渲染的时候都重新渲染，因为总是创建一个新的对象给value:
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
为了解决这个问题，提升value至父组件的状态。
```javascript
class App extends React.Component {
  constructor(props) {
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