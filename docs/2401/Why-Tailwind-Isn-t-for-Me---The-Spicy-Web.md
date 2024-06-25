<!--yml

category: 未分类

date: 2024-05-27 15:12:43

-->

# 为什么 Tailwind 不适合我 | The Spicy Web

> 来源：[`www.spicyweb.dev/why-tailwind-isnt-for-me/`](https://www.spicyweb.dev/why-tailwind-isnt-for-me/)
> 
> *热点新闻！*全新课程 **CSS Nouveau** 现已上线！这是 **THE SPICY WEB** 课程系列的第一版发布。**去看看并今天注册！**

**2022 年 8 月更新**：仍在致力于设计系统课程（😅），但与此同时，我已经撰写了实用类的三大法则，并宣布了[Vanilla Breeze](https://www.vanillabreeze.dev)，这是一个新的开源工具，将把 Tailwind 的“class soup”转换为干净、可移植的纯 HTML + CSS！进展中…

**2022 年 3 月更新**：嗯，JIT 现在是管理 Tailwind 输出生成的默认方式，所以很酷。不幸的是，Tailwind 的视野只在[其怪异程度令人惊叹的方向上](https://tailwindcss.com/docs/adding-custom-styles#using-arbitrary-values)增长了。我对 TW 的怀疑一直存在，事实上，我已经开始编写一个课程，*专门设计*教人们如何*远离*Tailwind，并使用当今“纯粹”CSS 的最佳实践。**敬请关注。** 😁 - JW

**2021 年 3 月更新**：[Tailwind 的试验性新 JIT（Just-In-Time）编译器](https://twitter.com/tailwindcss/status/1374675986965528576?s=21)有望缓解此处概述的一些担忧，并提供一些有趣的新优势。我还没有尝试过，但一旦我尝试过，我会形成额外的想法，并从这里链接到它们。- JW

* * *

最近我在网络上为[Tailwind CSS](https://tailwindcss.com)陷入了不止一场激烈的争论。我为此感到不骄傲。我不喜欢与任何人对立。我认为构建 Tailwind 的人都是有才华的人。但从纯技术角度来看，我简直不喜欢 Tailwind。**它不是为我而建的**。

从某种意义上说，这没关系。世界上有*很多*我永远不会使用的网络技术。这并不意味着它们不好。有很多酷炫的技术栈供选择。

但我一直遇到的问题是这种日益普遍的观点：Tailwind *是未来*（兄弟）。这是*应该做的事情*。换句话说，它有观点，并且它激发了一群狂热的信徒。同样，在某种程度上，这没问题。比如，Rails 就很有观点，我喜欢用 Rails。

但是 Tailwind 确实提出了一个挑战。我将直接引用创始人亚当·瓦森在 Tailwind 网站上突出显示的内容：

> “最佳实践”实际上并不奏效。
> 
> 我已经写了几千字来解释为什么传统的“语义类名”是 CSS 难以维护的原因，但事实上，除非你真的尝试过，否则你永远不会相信我。如果你能够克制住厌恶的冲动，给它一个机会，我真的认为你会想知道你是如何以其他方式使用 CSS 的。

挑战接受。

我试过了。我用过了。很多。我最大的一家客户要求我开发的项目是基于 React 和 Tailwind 构建的。所以无论你拿什么来对付我，你都不能指责我没有充分尝试过 Tailwind。

对我来说仍然不是我的菜。事实上，我对 Tailwind 有一些真正的担忧，而我发现极为令人沮丧的是，每当我提出这些担忧时，都会立即遭到死忠的 Tailwind 粉丝的强烈反对，他们指责我（用了很多话）只是一个该死的白痴。作为一个自 90 年代末以来一直在网络行业全职工作的程序员，这让我感到非常不舒服。

所以由于 Twitter 和 Hacker News 的评论显然不适合进行如此重要的技术对话，我现在将尝试概述 Tailwind 不适合我的真实原因。

这第一个原因是审美上的关注点，但与我将很快概述的真正技术挑战密切相关。但至少，我**讨厌**只使用实用程序 CSS 的 HTML 看起来的方式。讨厌，讨厌，讨厌。当亚当直截了当地请求我们“克制一下厌恶的冲动，给它一个机会……”时，这是一种默认的承认，以这种方式编写标记最初看起来很丑陋和奇怪——但不知何故，我们最终会“克服它”，因为好处是如此之大。

写了一年的 Tailwind，我还没有摆脱它。抱歉，伙计们！你们永远都不会让我欣赏这一点：

```
<div class="min-w-0 flex-auto space-y-0.5">
  <p class="text-lime-600 dark:text-lime-400 text-sm sm:text-base lg:text-sm xl:text-base font-semibold uppercase">
    <abbr title="Episode">Ep.</abbr> 128
  </p>
  <h2 class="text-black dark:text-white text-base sm:text-xl lg:text-base xl:text-xl font-semibold truncate">
    Scaling CSS at Heroku with Utility Classes
  </h2>
  <p class="text-gray-500 dark:text-gray-400 text-base sm:text-lg lg:text-base xl:text-lg font-medium">
    Full Stack Radio
  </p>
</div> 
```

现在我已经听到很多人在电脑屏幕前尖叫着告诉我“伙计，如果你想保持你的 HTML 干净，就使用 `@apply`！问题解决了！”嗯，这是一个潜在的解决方案，事实上，这就是我们在上述项目中所做的。我们的大部分 HTML 都是围绕组件作用域的类名（概念上与 BEM 相当接近），因此我们广泛使用 `@apply`。但这就引出了我的下一个担忧。

### 原因 2：`@apply` 从根本上是不兼容和不标准的（并且在很大程度上是不必要的）。#

这就是很多 Tailwind 粉丝遇到困难并一遍又一遍地与我争论的地方，所以我会尽可能清晰和明显地解释这一点。

1.  在 CSS 文件中使用 `@apply mt-3` *只有* 当你使用 Tailwind 时才有效。它需要在你的构建过程中存在 Tailwind。如果你从构建过程中删除 Tailwind，那么这个语句就不起作用，你的 CSS 就会出错。

1.  虽然你可以拿到一个网站的生成的输出 CSS 并且在不使用 Tailwind 的情况下使用它，但通常情况下它是一个捆绑编译的成百上千个小 CSS 文件在代码库中散布（如果你像我们一样编写 CSS-per-component 文件）。这不是你可以依赖的源代码。

1.  因此，事实上，为 Tailwind 构建的 CSS 文件是非标准的（即专有的）并且与所有其他 CSS 框架和工具不兼容。一旦你使用 Tailwind，*你就再也离不开*（da da dum 😱）。

1.  而且作为额外的奖励，到处写`@apply`的所有 CSS 文件基本上意味着你不是在学习和编写 CSS。你正在编写 Tailwind。无论你写了多少次`@apply flex`，那都**不是**写`display: flex`。

现在我意识到，我们大多数人并不习惯定期在项目中更换 CSS 框架。但相信我，我已经做过这个！我现在正在一个客户项目上，我们正在从 Foundation 迁移到 Bulma。虽然这需要更新一堆 HTML 和一些正在使用的样式表，但请放心，以前我们编写的任何自定义样式都将再次无忧无虑地工作，因为当你编写普通的 CSS（甚至是 Sass）时，无论如何都会生效。

而且，虽然`@apply`在表面上看起来很酷，但最终会变成一个巨大的支撑。例如，我喜欢 Tailwind 使用 CSS Grid 技术编写样式的方式很简单。不幸的是，尽管如此，我仍然不太理解 Grid 语法本身。我对开放的 CSS 标准一无所知。

至于为什么`@apply`在整体计划中基本上是不必要的，这就引出了我的第三点。

### 第三个原因：Tailwind 关注设计系统和标记的重点大部分可以被 CSS 自定义属性（即变量）取代—这是一个标准。#

初始时，人们喜欢 Tailwind 是因为它开箱即用，具有漂亮的设计系统和许多可以调整的标记（颜色、字体大小、间距等）。很容易快速获得好看的结果。

问题是所有这些标记都是在 JavaScript 中定义的。一个 CSS 框架。使用 JavaScript 进行设计标记。在 2021 年。

我很遗憾要告诉你，但所有现代浏览器都支持这个叫做 CSS 自定义属性的东西。你可以在`:root`级别定义一次设计标记作为变量，并**在任何地方**使用它们。你甚至可以在网站加载时实时修改它们，或在 DOM 树的特定部分中重载它们。而且它们与 Web 组件*非常*兼容。稍后再详细介绍。

例如，在 Tailwind 中，你可以写 `class="mb-8"`，然后应用 `margin-bottom: 2rem` 样式。但你猜猜你可以做什么？在你的样式表中定义 `:root { --spacing-8: 2rem }`，然后在任何你想要的地方写 `margin-bottom: var(--spacing-8)`。就是在任何地方：一个样式表，或一个 JS 组件，*甚至是* 直接在 HTML 中的 `style=` *属性！*

一旦你开始考虑如何适应响应式断点等问题，故事就会变得有些混淆，但是在这里的原则仍然是，Tailwind 在设计系统中使用了一个非标准的基于 JavaScript 的构建过程，而你可以使用所有现代浏览器 *本地* 支持的语法构建设计系统。

谈到现代 Web 浏览器中的本地功能...

### 第四个原因：Tailwind 忘记了 Web 组件的存在。#

这可能是针对 Tailwind 的最大批评。它似乎是在一个 Web 组件不存在的世界中构思和推广的。Tailwind CSS 在 Shadow DOM 中完全无法使用。一些有远见的开发人员提出了解决方案，通过构建过程将一些 Tailwind 样式注入到组件中，但这绝对是一种妥协。

与此同时，今天有多种构建基于 Web 组件的设计系统的方法，全局主题和通过 Shadow DOM（以及暴露的部件）进行的组件样式化都能够一起工作。同样，你可以基于所有现代浏览器内置且本地支持的技术进行所有这些操作。在你耸耸肩膀，回到 React 或 Vue 之前，请记住，Web 组件不仅是今天 HTML/CSS/JS 规范的一个组成部分，而且越来越多地成为浏览器技术进一步发展的核心（例如，未来如何工作的高级表单控件定制）。

在这方面，Tailwind 对你的帮助不比 Bootstrap 或 Foundation 或任何其他几年/几十年前编写的 CSS 框架更多。（甚至是我心爱的 Bulma！ 😢）

### 第五个原因：最后，Tailwind 鼓励使用 div/span 标记汤。#

我几乎把这个包含在前一个观点中，但它确实需要单独的讨论。我现在已经相信，到处在你的标记中使用 `<div>` 和 `<span>` 标签是一种反模式。我们生活在一个现代浏览器完全支持和启用自定义元素（又称 `<whatever-you-can-dream-of>`）的世界。当你可以编写 `<ui-card></ui-card>` 时，没有什么理由你被迫写 `<div class="card"></div>`。事实上，你完全可以在元素中使用自定义属性来编写 *极具表现力的标记*，与过去相比，这种标记看起来非常 futurist！

以 [Shoelace](https://shoelace.style) Web 组件库为例。这是一个按钮：

```
<sl-button type="default" size="small">
  <sl-icon slot="prefix" name="gear"></sl-icon>
  Settings
</sl-button> 
```

这是一个模态窗口：

```
<sl-dialog label="Dialog" style="--width: 50vw;">
  Lorem ipsum dolor sit amet, consectetur adipiscing elit.
  <sl-button slot="footer" type="primary">Close</sl-button>
</sl-dialog> 
```

请注意，这不是 JSX。这不是 XML。这也不是某种你需要将其转换为普通 HTML 的花哨模板语言。

**这是 HTML。**这就是现代标记语言的样子。

将其与 Tailwind 官方网站上的一个示例进行比较：

```
<button class="hover:bg-light-blue-200 hover:text-light-blue-800 group flex items-center rounded-md bg-light-blue-100 text-light-blue-600 text-sm font-medium px-4 py-2">
  New
</button> 
```

呃…… 🤢

有一个未来的 HTML/CSS/JS 世界（在很大程度上它已经存在了），在那里你可以使用原生 CSS 快速轻松地编写定制的网格/弹性布局，使用 CSS 变量设置设计令牌，利用像 Shoelace 这样的精心构建的 Web 组件库（甚至混合两三个），最终获得一个**看起来很棒**且功能相当不错的网站/应用程序——而不需要 *任何* Tailwind 的大量实用程序类，然后需要清除它们以将性能降至可管理的水平。

换句话说，Tailwind 的主要卖点（除了通过实用程序类进行快速原型设计之外）是其具有吸引人的设计系统——但它实现该设计系统的方式实际上有点糟糕！（默认情况下与 Web 组件不兼容，仅最小程度地利用 CSS 变量，不鼓励使用相关的作用域化样式的自定义元素/属性……）

这就引出了一个问题：Tailwind 究竟能让我们如何“构建现代网站”？从纯技术层面上讲，我真的不认为它比 Bootstrap 好多少。而 Bootstrap 至少提供了一个免费的开源组件库。如果你使用 Tailwind，[他们会让你付费](https://tailwindui.com)。

### 结论：如果你喜欢 Tailwind，就使用它吧！但不要试图说服我它是未来。#

听着，我们可以就任何技术的相对优点或问题来回讨论。选择 Tailwind 明显有一些好处，最值得注意的是你可以通过简单地用实用程序类添加一堆 div 标签，快速从空白页面转变为花哨的设计。

但是在与 Tailwind 一年多的经验之后，并将其与其他处理 HTML、样式和基于组件的 Web 开发方法进行权衡后，我完全相信 Tailwind 并不代表我希望看到 Web 发展的方向。对所有 Tailwind 的粉丝表示歉意，但你们没有令我改变看法的令人信服的论点。

这就是为什么 Tailwind 不适合我。你的情况可能不同。🙃
