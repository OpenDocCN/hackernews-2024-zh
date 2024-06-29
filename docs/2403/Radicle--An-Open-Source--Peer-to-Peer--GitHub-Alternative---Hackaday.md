<!--yml

category: 未分类

date: 2024-05-27 15:02:13

-->

# Radicle：一个开源的点对点GitHub替代品 | Hackaday

> 来源：[https://hackaday.com/2024/03/16/radicle-an-open-source-peer-to-peer-github-alternative/](https://hackaday.com/2024/03/16/radicle-an-open-source-peer-to-peer-github-alternative/)

最近，某些大型社交网络的行为突显了少数人对群众具有重大影响力，并且有时这种权力被滥用。因此，联邦化（或去中心化）服务的发展激增，例如Mastodon和Matrix。但是关于开发呢？虽然GitHub和类似服务不太可能被用于政治操纵，但它们仍然是具有共同故障点的集中式服务。[Radicle是一个基于Git的开源点对点协作堆栈](https://radicle.xyz)，但其背后采用公钥加密作为标准，并且使用[八卦协议](https://en.wikipedia.org/wiki/Gossip_protocol)确保网络范围内的广泛数据共享，从而具有一定的容错能力。

本质上，代码及其关联文档通过身份加密保护。[Git协议](https://git-scm.com/docs/protocol-v2)用于对等端到端的实际数据传输，这意味着更新仅以增量形式发送，而不是完整副本，从而最大化通道带宽效率。自定义的八卦协议用于在对等网络中传输元数据。该项目秉承本地优先的理念，用户在其硬件上运行完整节点，所有功能都可用，即使离线，对于经常在网络访问不稳定的位置使用笔记本电脑的用户来说非常方便。

根据其Zulipchat实例来看，这是一个非常活跃的空间，也许值得深入了解并看看是否适合您。想要加入Fediverse，但只有一台空闲的MS-DOS机器来尝试吗？[我们已经覆盖了](https://hackaday.com/2023/08/09/ms-dos-meets-the-fediverse/)。想使用Git但不在线？[您需要一个私人Git服务器](https://hackaday.com/2018/06/27/keep-it-close-a-private-git-server-crash-course/)。最后，Git用得太多了？[不如试试Gitless](https://hackaday.com/2023/06/18/too-much-git-try-gitless/)。

感谢[匿名者]的提示！不，这对我们来说并不陌生 :D
