---
title: FrontEnd Daily Learning
catalog: true
date: 2019-07-11 19:44:53
subtitle:
header-img:
tags: FE
---
##### 前言
recap and cheat sheet ，记录每天学到的小东西/想法。

##### 2019/7/11
React 16.8 提出了 `hook` 的概念，函数式组件也可以拥有自己的状态。现在的工作项目已经摒弃了 `class` ，改用函数式组件，正在慢慢摸索中。
今天在腾讯[IMWeb前端博客](http://imweb.io/)中看到了两篇介绍 `React hook` 的文章，受益匪浅，简单记录一下。
1）[react hook——你可能不是“我”所认识的useEffect](https://imweb.io/topic/5cd845cadcd62f86299fcd76)
这篇介绍了 `useEffect` 这个API，用它模拟了class组件的生命周期函数。`useEffect` 用于执行副作用，相当于 `ComponentDidMount` 和 `ComponentDidUpdate`。该API有两个参数和一个返回值。第一个参数是一个副作用函数，返回值是清除函数，相当于 `ComponentWillUnmount`，每一次 `render` 都会执行副作用和清除上一次副作用。**第二个参数是一个数组，传入的是副作用函数所需要的依赖，当任一依赖更新时，会重新生成一个新的副作用并执行；如果传入一个空数组，没有依赖，只会执行一次，相当于 `didMount`；如果不传，就是没有说明自己有没有依赖（注意是不知道有没有，不是没有！），每次 `render` 时就执行，相当于 `ComponentDidUpdate` 。**
最后还讲了 `useEffect` 和 `useLayoutEffect` 的区别，简单来说前者是异步的，后者是同步的。还没好好深入这部分，TODO。

2）[可能你的react函数组件从来没有优化过](https://imweb.io/topic/5d1e3657f7b5692b080f2651)
优化问题真是我一个痛点。这篇文章很清楚地解释了 `Hooks` 一些可用于组件优化的API。强推！
特别地，文章介绍了当函数组件中传入的 `props` 值为函数时，由于每一次执行或重新执行，作用域里面一切都是重新开始，函数不是简单数据类型，不能画上等号，子组件都会重新渲染。针对这个问题文章提出了几种解决办法：
1） 作为 `props` 的函数在函数组件外定义，函数组件用 `React.memo()` 包裹。
 `React.memo()` 类似于 `PureComponent` 和 `ComponentDidUpdate` ，如果函数组件的 `props` 值都一样，就会跳过该组件的执行，减少不必要的渲染，实现性能优化。
2） 作为 `props` 的函数在函数组件内定义，使用`useCallBack` 或 `useMemo`包裹，函数组件用 `React.memo()` 包裹。
`useCallback(() => {}, [deps])` 返回一个函数，当 `deps` 不变时（如传入空数组，表示没有依赖），都是同一个函数。`const a = useMemo(() => memorizeValue, [deps])`，当 `deps` 不变时，`a` 的值还是上次的 `memorizeValue`，省去了重新计算的过程。
注意当 `memorizeValue` 是一个函数时，`useCallback(fn, inputs) <=> useMemo(() => fn, inputs)`。


##### 2019/7/10
今天在项目中接触到了 `symbol`，鉴于之前一直没有注意这个数据类型，在今天补上。 `symbol` 是 ES6 新增的**基本**数据类型。它的使用如下：
```
const s1 = Symbol();
const s2 = Symbol();
console.log(s1 === s2); // false

const s3 = new Symbol() // TypeError: Symbol is not a constructor
Symbol("foo") === Symbol("foo"); // false
```
`Symbol()` 返回的每个 `symbol` 值都是唯一的，可以接受一个字符串作为参数。它最常被用于对象属性的标识符，如：
```
const obj = {}
const foo = Symbol("foo")
obj[foo] = "foo"
obj.bar = "bar"

console.log(obj); // { bar: "bar" , Symbol(foo): "foo"}
console.log(foo in obj); // true
console.log(obj[foo]); // foo
console.log(Object.keys(obj)); // ["bar"]
console.log(Object.getOwnPropertySymbols(obj)) // [Symbol(foo)]
```
也就是说， `Object.key()` 不会返回 `symbol` 值，同理，`Object.getOwnPropertyNames()`、`for..in`、`for...of` 也不会返回。`JSON.stringify()` 也会忽略：
```
JSON.stringify({[Symbol('foo')]: 'foo'});                 
// '{}'
```
`Symbol` 还有两个方法。 `Symbol.for(key)` 是根据指定的 `key` 搜索现有的 `symbol` 并返回, 如果找不到，会使用 `key` 在全局的 `symbol` 注册表中创建一个新的 `symbol` 。`Symbol.keyFor(sym)` 是在全局注册表中检索，返回共享的 `symbol key` 。


##### 2019/7/9
一个不好的编程习惯：让函数在内部获取自己所需的依赖。例如：
```
// 一个可复用函数
function doSomething(key: string) {
    return (
        <Button
            loading = {key==="str1"? Compoment1 : Compoment2}
        >
            button
        </Button>
    )
}
```
`lodaing` 所需的组件应该是作为函数参数传入的，不然如果我们新增了一个 `key`，很容易忽略了该地方的修改。这个观点类似于依赖反转。