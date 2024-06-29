<!--yml

category: 未分类

date: 2024-05-27 13:29:20

-->

# 为什么Windows真的使用反斜杠作为路径分隔符？ | OS/2博物馆

> 来源：[https://www.os2museum.com/wp/why-does-windows-really-use-backslash-as-path-separator/](https://www.os2museum.com/wp/why-does-windows-really-use-backslash-as-path-separator/)

几乎所有现代PC用户都会有一个疑问：为什么Windows使用反斜杠作为路径分隔符，而其他地方使用正斜杠？明显的中间答案是“因为DOS和OS/2使用了反斜杠”。Windows 9x和NT无论是直接还是间接地都是从DOS和OS/2衍生出来的，并且在很大程度上继承了DOS的文化背景。

当然，这并不是一个很好的答案。显而易见的下一个问题是，为什么DOS使用反斜杠作为路径分隔符？当DOS 2.0添加对分层目录结构的支持时，它受到了UNIX（或更确切地说是XENIX）的很大影响，使用正斜杠作为路径分隔符本应是一个逻辑的选择。这是所有人都可以同意的。除此之外，事情就有些混乱了。

唯一清楚的是微软和IBM在DOS 2.0中使用反斜杠作为路径分隔符。据报道，微软当初想使用正斜杠作为路径分隔符，但IBM否决了这个想法，因为这会与已经使用正斜杠作为开关字符的DOS 1.x不兼容。

微软的老员工都一致认为，IBM曾强烈反对将斜杠作为开关字符。但他们对于这种特定用法的起源并不太清楚。

有些[荒谬的理论](https://superuser.com/questions/176388/why-does-windows-use-backslashes-for-paths-and-unix-forward-slashes)，比如“斜杠来源于CP/M”。其实不是的。实际上没有证据表明CP/M在任何地方使用正斜杠，除了产品名称。大多数CP/M命令根本没有选项。第三方的CP/M工具（比如微软的）可能会使用斜杠，但操作系统本身并没有。

有些明显荒谬的理论，比如“CP/M从VMS那里得到了斜杠”，这是不可能的，因为CP/M比VMS更老，并且CP/M本身根本不使用正斜杠。

SCP的86-DOS同样没有使用正斜杠。这意味着DOS没有从其直接或间接的前辈（86-DOS和CP/M）那里继承正斜杠。它必定来自其他地方。我们能找到这个来源吗？

事实上，即使是PC DOS 1.0（1981年），也极少使用正斜杠。FORMAT命令有/S选项用于复制系统文件，LINK有/P选项在写入结果可执行文件之前暂停，以便可以更换软盘。就是这样。

值得注意的是，LINK和FORMAT都是由微软编写的。一个重要的点是，PC DOS 1.0在发布时确实使用正斜杠作为选项分隔符。

PC DOS 1.1（1982年）显著增加了斜杠的使用。COPY命令有/A、/B和/V等开关，DIR命令有/P和/W等开关，而DISCKOMP、DISKCOPY和FORMAT现在都有/1开关（用于单面软盘操作）。链接器也新增了选项，如/DSALLOCATION、/HIGH、/LINE、/MAP、/PAUSE和/STACK。

那种斜杠的使用方式毫无疑问是IBM坚持将其保留在DOS 2.0中的原因。更改斜杠的语义显然具有破坏数据的潜力，特别是在运行为DOS 1.1编写的批处理文件时。例如像 'COPY FOO + BAR /A'，当/A是一个开关时，其语义与/A是磁盘根目录中的文件或目录时完全不同。

回顾来看，为了保留与一个短命的早期PC操作系统的向后兼容性而更改路径分隔符似乎有些愚蠢，但事后诸葛亮，1982年时UNIX及其衍生品的未来流行显然并不明显。

无论如何，显然DOS自1.0版本起就使用了斜杠，即使其直接前身CP/M和86-DOS没有使用。那么，它是从哪里来的呢？旧版Microsoft语言产品的手册提供了一个很好的提示。[Microsoft为8080提供的工具](ftp://bitsavers.informatik.uni-stuttgart.de/pdf/microsoft/cpm/Microsoft_FORTRAN-80_Users_Manual_1977.pdf)（如F80 FORTRAN编译器、M80宏汇编器、8080链接器）都将斜杠用作分隔命令行选项的开关字符，至少从1977年开始就是如此。换句话说，Microsoft甚至在8086存在之前就已经使用斜杠作为开关字符，并继续在[基于8086的工具](ftp://bitsavers.informatik.uni-stuttgart.de/pdf/microsoft/cpm/Microsoft_8086_Utility_Software_Package_1981.pdf)中使用它。

Microsoft发明了斜杠的使用吗？当然没有。来自Hans Spiller的[Larry Osterman博客上的线索](https://web.archive.org/web/20140507004133/http://blogs.msdn.com/b/larryosterman/archive/2005/06/24/432386.aspx?PageIndex=2#comments)提供了一个很好的提示，他是一位早期的Microsoft编译器作者。即使是这个线索看起来也是错误的，因为所声称的历史是从DEC TOPS-10到CP/M再到DOS，即使CP/M本身并没有使用斜杠。但是TOPS-10确实使用了大量的斜杠。

在他的[*Computer Connections*](https://d1yx3ys82bpsa0.cloudfront.net/kildall-p.1-78-publishable-lowres.pdf)回忆录中，Gary Kildall声称（第26页），Bill Gates和Paul Allen在使用DEC PDP-10分时系统时几乎肯定是在运行TOPS-10，当时他们偷窃了其他用户的密码（因为操作系统未经擦除即回收了其他用户使用的内存页面，留下包括明文密码在内的信息）。

Microsoft's own *MS-DOS Encyclopedia*（1988）提到，早期微软员工Marc McDonald开发了一个名为M-DOS的8位多任务操作系统，该系统“仿照DEC TOPS-10操作系统”。同一本书还明确指出，“MS-DOS的1.x版本从DEC操作系统的传统中借鉴，已经在命令行中使用斜杠作为开关”。

在这里，微软自己说斜杠不是来自CP/M，也不是来自IBM，而是来自DEC，明确提到了[TOPS-10](https://en.wikipedia.org/wiki/TOPS-10)。我们能找到支持的证据吗？

### 这就是TOPS-10！

DEC TOPS-10可以追溯到1960年代末，肯定足够早以至于CP/M和DOS可能受其影响。另一方面，UNIX并不足够早以至于对CP/M没有任何影响，因为它们几乎是在同一时间内开发的，即1970年代初。虽然UNIX对DOS 2.0有很大影响，但几乎没有证据表明它在任何方面影响了DOS 1.x（而CP/M显然是一个非常主要的影响）。

DEC PDP-10主机出现在1966年，并自1970年起作为DECsystem-10进行市场推广，运行TOPS-10操作系统。配备TOPS-10的PDP-10可能是第一个广泛使用的分时系统，并在1970年代被许多大学采用。因此，许多早期的微软员工，包括盖茨和艾伦，可能对TOPS-10很熟悉。

如今，很容易找到[TOPS-10手册](ftp://bitsavers.informatik.uni-stuttgart.de/scandocs.trailing-edge.com/tops10-aa-0916d-tb.pdf)，浏览其中的内容会有很大的收获。这里举个简短的例子（第2-220页）：

```
.DIR PROG
PROG        FOR     1 <055>    10-SEP-79    DSKC: [27,5434] 
PROG        REL     1 <055>    10-SEP-79
PROG        EXE    68 <055>    10-SEP-79 
  TOTAL OF 131 BLOCKS IN 5 FILES ON DSKC: [27,5434]
```

DIR命令列出目录内容。文件名具有固定长度和三字符扩展名。可执行文件的扩展名为.EXE。磁盘名称末尾带有冒号。这看起来非常像DOS，与UNIX毫不相似。

注意，方括号中的奇数是指定目录的TOPS-10方式。幸好DOS没有采纳这种方式。还要注意，DOS复制了不明显的行为，其中‘DIR PROG’等同于‘DIR PROG.*’，将列出带有任何扩展名的文件。这很不可能是巧合。

TOPS-10标准文件扩展名列表（手册附录D）让人想起了很多DOS的记忆。可执行文件的扩展名为.EXE仅是其中之一；还有用于目标文件的.OBJ，用于符号文件的.SYM，用于交叉引用文件的.CRF，用于系统文件的.SYS，用于BASIC源文件的.BAS，或用于ASCII文本文件的.TXT。这些文件扩展名非常类似DOS，与UNIX则完全不同。

[TOPS-10 宏汇编器](http://bitsavers.org/pdf/dec/pdp10/TOPS10_softwareNotebooks/vol13/AA-C780C-TB_Macro_Assembler_Reference_Manual_Apr78.pdf) 也使用了从 MASM 知名的 IRP 宏构造，但其他汇编器（特别是 Intel 的 ASM86 使用完全不同的宏语法）则不使用。微软极有可能是在 DEC 的汇编器中找到了 MASM 的灵感。

### 案件结案？

只有微软的老员工可能能回答这些问题，但必须说明，40 年后人类的记忆力往往不可靠。然而，有历史证据表明微软使用了 TOPS-10，有强烈的间接证据表明 DOS 的几个方面受到了 TOPS-10 的影响（包括使用前斜杠作为选项分隔符），并且《MS-DOS 百科全书》明确表示斜杠的使用来自 DEC 的操作系统，包括对 TOPS-10 的提及。

可以说，今天的 Windows 使用反斜杠作为路径分隔符，是因为 50 年前的 TOPS-10 使用了前斜杠作为选项分隔符。

**更新：** 在发布这篇文章后，我意识到虽然它很好地解释了为什么 DOS 没有使用斜杠作为路径分隔符，但对为什么改用反斜杠却模糊不清。答案在于 [IBM Model F 键盘](https://en.wikipedia.org/wiki/Model_F_keyboard#/media/File:IBM_Model_F_XT.png)。前斜杠和反斜杠的视觉相似性可能起到了一定作用，但决定性因素是人体工程学。

路径分隔符需要一个不需要按 Shift 键输入且没有已建立使用的按键（例如句点、逗号或分号键，这些键多数被微软语言工具使用）。反引号（`）可能是唯一可用的其他键，考虑到选择，反斜杠更合理。

非美国键盘的人体工程学自然没有考虑到，这导致了路径分隔符在某些国家的键盘上（例如德国）极为难以输入，或许这是对人体工程学重要性的一种反讽提醒。
