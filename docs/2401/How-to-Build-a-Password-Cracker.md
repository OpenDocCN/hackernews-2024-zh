<!--yml

category: 未分类

日期：2024-05-27 15:14:57

-->

# 如何构建密码破解器

> 来源：[`www.sevnx.com/blog/post/building-a-password-cracker`](https://www.sevnx.com/blog/post/building-a-password-cracker)

因此，这里的主要要点是，增加核心 GPU 时钟的 MHz 值会带来明显的性能提升，但需要通过增加功率/电压值来保持稳定性。**在超频时请务必小心，因为如果值在错误的配置下被推得太远，实际硬件损坏可能会发生。此外，如果你在家里运行你的设备，请注意你所消耗的电量，因为根据你的插座能力，你可能会开始触发断路器（请查看上面的电源供应部分）。**

#### 软件

**操作系统（OS）** – 当涉及到首选操作系统时，像 Debian 或 Ubuntu 这样的 Linux 服务器操作系统是首选，因为它是免费的，而且可以在无头模式下运行。无头模式意味着 GPU 不必花费任何资源来渲染 GUI，而是留下这些资源让 hashcat 去充分利用，从而实现最佳性能。但如果你对 Windows 操作系统感到满意，无论是 Windows 10/11 还是 Server 2019/2022，这意味着需要额外花费一些钱购买许可证（如果是服务器操作系统，需要花费相当可观的一笔钱），你仍然可以完全运行 hashcat。你可能会遇到一些性能下降，但你可以通过一些超频和优化 Windows 来使其以最小的 GUI 运行。最近 Windows 上的驱动程序和超频支持似乎更好，但这是一个有争议的观点，因为 *nix 上的 GreenWithEnvy 超频工具和驱动程序支持最近一直很好。

**驱动程序/CUDA**：确保安装最新的驱动程序，但有时您可能会发现某些算法在旧驱动程序上性能更好，所以如果时间允许，可以尝试不同的驱动程序，但通常最新的是最好的原则适用。本博客不涵盖 *nix 上安装过程的步骤，因为这个过程根据操作系统的不同而变化。对于所有 Windows 用户来说，简单地获取最新的 Nvidia 或 AMD 驱动程序，并完成安装过程即可。对于 Nvidia CUDA，获取最新的 CUDA SDK（在撰写本博客时为 11.8.*）并完成基本安装。确保 CUDA DLL 文件 nvrtc64_*_*.dll 从 CUDA 安装的 Program Files 文件夹中传输到 C:\windows\system32\ 文件夹中，以避免在 hashcat 中出现“未检测到 CUDA SDK 工具包安装”错误。在撰写本博客时，该文件名为 nvrtc64_112_0.dll。

**超频（Overclocking）**：如果你使用的是 Windows 操作系统，有免费的超频软件套件，EVGA Precision X1 和 MSI Afterburner 非常适合新手和专业用户进行超频。Linux 上的超频可以通过 GreenWithEnvy 工具实现，但需要图形用户界面。使用命令行工具确实增加了在配置文件中出错的风险，但对于高级用户来说是一种选择。

[`www.evga.com/precisionx1/`](https://www.evga.com/precisionx1/)

[`www.msi.com/Landing/afterburner/graphics-cards`](https://www.msi.com/Landing/afterburner/graphics-cards)

[`github.com/dankamongmen/GreenWithEnvy`](https://github.com/dankamongmen/GreenWithEnvy)

**hashcat** – 在撰写本文时，hashcat 当前版本为 6.2.6。尽管还有其他免费和商业工具可用于密码恢复/密码破解，但在许多算法中，尤其是在安全性对抗中我们通常关注的算法中，hashcat 仍然是翘楚。你可以从[`hashcat.net/hashcat/`](https://hashcat.net/hashcat/)站点获取二进制文件，或者使用他们的 GitHub[`github.com/hashcat/hashcat`](https://github.com/hashcat/hashcat)构建最新版本。利用预构建的二进制文件是开始的最简单选择，但如果正在开发中添加了新的算法，从源代码构建也是一个好处。

**hashcat-utils**（包括 maskprocessor/statsprocessor/princeprocessor）

这是一个非常棒的工具集合，可以根据不同的情况或密码模式来帮助你。例如，在 SEVN-X 这里，我们利用 cutb 工具创建了所谓的密码沙拉（我们将在博客中进一步介绍这种技术）。我们还利用组合工具生成短语和更长的密码候选项。Maskprocessor 在生成自定义字典时非常方便。你可能会发现使用其中一个可用的实用程序解决你遇到的问题，所以一定要检查一下。

[`hashcat.net/wiki/doku.php?id=hashcat_utils`](https://hashcat.net/wiki/doku.php?id=hashcat_utils)

**pack/pack2**

令人惊叹的密码分析破解工具包（PACK）非常适用于密码分析和构建自定义规则等。Pack2 是 PACK 的 Rust 版替代方案，但正如作者所说，它还在不断改进中。

#### 关于基准测试的几点说明

现在流行对 PC 构建进行基准测试，特别是对于那些寻求购买专门用于此目的的 GPU 的人来说，对 hashcat 进行超越游戏的基准测试非常有用。有几个资源可以做到这一点，我们在 SEVN-X 发现，通常在 GitHub/Twitter 上搜索就可以搞定。还有这个网站包含了一个相当不错的 GPU 基准测试列表：

[`www.onlinehashcrack.com/tools-benchmark-hashcat-gtx-1080-ti-1070-ti-rtx-2080-ti-rtx-3090-3080-4090.php`](https://www.onlinehashcrack.com/tools-benchmark-hashcat-gtx-1080-ti-1070-ti-rtx-2080-ti-rtx-3090-3080-4090.php)

此外，对于所有认为基准性能就是你购买了看到基准性能的 GPU 时会得到的性能的人，请让我在这里告诉你，不是这样的。hashcat 中的基准测试是针对一个哈希值进行模拟的，这意味着多个哈希值会增加你的破解时间。基准测试不会模拟词汇表或规则，简单来说，它只是以速度作为基准来模拟暴力破解可能的密码组合。因此，当你开始使用词汇表或词汇表与规则的组合破解该哈希值时，你的速度将不会达到基准的水平，即使在暴力破解模式下，如果是多个哈希值，速度也不会相同。还有超频、驱动程序更新、hashcat 算法优化以及可能使你的结果与别人的基准不同的潜在硬件差异，所以请注意这一点。

#### 字典和词汇表

当涉及到破解哈希值时，有几个很好的资源可以用于开始使用词汇表、字典或密码列表。其中最臭名昭著的是广泛可用的 rockyou.txt 明文密码集合。值得注意的是，生成自定义词汇表可能会有所帮助，或者建立自己的全面收集以增加破解下一个哈希值的机会。有像 cewl 这样的工具可以帮助从客户网站上抓取关键字。

如果你想要构建自己的词汇表，可以有很多内容可加入。还有能够收集公开可用的密码转储并从中构建列表的能力。首先，建议构建词汇表（而不是密码列表）的方式是从以下内容开始：

*这个列表是基于你试图破解英语用户的密码的假设。它作为一个良好的基线来源，并应检查重复项，以确保我们在破解过程中不浪费资源。创建自己的列表中最耗时的部分是收集数据集并整理好全部内容。

+   英语单词词典

+   名字

+   姓氏

+   国家名称

+   郡名

+   城市名称

+   街道/道路/巷道等名称和有效地址

+   维基百科标题/词语

+   Urban dictionary 词语/短语

+   Alexa 前 100 万名（品牌/组织/商业等名称）

+   媒体（视频）- IMDB/TVDB 电影标题/剧集标题/角色名称/演员名称/词语

+   媒体（音乐）- 歌曲/专辑/艺术家/歌词/词语

+   媒体（游戏）- 标题/角色/词语

+   媒体（书籍）- 标题/角色/词语

+   媒体（引用）- 任何名言

+   流行文化- 梗、引用、话题标签等

创建列表的另一个很好的来源是用户名。有几个收藏品，但用户名或电子邮件前缀（不包括@someting 部分）是相当不错的规则集候选项。

现在，如果您想建立一个公共密码泄漏集合，有几个来源可以用来开始构建您自己的集合。GitHub 上有大量不同的集合，或者如果您愿意冒险的话，也可以在 torrent 网站上找到。这些大部分只是一个谷歌搜索的距离！

#### 很棒的资源

当您刚开始涉足密码破解领域或考虑组建您的第一台专用破解机时，这些都是高质量的资源和地方，在这里，SEVN-X 在多年的建设中学到了很多知识，而且您可以在这篇博客之外找到更多信息。

链接：[`hashcat.net/wiki/`](https://hashcat.net/wiki/)

+   那个 hashcat 维基是该工具的宝贵资源。这里有大量的文档和链接到其他资源，可以帮助应对各种情况。

+   Defcon Password Village 提供了一些很棒的指南和从 hashcat 到硬件的初学者友好的笔记。

如果您对博客文章、密码破解机配置或 hashcat 有任何问题或反馈，请发送邮件至 hashburglar@sevnx.com。
