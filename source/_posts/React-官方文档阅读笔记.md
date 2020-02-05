---
title: React 官方文档阅读笔记
catalog: true
date: 2020-02-05 14:52:05
subtitle:
header-img:
tags: FE 
---

## ADVANCED GUIDES
### [Accessibility](https://reactjs.org/docs/accessibility.html)
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

### [Code-Splitting](https://reactjs.org/docs/code-splitting.html)
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

### [Context](https://reactjs.org/docs/context.html)
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
- React.creatContext : `const MyContext = React.createContext(defaultValue);`
默认值只会在未找到 `Provider` 的时候使用。
- Context.Provider : `<MyContext.Provider value={/* some value */}>`
当 `Provider` 修改了 value 值时，所有消费者子孙都会重新渲染。子组件的 `shouldComponentUpdate` 无法捕获到这种更新，因此即使父组件取消更新，子组件也会照常更新。
- Class.contentType : 在类外 `ClassName.contentType = MyContent`，在类中 `static contextType = MyContext;`
类中有 `contentType` 属性， context 赋值给这个属性（订阅），通过 `this.content` 取到值。
- Context.Consumer : `<MyContext.Consumer>{alue => /* render something based on the context value */}</MyContext.Consumer>`
- Context.displayName : `MyContext.displayName = 'MyDisplayName';`
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
