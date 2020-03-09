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

#### 2020/03/09
[Yarn vs NPM](https://www.keycdn.com/blog/npm-vs-yarn)
npm 和 yarn 都是包管理工具，yarn 是 Facebook 研发的，意在解决 npm 一致性、安全性和速度方面的一些问题。两者只是安装的手段不同，内部依赖的 npm structure 是相同的。yarn 相比 npm 的优势在于：
1. yarn.lock 文件可以保证每个设备上安装的包都是相同的
2. npm 只能通过序列化的方式一个接一个地安装包，yarn 可以同时执行多个安装步骤，因此速度更快
3. npm 会自动从依赖项运行代码并允许动态添加软件包，yarn 只能从 yarn.lock 或 package.json 文件安装，因此更安全；且其在安装前使用校验和，以确保每个包的完整性

#### 2020/03/07
一、TDZ: [What is the temporal dead zone?](https://stackoverflow.com/questions/33198849/what-is-the-temporal-dead-zone)
ES6 新增了 `const` 和 `let` 两个关键字。他们与 `var` 一样，声明都会被提升（hoisted）。但对于 `const` 和 `let` 而言，存在一个“暂时性死区”的概念：如果在声明之前访问一个 `var` 变量，会返回 `undefined` ，但访问 `let` 或 `const` 变量会返回 `ReferenceError` ：
```
console.log(aVar); // undefined
console.log(aLet); // causes ReferenceError: aLet is not defined
var aVar = 1;
let aLet = 2;
```
只有声明了变量（不是赋值），TDZ 才会结束：
```
//console.log(aLet)  // would throw ReferenceError

let aLet;
console.log(aLet); // undefined
aLet = 10;
console.log(aLet); // 10
```
从上面的例子很容易陷入 “let 声明不会被提升” 的误区，实际上通过下面这个例子就可以证明声明会被提升：
```
let x = 'outer value';
(function() {
  // start TDZ for x
  console.log(x);
  let x = 'inner value'; // declaration ends TDZ for x
}());
```
上面例子输出 `ReferenceError`，如果声明没有提升，会输出 "outer value" 。

二、[What is Waiting (TTFB) in DevTools, and what to do about it](https://scaleyourcode.com/blog/article/27)

#### 2020/03/06
考虑如下树形数据结构：
```
// interface TreeData
{
  value: "",
  children: [
    {
      value: "",
      children: [
        {value:"",children:[{value:""}]},
        {value:""}
      ]
    }
  ]
}
```
当我们有一组 children 标签和一个节点的 value 值，想要得到该节点在树形结构中的详细信息，可以使用递归遍历：
```
export function getLabelDetail(value: string, labels: TreeData[]) {
  const checkIfHitNode = (value: string, node: TreeData) => {
    if (node.value === value) {
      return node;
    } else if (node.children) {
      for (let i = 0; i < node.children.length; i++) {
        const result = checkIfHitNode(value, node.children[i]);
        if (result) {
          return result;
        }
      }
    } else {
      return null;
    }
  };

  for (let i = 0; i < labels.length; i++) {
    const hit = checkIfHitNode(value, labels[i]);
    if (hit) {
      return hit;
    }
  }
}
```
注意 `checkIfHitNode()` 函数的输入值是 value 以及一个根节点，我们从根节点出发，依次检查 `children` 中是否包含要寻找的节点。

#### 2020/03/05
`URL()` 函数用于构造 URL，参考 [mdn](https://developer.mozilla.org/en-US/docs/Web/API/URL)，内置多种静态属性，常用属性的输出值如下：
示例：`https://domain.cc:80/article?page=1`

|properties|meaning|output|
|---|---|---|
|href|完整 URL|`https://domain.cc:80/article?page=1`|
|origin|URL 协议、域名及端口号|`https://domain.cc:80`|
|protocol|URL 协议|`https:`|
|host|URL 域名及端口号|`domain.cc:80`|
|hostname|URL 域名|`domain.cc`|
|port|URL 端口号|`80`|
|pathname| "/" 后的文件路径|`/article`|
|search|URL 请求参数|`?page=1`|
|hash|"#" 后的内容||

在 URL 对象上调用 `toString()` 方法，返回 `url.href` :
```
const url = new URL("http://......")
url.toString() // a synonym for URL.href
```

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
