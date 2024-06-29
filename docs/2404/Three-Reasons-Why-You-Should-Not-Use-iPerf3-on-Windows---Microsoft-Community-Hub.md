<!--yml

分类：未分类

日期：2024-05-27 13:20:57

-->

# 三个原因为何不应在 Windows 上使用 iPerf3 - Microsoft 社区中心

> 来源：[https://techcommunity.microsoft.com/t5/networking-blog/three-reasons-why-you-should-not-use-iperf3-on-windows/ba-p/4117876](https://techcommunity.microsoft.com/t5/networking-blog/three-reasons-why-you-should-not-use-iperf3-on-windows/ba-p/4117876)

这里是来自微软商业支持 - Windows 网络团队的 James Kehr 的文章。本文将解释为什么不应该在 Windows 上使用 iPerf3 进行合成网络基准测试和测试。接下来简要解释为什么应该使用 ntttcp 和 ctsTraffic 代替。

***更新（2024年4月22日）：根据评论中的反馈，文章中进行了各种更新标记。***

# 原因 1 – ESnet 不支持 Windows

[iPerf3](https://software.es.net/iperf/)由一个名为 ESnet（能源科学网络）的组织拥有和维护。他们不正式支持也不建议在 Windows 上使用 iPerf3。他们建议使用 iPerf2。稍后详细了解微软的建议。

这里是从官方[ESnet iPerf3 FAQ](https://software.es.net/iperf/faq.html)，于2024年4月18日检索到的一些直接引用。

***我试图在 Windows 上使用 iperf3，但遇到了问题。我该怎么办？***

*iperf3 在 Windows 上没有官方支持，但 iperf2 有。我们建议您使用 iperf2。*

***更新（2024年4月22日）：请阅读[评论](https://nam06.safelinks.protection.outlook.com/?url=https%3A%2F%2Ftechcommunity.microsoft.com%2Ft5%2Fnetworking-blog%2Fthree-reasons-why-you-should-not-use-iperf3-on-windows%2Fbc-p%2F4119600%2Fhighlight%2Ftrue%23M633&data=05%7C02%7CJames.Kehr%40microsoft.com%7C27c73f372e86443d986208dc62ecea72%7C72f988bf86f141af91ab2d7cd011db47%7C1%7C0%7C638494016911846998%7CUnknown%7CTWFpbGZsb3d8eyJWIjoiMC4wLjAwMDAiLCJQIjoiV2luMzIiLCJBTiI6Ik1haWwiLCJXVCI6Mn0%3D%7C0%7C%7C%7C&sdata=mXIpLUx6lxFC8R%2FMoMHfQM7aeNsscyXRkFLzIiQqWUw%3D&reserved=0)(s)，详细了解 iPerf2 与 iPerf3 的差异。***

并从 ESnet 的“[获取 iPerf3](https://software.es.net/iperf/obtaining.html)”文章中，于2024年4月18日检索到。

*iperf3 的主要开发是在 CentOS 7 Linux、FreeBSD 11 和 macOS 10.12 上进行的。目前，这些是唯一官方支持的平台…*

微软不推荐使用 iPerf3 的另一个原因。

# 原因 2 – iPerf3 在 Windows 上的仿真

iPerf3 不会调用 Windows 的本地 API。它只能调用 Linux/POSIX 的 API。

iPerf3 社区使用 Cygwin 作为仿真层，在 Windows 上实现 iPerf3 的功能。您可以在他们的[常见问题解答](https://www.cygwin.com/faq.html)中了解更多关于 Cygwin 的信息。

iPerf3 调用被发送到 Cygwin，由其转换为 Windows API 调用。然后 Windows 网络堆栈开始起作用。在 Windows 上，iPerf3 的维护者们做得非常好，但最终，这种方法可能会存在一些问题。

并非所有 iPerf3 的功能在 Windows 上都能正常工作。基本选项运行良好，但某些网络测试所需的高级功能可能在 Windows 上不可用或表现出意外行为。

模拟往往会导致性能损失。在对延迟敏感的操作，如网络测试中，模拟的开销可能导致吞吐量低于预期。

最后，iPerf3 使用不常见的 Windows Socket (winsock) 选项，与本地的 Windows 应用程序不同。对于通用的吞吐量测试来说这没问题，但对于应用程序测试来说，这些不常见的套接字选项无法模拟真实的 Windows 本地应用程序行为。

# 第三个原因 - 你可能在使用旧版本的 iPerf3。

***更新（2024年4月22日）：iperf.fr 不再提供旧版 Windows iPerf3 二进制文件。该网站现在链接到其他维护活跃的 iPerf3 for Windows 二进制文件的网站。非常感谢 [@Harvester](/t5/user/viewprofilepage/user-id/598981) 在评论中指出这一点，以及感谢iperf.fr团队更新了他们的网站！***

去网络上搜索“iPerf3 on Windows”。继续吧，打开一个标签页，使用你选择的搜索引擎。我相信你一定是用的 Copilot 的 Bing。

最终的结果是什么，因此最可能点击的链接是什么？我打赌这个站点是 iperf.fr。

iperf.fr 上最新的 iPerf3 版本为来自2016年6月8日的3.1.3版。在写作时已经将近8年了。

当前版本的 iPerf3，[直接来自 ESnet](https://github.com/esnet/iperf)，是 3.16 版。相比人们最有可能下载的版本，这包含了 15 个版本的修复程序、特性和更改。

此份特定的 iPerf3 复制品，来自 iperf.fr，包含一个带有缺陷的 cygwin1.dll 版本，将套接字缓冲区限制在1MB。这会导致在高速高延迟和高带宽网络上性能不佳，因为 iPerf3 无法发送足够的数据以饱和链路，从而导致测试不准确。

你应该在哪里寻找 Windows 上的 iPerf3？

从 ESnet 的文章“获取 iPerf3”中，他们说：

*Windows: 使用 [Cygwin](https://www.cygwin.com/) 构建的 Windows iperf3 二进制文件可以在多个地方找到，包括 [https://files.budman.pw/](https://files.budman.pw/)（[讨论线程](https://www.neowin.net/forum/topic/1234695-iperf/)）。*

# **微软的推荐**

微软维护两个合成网络基准测试工具：ntttcp（Windows NT Test TCP）和ctsTraffic。[ntttcp 的最新版本在 GitHub 上维护](https://github.com/microsoft/ntttcp)。这是一个使用 Windows 网络的本地工具，与本地 Windows 应用程序使用方式相同。

那么 Linux 呢？

ntttcp 也有 Linux 版本！详细信息可以在 [ntttcp for Linux GitHub](https://github.com/microsoft/ntttcp-for-linux) 存储库中找到。这是专为 Linux 构建的一个独立代码库，与 Windows 上的 ntttcp 兼容，但与 Windows 版本并非完全相同。

Ntttcp 允许在 Windows 到 Windows、Linux 到 Linux 以及 Windows 到 Linux 之间执行原生 API 合成网络测试。

[ctsTraffic](https://github.com/microsoft/ctsTraffic) 仅支持 Windows 到 Windows。ntttcp 更类似于 iPerf3，而 ctsTraffic 则具有不同的选项和目标。ctsTraffic 专注于端到端的 goodput 场景，而 ntttcp 和 iPerf3 更注重隔离网络堆栈的吞吐量。

## 您如何使用 ntttcp？

Azure 团队撰写了一篇关于 Windows 和 Linux 上基本 ntttcp 功能的优秀文章。我不认为需要重复创造轮子，因此我将简单地链接您到该文章。

[https://learn.microsoft.com/en-us/azure/virtual-network/virtual-network-bandwidth-testing?tabs=windo...](https://learn.microsoft.com/en-us/azure/virtual-network/virtual-network-bandwidth-testing?tabs=windows)

在测试 Windows 和 Linux 之间的互操作性时，已知存在一些限制。详细信息请参阅 [ntttcp for Linux GitHub 上的维基文章](https://github.com/microsoft/ntttcp-for-linux/wiki/How-to-interop-with-Windows-NTttcp%3F)。

# 测试

在准备本文时，我使用了两台 Windows Server 2022 虚拟机建立了一个实验室。测试使用了最新版本的 iPerf3（3.16）、ntttcp（5.39）和 ctsTraffic（2.0.3.3）。

默认的 iPerf3 参数是我在 Microsoft 支持客户中最常见的配置。因此，我正在调整 ntttcp 和 ctsTraffic 以更好地匹配 iPerf3 的默认单连接、128KB 缓冲区长度行为。虽然这不是一个完美的比较，但这确实使它更接近。

单流测试用于目标分析，因为许多应用程序不执行多线程传输。带宽和最大吞吐量测试应该使用大缓冲区进行多线程处理，但这是另一个话题。

如果您希望运行自己的测试，请不要忘记在 Windows Defender 防火墙上允许网络流量。

## iPerf3

iPerf3 服务器命令：

```
iperf3 -s
```

iPerf3 客户端命令：

```
iperf3 -c <IP> -t 60
```

多次测试的平均值约为 7.5 Gbps。最高结果为 8.5 Gbps，最低为 5.26 Gbps。

## ntttcp

Ntttcp 服务器命令：

```
ntttcp -r -m 1,*,<IP> -t 60
```

Ntttcp 客户端命令：

```
ntttcp -s -m 1,*,<IP> -l 128K -t 60
```

ntttcp 多次测试的平均值约为 12.75 Gbps。最高测试平均为 13.5 Gbps，最低测试为 12.5 Gbps。

Ntttcp 执行所谓的预先发布接收，这是该工具的独特功能。这减少了应用程序等待时间，作为网络堆栈隔离的一部分，使得应用程序对套接字消息的响应比正常情况更快。

*-r* 是接收端，*-s* 是发送端。

*-m* 是数值映射，即：<线程数>、<CPU 亲和性>、<目标 IP>。在此测试中，我们使用单线程，无 CPU 亲和性（*），而 -r 和 -s 两侧使用目标 IP 地址作为最终值。

*-t* 是测试时间，以秒为单位。

*-l* 设置缓冲区长度。您可以使用 K|M|G 作为 ntttcp 的简写，表示千字节、兆字节和吉字节。

## ctsTraffic

这些命令在 PowerShell 中运行，以便更轻松地阅读值。

ctsTraffic 服务器命令：

```
.\ctstraffic.exe -listen:* -Buffer:"$(128KB)" -Transfer:"$(1TB)" -ServerExitLimit:1 -consoleverbosity:1 -TimeLimit:60000
```

ctsTraffic 客户端命令：

```
.\ctstraffic.exe -target:<IP> -Connections:1 -Buffer:"$(128KB)" -Transfer:"$(1TB)" -Iterations:1 -consoleverbosity:1 -TimeLimit:60000
```

结果约为 9.2 Gbps 平均。它比 iPerf3 快一点，且更为一致，但不如 ntttcp 快。ctsTraffic 较慢的两个主要原因是数据完整性检查和使用推荐的重叠 IO 模型。这意味着 ctsTraffic 使用单一挂起接收，而不像 ntttcp 那样预先设置接收。

*-Buffer* 是缓冲区长度（ntttp：-l）。

*-Transfer* 是每次迭代发送的数据量。

*-Iterations/-ServerExitLimit* 是数据集将被传输的次数。

*-Connections* 是将使用的并发 TCP 流的数量。

*-TimeLimit* 是运行测试的毫秒数。当达到时间限制时，即使迭代传输未完成，测试也会停止。

感谢您的阅读，希望这有助于您了解 Windows 上的合成网络基准测试！
