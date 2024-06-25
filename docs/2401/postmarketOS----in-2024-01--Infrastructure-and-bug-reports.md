<!--yml

类别：未分类

日期：2024-05-27 14:35:56

-->

# postmarketOS // 在 2024-01：基础设施和错误报告

> 来源：[https://postmarketos.org/blog/2024/01/08/infrastructure-and-testing/](https://postmarketos.org/blog/2024/01/08/infrastructure-and-testing/)

又是新的一年，当然这意味着我们都在对我们的项目有着宏伟的想法。postmarketOS 也不例外，在追求透明和反思的过程中，我们受到启发，写下了这篇关于计划和最近发生的事情的博客文章。

首先，正如你可能知道的，我们最近[在 OpenCollective 上发布](https://opencollective.com/postmarketos)，哇，反馈真的是惊人的。我们核心团队绝对没有预料到会有这么多慷慨，我们正在努力寻找最佳方法利用这笔钱来改善 postmarketOS 的可持续性和可靠性。OpenCollective 让我们可以轻松管理开支，为人们的工作支付报酬，并接受捐赠。一切都是100%透明的。我们对这给我们带来的自主权非常兴奋。

我们最近的基础设施遇到了一些困难。因此，我们决定利用一些捐赠的资金投资于一台更大、更好、更可靠的服务器，并开始迁移服务。到了一月底，主要的瓶颈应该会得到解决（我们正在幕后努力工作），基础设施应该会恢复良好并快速运行。

在未来，我们希望将我们的软件包存储库和设备镜像完全与其他服务分开，以最小化*未经计划的停机*的风险。

我们最终还会获得第二台专用服务器，用作 CI runner。我们目前受益于 GitLab.com 的（“面向开源项目的免费高级版”），没有这个，postmarketOS 的运行就不可能。虽然我们对 GitLab 的慷慨表示非常感激，但如果我们不依赖它会更好，因为 GitLab 可能随时决定以后不再那么慷慨。因此，我们将继续减少对这种基础设施的依赖。

此外，我们希望这些变化将使我们能够继续支持更多的主流和社区设备 - 我们提供预构建镜像 - 并为我们心中的其他一些实验留出空间。

测试是目前阻碍我们前进的一件事。我们支持如此多的设备，有如此多不同的配置，以至于对一些更核心的组件（比如 initramfs）进行更改是非常困难的。测试团队[你可以在这里加入](https://wiki.postmarketos.org/wiki/Testing_Team)给了很大的帮助，但可以理解的是，人们可能不愿意测试潜在的风险变更，或者一些更无聊的日常事务。提供一种好的方法来测试一些特性也是有挑战性的（即如果它不是在正常使用情况下触发的代码路径）。

撰写测试指南总是有用的，但我们完全可以做得更好 - 我们已经有了出色的 `pmbootstrap ci` 命令，它可以让您在本地运行大多数仓库的 CI，并且我们计划最终拥有适当的自动化测试，这将使我们能够适当地验证许多最重要的事情。

十二月份出人意料地有趣，考虑到这个时间和每个人似乎都准备好了过节。

+   我们发布了稳定的 v23.12 版本，基于 Alpine 3.19。请注意，这里的大多数其他更改都未包含在此稳定版本中，但可以在我们的滚动边缘频道中使用。

+   postmarketOS 团队增加了一个成员，[Nikita Travkin](https://gitlab.com/TravMurav) 成为第一个通过 [新流程](https://postmarketos.org/blog/2023/12/03/how-to-become-tc/) 被选为可信贡献者。

+   postmarketOS 增加了对 5 个新设备的支持，以及一个新的通用 x86_64 设备，适用于您喜爱的所有 x86 平板电脑，甚至是笔记本电脑或常规个人电脑。特别提及 Quartz64，它于 11 月底获得支持。

+   我们发布了 [logbookd](https://git.sr.ht/~martijnbraam/logbookd)，一个非常可爱的日志记录实现，实际上将日志保存到存储中（！！！），因此您可以检索以前启动时的日志。它还添加了非常需要的彩色输出，允许进行一些（稍微模糊的）服务过滤，并且默认包含内核日志。感谢 Martijn。

    +   不久之后，我们 [修复了](https://gitlab.com/postmarketOS/pmaports/-/commit/7efcec020d35491fae66e65c3bcad6e7c9d83638) 一个问题，其中 syslog 实际上未启用，导致 logbookd 和 openrc-syslog 争相竞争，哎呀！

+   我们最终在 SDM845 上发布了传感器，结果发现 OnePlus 6T 的加速度计颠倒了，是的，这是一个相当严重的测试疏忽，让这个问题通过了，绝对是一个教训。

+   CODEOWNERS 文件已经进行了清理，并获得了自己的 CI，这将帮助我们保持其良好状态，我们最终希望对大多数设备都有它，以便将正确的人通知他们设备的更改。感谢 Luca。

+   postmarketOS 艺术包 [已更新](https://gitlab.com/postmarketOS/pmaports/-/merge_requests/4614)，以正确显示在 GNOME 壁纸设置中。感谢 Pablo。

+   ARMv7 Tegra 设备全部转移到了一个通用的设备包中，依赖于 U-Boot 来选择正确的 DTB。出色的工作 Svyatoslav (@Clamor)。

+   cgroups 现在默认已经启用，以简化使用像 Docker 或许多其他 cgroups 有用的东西的服务。感谢 Clayton。

+   Unl0kr 已经通过一个解决方案升级，以修复在物理键盘上快速输入时丢失输入的问题。感谢 Jane 和 Ollie [实现了这个修复](https://github.com/calebccff/lv_drivers/pull/5)。

+   systemd-boot 已经通过一个重要的补丁升级，该补丁已经上游合并，并且还有其他改进。感谢 Clayton。

+   大量的 [Lomiri](https://lomiri.com/) 包已经开始[进入 Alpine](https://pkgs.alpinelinux.org/packages?name=*lomiri*&branch=edge&repo=&arch=aarch64&maintainer=)，这让我们离在 postmarketOS 上使用 Lomiri 更近了一步。这要归功于长期以来 [Luca Weiss](https://fosstodon.org/@z3ntu) 的努力，最近又有了 [@Justsoup](https://mstdn.social/@justsoup) 的加入。

目前在流水线上有一些非常不错的改进，请随时查看并测试它们（仅在它们要求测试时！）- 但请尽量不要打扰开发人员。顺序不分先后：

自从我们做了 YYYY-MM 的 postmarketOS 博客以来已经有一段时间了。这是一个尝试将它们重新带回来。如果你喜欢阅读这篇文章，请告诉我们。根据反馈（以及需要完成的其他工作的数量），我们可能会再次更加定期地写博客。

如果你欣赏我们在 postmarketOS 上的工作，并想支持我们，请考虑[加入我们的 OpenCollective](https://opencollective.com/postmarketos)。
