<!--yml

分类：未分类

日期：2024-05-27 14:30:36

-->

# FOSDEM 2024 后续

> 来源：[https://genodians.org/nfeske/2024-02-15-fosdem-aftermath](https://genodians.org/nfeske/2024-02-15-fosdem-aftermath)

## FOSDEM 2024 后续

2024年2月15日 由[Norman Feske](index)

今年的FOSDEM再次是一个充满事件的经历。现在，大多数演示已经可用。但我利用这个机会重新录制了一个更完整的版本的演讲，我想与你分享。

与[Stefan](https://genodians.org/skalk)、[Johannes](https://genodians.org/jschlatow)、[Ben](https://genodians.org/atopia)和我一起，四位Genodians代表今年参加了FOSDEM。我们特别准备参与了两个开发者室。Johannes在微内核开发者室介绍了我们的[Goa SDK](https://fosdem.org/2024/schedule/event/fosdem-2024-3089-streamlining-application-development-for-genode-with-goa)，而我在[FOSS-on-mobile](https://fosdem.org/2024/schedule/track/foss-on-mobile-devices/)开发者室介绍了我们最近的[移动电话工作](https://fosdem.org/2024/schedule/event/fosdem-2024-3017-genode-on-the-pinephone-on-track-to-real-world-usability/)。

#### FOSS-on-mobile 开发者室

不幸的是，两个开发者室在周六同时进行。所以我只能参加每个会议的一半。我发现FOSS-on-mobile室呈现的各种主题非常令人愉悦。对我个人来说，FOSS-on-mobile室中[U-Boot for modern Qualcomm phones](https://fosdem.org/2024/schedule/event/fosdem-2024-1716-u-boot-for-modern-qualcomm-phones/)和[The Universal Serial Bug](https://fosdem.org/2024/schedule/event/fosdem-2024-3200-universal-serial-bug-a-tale-of-spontaneous-modem-resets/)的演示尤为突出。我希望在开始使用PinePhone之前就看到[From phone hardware to mobile Linux](https://fosdem.org/2024/schedule/event/fosdem-2024-2234-from-phone-hardware-to-mobile-linux/)的优秀讲座。关于我自己关于[Genode on the PinePhone on track to real-world usability](https://fosdem.org/2024/schedule/event/fosdem-2024-3017-genode-on-the-pinephone-on-track-to-real-world-usability/)的演讲，显然我对我想覆盖的广度话题有些过于雄心勃勃，时间不够。我准备的三个演示中，只能展示一个。

本周早些时候，我利用机会重新录制了（稍微扩展的版本），这次更多地展示了背景故事，并展示了所有预期的演示。

<https://genode.org/files/mobile_sculpt_fosdem_2024.mp4>

[下载为mp4](https://genode.org/files/mobile_sculpt_fosdem_2024.mp4)

#### 微内核开发者室

在一天的下半场，我参加了[微内核](https://fosdem.org/2024/schedule/track/microkernel/)开发室。观看乌多·斯坦伯格（Udo Steinberg）展示他的NOVA工作总是一种壮举，今年的关于[使用NOVA微超级监控程序进行大规模可信计算](https://fosdem.org/2024/schedule/event/fosdem-2024-3227-using-the-nova-microhypervisor-for-trusted-computing-at-scale/)的演讲也不例外。毫无疑问，我非常期待约翰内斯关于[使用Goa为Genode简化应用开发](https://fosdem.org/2024/schedule/event/fosdem-2024-3089-streamlining-application-development-for-genode-with-goa/)的演讲。

我必须承认，我觉得微内核开发室中关于单内核特定的讨论有点恼人。或许这听起来有些奇怪，因为我清楚地记得在2014年安蒂·坎特（Antti Kantee）关于[Rump Kernels, Just Components](https://archive.fosdem.org/2014/schedule/event/01_uk_rump_kernels/)的演讲以及2019年马丁·卢西纳（Martin Lucina）关于[Solo5 unikernel](https://archive.fosdem.org/2019/schedule/event/solo5_unikernels/)的演讲都引发了我的好奇心，并与微内核社区产生了协同效应。然而，这一次，我完全没有能够把所展示的单内核项目与微内核世界联系起来。而微内核架构关注的是责任分离，单内核却是在消除分离手段。它们的共同点似乎仅限于一些低级编程细节。我意识到上述言论可能显得有些轻蔑。这并不是我的意图，因为每个项目显然都是情感的事业，并具有技术上的优点。它们只是偶然既不是基于组件的操作系统，也不是与微内核相关的项目。

每个开发室的时间表最终取决于提交的内容。因此，重新确立微内核开发室的明确焦点的最佳方式是开源微内核项目在下次提交时提供强有力的提案。

#### 周日亮点

周日，我喜欢在K大楼周围漫步。丹尼尔·斯滕伯格（Daniel Stenberg）以一种全面而娱乐的方式声称[你也可以做出curl！](https://fosdem.org/2024/schedule/event/fosdem-2024-1931-you-too-could-have-made-curl-/)。另一个亮点当然是尼尔·瓦尔菲尔德（Neal Walfield）关于[Sequoia PGP](https://fosdem.org/2024/schedule/event/fosdem-2024-3297-sequoia-pgp-rethinking-openpgp-tooling/)的演讲，这让我感到开放PGP的最好时光仍在前方。

FOSDEM聚集了许多深深关心自由软件和开源及其各自项目共享价值观的人。作为一个特别值得记住的经历，我有幸在PinePhone上展示了Sculpt OS，并把我的[Genode平台](https://genode.org/documentation/genode-platforms-23-05.pdf)书交给了PinePhone背后的人们。Lukasz和Tl的热情和积极的反应完全出乎我的意料。

#### 其他需要记住的事情是什么？

[本](https://genodians.org/atopia/index)在选择餐馆方面很有一手，比如[这家](https://www.beiruti.eu/)。记住，旅途中有本在身边是件好事！

我想成为德国铁路的忠实粉丝，我是认真的。但今年从德累斯顿到布鲁塞尔的旅程是我经历过的最糟糕的火车之旅。人们在拥挤的ICE列车外赞美？确认。站在火车厕所旁边两个小时，吸入各种强度的气味？确认。医疗紧急停车？确认。数小时的延误？确认。光明面是，从布鲁塞尔返回德累斯顿的旅程是我能想象到的最顺畅的旅程。成功的机会是五十五十。
