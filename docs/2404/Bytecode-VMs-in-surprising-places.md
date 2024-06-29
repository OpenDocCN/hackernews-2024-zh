<!--yml

category: 未分类

date: 2024-05-27 13:40:55

-->

# 令人惊讶的地方的字节码虚拟机

> 来源：[https://dubroy.com/blog/bytecode-vms-in-surprising-places/](https://dubroy.com/blog/bytecode-vms-in-surprising-places/)

2024年4月30日

在Twitter上回答一个问题时^(，Richard Hipp写到[为什么SQLite使用字节码虚拟机](https://sqlite.org/draft/whybytecode.html)来执行SQL语句。)

大多数人可能将字节码虚拟机与通用编程语言（如JavaScript或Python）联系起来。但有时它们出现在令人惊讶的地方！这里是我知道的一些例子。

## eBPF

你知道在Linux内核内部有一个包括字节码解释器和JIT编译器的扩展机制吗？

我不知道。好吧，它被称为[eBPF](https://en.wikipedia.org/wiki/EBPF)，它相当有趣：一个具有十个通用寄存器和超过一百种不同操作码的寄存器式虚拟机。

eBPF中的“BPF”代表*Berkeley packet filter*，其基本思想在[1993年的USENIX论文](https://www.tcpdump.org/papers/bpf-usenix93.pdf)中描述：

> 许多版本的Unix提供用户级别的数据包捕获功能，使得可以使用通用工作站进行网络监控。因为网络监视器作为用户级别进程运行，数据包必须通过内核/用户空间保护边界进行复制。通过部署称为数据包过滤器的内核代理，可以将此复制最小化，并尽早丢弃不需要的数据包。原始的Unix数据包过滤器设计围绕一个栈式过滤器评估器，该评估器在当前的RISC CPU上执行效果不佳。BSD数据包过滤器（BPF）使用了一个新的、基于寄存器的过滤器评估器，比原始设计快了多达20倍。

因此，最初它被设计用于一个相当受限制的用例：表示网络数据包的一个有向无环控制流图的过滤函数。很长一段时间以来，Linux的实现同样简单：两个通用寄存器，一个开关式解释器，没有反向分支。

2011年的一个补丁为[x86-64的JIT编译器](https://lwn.net/Articles/437981/)。2012年，[首个非网络使用案例出现](https://lwn.net/Articles/475043/)。然后，2014年，在成为[内核通用虚拟机](https://lwn.net/Articles/599755/)的路上，BPF实现得到了大幅扩展：

> 它扩展了可用寄存器集，从两个扩展到十个，增加了许多与真实硬件指令紧密匹配的指令，实现了64位寄存器，使得BPF程序可以调用（严格控制的）一组内核函数，等等。内部BPF更容易编译成快速的机器码，并使得将BPF挂接到其他子系统变得更容易。

## DWARF表达式

DWARF是由像GCC和LLVM这样的编译器使用的文件格式，用于在编译后的二进制文件中包含调试信息。假设你正在尝试调试以下C++代码：

```
void add_two(int x)
{
    int ans = x + 2;
    return ans + 2; // Oops!
}
```

在调试器中，您可能想要打印`ans`变量的值。但根据编译器和代码的不同，这可能会出乎意料地困难！它可能在寄存器中，堆栈上，或者可能已经被优化掉（只是几种可能性之一）。

解决方案是允许编译器指定一个*表达式*来计算本地变量的值。因此，[DWARF规范](https://dwarfstd.org/doc/DWARF5.pdf)包括一个表达式语言：

> **2.5 DWARF表达式**
> 
> DWARF表达式描述如何计算值或指定位置。它们以操作DWARF操作栈为基础表达。
> 
> DWARF表达式被编码为一系列操作流，每个操作由操作码和零个或多个文字操作数组成。操作数的数量由操作码隐含确定。

由调试器来评估表达式。GDB和LLDB都有基于开关的DWARF表达式解释器。

## GDB代理表达式

但事实证明，GDB还有另一个字节码解释器！

> 使用GDB的`trace`和`collect`命令，用户可以指定程序中的位置，并在达到这些位置时评估任意表达式。
> 
> 当GDB调试远程目标时，运行在目标上的GDB代理代码自己计算表达式的值。为了避免在代理上具有完整的符号表达式求值器，GDB将源语言中的表达式转换为一个更简单的字节码语言，然后将字节码发送到代理；代理然后执行字节码，并记录值供GDB稍后检索。
> 
> 字节码语言很简单；有四十多个操作码，其中大部分是C操作的常见词汇（加法、减法、移位等）以及各种大小的文字和内存引用操作。字节码解释器严格操作在机器级别的值上 —— 各种大小的整数和浮点数 —— 并且不需要类型或符号信息；因此，解释器的内部数据结构简单，并且每个字节码只需要几个本机机器指令来实现。解释器体积小，易于确定评估表达式所需的内存和时间限制，适合用于实时应用程序中的调试代理。

*(来源自[使用GDB调试，附录F：GDB代理表达式机制](https://sourceware.org/gdb/current/onlinedocs/gdb.html/Agent-Expressions.html))*

## WinRAR

WinRAR ^(是Windows的文件压缩实用程序，具有专有文件格式。Google的漏洞研究员Tavis Ormandy [发现RAR格式包含用于数据转换的字节码编码](https://blog.cmpxchg8b.com/2012/09/fun-with-constrained-programming.html)：)

> 信不信由你，RAR文件可以包含一个名为RarVM的简单类似x86的虚拟机的字节码。这设计为提供过滤器（预处理器），以对输入数据执行一些可逆转换，增加冗余性，从而提高压缩效果。

他的[rarvmtools仓库](https://github.com/taviso/rarvmtools)包含了一些关于架构的细节：

> 熟悉x86（最好是英特尔汇编语法）将是一个优势。
> 
> RarVM有8个命名寄存器，称为r0到r7。r7用作与堆栈相关操作（如推送、调用、弹出等）的堆栈指针。然而，与x86一样，对于r7没有设置限制，尽管如果执行堆栈相关操作，它将被屏蔽以适应地址空间，在该操作的持续时间内。

还有一些其他信息：

+   [TrueType](https://developer.apple.com/fonts/TrueType-Reference-Manual/)字体规范包括一组超过200条指令，用于字形的渲染和提示。

+   [PostScript](https://en.wikipedia.org/wiki/PostScript)不仅是一种页面描述语言，而且是一种相对强大的基于堆栈的编程语言。PostScript文件是纯文本，因此PostScript渲染器不一定使用字节码，但规范也包括了二进制编码。

👉 *[在Hacker News上讨论](https://news.ycombinator.com/item?id=40211205)*。
