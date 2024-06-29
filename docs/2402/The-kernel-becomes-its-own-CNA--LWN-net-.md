<!--yml

category: 未分类

日期：2024-05-27 14:51:40

-->

# 内核成为了自己的 CVE 编号机构 [LWN.net]

> 来源：[https://lwn.net/Articles/961961/](https://lwn.net/Articles/961961/)

# 内核成为了自己的 CVE 编号机构（CNA）。

[由 corbet 于 2024 年 2 月 13 日发布]

格雷格·克罗哈-哈特曼（Greg Kroah-Hartman）已经

[宣布](http://www.kroah.com/log/blog/2024/02/13/linux-is-a-cna/)

内核项目已经被接受为 CVE 编号机构（CNA）。内核将处理 CVE 编号的方式描述在

[此文档补丁](/ml/linux-kernel/2024021314-unwelcome-shrill-690e@gregkh/)

：

> 作为正常的稳定发布流程的一部分，由负责 CVE 编号分配的开发者识别的可能是安全问题的内核更改，会自动分配 CVE 编号。这些分配会作为公告频繁发布在 linux-cve 邮件列表上。
> 
> 注意，由于 Linux 内核所处的系统层级，几乎任何 bug 都可能被利用来 compromise 内核的安全性，但在修复 bug 时通常无法明显看出其被利用的可能性。因此，CVE 分配团队非常谨慎，并为他们识别的任何 bug 分配 CVE 编号。这解释了 Linux 内核团队发布的似乎很多的 CVE 数量。

* * *

（

[登录](https://lwn.net/Login/?target=/Articles/961961/)

发布评论）
