<!--yml
category: 未分类
date: 2024-05-27 14:45:13
-->

# Release v2.5.0 · arp242/goatcounter · GitHub

> 来源：[https://github.com/arp242/goatcounter/releases/tag/v2.5.0](https://github.com/arp242/goatcounter/releases/tag/v2.5.0)

Use navigator.sendBeacon by default in count.js. This will allow using click events on things like PDF files and external URLs:

```
<a href="file.pdf" data-goatcounter-event="file.pdf">
<a href="http://example.com" data-goatcounter-event="ext-example.com"> 
```