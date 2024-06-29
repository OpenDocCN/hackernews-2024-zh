<!--yml

category: 未分类

date: 2024-05-29 13:27:20

-->

# 为什么我使用Firefox

> 来源：[https://xn--ime-zza.eu/3](https://xn--ime-zza.eu/3)

<main>

# 为什么我使用Firefox

2024年2月26日

“我应该使用哪个浏览器？”这样的问题经常在[r/browsers subreddit](https://www.reddit.com/r/browsers/)上出现。我有时会回复这些帖子，但我的快速回复通常只包含一两个要点。说实话，直到最近，我自己甚至不确定为什么我使用Firefox。当然，它是一个相当不错的浏览器，但这并不能解释为什么我已经固执地忠于Firefox超过十年。经过更深思熟虑，我得出了以下几个原因。

## 1\. *about:config*页面

在Firefox中，有一个内部的*about:config*页面，其中包含数千（几万？）个用户可以自由编辑的单独配置。如果你不喜欢Firefox中的特定UI元素或行为，有很大可能性可以通过配置进行更改。*about:config*页面还用于单独启用实验性的Web平台功能（无需像Chrome那样需要重新启动浏览器）。

这里是我编辑或添加的一些配置：

devtools.toolbox.zoomValue = 1.2

将Firefox开发工具的默认文本大小增加到120%

browser.tabs.closeWindowWithLastTab = false

当用户关闭最后一个标签时，阻止整个浏览器窗口关闭（我觉得这种行为很烦人）。

devtools.inspector.showUserAgentStyles = true

在Firefox开发工具的CSS规则窗格中显示用户代理样式（为什么默认情况下用户代理样式是隐藏的？）

browser.chrome.guess_favicon = false

阻止Firefox尝试从默认位置加载网站的favicon（当HTML文档中未声明图标时）（我使用这个配置来消除devtools控制台中烦人的“favicon未找到”错误）

browser.urlbar.resultMenu.keyboard­Accessible = false

删除URL栏下拉列表中每个项目的菜单按钮（这些按钮使通过项目的选项卡变慢）

## 2\. Mozilla无法在其服务器上解密我的数据

所有主流浏览器都有一个功能，用于在设备之间同步用户的浏览数据（Firefox Sync、Chrome Sync、Apple iCloud等）。用户的数据存储在浏览器供应商的服务器上，并且这些数据当然是加密的。但是浏览器供应商能解密这些数据吗？谷歌可以。苹果声称他们不能，但他们过去曾向执法部门披露用户数据，所以我不信任他们。Mozilla表示他们不能，并且我信任他们。

看起来Mozilla确保他们绝对不能访问Firefox用户同步浏览数据。加密强度足够强大，以至于使用当前技术要破解这些数据将需要数万亿年，因此它非常安全。然而，如果我不知何故地丢失了我激活了Firefox Sync的所有设备，我的浏览数据在Mozilla的服务器上将永远丢失；没有办法恢复它。尽管如此，我喜欢使用一家公司的浏览器，他们不想在自己的服务器上访问我的数据。我觉得这才是应有的做法。

## 3\. 翻译网页也是完全私密的

Firefox翻译是一个相对较新的功能，允许用户直接在浏览器中将网页翻译成不同语言（从一小部分支持的语言中选择），而无需向任何服务器发送任何数据。此功能基于机器学习和神经网络。

这是Mozilla为保护用户隐私而额外努力的又一个例子。

## 4\. Mozilla开发了他们自己的浏览器引擎

Firefox使用Mozilla的Gecko浏览器引擎。没有其他主要浏览器使用Gecko。Web是我最喜欢的平台，由于浏览器引擎的多样性对Web有益，我想支持Gecko。通过使用Firefox并报告Firefox和Web兼容性问题，我正在尽我的一份力。

*请允许我引用[Google的常见问题解答](https://www.chromium.org/blink/developer-faq/#hold-up-isnt-more-browsers-sharing-webkit-better-for-compatibility)（2013年在分叉WebKit时）：

> ### 慢着，更多使用WebKit的浏览器对兼容性更好吗？
> ### 
> 记住，WebKit对开发者来说并非一个均质的目标。例如，像WebGL和IndexedDB这样的特性只在一些基于WebKit的浏览器中支持。了解WebKit对开发者有助于解释细节，比如为什么在WebKit浏览器中，`<video>`、字体和3D变换的实现存在差异。
> 
> 如今，Firefox使用的是Gecko引擎，而不是基于WebKit的。尽管如此，这两者具有高度的兼容性。我们采用了类似Mozilla的方法，拥有一个独特但兼容的开源引擎。我们还将继续开放式地跟踪缺陷和实现状态，以便您随时了解和贡献我们正在进行的工作。
> 
> 从短期来看，单一文化似乎有利于开发者的生产力。然而，从长期来看，单一文化不可避免地导致停滞不前。我们坚信，在渲染引擎中提供更多选择将促进更多创新和更健康的Web生态系统。
> 
> ### 这对Web标准有何影响？
> ### 
> 引入新的浏览器引擎增加了多样性。虽然这本身并不是我们的目标，但它确实有益的影响，确保存在多个可以相互操作的接受标准的实现。每个引擎都会从不同的角度解决同样的问题，这意味着Web开发人员对最终结果的性能和安全性特性更加有信心。这也减少了一个实现的怪癖成为事实标准的可能性，这对整个开放网络都是有利的。

我无法说得更好了。目前我们有三个主要的浏览器引擎——以及几个正在开发中的较小引擎——其中三个引擎中，Gecko 是唯一可能面临风险的一个。如果 Gecko 是一种真实的动物，我不确定它的[保护状态](https://en.wikipedia.org/wiki/Conservation_status)会是什么（可能是“保护依赖”），但我不打算轻易放弃它。

## 5\. Android 上最好的扩展支持

在过去的十年里，网络不幸地变得更慢、更烦人了。阻止广告和其他类型问题内容的扩展已成为正常网络浏览体验所必需的。在 Android 上，Firefox 显然是最佳的浏览器扩展支持平台。这包括 uBlock Origin（最好的广告拦截器）以及用于向网站添加用户样式和用户脚本的扩展。我积极地在桌面上使用所有这些扩展（uBlock Origin、[Stylus](https://add0n.com/stylus.html)、Tampermonkey），来调整网站以符合我的喜好。很棒的是，Android 上的 Firefox 用户也可以做到同样的事情。

## 6\. 一个出色的画中画播放器

最后，我应该提到 Firefox 中一个普通用户可能会觉得有用的实际功能。我并不是真的用 Firefox 来使用它的一般功能，但如果有一个我真的喜欢的功能，那就是桌面版 Firefox 中出色的原生画中画视频播放器。它拥有任何人都可能需要的一切功能。可以通过悬停在任何视频上时显示的覆盖按钮快速打开。它可以调整大小并随意放置在屏幕上的任何位置。它拥有全面的控制功能，包括暂停、静音和视频跳转的进度条。我经常使用它。

## 总结

我相信 Mozilla，比我相信 Google、Apple、Microsoft 或其他任何制造Web浏览器的公司都更多。这种信任基于Mozilla在开发服务如Firefox Sync、Firefox Translate等时选择的最高用户隐私级别。Web浏览器是一个人在线生活中不可或缺的一部分，所以选择一个最信任的公司的浏览器是有道理的。

除此之外，Firefox 提供了最高级别的自定义功能，无论是通过浏览器扩展还是内部配置。对我来说，这一点很重要，因为我更喜欢网站而非本地应用。

任何出色的功能，比如画中画播放器，只是锦上添花。我理解对大多数人来说可能恰恰相反。他们更关心功能，而不是隐私和定制性。这很好。没有对错之分。每个人都应该使用最适合自己的浏览器。

[Mastodon 上的回复](https://mastodon.social/@simevidas/111995936867105049)

</main>
