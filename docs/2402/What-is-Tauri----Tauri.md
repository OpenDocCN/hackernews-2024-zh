<!--yml

category: 未分类

date: 2024-05-29 13:18:57

-->

# 什么是 Tauri？ | Tauri

> 来源：[https://beta.tauri.app/guides/](https://beta.tauri.app/guides/)

Tauri 是一个为所有主要桌面和移动平台构建微小、快速二进制文件的框架。开发者可以集成任何编译为 HTML、JavaScript 和 CSS 的前端框架，以构建他们的用户体验，同时在需要时利用 Rust、Swift 和 Kotlin 等语言进行后端逻辑。

开始使用[`create-tauri-app`](https://github.com/tauri-apps/create-tauri-app)通过以下命令之一构建。确保遵循[Tauri 的先决条件指南](/start/prerequisites/)安装所有 Tauri 需要的依赖项，然后查看[前端配置指南](/start/frontend/)以获取推荐的前端配置。

<starlight-tabs class="astro-mkxqpaln">```
 sh  <(curl https://create.tauri.app/sh)  --beta 
```

```
 $env:CTA_ARGS="--beta"; irm https://create.tauri.app/ps | iex 
```

```
 npm  create  tauri-app@latest  --  --beta 
```

```
 yarn  create  tauri-app  --beta 
```

```
 pnpm  create  tauri-app  --beta 
```

```
 bun  create  tauri-app  --beta 
```

```
 cargo  install  create-tauri-app

cargo  create-tauri-app  --beta 
```</starlight-tabs>

在创建了您的第一个应用程序之后，您可以在[功能和配方列表](/plugin/)中探索 Tauri 的不同功能和配方。

Tauri有三个主要优势供开发者构建：

+   用于构建应用程序的安全基础

+   通过使用系统的原生 Web 视图实现更小的捆绑包大小

+   灵活性使开发者可以使用任何前端和多种语言的绑定

在[Tauri 1.0 博客文章](/blog/tauri-1-0/)中了解更多有关 Tauri 哲学的信息。

通过使用 Rust 构建，Tauri 能够利用 Rust 提供的内存、线程和类型安全性优势。即使不需要由 Rust 专家开发，基于 Tauri 构建的应用程序也可以自动获得这些好处。

Tauri 还进行了针对主要和次要版本的安全审计。这不仅覆盖了 Tauri 组织中的代码，还包括 Tauri 依赖的上游依赖项。当然，这并不能消除所有风险，但为开发者构建应用程序提供了坚实的基础。

阅读[Tauri 安全策略](https://github.com/tauri-apps/tauri/security/policy)和[Tauri 1.0 审计报告](https://github.com/tauri-apps/tauri/blob/dev/audits/Radically_Open_Security-v1-report.pdf)。

Tauri 应用程序利用每个用户系统上已有的 Web 视图。一个 Tauri 应用程序仅包含特定于该应用程序的代码和资产，无需为每个应用程序捆绑浏览器引擎。这意味着最小的 Tauri 应用程序可以小于 600KB。

了解更多有关在[应用大小概念](/concept/size/)中创建优化应用程序的信息。

由于 Tauri 使用 Web 技术，这意味着几乎任何前端框架都与 Tauri 兼容。[前端配置指南](/start/frontend/)包含流行前端框架的常见配置，而[渲染概念](/concept/rendering/)讨论了与 Tauri 最佳配合的渲染技术（如单页应用程序和静态站点生成）。

使用 JavaScript 中的 `invoke` 函数和 Swift 及 Kotlin 绑定的开发者可以使用[Tauri 插件](/develop/plugins/)。

[TAO](https://github.com/tauri-apps/tao)负责 Tauri 窗口的创建，而[WRY](https://github.com/tauri-apps/wry)负责 Web 视图的渲染。这些是由 Tauri 维护的库，如果需要在 Tauri 所暴露的范围之外进行更深入的系统集成，可以直接使用。

此外，Tauri 维护了许多插件，以扩展核心 Tauri 的功能。您可以在[插件部分](/plugin/)找到这些插件，以及社区提供的插件。

© 2024 Tauri 贡献者。CC-BY / MIT
