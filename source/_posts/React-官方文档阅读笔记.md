---
title: React 官方文档阅读笔记
catalog: true
date: 2020-02-05 14:52:05
subtitle:
header-img:
tags: FE 
---

## ADVANCED GUIDES
### Accessibility
[link](https://reactjs.org/docs/accessibility.html)
1. 表单控件如 `<input/>` 、 `<textarea/>` 等，通常需要 `<label/>` 标签提高可访问性。在 react 中，属性 `for` 被替代为 `htmlFor` ：
```
<label htmlFor="namedInput">Name:</label>
<input id="namedInput" type="text" name="name"/>
```

2. 聚焦 input 组件的方法：ref 也可以在父组件创建，通过 props 转发给子组件。
```
class CustomTextInput extends React.Component {
  constructor(props) {
    super(props);
    // Create a ref to store the textInput DOM element
    this.textInput = React.createRef();
  }
  render() {
  // Use the `ref` callback to store a reference to the text input DOM
  // element in an instance field (for example, this.textInput).
    return (
      <input
        type="text"
        ref={this.textInput}
      />
    );
  }
}

// focus it elsewhere in our component when needed:
focus() {
  // Explicitly focus the text input using the raw DOM API
  // Note: we're accessing "current" to get the DOM node
  this.textInput.current.focus();
}
```
3. HTMLELement 上有 `onBlur` 及 `onFocus` 两个属性，可用于判断当前元素是否获得焦点。实际应用中，可以用于控制 popover 的显示：当离开某元素时， 隐藏 popover 。

### Code-Splitting
[link](https://reactjs.org/docs/code-splitting.html)
1. **Dynamic import**
```
// before
import { add } from './math';
console.log(add(16, 26));

// after
import("./math").then(math => {
  console.log(math.add(16, 26));
});
```
2. **React.lazy**
可以将动态导入变成一个组件，返回值必须包裹在 `<Suspense></Suspense>` 内部（可包裹多个 lazy compoments）：
```
// before
import OtherComponent from './OtherComponent';

// after
const OtherComponent = React.lazy(() => import('./OtherComponent'));

function MyComponent() {
  return (
    <div>
      <Suspense fallback={<div>Loading...</div>}>
        <OtherComponent />
      </Suspense>
    </div>
  );
}
```
3. **Error boundaries**
之后有专门的章节介绍这部分。
4. **Route-based code splitting**
基于路由的动态引入，与 React Router 配合使用：
```
import { BrowserRouter as Router, Route, Switch } from 'react-router-dom';
import React, { Suspense, lazy } from 'react';

const Home = lazy(() => import('./routes/Home'));
const About = lazy(() => import('./routes/About'));

const App = () => (
  <Router>
    <Suspense fallback={<div>Loading...</div>}>
      <Switch>
        <Route exact path="/" component={Home}/>
        <Route path="/about" component={About}/>
      </Switch>
    </Suspense>
  </Router>
);
```
5. **Named Exports**
若需要动态导入的模块不是默认导出，那我们可以创建多一个文件，将需要导入的模块转为默认导出：
```
// ManyComponents.js
export const MyComponent = /* ... */;
export const MyUnusedComponent = /* ... */;

// MyComponent.js
export { MyComponent as default } from "./ManyComponents.js";

// MyApp.js
import React, { lazy } from 'react';
const MyComponent = lazy(() => import("./MyComponent.js"));
```

### Context
[link](https://reactjs.org/docs/context.html)
上下文提供了一种通过组件树传递数据的方法，而不必逐层传递 props 。
1. 在类中使用方法如下：
```
// Context lets us pass a value deep into the component tree
// without explicitly threading it through every component.
// Create a context for the current theme (with "light" as the default).
const ThemeContext = React.createContext('light');

class App extends React.Component {
  render() {
    // Use a Provider to pass the current theme to the tree below.
    // Any component can read it, no matter how deep it is.
    // In this example, we're passing "dark" as the current value.
    return (
      <ThemeContext.Provider value="dark">
        <Toolbar />
      </ThemeContext.Provider>
    );
  }
}

// A component in the middle doesn't have to
// pass the theme down explicitly anymore.
function Toolbar(props) {
  return (
    <div>
      <ThemedButton />
    </div>
  );
}

class ThemedButton extends React.Component {
  // Assign a contextType to read the current theme context.
  // React will find the closest theme Provider above and use its value.
  // In this example, the current theme is "dark".
  static contextType = ThemeContext;
  render() {
    return <Button theme={this.context} />;
  }
}
```
2. **API**
(1) React.creatContext : `const MyContext = React.createContext(defaultValue);`
默认值只会在未找到 `Provider` 的时候使用。
(2) Context.Provider : `<MyContext.Provider value={/* some value */}>`
当 `Provider` 修改了 value 值时，所有消费者子孙都会重新渲染。子组件的 `shouldComponentUpdate` 无法捕获到这种更新，因此即使父组件取消更新，子组件也会照常更新。
(3) Class.contentType : 在类外 `ClassName.contentType = MyContent`，在类中 `static contextType = MyContext;`
类中有 `contentType` 属性， context 赋值给这个属性（订阅），通过 `this.content` 取到值。
(4) Context.Consumer : `<MyContext.Consumer>{alue => /* render something based on the context value */}</MyContext.Consumer>`
(5) Context.displayName : `MyContext.displayName = 'MyDisplayName';`
用于 React DevTools 。
3. **从嵌套组件更新 context**
初始值除了包含默认值，还可以包含一个 `onChange` 函数。
```
// Make sure the shape of the default value passed to
// createContext matches the shape that the consumers expect!
export const ThemeContext = React.createContext({
  theme: themes.dark,
  toggleTheme: () => {},
});
```

### Error Boundaries
[link](https://reactjs.org/docs/error-boundaries.html)
错误边界是 React 16 引入的新概念，可以在 render 过程、生命周期方法、以及组件树的构造中捕获**子组件**的错误，显示备用的 UI 而不是崩溃界面。注意，它无法捕获事件处理、异步代码、服务端渲染及它本身的错误。
将一个类组件转换为一个错误边界组件主要是实现两个方法：在 `static getDerivedStateFromError()` 中捕获子组件错误并触发备用 UI 的渲染；在 `componentDidCatch()` 中打印错误信息。
```
class ErrorBoundary extends React.Component {
  constructor(props) {
    super(props);
    this.state = { hasError: false };
  }

  static getDerivedStateFromError(error) {
    // Update state so the next render will show the fallback UI.
    return { hasError: true };
  }

  componentDidCatch(error, errorInfo) {
    // You can also log the error to an error reporting service
    logErrorToMyService(error, errorInfo);
  }

  render() {
    if (this.state.hasError) {
      // You can render any custom fallback UI
      return <h1>Something went wrong.</h1>;
    }

    return this.props.children; 
  }
}

// use it as a regular component
<ErrorBoundary>
  <MyWidget />
</ErrorBoundary>
```

### Forwarding Refs
[link](https://reactjs.org/docs/forwarding-refs.html)
React 提供了一个向下转发子组件 ref 的方法 `React.forwardRef`，用法如下：
```
const FancyButton = React.forwardRef((props, ref) => (
  <button ref={ref} className="FancyButton">
    {props.children}
  </button>
));

// You can now get a ref directly to the DOM button:
const ref = React.createRef();
<FancyButton ref={ref}>Click me!</FancyButton>;
```
`React.forwardRef` 接受两个参数，第一个参数是 props ，第二个参数是 ref ，作为子组件引用的赋值对象。如果是普通的类组件和函数组件，不会接受 ref 作为参数，也不能在 props 中访问到。
文档接下来还举了一个在高阶组件中转发 ref 的例子，目前还没完全看懂。🧐

### Fragments
[link](https://reactjs.org/docs/fragments.html)
当我们需要对一组子元素进行分组，又不希望添加额外的 DOM 节点时，可以使用 fragments ，有两种写法：`<></>` 和 `<React.Fragment></React.Fragment>` ，第二种写法可以添加 `key` 属性。
```
render() {
  return (
    <React.Fragment>
      <ChildA />
      <ChildB />
      <ChildC />
    </React.Fragment>
  );
}

// or
render() {
  return (
    <>
      <ChildA />
      <ChildB />
      <ChildC />
    </>
  );
}
```

### Higher-Order Components
定义：高阶组件是一个接受组件作为参数，返回新组件的函数。
```
const EnhancedComponent = higherOrderComponent(WrappedComponent);
```
既然是函数组件，那就应该是 pure fuction ，且不能有任何的副作用。它是通过将原始组件包裹在一个新的类组件，并返回这个类组件来实现功能的：
```
// This function takes a component...
function withSubscription(WrappedComponent, selectData) {
  // ...and returns another component...
  return class extends React.Component {
    constructor(props) {
      super(props);
      this.handleChange = this.handleChange.bind(this);
      this.state = {
        data: selectData(DataSource, props)
      };
    }

    componentDidMount() {
      // ... that takes care of the subscription...
      DataSource.addChangeListener(this.handleChange);
    }

    componentWillUnmount() {
      DataSource.removeChangeListener(this.handleChange);
    }

    handleChange() {
      this.setState({
        data: selectData(DataSource, this.props)
      });
    }

    render() {
      // ... and renders the wrapped component with the fresh data!
      // Notice that we pass through any additional props
      return <WrappedComponent data={this.state.data} {...this.props} />;
    }
  };
}
```
注意区分 HOCs 和 container components 。HOCs 使用了 容器组件作为其实现的一部分。
此外，注意不能在 render 函数中使用 HOCs ，否则会导致每次 render 时整个子组件熟4都会卸载再装载，丢失状态。
```
render() {
  // A new version of EnhancedComponent is created on every render
  // EnhancedComponent1 !== EnhancedComponent2
  const EnhancedComponent = enhance(MyComponent);
  // That causes the entire subtree to unmount/remount each time!
  return <EnhancedComponent />;
}
```