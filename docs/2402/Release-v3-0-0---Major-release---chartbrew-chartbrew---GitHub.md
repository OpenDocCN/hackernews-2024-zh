<!--yml

类别：未分类

日期：2024-05-27 15:05:18

-->

# 发布 v3.0.0 - 主要更新 · chartbrew/chartbrew · GitHub

> 来源：[https://github.com/chartbrew/chartbrew/releases/tag/v3.0.0](https://github.com/chartbrew/chartbrew/releases/tag/v3.0.0)

# v3 已经到来 🚀

> [📺 在 YouTube 上观看 v3 的预览](https://www.youtube.com/embed/15nAw318Vo4?si=Ha-uKlqItUiCfeXE)

## v3.0.0 更新日志

发生了很多变化，因此此更新日志将仅列出显著的变更。

### 🔥 破坏性变更

+   `REACT_APP_API_HOST` 现在是 `VITE_APP_API_HOST`

+   `REACT_APP_CLIENT_HOST` 现在是 `VITE_APP_CLIENT_HOST`

+   新变量 `VITE_APP_CLIENT_PORT` 用于指定应用程序运行或提供的位置。默认值为 `4018`，但如果在不同端口上运行应用程序，则必须更改此值。

+   NodeJS v20 是最低要求的版本

+   团队角色已更改。以前的 `admin` 和 `editor` 角色现在设置为 `projectAdmin`。这将不允许他们创建连接和数据集，因此您可能需要在 UI 中重新分配团队成员的角色。

### ✨ 新的变更

+   `Connections` 和 `Datasets` 现在可以在团队级别重复使用，并且可以从任何仪表板访问它们。

+   **引入了对 Connections 和 Datasets 的 `tags`，以允许跨仪表板的复杂访问**

+   针对可变高度和拖放功能的仪表板的新格局设计，非常令人满意

+   全新的 UI 设计，色彩和整体用户体验得到了改进

+   新的连接创建向导界面，帮助您更快地入门

+   新的数据集构建器，内置图表预览和编辑器

+   可以直接从社区模板创建新的仪表板

+   新设计的用户仪表板，重点放在团队上

+   新的角色分为团队和仪表板级别。您的团队可以拥有 `teamOwner` 和 `teamAdmin` 角色，而您的客户可以根据其自身的 `projectAdmin` 和 `projectViewer` 角色访问特定仪表板。

+   新的公共仪表板设计（主题即将推出！）

+   新的 Strapi 插件更新以与 v3 兼容，新的仪表板布局和团队切换功能
