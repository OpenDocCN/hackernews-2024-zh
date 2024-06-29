<!--yml

分类：未分类

日期：2024-05-27 13:07:55

-->

# 引入Enhance WASM — Begin Blog

> 来源：[https://begin.com/blog/posts/2024-04-08-introducing-enhance-wasm](https://begin.com/blog/posts/2024-04-08-introducing-enhance-wasm)

## 后端无关的服务器端渲染（SSR）用于Web组件

Web组件是扩展HTML的浏览器本地方法。但作为主要基于浏览器的技术，它们是用JavaScript定义的，这限制了它们要么仅在客户端进行渲染 —— 这会导致性能不稳定、SEO效果差，并且可访问性不佳 —— 要么在服务器端JavaScript运行时内渲染，而这并不总是适用于使用其他后端运行时的商店。

**[Enhance WASM](https://enhance.dev/wasm)**为任何后端运行时解锁**服务器端渲染的Web组件**。

+   **一次编写，随处渲染**：编写标准的Web组件，并在任何后端部署它们。**[Enhance WASM](https://enhance.dev/wasm)**负责在任何服务器环境下进行渲染。

+   **无缝集成**：轻松将**[Enhance WASM](https://enhance.dev/wasm)**集成到您现有的项目中，只需进行最少的设置。详尽的文档、高质量的基线组件和广泛的社区支持使采用变得轻松。

+   **更好的开发体验（DX）**：别再浪费时间在缓慢的构建步骤和重新实现已经工作的脆弱前端代码上了。Web标准确保了可靠性和性能，无需重写。

+   **性能优化**：不再有旋转图标或骨架屏。享受更快的加载时间和改进的SEO效果。您的用户获得更快的体验，您获得更好的开发速度。

我们真的相信这是前端开发的一个飞跃时刻。服务器端渲染是个性化Web应用的关键要求。那些优先考虑Web标准稳定性、性能和可访问性的组织，在各种后端运行时中运行工作负载。**现在我们可以构建跨越运行时差距的浏览器本地Web界面**。

## 尝试一下，参与其中！

Enhance是完全开源的代码，我们需要您的帮助！我们立即开放Enhance WASM，支持**[Node](https://github.com/enhance-dev/enhance-ssr)**、**[Deno](https://github.com/enhance-dev/enhance-ssr-deno)**、**[Python](https://github.com/enhance-dev/enhance-ssr-python)**、**[Ruby](https://github.com/enhance-dev/enhance-ssr-ruby)**、**[PHP](https://github.com/enhance-dev/enhance-ssr-php)**、**[Java](https://github.com/enhance-dev/enhance-ssr-java)**、**[C#](https://github.com/enhance-dev/enhance-ssr-c-sharp)**、**[Rust](https://github.com/enhance-dev/enhance-ssr-rust)**和**[Go](https://github.com/enhance-dev/enhance-ssr-go)**等运行时。我们需要您的帮助来测试和实施对这些以及其他运行时的支持！如果您想看到未在此处提及的运行时，请告诉我们[请告诉我们](https://github.com/enhance-dev/enhance-ssr/issues)。

示例起始项目：

## 常见问题

*组件如何编写？*

组件的编写完全遵循 Web 组件规范。Enhance WASM 还支持更面向后端的纯函数式风格，并提供几个辅助库，使常见的客户端模式更清晰。

*我需要运行客户端 JS 来渲染组件吗？*

不需要。Web 组件完全在服务器端渲染 HTML。然后，如果元素需要，可以选择在客户端进行升级。值得注意的是，我们发现大多数元素只需在服务器端渲染，根本不需要任何客户端 JavaScript。

*我可以在这些组件中使用客户端 JS 吗？*

当然可以！

*那 Lit 呢？*

我们绝对听到社区中的人们说，“只需使用 Lit”，但根据 Lit 的文档，他们自己的 SSR 渲染器今天还没有准备好投入生产，并且即使准备好了也只支持 Node.js。Lit 最终更专注于客户端升级，而不是服务器端的使用，这完全没问题！Enhance 鼓励 SSR 和“HTML 优先”，将客户端元素升级视为渐进增强的一步。如果对你的项目有意义，你可以使用 Enhance 渲染初始 HTML，然后用 Lit 处理客户端交互。我们发现大多数元素实际上并不需要客户端交互。
