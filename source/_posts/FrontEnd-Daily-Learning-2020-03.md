---
title: 'FrontEnd Daily Learning [2020.03]'
catalog: true
date: 2020-03-03 16:48:47
subtitle:
header-img:
tags: FE
---
#### About

📅 2020 年 3 月的零散学习记录。

🤦‍♀️ 新的一月，新的打气。希望自己这个月能多记笔记。

#### 2020/03/04
BroadcastChannel（[mdn](https://developer.mozilla.org/en-US/docs/Web/API/BroadcastChannel)）提供了在同源的不同的 windows，tabs，frames，iframes 之间通信的方法。通过触发一个 message 事件，消息可以广播到所有监听了该频道的 BroadcastChannel 对象。
实际应用中，我们可以用于检测用户是否打开了多个窗口。在 window 中注册一个 BroadcastChannel 对象，监听 `load` 和 `hashchange` 事件，广播当前页面的 url ，并监听 message 事件。这样当打开第二个页面时，第一个页面就会接收到其发送的消息，提示用户执行相关操作。代码如下：
```
if (
  process.env.NODE_ENV === "production" &&
  typeof BroadcastChannel !== "undefined"
) {
  function warn() {
    Modal.warn({
      title: "Warning!",
      content: <p>"该页面已在另一窗口打开，同时打开两个窗口可能导致数据无法提交。"</p>,
      width: 600
    });
  }
  const bc = new BroadcastChannel("page");
  bc.onmessage = ev => {
    if (ev.data.url === window.location.href) {
      warn();
    }
  };
  window.addEventListener("load", () => {
    bc.postMessage({
      url: window.location.href
    });
  });

  window.addEventListener("hashchange", function(event) {
    bc.postMessage({
      url: window.location.href
    });
  });
}
```

#### 2020/03/03
如果我们希望升级应用时，前端页面也可以感知到并收到提示，可以每隔一定时间请求 currentScript（返回当前正在执行的 `<script>` 元素，[mdn](https://developer.mozilla.org/en-US/docs/Web/API/Document/currentScript)），由于升级时文件的 hash 会发生变动，当返回 404 时，即可提示用户刷新页面。
注意：请求时加上时间戳，保证了不会取到浏览器的 cache 。
```
const selfUrl = (document.currentScript as HTMLScriptElement).src;
const it = setInterval(() => {
  axios[process.env.NODE_ENV === "development" ? "get" : "head"](selfUrl, {
    params: { t: Date.now() }
  }).catch(err => {
    if (err.response && err.response.status === 404) {
      clearInterval(it);
      Modal.confirm({
        title: "系统升级，是否刷新页面？",
        onOk() {
          window.location.reload();
        }
      });
    }
  });
}, 60000);
```
关于 `document.currentScript`，注意如果当前正在执行的代码是处在某个回调函数或者事件处理函数中的，那么 currentScript 属性不会指向包含那个函数的 `<script>` 元素,而是会返回 `null` 。
