<!--yml

分类：未分类

日期：2024-05-27 12:58:22

-->

# 发布 v4.6.0：终结 · pivpn/pivpn · GitHub

> 来源：[https://github.com/pivpn/pivpn/releases/tag/v4.6.0](https://github.com/pivpn/pivpn/releases/tag/v4.6.0)

大家好，

是时候说再见了。

这是 PiVPN 的最后一个官方版本发布。

我继承了这个项目，来自 [@0-kaladin](https://github.com/0-kaladin) 和 [@redfast00](https://github.com/redfast00)，他们已经继续了他们的生活。我将其作为自己的项目进行维护，并在我无法在场时，借助 [@orazioedoardo](https://github.com/orazioedoardo) 的帮助，他非常了不起。他在我不在时承担了项目的重任并使其继续运行，他也为此项目贡献了很多！

现在是我也要继续前进的时候了！

我对 PiVPN 的关注越来越少，继续保持它的愿望也不再像过去那样强烈。当 PiVPN 创建时，它填补了一个巨大的空白，有着明确的使命和目标，我觉得这些已经实现了！我们从 OpenVPN 这种难以设置和复杂管理的工具，转变为 WireGuard 可以运行在任何设备上且易于设置。现在市面上有许多工具做得比 PiVPN 更好，我真诚地认为 PiVPN 的使命已经完成，不再具有现实意义。自然界的一切都有始有终，这就是 PiVPN 结束旅程的方式。

PiVPN 是你们许多人的家，从 Linux、bash、开源开始，每个人一直都非常欢迎，就像我[7年前](https://github.com/pivpn/pivpn/commit/fbec57d1fda70341394bfd2bc90e1dab0af2c125)开始的那样。我无法表达我对所有[84位贡献者](https://github.com/pivpn/pivpn/graphs/contributors)为这个项目做出的贡献有多感激。

这是一段波澜壮阔的旅程，我从 PiVPN 和每一个人身上学到了很多！

谢谢大家！！

PiVPN 仓库将被存档并设置为只读，将不再进行维护。除非 [@0-kaladin](https://github.com/0-kaladin) 重新回来并决定改变这一情况。

PiVPN 网站及其文档托管在 GitHub 上，因此它将在 pivpn.io 域名下保持可访问性，只要 [@0-kaladin](https://github.com/0-kaladin) 继续支付费用，我也将尽可能继续托管安装的重定向。我仍将提交一些更新文档的提交来说明项目的状态，但仅此而已。

我将继续拥有这个仓库的所有权，但我不会把它交给别人。首先，我觉得不应该由我来决定将这个项目交给谁，其次，也没有其他适合接手这个项目的人选。

"但我想并且能够维护它，我可以接手吗？" 让我直截了当地说：不行！我不认识你，我不信任你！你可以 fork 它并继续！

关于这个版本，以下是它带来的新内容：

### 新功能

+   添加在无人值守安装中使用Pi-hole的可能性（[#1825](https://github.com/pivpn/pivpn/pull/1825)）

### Bug修复和重构

+   更新子网生成和客户端创建

    +   refactor(core): 允许任何子网和网络掩码

    +   fix(scripts): 防止超出子网允许的客户端数量

    +   fix(scripts): 正确删除ipv6四位组中的前导零

    +   refactor(core): 新的概率子网生成，可以回退到其他RFC1918子网

**完整变更日志**: [`v4.5.0...v4.6.0`](https://github.com/pivpn/pivpn/compare/v4.5.0...v4.6.0)

再次，非常感谢大家的一切！希望再见！

4s3ti
