<!--yml

category: 未分类

date: 2024-05-27 15:11:59

-->

# Web Calendars Should Be Discoverable and Provide iCalendar Feeds by Eric Lathrop

> 来源：[https://ericlathrop.com/2024/01/web-calendars-should-be-discoverable-and-provide-icalendar-feeds/](https://ericlathrop.com/2024/01/web-calendars-should-be-discoverable-and-provide-icalendar-feeds/)

<main>

许多网站都有活动列表，实际上就构成了一个日历。谁愿意记住要检查一堆随机网站的日历？我更愿意像我使用[RSS订阅文章列表](https://ericlathrop.com/2020/12/rediscovering-rss/)一样，订阅这些日历，然后用[我的RSS阅读器](https://miniflux.app/)阅读。

为了方便订阅日历，网站应该发布他们的活动列表的[iCalendar](https://en.wikipedia.org/wiki/ICalendar)版本。一个缺失的部分是自动发现。类似于[RSS自动发现](https://www.rssboard.org/rss-autodiscovery)，我建议我们使用`link`标签提供一种方式，让网络浏览器和日历软件找到iCalendar订阅：

```
<link
  rel="alternate"
  type="text/calendar"
  title="Example Calendar"
  href="https://example.com/calendar.ics"> 
```

这样可以让您的浏览器显示一个按钮，将日历导入到您机器配置的日历软件中。它还可以让您将网页URL粘贴到您的日历软件中，并且软件可以自动发现iCalendar订阅并导入它。

</main>
