<!--yml

category: 未分类

date: 2024-05-29 12:32:13

-->

# 现代网页臃肿意味着一些页面加载了21MB的数据——入门级手机无法运行一些简单的网页，有些网站的渲染难度甚至超过了PUBG | Tom's Hardware

> 来源：[https://www.tomshardware.com/tech-industry/modern-web-bloat-means-some-entry-level-phones-cant-run-simple-web-pages-and-load-times-are-high-for-pcs-some-sites-run-worse-than-pubg](https://www.tomshardware.com/tech-industry/modern-web-bloat-means-some-entry-level-phones-cant-run-simple-web-pages-and-load-times-are-high-for-pcs-some-sites-run-worse-than-pubg)

本月初，[Danluu.com发布了一份详尽的23页分析/评论/宣言](https://danluu.com/slow-device/)，分析了当前未优化的网页和网页应用性能的现状，发现仅仅加载一个网页就可能让一个可以以40帧每秒运行流行游戏PUBG的入门级设备感到困扰。事实上，Wix网页需要加载21MB的数据来显示一个页面，而更著名的网站Patreon和Threads加载13MB的数据来显示一个页面。这可能导致加载时间缓慢，高达33秒，甚至有时导致页面根本无法加载。

主要的测量数据显示在下面的表格中（展开推文以查看表格），该表格测量了几个网站和低配设备上的最大内容绘制（LCP）时间。正如LCP的全称所暗示的，这通常是用户打开页面后设备渲染其主要内容的时间。除了加载时间之外，还显示了每个网站的带宽需求在左侧列中。测试包括最低端的Itel P32、入门级的Tecno S8C以及像Apple M3 Max、M1 Pro和M3这样更强大的系统。

如上述测试所示，一些最具挑战性的网站包括...Quora和几乎所有主要的社交媒体平台。像Squarespace这样的新内容生产平台和像Discourse这样的新论坛平台在某些设备上的性能显著较差，往往达到无法使用的程度，甚至不如它们的老对手。

Tecno S8C是一款在新兴市场广泛使用的突出的入门级手机之一，是一个特别引人注目的测试设备。在某些方面，这款设备确实非常出色，包括其[能够以40帧每秒运行*绝地求生：刺激战场*](https://www.youtube.com/watch?v=McawfNlydqk)的能力——但同一款设备甚至无法运行Quora，并且在社交媒体网站上滚动时经历了几乎无法使用的延迟。

那个例子很可能是总结整个观点的最佳概括，即现代网页和应用设计越来越倾向于对带宽和处理能力不切实际的假设。Quora是一个人们*回答问题*的网站——没有任何理由这些网站中的任何一个应该比《绝地求生：大逃杀》游戏更难运行。

全文详细讨论了这些观点，并包括了几位网页开发者的引用和引用。例如，Discourse的创始人兼前CEO Jeff Atwood曾说过，高通公司在他们的工作中是*"糟糕的。我希望他们破产"……因为高通的CPU比苹果的落后15%。这似乎是对市场上提供大多数智能手机SoC的制造商的极端评价。您可以[在此处阅读完整报告](https://danluu.com/slow-device/)，获取有关测试和结果性能测量的更多详细信息。

获取Tom's Hardware的最佳新闻和深度评论，直接发送到您的收件箱。
