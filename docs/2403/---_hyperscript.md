<!--yml
category: 未分类
date: 2024-05-29 12:33:25
-->

# ///_hyperscript

> 来源：[https://hyperscript.org/](https://hyperscript.org/)

# An easy & approachable language for modern web front-ends

Enhance HTML with concise DOM, event and async features. **Make writing interactive HTML a joy.**

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

**Install:** `<script src="https://unpkg.com/hyperscript.org@0.9.12"></script>` 

## Events are first class citizens

```
<button _="on click send hello to <form />">Send</button>

<form _="on hello alert('got event')"> 
```

Highly interactive user experiences **without promises, async / await or callback hell**.

**_hyperscript** transparently handles asynchronous behavior for you.

```
<div _="on click wait 5s send hello to .target">

<div _="init fetch https://stuff as json then put result into me">Using fetch() API...</div> 
```

## Enhance existing code

```
<div _="init js alert('Hello from JavaScript!') end"></div>

<div _="init js(haystack) return /needle/gi.exec(haystack) end">

<div _="install Draggable(dragHandle: .titlebar)"> 
```

```
<div _="on click tell <p/> in me add .highlight">

<div _="tell <details /> in .article set you.open to false"> 
```

## Debugging and extending

**Graphically [debug](/docs#debugging)** and step through code as it runs.

```
on click breakpoint 
```

**_hyperscript** is natively **[extensible](/docs/#extending)**. Add new commands or expressions using vanilla javascript.

```
_hyperscript.addCommand(
  "foo",
  (parser, rt, tokens) => ...) 
```