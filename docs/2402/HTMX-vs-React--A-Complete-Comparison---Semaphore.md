<!--yml

category: 未分类

日期：2024-05-27 15:04:30

-->

# HTMX vs React：完整比较 - Semaphore

> 来源：[https://semaphoreci.com/blog/htmx-react](https://semaphoreci.com/blog/htmx-react)

HTMX的终极目标是在HTML中直接提供现代浏览器交互性，无需JavaScript。尽管相对较新，HTMX于2020年末首次发布后很快引起了IT网络社区的关注。

在[2023 JavaScript Rising Stars](https://risingstars.js.org/2023/en#section-framework)“前端框架”类别中排名第二（仅次于React），并获得GitHub加速器的位置以及在GitHub上超过20k星标，HTMX正在迅速赢得人气。为什么会引起这么多关注？它是否有可能颠覆React？让我们一探究竟！

在这篇HTMX与React的指南中，您将了解我们为什么选择HTMX，它是什么，提供了哪些特性，并从性能、社区、功能等方面与React进行比较。

## HTMX vs React 简介

请查看下面的总结表格，直接进行HTMX与React的比较：

|  | **HTMX** | **React** |
| --- | --- | --- |
| 开发者 | Big Sky Software | Meta |
| 开源 | ✅ | ✅ |
| GitHub 星标 | 29k+ | 218k+ |
| 体积 | 2.9 kB | 6.4 kB |
| 语法 | 基于HTML，带有自定义属性 | 基于JSX，是JavaScript的扩展版本 |
| 目标 | 直接在HTML中添加现代交互特性 | 提供基于组件的全功能UI JavaScript库 |
| 学习曲线 | 缓和 | 陡峭 |
| 特性 | AJAX请求和一些其他次要功能 | 可组合性、单向数据绑定、状态管理、钩子等诸多功能 |
| 性能 | 优秀 | 良好，在大规模应用或复杂Web应用中尤为突出 |
| 整合 | 可嵌入到现有HTML页面中 | 可嵌入到现有HTML页面中，但主要用于基于JavaScript的项目 |
| 社区 | 小型但正在增长 | 市场上最大的社区 |
| 生态系统 | 小型 | 非常丰富 |

## 我们如何走到React：从jQuery到现代Web框架

在Web开发的早期阶段，开发人员依赖于jQuery来处理AJAX请求、DOM操作和事件处理。随着时间的推移，在线应用程序变得更加现代化、结构化和可扩展。这就是Angular、React和Vue等框架和库产生影响的时刻。

React引入了基于组件的架构，彻底改变了Web开发。其声明式UI和单向数据流的方法简化了Web开发，促进了可重用性和可维护性。这些特点使React成为构建动态、响应式和交互式Web应用的首选解决方案。

## 我们如何走到HTMX：从Web框架到更现代的HTML

尽管像React、Vue和Angular这样的Web框架非常适合构建结构化的Web应用程序，但其复杂性可能对寻求简单性的开发人员构成巨大的负担。这就是HTMX发挥作用的地方！

HTMX是一种现代交互的轻量级解决方案，类似于React，但集成简单，没有jQuery的开销。它通过自定义属性扩展HTML，使得可以不用JavaScript代码即可进行AJAX请求。HTMX的理念在于保持简单，使开发者能够在不放弃熟悉的HTML的情况下探索Web的魔力。

HTMX作为一个简化且灵活的选择，在由更复杂前端框架主导的宇宙中显得格外耀眼。在下面的章节中了解更多。

## HTMX：交互的新现代方法

[HTMX](https://htmx.org/)是一个轻量级、无依赖、可扩展的JavaScript前端库，可直接从HTML中访问现代浏览器功能。详细来说，它允许您直接在HTML代码中处理[AJAX请求](https://developer.mozilla.org/en-US/docs/Glossary/AJAX)、[CSS过渡](https://developer.mozilla.org/en-US/docs/Web/CSS/CSS_Transitions/Using_CSS_transitions)、[WebSockets](https://developer.mozilla.org/en-US/docs/Web/API/WebSockets_API)和[服务器发送事件](https://developer.mozilla.org/en-US/docs/Web/API/Server-sent_events/Using_server-sent_events)。

该库通过设置特殊的HTML属性，即可让您访问大多数这些功能，而无需编写一行JavaScript代码。这样，HTMX将HTML带入了一个新的层次，使其成为一个完整的超文本。

现在让我们通过一些HTMX示例来看看这个库提供了什么。

### AJAX请求触发器

HTMX背后的主要概念是能够直接从HTML发送AJAX请求。这得益于以下属性：

+   [`hx-get`](https://htmx.org/attributes/hx-get/)：向指定URL发送`GET`请求。

+   [`hx-post`](https://htmx.org/attributes/hx-post/)：向指定URL发送`POST`请求。

+   [`hx-put`](https://htmx.org/attributes/hx-put/)：向指定URL发送`PUT`请求。

+   [`hx-patch`](https://htmx.org/attributes/hx-patch/)：向指定URL发送`PATCH`请求。

+   [`hx-delete`](https://htmx.org/attributes/hx-delete/)：向指定URL发送`DELETE`请求。

当触发具有HTMX属性之一的HTML元素时，将发起到指定URL的指定类型的AJAX请求。默认情况下，元素由[“自然”事件](https://htmx.org/docs/#triggers)（例如`<input>`、`<textarea>`和`<select>`的`change`，`<form>`的`submit`，其他所有元素的`click`）触发。您可以通过属性[`hx-trigger`](https://htmx.org/attributes/hx-trigger/)自定义此行为。

现在，考虑下面的HTMX示例：

```
<div hx-get="/users">
    Show Users
</div>
```

这告诉浏览器：

> “当用户点击`<div>`时，向`/users`端点发送`GET`请求，并将响应渲染到`<div>`中”

请注意，为了使此机制生效，`/users`端点应返回原始HTML。

### 查询参数和主体数据

HTMX设置查询参数和主体数据的方式取决于HTTP请求的类型：

+   **`GET`请求**：默认情况下，`hx-get`不会自动将任何查询参数包含在AJAX请求中。要设置查询参数，请在传递给`hx-get`的URL中指定它们。否则，可以使用[`hx-params`](https://htmx.org/attributes/hx-params/)属性覆盖HTMX的默认行为。

+   **非`GET`请求**：当元素是一个`<form>`时，AJAX请求的正文将包括所有输入的值，使用它们的`name`属性作为参数名。当它不是一个`<form>`时，正文将涉及最近的包围`<form>`中所有输入的值。否则，如果元素有一个`value`属性，那将用于正文。要将其他元素的值添加到正文中，请使用带有要包含在请求正文中的所有元素的CSS选择器的[`hx-include`](https://htmx.org/attributes/hx-include/)属性。然后，您可以使用`hx-params`属性来过滤一些正文参数。您还可以编写一个自定义的[`htmx:configRequest`](https://htmx.org/events/#htmx:configRequest)事件处理程序来以编程方式修改正文定义逻辑。

### AJAX结果处理

正如前面提到的，HTMX会用服务器返回的HTML内容替换触发AJAX请求的元素的内部HTML。您可以使用[`hx-swap`](https://htmx.org/attributes/hx-swap/)和[`hx-target`](https://htmx.org/attributes/hx-target/)属性自定义此行为：

+   `hx-swap`定义了如何处理服务器返回的HTML，接受以下自解释值之一：`innerHTML`（默认）、`outerHTML`、`beforebegin`、`afterbegin`、`beforeend`、`afterend`、`delete`、`none`。

+   `hx-target`接受CSS选择器，并指示HTMX将交换逻辑应用于所选元素。

现在，看一下下面的HTMX片段：

```
<button
  hx-post="/tasks"
  hx-swap="afterend"
  hx-target=".todo-list"
>
  Add task
</button>
```

这告诉浏览器：

> “当用户点击`<button>`节点时，执行到`/tasks`端点的`POST`请求，并将服务器返回的HTML附加到`.todo-list`元素上”

很棒！您刚刚探索了HTMX的基础知识和其工作原理的基础。请记住，这些只是该库支持的部分功能。欲了解更多信息，请[查阅文档](https://htmx.org/docs/)。

## HTMX vs React：比较这两种Web技术

现在您知道HTMX是什么以及它如何工作，让我们看看它与React的比较，React作为前端Web开发库的现任之王。本节将探讨确定HTMX和React哪个更好时需要考虑的关键方面。

现在是时候深入了解HTMX与React的比较了！

### 方法

+   **HTMX**：它扩展HTML，提供了在标记中直接与服务器交互的能力。它优先考虑简单性、简洁性和可读性：

```
<div hx-get="/hello-world">
    Click me!
</div> 
```

+   **React**：一个基于可重用组件的JSX编写的用户界面构建的全功能JavaScript库：

```
import React, { useState } from "react"

export default const HelloWorldComponent = () => {
  const [responseData, setResponseData] = useState(null)

  const handleClick = () => {
    fetch("/hello-world")
      .then(response => response.text())
      .then(data => {
        setResponseData(data)
      })
  }

  return (
    <div onClick={handleClick}>
      {responseData ? <>{responseData}</> : "Click me!"}
    </div>
  )
} 
```

### 学习曲线

+   **HTMX**：由于其基于 HTML 的语法和方法，HTMX 提供了平滑的学习曲线。对于已经熟悉传统网页开发的开发人员，他们可以在几天内掌握它，而新手则可以从零开始使用它。

+   **React**：由于其独特的网页开发方法，React 具有陡峭的学习曲线。在构建第一个 React 应用程序之前，您需要了解 SPA（[单页面应用](https://en.wikipedia.org/wiki/Single-page_application)）的概念，虚拟 DOM，JSX，状态管理，props，重新渲染等等。这可能会让一些初学者感到不知所措。

### 特性

+   **HTMX**：该库背后的核心概念可以概括为在 HTML 中实现 AJAX 调用，而无需 JavaScript 代码。虽然还有其他很酷的功能，但这基本上概括了 HTMX 提供的内容。

+   **React**：使 React 如此受欢迎的一些功能包括基于组件的架构，支持代码重用，采用 JSX 语法进行轻松的 UI 开发，强大的状态管理，[hooks](https://react.dev/reference/react/hooks) 支持，同时支持客户端和服务器端渲染，高效的虚拟 DOM，CSS-in-JS 支持等等。

### 性能

+   **HTMX**：其轻量级且无依赖的特性意味着依赖于 HTMX 的网页将具有快速的初始页面加载和减少的客户端处理。总体而言，在处理简单交互的应用程序时，HTMX 的表现非常出色。

+   **React**：使用 React 构建的 SPA 应用通常包含大量 JavaScript。这导致网络利用率和客户端渲染时间增加。然而，虚拟 DOM 和高效的协调算法使 React 能够迅速更新 UI，使其适用于大规模应用程序。

### 集成

+   **HTMX**：它可以嵌入到任何 HTML 网页中。HTMX 与可以返回原始 HTML 内容的后端技术（如 Node.js、Django、Laravel、Spring Boot、Flask 等）本地集成。

+   **React**：作为一个前端库而不是框架，技术上可以将其集成到任何现有网站中。同时，集成 React 可能需要额外的配置，特别是在非围绕 JavaScript 构建的前端项目中。

注意，HTMX 和 React 可以在同一个项目中共存。这意味着您可以在页面的不同部分同时使用 React 和 HTMX，并且甚至可以有依赖于 HTMX 属性的 React 组件。

### 使用案例

+   **HTMX**：最适合需要简单、现代和动态交互的项目。HTMX 是一个轻量且高效的选择，适用于不需要完整前端框架的高级功能的项目。对于希望提供交互式 HTML 页面但不想编写专门的客户端 JavaScript 逻辑的后端开发人员而言，它也是理想选择。

+   **React**：最适合开发需要提供丰富用户体验和/或处理复杂状态的单页应用和复杂Web应用。对于希望在多个项目中重复使用UI组件的大型团队也是一个很好的选择。

从React迁移到HTMX是可能的，并且可以使代码库减少[67% smaller codebase](https://htmx.org/essays/a-real-world-react-to-htmx-port/)。然而，仅建议在您不需要使React如此受欢迎的所有功能（如高级状态管理）时才这样做。

+   **HTMX**：2020年底首次发布，您不能指望HTMX像React那样受欢迎。因此，关于它的指南、教程和操作视频并不多。尽管如此，该项目已经在GitHub上获得了超过[29k starts on GitHub](https://github.com/bigskysoftware/htmx)，并且引起了很大的关注。

+   **React**：拥有数百万全球开发人员和超过[218k stars on GitHub](https://github.com/facebook/react)的React是无可争议的前端开发库冠军。根据[Statista调查](https://www.statista.com/statistics/1124699/worldwide-developer-survey-most-used-frameworks-web/)，React是目前全球使用最广泛的前端库，市场份额超过40%。因此，有数十万篇教程、文章和视频专门介绍React。

### 生态系统

+   **HTMX**：虽然该库是可扩展的，但项目相对较新，可用的HTMX库和实用程序并不多。截至本文撰写时，[htmx tag on npm](https://www.npmjs.com/search?q=keywords:htmx)仅计数35个包。

+   **React**：仅在npm上就有超过6,000个与之相关的库。这只是React相关标签之一，您可以找到成千上万其他兼容库。

## 在HTMX和React之间选择前端库时，您应该选哪一个？

和比较两种技术总是没有绝对的赢家一样。HTMX和React都是优秀的前端开发库，选择其中一个取决于项目的需求和目标。

当创建需要状态管理、提供复杂功能并需要可重复使用组件的Web应用时，React是更合适的选择。而构建具有简单交互性和无特别高级功能的网站时，HTMX可能是更好的解决方案。

要帮助您在HTMX和React之间做出明智的决定，请让我们看看这两个库的优缺点！

### HTMX：优缺点

**👍 优点**：

+   简单直观的基于HTML的语法。

+   只需几个HTML属性即可完成AJAX请求和DOM更新。

+   直接在HTML中实现动态交互，无需JavaScript。

+   可轻松集成到现有的HTML网页中。

+   轻量级库，仅几KB。

**👎 缺点**：

+   需要返回原始HTML的后端UI端点，因此更多地与前端绑定。

+   仍然相对较新。

### React：优缺点

**👍 优点**：

+   使用JSX编写的可重用组件来构建UI。

+   复杂的状态管理和支持许多其他有用的功能。

+   全球使用最广泛的前端Web库。

+   由Meta开发和维护。

+   对后端无明确意见。

**👎 缺点**：

+   学习和掌握并不容易。

+   难以集成到非基于JavaScript的项目中。

## 结论

在本文《HTMX vs React》中，您了解了HTMX是什么，它如何工作以及它与React的竞争关系。HTMX能够提供现代HTML交互性，而不引入完整的Web框架所带来的复杂性。尽管它的未来一片光明，但HTMX并不打算取代React。要更好地理解HTMX的优势所在，请查看官方网站上的[HTMX示例](https://htmx.org/examples/)列表。

相反，这两个库可以共存并针对不同的使用场景。正如在这里所学到的，React 非常适合具有丰富用户体验和复杂功能的 Web 应用程序，而HTMX则更适合需要简单交互的网页。
