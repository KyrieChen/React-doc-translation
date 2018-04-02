## Forwarding Refs
Ref传递是一种将ref通过组件传递给其后代的技术。这技术在高阶组件（HOC）中十分有用。让我们以一个HOC的例子开始，这个HOC是一个logs组件用来打印出props。
```javascript
function logProps(WrappedComponent) {
  class LogProps extends React.Component {
    componentDidUpdate(prevProps) {
      console.log('old props:', prevProps);
      console.log('new props:', this.props);
    }

    render() {
      return <WrappedComponent {...this.props} />;
    }
  }

  return LogProps;
}
```
这个`logProps`高阶组件传递所有的`props`给`WrappedComponent`，所以渲染出来的东西应该一样的。例如，我们可以用这个`logProps`高阶组件去打印出传递给`FancyButton`组件的所有`props`。
```javascript
class FancyButton extends React.Component {
  focus() {
    // ...
  }

  // ...
}

// Rather than exporting FancyButton, we export LogProps.
// It will render a FancyButton though.
export default logProps(FancyButton);
```
上面的例子有个需要提醒的问题：`refs`将不会给传递下去。因为`ref`不是属性，React用了不同的方式处理它，像`keys`一样。如果你给高阶组件添加了`ref`，这个`ref`将引用最外层的容器组件，不是包裹组件。这意味着，用来标记`FancyButton`的`ref`将会标记到`logProps`组件。
```javascript
import FancyButton from './FancyButton';

const ref = React.createRef();

// The FancyButton component we imported is the LogProps HOC.
// Even though the rendered output will be the same,
// Our ref will point to LogProps instead of the inner FancyButton component!
// This means we can't call e.g. ref.current.focus()
<FancyButton
  label="Click Me"
  handleClick={handleClick}
  ref={ref}
/>;
```
幸运得是，我们可以通过使用`React.forwardRef`API来明确地传递refs到内部的`FancyButton`组件。`React.forwardRef`接受一个render函数，这个函数接受`props`和`ref`作为参数，返回一个React节点。例如：
```javascript
function logProps(Component) {
  class LogProps extends React.Component {
    componentDidUpdate(prevProps) {
      console.log('old props:', prevProps);
      console.log('new props:', this.props);
    }

    render() {
      const {forwardedRef, ...rest} = this.props;

      // Assign the custom prop "forwardedRef" as a ref
      // 指派自定义的属性"forwardedRef"作为一个ref
      return <Component ref={forwardedRef} {...rest} />;
    }
  }

  /* Note the second param "ref" provided by React.forwardRef.
  注意第二个参数"ref"提供自React.forwardRef
  We can pass it along to LogProps as a regular prop, e.g. "forwardedRef"
  我们可以将ref当做一个普通的属性往下传，比如这里的"forwardedRef"
  And it can then be attached to the Component
  然后这就能够触及相应的组件了 */
  
  function forwardRef(props, ref) {
    return <LogProps {...props} forwardedRef={ref} />;
  }

  // These next lines are not necessary,
  // But they do give the component a better display name in DevTools,
  // e.g. "ForwardRef(logProps(MyComponent))"
  const name = Component.displayName || Component.name;
  forwardRef.displayName = `logProps(${name})`;

  return React.forwardRef(forwardRef);
}
```