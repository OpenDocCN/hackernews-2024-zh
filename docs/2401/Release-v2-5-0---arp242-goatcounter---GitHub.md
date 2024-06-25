<!--yml

分类：未分类

日期：2024-05-27 14:45:13

-->

# 发布 v2.5.0 · arp242/goatcounter · GitHub

> 来源：[`github.com/arp242/goatcounter/releases/tag/v2.5.0`](https://github.com/arp242/goatcounter/releases/tag/v2.5.0)

默认情况下在 `count.js` 中使用 `navigator.sendBeacon`。 这将允许在诸如 PDF 文件和外部网址之类的内容上使用点击事件：

```
<a href="file.pdf" data-goatcounter-event="file.pdf">
<a href="http://example.com" data-goatcounter-event="ext-example.com"> 
```
