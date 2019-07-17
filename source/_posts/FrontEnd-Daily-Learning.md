---
title: FrontEnd Daily Learning
catalog: true
date: 2019-07-11 19:44:53
subtitle:
header-img:
tags: FE
---
#### 前言
recap and cheat sheet ，记录每天学到的知识/想法。
每日一问：今天你比昨天更博学了吗？


#### 2019/7/16
一、今日阅读：[How to read an often-changing value from useCallback?](https://reactjs.org/docs/hooks-faq.html#how-to-read-an-often-changing-value-from-usecallback)
直接上官网代码：
```
function Form() {
  const [text, updateText] = useState('');
  const textRef = useRef();

  useEffect(() => {
    textRef.current = text; // Write it to the ref
  });

  const handleSubmit = useCallback(() => {
    const currentText = textRef.current; // Read it from the ref
    alert(currentText);
  }, [textRef]); // Don't recreate handleSubmit like [text] would do

  return (
    <>
      <input value={text} onChange={e => updateText(e.target.value)} />
      <ExpensiveTree onSubmit={handleSubmit} />
    </>
  );
}
```
&emsp;&emsp; 也就是说，当 `text` 的值经常发生变化时，即使 `handleSubmit` 用 `useCallback` 包裹了，还是会重新声明。解决办法是传入一个 `ref` 对象代替原始值。也可以写一个 `custom hook` :
1）官网的版本：
```
function useEventCallback(fn, dependencies) {
  const ref = useRef(() => {
    throw new Error('Cannot call an event handler while rendering.');
  });

  useEffect(() => {
    ref.current = fn;
  }, [fn, ...dependencies]);

  return useCallback(() => {
    const fn = ref.current;
    return fn();
  }, [ref]);
}
```
实际使用：
```
function Form() {
  const [text, updateText] = useState('');
  // Will be memoized even if `text` changes:
  const handleSubmit = useEventCallback(() => {
    alert(text);
  }, [text]);

  return (
    <>
      <input value={text} onChange={e => updateText(e.target.value)} />
      <ExpensiveTree onSubmit={handleSubmit} />
    </>
  );
}
```
2）导师的版本：（他来了，他带着代码又来了）
```
/**
 * @param callback
 * @param oRefs
 */
export function useCallbackWithRefs<
  Refs,
  Callback extends (...args: any[]) => void
>(callback: (refs: Refs) => Callback, oRefs: Refs) {
  const refs = useRef(oRefs);
  useEffect(
    () => {
      refs.current = oRefs;
    },
    [oRefs]
  );

  return useCallback(
    (...args: any[]) => callback(refs.current)(...args),
    []
  ) as Callback;
}
```
实际使用：
```
function Form() {
  const [text, updateText] = useState('');
  
  // 原先的 callback 也可以有参数
  const handleSubmit = useCallbackWithRefs(
    refs => (params: any) => {
      // 要注意在函数内必须使用 `refs.xxx`，不能直接使用函数外部的任何变量 `xxx`
      console.log(refs.text);
  }, {text});

  return (
    <>
      <input value={text} onChange={e => updateText(e.target.value)} />
      <ExpensiveTree onSubmit={handleSubmit} />
    </>
  );
}
```
&emsp;&emsp;总的来说，官网的实现是将 `callback` 作为 `ref` 对象，并作为 `useCallback` 的依赖，不会频繁改变；如果在函数 `handleSubmit` 中要访问外部变量 `text` ，直接使用 `test` 即可。第二种实现方式是把经常变化的值作为 `ref` 对象，返回值 `useCallback` 第一个参数是 `refs => callback` ，依赖是空数组；这时如果要在函数 `handleSubmit` 中要访问外部变量 `text` ，必须使用 `refs.test` ，否则访问的只是 `test` 的初始值。
&emsp;&emsp;React 博大精深，接下来要好好研读一下这个 [FAQ](https://reactjs.org/docs/hooks-faq.html)。
二、今日网址
&emsp;&emsp;一个 [emoji copy](https://www.emojicopy.com/) 网站，我们 👧 就是喜欢这些花里胡哨的东西。


#### 2019/7/15
&emsp;&emsp;今天在项目中使用 `React Hooks` 又踩坑了，看来自己对这部分还是没有理解透彻。在使用 `useCallback` 和 `useEffect` 时，要注意第二个参数，也就是传入的 `[deps]`。如果使用 `useCallback(fn,[deps])` ， `[deps]` 应该包含函数 `fn` 所涉及的所有变量；如果使用 `useEffect(fn,[deps])` ， 当 `deps` 的值变化时，就会执行 `fn`，因此`[deps]` 不一定要包含函数 `fn` 所涉及的所有变量，而是应该传入会引起该函数执行的那些参数。
&emsp;&emsp;今日踩坑记录：为了优化子组件，作为 `props` 的函数都使用 `useCallback` 包裹了，并传入了空数组作为第二个参数，表示没有依赖。但是函数中的运算需要用到组件中一个变量，如果没有将该变量作为 `deps` ，这个变量就会一直保持初始值，值并不会改变，运行结果就会与预期不符。（真的太蠢了，缓缓躺倒）


#### 2019/7/14
周末当然是约会啦。😍


#### 2019/7/13
一、今日阅读：[How to compare oldValues and newValues on React Hooks useEffect?](https://stackoverflow.com/questions/53446020/how-to-compare-oldvalues-and-newvalues-on-react-hooks-useeffect)
&emsp;&emsp; React class 组件提供了 `ComponentDidUpdate` 之类的方法来获取到当前 `props` 和前一个 `props` ，并进行比较，决定是否进行更新。函数式组件只有 `useEffect` 函数来模仿生命周期函数，当我们需要获取组件先前的 `props` 时，可以使用下面的 `custom hook` ：
```
function usePrevious(value) {
  const ref = useRef();
  useEffect(() => {
    ref.current = value;
  });
  return ref.current;
}
```
&emsp;&emsp;之后在 `useEffect` 中使用上面的函数来模拟 `ComponentDidUpdate` ：
```
const Component = (props) => {
    const {receiveAmount, sendAmount } = props
    const prevAmount = usePrevious({receiveAmount, sendAmount});
    useEffect(() => {
        if(prevAmount.receiveAmount !== receiveAmount) {

         // process here
        }
        if(prevAmount.sendAmount !== sendAmount) {

         // process here
        }
    }, [receiveAmount, sendAmount])
}
```
&emsp;&emsp;有时候在 debug 时，我们想知道组件为什么会重新渲染，是那些 `props` 更新了，也可以使用上面的方法来获取 `prevProps`，并在函数组件最开始时写一个 `useEffect` 将参数都打印出来，使用 `===` 比较。 



#### 2019/7/12
一、今日技能
&emsp;&emsp;项目中经常会涉及到 JSON 字符串的解析，解析出错就扑街了，一般是用 `try {...} catch {...}` 包裹。今天导师在 code review 时建议我使用如下函数，该函数也可以覆盖空字符串的检测：
```
function safeJsonParse<T>(str): { ok: true, value: T } | { ok: false } {
  try {
    return {
      ok: true,
      value: JSON.parse(str)
    }
  } catch {
    return {
      ok: false
    }
  }
}
```
二、今日阅读：[从浏览器多进程到JS单线程，JS运行机制最全面的一次梳理](https://imweb.io/topic/5b72d4ef15554e6d3409f817)。
&emsp;&emsp;这一篇很好地梳理了进程、线程、浏览器**多进程**、浏览器内核**多线程**、JS单线程、JS运行机制的相关知识，很连贯，建议时不时回顾。简单记录几个知识点：
1）进程是CPU资源分配的最小单位，线程是CPU调度的最小单位。（想当年考操作系统的时候还背得滚瓜烂熟）
2）**浏览器是多进程的**，包括的主要进程有：
- Browser进程（浏览器的主进程，只有一个）
- 浏览器渲染进程（浏览器内核，Renderer进程，渲染进程，内部是多线程的）：默认每个Tab一个进程
- 第三方插件进程
- GPU进程：3D绘制

3）重点是浏览器内核，它是多线程的，主要常驻线程有：
- GUI渲染线程：渲染页面，解析HTML，CSS，构建DOM树和RenderObject树，布局和绘制，`repaint` 和 `reflow` 等。
- JS引擎线程：处理任务队列中的任务，与 GUI 渲染线程是互斥的。
- 事件触发线程：控制事件循环，把事件添加到任务队列的末尾。
- 定时触发线程：`setTimeout` 和 `setInterval`，同样也是计时完毕后添加到队列末尾。
- 异步 http 请求线程

4）Browser进程和浏览器内核之间是需要通信的
5）时间循环机制 `Event Loop` ：JS分为同步任务和异步任务，同步任务都会在主线程上运行，形成一个执行栈；主线程之外，由事件触发线程管理一个**任务队列/事件队列**，异步任务的运行结果会被添加到任务队列中。一旦执行栈中所有同步任务执行完毕，系统就会读取任务队列，将可运行的异步任务添加到执行栈中开始执行。 
6）进阶：`macrotask` / `task` 和 `microtask`/ `job`。ES6中的 `Promise` 就属于 `microtask`微任务 ，而主代码块，事件队列中的时间如 `setTimeout` 和 `setInterval` 就属于 `macrotask` 宏任务。总结的运行机制就是：
- 执行宏任务。（从执行栈中获取，如果没有就从事件队列中获取）
- 执行过程中如果遇到微任务，就添加到微任务队列 `Job Queues` 中。（作者猜测这个队列由JS引擎线程维护，因为是在主线程下无缝执行的）
- 宏任务执行完毕后，立即依次执行当前微任务队列中所有微任务。
- 检查渲染，由 GUI 线程接管。
- 渲染完毕后，由 JS 引擎线程接管，从时间队列中获取并执行下一个宏任务。

7）在第六点中提到的是一个进阶的概念，对应一道题目，如果代码中依次有 `setTimeout` 和 `Promise` ，是会先打印出 `Promise` 的执行结果的。


#### 2019/7/11
&emsp;&emsp; React 16.8 提出了 `hook` 的概念，函数式组件也可以拥有自己的状态。现在的工作项目已经摒弃了 `class` ，改用函数式组件，正在慢慢摸索中。今天在腾讯[IMWeb前端博客](http://imweb.io/)中看到了两篇介绍 `React hook` 的文章，受益匪浅，简单记录一下。
 一、[react hook——你可能不是“我”所认识的useEffect](https://imweb.io/topic/5cd845cadcd62f86299fcd76)
&emsp;&emsp;这篇介绍了 `useEffect` 这个API，用它模拟了class组件的生命周期函数。`useEffect` 用于执行副作用，相当于 `ComponentDidMount` 和 `ComponentDidUpdate`。该API有两个参数和一个返回值。第一个参数是一个副作用函数，返回值是清除函数，相当于 `ComponentWillUnmount`，每一次 `render` 都会执行副作用和清除上一次副作用。**第二个参数是一个数组，传入的是副作用函数所需要的依赖，当任一依赖更新时，会重新生成一个新的副作用并执行；如果传入一个空数组，没有依赖，只会执行一次，相当于 `ComponentDidMount`；如果不传，就是没有说明自己有没有依赖（注意是不知道有没有，不是没有！），每次 `render` 时就执行，相当于 `ComponentDidUpdate` 。**
&emsp;&emsp; 最后还讲了 `useEffect` 和 `useLayoutEffect` 的区别，简单来说前者是异步的，后者是同步的。还没好好深入这部分，TODO。
二、[可能你的react函数组件从来没有优化过](https://imweb.io/topic/5d1e3657f7b5692b080f2651)
&emsp;&emsp;优化问题真是我一个痛点。这篇文章很清楚地解释了 `Hooks` 一些可用于组件优化的API。强推！
&emsp;&emsp; 特别地，文章介绍了当函数组件中传入的 `props` 值为函数时，由于每一次执行或重新执行，作用域里面一切都是重新开始，函数不是简单数据类型，不能画上等号，子组件都会重新渲染。针对这个问题文章提出了几种解决办法：
1） 作为 `props` 的函数在函数组件外定义，函数组件用 `React.memo()` 包裹。
 &emsp;&emsp; `React.memo()` 类似于 `PureComponent` 和 `ComponentDidUpdate` ，如果函数组件的 `props` 值都一样，就会跳过该组件的执行，减少不必要的渲染，实现性能优化。
2） 作为 `props` 的函数在函数组件内定义，使用`useCallBack` 或 `useMemo`包裹，函数组件用 `React.memo()` 包裹。
&emsp;&emsp; `useCallback(() => {}, [deps])` 返回一个函数，当 `deps` 不变时（如传入空数组，表示没有依赖），都是同一个函数。`const a = useMemo(() => memorizeValue, [deps])`，当 `deps` 不变时，`a` 的值还是上次的 `memorizeValue`，省去了重新计算的过程。
&emsp;&emsp; 注意当 `memorizeValue` 是一个函数时，`useCallback(fn, inputs) <=> useMemo(() => fn, inputs)`。


#### 2019/7/10
&emsp;&emsp;今天在项目中接触到了 `symbol`，鉴于之前一直没有注意这个数据类型，在今天补上。 `symbol` 是 ES6 新增的**基本**数据类型。它的使用如下：
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


#### 2019/7/9
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