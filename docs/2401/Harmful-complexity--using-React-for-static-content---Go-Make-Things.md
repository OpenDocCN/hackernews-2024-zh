<!--yml

类别：未分类

日期：2024-05-27 14:28:04

-->

# 有害复杂性：将 React 用于静态内容 | Go Make Things

> 来源：[`gomakethings.com/harmful-complexity-using-react-for-static-content/`](https://gomakethings.com/harmful-complexity-using-react-for-static-content/)

# 有害复杂性：将 React 用于静态内容

昨天，我写了一篇关于[两种有害复杂性](https://gomakethings.com/the-two-types-of-harmful-complexity-in-a-web-project/)的文章，这是我在与[咨询客户](https://gomakethings.com/consulting)合作时经常遇到的……

1.  使用过度工程化的工具（如 React）用于简单的、大多数静态网站。

1.  使用过少的工具为大型、交互式、数据驱动的用户界面。

今天，我想谈谈第一种类型，即使用过度工程化的工具。让我们深入了解一下！

## React 不好

我对此感觉非常明确。React 是糟糕的软件。

React 背后的想法非常酷。更新一些数据，然后 UI 会自动 *react*。但 React 不是唯一能做到这一点的工具，而且他们也不是这个想法的发起者。他们只是帮助推广了它。

如果您的站点主要是静态内容，React 对于使用它的人和必须处理和维护它的开发人员都是积极有害的。

React 极大，荒谬地庞大。[它很慢](https://gomakethings.com/react-is-still-absolutely-terrible-for-web-performance/)（[在其核心](https://gomakethings.com/react-is-slow-at-its-very-core/)，并且永远不会不慢）。用 JavaScript 渲染 HTML 意味着你的非常弹性的 HTML 现在非常脆弱。

所有这些对用户来说都是可怕的。但 React 对于你的开发团队也是可怕的。

React 需要一个构建步骤，并且该构建步骤通常围绕着 Webpack。Webpack 很强大，但也非常复杂，而且在能做和不能做的方面有些死板。

一旦你进入 React 生态系统，你就会被激励为更多的事情使用它。

这意味着你的 HTML、CSS 和 JavaScript 最终都会被 JavaScript 渲染。这对你的用户来说是可怕的。但这也意味着只有专注于 JavaScript 的开发人员才能在你的站点上工作，而且通常对 HTML 和 CSS 的了解不如对 JavaScript 的了解多。

这导致了网站，坦率地说，HTML 和 CSS 非常糟糕，直接损害了最终用户，通常是不可访问的，并且随着时间的推移越来越难以维护。

React 就像一个黑洞。它有引力，慢慢地吸收你整个站点。

## 静态站点应该是（大多数时候）静态的

如果你的站点主要是不变的内容，或者不基于用户交互而改变，那么它应该作为静态 HTML 文件从服务器提供。

这可能意味着使用静态站点生成器，或者可能意味着服务器端渲染。

无论哪种情况，最终结果都将是一个……

+   对于您的用户更快、更有弹性。

+   对于您的开发人员更容易维护。

+   可以由没有 JS 专业知识的人来处理。

那么大多数静态站点上的交互式内容呢？

我建议为此使用*交互岛*。我个人认为[Web 组件是处理这个问题的最佳方式](https://gomakethings.com/the-elevator-pitch-for-web-components/)，但也有其他有效的方法。

明天，我们将讨论静态站点生成器与服务器端渲染的优势，以及如何选择使用哪种方法。

如果您有一个需要专家帮助的项目，[我目前有一个咨询名额空缺](https://gomakethings.com/consulting)。我很乐意和您合作！
