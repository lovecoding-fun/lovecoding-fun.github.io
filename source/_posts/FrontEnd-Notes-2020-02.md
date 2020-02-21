---
title: FrontEnd Notes [2020.02]
catalog: true
date: 2020-02-21 10:29:52
subtitle:
header-img:
tags:
---

#### About

📅 2020 年 2 月的零散学习记录。

动笔时发现已经是2月21号了，惭愧啊惭愧。

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