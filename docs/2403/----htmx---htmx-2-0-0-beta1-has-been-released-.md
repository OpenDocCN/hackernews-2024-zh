<!--yml

category: 未分类

date: 2024-05-27 15:02:47

-->

# htmx ~ htmx 2.0.0-beta1 已发布！

> 来源：[https://v2-0v2-0.htmx.org/posts/2024-03-15-htmx-2-0-0-beta1-is-released/](https://v2-0v2-0.htmx.org/posts/2024-03-15-htmx-2-0-0-beta1-is-released/)

# htmx 2.0.0-beta1 已发布！

## htmx 2.0.0-beta1 发布

我很高兴宣布 htmx 2.0 的第一个 beta 版本发布！

这是一个 *beta* 版本，不应用于生产环境。我们发布它是为了开始对真实场景进行最终测试。

如果您能够这样做，请切换到该发布版本，并快速检查您的所有 htmx 功能，然后告诉我们是否存在问题。

我们特别关注以下变更：

+   我们移除了已弃用的 `hx-sse` 和 `hx-ws` 属性，转而使用扩展（在 1.x 中可用）

+   `DELETE` 请求现在使用参数而不是表单编码体来传递负载（这符合规范）。

完整的升级指南可在此处找到：

[htmx 1.x -> 2.x 迁移指南](https://v2-0v2-0.htmx.org/migration-guide-htmx-1/)

请注意，htmx 2.x 将不再支持 IE，但 1.x 将继续为 IE 用户提供支持。

### 安装

该 beta 版本可以通过包管理器安装，引用版本 `2.0.0-beta1`，或者可以通过 CDN 进行链接：

```
<script src="https://unpkg.com/htmx.org@2.0.0-beta1/dist/htmx.min.js"></script> 
```

或 [下载](https://unpkg.com/htmx.org@2.0.0-beta1/dist/htmx.min.js)

### 扩展

扩展已从主代码库中移除，现在在此 GitHub 仓库中：

[https://github.com/bigskysoftware/htmx-extensions/tree/main/ext](https://github.com/bigskysoftware/htmx-extensions/tree/main/ext)

它们可在 [https://extensions.htmx.org](https://extensions.htmx.org) 获取

扩展中有一个破坏性变更，SSE，因此您必须使用该扩展的新版本：

https://github.com/bigskysoftware/htmx-extensions/blob/main/ext/sse.js

### 新功能

+   添加配置选项以忽略嵌套的超出范围交换而不处理它们（#1235）

+   添加 `textContent` 交换样式（#2356）

+   在 htmx.js 中完成完整的 JSDoc + 生成的 TypeScript 定义（感谢 @Telroshan！）（#2336）

+   完成了 [扩展网站](https://extensions.htmx.org)

+   添加了一个 [htmx 1.x 兼容扩展](https://github.com/bigskysoftware/htmx-extensions/blob/main/src/htmx-1-compat/README.md)
