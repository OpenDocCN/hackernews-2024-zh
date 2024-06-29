<!--yml

category: 未分类

date: 2024-05-27 14:37:25

-->

# 发布 v1.1.0 · observablehq/framework · GitHub

> 来源：[https://github.com/observablehq/framework/releases/tag/v1.1.0](https://github.com/observablehq/framework/releases/tag/v1.1.0)

### Windows支持（还有Node 18）🎉

框架现在支持Windows操作系统！感谢您为 [#90](https://github.com/observablehq/framework/issues/90) 点赞。👍 框架内部使用POSIX路径分隔符（正斜杠 `/`），现在在读写文件时根据需要转换为Windows路径分隔符（反斜杠 `\`）。如果您遇到任何问题，请及时联系我们。

我们现在还支持Node 18。我们先前在[tsx](https://github.com/privatenumber/tsx)上有运行时依赖，允许我们将框架的源代码发布为TypeScript；现在我们发布预构建的JavaScript。这减少了组件数量，意味着我们不再依赖于Node 20.6中添加的[模块定制钩子](https://nodejs.org/docs/latest-v20.x/api/module.html#customization-hooks)。

### 自托管的`npm:`导入 🎉

框架现在在预览和构建期间自动从npm下载`npm:`导入！这意味着您的用户在使用构建站点时有更好的性能和安全性，因为您的站点没有**外部代码依赖**。例如，您现在可以强制在部署的站点上执行[内容安全策略](https://developer.mozilla.org/en-US/docs/Web/HTTP/CSP)，禁止跨源请求。它还改善了本地预览期间的性能，因为您只需下载库一次，并意味着您可以离线开发（例如在没有Wi-Fi的飞机上）。从npm下载的内容被缓存在源根目录下的`.observablehq/cache/_npm`文件夹中。您可以清除缓存并重新启动服务器以重新获取npm库的最新版本。

自托管的`npm:`导入适用于静态导入、动态导入和导入解析（`import.meta.resolve`），前提是规范器是静态字符串文字。例如，加载D3：

```
import * as d3 from "npm:d3";
```

```
const d3 = await import("npm:d3");
```

在上述两种情况下，都会从jsDelivr解析并下载`d3`的最新版本作为ES模块，包括其所有传递依赖项。隐式`npm:`导入，例如当您在Markdown中使用框架标准库提供的内置`d3`时，也现在是自托管的。 （如果您仍希望在运行时从CDN加载库而不是自托管，您可以导入URL，但我们不建议这样做，因为它会导致速度变慢，并且意味着您的站点可能在发布新版本库时出现问题。）

传递静态导入也会被注册为模块预加载（`<link rel="modulepreload">`），以便请求可以并行进行，并尽可能早地发生，而不是被链式加载。这显著提高了页面加载性能。框架现在还预加载`npm:`导入，例如用于CSV文件的`d3-dsv`方法。

导入解析允许您从npm下载文件。这些文件也会自动下载以供自托管。例如，要加载美国县的几何形状：

```
const data = await fetch(import.meta.resolve("npm:us-atlas/counties-albers-10m.json")).then((r) => r.json());
```

框架会根据[推荐的库](https://observablehq.com/framework/javascript/imports#implicit-imports)自动下载所需的文件，并解析静态和动态导入的导入解析。例如，`npm:@duckdb/duckdb-wasm`需要WebAssembly包，`npm:katex`需要样式表和字体，*等等*。对于那些不能静态分析的依赖项（例如`fetch`调用或动态参数传递给`import`的情况），您可以调用`import.meta.resolve`来注册需要从npm下载的其他文件。

哦，我们也正在为下一个版本准备`jsr:`导入功能。[#956](https://github.com/observablehq/framework/issues/956)

### CommonMark兼容性 🎉

框架以前在Markdown中存在一个问题，即空行的处理。现在问题已经解决，因此您可以使用标准的Markdown书写，不会有任何意外的惊喜！😌 特别是，根据[GitHub风格的Markdown规范](https://github.github.com/gfm/#html-blocks)，您现在可以在HTML块内部编写Markdown：

```
<div class="card">

## This is a card
### With a subtitle

And some *Markdown* text!

</div>
```

### 构建时文件名使用内容哈希

框架现在在发布的文件名中插入内容哈希在`_file`和`_import`中。这确保了在部署时破坏缓存，并启用[cache-control: immutable](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Cache-Control#immutable)以进一步提高性能。当部署到Observable时，您将自动获得优化的缓存。

### 草稿模式

新的**草稿**选项允许您在构建中排除某些页面。这使您可以在本地开发页面，但在准备好分享之前阻止它们被发布。要将页面标记为草稿，请使用front matter：

### markdown-it插件

您现在可以通过[markdown-it配置选项](https://observablehq.com/framework/config#markdownit)注册额外的[markdown-it](https://github.com/markdown-it/markdown-it) [插件](https://www.npmjs.com/search?q=keywords:markdown-it-plugin)。例如，要使用[markdown-it-footnote](https://github.com/markdown-it/markdown-it-footnote)插件：

```
import type MarkdownIt from "markdown-it";
import MarkdownItFootnote from "markdown-it-footnote";

export default {
  markdownIt: (md: MarkdownIt) => md.use(MarkdownItFootnote)
};
```

### 还有许多其他的改进…

[Apache ECharts](https://observablehq.com/framework/lib/echarts)现在作为Markdown中的默认选项`echarts`提供。（尽管我们仍然更喜欢我们自己的[Observable Plot](https://observablehq.com/framework/lib/plot)和[D3](https://observablehq.com/framework/lib/d3)库用于可视化！）

框架现在对静态HTML中引用的文件（如`<img>`和`<video>`元素以及样式表）进行实时更新。如果编辑了[自定义样式表](https://observablehq.com/framework/config#style)，框架也会进行实时更新。框架现在会跳过未使用的内置模块，并在构建过程中对CSS进行缩小。

框架的内置搜索现在可以正确索引使用任何[Unicode编写系统](https://en.wikipedia.org/wiki/Script_(Unicode))的文本。索引之前会对变音符号和HTML实体进行规范化；例如，现在您可以键入“manana”并检索包含“mañana”和“ma&ntilde;ana”的页面。当标题不是字符串时，索引不再崩溃。首页现在包含在索引中。新的[**关键词选项**](https://observablehq.com/framework/search)允许在前事项中索引额外增强的词语。搜索结果不再显示不必要的滚动条。

`FileAttachment.url()` 现在返回绝对URL而不是相对路径。当MIME类型未知时，`FileAttachment.mimeType` 现在是`"application/octet-stream"`而不是`"null"`。不再支持不支持CSS `:has()` 选择器的旧版浏览器。在启动预览服务器时，框架不再考虑 `HOSTNAME` 和 `PORT` 环境变量；而是使用命令行参数 `--host` 和 `--port`。Markdown标题现在在默认样式中应用了`text-wrap: balance`。

所有这些都发生在发布不到三周之内！😅 感谢大家的反馈和鼓励。

## 新的贡献者

**完整更新日志**: [`v1.0.0...v1.1.0`](https://github.com/observablehq/framework/compare/v1.0.0...v1.1.0)
