<!--yml

category: 未分类

date: 2024-05-27 14:32:03

-->

# OpenBSD系统调用固定 [LWN.net]

> 来源：[https://lwn.net/SubscriberLink/959562/0578b8e463f790c1/](https://lwn.net/SubscriberLink/959562/0578b8e463f790c1/)

| **这篇文章由LWN订户赞助**LWN.net的订户使本文及其周边内容成为可能。如果您喜欢我们的内容，请[订阅](/subscribe/)并支持我们推出更多文章。 |
| --- |

By **Daroc Alden**

2024年1月31日

[返回导向编程](https://en.wikipedia.org/wiki/Return-oriented_programming) (ROP) 攻击难以防御。诸如地址空间布局随机化、堆栈哨兵和其他技术的部分缓解措施通常被部署以试图挫败ROP攻击。现在，OpenBSD正在尝试一种新的缓解措施，使攻击者更难进行系统调用，尽管一些安全研究人员对其有效阻止真实世界攻击的效果表示怀疑。在他的公告中，Theo de Raadt表示这项工作"<q>在OpenBSD上使一些具体的低级攻击方法变得不可行，这将迫使使用其他方法。</q>"

返回导向编程是一种利用间接跳转来调用进程地址空间中已存在的代码片段的技术家族之一，以攻击者控制的顺序。最初的攻击涉及用精心选择的地址覆盖堆栈，使函数“返回”到一个新位置。自最初的发现以来，已开发出其他相关攻击，这些攻击利用函数指针、信号和其他间接跳转进行跳转。

12月份，De Raadt向OpenBSD邮件列表发送了一个[补丁](/Articles/959664/)，扩展了OpenBSD对进程可以进行系统调用的位置的限制。[之前的提交](https://github.com/openbsd/src/commit/83762a71f74848f4d09174ce350838b4204957c5)添加了一个声明新的ELF段的代码，该段指定了程序中特定系统调用的位置，以便内核在程序试图从错误位置调用系统调用时进行检测。由于OpenBSD没有稳定的系统调用接口（而是建议程序通过C库进行稳定接口调用），新的段不需要显式添加到大多数二进制程序中。现在该补丁已经[合并](/Articles/959883/)，完成了De Raadt表示已经历时五年的过程。

#### 背景

OpenBSD已经限制了程序可以进行系统调用的位置。2019年，De Raadt添加了代码以确保系统调用只能从四个位置进行：在静态二进制的文本中（静态链接C库，因此在运行时没有单独的部分）、在信号跳转区（系统调用需要从信号处理程序返回）、在`ld.so`的文本中（动态链接器需要进行系统调用以设置进程的地址空间）、在`libc.so`的文本中（OpenBSD系统调用存根所在的地方）。

这段代码依赖于新的[`msyscall()`](https://man.openbsd.org/msyscall)系统调用，以便让链接器告知内核`libc.so`（系统C库的共享对象）在地址空间中的映射位置。2023年2月，De Raadt通过引入[`pinsyscall()`](https://man.openbsd.org/OpenBSD-7.4/pinsyscall)来扩展这些保护措施，用于指定进程允许调用[`execve()`](https://man.openbsd.org/execve.2)的二进制中的位置。这两个系统调用只能由给定进程调用一次，由动态链接器完成。

```
    int msyscall(void *addr, size_t len);
    int pinsyscall(int syscall, void *start, size_t len);

```

尽管其签名更通用，`pinsyscall()`仅支持指定`execve()`调用的位置。

这些机制还旨在增加ROP攻击的难度。要求系统调用的地址必须在`msyscall()`块内确保攻击不能利用任何存在于专门指定区域之外的ROP小工具。要求`execve()`调用来自一个特定位置也旨在增加攻击者找到调用位置的难度，因为调用`execve()`以执行攻击者控制的程序是攻击中常见的步骤之一。

新的工作使得这两种机制变得过时，De Raadt建议一旦新代码在一个或两个发布中被采纳，“<q>将msyscall()和功能较弱的pinsyscall(2)变成NOP，并最终移除它们</q>”。

#### 补丁

新的工作增加了一个新的[`pinsyscalls()`](https://man.openbsd.org/pinsyscalls.2)系统调用：

```
    int pinsyscalls(void *start, size_t len, u_int *pintable, int npins);

```

`pinsyscalls()`发送一个“pintable”，指定进程地址空间中预期进行每个可能系统调用的位置。内核使用表中的信息，在进入内核时检查是否从指定位置调用。此检查旨在防止ROP攻击设置系统调用号，然后直接跳转到对应于不同系统调用的系统调用CPU指令。例如，希望进行`execve()`调用的攻击需要跳转到已添加到该调用allowlist的C库中的特定指令，而不是其他存根或仅仅作为系统调用指令解码的无关指令中间。

在设置新进程时，动态链接器使用`pinsyscalls()`通知内核该进程期望从哪里进行系统调用。新工作为选择的程序（`ld.so`、`libc.so`和`libc.a`）添加了一个“openbsd.syscalls” ELF部分。新的ELF部分包含一个程序偏移和系统调用号的数组，指示每个位置期望的系统调用。动态链接器读取此部分并用于向内核提供适当的pintable。链接到C库的程序因此可以立即从这项新保护中受益，而无需更改构建过程。与Linux不同，OpenBSD同时开发内核和用户空间组件，因此这项工作的用户空间组件已经就绪。

安全研究人员对这种检查在防止妥协方面的实用性表示怀疑。一位名叫"stein"的研究人员[指出](https://isopenbsdsecu.re/mitigations/pinsyscall/)：“攻击者如果能够执行ROP，可以简单地使用libc存根，而不是直接发出原始系统调用”，这指的是攻击可能直接跳转到为特定系统调用的allowlist中添加的指令。另一位研究人员Saagar Jha在新补丁上[评论道](https://federated.saagarjha.com/notice/AcmzyfPcwc8KDVlOqm)：“如果你把这个逻辑推到其极致，那就是‘应用程序应指定使用哪些系统调用’，这实际上就是plegde所做的，由内核强制执行，而不是某种奇怪的IP到系统调用号查找方案”。

OpenBSD确实存在现有的缓解措施，旨在使ROP攻击难以确定C库系统调用存根的位置。其中一种保护是[地址空间布局随机化](https://en.wikipedia.org/wiki/Address_space_layout_randomization)（ASLR），许多操作系统长期以来都已标准化。OpenBSD通过在启动时还对C库的各个部分重新以随机顺序进行链接，进一步随机化程序的地址空间，这意味着攻击不仅需要确定C库在内存中的偏移量，还需要确定希望跳转到库内特定代码的偏移量。不幸的是，动态链接的程序必须在符号重定位表中具有此信息，以允许对共享对象的调用。因此，可以构建一种读取内存的方法的攻击经常会泄漏足够的偏移量信息来规避这些保护措施。De Raadt在2023年的CanSecWest上发表了关于OpenBSD中ROP缓解措施的[演讲](https://twitter.com/i/broadcasts/1kvJpmkYkaZxE)（附有[幻灯片](https://www.openbsd.org/papers/csw2023.pdf)），其中包括设计用于增加程序内容泄露难度的几种其他保护措施。

与`pledge()`不同，此补丁的优势在于即使开发人员没有特别努力，也可以确保应用程序的安全。然而，这种保护对静态链接OpenBSD的C库的程序最为有用；使用动态链接的程序仍将在其地址空间中具有C库使用的所有系统调用。`pledge()`还允许在启动后丢弃不必要的权限，从而允许应用程序使用比`pinsyscalls()`静态防御更为严格的权限集。

这项工作难以完成。其中最大的障碍之一是使用Go编写的程序。在宣布新工作已合并的公告中，De Raadt说道：“<q>go独有的二进制内直接系统调用模型（在Unix软件历史中，只有go这样做）对这一努力提供了最大的阻力</q>”。他特别感谢Joel Sing在使Go生态系统与这些变化兼容方面的工作。

由于Linux允许程序直接进行系统调用，而无需通过受赞同的C库包装器，且不太可能改变此策略，因此需要采取额外步骤来在那里整合类似的机制。某些Linux程序直接进行系统调用以避免依赖特定的C库，但其他程序则直接进行系统调用以使用尚未由系统C库包装的新功能；OpenBSD没有这个问题，因为其C库和内核是同步开发的。

#### 结论

OpenBSD有着长期的历史，不断添加新颖的缓解措施，其中一些被其他项目采纳，而另一些则没有。考虑到对实际效益的质疑以及增加系统调用复杂性成本，这项工作似乎不太可能被其他地方采纳。然而，这项工作确实为在OpenBSD上构建ROP攻击增加了另一个屏障，对于仅使用少量系统调用且尚未使用承诺（pledge）的静态链接程序似乎尤为有益。

* * *

(

[登录](https://lwn.net/Login/?target=/Articles/959562/)

to post comments)
