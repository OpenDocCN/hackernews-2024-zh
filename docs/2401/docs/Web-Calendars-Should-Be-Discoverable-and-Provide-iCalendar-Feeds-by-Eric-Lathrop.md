<!--yml
category: 未分类
date: 2024-05-27 15:11:59
-->

# Web Calendars Should Be Discoverable and Provide iCalendar Feeds by Eric Lathrop

> 来源：[https://ericlathrop.com/2024/01/web-calendars-should-be-discoverable-and-provide-icalendar-feeds/](https://ericlathrop.com/2024/01/web-calendars-should-be-discoverable-and-provide-icalendar-feeds/)

<main>

Many web sites have lists of events, which really form a calendar. Who wants to remember to check a bunch of random web sites for calendars? I'd rather subscribe to those calendars the same way [I use RSS to subscribe to lists of articles](https://ericlathrop.com/2020/12/rediscovering-rss/) to read with [my RSS reader](https://miniflux.app/).

In order to facilitate subscribing to calendars, web sites should publish an [iCalendar](https://en.wikipedia.org/wiki/ICalendar) version of their event list. One missing piece is autodiscovery. Similar to [RSS autodiscovery](https://www.rssboard.org/rss-autodiscovery), I propose we use a `link` tag to provide a way for web browsers and calendar software to find iCalendar feeds:

```
<link
  rel="alternate"
  type="text/calendar"
  title="Example Calendar"
  href="https://example.com/calendar.ics"> 
```

This would let your browser display a button to import the calendar into your machine's configured calendar software. It would also let you paste web page URLs into your calendar software, and the software could discover the iCalendar feed and import it automatically.

</main>