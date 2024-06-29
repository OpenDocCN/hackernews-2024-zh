<!--yml

category: 未分类

date: 2024-05-27 12:56:14

-->

# Angular和Wiz共同更好 | 由Angular撰写 | 2024年3月 | Angular博客

> 来源：[https://blog.angular.io/angular-and-wiz-are-better-together-91e633d8cd5a?gi=a136e0cac466](https://blog.angular.io/angular-and-wiz-are-better-together-91e633d8cd5a?gi=a136e0cac466)

# Angular和Wiz共同更好

*作者：* [*贾廷·拉马纳桑*](https://x.com/JatinRamanathan)*，* [*明科·格切夫*](https://x.com/mgechev)

您可能知道Angular是谷歌的Web框架，但实际上谷歌还有另一个Web框架：Wiz。Angular和Wiz都被成千上万的工程师以及谷歌内部的数千个应用程序所使用。Wiz是一个内部框架，被谷歌的一些最受欢迎的产品使用，例如[搜索](https://google.com)，[照片](https://photos.google.com/)，[支付](https://payments.google.com/)等等。在过去的一年里，我们一直在探索Angular如何从Wiz的性能中受益，以及Wiz如何从Angular的开发者体验中受益。

历史上，Angular和Wiz一直在为不同的应用程序段提供服务：

+   Wiz专注于性能关键的应用程序。其中一个很好的例子是Google搜索，其目标是尽快渲染结果并具有相对较低的互动性。

+   Angular一直专注于提供高度交互式的应用程序，优先考虑开发者体验和快速交付复杂的UI。好的例子包括诸如[Gemini](https://gemini.google.com/)和Google Analytics之类的应用程序。

Angular和Wiz功能融合

# 什么是Wiz？

数百万用户通过慢网络和/或低端设备访问大型谷歌应用程序。在这种情况下，初始加载延迟和JavaScript的量至关重要。Wiz框架以多种方式满足这些要求。Wiz始终从服务器端渲染开始。页面上的所有内容，包括交互组件，都是通过高度优化的流式解决方案进行渲染的。这消除了大部分JavaScript在关键的初始渲染路径上。为了避免加载过多的JavaScript，Wiz仅加载页面上实际渲染的交互组件所需的代码。为了避免在客户端丢失用户事件，一个小型的内联库监听根元素上的用户事件并重播它们。这种以SSR为先的新颖方法对于最终用户的性能效果最佳，但这也带来了开发人员复杂度增加的折衷，特别是对于高度交互式应用程序。

# 混合要求

最近，我们看到这两个不同领域的融合。高性能应用程序需要更快地发布更多功能，以为用户提供价值并保持他们的参与。与此同时，高交互应用程序开始越来越多地发布 JavaScript。根据[HTTPArchive](https://httparchive.org/reports/state-of-javascript?lens=top1m&start=2018_06_01&end=latest&view=list)，过去6年中，桌面端 JavaScript 增长超过37%，移动端增长超过36%，这显著影响了性能。

基于 HTTPArchive 的顶级1m网站中 JavaScript 的增加

随着这些混合需求的增加，开发者很难决定哪个框架更适合他们的需求，我们开始看到使用案例的更大重叠。为了满足对高性能框架和出色开发者体验不断增长的需求，Angular 和 Wiz 共同努力，将两者的优势结合起来。未来，Angular 开发者将不再需要在开发者体验和性能之间做出选择。

# 将两个世界融合在一起

**Angular 和 Wiz 的合作**展示了我们使开发者能够自信构建 Web 应用的使命。根据我们收到的开发者反馈，我们寻找机会将我们在 Google 发现的最佳 Web 开发实践开源化。与此同时，我们希望将我们在 Angular 社区中积累的出色开发者体验带给整个 Google。

在实践中，这表现为对每个框架的增量和渐进改进。您可能已经看到我们与 Angular 最新变化的合作成果，比如[可推迟视图](https://angular.dev/guide/defer#why-use-deferrable-views)和我们对部分水合的探索。它们都受到 Wiz 精细化代码加载和事件委托库的极大启发。

与此同时，Wiz 采用了 Angular 的 Signals 库，现在驱动 YouTube 的用户界面，在数十亿设备上运行。Angular Signals 允许 Wiz 采用精细化的 UI 更新。我们放弃了依赖开发者仔细记忆每个 UI 更新运行路径的方法。这带来了显著的性能改进。要了解更多，请参阅 ng-conf 的[keynote](https://www.youtube.com/watch?v=nIBseTi6RVk&t=2455s)。我们很高兴看到 Angular Signals 如何改进全球最大的网站，如搜索和 GMail。

# 我们从这里往哪里走？

我们的长期目标是在未来几年逐步而负责任地[合并 Angular 和 Wiz](https://twitter.com/sarah_edo/status/1770478763253379488)。我们的策略是通过 Angular 逐步开源 Wiz 的功能，并遵循我们的开发开放模式，允许社区影响路线图并做出相应计划。我们将使用公开的 RFC 过程确保我们收集社区对相关提议功能的反馈。主要目标是改进 Angular 框架。

我们相信服务器端渲染（SSR）对于 Web 平台非常重要。我们在构建全球一些最常用的 Web 产品的经验告诉我们，当正确实施时，SSR 对最终用户体验有积极的影响。我们希望邀请社区使用支持 Google 搜索和 YouTube 等应用的关键库进行创新。

我们很兴奋地一起创新，以改善每个人的 Web 性能！
