<!--yml

类别：未分类

日期：2024-05-27 14:45:21

-->

# kmaasrud

> 来源：[https://kmaasrud.com/blog/opml-is-underrated.html](https://kmaasrud.com/blog/opml-is-underrated.html)

# OPML被低估了

作为对主要平台的普遍[侵犯化](https://doctorow.medium.com/social-quitting-1ce85b67b456)的回应，我认为我们正在见证旧网络精神的复兴，个人博客日渐流行，并且[小网络](https://neustadt.fr/essays/the-small-web/)的概念也在兴起。虽然这可能受到我参与的数字社区（大部分由程序员主导）的影响，但至少在经验上确实感觉到这是一个趋势。这带来了对开放网络标准的新兴兴趣。其中之一就是RSS，我和很多其他人越来越多地将其整合到我们的数字日常中，用来跟踪我们关注的人和来源的帖子。RSS与向更个性化和受控内容消费的趋势完美契合。不像<q>其他平台</q>那样由算法驱动的信息流——后者通常优先考虑互动而非相关性或质量——RSS允许我自主策划自己的信息流。对我来说，这感觉很重要，因为它让我能够掌控塑造我的观点和知识的内容，而不是将这种权力交给广告商。

但是，也存在问题。管理和跟踪RSS订阅源可能有些笨拙。它们的分散特性也使得发现性成为一个问题。这时就需要[OPML](http://opml.org/spec2.opml)登场了，这是一种大纲格式，最常用于存储[订阅源列表](http://opml.org/spec2.opml#subscriptionLists)。我向你保证，拥有一个存储你感兴趣的所有订阅源的单一文件是一场改变游戏规则的行动，因为它极大地简化了在不同平台和设备上组织、迁移和共享这些订阅源的过程。这里有个例子：

```
<?xml version="1.0" encoding="utf-8"?>
<opml version="2.0">
 <head>
 <title>A list of feeds I follow</title>
 </head>
 <body>
 <outline text="My favorite blog" xmlUrl="https://a-cool-blog.tld/blog/feed.xml" type="rss"
 htmlUrl="https://a-cool-blog.tld/blog" description="You can also add a description" />
 <!-- more outline elements with feeds... -->
 </body>
</opml>
```

每个大纲项必须具有类型`rss`（对于RSS和Atom订阅源都适用），并且必须包含`xmlUrl`属性。可以选择地添加一些额外的属性，比如用`text`添加标题，用`description`添加描述，以及用`htmlUrl`添加到博客首页的链接——这些附加的元数据非常有用。是的，它基于XML，我承认这并不是最容易处理的格式，但它有一些优势，我们将在后面详细讨论。

使用OPML，你不需要单独的应用程序或服务来分类订阅源。可以通过其大纲功能在单个OPML文件内实现分类，或通过管理多个专用于不同类别或用例的OPML文件。为你的YouTube订阅设置一个OPML文件，为你喜欢的Twitter/X和Mastodon用户设置另一个，再为新闻网站设置一个，再为个人博客设置一个——世界尽在你掌握。然而，很遗憾，并没有多少应用程序支持嵌套的OPML大纲或基于不同文件分类，但应该有！这是一个行动号召，开发者们：完美的副业项目！

除了个人便利性之外，OPML还有可能改善<q>小网</q>的生态系统。不仅在个人网站上分享RSS订阅源，还可以分享你订阅的列表，这有效地创建了一个基于有意识策划而非统计指标的推荐系统。你的OPML文件现在被称为*[博客列表](https://blogroll.org/what-are-blogrolls/)*，你正式可以称自己为**90年代的网页开发者**。开玩笑的，我认为背后有人的每个推荐的简单事实都是有利的。是的，这可能会促进较小的数字社交圈子，但我个人认为已知来源的透明性是对抗[过滤气泡](https://en.wikipedia.org/wiki/Filter_bubble)的最佳方式。这部分本身是整个社会学讨论的一个话题，如果你想进一步讨论，我很乐意在[我的邮件列表](https://lists.sr.ht/~kmaasrud/inbox)上聊聊。

现在，回到OPML基于XML的事实；我想强调一个经常被忽视的特性：通过使用XSL样式表，在浏览器中加载时可以显示经过HTML模板渲染的OPML文件。通过这种方式，你可以添加一个简短的介绍和指南，使得对这个格式不熟悉的人更容易理解博客列表。这也为展示每个订阅源提供了增加的背景或描述的可能性。

<details><summary>这是一个你可以使用的XSL样式表的示例：</summary>

```
<?xml version="1.0" encoding="UTF-8"?>
<xsl:stylesheet version="1.0" xmlns:xsl="http://www.w3.org/1999/XSL/Transform">
 <xsl:template match="/opml">
 <html xmlns="http://www.w3.org/1999/xhtml" xml:lang="en" lang="en">
 <head>
 <title>
 <xsl:value-of select="head/title"/>
 </title>
 <meta name="viewport" content="width=device-width, initial-scale=1"/>
 <style> /* Insert CSS here */ </style>
 </head>
 <body>
 <p>
 This is a list of blogs and news sources I follow. The page
 is itself an <a href="http://opml.org/">OPML</a> file, which
 means you can copy the link into your RSS reader to
 subscribe to all the feeds listed below.
 </p>
 <ul>
 <xsl:apply-templates select="body/outline"/>
 </ul>
 </body>
 </html>
 </xsl:template>
 <xsl:template match="outline" xmlns="http://www.w3.org/1999/xhtml">
 <xsl:choose>
 <xsl:when test="@type">
 <xsl:choose>
 <xsl:when test="@xmlUrl">
 <li>
 <strong>
 <a href="{@htmlUrl}"><xsl:value-of select="@text"/></a>
 (<a class="feed" href="{@xmlUrl}">feed</a>)
 </strong>
 <xsl:choose>
 <xsl:when test="@description != ''">
 <br/><xsl:value-of select="@description"/>
 </xsl:when>
 </xsl:choose>
 </li>
 </xsl:when>
 </xsl:choose>
 </xsl:when>
 </xsl:choose>
 </xsl:template>
</xsl:stylesheet>
```</details>

你可以通过在OPML文件顶部添加`<?xml-stylesheet type="text/xsl" href="path/to/stylesheet.xsl"?>`来链接到样式表。实际上，我在[我的博客列表](https://kmaasrud.com/blogroll.xml)上就这么做的，所以如果你需要一些灵感，可以去看看。

当我们都有点厌倦大平台时，OPML就像从旧网络时代带来的一股清新空气。它完全是关于在管理订阅源时使生活更轻松，分享有趣的发现，偶然发现新事物。因此，我鼓励你创建自己的博客列表，把它放在你的网站上，并分享你喜欢的东西。这是一个简单的举措，但它可能引发一些真实的连接，并带回我们怀念的社区氛围的一部分。

### 这里有一些我关注的网站上的帖子

TL;DR：rustc 将在夜间版默认使用 rust-lld 来显著减少在 x86_64-unknown-linux-gnu 上的链接时间。一些背景链接时间通常是编译时间的重要组成部分。当 rustc 需要构建一个二进制文件或共享库时，通常会…

via [Rust Blog](https://blog.rust-lang.org/) 2024年5月17日

在过去的几年里，我在我的在线社区中越来越多地注意到关于“小型网络”的讨论。看到这样的情况真是太酷了！我在早些时候的一篇文章中提到了这个概念，今天想再写几句。据我所知…

via [PKGW](https://newton.cx/~peter/) 2024年5月14日

今天，正好是我，马塞尔，五年前开始着手开发Music Assistant。起初只是一个快速脚本，用来同步我的播放列表，这样我可以在不同的流媒体服务提供商之间切换，但它发展成了一种独特的存在。Music Assistant 是我想称之为“音乐…

via [Home Assistant](https://www.home-assistant.io/) 2024年5月9日
