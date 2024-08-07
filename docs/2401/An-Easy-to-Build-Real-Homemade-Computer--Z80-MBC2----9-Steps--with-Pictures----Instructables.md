<!--yml

类别：未分类

日期：2024-05-27 14:38:36

-->

# 一款易于构建的真正家庭制作计算机：Z80-MBC2！：9 步（附图片）-Instructables

> 来源：[`www.instructables.com/An-Easy-to-Build-Real-Homemade-Computer-Z80-MBC2/`](https://www.instructables.com/An-Easy-to-Build-Real-Homemade-Computer-Z80-MBC2/)

如果您想知道计算机是如何工作并与“外部事物”进行交互的，现在有很多准备好的板可以使用，例如 Arduino 或 Raspberry 等。但是这些板都有相同的“限制”…它们隐藏了内部部件，因为它们使用了 MCU（微控制器单元）或 SOC（片上系统），因此您无法触及 CPU、I/O、内部总线等这些使计算机工作的东西。

还有另一种选择，使用一些旧的 8 位 CPU（所谓的“复古计算”）。它们易于理解，您可以免费找到大量文档和书籍，并且允许构建具有所有必需功能块（CPU、I/O、RAM、ROM/EPROM 等）的真正计算机。

但一般它们使用难以找到的零件，并需要过时的工具，比如一个 EPROM 编程器和擦除器或一个 GAL 编程器，而更简单的则功能非常有限。

因此，我混合了旧的和“新”的部件，制作出了一种独特的设计，不需要任何传统的 EPROM 编程器或花哨的集成电路，使用易于找到的元件。 Atmega32A MCU 充当 I/O 子系统，“模拟”EPROM 和所有 I/O 组件。此外，使用 Arduino 引导加载程序，可以轻松使用众所周知的 Arduino IDE 进行编程。

所需的集成电路有：

+   Z80 CPU CMOS（Z84C00）8Mhz 或更高

+   Atmega32A

+   TC551001-70（128KB RAM）

+   74HC00

如果您想要 16x GPIO 扩展（GPE 选项），还需添加一个 MCP23017。

Z80-MBC2 具有多引导功能，可以运行 CP/M 2.2、QP/M 2.71 和 CP/M 3（支持 128KB 分段内存），因此您可以使用大量的软件（例如，您可以轻松找到 Basic、C、汇编、Pascal、Fortran、Cobol 编译器，其中一些已经提供在 SD 上的虚拟磁盘中）。

硬盘使用微型 SD FAT16 或 FAT32 格式化（1GB 微型 SD 足够），因此可以轻松与 PC 交换文件（每个操作系统支持 16 个硬盘），使用**[cpmtoolsGUI](http://star.gmobb.jp/koji/cgi/wiki.cgi?action=ATTACH&page=CpmtoolsGUI&file=CPMTG%5FENG%5F20180903%2Ezip)**。

当然，您需要一个终端来与 Z80-MBC2 进行交互，一个常见的 USB 串行适配器，再加上一个终端仿真软件，将是一个便宜且简单的选择。
