<!--yml
category: 未分类
date: 2024-05-27 13:24:58
-->

# Google (and others) please implement link analytics with “ping” or “navigator.sendBeacon” | by Christian Bodt | Apr, 2024 | Medium

> 来源：[https://medium.com/@sirk390/google-and-others-please-implement-link-analytics-with-ping-or-navigator-sendbeacon-538508491900](https://medium.com/@sirk390/google-and-others-please-implement-link-analytics-with-ping-or-navigator-sendbeacon-538508491900)

# Google (and others) please implement link analytics with “ping” or “navigator.sendBeacon”

Did you notice that when hovering over some external links on a website, the links don’t redirect to the external site, but to an internal redirection link?

Redirect Link on Google News

These links get analytics from users’ clicks before redirecting them.

For example let’s looks at Google News.

**Google News**

On my computer for Google News, clicking this link and redirecting to the target takes between **1.6 and 2.7 seconds** (over three trials). It also downloads about **2.7Mb of HTML, javascript and CSS.** Most of it is cached by the service worker so you don’t have to wait for the network, but it code is executed and appears to slow down the redirection significantly.

Firefox Performance Analysis when clicking on a Google News article.

This is a lot of time spent just waiting for a redirection. Especially since this is the best-case scenario. It can be much longer when you have a bad internet connection.

Google Search on desktop is doing much better here, and uses the “ping” attribute.

**Ping Attribute**

The ping attribute was invented early in HTML5\. It was initially heavily debated and removed due to privacy concerns but is now back later in the HTML living standard.

The `ping` attribute in HTML is used to track clicks for analytics without disrupting the user's navigation.

Below is an example:

```
<a href="https://www.example.com/external-link" 
   ping="https://www.example.com/record-analytics">
```

As you can see, it is very easy to use but it has a small drawback: It works on most browsers except Firefox. Firefox doesn’t implement it for privacy reasons as you can see in the “Can I Use” screenshot below.

[https://caniuse.com/ping](https://caniuse.com/ping)

**Alternative: Use navigator.sendBeacon()**

Another option is to use `[navigator.sendBeacon](https://developer.mozilla.org/en-US/docs/Web/API/Navigator/sendBeacon)` This javascript function makes an HTTP request that continues in the background even when the navigation changes.

It is supported by all major browsers (including firefox). The drawback is that it is more complicated to implement.

**Major Websites**

Many major websites continue to use redirect links for tracking purposes despite newer technologies like the `ping` attribute or `navigator.sendBeacon`. Below are some other websites examples:

*   Google Search & Google Images on Mobile (Chrome)
*   Youtube (description or comment links)
*   Twitter (t.co links)

And probably many more (If you know some, please send comment and I will add them here).

So please for developers at Google and other companies, save us those hundreds of milliseconds, sometimes 3 seconds or more when we click on a link and implement `ping` or `navigator.sendBeacon` for async analytics.