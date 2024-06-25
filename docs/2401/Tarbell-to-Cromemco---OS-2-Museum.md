<!--yml

category: 未分类

date: 2024-05-27 15:05:16

-->

# Tarbell to Cromemco | OS/2 Museum

> 来源：[`www.os2museum.com/wp/tarbell-to-cromemco/`](https://www.os2museum.com/wp/tarbell-to-cromemco/)

在玩弄[旧版本的 86-DOS](https://www.os2museum.com/wp/86-dos-revisited/)时，我找到了[86-DOS 1.14](https://archive.org/details/86-dos-1.14)的磁盘映像。我在 SIMH 模拟器中运行了较旧版本的 86-DOS，该模拟器可以模拟 86-DOS 支持的 Cromemco 磁盘控制器。

不幸的是，86-DOS 1.14 磁盘是用于与不同且不兼容的 Tarbell 磁盘控制器一起使用的。SIMH 不模拟 Tarbell 控制器，因此无法启动磁盘。

然后我意识到 86-DOS 随附了所有所需的源代码，应该可以将 Tarbell 磁盘改为使用 Cromemco 磁盘控制器。两个 86-DOS 变种（Cromemco 和 Tarbell）使用完全相同的 8 英寸软盘格式（77 个磁道，每个磁道 26 个 128 字节的扇区），只是磁盘控制器不同，因此启动加载程序和 I/O 系统（都位于保留的磁道内）不同；FAT 文件系统上的实际文件相同。

86-DOS 1.14 磁盘上包含了所有必需的内容，唯一缺少的是一种编辑和重新组装源文件，并将生成的目标代码放置在磁盘上的方法。但这实际上也不太困难 — 唯一额外的要求是具有 SIMH 引导功能的 86-DOS 磁盘。幸运的是有一张适用的 86-DOS 1.10 磁盘。有了这个，就可以将（可启动的）Cromemco 86-DOS 1.10 磁盘和（不可启动的）Tarbell 86-DOS 1.14 磁盘都连接到 SIMH 并启动。

引导盘将是 A：而 B：将是尚未可启动的 Tarbell 磁盘。1.14 磁盘上的工具在 86-DOS 1.10 下运行时没有任何问题。

#### 准备 SIMH 环境

在任何工作开始之前，必须使用 IMDU 实用程序将压缩的 IMD 映像转换为未压缩的映像，以便 SIMH 可以对其进行写入。这已经完成了，并且可以通过此处获取带有所需磁盘映像和 SIMH 的存档。最初用于运行 86-DOS 的 SIMH 模拟器来自[这里](https://web.archive.org/web/20080819181906/http://www.86dos.org/)。

要使用双磁盘设置，请在 Windows 机器上运行“altairz80 tartocro”。请记住，要退出 SIMH，可以使用 Ctrl-Break。

86-DOS 启动后，第一步是编辑 BOOT.ASM 和 DOSIO.ASM，以支持 Cromemco 而不是 Tarbell 控制器。

为此，有 EDLIN，它... 不是一个很好的编辑器，但*确实*可以工作。基本的 EDLIN 文档可以在[Bitsavers](http://bitsavers.informatik.uni-stuttgart.de/pdf/seattleComputer/86-DOS_0.3_Users_Manual_1980.pdf#page=13&zoom=auto,-138,657)上找到。EDLIN 有自己的命令行；要访问特定行，请输入行号。

#### 修复 BOOT.ASM

在 BOOT.ASM 中，必须启用 CROMEMCOLARGE 并禁用 TARBELLDOUBLE。要做到这一点，将第 18 行从 ‘CROMEMCOLARGE: EQU 0’ 改为 ‘CROMEMCOLARGE: EQU 1’，将第 20 行从 ‘TARBELLDOUBLE: EQU 1’ 改为 ‘TARBELLDOUBLE: EQU 0’。

请注意，在 EDLIN 中复制模板（前一行内容）时，F3 键不起作用，但可以按 Esc，然后按 U（这必须是大写的 U，不是小写的 u；换句话说，首先按 Esc，然后按 Shift+U）。

写出修改后的文件，并在 EDLIN 命令行上输入‘E’退出 EDLIN（在此版本中，E 可以是小写）。SIMH 输出将看起来像这样：

```
B:edlin boot.asm

EDLIN  version 1.01
End of input file
*18
    18:*CROMEMCOLARGE:  EQU     0
    18:*CROMEMCOLARGE:  EQU     1
*20
    20:*TARBELLDOUBLE:  EQU     1
    20:*TARBELLDOUBLE:  EQU     0
*e

B:
```

#### 修复 DOSIO.ASM

现在需要类似地修改 DOSIO.ASM。必须禁用 TARBELLDD，启用 CROMEMCO4FDC，并额外启用 FASTSEEK（否则源文件将无法汇编）。要做到这一点，将第 26 行从 ‘TARBELLDD: EQU 1’ 改为 ‘TARBELLDD: EQU 0’，将第 27 行从 ‘CROMEMCO4FDC: EQU 0’ 改为 ‘CROMEMCO4FDC: EQU 1’，最后将第 47 行从 ‘FASTSEEK: EQU 0’ 改为 ‘FASTSEEK: EQU 1’。

#### 汇编修改后的源文件

现在汇编修改后的源文件。为了避免创建占用太多磁盘空间的大型列表文件，需要使用 SCP 汇编器的相当奇怪的语法，如下所示：

```
B:asm boot.bbz

Seattle Computer Products 8086 Assembler Version 2.40
Copyright 1979,80,81 by Seattle Computer Products, Inc.

Error Count =    0

B:asm dosio.bbz

Seattle Computer Products 8086 Assembler Version 2.40
Copyright 1979,80,81 by Seattle Computer Products, Inc.

Error Count =    0

B:
```

汇编器始终假定源文件的扩展名为 .ASM。假的‘bbz’扩展名告诉 ASM 在 B: 驱动器上查找源 .ASM 文件，还使用 B: 驱动器来输出生成的 .HEX 文件，并且不生成列表文件（.PRN）。

现在需要将 HEX 文件转换为二进制（.COM）文件。这可以通过 HEX2BIN 实用程序完成。只需运行‘hex2bin boot’和‘hex2bin dosio’。现在应该有一个新的 BOOT.COM 和 DOSIO.COM。

#### 安装修改后的代码

最后一步需要将引导扇区和 I/O 系统写入磁盘的正确位置。引导扇区是第一个扇区，I/O 系统紧随其后位于第一个磁道上。幸运的是，DEBUG 实用程序可以完成所有繁重的工作。

要将引导扇区放置在正确位置，运行‘debug boot.com’，然后输入‘w 100 1 0 1’，最后使用‘q’退出 DEBUG。像这样：

```
B:debug boot.com

DEBUG-86  version 1.03
>w 100 1 0 1
>q

B:
```

W 命令有四个参数。第一个是起始地址，在本例中为 100h，因为 .COM 文件加载到程序段的地址 100h。第二个参数是驱动器，以 DEBUG 中的驱动器 B: 为驱动器 1。第三个参数是起始记录/扇区，零是引导扇区。第四个也是最后一个参数是要复制的记录数（一个）。

现在必须为 I/O 系统重复这个过程。运行‘debug dosio.com’，然后‘w 100 1 1 8’，用‘q’退出 DEBUG。

```
B:debug dosio.com

DEBUG-86  version 1.03
>w 100 1 1 8
>q

B:
```

W 命令几乎相同，只是它写入 8 个扇区，并从逻辑扇区 1 开始（即在引导扇区之后的扇区）。

#### 启动它

如果一切顺利，修改后的 86-DOS 1.14 磁盘现在可以在 SIMH 中启动：

```
X:\tarcro>altairz80.exe 86dos114

Altair 8800 (Z80) simulator V3.8-1
2047 bytes [8 pages] loaded at 0.
2047 bytes [8 pages] loaded at ff800.

Press <return> to get monitor prompt, then type 'B' to boot

SCP 8086 Monitor 1.5
>B
☺
86-DOS version 1.14
Copyright 1980,81 Seattle Computer Products, Inc.

COMMAND v. 1.10
Current date is 00-00-19 80
Enter new date:
```

这就是！在模拟的 Cromemco 控制器上运行的 86-DOS 1.14。所有必要的工具都在 1.14 磁盘上，唯一缺少的是运行它们的方法。借用一个现有的 SIMH 兼容的 86-DOS 磁盘解决了这个问题。
