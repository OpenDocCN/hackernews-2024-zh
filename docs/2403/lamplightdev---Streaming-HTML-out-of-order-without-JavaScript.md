<!--yml

category: 未分类

date: 2024-05-27 14:30:14

-->

# lamplightdev - 在没有JavaScript的情况下以任意顺序流式HTML

> 来源：[https://lamplightdev.com/blog/2024/01/10/streaming-html-out-of-order-without-javascript/](https://lamplightdev.com/blog/2024/01/10/streaming-html-out-of-order-without-javascript/)

# 在没有JavaScript的情况下以任意顺序流式HTML

让我们从一个演示开始：**[https://ooo.lamplightdev.workers.dev](https://ooo.lamplightdev.workers.dev)**：

</images/ooo-a0475c62.mp4>

这是一个简单的页面，呈现了一个包含10个项目的列表。在浏览器中启用和**禁用JavaScript**后尝试一下。有几点需要注意：

1.  **'应用壳'首先渲染** - 您会看到页眉和页脚，但列表项目将渲染的加载占位符。

1.  一秒钟后，**加载占位符被替换**为项目列表 - 但每个项目本身也有一个加载占位符。

1.  **然后项目内容按任意顺序渲染**，替换加载占位符 - 首先看到项目5，然后随着生成过程看到其他项目。

1.  如果查看页面源代码，您会看到**HTML的顺序是发送时的顺序 - 而不是渲染时的顺序**。

1.  页面使用不包含自定义元素的**Shadow DOM**。

相当不错，对吧？这可能是一个刻意制造的例子，但它是一种有趣的技术，使得以前没有JavaScript时不可能实现的功能成为可能。

查看**[此演示的代码](https://github.com/lamplightdev/ooo)**，或者继续阅读它是如何工作的解释。

* * *

## 背景

### 流式HTML

**流式HTML的概念** - 将HTML以块的形式从服务器发送到浏览器中生成 - **并不新鲜**。 在现代前端框架和单页面应用程序的时代初期，整个页面在浏览器中生成，这一概念似乎处于次要地位 - 但随着全栈框架向服务器端渲染回归，**流式响应再次流行起来**。

**流式HTML的优势**在于，在整个响应生成之前发送到浏览器之前，您可以**立即渲染一些内容**以指示用户有活动发生，并且可以在**等待响应的耗时部分生成**时更早地开始下载CSS和JavaScript等资产。

到目前为止缺少的是一种**无序流式HTML**的方式 - 即在生成HTML时以块的形式进行流式传输，而无需担心将哪些块以何种顺序发送到浏览器 - 并且仍然能够像上面的演示中那样使浏览器按正确顺序渲染HTML块。

现代全栈框架通过使用各种巧妙的技术实现了这一功能，所有这些技术都需要参与到特定框架中，并且需要大量的 JavaScript。这对于你的用例来说可能没问题，但如果我们能够**在没有任何 JavaScript 或框架的情况下实现相同的功能**，那会怎样？*现在你可以*。

### 影子 DOM

**[影子 DOM](https://developer.mozilla.org/en-US/docs/Web/API/Web_components/Using_shadow_DOM)** 是一种将 DOM 片段与页面其余部分隔离渲染的方法。虽然通常与自定义元素相关联，但**可以与任何 HTML 标签**一起使用，如普通的 `<div>` 标签。

它还有**插槽**的概念 - 这些标签充当门户，您可以通过在要渲染的标签上指定 `slot` 属性，在父标签内的其他位置渲染 HTML。这里影子根附加到外部 `<div>` 标签上，并且内部 `<div>` 标签被渲染到该影子根中的插槽中：

```
<div>
  #shadowroot
    <header>Header</header>
    <main>
      <slot name="content"></slot>
    </main>
    <footer>Footer</footer>

  <div slot="content">
    This div will be rendered inside the slot above. Magic!
  </div>
</div> 
```

* * *

## 要求

那么如何使用影子 DOM 来无序流式传输 HTML？你需要几件事情：

1.  一个**支持流式响应的 HTTP 服务器**。你在这里很幸运，几乎所有语言都对此提供了广泛的支持。我选择了 **[Hono](https://hono.dev/)**，因为它是一个轻量级的服务器，基于 Web 标准构建，可以在 Node 和各种边缘平台上运行。但需要注意的是，并不依赖于 JavaScript 后端 - 同样可以在 PHP、Java、Go 等上实现相同的功能。

1.  一个**支持流式的模板语言**。理论上你不需要模板语言 - 你可以手工制作 HTML 并手动管理流式传输 - 但那是一项繁重的工作。在 JavaScript 的世界中，没有太多独立的模板语言支持流式传输，但最近有一个叫做 **[SWTL](https://github.com/thepassle/swtl)** 的项目做到了。SWTL 是为了在服务工作者中使用而创建的，但由于我们始终使用 Web 标准，因此也可以在服务器上使用。SWTL 的另一个优点是你几乎可以用它来处理任何东西 - 异步函数、生成器、数组、响应 - 它都能处理。

1.  **声明式影子 DOM** - 直到最近，自定义影子 DOM 是一种仅限于浏览器的技术 - 你只能使用 JavaScript 在浏览器中创建影子 DOM - 但现在，多亏了 **[声明式影子 DOM](https://developer.chrome.com/docs/css-ui/declarative-shadow-dom)** (DSD)，你可以在服务器上创建影子 DOM，并通过在 `<template>` 标签上使用新的 `shadowrootmode` 属性，浏览器将在不使用 JavaScript 的情况下渲染它。然后自动将影子根附加到包含的元素上：

```
<div>
  <template shadowrootmode="open">
    <header>Header</header>
    <main>
      <slot name="content"></slot>
    </main>
    <footer>Footer</footer>
  </template>

  <div slot="content">
    This div will be rendered inside the slot above
    without JavaScript. More magic!
  </div>
</div> 
```

* * *

## 将其组合起来

那么初始演示是如何创建的？让我们通过一个简化的代码示例来解析它：

```
import { Hono } from 'hono';
import { stream } from 'hono/streaming';

import { render, html } from 'swtl';

import {
  delayed,
  createReadableStreamFromAsyncGenerator
} from './utils.js';

const app = new Hono();

app.get('/', (ctx) => {

  const template = ({ name }) => html` <html>
      <head>
        <title>Streaming example>
      </head>
      <body>
        <div>
          <template shadowrootmode="open">
            <header>Header</header>
            <main>
              <slot name="content"></slot>
            </main>
            <footer>Footer</footer>
          </template>

            <!--
              The html above gets sent first to the browser
            -->

            <!--
              An artificial delay is added the slot content
              to simulate a slow response from the server:
            --> ${delayed(1000, html` <p slot="content">
                Hi ${name}!
              </p> `)} <!--
              the remaining html below is sent to the browser once
              the delayed content has been sent
            -->
          </div>
        </div>
      </body>
    </html> `;

  return stream(ctx, async (stream) => {
    ctx.res.headers.set('Content-Type', 'text/html');

    await stream.pipe(
      createReadableStreamFromAsyncGenerator(
        render(
          template({ name: 'Ada' })
        )
      )
    );
  });
});

export default app;
```

到这里就是全部啦！浏览**[这个演示的代码](https://github.com/lamplightdev/ooo)**，然后自己试试。我很想听听你对这种技术的想法，还有你能想到的任何新用途，请**[联系我](https://lamplightdev.com/contact/)**。下次见！👋。

*特别感谢[@passle_](https://twitter.com/passle_)，《SWTL》的作者，为本文的校对和反馈提供支持。*
