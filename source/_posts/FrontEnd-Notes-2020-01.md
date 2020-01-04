---
title: 'FrontEnd Notes [2020.01]'
catalog: true
date: 2020-01-02 09:57:13
subtitle:
header-img:
tags: FE
---
#### About

📅 2020 年 1 月的零散学习记录。

新年快乐！希望新的一年能坚持记笔记！

#### 2020/01/04
今天在处理正则表达式时遇到这样的疑惑：
```
const str = '\abc';
const reg = /\\abc/;
reg.test(str); // false
```
我们都知道反斜杠 `\` 是转义的作用，如果要输出斜杠，那么必须使用 `\\` ；在上面的代码中，`reg` 匹配是是 `\abc` ，为什么输出结果会是 `false` 呢？
经过查阅，发现在字符串中，'\' 也有转义的作用。如果反斜杠出现在字符的前面，那么他们就是一个整体，比如说 '\n' 表示换行，字符串 '\abc' 也涉及转义操作，由于 a 不是有效的转义符，所以就直接转成 'abc' 。下面的代码可以验证这个说法：
```
const str = "\abc";
console.info(str.length); // 3
"\abc" === "abc" //true
```
因此，上面的代码应该改成：
```
const str = '\\abc';
const reg = /\\abc/;
reg.test(str); // true
```

#### 2020/01/02
放大预览图片的某个部分：思路是计算 bbox 的 scale 数值，使之能填充满整个 container ；再计算原始图片的偏移量，使之只显示 bbox 部分。    
```
function Preview(props: {
  imgSrc: string;
  imgSize: { width: number; height: number };
  box: {
    x: number;
    y: number;
    width: number;
    height: number;
  };
}) {
  const [containerSize] = useState({ width: 350, height: 400 });

  const scale = Math.min(
    containerSize.width / props.box.width,
    containerSize.height / props.box.height
  );
  const boxCenter = {
    x: props.box.x + props.box.width / 2,
    y: props.box.y + props.box.height / 2
  };
  const containerCenter = {
    x: containerSize.width / 2,
    y: containerSize.height / 2
  };
  const tr = {
    x: containerCenter.x - boxCenter.x * scale,
    y: containerCenter.y - boxCenter.y * scale
  };

  return (
    <div style={{ position: "relative", ...containerSize, overflow: "hidden" }}>
      <img
        src={props.imgSrc}
        style={{
          ...props.imgSize,
          objectFit: "contain",
          position: "absolute",
          left: 0,
          top: 0,
          transform: `translate(${tr.x}px, ${tr.y}px) scale(${scale})`,
          transformOrigin: "top left",
          transition: "all 0.3s"
        }}
      />
    </div>
  );
}
```