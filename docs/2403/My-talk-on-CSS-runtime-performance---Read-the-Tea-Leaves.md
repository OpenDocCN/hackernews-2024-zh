<!--yml

category: 未分类

日期：2024年5月27日14:42:46

-->

# 我的CSS运行时性能演讲 | 读茶叶

> 来源：[https://nolanlawson.com/2023/01/17/my-talk-on-css-runtime-performance/](https://nolanlawson.com/2023/01/17/my-talk-on-css-runtime-performance/)

17 Jan

## 我的CSS运行时性能演讲

发布于2023年1月17日，由Nolan Lawson在[performance](https://nolanlawson.com/tag/performance/)，[Web](https://nolanlawson.com/category/web/)。[3 Comments](https://nolanlawson.com/2023/01/17/my-talk-on-css-runtime-performance/#comments)

几个月前，我在[performance.now](https://perfnow.nl/)的阿姆斯特丹举行了有关CSS性能的演讲。录像在网上可以观看：

[https://www.youtube.com/embed/nWcexTnvIKI?version=3&rel=1&showsearch=0&showinfo=1&iv_load_policy=1&fs=1&hl=en&autohide=2&wmode=transparent](https://www.youtube.com/embed/nWcexTnvIKI?version=3&rel=1&showsearch=0&showinfo=1&iv_load_policy=1&fs=1&hl=en&autohide=2&wmode=transparent)

VIDEO

（您也可以阅读[幻灯片](https://nolanlawson.github.io/css-talk-2022/)。）

这是我给过的最喜欢的演讲之一。这是几个问题的产物：

1.  最快实现[作用域样式](https://rfcs.lwc.dev/rfcs/lwc/0116-light-dom-scoped-styles)的方法是什么？（令人惊讶的是很少有人提出这个问题。）

1.  使用影子DOM是否提高了样式性能？（简短回答：[是的](https://nolanlawson.com/2022/06/22/style-scoping-versus-shadow-dom-which-is-fastest/)。）

为了回答这些问题（以及更多），我进行了大量关于浏览器如何在幕后工作的研究。这包括查阅[2013年旧的Servo讨论](https://github.com/servo/servo/wiki/Css-selector-matching-meeting-2013-07-19)，联系浏览器开发人员如[Manuel Rego Casasnovas](https://blogs.igalia.com/mrego/)和[Emilio Cobos Alvarez](https://github.com/emilio)，阅读[浏览器PRs](https://github.com/WebKit/WebKit/commit/596fdf7c2cec599f8c826787363c54c4b008a7fe)，以及编写大量基准测试。

最后，我对这次演讲感到非常满意。我的主要目标是为多年来浏览器厂商为提高CSS性能所做的所有英雄般的工作点个赞。其中很多工作都很复杂和神秘（比如[Bloom filters](https://en.wikipedia.org/wiki/Bloom_filter)），但我希望通过一些简单的图表和动画，我能让这些工作栩栩如生。

我希望从这次演讲中看到的两个结果是：

1.  Web开发人员花费更多时间思考和衡量CSS性能。（嘘，看看[我的Chrome追踪指南](https://nolanlawson.com/2022/10/26/a-beginners-guide-to-chrome-tracing/)中的`SelectorStats`部分！）

1.  浏览器厂商提供更好的开发工具来理解CSS性能。（在演讲中，我把这个比作CSS的[`SQL EXPLAIN`](https://www.postgresql.org/docs/current/sql-explain.html)。）

在这次演讲中，我*并不*想泼冷水于那些试图运用 CSS 做出复杂设计的人。越来越多地，我看到对新 CSS 特性如[`:has`](https://developer.mozilla.org/en-US/docs/Web/CSS/:has)和[容器查询](https://developer.mozilla.org/en-US/docs/Web/CSS/CSS_Container_Queries)的[雄心勃勃的使用](https://www.bram.us/2023/01/12/sibling-scopes-in-css-thanks-to-has/)，我不希望人们觉得他们应该避开这些技术，仅限于类和 ID。我只是希望网页开发者考虑到 CSS 的成本，并更习惯于使用开发工具来理解哪些 CSS 模式可能会减慢他们网站的加载速度。

在我演讲之后，我还收到了一些浏览器开发工具团队的良好反馈，所以我对 CSS 性能的未来充满希望。随着诸如影子 DOM 和[原生 CSS 作用域](https://css-tricks.com/early-days-for-css-scoping/)技术的普及，它甚至可能会减轻我对 CSS 性能的许多担忧。无论如何，这是一个令人着迷的研究课题，我希望听众对我的演讲感到好奇和愉悦。
