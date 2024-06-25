<!--yml

类别：未分类

时间：2024年05月27日14:53:35

-->

# 大脑转储：在联合广播上的ompromised证书

> 来源：[https://www.hezmatt.org/~mpalmer/blog/2024/01/16/pwned-certificates-on-the-fediverse.html](https://www.hezmatt.org/~mpalmer/blog/2024/01/16/pwned-certificates-on-the-fediverse.html)

发布日期：2024年1月16日星期二 | [固定链接](//www.hezmatt.org/~mpalmer/blog/2024/01/16/pwned-certificates-on-the-fediverse.html) | [无评论](//www.hezmatt.org/~mpalmer/blog/2024/01/16/pwned-certificates-on-the-fediverse.html#comments)

除了[ompromised keys](https://pwnedkeys.com)的收集和分发之外，[pwnedkeys](https://pwnedkeys.com)项目还将这些ompromised keys与颁发的SSL证书进行匹配。我很高兴地宣布，从2024年初开始，所有匹配的证书现在都正在通过[联合广播](https://botsin.space/@pwnedcerts)发布，感谢[botsin.space](https://botsin.space/)Mastodon服务器。

想知道哪些网站容易受到拦截和干扰，实时或准实时？您是否渴望知道是谁向在公开场合发布其私钥的人颁发证书？现在你可以了。

# 工作原理

发布ompromised certs的过程大致如下：

1.  [证书透明度](https://certificate.transparency.dev)（CT）日志中的所有证书都被汇集起来（使用我的[scrape-ct-log](https://github.com/mpalmer/scrape-ct-log)工具，这是西部最快的日志抓取工具！），并且每个证书的公钥指纹都被存储在一个[LMDB](https://lmdb.tech/)数据文件中。

1.  当新的私钥被确定为已被ompromised时，该密钥的指纹将与所有LMDB文件进行比对，这些文件将密钥指纹映射到证书（实际上是到CT日志条目ID，从中检索证书本身）。

1.  如果发现了一个或多个匹配项，则使用被ompromised key的证书会被转发给“tooter”，其[将它们发布供全世界赞叹](https://botsin.space/@pwnedcerts)。

这听起来似乎非常简单，实际上也是如此……在理论上。技巧在于优化流水线，使得每天大约五百万个新证书能够在我拥有的这台略显中年的服务器上得到索引，而不会被积压。

# 为什么不直接撤销证书？

关于这个有趣的故事…

我曾经通知CA有关使用ompromised keys颁发的证书，这导致他们不得不撤销相关证书。然而，有几家CA不喜欢撤销所有这些证书，因为这会花费他们的员工时间（从而花费钱财）。他们甚至走得更远，改变了他们的程序，不再接受标准的问题报告方式（通过电子邮件发送ompromised的一般性声明），而是要求进行特定于CA的跳跃来通知他们ompromised keys。

由于 WebPKI 中吊销的有效性最多只能称为“顺势疗法”，我决定不再费心与那些只想刁难的 CA 打击莫尔游戏，并停止向 CA 发送受损密钥通知。相反，我现在将受损证书的详细信息发布给所有人，这样用户就可以直接保护自己，如果他们选择这样做的话。

# 进一步的工作

你们中的聪明人可能已经注意到，在上述“工作原理”描述中，我的扫描覆盖范围有一点空白。CA 可以（而且确实会！）为已经受损的密钥颁发证书，包括已知十年以上的“弱”密钥（[1](https://bugzilla.mozilla.org/show_bug.cgi?id=1472052)，[2](https://bugzilla.mozilla.org/show_bug.cgi?id=1620772)，[3](https://bugzilla.mozilla.org/show_bug.cgi?id=1789521)）。然而，按照当前的实现方式，pwnedkeys 证书检查器不能自动找到这样的证书。

我的计划是扩展 CT 抓取/证书处理流水线，检查所有传入的证书是否与现有的（超过 2M 个）已被入侵密钥集合相匹配。不过，每天有超过 五百万个新证书需要检查，这并不一定像“只需对每个新证书点击 [pwnedkeys API](https://pwnedkeys.com/api/v1.html)”那样简单。可怜的老 API 服务器可能不会喜欢那样。

# 支持我的工作

如果您希望这个额外的匹配发生得更快一些，我已经设置了一个 [ko-fi 支持者页面](https://ko-fi.com/tobermorytech)，您可以通过 [给我买杯提神的饮料](https://ko-fi.com/tobermorytech) 来支持我的 [pwnedkeys](https://pwnedkeys.com) 和其他我参与的开源软件和项目。我会非常感激，并且您的支持让我知道我应该用我积累的大量受损密钥数据库做更有趣的事情。

* * *
