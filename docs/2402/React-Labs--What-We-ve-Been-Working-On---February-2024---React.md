<!--yml

category: 未分类

date: 2024-05-29 13:25:47

-->

# React Labs: What We've Been Working On – February 2024 – React

> 来源：[https://react.dev/blog/2024/02/15/react-labs-what-we-have-been-working-on-february-2024](https://react.dev/blog/2024/02/15/react-labs-what-we-have-been-working-on-february-2024)

2024 年 2 月 15 日，由[Joseph Savona](https://twitter.com/en_JS)、[Ricky Hanlon](https://twitter.com/rickhanlonii)、[Andrew Clark](https://twitter.com/acdlite)、[Matt Carroll](https://twitter.com/mattcarrollcode) 和[Dan Abramov](https://twitter.com/dan_abramov)撰写。

* * *

在 React Labs 帖子中，我们介绍了正在积极研究和开发的项目。自我们[上次更新](/blog/2023/03/22/react-labs-what-we-have-been-working-on-march-2023)以来，我们取得了显著进展，现在我们想分享我们的进展情况。

### 注意

React Conf 2024 将于 5 月 15 日至 16 日在内华达州亨德森举行！如果你有兴趣亲临参加 React Conf，请在 2 月 28 日前[注册抽奖领取门票](https://forms.reform.app/bLaLeE/react-conf-2024-ticket-lottery/1aRQLK)。

关于门票、免费直播、赞助等更多信息，请参阅[React Conf 网站](https://conf.react.dev)。

* * *

## React Compiler

React 编译器不再是一个研究项目：该编译器现在已经在生产中为 instagram.com 提供支持，我们正在努力将其推广到 Meta 的其他平台，并准备发布第一个开源版本。

正如我们在[之前的帖子](/blog/2023/03/22/react-labs-what-we-have-been-working-on-march-2023#react-optimizing-compiler)中讨论的，当状态变化时，React 有时会重新渲染得太频繁。自 React 早期以来，我们的解决方案是手动记忆化。在我们当前的 API 中，这意味着通过[`useMemo`](/reference/react/useMemo)、[`useCallback`](/reference/react/useCallback)和[`memo`](/reference/react/memo) API 手动调整 React 在状态更改时重新渲染的频率。但手动记忆化是一种妥协。它使我们的代码变得混乱，容易出错，并且需要额外的工作来保持更新。

手动记忆化是一种合理的妥协，但我们并不满意。我们的愿景是，当状态变化时，React 能够*自动*重新渲染 UI 的恰当部分，*而不会影响到 React 的核心思维模型*。我们认为 React 的方法——将 UI 视为状态的简单函数，使用标准的 JavaScript 值和习惯用法——正是为什么 React 能够让如此多的开发者易于接近的关键部分。因此，我们投入精力构建了一个用于优化的 React 编译器。

JavaScript 因其松散的规则和动态的特性而被公认为一门极具挑战性的语言。React 编译器通过同时模拟 JavaScript 的规则和“React 的规则”，能够安全地编译代码。例如，React 组件必须是幂等的——在相同输入条件下返回相同的值——且不能改变 props 或 state 的值。这些规则限制了开发者的行为，帮助确立编译器优化的安全空间。

当然，我们理解开发者有时会在一定程度上打破规则，我们的目标是使 React 编译器能够在尽可能多的代码上开箱即用。编译器试图检测代码是否严格遵循 React 的规则，并会在安全时编译代码，否则将跳过编译。我们正在针对 Meta 的大规模和多样化的代码库进行测试，以验证这一方法的有效性。

对于希望确保其代码遵循 React 规则的开发者，我们建议[启用严格模式](/reference/react/StrictMode)和[配置 React 的 ESLint 插件](/learn/editor-setup#linting)。这些工具有助于捕捉 React 代码中的微妙错误，提升应用程序的质量，并未即将推出的功能（如 React 编译器）未来保护您的应用。

欲亲眼目睹编译器的运作，请查看我们去年秋季的[演讲](https://www.youtube.com/watch?v=qOQClO3g8-Y)。在演讲时，我们展示了将 React 编译器尝试应用在 Instagram.com 的一个页面上的初步实验数据。此后，我们已将编译器在 Instagram.com 上投入生产。我们还扩展了团队规模，以加速在 Meta 的其他平台上和开源项目中的推广。展望未来，我们对此充满期待，并将在接下来的几个月内分享更多进展。

## 动作

我们之前分享过，我们正在探索通过服务器操作将数据从客户端发送到服务器的解决方案，以便您可以执行数据库变更和实现表单。在服务器操作的开发过程中，我们扩展了这些 API，以支持仅客户端应用程序中的数据处理。

我们将这些更广泛的功能集合简称为“动作”。动作允许您将函数传递给 DOM 元素，例如[`<form/>`](/reference/react-dom/components/form)：

```
 <form action={search}>

  <input name="query" />

  <button type="submit">Search</button>

</form> 
```

`action`函数可以同步或异步操作。您可以使用标准JavaScript在客户端定义它们，或者在服务器上使用[`'use server'`](/reference/rsc/use-server)指令。在使用操作时，React将为您管理数据提交的生命周期，提供像[`useFormStatus`](/reference/react-dom/hooks/useFormStatus)和[`useActionState`](/reference/react/useActionState)这样的钩子，以访问表单操作的当前状态和响应。

默认情况下，Actions在[transition](/reference/react/useTransition)中提交，使当前页面在处理操作时保持交互。由于Actions支持异步函数，我们还添加了在转换中使用`async/await`的功能。这允许您在像`fetch`开始的异步请求期间，通过转换的`isPending`状态显示挂起UI，并在整个更新过程中显示挂起UI。

除了Actions之外，我们还引入了一个名为[`useOptimistic`](/reference/react/useOptimistic)的功能，用于管理乐观状态更新。通过这个钩子，您可以应用临时更新，一旦最终状态提交，这些更新就会自动回滚。对于Actions，这使您可以在客户端乐观地设置数据的最终状态，假设提交成功，然后恢复为从服务器接收到的数据值。它使用常规的`async`/`await`工作，因此无论您是在客户端使用`fetch`，还是在服务器上使用Server Action，它都能正常工作。

库作者可以在他们自己的组件中使用`useTransition`实现自定义的`action={fn}`属性。我们的意图是，当设计他们的组件API时，库应采用Actions模式，为React开发人员提供一致的体验。例如，如果您的库提供了一个`<Calendar onSelect={eventHandler}>`组件，还考虑公开一个`<Calendar selectAction={action}>`API。

虽然最初我们专注于用于客户端-服务器数据传输的Server Actions，但我们对React的理念是在所有平台和环境中提供相同的编程模型。在可能的情况下，如果我们在客户端引入一个功能，我们的目标是使其在服务器上也能工作，反之亦然。这一理念使我们能够创建一套适用于应用程序在任何地方运行的API，从而使将来升级到不同环境变得更容易。

现在可以在金丝雀通道中使用**Actions**，并将在下一个React版本中发布。

## React金丝雀中的新功能

我们引入了[React Canaries](/blog/2023/05/03/react-canaries)作为选择，以在设计接近最终状态时，立即采纳各个新的稳定特性，而不是在稳定的semver版本中发布之前。

变更为React开发带来了一个新的阶段。以前，功能会在Meta内部进行研究和私下构建，所以用户只能在稳定版发布时看到最终成品。现在有了Canaries，我们在公共场合与社区的帮助下构建，以完成我们在React实验室博客系列中分享的功能。这意味着您能更早地了解新功能，因为它们在完成后而不是完全完成时即已听到。

React服务器组件、资源加载、文档元数据和Actions已经都进入了React Canary，并且我们在react.dev上为这些功能添加了文档。

+   **指令**: [`"use client"`](/reference/rsc/use-client) 和 [`"use server"`](/reference/rsc/use-server) 是为全栈React框架设计的打包器功能。它们标记了两个环境之间的“分割点”： `"use client"` 指示打包器生成 `<script>` 标签（如[Astro Islands](https://docs.astro.build/en/concepts/islands/#creating-an-island)）；而 `"use server"` 则告诉打包器生成一个POST端点（如[tRPC Mutations](https://trpc.io/docs/concepts)）。它们让您编写可重用组件，将客户端交互与相关的服务器端逻辑组合在一起。

+   **文档元数据**: 我们增加了对在组件树中任何位置渲染的内置支持[`<title>`](/reference/react-dom/components/title)，[`<meta>`](/reference/react-dom/components/meta)，以及元数据[`<link>`](/reference/react-dom/components/link)标签。这些在所有环境中都以相同方式工作，包括完全客户端代码、SSR和RSC。这提供了对像[React Helmet](https://github.com/nfl/react-helmet)等库开发先进特性的内置支持。

+   **资源加载**: 我们将Suspense与样式表、字体和脚本等资源的加载生命周期集成在一起，以便React考虑它们来确定是否准备好显示像[`<style>`](/reference/react-dom/components/style)，[`<link>`](/reference/react-dom/components/link)和[`<script>`](/reference/react-dom/components/script)中的内容。我们还添加了新的[资源加载APIs](/reference/react-dom#resource-preloading-apis)，如`preload`和`preinit`，以提供更大的控制权，以便在何时加载和初始化资源。

+   **Actions**: 如上所述，我们添加了Actions以管理从客户端向服务器发送数据。您可以为[`<form/>`](/reference/react-dom/components/form)等元素添加`action`，使用[`useFormStatus`](/reference/react-dom/hooks/useFormStatus)访问状态，使用[`useActionState`](/reference/react/useActionState)处理结果，并使用[`useOptimistic`](/reference/react/useOptimistic)进行乐观更新UI。

由于所有这些功能是协同工作的，单独在稳定通道中发布它们是困难的。发布Actions而没有用于访问表单状态的补充钩子将限制Actions的实际可用性。引入React Server Components而没有集成Server Actions将复杂化对服务器上数据的修改。

在我们将一组功能发布到稳定通道之前，我们需要确保它们能够协同工作，并且开发人员拥有一切必需的东西来在生产环境中使用它们。React Canaries允许我们单独开发这些功能，并逐步发布稳定的API，直到整个功能集完成。

React Canary中的当前功能集已经完整且准备好发布。

## React的下一个主要版本

经过几年的迭代，`react@canary`现在已经准备好发布到`react@latest`。上述新功能兼容您的应用程序运行的任何环境，提供了生产使用所需的一切。由于资源加载和文档元数据可能对某些应用程序造成破坏性影响，React的下一个版本将是一个主要版本：**React 19**。

还有更多工作要做以准备发布。在React 19中，我们还在添加长期请求的改进，这些改进需要支持Web组件等破坏性变更。我们现在的重点是实施这些变更，准备发布，完成新功能的文档，以及发布包含内容的公告。

在接下来的几个月中，我们将分享有关React 19包含的所有内容、如何采用新客户端功能以及如何为React Server Components构建支持的更多信息。

## 活动（从Offscreen改名为Activity）。

自我们上次更新以来，我们已经将我们正在研究的一个能力从“Offscreen”改名为“Activity”。名称“Offscreen”暗示它仅适用于应用程序中不可见的部分，但在研究该功能时，我们意识到应用程序的某些部分可能是可见但不活动的，例如模态框后面的内容。新名称更贴近标记应用程序某些部分为“活动”或“非活动”的行为。

Activity仍在研究中，我们剩下的工作是最终确定暴露给库开发人员的基本元素。在我们专注于发布更完整功能时，我们已经降低了这一领域的优先级。

* * *

除了这次更新之外，我们的团队还在会议上做了演讲，并在播客上露面，更详细地讲述了我们的工作并回答了问题。

感谢[Lauren Tan](https://twitter.com/potetotes)、[Sophie Alpert](https://twitter.com/sophiebits)、[Jason Bonta](https://threads.net/someextent)、[Eli White](https://twitter.com/Eli_White)和[Sathya Gunasekaran](https://twitter.com/_gsathya)审阅本篇文章。

感谢您的阅读，并[期待React Conf见面](https://conf.react.dev/)！
