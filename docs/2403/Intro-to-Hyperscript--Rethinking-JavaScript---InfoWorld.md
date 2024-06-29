<!--yml

category: 未分类

date: 2024-05-27 15:03:43

-->

# Intro to Hyperscript: Rethinking JavaScript | InfoWorld

> 来源：[https://www.infoworld.com/article/3708109/intro-to-hyperscript-rethinking-javascript.html](https://www.infoworld.com/article/3708109/intro-to-hyperscript-rethinking-javascript.html)

Some of us remember [HyperCard](https://hypercard.org/)，这是编程语言进化树中的一个有趣分支。如果你有特定年龄段的记忆，你可能甚至是通过HyperCard学习编程的。Hyperscript是一种较新的技术，它借鉴了HyperCard的一些优点，尤其是其简单性和类似英语的语法，并将其应用到浏览器环境中。它是一种JavaScript替代方案，可以独立工作，也可以与[HTMX](https://www.infoworld.com/article/3706951/htmx-dynamic-html-without-the-javascript.html)配合，简化[JavaScript前端](https://www.infoworld.com/article/3661810/reactive-javascript-the-evolution-of-front-end-architecture.html)的常见脚本需求。

## An example worth a thousand words

Before we get into too much discussion, let’s look at a [Hyperscript example](https://hyperscript.org/#be-async-with-no-extra-code)，以便更好地理解其精神。

```
 <div _="init fetch https://stuff as json then put result into me">Using fetch() API...</div> 
```

It’s pretty obvious what this code does. The underscore property is the special one the denotes a Hyperscript. In JavaScript, this same functionality might look something like this:

```
 <div id="myDiv" onload="async function() { 
  const response = await fetch('https://stuff', { 
    headers: { Accept: 'application/json', }
  }); 
  const data = await response.json(); 
  myDiv.innerHTML = JSON.stringify(data); 
}">
</div> 
```

In English, it says: "When the div element is loaded, send an async request to 'https://stuff' and put the results inside the div.”  Hopefully you can see already that Hyperscript feels almost like a behavioral extension of HTML.

## What is Hyperscript?

Hyperscript是一种简化的、更像英语的JavaScript。正如[Sacha Greif](https://www.infoworld.com/article/3688770/state-of-javascript-a-conversation-with-sacha-greif.html)指出的那样，你也可以把它看作是一种特定领域的语言，或者DSL。实质上，Hyperscript将JavaScript简化为专门用于构建前端UI常见和重复需求的语法。它引入了一系列约定，使得编码更加简洁。

Hyperscript是[HTMX](https://www.infoworld.com/article/3706951/htmx-dynamic-html-without-the-javascript.html)的姊妹项目，由同一开发者Carson Gross创作。这两个项目反映了Gross对简单性的执着追求，并试图将其无情地应用于大规模、活跃的问题领域。在HTMX中，我们看到了一个更强大的HTML，可以消除过去十年间随着开发者们拥抱响应式框架、JSON和[类REST API](https://www.infoworld.com/article/3706031/what-is-rest-the-de-facto-web-architecture-standard.html)而演变的复杂性。而在Hyperscript中，我们看到了JavaScript语言复杂度似乎永无止境扩展的一个替代方案。这是一个令人心动的建议。

## Tackling front-end complexity

面对前端开发者在代码战线上所面临的混乱感，确实让人感到不知所措。谁不想用一种易于记忆、可以从记忆中直接打出来的表达性语言来替换样板JavaScript呢？这是一种可以仅凭记忆就能执行日常基本编码的语言。

让我们考虑一个例子。在以下Hyperscript片段中，典型的[按钮点击计数器示例](https://hyperscript.org/docs/#control-flow)变成了：

```
 <button _="on click increment :x
            if :x <= 3
              put :x into the next <output/>
            else
              put '3 is the max...' into the next <output/>
            end">
Click Me
</button>
<output>--</output> 
```

在React中是相同的例子：

```
 import React from "react";

const Counter = () => {
  const [x, setX] = React.useState(0);

  const handleClick = () => {
    setX((prevX) => {
      if (prevX <= 3) {
        return prevX + 1;
      } else {
        return 3;
      }
    });
  };

  return (
    <div>
      <button onClick={handleClick}>Click Me</button>
      <output>{x}</output>
    </div>
  );
};

export default Counter; 
```

当然，更短并不一定意味着更简单。但在这种情况下，Hyperscript的自描述明显性与React相比显得更出色。现在，有些人可能会说React更复杂，因为它更强大；它是一种能够实现复杂功能的语言。但在这种情况下，我们只是在比较JavaScript前端开发中最常见的方法（即React）与Hyperscript。我们正在看的是*可以*简化的日常活动。

Hyperscript旨在取代JavaScript。或许*扩展*是一个更好的词。Hyperscript的创始人Caron Gross指出，这是一个“投机性”的项目。尽管如此，它经过深思熟虑，功能强大而雄心勃勃。可以想象使用Hyperscript开发企业级应用程序是完全可能的。

对于Hyperscript来说，最大的障碍可能是开发者对JavaScript的集体熟悉度和依赖。当然，有时它可能会令人困惑和棘手，但它是*我的*令人困惑和棘手的语言。如果Hyperscript确实流行起来，它可能会在大多数项目中与JavaScript结合使用。

## Hyperscript中的异步事件

让我们来看看[Hyperscript如何处理事件](https://hyperscript.org/docs/#sending-events)：

```
 <button _="on click send foo to the next <output/>">Send Foo</button>
<button _="on click trigger bar on the next <output/>">Send Bar</button>
<output _="on foo put 'I got a foo event!' into me
         on bar put 'I got a bar event!' into me">
No Events Yet...
</output> 
```

[响应式编程](https://www.infoworld.com/article/3701889/what-is-reactive-programming-programming-with-event-streams.html)是编程历史中的一个重要情节，Hyperscript完全接受了这一点。在这个例子中，我们可以看到Hyperscript如何处理异步事件。事件系统非常强大，包括一系列类似于过滤、事件消息对象、排队等反应式风格函数。

可以看到，“下一个 `<output/>`”这个短语可以引用DOM中的另一个元素，具体来说，是下一个 `<output/>` 元素，并将事件发送到它。这是一种非常简洁和明显的方式来完成一些本来会相当冗长和笨拙的事情，或者至少需要一些反应式的连线。这是Hyperscript通过设计消除关注点分离的一个例子。

## 循环、条件和日志

在一些前端上下文中（比如[JSX for React](https://www.infoworld.com/article/3307858/tutorial-programming-in-react-for-beginners.html)），循环可能会有些棘手。这里是在Hyperscript中的[循环示例](https://hyperscript.org/docs/#loops)：

```
 for x in [1, 2, 3] index i
  log i, "is", x
end 
```

这个例子还让我们看到了 Hyperscript 如何处理日志记录。使用逗号分隔的值非常简单直接。

我们已经看到了 Hyperscript 如何处理带有 `else` 的 `if` 命令。注意，它以 `End` 关键字结束（除非你已经在脚本的末尾，如在元素属性上发生的情况）：

```
 if :x <= 3
  put :x into the next <output/>
else
  put '3 is the max...' into the next <output/>
End 
```

Hyperscript 还支持一个 `unless` 修改器，可以引用诸如 CSS 属性之类的东西：

```
 <button _="on click toggle .bordered on #second-button">
Toggle Next Border
</button>
<button id="second-button"
        _="on click toggle .red unless I match .bordered">
Toggle My Background
</button> 
```

这段代码片段使得第二个按钮在没有 `.bordered` 类时改变其颜色。这是对使用 JavaScript、CSS 和 HTML 处理起来相对笨拙的代码的一种相当简明的处理。

## 行为的局部性

新程序员被灌输的原则之一是所谓的关注点分离（SoC）。大多数情况下，这个原则都是有效的。通过实施 SoC，我们获得了解耦合的组件，这使得系统更加强大。然而，需要考虑的反向流动是由 Gross 称为 [行为的局部性](https://htmx.org/essays/locality-of-behaviour/)。他将这个想法嵌入了 Hyperscript 中，你也可以在其他项目中找到类似的，比如 [Tailwind](https://www.infoworld.com/article/3622288/tailwind-css-learn-the-joys-of-functional-responsive-css.html)。

这里的思想是，分离关注点实际上可能导致更难以跟踪的系统。前端的关注点分离通常意味着将你的标记（视图）放在一个地方，你的 JavaScript（行为）放在另一个地方，你的 CSS（呈现）放在其他地方。在更架构化的情况下，这种设计的好处往往是值得怀疑的。然而，在基于 HTML、CSS 和 JavaScript 的项目中，必须在不同的上下文中跳转并始终记住任务的线索，这通常会令人头疼。这实际上是 JSX 及其背后的 [Styled Components 框架](https://www.infoworld.com/article/3678950/intro-to-css-in-js-generating-css-from-javascript.html) 魅力之一的体现。

### 违反了关注点分离

Hyperscript 将你通常会提取到 JavaScript 中的大部分琐碎工作包装在 Hyperscript 语法中，因此你可以直接将相当复杂的功能内联到标记中。这使得保持事物的整体性变得容易，并且使其更加自解释。

不足的是，如果你需要对超出内置语法范围的行为进行更改，可能会出现问题。换句话说，你将发现你的强耦合组件依赖于 Hyperscript 引擎本身，而你需要修改它。

我还没有在大型真实项目中使用过 Hyperscript，所以无法直接评论那个。然而，语言中确实有 [扩展性支持](https://hyperscript.org/docs/#extending)。

也可以同时运行JavaScript和Hyperscript，这样可以逐步增强功能。对我来说，这也引发了一个问题，即在React、Svelte或Vue项目中同时运行这些语言会是什么样子。

## 结论

当我开始接触Hyperscript时，我非常怀疑——甚至有点自大。然而，我逐渐被Hyperscript的感觉所吸引。我认为它会很快取代JavaScript吗？不，我不这么认为。但我可以想象一个世界，在这个世界里，HTMX和Hyperscript极大地简化了JavaScript前端开发。

这种向简化编程范式的推进中有一种我称之为*应用常识*的东西。通过质疑假设并重新利用我们所学的一切，也许我们可以构建更好的工具。

我们知道的一件事是，复杂性会在软件开发中扼杀创造力和生产力。作为工程师，我们永远面临着构建复杂抽象的风险——这些抽象可能是复杂性的最后一根稻草，会导致项目失败。

有时候我们进入技术行业是因为*喜欢*它；工具会自己生活。像React这样的工具，本身就成为了产品。在某些情况下这没问题，但在其他时候，像Hyperscript这样的简单替代方案可能更好。

至少，Hyperscript有新的想法可以贡献。就像HTMX一样，这种语言可以帮助改善在Web上开发的整体体验。

版权所有 © 2023 IDG Communications, Inc.
