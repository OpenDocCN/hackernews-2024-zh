<!--yml

类别：未分类

日期：2024-05-27 14:37:54

-->

# 我的第一篇被删除的维基百科文章 - 利亚姆的只写博客 —— LiveJournal

> 来源：[https://lproven.livejournal.com/189791.html](https://lproven.livejournal.com/189791.html)

到目前为止，这是我唯一一篇完全从维基百科中删除的文章。我从中挽救了一份副本

[Search.com](https://www.livejournal.com/away?to=http%3A%2F%2Fwww.search.com%2Freference%2FNon-PC_compatible_x86_computers)

，因为它现在已经从网站的历史中消失了，我把它放在这里，直到我决定做其他什么。

**非PC兼容的x86计算机**

在Microsoft操作系统主导商业计算的大部分时间里，人们认为所有基于Intel x86系列CPU的计算机都是IBM PC兼容的。这并不是事实，也从来没有。  

**MS-DOS系统**

从PC市场的早期开始，就存在其他不兼容IBM PC的MS-DOS系统，比如天狼星维克多计算机和一些早期型号的杏花电脑。它们采用8088或8086处理器，并运行其自己的MS-DOS版本，但内存布局、显示控制器和其他硬件与IBM的组件不同，并位于内存的不同区域。这些公司预计通过生产技术上更先进的设计，这些设计可以运行流行的MS-DOS操作系统，他们将能够利用庞大的MS-DOS应用软件库。

除了其低硬件规格之外，IBM PC可能被认为是最大的“设计缺陷”是现今所称的上部内存区域 - 即I/O区域位于内存地图的顶部（从640 KiB到1 MiB）。将I/O区域定位在内存地图的底部而不是顶部使得扩展这些机器的内存成为可能，而且确实很简单。更多的RAM只是出现在内存地图的顶部，直到8086处理器的1 MiB可寻址限制。因此，像维克多这样的机器通常可以占用800-900 KiB的RAM，比PC的640 KiB限制要多得多。与IBM PC设计相比的其他改进包括高密度1.2 MiB软盘驱动器作为标准配置，集成声卡和单色图形（与IBM PC的Hercules图形卡相当）。

然而，尽管这在理论上足以保证商业成功，但实际上这并不是一个商业上的成功，因为在早期版本中，MS-DOS非常基础，并且只为应用程序开发人员提供了微薄的功能。因此，大多数软件作者没有使用DOS的API来控制硬件，而是直接向外围设备的控制寄存器等写入PC硬件。这意味着大多数PC应用程序无法在其他非PC兼容的x86机器上运行。

随着 Windows 及特别是 Windows NT 的出现，这变得不再重要，因为它为 Windows 应用程序提供了广泛的 API，并阻止开发人员访问硬件寄存器。然而，第一个成功的 Windows 版本是 1990 年的 Windows 3.0，对于专有的 DOS 机器来说太晚了，尽管一些最后一批非个人电脑兼容的 Apricot 系统支持它们自己的特殊版本的 Windows 2.0，因此可能运行了许多 Windows 2.0 应用程序。

**多用户和网络机器**

甚至还有更少类似个人电脑的 8086 系统，如 Jarogate Sprite，一个多用户的 Concurrent CP/M 机器，类似于小型计算机，仅支持哑终端 - 系统单元上没有显示控制器或键盘接口。另一个是 3Com 3Server 和后来的 3Server386，它们是专用的个人电脑局域网服务器，既没有自己的显示器和键盘，甚至没有终端 - 控制、设置和管理都是通过网络远程完成的。这些运行 3Com 的基于 DOS 的 NOS，3+Share，其中包含其自己的内部多任务系统。尽管 3Server 取得了一定的商业成功，但它的伴随产品不太成功，即无盘专用局域网工作站：3Station。

PC 兼容机及其庞大的软件库的崛起，包括一系列替代操作系统，淘汰了这些早期系统，但后来还是出现了一些。

**UNIX 工作站**

英特尔的 80386 处理器是第一个包含内存管理硬件的处理器，因此能够运行完整的 UNIX 操作系统。Sun Microsystems 利用这一点发布了 Sun386i，一款基于 386 的 UNIX 工作站。尽管基于主要用于个人电脑的处理器并配备 ISA 扩展槽，但这仍然是一台工作站，而不是个人电脑。存储设备采用 SCSI 设备，支持 Sun Type 4 键盘和光学鼠标，并通过 Sun 固件引导至 SunOS 4。这些机器的开发代号是 Roadrunner。

尽管 Sun386i 原生运行 UNIX，但 Sun 开发这台机器时考虑了进入个人电脑市场的可能性。Sun386i 通常包含一个 PC 模拟器（模拟 8086），可以运行 MS-DOS 和正常的 DOS 应用程序。由于它只是一个 UNIX 任务，可以使用多个模拟器实例，基本上实现了一个多任务 DOS 系统。性能相当不错，但 386i 并未取得很大的市场成功。老的 UNIX 客户更喜欢 Sun-3 系列，因为大多数商业 UNIX 应用程序在 Sun386i 上不可用。对 MS-DOS 感兴趣的新客户对 UNIX 没有真正的用处，更喜欢运行 MS-DOS 的真正个人电脑，速度更快、更便宜。当他们的 SPARC 系统上市时，Sun 很快就放弃了 386i 系列。基于 80486 处理器的后继工作站（代号 Apache）进入了原型阶段，本应被命名为 Sun486i；不到 200 台。

**Windows NT 工作站**

微软的 Windows NT 操作系统最初旨在成为跨平台产品，直到 NT 4 版本为止，它运行在由多家厂商生产的 MIPS、DEC Alpha 和后来的 PowerPC 处理器驱动的系统上。这是不成功的高级计算环境计划及其高级 RISC 计算规范（ARCS）的一个衍生产品。

据报道，开发版本也曾在 SPARC 上运行，但从未商业发布过。

然而，随着 x86 处理器性能的持续提升，随着 Windows 2000 的推出，对所有非英特尔 CPU 的支持被取消了，尽管有一段时间，据称微软内部维护了 Alpha 版本，以确保代码的可移植性。

在 1990 年代中期，另一代短暂的专用 NT 工作站以 SGI 视觉工作站的形式出现。同样，这些工作站运行工作站固件 - 在本例中是 ARCS - 并且具有完全不像 PC 的总线、显示器等布局，尽管它使用了 x86 处理器。这是 Windows NT 中 HAL 第一次帮助增加工作站成功的时候，因为大多数 x86 应用程序可以在其上运行，这要归功于 HAL。

**苹果麦金塔**

在 2005 年 6 月 6 日的苹果全球开发者大会上，苹果 CEO 史蒂夫·乔布斯宣布，未来的麦金塔计算机将基于英特尔处理器，而不是公司之前的基于 PowerPC 的产品线。首个发货的是 iMac，在 2006 年 1 月 10 日。这一转变预计将于 2006 年年底完成。

首批发货给开发者的开发系统包括一个相对标准的 Pentium 4 主板，放在一个 PowerMac G5 的机箱里 - 包括一个普通的 PC BIOS ROM。

这导致许多观察者得出结论，即未来基于英特尔的 Macintosh 系统将是通用 PC 兼容机，但事实并非如此 - 最终的产品将传统 BIOS 替换为更新的 EFI 标准。苹果的新计算机仅仅是一系列基于英特尔的计算机中的最新产品，而不是个人电脑。

请参阅关于苹果英特尔转换的文章以获取更多信息。

**总结**

许多因素决定一台机器是否是个人电脑兼容机。这些因素包括但不限于 IBM-PC 兼容 BIOS、一组兼容的 I/O 端口、包含视频 ROM 和 RAM 的上限内存区域等；一般来说，系统的内存映射的详细排列，包括其固件、中断、I/O 端口等因素，以及一个 x86 微处理器。

这些因素不包括其扩展总线。一些机器可能没有这样的总线，一些机器可能拥有完全不兼容的总线，比如IBM个人系统/2机器。另一方面，存在PC标准插槽并不意味着一台机器是PC兼容的。许多非PC架构的机器具有或曾使用ISA插槽，包括Commodore Amiga、Sinclair QL兼容的Q40和Q60机器以及Peters Plus Sprinter；PCI已被用于PowerPC苹果麦金塔机器和Sun工作站，AGP则用于较新的麦金塔机器。

虽然其中许多机器相对较为不知名或不成功，但拥有x86处理器和类似PC的扩展总线，甚至能够运行微软操作系统，并不意味着一台计算机就是一台PC。

**外部链接**

*

[Sun 386i/250图片和描述](https://www.livejournal.com/away?to=http%3A%2F%2Fsites.inka.de%2Fpcde%2Fcollection%2Fsun386i_250.html)

*

[Sun 386i/150技术信息](https://www.livejournal.com/away?to=http%3A%2F%2Fwww.machine-room.org%2Fcomputers%2F7297%2F)

*

[Sun 386i/260技术信息](https://www.livejournal.com/away?to=http%3A%2F%2Fwww.machine-room.org%2Fcomputers%2F7298%2F)

*

[关于Jarogate Sprite的一些（不准确的）信息](https://www.livejournal.com/away?to=http%3A%2F%2Fwww.computer-archiv.de%2Fcomp0775.htm)

- 用德语表达