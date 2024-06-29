<!--yml

类别：未分类

时间：2024-05-29 12:33:25

-->

# ///_hyperscript

> 来源：[https://hyperscript.org/](https://hyperscript.org/)

# 用于现代 web 前端的易用且易于接触的语言

通过简洁的 DOM、事件和异步功能增强 HTML。**使编写交互式 HTML 成为一种乐趣**。

```
on pointerdown
  repeat until event pointerup
    set rand to Math.random() * 360
    transition
      *background-color
      to `hsl($rand 100% 90%)`
      over 250ms
  end 
```

**安装:** `<script src="https://unpkg.com/hyperscript.org@0.9.12"></script>`

## 事件是公民级成员

```
<button _="on click send hello to <form />">Send</button>

<form _="on hello alert('got event')"> 
```

高度互动的用户界面，**无需使用承诺、async / await 或回调地狱**。

**_hyperscript** 会**透明地**为您处理异步行为。

```
<div _="on click wait 5s send hello to .target">

<div _="init fetch https://stuff as json then put result into me">Using fetch() API...</div> 
```

## 增强现有代码

```
<div _="init js alert('Hello from JavaScript!') end"></div>

<div _="init js(haystack) return /needle/gi.exec(haystack) end">

<div _="install Draggable(dragHandle: .titlebar)"> 
```

```
<div _="on click tell <p/> in me add .highlight">

<div _="tell <details /> in .article set you.open to false"> 
```

## 调试和扩展

**图形化 [调试](/docs#debugging)** 并在代码运行时逐步执行。

```
on click breakpoint 
```

**_hyperscript** 原生**[可扩展](/docs/#extending)**。使用 vanilla javascript 添加新命令或表达式。

```
_hyperscript.addCommand(
  "foo",
  (parser, rt, tokens) => ...) 
```
