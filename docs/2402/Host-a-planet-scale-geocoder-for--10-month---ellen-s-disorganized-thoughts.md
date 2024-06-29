<!--yml

category: 未分类

date: 2024-05-27 14:57:37

-->

# Host a planet-scale geocoder for $10/month — ellen's disorganized thoughts

> 来源：[https://blog.ellenhp.me/host-a-planet-scale-geocoder-for-10-month](https://blog.ellenhp.me/host-a-planet-scale-geocoder-for-10-month)

## Host a planet-scale geocoder for $10/month

February 16, 2024

大约一个月前，我开始了一个[新的地理编码器](https://github.com/ellenhp/airmail)（[演示](https://airmail.rs/#demo-section)），或者说是一个关于地点和地址的搜索引擎。我想做一个非常廉价的运行项目。主持一个全球规模的[headway实例](https://github.com/headwaymaps/headway)的一个重要障碍是地理编码器。目前我们正在使用Pelias，它在它的领域做得很好，但是运行在不到8GB RAM上的ElasticSearch表现不佳。多年来我一直在探索这个问题，没想到能够让事情迅速进展起来。

我能够基于[nom](https://crates.io/crates/nom)拼凑出一个中等水平的地址解析器，从[Pelias parser](https://github.com/pelias/parser/)汲取了大量灵感。拥有了一个尚可的地址解析器后，我转向[tantivy](https://github.com/quickwit-oss/tantivy)作为搜索引擎，想利用它的内存映射搜索索引的能力。经过一番挖掘，我找到了可以在浏览器中运行的[tantivy-wasm](https://github.com/phiresky/tantivy-wasm)，它通过范围查询获取索引的片段。当我看到这些时，我的脑海中开始转动。我不想分叉一个搜索引擎库，因此使用[匿名内存映射](https://en.wikipedia.org/wiki/Mmap)和[userfaultfd](https://docs.kernel.org/admin-guide/mm/userfaultfd.html)为主线tantivy实现了自己的后端存储，通过范围查询按需从对象存储中获取索引的块。这样做奏效了，经过一些调优，延迟已经降到了相当可接受的范围内，通常在1-3秒之间，甚至在缓存热的情况下可以看到2毫秒的简单查询速度。

[Fly.io](https://fly.io/)每月向我收取大约$3的运行费用，并且在对象存储中每月收取将近$7，所以这是一个非常经济的项目。对于这个价格，我对结果感到非常满意。我计划在未来几个月内扩展它，以处理更多的语言环境和使用场景。我做了一个[演示网站，你可以在这里试用它](https://airmail.rs/#demo-section)，但不要期望奇迹。你得到的正是你付出的，而且我没有在演示网站中索引OpenAddresses，所以如果某些地方不在OpenStreetMap中，我肯定也没有它在索引中。代码都是[开源的](https://github.com/ellenhp/airmail/)。

Cheers ✨✨
