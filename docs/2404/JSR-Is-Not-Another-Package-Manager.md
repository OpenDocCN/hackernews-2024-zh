<!--yml

类别: 未分类

日期：2024-05-27 13:32:21

-->

# JSR不是另一个包管理器

> 来源：[https://deno.com/blog/jsr-is-not-another-package-manager](https://deno.com/blog/jsr-is-not-another-package-manager)

在过去几年中，出现了像yarn和pnpm这样的新包管理器，改善了包的下载方式。然而，JavaScript生态系统的核心npm包注册表几乎没有发展。其最后一个显著更新是几年前添加的“files”标签。作为一个以活跃进化著称的语言，JavaScript似乎矛盾地陷入了一个未能跟上步伐的分发模型中。

在我创造Node时，JavaScript并没有标准的模块系统。因此，npm注册表和Node默认使用了CommonJS（`require`），这是一个存在基本缺陷、在浏览器中无法使用的系统。因此，将近十年前的2015年，语言采用了ES模块的语法（`import`）。如今，大多数JavaScript都是使用ES模块编写的，然而在涉及TypeScript时，这些模块的分发路径仍然复杂。这个生态系统中的明显差距促使了[JSR](https://jsr.io)的创建，它不是另一个包管理器，而是一个旨在彻底改变JavaScript和TypeScript在服务器端运行时、浏览器和各种工具中共享方式的革新性注册表。

JSR通过简化长期困扰开发者的复杂性，根本改进了代码分发过程。作为仅支持ESM和以TypeScript为先的工具，JSR消除了让人沮丧的`package.json`配置和复杂的tsconfig编译器选项之间的纠缠。通过[包评分系统](https://jsr.io/docs/scoring)，JSR推动了代码分发中的最佳实践 —— 高分将授予那些在每个导出符号上包含全面JSDoc文档的包，这类似于Dart社区在[pub.dev](http://pub.dev)上的做法。正如在Go和Rust等现代编程生态系统中看到的那样，JSR提供了开箱即用的自动文档生成功能。

JSR是一个注册表，而不是npm注册表的另一个客户端。但这并不意味着你需要放弃npm的所有东西，或者完全切换到不同的JavaScript模块生态系统。[JSR旨在与npm注册表](https://jsr.io/docs/npm-compatibility)互补，而不是替代它。JSR包允许依赖于npm包 - [例如，查看此包](https://jsr.io/@moductor/libintl@1.5.0/dependencies)。此外，JSR包可以在现有的以npm为先的软件中使用，因为JSR本身充当npm注册表（可在npm.jsr.io访问），分发与npm兼容的tarballs。这使得JSR包可以包含在任何使用npm、yarn或pnpm的软件中，并与[私有注册表](https://jsr.io/docs/private-registries)集成。JSR分发的npm tarballs是最优的。

在 Deno，我们将安全性作为 JavaScript 开发中的首要关注点。虽然没有注册表能够全面监管所有发布的代码，但 JSR 提供了关于其发布者的透明度，并保护了发布过程。通过将 OIDC 令牌与 GitHub Actions 集成，JSR 利用 [软件工件供应链级别](https://slsa.dev/) 在 [Sigstore](https://www.sigstore.dev/) 中创建高级、可验证的源证书。这不仅确保了代码的真实性，还建立了开发者在实施什么的信任和责任。

JavaScript 是许多程序员共同使用的通用语言，使其既普遍又易于接触。该语言拥有一个核心中心——一个城镇广场——在这里开发者可以分享他们的工作，而不会有过多的复杂性。我们相信 JavaScript 将在未来的许多年中继续在软件开发中占据核心地位，而 JSR 旨在支持这种持久的重要性。虽然 JSR 不是一个包管理器，但它提供了一种新的方法来管理和保护代码，旨在成为一个稳定、前瞻性的平台，增强并保护 JavaScript 开发。因此，JSR 不仅代表生态系统中的又一个工具，而且是我们思考如何分发 JavaScript 和 TypeScript 的根本转变。

> 🚨️ 了解更多关于 JSR 的信息 🚨️
