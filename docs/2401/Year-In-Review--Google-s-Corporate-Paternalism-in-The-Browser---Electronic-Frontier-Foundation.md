<!--yml

category: 未分类

日期：2024-05-27 14:23:34

-->

# 年度回顾：Google 在浏览器中的企业式家长主义 | 电子前哨基金会

> 来源：[https://www.eff.org/deeplinks/2023/12/year-review-googles-corporate-paternalism-browser](https://www.eff.org/deeplinks/2023/12/year-review-googles-corporate-paternalism-browser)

企业式家长主义和在线广告跟踪技术的蔓延使得今年成为了一个重要的一年。谷歌及其子公司加强了对互联网创新的控制，同时采用了如今已经司空见惯的将这些事物营销为对用户有益的策略。在这里，我们将回顾今年最重要的变化，强调浏览器隐私工具（比如[Privacy Badger](https://privacybadger.org/)）比以往任何时候都更加重要。

## 从清单 V2 到清单 V3：遗留 Chrome 扩展的最终消亡

Chrome，所有测量标准中最受欢迎的网络浏览器，[最近宣布](https://developer.chrome.com/blog/resuming-the-transition-to-mv3/)了清单 V2 的正式终结日期，加快了其笨拙继任者清单 V3 的统治。[我们从一开始就在抱怨](https://www.eff.org/deeplinks/2021/12/chrome-users-beware-manifest-v3-deceitful-and-threatening)[这件事](https://www.eff.org/deeplinks/2021/12/googles-manifest-v3-still-hurts-privacy-security-innovation)，但是这是要点：随着时间的推移，MV3 的细节有所改善（即它不会完全破坏所有隐私扩展）。然而，它所带来的安全性好处是通过限制 *所有扩展可以做的事情* 而获得的。Chrome可以投资更加健全的扩展审核流程。这样做既可以保护创新，也可以保护安全性，但很明显，这一变化的真正目的在别处。直白地说：作为一家广告公司构建的浏览器，Chrome 将自己定位为浏览器隐私工具的守门人，它是这些工具应该如何设计的唯一裁判者。考虑到谷歌的跟踪器[至少存在于前 50,000 个网站中的 85%](https://spreadprivacy.com/duckduckgo-tracker-radar/)，为 2022 年带来了约[2250亿美元的整体利润](https://www.statista.com/statistics/266249/advertising-revenue-of-google/)，这是一个不足为奇但仍然令人失望的决定。

话虽如此，苹果的 Safari 浏览器也实施了类似的限制，据称是为了保护 Safari 用户免受恶意扩展的侵害。虽然保护用户免受恶意扩展的侵害很重要，但同样重要的是尊重他们的隐私。

## 话题 API

今年还见证了谷歌计划中的 ["隐私沙盒" 项目](https://www.eff.org/deeplinks/2023/09/how-turn-googles-privacy-sandbox-ad-tracking-and-why-you-should) 的推出，这个项目也使用了很多含糊其辞的营销来为其可疑的特性辩解。虽然它最终会摆脱第三方 cookie，这是一个老实说很好的举措，但它将以另一种称为 "主题 API" 的跟踪方式来替代该形式的跟踪。最多，这将减少能够通过 Chrome 浏览器跟踪用户的参与方数量（尽管我们不是唯一一个[质疑其所谓的好处的隐私专家](https://github.com/WebKit/standards-positions/issues/111#issuecomment-1359609317) [）](https://github.com/mozilla/standards-positions/issues/622#issuecomment-1372979100)。但它限制了跟踪，以至于只有一个强大的参与方，即 Chrome 本身，可以进行跟踪，然后将其了解传达给愿意付费的广告商。这只是将浏览器从用户代理转变为广告代理的又一步。

Privacy Badger 现在默认禁用了主题 API。

## YouTube 阻止使用广告拦截器的用户访问

最近，[安装了广告拦截器的人开始看到 Youtube 上的一个任性的消息](https://www.theverge.com/2023/10/31/23940583/youtube-ad-blocker-crackdown-broadening)。当试图观看视频时，拦截消息会给用户一个倒计时，直到他们禁用了广告拦截器为止，否则他们将无法再使用该网站。隐私和安全的好处都可以见鬼去了。YouTube，一个由谷歌拥有的公司，在第三季度广告收入达到了历史新高（仅为[80亿美元](https://www.statista.com/statistics/289657/youtube-global-quarterly-advertising-revenues/)），对于这一事件并没有任何含糊其辞、充满欺骗性语言的公告。如果你使用的是 Chrome 或基于 Chromium 的浏览器，那么除非你关闭广告拦截器，否则期待 YouTube 会崩溃。

## 隐私工具 > 企业家长作风

显然这一切都很糟糕。用户的安全不应该以牺牲隐私为代价。实际上，一个深深地与另一个交织在一起。所有这些糟糕的决定都突显了隐私工具的重要性。Privacy Badger 只是众多工具之一。Privacy Badger 之所以存在，不仅是为了保护无力的用户，它是一个即插即用的工具，静静地（但凶猛地）在幕后阻止跟踪行业的发展，而且它存在于其他志同道合的隐私项目的生态系统中，这些项目互相补充。一个工具可能会漏掉，而另一个则精准定位。

今年，Privacy Badger 推出了令人兴奋的支持项目和新功能：

在我们制定全面的隐私保护措施之前，在企业技术停止滥用我们不希望被监视的愿望之前，隐私工具必须被赋予权力来弥补这些损害。用户应该有权选择什么是对他们隐私的意义，而不是由谷歌等广告公司做出决定。

*这篇博客是我们年度回顾系列的一部分。[阅读关于2023年数字权利斗争的其他文章](https://eff.org/deeplinks/2023/12/2023-year-review)。*
