<!--yml

类别：未分类

日期：2024-05-27 13:40:21

-->

# 作为sudo替代的“run0” [LWN.net]

> 来源：[https://lwn.net/Articles/971745/](https://lwn.net/Articles/971745/)

# 作为sudo替代的“run0”

[发表于2024年4月30日，作者 corbet]

[这个Mastodon流](https://mastodon.social/@pid_eins/112353324518585654)

来自Lennart Poettering的描述

`sudo`

替代方案——称为

`run0`

——将是即将发布的systemd 256版本的一部分。它采取了一种相当不同的方式来执行特权命令，完全避免使用setuid（他称之为“SUID”）权限。

> 所以，在我理想的世界里，我们将拥有一个完全没有SUID的操作系统。让我们把SUID的概念抛弃在UNIX糟糕理念的垃圾堆中。对于需要小心手动清理的，处于普通代码控制之下的一半特权代码执行环境，不再是2024年安全工程的正确做法。

* * *

（

[登录](https://lwn.net/Login/?target=/Articles/971745/)

发表评论）
