<!--yml

类别：未分类

日期：2024-05-29 13:19:29

-->

# 我认为最便宜的APC Back-UPS单位除了在Windows中之外不能在其他系统监控 - 持续的挣扎

> 来源：[https://strugglers.net/~andy/blog/2024/02/23/i-dont-think-the-cheapest-apc-back-ups-units-can-be-monitored-except-in-windows/](https://strugglers.net/~andy/blog/2024/02/23/i-dont-think-the-cheapest-apc-back-ups-units-can-be-monitored-except-in-windows/)
> 
> TL;DR：尽管其他方面似乎工作正常，但我在Linux下无法监控BX1600MI Back-UPS而不看到持续不断的虚假电池拆卸/重新连接和电源故障/恢复事件，每个事件持续时间不到2秒。我尝试过多台计算机和多个同款UPS。在他们自己的专有Windows软件中不会发生这种情况，所以我认为他们已经改变了协议。

除了将近二十年前免费得到一台之外，我在家从未费心买过UPS。我们的电网非常可靠。根据“[uptimed](https://github.com/rpodgorny/uptimed)”的可用性信息，我家里的文件服务器在过去14年里有99.97%的时间处于开启状态。这包括搬家期间以及厨房重新装修期间电力中断了几个小时的那一天！

然而，在2023年12月，我们的电烤箱出现故障，使插座断开，导致所有设备突然断电。我的文件服务器受到了严重影响，一块硬盘损坏了。虽然它有冗余文件系统，但我并不喜欢这种情况。

我决定可以负担得起一个相对便宜的UPS。

我做了一些研究，并阅读了一些关于[APC Back-UPS系列](https://www.apc.com/uk/en/product/BX1600MI/apc-backups-1600va-tower-230v-6x-iec-c13-outlets-avr/?%3Frange=61883-backups&selectedNodeId=27590290410)的评论，它们是他们最便宜的产品。许多人对它们不屑一顾，称它们是廉价的垃圾，塑料结构脆弱，电池不可更换。但没有迹象表明这样的型号不会工作，我觉得很难在这里支付大笔费用。

我找到了YouTube视频，展示了技术人员在3到5年内更换电池的过程。自己操作会使保修无效，但无论如何，3年后保修就到期了。对于像我这样避免硬件的人来说，这看起来相当可行。

对我来说，能够由Linux计算机监控UPS非常重要。这里的整个重点在于计算机能够检测电池接近耗尽并优雅地自行关闭。Linux有两个主要选择：[**apcupsd**](http://www.apcupsd.org/) 和 [Network UPS Tools](https://networkupstools.org/index.html)（“**nut**“）。

查看Back-UPS BX1600MI型号，它有一个USB端口用于监控，并称可以使用APC自己的Powerchute Serial Shutdown Windows软件进行监控。在**nut**的硬件兼容性列表中有一条“Back-UPS（USB）”的条目，说明“根据公开可用的协议支持”。我已经下单购买。

UPS在作为不间断电源方面表现如预期的那样工作。不过，使用**nut**与其进行通信非常困难。**nut**一直报告失去通信。

我尝试使用**apcupsd**。它保持连接，但显示一连串电池分离/重新连接以及电源故障/恢复事件，每次不到2秒。通常在停电时，你期望UPS本身有视觉和听觉警报，但我没有收到任何信息，不过我不知道这是否因为这些事件太短暂了。

我联系了APC支持，但他们非常迅速地告诉我，他们不支持除了他们自己的Windows专用Powerchute Serial Shutdown（PCSS）之外的任何其他软件。

我随后在**apcupsd**邮件列表上询问了这个问题。第一个回复如下：

> “你的UPS出了问题，很可能是电池坏了，但既然你说UPS是全新的，那就换一个新的吧。”

由于这个东西是全新的，我不打算通过APC进行保修索赔。我只是联系了供应商，告诉他们我认为它有问题，想要退货。他们实际上提供了先送我另一个新的，我再寄回手头的那个，所以我选择了这个方案。

与此同时，我抽空安装了Windows 10虚拟机并将USB传递给它。猜猜看？在Windows的PCSS中没有出现任何偶发事件。当我拔掉电源等操作时，它检测到了预期的事件。我没有证据表明UPS在任何方面有问题。你可能已经猜到接下来会发生什么了。

替换的UPS（同型号）表现完全相同：出现偶发事件。这似乎是APC Back-UPS在非Windows系统上的常见问题。

回到我在**apcupsd**邮件列表上的主题，我再次询问是否有人能够在非Windows系统上使用这个设备。到目前为止，我唯一得到的实质性回复是：

> “BX系列就是廉价的塑料垃圾，最糟糕的是，甚至连BExx0家族也没有这么糟糕——施耐德直接回应所有充斥市场的中国垃圾[...]没有理智的人会购买这些东西，但是，我们就是在这里。”

就我所知，目前不能从非Windows系统监控Back-UPS型号。这将是我的工作理论，除非有人能联系我告诉我我错了，我很想知道这件事。我觉得我已经尽力通过询问用于在Unix系统上监控APC UPS的软件邮件列表来找到这样的人。

在与供应商讨论后，他们推荐了一款[Riello NPW 1.5kVA](https://www.riello-ups.co.uk/products/1-ups/63-net-power)，该产品[在**nut**的支持列表中完全支持](https://networkupstools.org/stable-hcl.html?manufacturer=Riello&model=NPW%20600/800/1000/1500/2000)。他们将接收APC设备并全额退款；Riello的价格比APC贵约30英镑。
