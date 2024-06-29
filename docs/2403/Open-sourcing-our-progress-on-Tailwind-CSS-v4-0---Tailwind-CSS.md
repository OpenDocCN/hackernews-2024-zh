<!--yml

category: 未分类

date: 2024-05-27 14:39:50

-->

# 我们在 Tailwind CSS v4.0 上的开源进展 - Tailwind CSS

> 来源：[https://tailwindcss.com/blog/tailwindcss-v4-alpha](https://tailwindcss.com/blog/tailwindcss-v4-alpha)

上个夏天在 Tailwind Connect 大会上，[我分享了 Oxide 的预览](https://www.youtube.com/watch?v=CLkxRnRQtDE&t=2146s) — 这是 Tailwind CSS 的一个新的高性能引擎，我们一直在开发它，旨在简化开发者的体验，并充分利用近年来 Web 平台的发展。

最初新引擎将作为 v3.x 发布，但尽管我们致力于向后兼容性，但这显然是框架的一个新世代，因此它应该是 v4.0。

目前还处于早期阶段，我们有[很多工作要做](/blog/tailwindcss-v4-alpha#roadmap-to-v4-0)，但今天我们[公开了我们的进展](https://github.com/tailwindlabs/tailwindcss/tree/next)，并标记了第一个公开的 [v4.0.0-alpha](https://www.npmjs.com/package/tailwindcss/v/4.0.0-alpha.3)，以便您开始尝试并帮助我们在今年晚些时候稳定发布。

为了保留一些稳定版本的期待感，我将尽量简短，但如果您喜欢尝试非常早期和实验性的东西，这里应该有足够的信息让您开始。

* * *

这个新引擎是从头开始重写的，利用我们现在对框架的全面了解来更好地建模问题空间，通过减少代码量，使得系统更快。

+   **高达 10 倍的速度提升** — 我们可以将 Tailwind CSS 网站的完整构建时间从 960ms 缩短至 105ms，或将我们的 Catalyst UI kit 的构建时间从 341ms 缩短至 55ms。

+   **更小的足迹** — 这个新引擎的安装包体积减少了超过 35%，即使包含了我们重写为 Rust 和 Lightning CSS 的更重的原生包。

+   **Rust 处理关键部分** — 我们已将框架中一些最昂贵且可并行化的部分迁移到了 Rust，同时保持框架核心在 TypeScript 中以实现可扩展性。

+   **仅一个依赖项** — 这个新引擎仅依赖于 Lightning CSS。

+   **自定义解析器** — 我们编写了自己的 CSS 解析器，并设计了符合我们需求的数据结构，使得我们的解析速度比使用 PostCSS 时快了两倍以上。

* * *

Tailwind CSS v4 不再只是一个插件 — 它是一个处理您的 CSS 的一体化工具。我们直接将[Lightning CSS](https://lightningcss.dev/)集成到框架中，因此您无需配置任何关于 CSS 流水线的内容。

+   **内置 `@import` 处理** — 无需设置和配置诸如 `postcss-import` 的工具。

+   **内置供应商前缀** — 您不再需要将`autoprefixer`添加到您的项目中。

+   **内置嵌套支持** — 无需插件即可展开嵌套的 CSS，它能直接使用。

+   **语法转换** — 现代 CSS 功能如 `oklch()` 颜色和媒体查询范围会被转译为更好支持的语法。

我们仍然提供一个 PostCSS 插件，但我们也在探索第一方打包器插件，并且在这个首个 alpha 版本中，我们还提供了一个官方的 Vite 插件供您今天试用。

* * *

我们正在展望 Tailwind CSS v4 的未来，并努力构建一个在未来几年内仍然具有前沿感的框架。

+   **本地级联层** — 我们现在真正使用 `@layer` 规则，解决了我们过去曾经苦苦挣扎的许多特异性问题。

+   **显式定义的自定义属性** — 我们使用 `@property` 来定义我们的内部自定义属性，包括正确的类型和约束条件，使得可以执行诸如过渡背景渐变等操作。

+   **使用 `color-mix` 来调整不透明度** — 现在使用 CSS 变量时，使用我们的不透明度修改器语法变得比以往更加简单，甚至可以调整 `currentColor` 的不透明度。

+   **核心中的容器查询** — 我们直接在核心中增加了对容器查询的支持，引入了新的 `@min-*` 和 `@max-*` 变体以支持容器查询范围。

我们还在努力更新我们的色彩调色板，使用广色域色彩，并支持其他现代 CSS 功能，如 `@starting-style`、锚点定位等。

* * *

新架构使得可以组合应用于其他选择器的变体，例如 `group-*`、`peer-*`、`has-*`，以及我们为 v4 引入的新的 `not-*` 变体。

在早期版本中，像 `group-has-*` 这样的变体在框架中是显式定义的，但现在 `group-*` 可以与现有的 `has-*` 变体组合，还可以与其他变体如 `focus` 组合：

```
<div  class="group">  <div  class="group-has-[&:focus]:opacity-100">  <div  class="group-has-focus:opacity-100">  </div>  </div> 
```

在这种组合性方面没有限制，甚至可以编写像 `group-not-has-peer-not-data-active:underline` 这样的东西，如果因为某些可怕的原因您确实需要这样做。

* * *

在这些早期的 alpha 版本中，您会注意到甚至无法配置您的 `content` 路径。对于大多数项目来说，您再也不需要这样做了 — Tailwind 会自动找到您的模板文件。

这是通过两种方式之一来实现的，具体取决于您如何将 Tailwind 集成到您的项目中：

+   **使用 PostCSS 插件或 CLI**，Tailwind 将遍历您的整个项目，寻找模板文件，我们内置了许多启发式方法以保持快速，例如不遍历您 `.gitignore` 文件中的目录，以及忽略二进制文件格式。

+   **使用 Vite 插件**，我们依赖于模块图。这非常棒，因为我们确切地知道您实际使用了哪些文件，因此性能最大化，没有误报或漏报。我们希望未来能将这种方法扩展到 Vite 生态系统之外的其他打包器插件中。

我们将来会引入一种明确配置内容路径的方式，但我们很好奇这种自动化方法在大家项目中的表现如何 — 在我们自己的项目中，它运行得非常出色。

* * *

Tailwind CSS v4.0的一个主要目标是使框架感觉更像是CSS原生的，而不是JavaScript库。

安装完后，你可以通过普通的CSS `@import`语句将其添加到项目中：

而不是在JavaScript配置文件中设置所有自定义内容，现在你只需使用CSS变量：

```
@import "tailwindcss";

@theme {
  --font-family-display: "Satoshi", "sans-serif";

  --breakpoint-3xl: 1920px;

  --color-neon-pink: oklch(71.7% 0.25 360);
  --color-neon-lime: oklch(91.5% 0.258 129);
  --color-neon-cyan: oklch(91.3% 0.139 195.8);
}
```

特殊的`@theme`指令告诉Tailwind基于这些变量提供新的工具和变体，让你可以在标记中使用类似`3xl:text-neon-lime`的类：

```
<div class="max-w-lg 3xl:max-w-xl">
  <h1 class="font-display text-4xl">
    Data to <span class="text-neon-cyan">enrich</span> your online business
  </h1>
</div>
```

添加新的CSS变量的行为类似于在框架早期版本中的`extend`，但你可以通过类似`--color-*: initial`的语法清除命名空间以覆盖整个变量集：

```
@import "tailwindcss";

@theme {
  --color-*: initial;

  --color-gray-50: #f8fafc;
  --color-gray-100: #f1f5f9;
  --color-gray-200: #e2e8f0;

  --color-green-800: #3f6212;
  --color-green-900: #365314;
  --color-green-950: #1a2e05;
}
```

我们仍在微调一些命名约定，但你可以[在GitHub上查看默认主题](https://github.com/tailwindlabs/tailwindcss/blob/next/packages/tailwindcss/theme.css)，了解可供定制的内容。

如果你不想显式清除默认主题，而是想从头开始，可以直接导入`"tailwindcss/preflight"`和`"tailwindcss/utilities"`来跳过导入默认主题：

```
@import  "tailwindcss";  @import  "tailwindcss/preflight"  layer(base);  @import  "tailwindcss/utilities"  layer(utilities);    @theme  {  --color-*: initial;  --color-gray-50:  #f8fafc;  --color-gray-100:  #f1f5f9;  --color-gray-200:  #e2e8f0;  --color-green-800:  #3f6212;  --color-green-900:  #365314;  --color-green-950:  #1a2e05;  } 
```

我们还将所有主题值作为本地CSS变量提供给你的自定义CSS：

```
:root {
  --color-gray-50: #f8fafc;
  --color-gray-100: #f1f5f9;
  --color-gray-200: #e2e8f0;

  --color-green-800: #3f6212;
  --color-green-900: #365314;
  --color-green-950: #1a2e05;
}
```

这使得在任意值中引用你的主题值变得更加容易，无需使用`theme()`函数：

```
<div class="p-[calc(var(--spacing-6)-1px)]">

</div>
```

这还使得在使用UI库如Framer Motion时可以使用你的主题值，而不必使用`resolveConfig()`函数：

```
import { motion } from "framer-motion"

export const MyComponent = () => (
  <motion.div
    initial={{ y: 'var(--spacing-8)' }}
    animate={{ y: 0 }}
    exit={{ y: 'var(--spacing-8)' }}
  >  {children}  </motion.div>
)
```

* * *

我们不轻易进行重大改动，但目前在v4中有几件我们正在做的事情与以往有所不同，值得分享：

+   **移除不推荐使用的工具** — 我们已经移除了很久没有文档记录的工具，比如`text-opacity-*`、`flex-grow-*`和`decoration-slice`，而采用它们的现代替代品，如`text-{color}/*`、`grow-*`和`box-decoration-slice`。

+   **PostCSS插件和CLI现在是独立的包** — 主要的`tailwindcss`包不再包含它们，因为并非所有人都需要，现在它们应该通过`@tailwindcss/postcss`和`@tailwindcss/cli`单独安装。

+   **没有默认的边框颜色** — `border`工具以前默认为`gray-200`，但现在默认为与浏览器相同的`currentColor`。我们做出这个改变是为了在项目中使用`zinc`或`slate`等其他主要灰色时，不会意外引入错误的灰色。

+   **默认情况下环形的宽度为1像素** — `ring`工具以前默认为3像素的蓝色环，现在默认为1像素并使用`currentColor`。我们在项目中发现自己更多地使用`ring-*`工具作为边框的替代品，并使用`outline-*`来表示焦点环，所以我们认为在这里保持一致性是一个有益的改变。

在您的项目中可能会出现一些其他非常低级的实现细节变化，但并非像这些变化那样有意为之。如果您碰到什么意外情况，请告诉我们。

* * *

这个新引擎是从头开始重写的，直到现在我们完全专注于使用新配置方法重新构想的开发者体验。

**我们非常重视向后兼容性**，这也是我们在今年晚些时候发布稳定v4.0版本前需要大量工作的地方。

+   **支持JavaScript配置文件** — 重新引入与经典的`tailwind.config.js`文件兼容，以便轻松迁移到v4版本。

+   **显式内容路径配置** — 在自动内容检测对您的设置不够理想时，可以告诉Tailwind确切的模板位置。

+   **支持其他暗模式** — 目前我们仅通过媒体查询支持暗模式，仍需重新实现选择器和变体策略。

+   **插件和自定义实用工具** — 目前我们不支持插件，也不支持编写自定义实用工具以自动适应不同变体。显然，在稳定版本发布之前我们会解决这个问题。

+   **前缀支持** — 目前还没有办法为您的类配置前缀，但我们肯定会在v4.0发布之前引入这一功能。

+   **白名单和黑名单** — 目前不能强制Tailwind生成特定类，或者防止它生成其他类。

+   **支持`important`配置** — 目前无法使所有实用工具都带有`!important`，但我们计划实现这一功能。

+   **支持`theme()`函数** — 对于新项目不需要此功能，因为现在可以使用`var()`，但我们将为向后兼容性实现它。

+   **独立CLI** — 我们还没有为新引擎工作的独立CLI，但在v4.0发布之前肯定会有。

此外，我相信我们会发现很多需要修复的错误，一些令人兴奋的新CSS特性也会悄悄地加入，以及一些需要在正式发布之前进一步完善的新API。

我不想在特定发布时间表上做承诺，但我个人非常希望在暑假开始之前将v4.0版本标记为稳定版本。

* * *

我们已经发布了几个alpha版本，您可以从今天开始在您的项目中使用它。

如果您正在使用VS Code的Tailwind CSS智能感知扩展，请确保从扩展页面切换到预发布版本，如果您使用我们的Prettier插件，请确保安装最新版本。

如果您发现问题，请在[GitHub上告诉我们](https://github.com/tailwindlabs/tailwindcss/issues/new/choose)。在我们标记稳定版本之前，我们真的希望这个工具能够无懈可击，报告您发现的任何问题将极大地帮助我们。

安装Tailwind CSS v4 alpha和我们的新Vite插件：

```
$ npm install tailwindcss@next @tailwindcss/vite@next
```

然后将我们的插件添加到您的`vite.config.ts`文件中：

```
import tailwindcss from '@tailwindcss/vite'
import { defineConfig } from 'vite'

export default defineConfig({
  plugins: [tailwindcss()],
})
```

最后，在您的主 CSS 文件中导入 Tailwind：

安装 Tailwind CSS v4 alpha 和单独的 PostCSS 插件包：

```
$ npm install tailwindcss@next @tailwindcss/postcss@next
```

然后将我们的插件添加到您的 `postcss.config.js` 文件中：

```
module.exports = {
  plugins: {
    '@tailwindcss/postcss': {}
  }
}
```

最后，在您的主 CSS 文件中导入 Tailwind：

安装 Tailwind CSS v4 alpha 和单独的 CLI 包：

```
$ npm install tailwindcss@next @tailwindcss/cli@next
```

接下来，在您的主 CSS 文件中导入 Tailwind：

最后，使用 CLI 工具编译您的 CSS：

```
$ npx @tailwindcss/cli@next -i app.css -o dist/app.css
```
