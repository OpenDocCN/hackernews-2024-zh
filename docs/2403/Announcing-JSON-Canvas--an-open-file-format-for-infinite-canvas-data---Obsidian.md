<!--yml

category: 未分类

date: 2024-05-27 14:49:39

-->

# 宣布 JSON Canvas：一种开放的无限画布数据文件格式 - Obsidian

> 来源：[https://obsidian.md/blog/json-canvas/](https://obsidian.md/blog/json-canvas/)

今天，我们非常高兴地宣布，[Obsidian Canvas](https://obsidian.md/canvas) 文件格式现在称为 [JSON Canvas](https://jsoncanvas.org/)，并且拥有自己的网站、规范以及开源资源，位于 [jsoncanvas.org](https://jsoncanvas.org/)。

JSON Canvas 可以自由实现为[任何应用或工具](https://jsoncanvas.org/docs/apps/)的导入、导出和存储格式。所有与 JSON Canvas 相关的资源都在 MIT 许可下开源，并可以在 [GitHub](https://github.com/obsidianmd/jsoncanvas) 上找到。

对于 Obsidian Canvas 的 [发布](https://obsidian.md/changelog/2022-12-05-desktop-v1.1.0/)，我们创建了 `.canvas` 格式并提供了开放的规范。我们创建这种格式是因为我们认为遵循自从开始以来一直指导我们的 [原则](https://obsidian.md/about) 是至关重要的。您的 Obsidian 数据应始终以本地方式存储，离线访问，完全在您的控制之下，以易于检索和阅读的开放文件格式（如 [stephango.com/file-over-app](https://stephango.com/file-over-app)）。对于笔记，我们通过使用 Markdown 语法的纯文本 `.md` 文件实现了这一点，这是一种广泛支持的格式。然而，无限画布数据目前尚未有类似的已确立格式。

尽管无限画布工具并不新鲜，但它们已经迅速地 [增长了人气](https://infinitecanvas.tools/)。JSON Canvas 格式的创建旨在为使用无限画布应用创建的数据提供长期性、可读性、互操作性和可扩展性。该格式设计简单易解析，并赋予用户对其数据的所有权。

JSON Canvas 规范目前处于 [1.0 版](https://jsoncanvas.org/spec/1.0)。该规范相对保守，不支持画布应用可能想要实现的每个功能。然而，我们认为它是一个有用的起点，并计划随着时间的推移继续改进它。
