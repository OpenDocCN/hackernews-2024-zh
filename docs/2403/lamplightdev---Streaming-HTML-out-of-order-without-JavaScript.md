<!--yml
category: 未分类
date: 2024-05-27 14:30:14
-->

# lamplightdev - Streaming HTML out of order without JavaScript

> 来源：[https://lamplightdev.com/blog/2024/01/10/streaming-html-out-of-order-without-javascript/](https://lamplightdev.com/blog/2024/01/10/streaming-html-out-of-order-without-javascript/)

# Streaming HTML out of order without JavaScript

Let's start with a demo: **[https://ooo.lamplightdev.workers.dev](https://ooo.lamplightdev.workers.dev)**:

 </images/ooo-a0475c62.mp4> 

This is a simple page that renders a list of 10 items. Try it with and **without JavaScript enabled** in your browser. There's a few things to notice:

1.  **The 'app shell' renders first** - you see the header and the footer, but there's a loading placeholder where the list of items will be rendered.

2.  After a second the **loading placeholder is replaced** with the list of items - but with each item itself having a loading placeholder.

3.  **The content of the items then renders out of order**, replacing the loading placeholders - you see item 5 first, then the other items as they are generated.

4.  If you look at the page source you'll see that the **html is in the order it was sent - not the order it was rendered in**

5.  The page makes use of **Shadow DOM** without Custom Elements.

Pretty nice, right? It may be a contrived example but it's an interesting technique that enables things that have not been possible before without JavaScript.

Have a look at the **[code for this demo](https://github.com/lamplightdev/ooo)**, or read on for an explanation of how it works.

* * *

## Background

### Streaming HTML

The **concept of streaming HTML** - sending HTML from a web server to a browser in chunks as it is generated - is **nothing new**. It seemed to take a back seat at the beginning of the age of modern front-end frameworks and Single Page Applications - where the entire page was generated in the browser - but as the pendulum swings back towards server-side rendering with full stack frameworks, **streaming responses are becoming popular again**.

The **advantages of streaming HTML** over waiting for the entire response to be generated before sending it to the browser are clear - you can **render something immediately** to indicate to the user that something is happening, and you can start downloading assets like CSS and JavaScript earlier, while you **wait for the more time consuming parts of the response** to be generated.

What's been lacking up to this point is a way to **stream HTML out of order** - that is to stream the HTML in chunks as it's generated without worrying about the order in which the chunks are sent to the browser - and still have the browser render the chunks of HTML in the correct order as in the demo above.

Modern full stack frameworks enable this functionality by using a variety of clever techniques, all of which require buy-in to the particular framework and a **hefty chunk of JavaScript**. That might be fine for your use case, but what if we could **achieve the same thing without any JavaScript or framework?** *Well now you can*.

### Shadow DOM

**[Shadow DOM](https://developer.mozilla.org/en-US/docs/Web/API/Web_components/Using_shadow_DOM)** is a way to render a piece of DOM in isolation from the rest of the page. Whilst often associated with Custom Elements, Shadow DOM **can be used with any HTML tag**, such as the humble `<div>` tag.

It also has the concept of **slots** - tags that act as portals that you can render HTML into from elsewhere within the parent tag by specifying a `slot` attribute on the tag you want to render. Here the shadow root is attached to the outer `<div>` tag, and the inner `<div>` tag is rendered into the slot in that shadow root:

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

## Requirements

So how do you use Shadow DOM to stream HTML out of order? There's a few things you need:

1.  A **http server that supports streaming responses**. You're in luck here, there is pretty much universal support for this across all languages. I've opted for **[Hono](https://hono.dev/)** as it's a lightweight server, built on web standards, that runs on node as well as a wide variety of edge platforms. It's worth noting though that there's no dependency on a JavaScript backend - the same thing can be achieved on PHP, Java, Go, etc.

1.  A **templating language that supports streaming**. In theory you don't need a templating language - you could handcraft the HTML and manually manage the streaming - but that's a lot of work. In the JavaScript world, there aren't a lot of standalone templating languages that support streaming, but a recent project called **[SWTL](https://github.com/thepassle/swtl)** does. SWTL was created to be used in Service Workers, but since we're using web standards all the way down, it can be used on the server too. The other great thing about SWTL is that you can chuck pretty much anything at it - async functions, generators, arrays, responses - and it will handle it all.

2.  **Declarative Shadow DOM** - Until recently custom Shadow DOM was a browser only technology - you could only create Shadow DOM in the browser using JavaScript - but now, thanks to **[Declarative Shadow DOM](https://developer.chrome.com/docs/css-ui/declarative-shadow-dom)** (DSD), you can create Shadow DOM on the server and the browser will render it without JavaScript by using a new `shadowrootmode` attribute on a `<template>` tag. The shadow root is then automatically attached to the containing element:

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

## Piecing it together

So how was that initial demo created? Let's break it down in a simplified code example:

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

And that's all there is to it! Browse the **[code for this demo](https://github.com/lamplightdev/ooo)** and have a play with it yourself. I'd love to hear your thoughts on this technique - and any novel use cases you can think of for it - so please **[get in touch](https://lamplightdev.com/contact/)**. Until next time 👋.

*With thanks to [@passle_](https://twitter.com/passle_), author of SWTL, for proofreading and feedback on this article.*