<!--yml

分类：未分类

日期：2024-05-27 15:22:49

-->

# 注意 arXiv 用户：重新实现的 RSS – arXiv 博客

> 来源：[https://blog.arxiv.org/2024/01/31/attention-arxiv-users-re-implemented-rss/](https://blog.arxiv.org/2024/01/31/attention-arxiv-users-re-implemented-rss/)

arXiv 已重新实现我们的 RSS 订阅以改善用户的结果。这是 arXiv 的 RSS 订阅的完全重写版本，运行在云端。它应该创建一个更准确的每天文章列表。有关技术细节，请参阅 [RSS Feeds – arXiv info](https://info.arxiv.org/help/rss.html)。

与旧的 RSS 订阅的变化：

+   新的 rss.arxiv.org 基本 URL，而不仅仅是 arxiv.org（arxiv.org/rss 将重定向到 rss.arxiv.org）

+   状态页面在 /feed/status

+   新内容每天午夜更新（美国东部时间，如常）

+   可以请求多个类别，例如 +hep-lat+math.CO

+   项目数量限制为 2000

+   新、跨、替换和替换-交叉的分类和顺序现在与列表匹配

+   现在为每篇论文提供了作者列表

+   RSS 输出符合 RSS 2.0 规范。RSS 0.91 和 1.0 不再受支持。Atom 格式仍然受支持 — 例如 http://rss.arxiv.org/atom/math。
