<!--yml

category: 未分类

日期：2024-05-27 15:11:51

-->

# htmx ~ htmx 2.0.0-alpha1 版本已发布！

> 来源：[`v2-0v2-0.htmx.org/posts/2024-01-26-htmx-2-0-0-alpha1-is-released/`](https://v2-0v2-0.htmx.org/posts/2024-01-26-htmx-2-0-0-alpha1-is-released/)

# htmx 2.0.0-alpha1 版本已发布！

## htmx 2.0.0-alpha1 版本发布

我很高兴地宣布 htmx 2.0 的第一个 alpha 版本发布

这是一个 *alpha* 版本，不应被视为生产就绪。我们发布它是为了开始测试代码针对实际情况的表现，以找出存在的问题和需要改进的地方。

如果您能够这样做，请切换到该版本，并快速运行您的任何 htmx 功能，如果有任何问题，请告诉我们。

我们特别关注以下变化：

+   我们已删除了 hx-sse 和 hx-ws 属性，以支持扩展（在 1.x 中可用）

+   我们将 `head` 标签处理从 `head-extension` 集成到核心中，现在头部处理是增强链接的默认行为。

+   `DELETE` 请求现在使用参数而不是表单编码的主体进行载荷（这符合规范。）

完整的升级指南可在此处找到：

[htmx 1.x -> 2.x 迁移指南](https://v2-0v2-0.htmx.org/migration-guide-htmx-1/)

注意，htmx 2.x 将不再兼容 IE，但 1.x 将继续为 IE 用户提供支持。

### 安装

该 alpha 版本可以通过包管理器引用版本 `2.0.0-alpha1` 进行安装，也可以通过 CDN 进行链接：

```
<script src="https://unpkg.com/htmx.org@2.0.0-alpha1/dist/htmx.min.js"></script> 
```

或者 [下载](https://unpkg.com/htmx.org@2.0.0-alpha1/dist/htmx.min.js)

### 扩展

扩展已从主代码库中删除，并现在在此 github 存储库中：

[`github.com/bigskysoftware/htmx-extensions/tree/main/ext`](https://github.com/bigskysoftware/htmx-extensions/tree/main/ext)

它们最终将在 [`extensions.htmx.org`](https://extensions.htmx.org) 和 NPM 上提供，但现在必须从 github 存储库中链接。

扩展中有一个重要变化，SSE，因此您必须使用该扩展的较新版本：

[`github.com/bigskysoftware/htmx-extensions/blob/main/ext/sse.js`](https://github.com/bigskysoftware/htmx-extensions/blob/main/ext/sse.js)

### 新特性

+   如上所述，htmx 现在默认包含 `head` 标签合并功能

+   htmx 现在可以在 Web 组件的 Shadow DOM 中正常工作

+   属性继承现在可以通过 `htmx.config.disableInheritance` 配置变量完全禁用

+   响应代码处理现在可通过 `htmx.config.responseHandling` 配置变量进行配置

+   基于模板的解析现在是解析新内容的标准机制，这应该会大大减少与表元素混合在其他内容中的问题
