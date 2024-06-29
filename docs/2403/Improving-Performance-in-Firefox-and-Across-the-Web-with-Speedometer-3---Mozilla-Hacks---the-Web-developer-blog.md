<!--yml

category: 未分类

date: 2024-05-27 14:49:46

-->

# 提升 Firefox 和整个 Web 的性能，借助 Speedometer 3 - Mozilla Hacks - Web 开发者博客

> 来源：[https://hacks.mozilla.org/2024/03/improving-performance-in-firefox-and-across-the-web-with-speedometer-3/](https://hacks.mozilla.org/2024/03/improving-performance-in-firefox-and-across-the-web-with-speedometer-3/)

与其他主要浏览器引擎开发者合作，Mozilla 今天很高兴[宣布 Speedometer 3](https://browserbench.org/announcements/speedometer3)。与 Speedometer 的先前版本一样，这个基准测试衡量了[我们认为在线性能最重要的事情](https://www.mozilla.org/en-US/about/webvision/full/#performance)：响应速度。但今天的发布比以往任何时候更开放、更具挑战性，是驱动浏览器性能改进的最佳工具。

这实现了[2022 年 12 月设定的愿景](https://twitter.com/mozhacks/status/1603435347190419456)，将整个行业的专家聚集在一起，重新思考如何尽可能地反映真实世界的 Web，并以共同的目标为指导。这是 Speedometer 基准测试，或者任何主要浏览器基准测试，首次通过跨行业协作开发，得到每个主要浏览器引擎（Blink、Gecko 和 WebKit）的支持。共同合作意味着我们可以建立对优化重点的共识，并便于对基准测试本身进行广泛审查：这使它成为改善整个 Web 的更强有力的工具。

并且我们看到了成果：[2023 年 Firefox 对真实用户的速度有所提升](https://hacks.mozilla.org/2023/10/down-and-to-the-right-firefox-got-faster-for-real-users-in-2023/)，直接归功于[优化](https://hacks.mozilla.org/2023/09/faster-vue-js-execution-in-firefox/) [Speedometer 3](https://spidermonkey.dev/blog/2023/11/27/newsletter-firefox-118-121.html)。这需要许多团队的协作：理解真实世界的网站，构建新工具来推动优化，并在 Gecko 内部进行大量改进，以使 Firefox 用户的网页运行更加流畅。在这个过程中，我们已经发布了数百个 bug 修复，涉及 JS、DOM、布局、CSS、图形、前端、内存分配、基于 profile 的优化等等。

我们很高兴看到所有主要浏览器引擎中的核心优化转化为真实用户的提升响应速度，并期待继续共同努力，构建改善 Web 性能测试的工具。
