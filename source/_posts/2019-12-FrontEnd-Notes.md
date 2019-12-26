---
title: "FrontEnd Notes [2019.12]"
catalog: true
date: 2019-12-09 23:33:36
subtitle:
header-img:
tags: FE
---

#### About

📅 2019 年 12 月的零散学习记录。

#### 2019/12/26
今日阅读：[When to useMemo and useCallback](https://kentcdodds.com/blog/usememo-and-usecallback)
起因是刷 reddit 时看到了这篇 [帖子](https://www.reddit.com/r/reactjs/comments/efjgfc/should_i_use_usecallback_in_every_function/)，提问者的问题是 "Should I use useCallback in every function declared inside a functional component?"，评论中有人贴出了这个链接，更正了我对性能优化的一些错误认知。
作者在文章中举了一个例子，在某个函数组件中有这样一个函数：
```
const dispense = candy => {
  setCandies(allCandies => allCandies.filter(c => c !== candy))
}
```
问题：如果用 `useCallback` 包裹，那么性能会不会更好呢？
```
const dispense = React.useCallback(candy => {
  setCandies(allCandies => allCandies.filter(c => c !== candy))
}, [])
```
我刚开始的答案是使用 `useCallback` 会更好，毫无疑问是错的。上面的函数也可被写成：
```
const dispense = candy => {
    setCandies(allCandies => allCandies.filter(c => c !== candy))
  }
+ const dispenseCallback = React.useCallback(dispense, [])
```
diff 之后会发现只是多了一行，该有的声明还是有，还需要做更多的工作。现在不仅仅要定义函数，还要定义依赖数组，调用 `useCallback` 进行赋值分配等工作。之前我一直误以为，使用 `useCallback` 后，函数只会在第一次渲染时定义并赋值，第二次渲染不会做额外的操作。其实不是的，这里的相等可以理解为是 “指向的位置” 相等，类似 `ref` 和 `ref.current`，重新渲染时就相当于执行 `ref.current = newVal` 。
那什么时候需要用到 `useCallback` 呢？官方文档是这样写的：
> This is useful when passing callbacks to optimized child components that rely on reference equality to prevent unnecessary renders (e.g. shouldComponentUpdate).

最简单粗暴的经验，是当你需要将这个函数传递给子组件时，可以使用 `useCallback` 来避免子组件不必要的渲染（子组件使用 `useMemo`）；更准确的说法，是当你需要依赖引用相等性时，就应使用 `useCallback` 。  

#### 2019/12/17
保留四位有效数字：
```
x.toPrecise(4) // return string
```

#### 2019/12/9

使用原生方法实现文件点击/拖拽上传，并限制只能上传单个 JSON 文件：
主要思想是监听 `onClick` 和 `onDrop` 事件，注意在 React 中，如果我们需要注册捕获阶段的事件处理函数，则应为事件名添加 `Capture`。例如，使用处理捕获阶段的点击事件 `onClickCapture`，而不是 `onClick`。详情可参考官方文档 [Supported Events](https://reactjs.org/docs/events.html#supported-events) 。

```
// upload.tsx
  ...
  const handleUploadClick = () => {
    const input = document.createElement("input");
    input.type = "file";
    input.accept = ".json,application/json";
    input.onchange = ev => {
      // double-checked
      if (
        input.files &&
        input.files.length === 1 &&
        input.files[0].type === "application/json"
      ) {
        // deal with input.files
      }
    };
    input.click();
  };

  const handleUploadDrop = (ev:       React.DragEvent<HTMLDivElement>) => {
    if (ev.dataTransfer.files.length > 1) {
      console.error("只允许上传单个文件");
    } else if (ev.dataTransfer.files[0].type !== "application/json") {
      console.error("只允许上传json文件");
    } else {
      // deal with ev.dataTransfer.files[0]
    }
  };

  ...

  render() {
    return (
      <div
        onClickCapture={ev => {
          ev.preventDefault();
          ev.stopPropagation();
          handleUploadClick();
        }}
        onDropCapture={ev => {
          ev.preventDefault();
          ev.stopPropagation();
          handleUploadDrop(ev);
        }}
      >
        Click or drag file to this area to upload
      </div>
    )
  }

```
