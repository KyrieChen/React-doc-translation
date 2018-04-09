## Mounting
当一个组件的实例被创造和插入到DOM中的时候，这些方法会被调用。
- constructor()
- static getDerivedStateFromProps()
- componentWillMount() / UNSAFE_componentWillMount()
- render()
- componentDidMount()

## Updating
属性和状态的改变能造成一次更新(update)。组件在被重新渲染的时候，这些方法会被调用。
- componentWillReceiveProps() / UNSAFE_componentWillReceiveProps()
- static getDerivedStateFromProps()
- shouldComponentUpdate()
- componentWillUpdate() / UNSAFE_componentWillUpdate()
- render()
- getSnapshotBeforeUpdate()
- componentDidUpdate()

## Unmounting
当一个组件从DOM中被移除的时候，这个方法被调用。
- componentWillUnmount()

## Error Handling
在渲染中，在声明周期函数中，或者再任意子组件的构造函数中出现error，这个方法都会被调用。
- componentDidCatch()

## 详细内容

### static getDerivedStateFromProps()
```javascript
static getDerivedStateFromProps(nextProps, prevState)
```
`getDerivedStateFromProps`被调用在组件被实例化后和当它接收到新的属性(prop)的时候。它会返回一个对象去更新状态(state)，或者返回`null`来表示新的属性(prop)不需要任何状态(state)更新。注意，如果父组件造成你的组件重新渲染，这个方法会被调用即使属性们(props)都没有变化。如果你只是想处理属性更改的情况，则可能需要比较新值和之前的值。调用`this.setState()`一般不会触发`getDerivedStateFromProps()`。

