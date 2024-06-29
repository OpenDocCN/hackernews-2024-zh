<!--yml

category: 未分类

date: 2024-05-27 14:51:38

-->

# 隐身明处：介绍WebTunnel | Tor项目

> 来源：[https://blog.torproject.org/introducing-webtunnel-evading-censorship-by-hiding-in-plain-sight/](https://blog.torproject.org/introducing-webtunnel-evading-censorship-by-hiding-in-plain-sight/)

今天，3月12日，在[世界反网络审查日](https://en.wikipedia.org/wiki/World_Day_Against_Cyber_Censorship)，Tor项目的反审查团队很高兴正式宣布发布WebTunnel，这是一种新型Tor桥梁，旨在帮助处于严重审查地区的用户连接到Tor网络。现在可以在Tor浏览器的稳定版本中使用WebTunnel，它加入了由Tor项目开发和维护的绕过审查技术系列。

不同类型的桥梁的发展对于使Tor更具抗审查能力，并在动态和不断变化的审查景观中领先对手至关重要。尤其是在我们正在经历的2024年全球选举超周期中，[绕过审查技术在捍卫互联网自由方面变得至关重要](https://blog.torproject.org/2024-defend-internet-freedom-during-elections/)。

如果您曾考虑过成为Tor桥梁运营商来帮助他人连接Tor，现在是一个绝佳的时机！您可以在[Tor社区门户](https://community.torproject.org/relay/setup/webtunnel/)找到运行WebTunnel桥梁的要求和说明。

## 什么是WebTunnel以及它如何工作？

WebTunnel是一个抗审查的可插拔传输，旨在模仿加密的Web流量（HTTPS），灵感来源于[HTTPT](https://www.usenix.org/conference/foci20/presentation/frolov)。它通过将有效负载连接包装成类似于WebSocket的HTTPS连接来工作，对于网络观察者来说，看起来就像是普通的HTTPS（WebSocket）连接。因此，对于不知道隐藏路径的旁观者来说，它看起来就像是向网页服务器发起普通HTTP连接，给用户的印象就是他们只是在浏览网页。

实际上，WebTunnel与普通的Web流量非常相似，以至于可以与同一网络端点上的网站共存，即相同的域名、IP地址和端口。这种共存允许标准流量反向代理将普通Web流量和WebTunnel分别转发到它们的应用服务器。因此，当有人尝试访问共享网络地址上的网站时，他们只会感知到该网站地址的内容，而不会注意到存在一个秘密桥梁（WebTunnel）。

## 比较WebTunnel和obfs4桥梁

对于大多数 Tor 浏览器用户，WebTunnel 可作为 obfs4 的替代方案。尽管 obfs4 和其他完全加密的流量旨在完全区分和难以识别，但 WebTunnel 通过模仿已知和典型的网络流量，更有效地适用于存在协议白名单和默认拒绝网络环境的场景。

将网络流量审查机制比作一个硬币分类机，其中硬币代表流动的流量。传统上，这样的机器检查硬币是否符合已知的形状，并在符合时放行或者不符合时丢弃。在完全加密的未知流量情况下，正如在发布的研究《中国防火墙如何检测和阻止完全加密流量》（[点击此处阅读](https://www.usenix.org/conference/usenixsecurity23/presentation/wu-mingshi)）中所示，它不符合任何特定形状，则可能会受到审查。在我们的硬币类比中，硬币不仅不能匹配任何已知的被阻塞协议形状，还需要匹配一个被认可的允许形状--否则将被丢弃。obfs4 流量既不符合任何已知的允许协议，也不是文本协议，因此将被拒绝。与之相反，类似 HTTPS 流量的 WebTunnel 流量将被通过，因为 HTTPS 是允许的协议。

如果您想了解更多关于桥接的信息，包括不同设计及其工作原理，请查看我们的视频系列：[点击这里查看](https://www.youtube.com/watch?v=8mdtSgHWhXY)。

## 如何使用 WebTunnel 桥接？

### 🌉 第一步 - 获取 WebTunnel 桥接

目前，WebTunnel 桥接仅通过 Tor 项目桥接网站分发。我们计划增加更多的分发方式，如 Telegram 和 moat。

1.  使用您的常规网页浏览器访问网站：[https://bridges.torproject.org/options](https://bridges.torproject.org/options)。

1.  在“高级选项”中，从下拉菜单中选择“webtunnel”，然后点击“获取桥接”。

1.  解决验证码。

1.  复制桥接线路。

### 💻 第二步 - 下载并安装 Tor 浏览器桌面版

注意：WebTunnel 桥接不适用于旧版 Tor 浏览器（12.5.x）。

1.  下载并安装最新版本的 Tor 浏览器桌面版。

1.  打开 Tor 浏览器并转到连接首选项窗口（或点击“配置连接”）。

1.  点击“手动添加桥接”，并添加第一步中提供的桥接线路。

1.  关闭桥接对话框，点击“连接”。

1.  记录使用 WebTunnel 时遇到的任何问题或意外行为。

### 📲 或下载并安装 Tor 浏览器 Android 版

1.  下载并安装最新版本的 Tor 浏览器 Android 版。

1.  运行 Tor 浏览器并选择配置桥接选项。

1.  选择“提供我知道的桥接”，输入提供的桥接地址。

1.  点击“确定”，如果一切顺利，将会连接成功。

### ✍️ 第三步 - 与我们分享反馈

您的反馈对帮助我们识别任何问题并确保 WebTunnel 桥接的可靠性至关重要。对于生活在审查地区的用户，我们希望了解这种新桥接的性能如何与其他规避方法如 obfs4 和雪花相比。

## 感谢所有为使 WebTunnel 成为可能的志愿者们。

我们拥有的工具越多，我们就越能够精确地进行响应，阻止审查者，使数百万用户能够访问自由开放的互联网。我们于 2023 年 10 月首次[宣布这种新桥接类型，并呼吁测试者](https://forum.torproject.org/t/call-for-testers-webtunnel-a-new-way-to-bypass-censorship-with-tor-browser/9855)，请求那些可以安全使用 WebTunnel 的 Tor 用户提供反馈。许多人积极行动起来，我们收到了大量的反馈，无论是公开的还是私密的，这使我们能够对 WebTunnel 进行了许多稳定性改进。

现在，全球范围内[有 60 个 WebTunnel 桥接](https://metrics.torproject.org/rs.html#search/transport:webtunnel?fields=transports)，以及[超过 700 名日常活跃用户使用 WebTunnel](https://metrics.torproject.org/userstats-bridge-transport.html?start=2023-12-08&end=2024-03-07&transport=webtunnel)，覆盖不同平台。然而，尽管 WebTunnel 在中国和俄罗斯等地区运作良好，但目前在伊朗的某些地区仍无法正常工作。

我们的目标是确保 Tor 对每个人都有效。在导致数百万人处于风险中的地缘政治冲突中，互联网对我们进行沟通、见证和分享全球发生的事情、组织、捍卫人权以及建立团结至关重要。这就是为什么我们社区的志愿者贡献至关重要。记住，参与的方式有很多：您可以运行更多的[桥接](https://community.torproject.org/relay/setup/bridge)、[雪花代理](https://snowflake.torproject.org/)和[中继](https://community.torproject.org/relay/)，继续我们对抗审查和支持自由开放访问不受限制互联网的斗争。
