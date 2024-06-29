<!--yml

category: 未分类

date: 2024-05-27 13:24:58

-->

# 谷歌（及其他）请使用“ping”或“navigator.sendBeacon”实现链接分析 | 作者：Christian Bodt | 2024年4月 | Medium

> 来源：[https://medium.com/@sirk390/google-and-others-please-implement-link-analytics-with-ping-or-navigator-sendbeacon-538508491900](https://medium.com/@sirk390/google-and-others-please-implement-link-analytics-with-ping-or-navigator-sendbeacon-538508491900)

# 谷歌（及其他）请使用“ping”或“navigator.sendBeacon”实现链接分析

您是否注意到在网站上悬停在某些外部链接时，链接不会直接重定向到外部站点，而是转到内部重定向链接？

谷歌新闻上的重定向链接

这些链接在重定向用户之前获取用户点击的分析数据。

例如，让我们看看谷歌新闻。

**谷歌新闻**

在我的电脑上，对于谷歌新闻，点击此链接并重定向到目标需要**1.6到2.7秒**（三次试验）。它还下载了约**2.7Mb 的 HTML、JavaScript 和 CSS**。大部分由服务工作线程缓存，因此您无需等待网络，但代码执行会显著减慢重定向速度。

点击谷歌新闻文章时的 Firefox 性能分析。

这是在等待重定向时花费的大量时间。特别是在这种最佳情况下。如果您的网络连接不好，时间可能会更长。

桌面上的谷歌搜索在这里表现得更好，使用了“ping”属性。

**ping 属性**

`ping` 属性是在 HTML5 早期引入的。由于隐私问题，最初经过了激烈的讨论并被移除，但现在在 HTML 生存标准中重新回归。

HTML 中的 `ping` 属性用于在不干扰用户导航的情况下跟踪点击分析数据。

下面是一个例子：

```
<a href="https://www.example.com/external-link" 
   ping="https://www.example.com/record-analytics">
```

如您所见，它非常容易使用，但有一个小缺点：它在大多数浏览器上都能运行，除了 Firefox。由于隐私原因，Firefox 没有实现它，您可以在下面的“Can I Use”截图中看到。

[https://caniuse.com/ping](https://caniuse.com/ping)

**替代方案：使用 navigator.sendBeacon()**

另一个选择是使用 `[navigator.sendBeacon](https://developer.mozilla.org/en-US/docs/Web/API/Navigator/sendBeacon)` 这个 JavaScript 函数可以在导航变化时继续在后台进行 HTTP 请求。

所有主要浏览器（包括 Firefox）都支持它。缺点是实现起来比较复杂。

**主要网站**

许多主要网站仍然使用重定向链接进行跟踪，尽管有像 `ping` 属性或 `navigator.sendBeacon` 这样的新技术。以下是一些其他网站的示例：

+   移动端的谷歌搜索和谷歌图片（Chrome）

+   Youtube（描述或评论链接）

+   推特（t.co 链接）

还有可能有很多（如果您知道一些，请发表评论，我将在这里添加它们）。

所以，请为谷歌和其他公司的开发者们节省那几百毫秒，有时甚至是3秒或更多，当我们点击链接时，实现`ping`或`navigator.sendBeacon`来进行异步分析。