### UNSAFE_componentWillMount()
```javascript
UNSAFE_componentWillMount()
```
`UNSAFE_componentWillMount`就在挂载发生之前被调用。它在`render()`之前被调用，因此在这个方法中同步地调用`setState()`不会触发额外的渲染。通常，我们推荐使用`constructor()`来代替初始化状态。避免在这个方法中引入任何副作用或者订阅。对于这些情况，使用`componentDidMount()`代替。这个方法唯一在服务端渲染中调用的生命周期
钩子。
> 注意：这个生命周期函数以前被命名为`componentWillMount`。这个名字会保留到React17之前。使用[rename-unsafe-lifecycles codemod](https://github.com/reactjs/react-codemod#rename-unsafe-lifecycles)自动地更新你的组件。

### componentDidMount()
```javascript
componentDidMount()
```
`componentDidMount()`方法在组件挂载之后立即被调用。需要DOM节点的初始化操作应该放在这里。如果你需要从远程终端加载数据，这是实例化网络请求的好地方。这个方法是个设置订阅的好地方。如果你设置了，不要忘记在`componentWillUnmount()`中取消订阅。在这个方法中调用`setState()`会触发额外的渲染，但是它会在浏览器更新视图之前发生。这保证了即使`render()`会被调用两次在这个案例中，用户也不会看到中间状态(state)。谨慎使用此模式，因为会造成性能问题。但是，如果需要在渲染一些依赖DOM节点大小和位置的东西之前测量该DOM节点。

### UNSAFE_componentWillReceiveProps()
> 注意，建议你使用静态的`getDerivedStateFromProps`生命周期来代替`UNSAFE_componentWillReceiveProps`。想知道更多请见：[Learn more about this recommendation here](https://reactjs.org/blog/2018/03/29/react-v-16-3.html#component-lifecycle-changes)。
`UNSAFE_componentWillReceiveProps()`在一个挂载完的组件收到新属性之前被调用。如果你需要更新状态(state)来响应属性(prop)的变化，例如去重置它，你可能要比较`this.props`和`nextProps`，和通过使用`this.setState()`这个方法来执行状态(state)转换。
注意如果父组件造成你的组件重新渲染，这个方法将会被调用即使属性没有改变。确保比较了当前和下一个值如果你只是想处理改变的情况。
React不会在[mounting](https://reactjs.org/docs/react-component.html#mounting)过程中的初始化属性的时候调用`UNSAFE_componentWillReceiveProps()`。它仅仅会被调用在如果一些组件的属性可能更新。调用`this.setState()`通常不会触发`UNSAFE_componentWillReceiveProps()`。

### shouldComponentUpdate()
```javascript
shouldComponentUpdate(nextProps, nextState)
```
使用`shouldComponentUpdate()`来让React知道，如果一个组件的输出结果不受当前改变的属性(prop)和状态(state)影响。默认的行为是每个状态(state)变化的时候重新渲染，在绝大多数情况下，你应该依靠默认行为。
`shouldComponentUpdate()`在接受到新属性和新状态发生渲染之前调用。默认返回`true`，这方法不在初始化渲染或者当`forceUpdate()`被调用的时候被调用。
返回`false`不会阻止子组件因为他们状态的改变导致的重新渲染。
当前，如果`shouldComponentUpdate()`返回`false`，然后`UNSAFE_componentWillUpdate()`、`render()`和`componentDidUpdate()`将不会被调用。注意的是在未来React可能把`shouldComponentUpdate()`为一个提示而不是一个详细的指令，然后返回`false`可能还是会导致组件的重新渲染。
如果你在分析后认定一个组件是慢的，你可能要改成继承自`React.PureComponent`，它用浅属性和状态比较来实现`sholdComponentUpdate()`。如果你确信要手动写它，你可以比较`this.props`和`nextProps`，`this.state`和`nextState`，然后返回`false`来告诉React这次更新可以跳过。
我们不推荐做深相等比较来检查，或者是在`shouldComponentUpdate()`使用`JSON.stringify()`。这是非常低效的并且会损害性能。

### UNSAFE_componentWillUpdate()
```javascript
UNSAFE_componentWillUpdate(nextProps, nextState)
```
`UNSAFE_componentWillUpdate()`就在收到新属性和状态渲染之前被调用，使用它作为一个在更新发生之前的时机去做一些准备。这个方法不被用于初始化渲染。
注意你们不可以在这里调用`this.setState()`，也不应该做任何在`UNSAFE_componentWillUpdate()`返回前会触发一次React组件更新的操作。例如：dispatch a Redux action。
如果你需要更新状态作为属性改变的响应，使用`UNSAFE_componentWillReceiveProps()`替代它。
> 注意如果`shouldComponentUpdate()`返回`false`，`UNSAFE_componentWillUpdate()`不会被调用。

### getSnapshotBeforeUpdate()
`getSnapshotBeforeUpdate()`会被正确地调用在最近的渲染输出到例如DOM之前。它使你的组件能够在他们可能被改变之前捕捉到当前的值(value)，例如：scroll position。任何由这个生命周期函数返回的值(value)会被当做一个参数传递给`componentDidUpdate()`。例如：
```javascript
class ScrollingList extends React.Component {
  listRef = React.createRef();

  getSnapshotBeforeUpdate(prevProps, prevState) {
    // Are we adding new items to the list?
    // Capture the current height of the list so we can adjust scroll later.
    if (prevProps.list.length < this.props.list.length) {
      return this.listRef.current.scrollHeight;
    }
    return null;
  }

  componentDidUpdate(prevProps, prevState, snapshot) {
    // If we have a snapshot value, we've just added new items.
    // Adjust scroll so these new items don't push the old ones out of view.
    if (snapshot !== null) {
      this.listRef.current.scrollTop +=
        this.listRef.current.scrollHeight - snapshot;
    }
  }

  render() {
    return (
      <div ref={this.listRef}>{/* ...contents... */}</div>
    );
  }
}
```
在上面的例子中，在`getSnapshotBeforeUpdate`而不是`componentWillUpdate`中读取`scrollHeight`属性十分重要，为了能支持异步渲染。用异步渲染，这里可能被延迟在"render"阶段的生命周期(`componentWillUpdate `和`render`)和`commit`阶段的生命周期(`getSnapshotBeforeUpdate`和`componentDidUpdate`)。如果用户在这期间做了一些像调整浏览器大小的事情，从`componentWillUpdate`中读取到的`scrollHeight`值就是过时的。

### componentDidUpdate()
```javascript
componentDidUpdate(prevProps, prevState, snapshot)
```
`componentDidUpdate()`在更新出现之后被立即调用。此方法不用于初始化渲染。使用它作为一个当组件更新完成后在DOM上操作的时机。这是一个做一些网络请求的好地方只要你比较当前的属性值和之前的属性值。例如如果属性没有改变，是不需要发送网络请求的。
如果你的组件执行了`getSnapshotBeforeUpdate()`生命周期，它返回的值(value)将会被作为第三个参数传入`componentDidUpdate()`，否则这个参数是`undefined`。
> 注意，当`shouldComponentUpdate()`方法返回`false`的时候`componentDidUpdate()`将不会被调用。

### componentWillUnmount()
```javascript
componentWillUnmount()
```
`componentWillUnmount()`在一个组件被卸载和销毁之前被立即调用。在这个函数中执行任何必要的清理，比如取消定时器，取消网络请求，清理任何在`componentDidMount()`方法中创建的订阅。

### componentDidCatch()
```javascript
componentDidCatch(error, info)
```
错误边界是一个在他们的子组件树中捕获任何JavaScript错误的React组件，log这些错误，展示一个后退UI而不是组件树崩溃。错误边界在渲染过程中捕获错误，在生命周期方法中，和在他们下面整棵树的构造函数中。
如果一个class组件定义了这个生命周期方法，它会成为一个错误边界。在其中调用`setState()`能让你捕获一个在子树中未经过处理的JavaScript错误然后展示一个回退UI。错误边界仅用于从意外的异常中恢复，不要尝试用他们来控制流。
更多细节，见[Error Handling in React 16](https://reactjs.org/blog/2017/07/26/error-handling-in-react-16.html)。
> Error boundaries only catch errors in the components below them in the tree. An error boundary can’t catch an error within itself.

[React生命周期在线参考](http://projects.wojtekmaj.pl/react-lifecycle-methods-diagram/)
