<!--yml

分类：未分类

日期：2024-05-27 14:31:52

-->

# Why Isn’t the Element 100% Supported on CanIUse.com? | anderegg.ca

> 来源：[https://anderegg.ca/2024/02/02/why-isnt-the-html-element-100-supported](https://anderegg.ca/2024/02/02/why-isnt-the-html-element-100-supported)

## 为什么在 CanIUse.com 上 `<html>` 元素不被 100% 支持？

### February 02, 2024

今早我看了一个 [Mastodon 帖子](https://mastodon.gamedev.place/@Ronflaix/111862153259345050)，让我感到很好笑。早上的时候，看到 `html` 元素在 CanIUse.com 上 [没有 100% 的支持](https://caniuse.com/mdn-html_elements_html) ，感觉有点疯狂！自从 1994 年开始使用它，当时它的确很好用！这让我进入了一个小小的深坑。

背景简介。[Can I Use…](https://caniuse.com) 是一个帮助网页开发者追踪网页技术采用率的网站。它估计浏览器使用情况，测量特性兼容性，并提供一个反映特性可用性的数字。自 [2010 年推出](https://web.archive.org/web/20100430032738/http://caniuse.com/) 以来，我一直在使用这个网站，并且一直觉得它非常有用。

那么，为什么目前 CanIUse.com 上显示 `html` 元素只有 97.34% 的支持？这比当前 [`audio` 元素的支持率](https://caniuse.com/audio) 还要低！看起来 [`a`](https://caniuse.com/mdn-html_elements_a) 和 [`p`](https://caniuse.com/mdn-html_elements_p) 元素的支持率也正好是相同的 97.34%。

在调查的过程中，我学到的一件事是，这个网站上的许多数据实际上来自 [MDN](https://developer.mozilla.org/en-US/) ^(. MDN 是我另一个使用和信任的资源，所以这对我来说很合理。它也经常有关于特性采用情况的统计数据，所以 CanIUse.com 借用这些数据是有道理的。)

查看 [MDN 上的 `html` 元素页面](https://developer.mozilla.org/en-US/docs/Web/HTML/Element/html)，它有一个 [浏览器兼容性部分](https://developer.mozilla.org/en-US/docs/Web/HTML/Element/html#browser_compatibility)。其中有两行显示了很多红色的 X。第一行是关于 `html` 元素上的可选 `manifest` 属性。这个属性已被废弃且从未标准化。第二行涉及 “[secure context required](https://w3c.github.io/webappsec-secure-contexts/)”，这是一个 [编辑草案](https://www.w3.org/standards/types/#x2-3-editor-s-draft) — 也就是说，目前不在标准化追踪上。我不知道这与 `html` 元素之前的关联是如何的，但该用法同样 [从未标准化且已废弃](https://caniuse.com/mdn-html_elements_html_manifest_secure_context_required)。

因此，在这里列出的两个特性，几乎所有浏览器都*正确地*不支持。但是，看起来这并不是缺失2.66%的原因。有一些浏览器被列为“支持情况未知”。将这些浏览器当前的使用情况加起来，总共为1.27%。还有一个条目是Android浏览器版本2.1至4.3，据说不支持`html`元素 —— 我觉得这很可疑 —— 但列为使用份额为0%。我想这里可能有一些四舍五入误差，可能会把1.27%增加到2.66%？但我仍然觉得这一点很不清楚。而且，我非常有信心那些旧版本的浏览器支持`html`元素！

所以，对于这个问题，我没有一个很好的答案。如果你有的话，请在[这里告诉我](https://mastodon.social/@gavinanderegg)！我一直以来对CanIUse.com上的数据持保留态度，但今后我会再添加几个。我仍然认为这是一个很好的资源。

* * *

**更新：** [rezonant](https://cliff.social/@rezonant) 在Mastodon上提醒我，他在Hacker News上发表了一篇[评论](https://news.ycombinator.com/item?id=39236148)，提出了一个可能的解释。简而言之：如果你在右上角的“使用率”旁边切换到“所有已追踪的百分比”，然后将“支持情况未知”的浏览器数字加起来，你会得到99.98%。这样更容易看出四舍五入误差可能使这个数字正确。但我认为这里对待旧版本浏览器的方式令人困惑。这并不会对总体数字产生太大影响，但在像`html`这样的基础元素的情况下，这似乎很奇怪。

* * *
