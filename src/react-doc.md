## 何时 `setState` 是异步的？
目前，`setState` 在 event handlers 里是异步的。
这将确保在父组件和子组件在click事件中都调用 `setState` 的时候，子组件不会 re-render 两次。相反，React 在浏览器事件结束时“冲掉”状态的更新。结果就是在大型应用里性能显著的提升了。

## React为什么不同步的更新 `this.state`？ 
如上个问题所述，React 在开始 re-render 之前，故意“等到”所有组件在他们的事件处理函数中调用`setState()`。这样子就可以通过避免不必要的 re-render 来提高性能。
然而，你可能还是觉得疑惑，为什么 react 不立马更新 `this.state` 而不 re-render。这里有两个主要的原因：

- 这样会打破 `props` 和 `state` 的一致性，造成难以调试的问题
- 这会让我们正在努力实现的一些新功能无法实现

这篇 [GitHub comment](https://github.com/facebook/react/issues/11527#issuecomment-360199710) 深入了一些例子来说明。