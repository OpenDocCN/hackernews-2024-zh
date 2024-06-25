<!--yml

类别：未分类

日期：2024-05-27 14:47:19

-->

# 什么是“终端”、“壳”、“tty”和“控制台”的确切区别？- Unix和Linux Stack Exchange

> 来源：[https://unix.stackexchange.com/questions/4126/what-is-the-exact-difference-between-a-terminal-a-shell-a-tty-and-a-con](https://unix.stackexchange.com/questions/4126/what-is-the-exact-difference-between-a-terminal-a-shell-a-tty-and-a-con)

终端是电线的末端，壳是乌龟的家，tty是一个奇怪的缩写，而控制台是一种类型的柜子。

从词源学上讲，无论如何。

从Unix术语上讲，简短的答案是

+   终端 = tty = 文本输入/输出环境

+   控制台 = 物理终端

+   壳 = 命令行解释器

* * *

控制台、终端和tty密切相关。最初，它们指的是一种可以与计算机交互的设备：在Unix的早期，这意味着一种类似于打字机的[电传打印机](https://en.wikipedia.org/wiki/Teleprinter)式设备，有时被称为电传打字机，或者简称为“tty”。 “终端”一词来自电子角度，而“控制台”一词来自家具角度。在Unix历史的早期阶段，电子键盘和显示器成为终端的常态。

在Unix术语中，**tty**是一种特定类型的[设备文件](https://en.wikipedia.org/wiki/Device_file)，它实现了除读写之外的一些其他命令（[ioctls](https://en.wikipedia.org/wiki/Ioctl#Terminals)）。在其最常见的含义中，**终端**与tty是同义词。一些tty由内核代表硬件设备提供，例如输入来自键盘，输出发送到文本模式屏幕，或者输入和输出通过串行线传输。其他tty，有时称为**伪终端**，由称为[**终端模拟器**](https://en.wikipedia.org/wiki/Terminal_emulator)的程序（通过一个薄内核层）提供，例如[Xterm](https://en.wikipedia.org/wiki/Xterm)（在[X窗口系统](https://en.wikipedia.org/wiki/X_Window_System)中运行），[Screen](https://en.wikipedia.org/wiki/GNU_Screen)（提供程序和另一个终端之间的隔离层），[SSH](https://en.wikipedia.org/wiki/Secure_Shell)（将一台机器上的终端连接到另一台机器上的程序），[Expect](https://en.wikipedia.org/wiki/Expect)（用于脚本化终端交互），等等。

术语“终端”也可以具有更传统的含义，即通过该设备与计算机进行交互，通常具有键盘和显示器。例如，X终端是一种[瘦客户端](https://en.wikipedia.org/wiki/Thin_client)，是一种特殊用途的计算机，其唯一目的是驱动键盘、显示器、鼠标和偶尔其他人机交互外围设备，实际应用程序在另一台更强大的计算机上运行。

**控制台**通常是物理上连接到计算机的终端。控制台在操作系统中显示为（内核实现的）tty。在某些系统中，如 Linux 和 FreeBSD，控制台显示为多个 tty（特殊的键组合在这些 tty 之间切换）；为了混淆事情，对每个特定的 tty 给出的名称可以是“控制台”、“虚拟控制台”、“虚拟终端”和其他变体。

另请参阅[为什么虚拟终端是“虚拟”的，什么/为什么/在哪里是“真实”的终端？](https://askubuntu.com/q/14284/1059)。

* * *

[**shell**](https://en.wikipedia.org/wiki/Shell_%28computing%29)是用户登录时看到的主要界面，其主要目的是启动其他程序。（我不知道原始的隐喻是 shell 是用户的家庭环境，还是其他程序正在其中运行的 shell。）

在 Unix 环境中，**shell** 已专门用于表示[命令行 shell](https://en.wikipedia.org/wiki/Shell_%28computing%29#Command-line_shells)，其核心在于输入要启动的应用程序的名称，然后是应用程序应该处理的文件或其他对象的名称，最后按下`Enter`键。其他类型的环境不使用“shell”一词；例如，窗口系统涉及“[窗口管理器](https://en.wikipedia.org/wiki/Window_manager)”和“[桌面环境](https://en.wikipedia.org/wiki/Desktop_environment)”，而不是“shell”。

有许多不同的 Unix shell。用于交互使用的流行 shell 包括[Bash](https://en.wikipedia.org/wiki/Bash_(Unix_shell))（大多数 Linux 安装的默认 shell）、[Zsh](https://en.wikipedia.org/wiki/Z_shell)（强调功能和可定制性）和[fish](https://en.wikipedia.org/wiki/Fish_(Unix_shell))（强调简单性）。

命令行 shell 包括流程控制结构以组合命令。除了在交互提示符处键入命令外，用户还可以编写脚本。最常见的 shell 具有基于[Bourne shell](https://en.wikipedia.org/wiki/Bourne_shell)的通用语法。在讨论“**shell 编程**”时，几乎总是指的是 Bourne 风格的 shell。一些经常用于脚本编写但缺乏高级交互功能的 shell 包括[KornShell (ksh)](https://en.wikipedia.org/wiki/KornShell)和许多[ash](https://en.wikipedia.org/wiki/Almquist_shell)变种。几乎任何类 Unix 系统都安装了一个 Bourne 风格的 shell，通常是 ash、ksh 或 Bash。

在 Unix 系统管理中，用户的 **shell** 是在其登录时调用的程序。普通用户账户具有命令行 shell，但具有受限访问权限的用户可能具有[受限 shell](https://en.wikipedia.org/wiki/Restricted_shell)或其他特定命令（例如，仅用于文件传输的账户）。

* * *

终端和 shell 之间的分工并不完全明显。这里是它们的主要任务：

+   输入：终端将键转换为控制序列（例如 `Left` → `\e[D`）。Shell 将控制序列转换为命令（例如 `\e[D` → `backward-char`）。

+   行编辑、输入历史和自动补全由 shell 提供。

    +   终端可以提供自己的行编辑、历史和自动补全，只有当准备好执行一行时才将其发送到 shell。唯一以这种方式运行的常见终端是 Emacs 中的 `M-x shell`。

+   输出：shell 发出指令，比如“显示 `foo`”，“将前景色切换为绿色”，“将光标移到下一行”等。终端执行这些指令。

+   提示符仅是 shell 的概念。

+   Shell 永远不会看到它运行的命令的输出（除非重定向）。输出历史记录（回滚）纯粹是终端的概念。

+   应用程序间的复制粘贴由终端提供（通常使用鼠标或键序列，例如`Ctrl`+`Shift`+`V` 或 `Shift`+`Insert`）。Shell 也可能有自己的内部复制粘贴机制（例如 `Meta`+`W` 和 `Ctrl`+`Y`）。

+   [作业控制](https://zh.wikipedia.org/wiki/%E5%B7%A5%E4%BD%9C%E5%8F%AF%E6%8E%A7%E5%88%B6)（在后台启动程序并管理它们）主要由 shell 完成。然而，处理像`Ctrl`+`C` 杀死前台作业和`Ctrl`+`Z` 暂停它这样的键组合是终端的职责。
