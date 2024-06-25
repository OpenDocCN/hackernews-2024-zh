<!--yml

分类：未分类

日期：2024年05月27日 14:44:34

-->

# 公开的 RSS – Chris Coyier

> 来源：[https://chriscoyier.net/2024/01/13/exposed-rss/](https://chriscoyier.net/2024/01/13/exposed-rss/)

2024年1月13日

# 公开的 RSS

我发现有些网站虽然实际上有 RSS 订阅源，但在其网站上却没有“RSS”或“Feed”链接。我不喜欢这样，但我能理解。也许他们选择了一个没有这个功能的现成主题。也许他们只是忘了。甚至可能他们根本不想要。

我从未考虑过那些在 HTML 中不公开 RSS 订阅源的网站。[正如 Robb Knight 所说](https://rknight.me/blog/please-expose-your-rss/)：

> 这被称为 RSS 自动发现，是一种向外部软件公开 RSS 订阅源的标准方法，以帮助[浏览器和其他软件自动找到一个站点的 RSS 订阅源](https://www.rssboard.org/rss-autodiscovery)。
> 
> 与标准链接一样，很多网站也缺少这一点。这是（至少作为第一步）像[NetNewsWire](https://netnewswire.com/)这样的订阅阅读器在你粘贴 URL 时会使用的东西，以自动找到一个订阅源。如果你有一个 RSS 订阅源，你的网站的`head`中应该有以下内容：

```
`<link rel="alternate" type="application/rss+xml" title="My Cool Website" href="https://example.com/feed.xml" />

<link rel="alternate" type="application/atom+xml" title="My Cool Website" href="https://example.com/atom.xml" />`Code language: HTML, XML (xml)
```

我通常甚至都不会费心去寻找订阅链接。如果我感觉到你的网站有一个订阅源，而我又想订阅它，我只需把你的首页 URL 复制到我的订阅阅读器的“添加”功能中，然后让它自行查找所需内容。如果找不到，我会感到有点遗憾。

所以是的 —— 如果你打算拥有一个订阅源，或者免费获取一个，确保你的网站上有那个 `<link>` 标签，否则像我这样的任何人都会以为你根本没有一个。

### *相关*

🤘
