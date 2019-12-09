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
