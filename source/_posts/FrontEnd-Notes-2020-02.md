---
title: FrontEnd Notes [2020.02]
catalog: true
date: 2020-02-21 10:29:52
subtitle:
header-img:
tags: FE
---

#### About

📅 2020 年 2 月的零散学习记录。

动笔时发现已经是2月21号了，惭愧啊惭愧。

#### 2020/02/27
今天在处理函数式组件时遇到了一个问题，将某个函数作为回调函数传到子组件，在回调函数中打印 state ，只能取到初始值，没有打印出最新结果。排查了很久，后来发现在子组件中，该函数用于注册事件监听函数，因此事件监听绑定的函数，是第一次渲染时传进去的，后续并不会更新，永远只能获取到初始值。
不考虑父子组件，用最简单的例子复现这个问题：
```
const App:React.FC = () => {
  const [state,setState] = useState(0);

  useEffect(() => {
    window.addEventListener("wheel", handleWheelChange);
    return () => {
      window.removeEventListener("wheel", handleWheelChange);
    }
  },[]);

  const handleWheelChange = () => {
    // always print 0
    console.log(state)
  };
  
  return <></>
} 
```
如果要解决这个问题，可以使用 ref ，取值时通过 `ref.current` 取：
```
const App:React.FC = () => {
  const [state,setState] = useState(0);
  // create ref
  const stateRef = useRef(state);

  // update ref.current when state changes
  useEffect(() => {
    ref.current = state;
  },[state])

  useEffect(() => {
    window.addEventListener("wheel", handleWheelChange);
    return () => {
      window.removeEventListener("wheel", handleWheelChange);
    }
  },[]);

  const handleWheelChange = () => {
    // access newest state correctly
    console.log(ref.current)
  };
  
  return <></>
} 
```
上述代码只是一种思路的表达，在实际项目中，直接使用 `useRef` 初始化变量就好了，没有必要先 `useState` ，再对 state `useRef` 。但有时使用了 `useReducer` ，如果只需要用到其中某个变量，就可以用上述代码中提到的方法。
第二种方法是通过调用 `setState` ，传入函数来取到最新的 state ，如：
```
...
const handleWheelChange = () => {
    setState(prev => {
      // prev is the newest state, do anything you need
      // remenber to return value
      return prev;
    })
  };
...
```

#### 2020/02/24
今日阅读：[How to use throttle or debounce with React Hook?](https://stackoverflow.com/questions/54666401/how-to-use-throttle-or-debounce-with-react-hook)
在使用函数式组件时，如果我们需要引入 lodash 的 `throttle` 或 `debounce` ，很容易写出如下错误代码：
```
const ExComp:React.FC = () => {
  ....
  // call it somewhere
  const throttle = lodash.throttle((value) => {
    // do something
  },2000)
  ...
  return <></>
}
```
上述代码中，在函数式组件中引入了一个 throttle 函数（底层实现是 `setTimeout`），然而对于函数式组件而言，每一次 rerender 都会重新初始化变量，所以 throttle 并不会生效。解决这个问题可以使用 `useRef` 或是 `useCallback` 。如：
```
const App = () => {
  const [value, setValue] = useState(0)；

  const throttled = useRef(
    throttle((newValue) => console.log(newValue), 1000)
  )
  useEffect(() => throttled.current(value), [value])

  // or
  // const throttled = useCallback(
  //  throttle((newValue) => console.log(newValue), 1000),
  //  []
  // );
  // useEffect(() => throttled(value), [value])

  return (
    <button onClick={() => setValue(value + 1)}>{value}</button>
  )
```
调用时直接使用 `throttled.current()` （或 `throttled()` ）即可。注意如果在 throttle 函数中需要用到函数外的 state 值，只能取到初始值（闭包），这种时候只能将所需参数作为 params 传进去。或是将 throttle 函数移到组件外。

#### 2020/02/21
在使用 react hooks 时，如果需要在组件中实例一个类（如 `new stroe()`）；为了让该变量变成一个 state ，而不是每次重新渲染时都会重新初始化，我们会使用 `useState` ：
```
import { useState } from "react"

const XXX:React.FC = () => {
  const [store] = useState(new Store());

  return <></>;
}
```
引入惰性初始 state ，传入 function ，关于惰性初始 state 可以参考文档 [Lazy initial state](https://reactjs.org/docs/hooks-reference.html#lazy-initial-state) 部分，更详细的解释可以参考 StackOverflow 上这个 [回答](https://stackoverflow.com/a/58539958) ，具体而言，传入 function 时只会在第一次渲染时调用；如果不引入惰性初始 state ，虽然 initialState 只会在第一次渲染时使用，但传入 `useState` 的函数还是会进行初始化和调用。
```
import { useState } from "react"

const XXX:React.FC = () => {
  const [store] = useState(() => new Store());

  return <></>;
}
```
这样带来的问题是，如果 store 中的成员更新了，并不会触发重新渲染。为了解决这个问题，我们可以直接引入 Mobx ，参考 mobx-react-lite 中 [Lazy initialization](https://github.com/mobxjs/mobx-react-lite#lazy-initialization) 这一节：
```
import { observer } from "mobx-react-lite"
import { observable } from "mobx"
import { useState } from "react"

const XXX:React.FC = observer(() => {
  const [store] = useState(() => observable(new Store()));

  return <></>;
})
```