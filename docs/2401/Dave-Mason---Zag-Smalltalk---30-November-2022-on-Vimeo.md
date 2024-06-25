<!--yml

分类：未分类

日期：2024 年 5 月 27 日 15:03:39

-->

# Dave Mason - Zag Smalltalk - 2022 年 11 月 30 日在 Vimeo 上录制

> 来源：[`vimeo.com/802502826`](https://vimeo.com/802502826)

Dave Mason（[sarg.ryerson.ca/dmason/](http://sarg.ryerson.ca/dmason/)）是多伦多大都会大学（以前称为 Ryerson）的计算机科学教授，已经从事了 41 年。他在操作系统、软件可靠性和编程语言方面进行了研究。目前的研究主要集中在 Smalltalk 和其他动态语言上。如果被迫编写非常低级别的项目，比如虚拟机，他愿意使用 Zig 或 Rust，但对于任何其他目的，他坚持使用生产力更高的语言 - 主要是 Smalltalk。

Zag Smalltalk（[github.com/dvmason/Zag-Smalltalk](https://github.com/dvmason/Zag-Smalltalk)）是一个基于原则的 Smalltalk 虚拟机。“基于原则”意味着只有 3 个操作：消息发送、赋值和返回。

1) 它从头开始设计为使用多处理，以利用多核系统。

2) 没有像 ifTrue:ifFalse 或 whileTrue: 这样的方法特殊处理，尽管当然有一些方法是通过原始方法实现的。它会对方法和区块进行积极的内联。

3) “源”代码以 AST 形式维护。

4) 编译代码以线程代码和 JIT 编译的机器代码的双重形式运行，二者之间无缝交互。

5) 它具有部分复制和部分非移动的垃圾收集器。

6) 它保留了更多的值作为立即值，包括符号。

7) 它使用单级调度机制来实现 Smalltalk 的调度语义

研究问题是，“这能够快速到达竞争水平吗？”初步结果是令人鼓舞的。

记录于 2022 年 11 月 30 日。

英国 Smalltalk 用户组是一群对 Smalltalk 编程语言及相关技术感兴趣的专业人士和爱好者。您可以在以下找到我们：

[uksmalltalk.org/](https://www.uksmalltalk.org/) 和 [twitter.com/uksmalltalk](https://twitter.com/uksmalltalk)
