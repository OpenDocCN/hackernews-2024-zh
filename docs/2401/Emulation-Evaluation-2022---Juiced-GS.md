<!--yml

类别：未分类

日期：2024-05-27 14:25:47

-->

# 仿真评估 2022 - Juiced.GS

> 来源：[https://juiced.gs/emulators/](https://juiced.gs/emulators/)

（*本文收集了 Ivan Drucker 在* Juiced.GS *的 2020 年 [12 月](https://juiced.gs/store/volume-25-2020/)、[2021 年](https://juiced.gs/store/volume-26-2021/) 和 [2022 年](https://juiced.gs/store/volume-27-2022/) 发布的模拟器评论。有关此处未审查的更新模拟器的完整评论，以及对下面所述模拟器最新版本的更新报告，请参阅 [2023 年 12 月号](https://juiced.gs/store/volume-28-2023/)！*）

并非我们所有人都随时随地都有自己的 Apple II 计算机。次好的选择（有时甚至更好）就是模拟器！模拟器是一种虚拟的 Apple II，通过软件重新创建。它的硬件组件，从 6502 开始，都被模仿，作为针对不同操作系统编写的应用程序的一部分。磁盘被表示为包含其数据的“映像文件”，在模拟器中运行时，Apple II 软件无法意识到它并非在真实设备上运行！模拟器使我们能够在更现代的设备上使用我们喜爱的计算机软件。

Apple II 社区拥有各种优秀的模拟器选择，适用于 Mac、Windows、Linux 和移动操作系统。选择合适的模拟器可能会很困难 —— 因此让我概述每个的优缺点。

尽管这个总结不是 Apple II 模拟器的详尽列表，但我已经包括了模拟增强型 Apple IIe 或 IIGS、在当前主要操作系统版本上运行的标题，以及自 2010 年以来已经更新的标题。开源项目必须具有特定的发布点（除非它们是基于网络的），而不仅仅是代码存储库；我只审查最近声明的发布，而不是随后提交的任何代码。Windows 和 macOS 的发布必须具有可立即运行的二进制可执行文件。除一款外，所有标题均免费。

所有 macOS 标题都是原生支持较新的苹果芯片和较旧的基于英特尔的机器，除非另有说明；仅限英特尔的标题将在苹果芯片型号上通过转换运行。Mac 用户如果收到“应用程序因无法验证开发人员而无法打开”的警告，则需要在首次运行时，对应用程序进行控制点击，选择菜单中的“打开”，然后从随后的警告对话框中选择“打开”。

对于 Windows 标题，使用高 DPI 显示器的用户可能需要通过在 Windows 资源管理器中查看其属性来调整模拟器 EXE 文件的兼容性设置。

这篇文章的修订版包括 2020 年 12 月原始文章、2021 年 12 月和 2022 年 12 月的更新文章。同时还进行了编辑更改。

## **最佳的 Apple II 模拟器**

这些模拟器功能丰富，提供准确的仿真，并相对易于使用。

### **Virtual II**

**平台：** macOS

**模拟的型号：** II、II plus、IIe（U）、IIe（E）、IIc

**图像支持：** DSK、PO、D13（有限）、NIB、HDV、2MG、WOZ、V2D、GZ

**状态：** 持续开发中（v11.1 于 2022 年 9 月发布）

**作者：** Gerard Putter

**提供自：** [https://virtualii.com/](https://virtualii.com/)

Virtual II 几乎能够满足你对模拟器的所有需求。它具有经过精心打磨的、易于导航的界面，并且忠实地再现了除 IIc Plus 之外的所有 8 位 Apple II 型号。然而，Virtual II 真正突出的地方在于它的虚拟硬件集合以及您可以部署它的灵活性。就像真正的 Apple II 一样，你可以选择要放入哪个插槽的外围设备卡，以及要将这些卡连接到哪些设备上。包括软驱和硬盘接口卡、内存卡、并行打印机卡、串行卡、CP/M 卡、时钟卡、Mockingboard 卡和鼠标卡。其中一些与 macOS 整合：超级串行卡可以通过实际的 macOS 界面发送和接收，其中一个打印机可以输出到 macOS 文本文件。Apple IIc 的仿真是一个较新的功能，缺乏其他型号的虚拟硬件功能，除了软驱和硬盘支持外，几乎没有提供任何功能。在版本 11.1 中，IIc ROM 0 被错误地标记为 Apple IIc Plus。

内置的 Apple II 硬件也有所体现。CPU 速度（对于 6502/65C02 和可选的 Z-80）可以加速。还模拟了磁带端口和游戏连接器，并且对于模拟 II/II+，可以选择板载内存数量和 shift 键修改。程序员还将喜欢集成的 6502 调试器。可以在单独的窗口中模拟多台机器，并且可以将 Apple II 屏幕录制到电影文件中。随时可以保存模拟机器状态。还提供了从 ShrinkIt 存档中提取文件的功能。

磁盘支持值得一提。对于 5.25 英寸软盘，一个或多个模拟的 Disk II 卡可以承载两个 Disk II 驱动器，并模拟驱动器本身的物理声音（通过音频样本）。对于 3.5 英寸软盘和硬盘，提供了虚拟硬件（没有真实的世界对应物）：一个或多个“SCSI II”卡可以承载一个或两个“OmniDisk”驱动器，每个驱动器都可以作为可移动的 ProDOS 硬盘运行，并支持最多 32mb 的任何磁盘镜像。

更重要的是，任何 Mac 文件夹都可以被“插入”到一个驱动器中，并被视为 ProDOS 或 DOS 3.3 磁盘（根据您使用的是 Disk II 还是 OmniDisk，大小为 140K 或 32 MB），极大地简化了在 macOS 和模拟 Apple II 之间传输单个文件。Virtual II 还提供了自己的 Spotlight 接口，用于跨磁盘镜像搜索 Apple II 文件名，并且可以通过 macOS QuickLook 快速查看磁盘镜像的目录，巧妙地将文件名叠加在软盘标签的图形上。磁盘写入不会保存到磁盘镜像中，直到虚拟磁盘被弹出、模拟器窗口关闭，或从菜单中选择“刷新”命令。

我可以继续说下去，但我会在此结尾，说 Virtual II 除了其多功能性之外，还将无数细节处理得非常到位，具有出色的整体质量和表现，并提供了优秀的文档；其 39 美元的价格完全可以理解。（还提供了一个价格为 19 美元的“限制”版本，但省略了一些有用的功能，如保存状态、屏幕录制和 macOS 文件夹挂载；另外还提供了一个免费的“评估”版本，额外添加了水印和强制暂停。）

### **AppleWin**

**平台：** Windows（还有 macOS 和 Linux 变体）

**模拟的机型：** Apple II、II Plus/J-Plus、IIe（U）、IIe（E）；一些 Apple II 克隆机

**图像支持：** DSK、PO、HDV、2MG、NIB、WOZ、ZIP、GZ

**状态：** 活跃开发中（v1.30.12.0，于2022年9月27日发布）

**作者：** 汤姆·查尔斯沃斯、尼克·韦斯特盖特、迈克尔·波霍雷斯基 等

**可从以下位置获取：** [https://github.com/AppleWin/AppleWin/releases](https://github.com/AppleWin/AppleWin/releases)

**macOS 变体：** [https://github.com/sh95014/AppleWin/](https://github.com/sh95014/AppleWin/)

**Linux 变体：** [https://github.com/audetto/AppleWin/](https://github.com/audetto/AppleWin/)

在微软 Windows 平台上，卓越的 8 位 Apple II 模拟器是 AppleWin，其起源可以追溯到 1994 年。AppleWin 界面简洁直观，几乎与其 Windows 3.1 时代的版本保持不变。它提供了准确的仿真，包括多种模拟显示器样式，以及一系列强大的虚拟硬件设备，预先分配到 Apple II 的典型插槽中。其中包括四个 5.25 英寸软盘驱动器和两个硬盘驱动器；Mockingboard（与大多数模拟器不同的是，具有语音功能）、Phasor 和 SAM 声卡；一个 CP/M 卡；一个 Uthernet I 或 II 网卡；一个 Super Serial 卡（可以通过真实的 Windows 串口或通过 TCP 进行通信）；一个鼠标；SNES MAX 和 4Play 游戏控制器卡，以及一个并行打印机卡。没有实际的打印机仿真；输出被发送到 Windows 文本文件中。不幸的是，窗口大小不能随意缩放。

通过从命令提示符或批处理文件中启动 AppleWin，或者通过将适当的命令行开关附加到 Windows 快捷方式的目标，可以获得更多硬件配置选项。虽然这些选项并非每天都有兴趣，但遗憾的是，这种方法是必需的，以实现潜在有益的配置，例如超过 128K 的内存，或者将默认外围卡从其插槽中移除。如果一些命令行启动选项被添加到配置窗口中，那么这将有利于 AppleWin。尽管如此，能够存在这些选项仍然是不错的。

AppleWin 与必要的 Apple II 磁盘映像工具 CiderPress 集成，可以方便地在 Windows 和模拟的 Apple II 之间交换文件。模拟机器状态可以被保存，并且 CPU 可以被加速。对于 Apple II 程序员来说，AppleWin 包含一个强大的调试器。强烈建议阅读附带的帮助文件，以了解所有模拟器的功能以及如何访问它们。

对于想要模拟 Apple II 的 Windows 用户来说，AppleWin 做得非常好。此外，macOS 和 Linux 用户可以通过 2022 年新出现的独立维护的变体使用 AppleWin；我没有对这些进行详尽的尝试，但是 macOS 版本，称为 Mariani，似乎做到了应有的功能。Linux 版本需要从源代码构建。

### **KEGS（包括 GSport 和 GSplus）**

**平台：** macOS、Linux（Windows 通过 GSport 或 GSplus）

**状态：** 活跃开发中（v1.16 于 2022 年 1 月 23 日发布）

**模拟的型号：** IIGS（ROM 01 和 3）

**映像支持：** DSK、PO、NIB、HDV、2MG、WOZ、SDK、ZIP、GZ

**作者：** Kent Dickey

**来源：** [https://kegs.sourceforge.net/](https://kegs.sourceforge.net/)

**GSport：** [https://david-schmidt.github.io/gsport](https://david-schmidt.github.io/gsport)

**GSplus：** [https://apple2.gs/plus](https://apple2.gs/plus)

**Android 变体：** [https://play.google.com/store/apps/details?id=com.froop.app.kegs](https://play.google.com/store/apps/details?id=com.froop.app.kegs)

KEGS（Kent’s Emulated GS）是一个功能非常丰富的 Apple IIGS 模拟器，起源于 1990 年代，具有略显笨拙（尽管不难）的文本菜单驱动界面。在经过 16 年的停滞后，KEGS 在 2020 年底开始接收出色的增强功能，超越了其分支 GSport 和 GSplus（这两个分支在本文的原始版本中分别进行了审查）。尽管存在一些不足之处，并且可能会在启动时存在一次性挑战，但 KEGS 是一个一流的模拟器，现在是 Mac 和 Linux 用户的最佳 Apple IIGS 模拟选项。（2022 年，Windows 支持正在筹备中；GSport 和 GSplus 仍然是 Windows 用户的有价值的替代品。）

KEGS 提供了大多数真实 Apple IIGS 的功能。您可以使用在 IIGS 控制面板中可见的内置“插槽卡”的虚拟版本。65816 CPU 可以加速，并提供调试器。内存可以扩展到 14 MB。在仿真窗口下方的技术状态区域很丑，但可以隐藏。一个奇怪的地方是，屏幕边框有时在两侧不对称。

KEGS 中的磁盘使用是灵活的。有两个 3.5 英寸软盘驱动器，两个 5.25 英寸软盘驱动器，以及可以用于最大 32 MB 的 ProDOS 磁盘映像的十一个 SmartPort 驱动器。当 ProDOS 或 GS/OS 正在扫描每个驱动器以查找特定卷时，5.25 英寸驱动器会虚拟地“嘎吱嘎吱”地转动，这确实会减慢速度。您可以将主机操作系统中的文件夹挂载为 ProDOS 磁盘，从而可以轻松在两个系统之间传输文件；版本 1.16 中缺少此“DynaPro”功能的文档，但[在此可用](https://groups.google.com/g/comp.emulators.apple2/c/fddXQE73tGg/m/Frtt5skSAgAJ?utm_source=a2.click&utm_medium=urlshortener)。

KEGS 的最新功能包括对 Apple silicon 和 Intel Mac 的本机支持，可调整大小的窗口，虚拟 Mockingboard，WOZ 磁盘映像支持，操纵杆支持，改进的调试器，从主机操作系统复制粘贴到模拟的 GS/OS，以及以 2.8 MHz 运行时的精确定时准确性。

Mac 用户需要阅读下载文件中提供的“doc”文件夹中的 README.mac.txt 文件。Linux 用户需要从源代码构建。所有 KEGS 用户都应该仔细阅读详细的 README 文件，也许还希望阅读更具吸引力的 GSport 和 GSplus 的文档。

GSport 由 David Schmidt 等人开发，GSplus 由 Dagen Brock 开发，是基于 2004 年发布的 KEGS 的衍生产品。它们缺少许多以上最近添加的功能，这些变体不再开发。与 KEGS 相比，它们提供了可用的 Windows 版本，以及模拟 Uthernet I 和各种打印机（如 ImageWriter LQ）。GSport 还模拟 AppleTalk。对于希望使用这些功能的 Mac 用户，GSplus 是更好的选择，因为它是一个 64 位应用程序，可以在现代 macOS 版本上运行（通过 Intel 翻译在新的基于 Apple silicon 的 Mac 上运行，尽管开发者可以轻松重新编译为本机应用）。

在 2022 年，我发现了一个独立的 KEGS 移植到 Android 手机的版本，最后更新于 2013 年，它基于 GSport 和 GSplus 的相同 2004 年发布版本。我没有大量使用它，但足以看出它确实可以将 Apple II 放入您的口袋中，并巧妙地使用整个触摸屏作为鼠标。我建议您阅读其产品描述中的“可用性提示”，以及从那里链接的源代码页面上的附加信息。

虽然 KEGS 是一个很棒的模拟器，但我希望它的用户界面更加完善（特别是在选择磁盘时），有更多准备好的 Mac 打包（一个应用程序图标会很好，还有一个启动器文件可以避免需要从终端运行），一种方法来禁用或加速插槽 6 中的驱动器，并将 GSport 和 GSplus 提供的网络和打印增强功能合并进去。

## **优秀的 Apple II 模拟器**

这些模拟器可能稍显粗糙或者稍难使用，但它们仍然非常实用。

### **OpenEmulator**

**平台：**macOS

**模拟的型号：**Apple II、II Plus/Europlus/J-Plus、IIe (U)、IIe (E)；Apple I（及其克隆）

**图片支持：**DSK、PO、NIB、HDV、2MG、WOZ、DC42、V2D、FDI、VDI、VMDK

**状态：**积极开发中（v1.1.1 于 2022 年 3 月 11 日发布）

**作者：**Marc S. Ressl、Tobias Eriksson、Zellyn Hunter、4AM 等

**获取地址：**[https://openemulator.github.io/](https://openemulator.github.io/)

OpenEmulator 是一个易于使用的模拟器，旨在高度保真地呈现真实 Apple II 的外观，包括能够选择不同的历史显示器类型。模拟的机器状态可以随时保存。它通过虚拟插槽提供灵活性，可以连接虚拟硬件接口卡和配件。所表示的外设包括软盘（带有从真实驱动器采样的机械声音）和硬盘（支持多种图像格式），内存扩展，以及一些不寻常的选项，比如模拟的 Apple Graphics Tablet（与 Dazzle Draw 配合使用效果很好）、SilentType 热敏打印机和精密软盘调整。然而，也存在一些显著的遗漏，如时钟卡、鼠标卡、内存卡、声卡和 CPU 加速。

近年来，OpenEmulator 并没有获得太多新的功能。话虽如此，开源社区成员仍然使 OpenEmulator 保持了相关性，添加了对 WOZ 磁盘映像和 Apple IIe 模拟的支持。也有一些瑕疵，比如一个选项来模拟 Apple III 只会产生一个错误，以及一个神秘的虚拟 CPU 插槽，在模拟器明显运行时显示“未连接”。2022 年，OpenEmulator 更新了bug修复，并且成为了 Apple Silicon 原生。

对于想要一个免费、用户友好的 8 位 Apple II 模拟器并具有相当大的灵活性的 Mac 用户来说，OpenEmulator 是一个非常实用的选择。但如果它不能做你想要的事情，不要指望等待新版本中出现该功能。

### **Apple in PC (AIPC)**

**平台：**Windows

**模拟的型号：**Apple II Plus、IIe (E)

**图片支持：**DSK、PO、NIB、HDV、2MG、WOZ

**状态：**不明确（v0.1.46.1 于 2020 年 6 月 5 日发布）

**作者：**Keonwoo Kim

**获取地址：**[https://github.com/sosaria7/appleinpc/releases](https://github.com/sosaria7/appleinpc/releases)

AIPC起源于2000年代初，并在2016年以新的更新重新出现之前的很多年里很难找到。对于Windows用户来说，这是一个坚固的模拟器，以简单、实用的界面能够胜任基本的任务。

模拟的硬件包括鼠标、操纵杆、Mockingboard、Phasor、硬盘和5.25英寸软盘驱动器。卡片可以分配到任意槽位，一个不寻常的功能是可以定制Apple II屏幕颜色。Apple II机器状态可以在会话之间保存。模拟窗口不可缩放，只能以1x、2x和全屏显示。有一个6502调试器可用，但功能非常有限。虽然AIPC没有做什么事情，AppleWin也能做到，但我仍然喜欢它，因为它可能会有用，例如，如果配置一个具有多个软盘驱动器的虚拟Apple II，而AppleWin的固定槽位配置不提供此功能。

### **Ample**

**平台：** macOS

**模拟机型：** 所有8位Apple II型号（包括Apple IIc和IIc Plus）；Apple IIGS（ROM 00、01、3）；Apple-1；Apple III；Franklin、Laser和许多其他克隆机和国外市场变体

**图像支持：** DSK、PO、NIB、HDV、2MG、WOZ、DC42、EDD、CHD

**状态：** 持续开发中（当前版本：r38/250，发布于2022年11月30日）

**作者：** Kelvin Sherlock

**下载地址：** [https://github.com/ksherlock/ample/releases](https://github.com/ksherlock/ample/releases)

Ample实际上不是一个模拟器，而是Mac用户的宝贵实用工具，它使得功能强大的多机多平台模拟器MAME（[以下评论](https://juiced.gs/emulators/#mame)）在Apple II模拟方面使用起来更加简便。

Kelvin Sherlock的Ample解决了MAME启动时的复杂性问题。与其学习命令行语法或在笨拙的MAME图形界面中摸索，Ample为您完成了繁重的工作，提供了简单的控制，帮助您尽量减少麻烦，解锁MAME的功能。这并不意味着Ample将MAME转变成为像典型模拟器一样用户友好的东西，但它已经走了一部分路。

除了提供预构建的、专门针对Apple II的MAME版本，以及为您获取所需的ROM文件外，Ample还提供了一个简单的窗口，让您指定您希望模拟的确切机器，以及您想要放入其槽位的外围卡（超过50个可用的卡片，按全名列出）。您还可以轻松选择其他模糊的MAME选项，例如捕获Mac鼠标，这消除了我在MAME评论中提到的GS/OS中的“双指针”问题。

Ample将会按照您配置的方式启动MAME，尽管一旦运行，如果你想做一些事情，比如切换磁盘，你仍然需要按下Fn-delete，然后按tab键，以在MAME的棘手的本地界面中导航。Ample还提供了MAME及其Apple II专用模块的文档快捷方式。

虽然使用专门的 Apple II 仿真器标题仍然不像使用专用 Apple II 仿真器标题那样流畅，但对于任何对使用 MAME 进行 Apple II 仿真感兴趣的 Mac 用户来说，Ample 都值得一试，特别是如果之前因此而受到阻止。

### **Agat**

**平台：** Windows

**仿真的型号：** Apple II，II Plus/J-Plus，IIe（U），IIe（E）； Apple-1

**图像支持：** DSK，PO，NIB，HDV

**状态：** 不明确（当前版本：1.29.2，2019年2月16日发布）

**作者：** Sergey Gromov 和 Oleg Odintsov

**可用于：** [https://sourceforge.net/projects/agatemulator/files/agatemulator](https://sourceforge.net/projects/agatemulator/files/agatemulator)

Agat 不像其他 Apple II 仿真器那样广为人知，但它值得关注。以苏联制造的 Apple II 灵感为名（它也模拟了它），Agat 易于使用，具有一些原始功能，并且表现良好，提供了各种可以灵活配置的虚拟硬件。

外围设备包括 5.25 英寸软盘和硬盘卡（您可能希望禁用其中的机械硬盘声音样本），时钟，Mockingboard，“Slinky” 内存扩展，鼠标，CP/M，磁带，加速 CPU 和打印机（具有文本文件输出选项）。 Agat 的一些更引人注目的选项包括 Liberty 软盘驱动卡，支持 200K（SSSD）至 800K（DSDD）的 3.5 英寸磁盘大小，Apple 固件卡在插槽 0 中（而不是 16K 内存卡），以及一个简单模拟 Apple II 监视器的集成调试器。声音仿真效果不佳。

尽管这里列出的大多数仿真器都提供了保存不同机器配置的能力，但 Agat 是唯一一个在库中以丰富的颜色显示它们的，它预设了 45 种可供选择的配置（并非所有都是 Apple II 计算机）。 这是一个快速尝试其各种功能的好方法，您可以同时运行几台仿真机。 运行中的机器状态无法保存。

Agat 似乎不太可能获得重大改进，但对于 Windows 用户来说，它比 AppleWin 提供了更多的灵活性（虽然不那么忠实），使用起来也相当简单。一定要点击包含的帮助按钮以查看键盘快捷键。

### **microM8**

**平台：** macOS（Intel，在 Apple 硅上以翻译运行），Windows，Linux

**仿真的型号：** Apple II，II Plus，IIe（U），IIe（E）

**图像支持：** DSK，PO，D13，NIB，HDV，2MG，WOZ

**作者：** Paleotronic

**状态：** 活跃开发中（构建 201912161141，2019年12月16日发布）

**可用于：** [https://paleotronic.com/software/microm8/](https://paleotronic.com/software/microm8/)

microM8（早期版本称为 Octalyzer）是一个非常好的，但不寻常的 Apple II 模拟器。它与其他模拟器如此不同，以至于很难理解。虽然典型的模拟器通过软件模拟物理硬件，但 microM8 进入了一个真实的 Apple II 看不到的领域，如透视 3D 图形和可倒带的实时游戏。这个模拟器似乎不太关心忠实地复制 Apple II 的历史，而更关心通过虚拟化使 Woz 的宝贝执行酷炫的新技巧。

虽然 microM8 大部分可以像使用其他 Apple II 模拟器那样使用，但它的核心并不在此。当你启动它时，如果驱动器是空的，你不会看到顶部是一个空白的“Apple ][”；相反，你会看到一个色彩丰富的动画闪屏，邀请你做各种事情，例如加载其自己的增强版 BASIC 或连接到互联网 BBS。也许 microM8 最吸引人的特性之一就是能够从大型在线仓库加载驱动器。USB 游戏控制器可以被识别。虚拟硬件，固定插槽，包括 5.25 英寸软盘驱动器、硬盘、鼠标、调制解调器、彩色打印机、Mockingboard 和 CP/M 卡。

尽管 microM8 看起来友好，并且有在线文档，但起初使用起来可能有些挑战。它有许多键盘命令，在帮助屏幕上有描述，而且在从命令提示符启动时还提供了许多选项。将磁盘映像文件拖放到模拟器窗口的左侧或右侧即可插入。幸运的是，还有一个菜单系统，当你将鼠标悬停在模拟器窗口的左上角时会出现。 （还可以下载一个可选的 GUI 控制面板，但我并没有发现它特别有用。）功能集非常丰富，microM8 的一些功能非常复杂，例如强大的基于 web 的 6502 调试器，HTTP 控制 API 和软件打包机制。还有许多视频模式、屏幕录制等选择。我相信我还没有发现所有 microM8 可以做的事情。

microM8 是一个高质量的产品，极具创意。如果将 Apple II 模拟器比作电影，它可能成为一个受崇拜的经典。即使你已经拥有自己选择的模拟器，使用它也是一种乐趣。

## **适用于技术用户**

这些模拟器不是即插即用的，所以准备好动手吧。

### **MAME（Apple II 模拟器）**

**平台：** Windows（以及 macOS 和 Linux 变种）

**模拟的机型：** 所有 8 位 Apple II 机型（包括 Apple IIc 和 IIc Plus）；Apple IIGS（ROM 00、01、3）；Apple-1；Apple III；Franklin、Laser 和许多其他克隆机

**图片支持：** DSK、PO、NIB、HDV、2MG、WOZ、DC42、EDD、CHD

**状态：** 持续开发中（0.250，2022年11月30日发布）

**作者：** 各种各样

**下载地址：** [https://mamedev.org/](https://mamedev.org/)

MAME并非专门的Apple II模拟器。相反，它是一款超级模拟器，于1997年首次发布，现在支持成千上万种复古街机游戏和计算机，如果您能提供所需的机器ROM文件。这是历史游戏模拟中的一项重大成就。

MAME最初模拟街机游戏机，但其范围已扩展到包括个人计算机（通过整合了一个名为MESS的分支项目），其中包括Apple II。MAME的Apple II功能非常强大，但不太好玩，因为它们受到一个与所有其他模拟的街机游戏和计算机共享的不直观的通用界面的限制；直接从命令提示符启动MAME的Apple II模拟可能更容易。此外，没有太多的Apple II特定文档。对于大多数人来说，专用的Apple II模拟器会做得更好。

话虽如此，MAME的理念不仅仅是模拟，而是硬件的永存，因此提供了大量的虚拟历史Apple II接口卡和外设，其中一些我甚至不知道存在。它也是我所知道的唯一一款提供Apple III模拟的现代模拟器，而且MAME可以运行在几乎每个个人计算机平台上。如果您对这些功能感兴趣，MAME可能值得一试。此外，MAME的Apple II功能经常得到改进。MAME的Apple IIGS模拟之所以值得注意，是因为它确实存在，并且可以工作，但它也有一些具有挑战性的方面，比如在窗口模式下没有隐藏宿主机鼠标指针，除非使用一个隐晦的命令行选项。

在使用MAME时，一个重要的键盘命令是前删除（在没有Del键的Mac上是Fn-delete），它在启用MAME的键盘控制和将所有键发送到模拟的Apple II之间切换。另外，如果您希望MAME在窗口中运行，而不是全屏，请从命令提示符启动它，并使用“-window”选项。请注意，MAME图形界面中提供的许多设置既不作为机器状态的一部分保存，也不作为配置的一部分保存，因此您最好完全绕过GUI，并使用命令提示符选项和配置文件编辑来精确指定您在虚拟Apple II中需要的内容。我强烈建议阅读MAME的通用文档以及在主分发站点的wiki中找到的其Apple II专用注释。推荐Mac用户使用Ample（[在此上面评论](https://juiced.gs/emulators/#ample)）以减少启动MAME的难度。

### **LinApple**

**平台：** Linux

**模拟的型号：** Apple II，II Plus，IIe（E）

**图像支持：** DSK，PO，NIB，HDV

**状态：** 不清楚（v2b于2015年6月26日发布；存在更新的分支，但没有正式发布）

**作者：** 各种各样（创建者：Andrey Tzar）

**可从：** [https://linapple.sourceforge.net/](https://linapple.sourceforge.net/)（2015 年发布），[https://github.com/linappleii/linapple](https://github.com/linappleii/linapple)（当前）

LinApple 类似于 2007 年的 AppleWin，但具有不同的用户界面。与 AppleWin 相比，LinApple 缺少一些功能，并添加了一些新功能。LinApple 的大部分代码都是基于 AppleWin 的，因此它很好地模拟了 Apple II，但在操作上与 AppleWin 相比感觉完全不同。没有图形控件。所有操作都通过功能键完成——幸运的是，提供了一个帮助屏幕——并且通过编辑文本文件进行配置。我通常认为这些是负面的，但对于 Linux 来说并不罕见。同样，LinApple 必须从源代码编译。换句话说，LinApple 需要比其他模拟器更多的技术能力。

类似于 AppleWin，LinApple 在它们通常的插槽中提供了常用的虚拟外围设备。包括两个 5.25 英寸软盘驱动器，两个硬盘驱动器，Mockingboard 和 Phasor 声卡，各种显示器仿真，一个打印机卡，可以写入主机操作系统中的文本文件，并且一个鼠标卡。128K 以上的内存不可用。CPU 可以加速，并且可以保存机器状态。新增的功能之一是一个集成的 FTP 浏览器，用于从远程服务器上的镜像加载虚拟磁盘驱动器，尽管这很难实现。

LinApple 的一个迷人之处是作者对项目和 Apple II 显而易见的热情——他的个性体现在帮助屏幕、文档和配置文件的语言中。然而，自 2015 年以来，他已经没有维护该项目。其他人后来接手并增强了它，但并未发布正式版本。

如果你正在使用 Linux，并且想要模拟一个 8 位 Apple II，那么 LinApple 可以胜任这项工作，只要你能克服它的障碍。

## **基于 Web 的 Apple II 模拟器**

有一些在 Web 浏览器中运行的 Apple II 模拟器，即使在“封闭的花园”平台上，如 iOS/iPadOS，以及基于浏览器的计算机，如 Chromebook 上也提供了虚拟 Apple II。它们还为世界上任何一台计算机的用户提供了 Apple II，无需任何软件！由于在浏览器中运行的开发挑战，这些模拟器的功能通常会受到限制，因此我的评价要比在运行本机代码的桌面浏览器上更加宽容。

### **Apple ][js（以及 //jse 和 1js）**

**平台：** Web 浏览器

**模拟的型号：** Apple II、II Plus/J-Plus、IIe（U）、IIe（E）、Apple I

**图片支持：** DSK、PO、D13、NIB、HDV、2MG、WOZ

**状态：** 主动开发中（持续变化，没有正式发布）

**作者：** 威尔·斯卡林

**可从：** https://scullinsteel.com

由Will Scullin开发的Apple ][js（以及Apple //jse和Apple 1js）是一款令人印象深刻的基于Web的开源模拟器，采用TypeScript和HTML5编写。提供了一个软件库，并在典型的插槽中模拟了常见硬件，包括5.25英寸软盘驱动器、硬盘、打印机和鼠标。可以上传本地存储的磁盘映像。并非每个功能都显而易见，而且不幸的是，没有文档。屏幕键盘确保可以发送所有Apple II按键，但在移动电话上，它的键很小，在桌面浏览器上，它的放置位置位于Apple II显示屏下方，降低了屏幕区域。尽管Apple ][js的功能不及本文描述的其他模拟器那么全面，但它成功地执行了基本操作，甚至更多——考虑到Web浏览器的受限环境，这并不是一件小事。

### **cyanIIde**

**平台：**Web浏览器

**模拟的型号：**Apple IIe（E）

**图像支持：**DSK、PO、NIB、HDV、2MG、WOZ

**状态：**积极开发中（2022年8月6日发布了第二次公告）

**作者：**Paleotronic

**来源：**[https://paleotronic.com/cyaniide](https://paleotronic.com/cyaniide)

cyanIIde（读作“氰化物”）是一款由microM8模拟器制造商推出的Web端、无费用、闭源模拟器。它托管在Paleotronic网站上，但主要用于在个人网站上嵌入Apple II程序，尽管关于这样做的文档很少。cyanIIde以在Web浏览器中具有周期精确视频渲染、游戏手柄支持和打印机仿真等特点而著称，还具有独特的Applesoft和汇编语言编辑器。尽管用户界面上的功能不多，但cyanIIde似乎提供了高质量的仿真体验。您只能插入磁盘映像，就这些了。移动设备用户需要一个物理键盘进行输入。cyanIIde可能不是最灵活的用于日常使用的基于Web的模拟器，但如果您想要在您的网站上嵌入一个Apple II，它还是相当酷的。由于使用WebAssembly编写，cyanIIde可能不适用于每个浏览器。

## **其他Apple II模拟器**

尽管它们缺少必要的功能或支持，但还值得一提的是其他一些Apple II模拟器。

### **ApplePi**

**平台：**Linux

**模拟的型号：**Apple II、II Plus、IIe（U）、IIe（E）

**图像支持：**DSK、PO、HDV、2MG

**状态：**积极开发中（2022年2月9日发布了v0.2.3）

**作者：**Bruce Ward

**来源：**[https://github.com/FZBunny/applepi](https://github.com/FZBunny/applepi)

ApplePi 是一个最近为 Linux 用户推出的模拟器，具有软驱和硬盘仿真功能，强大的调试器以及清晰的图形用户界面。它的虚拟外设支持非常有限。虽然设计用于树莓派，但 ApplePi 在任何 Debian 或 Ubuntu 类型的系统上都能很好地工作。（请注意，这个模拟器与 Apple II Pi 项目无关。）不幸的是，ApplePi 不支持 WOZ 或 NIB 格式的磁盘镜像，也不支持 13 扇区磁盘，并且它为所有模拟机器配备了 65C02，即使真实硬件配备了 6502，这可能降低了与一些早期软件的兼容性。尽管存在这些问题，但如果你只需要基础功能，ApplePi 是我在 Linux 上见过的最简单、最吸引人的标题。

### **时钟信号（Apple II 模块）**

**平台：**macOS

**模拟的型号：**Apple II、II Plus、IIe（U）、IIe（E）

**图像支持：**DSK，PO，NIB，HDV，2MG，WOZ

**状态：**积极开发中（v2022-09-16）

**作者：**汤姆·哈特

**可从以下地址获取：**[https://github.com/TomHarte/CLK/releases](https://github.com/TomHarte/CLK/releases)

《时钟信号》（Apple II 模块），由汤姆·哈特编写，是 macOS 的一款多机模拟器，于 2018 年首次获得 Apple II 支持。它专注于高性能和极其准确的模拟模拟视频和音频，涵盖了各种古董机器，不仅仅是 Apple II（它对 Apple II 在模糊 CRT 上的再现确实令人印象深刻）。然而，这个模拟器非常需要一个手册或者一个更易于发现命令的界面；它的极简主义“它只是工作”的设计理念让人难以理解，完整的功能集也是未知的 —— 我甚至搞不清楚如何按重置。一些 bug 也是明显的。作为一个带有非常忠实的低保真 CRT 仿真的双软盘 8 位 Apple II 模拟器，《时钟信号》虽然可用，但由于有更灵活的 Apple II 模拟器可用，它并不是我的首选。

## **未经审查**

有很多 Apple II 模拟器项目我选择排除，因为截至 2022 年 12 月，它们尚未发布，或者需要专门的硬件，或者功能不完整，或者需要在非 Linux 系统上从源代码构建，或者它们没有模拟增强的 Apple IIe 或 IIGS。尽管如此，它们的作者们付出了努力，这些标题都有着自己的关注点和特点，希望它们能成长为出色的 Apple II 模拟器。

这些模拟器被排除在外，因为它们至少不模拟增强的 Apple IIe 或 Apple IIGS：[8bitworkshop.com](http://8bitworkshop.com/?fbclid=IwAR2ePmKmesBUxnrnGJBfCKRfxDrfAyGqGHsVwQSJe3yhgFDLvhZDoQIAXYw)，Accurapple，Bobbin，Epple ][，Reinette II Plus Dot Py。

这些模拟器被排除在外，因为截至 2022 年末，它们尚未有正式发布，或者自 2010 年以来尚未有正式发布，或者需要在非 Linux 平台上从源代码构建：Apple2TS，JACE，XGS。

这些模拟器被排除在外，因为它们的最新版本不适用于当前主要操作系统的版本：Aiie!，pico-iie，Sweet16。

由于截至 2022 年末，这些模拟器的最新版本存在显著的错误或缺失功能，因此被排除在外：Clemens，Steve ][。
