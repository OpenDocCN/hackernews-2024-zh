<!--yml

category: 未分类

date: 2024-05-27 14:55:23

-->

# 忘记 Unix 的历史会让我们编码陷入困境 • The Register

> 来源：[https://www.theregister.com/2024/02/16/what_is_unix/](https://www.theregister.com/2024/02/16/what_is_unix/)

FOSDEM 2024 Unix 历史中有许多重要的教训，但它们正在被遗忘。这导致了大量的努力被浪费。

本文是基于 *The Reg* FOSS 专栏在 FOSDEM 2024 上的 [演讲](https://fosdem.org/2024/schedule/event/fosdem-2024-3095-one-way-forward-finding-a-path-to-what-comes-after-unix/) 系列文章的第二篇。

FOSDEM 是一场关于自由开源软件的会议，而当今，或许令人遗憾的是，FOSS 与 Unix 有关。越来越多的人认为，[Unix 今天就是 Linux](https://www.theregister.com/2023/01/17/unix_is_dead/)。问题在于，每个人都在忘记 UNIX 的真正含义，了解你的历史至关重要。正如乔治·圣塔亚纳经常被错误引用的那样说过：

这段话经常被错误引用，这也许是为什么亨利·斯宾塞（做了正则表达式的原始 [FOSS 实现](https://groups.google.com/g/mod.sources/c/OqVZYQNSmDs/m/G_T2jHu5QI8J)） [改述了它](https://groups.google.com/g/sci.space.shuttle/c/L8-Upf8gZoY/m/NN6ngTI0K0QJ)：

圣塔亚纳预言的事情正在今天世界各地成为现实，斯宾塞的预测也是如此。虽然还有其他选择，但几乎没人注意它们。为什么会这样？

### 'Unix' 究竟是什么？

首先，这并不意味着与原始 AT&T 代码库有关。自 1993 年以来，这个定义就不再正确了。[31 年前，Novell 从 AT&T 手中购买了 Unix](https://www.theregister.com/2013/07/25/novell_peaked_with_netware_four/)，保留了代码，并将 UNIX 商标捐赠给了 Open Group。从那时起，“Unix” 的名称就意味着“通过 Open Group 的测试”，这基本上是 POSIX 曾经的含义。原始代码库实际上已经消失。[SCO Group 的子公司 Xinuos](https://www.theregister.com/2021/03/31/ibm_redhat_xinuos/) 仍在销售基于它的产品，但他们的主要产品基于 FreeBSD。

但与 x86-64 和 Arm64 上的 macOS 以及一些其他操作系统（如 HP/UX 和 IBM AIX）并存的是 [Open Group 注册的 UNIX® 认证产品](https://www.opengroup.org/openbrand/register/)，曾经有两个不同的 Linux 发行版，[华为 EulerOS](https://www.opengroup.org/openbrand/register/brand3622.htm) 和 [浪潮 K/UX](https://www.opengroup.org/openbrand/register/brand3617.htm)，它们都是中国的 CentOS Linux 派生版本。不管你喜欢与否，这意味着 Linux 不再是“类 Unix 操作系统”。它就是一个 Unix。

它是一个 [Unix 大家族的成员](https://en.wikipedia.org/wiki/History_of_Unix#/media/File:Unix_history-simple.svg)。甚至 [剑桥大学的简化版本](https://www.cl.cam.ac.uk/teaching/0708/OSFounds/P04-4.pdf) [PDF] 也显示了近 30 个分支。维基百科的家族树显示了大量的分支和线路：从 Version 1 到 4，然后从 Version 5 到 6，然后 Version 7，System III 和 System V … 还有很多专有的派生版本和所有的 BSD 系统。当然还有 Linux，没有共享的代码，但有共同的 *设计*。

但是如果你再退一步，考虑到现在 Unix 意味着兼容性，*它们都是一个分支*。这不再是关于传承的线路，因为从某种意义上来说，曾经的 POSIX 定义 *成为了* Unix。但是如果你的目标仅仅是通过兼容性测试，你可以使用某种兼容性层或桥接实现。这就是为什么 IBM z/OS 的几个版本出现在 Open Group 列表中。这看起来很奇怪，因为它们根本不像 Unix。

这带来了其他问题，比如 Unix-like 操作系统究竟是什么？任何了解如何在 Linux 或 macOS 终端内工作的人都知道 Unix 是什么样子的。一些识别特征包括：

+   一个根目录为 `/` 的文件系统。

+   一个 shell。（哪个都行：`bash`、`tcsh`、`zsh`。）

+   像 `ls`、`cat`、`echo`、`mkdir`、`rmdir`、`touch` 这样的命令。

+   区分大小写的文件系统。

+   纯文本文件，以及与重定向和管道相关的命令。

当然，这些都可能有例外。例如，macOS 是一个 Unix™。它通过了测试，并且 Apple 为认证付费。但它隐藏了大部分真正的 Unix 目录树，它的 `/etc` 相对为空，它没有 X 服务器 —— 它是一个 [可选附加组件](https://www.xquartz.org/)。最重要的是，它不区分大小写。

但是，它仍然可以明显地被识别为 Unix。

因此，根据这些一般特征的列表，再加上一个不太显眼的特征 —— 主要用 C 或类似 C 的语言编程 —— 并要求操作系统看起来像 Unix *而不是其他任何东西*，这样的家族就更大了。

所以，如果只是关于兼容性，而我们不再关心代码从哪里继承来的，这就提出了其他问题。我们如何对它们进行分类？Unix-like 操作系统究竟是什么？

### 代和之间的间隔

在 AT&T 原始 Unix 系列中连续项目的观察非常有教益。在它的第一个十年里，Unix 是一个研究项目，而不是一个商业产品，在那段时间内它经历了大约七代。

事实上，即使其他项目开始在 Unix V4 和 V8 之间分支并走上自己的道路，原始的研究项目仍在继续。维基百科和剑桥家族树显示了 AT&T 线在第十版之后停止。但实际上，还有两个更多的代。

如果我们把整个大家庭看作一个单元，在图表上只占一个格子，那就会留下一些离群值。Linux 是其中之一。Minix 是另一个。还有其他未显示的系统，其中有一整类系统非常类似 Unix：微内核系统。

最初的微内核 [CMU Mach](https://www.cs.cmu.edu/afs/cs/project/mach/public/www/mach.html) 导致了一大批 Unix 操作系统，包括 [Open Group 的 OSF/1](https://kb.iu.edu/d/agoq) 和 [DEC Tru64](https://kb.iu.edu/d/agoo)，以及 [MkLinux](https://www.mklinux.org/) 和著名的 [GNU HURD](https://www.gnu.org/software/hurd/)。唯一不是历史上的奇迹或被忽视的小众是 Apple 的 macOS 家族，包括 iOS、iPadOS 等。

安迪·坦尼伯姆博士的 Minix 是 Linux 的间接前身之一，尽管 Minix 1 和 2 主要是单片内核。然而 [Minix 3](https://www.minix3.org/) 是真正的微内核，现代 Intel CPU 的管理处理器内都运行着其版本。

还有其他系统。对 Mach 的研究导致了 [L4](http://os.inf.tu-dresden.de/L4/) 和 [其他系统](https://www.l4ka.org/)，并且工作 [仍在继续](https://sel4.systems/)。[QNX](https://blackberry.qnx.com/en) 是一种商业微内核类 Unix 操作系统，广泛用于数十亿的嵌入式设备……尽管你可能唯一接触过它的时候是 [Blackberry 10](https://www.theregister.com/2015/10/02/bb10_five_reasons_why_it_failed/)。

尽管它们比你想象中的更成功，但微内核要运行良好并不容易。小内核加上大量在用户空间执行的进程并不难——AmigaOS 在 1980 年代中期成功做到了这一点。问题在于保持各个进程的隔离性，同时又能够彼此通信。AmigaOS 运行在没有内存管理硬件的机器上——所有东西都在一个地址空间中。当你可以读写另一个进程的内存时，与另一个进程通信就变得容易了。

真正微内核的概念是内核在 ring 0，即超级用户模式下运行。它负责进程调度、内存分配，其他功能几乎全都在用户空间的“服务器”中实现。让它们*快速*互通是个难题。

例如，macOS 使用 Mach，但为了提供 Unix 兼容的 API，它采用了大量的 BSD 内核，并将其称为“Unix 服务器”。Mach 仍然管理进程，而不是这个 Unix 服务器，但即便如此，该服务器仍然庞大且复杂。并且速度仍然不快，因此 NeXT 将其移入了 Mach 内核中，这意味着它也在内核空间中运行，与现在不那么微型的 Mach 内核在同一内存空间中运行。

这留给我们两个类Unix操作系统家族。单体内核，其中包括几乎所有旧的专有商业Unix系统，以及所有的BSD系统 - 还有Linux。另外还有微内核，我们可以很宽泛地分为两类。一类是商业的，今天主要指的是苹果的操作系统和QNX。QNX是专有的，而苹果的内核和用户空间是开源的，但影响不大其他系统。

还有一个开源家族，包括Minix 3、GNU Hurd、L4、seL4等等。一些被用于嵌入式控制系统，比如[英特尔的ME](https://www.intel.com/content/www/us/en/support/articles/000008927/software/chipset-software.html)，但在主流上没有很明显的使用。

但这并非终点。尽管大多数商业Unix起源于1979年的Unix第7版，但丹尼斯·里奇和肯·汤普逊的工作并未在那一年结束。

### 把它提升到11

第七版Unix之后的版本有时被统称为[研究Unix](https://en.wikipedia.org/wiki/Research_Unix)，但我在网上询问过时，人们更多地告诉我这是*错误*，而非有用的信息。

尽管如此，在AT&T Unix System Laboratories内的较少知名版本包括1985年的第八版，1986年的第九版，以及1989年的第十版。有些零碎的[源代码可供查阅](https://www.theregister.com/2017/03/30/old_unix_source_code_opened_for_study/)，但信息很少。（欢迎提供这些内容的工作链接或参考！）

对于这位作者来说，第十版之后发生的事情才真正有趣。到1992年，Unix成为商业新闻头条，当然了。FreeBSD和NetBSD正在发生，Linux也开始引起关注。

与此同时，贝尔实验室做了其他的事情。

想想传统的老式Unix是什么构成。它关乎的是“一切都是文件”，字节流，熟悉的shell等等。但是在所有现代计算机上有重要的东西并没有被这些覆盖。

特别是图形并不真正构成经典Unix内核的一部分。macOS和Linux都是Unix系统，BSD也是，但它们拥有截然不同的图形用户界面层。这是因为Unix是一种[小型计算机](https://www.britannica.com/technology/minicomputer)操作系统。正如[Dennis Ritchie本人所描述的](https://www.bell-labs.com/usr/dmr/www/hist.html)，它是在一台[DEC PDP-7](https://www.soemtron.org/pdp7.html)上编写的，并且后来移植到更大的[DEC PDP-11](https://gunkies.org/wiki/PDP-11/20)上。

这些是部门规模的、多用户计算机。一个主机，加上串行连接的哑文本终端，没有图形也没有网络 - 即便如此，这在20世纪70年代是高端设备。

在商业 Unix 蓬勃发展的时期，即 1980 年代，有非常昂贵的*工作站*：实际上是个人单用户小型计算机。直到 1980 年代末，才有了像廉价的带有内置图形和相对快速扩展总线的 32 位计算机（因此，相对便宜的快速网络也开始流行）才成为主流。然后 Unix 获得了网络支持，至今仍然如此。但是，让一个 Unix 机器与另一个 Unix 机器通信的方式并未考虑在原始设计中。相反，它有终端类型定义文件。即使今天，几十年后，网络和图形用户界面仍需要辅助的工具，比如 NFS、SSH、X11 等等。这些工具运行良好，但这些都是后来添加的额外组件，出现在设计已经定型多年后。

这是 Unix 第十版出现的时候，即 1989 年，网络和图形用户界面（GUI）随处可见的时期。

今天，Wayland 的爱好者喜欢谈论他们如何现代化 Linux 图形堆栈。但是 Linux 是一个 Unix 系统，在 Unix 系统中，[一切都被视为文件](https://hackmd.io/@jkyang/unix-everything-is-a-file)。那么，所有的 Wayland 传道者，请告诉我们：在文件系统中，我在哪里可以找到描述屏幕上窗口的文件？哪个文件保存窗口的坐标、Z 序、颜色深度和内容？

如果这些东西不在文件系统中，你确定这是一个 Unix 工具吗？

研究 Unix 第 10 版开始解决了一些问题，但为时已晚。这个行业采用了更古老、更简单的版本，进行了商业化，并且结果正在全球范围内突变和扩散。

因此，Dennis Ritchie 和 Ken Thompson 做了[尼古劳斯·维尔斯做过的事情](https://www.theregister.com/2024/01/04/niklaus_wirth_obituary/)，使用 Modula-2 和后来的 Oberon。他们忽略了行业正在做的事情，回到了他们的原始想法，并继续对其进行精炼。

这就是 Unix 发展的下一个步骤，在软件复杂性和膨胀的时代，是时候重新审视它，看看 Unix 的发明者们是否有正确的想法，就像 Linux 正在被创造的时候一样。®

*这是[多部分系列的第二部分](https://www.theregister.com/Tag/One%20Way%20Forward)，源于我们的秃鹫在 FOSDEM 2024 的演讲。接下来的部分即将到来……*
