<!--yml

分类：未分类

日期: 2024-05-29 13:23:03

-->

# 应该整合 Shell, Awk 和 Make

> 来源：[https://www.oilshell.org/blog/2016/11/13.html](https://www.oilshell.org/blog/2016/11/13.html)

| [博客](/blog/) | [oilshell.org](/)

## 应该整合 Shell, Awk 和 Make

2016-11-13

[Unix 的 shell](/cross-ref.html?tag=shell#shell)自从 1971 年初代发布以来，就一直是 Unix 的组成部分，而 [awk](/cross-ref.html?tag=awk#awk) 和 [Make](/cross-ref.html?tag=make#make)则是后来添加到 Unix 中的，两者都是在 1977 年发布，由贝尔实验室开发的。

这三个工具最初作为专门领域语言设计，具有**明确的角色**：

+   Shell 关注于进程间的**顺序与并行 构造**，包括管道。

+   Awk 关注于**数据流计算**，在数据的行上进行计算。它具有正则表达式和关联数组，预示了 Perl、Python、Ruby、JavaScript 等语言的发展。

+   Make 关注于**数据驱动、增量和并行计算**，同样地，它使用 Unix 进程。

时至今日，它们尽管经历了四十年，依然相当受欢迎：

+   Windows 10 的一个重要新特性是能够运行 27 年前的程序：bash。这一改进显然对已经存在了三十年的操作系统的开发环境有所提升。

+   尽管存在许许多多“Make 替换版本”，但在 Unix 环境中，它是最常见的构建工具。直到 2016 年，仍然还在编写 Makefile 教程。2015 年，围绕 GNU Make 的经典书籍《GNU 建设艺术》问世。

+   Awk 因其后继者的采用而不那么受欢迎，但仍然深深嵌入了 Linux、FreeBSD 和其他基础软件的构建过程中。

这些工具的成功导致了一个常见的反模式：它们发展成了像 ALGOL 类似的语言，**具有古怪的语法**，并由向后兼容性所约束。

明天，我将通过三种语言的示例代码展示这一点。在此期间，我将用一些观察来支持此观点：

+   常常可以看到所有三个语言在 **同一源代码目录** 内共存（比如 Linux 和 FreeBSD），这在一定程度上增加了全面理解的难度。

+   不仅它们位于同一目录树内，而且它们经常**嵌入**在彼此中。当你在 Makefile 中看到 awk 时，实际上是**三个**交织的语言，因为 make 是一个 **宏语言**，它直接将命令字符串传递给 `/bin/sh`。

组合两种语言时编码困难，更不用说三种语言了。这是我 `~/src` 文件夹中的一个示例：

```
$ grep AWK */Makefile
busybox-1.24.2/Makefile:        ($(OBJDUMP) -h $< | $(AWK) '/^ +[0-9]/{print $$4 " 0 " $$2}'; $(NM) $<) | sort > $@
cityhash-1.1.1/Makefile:  $(AWK) 'BEGIN { files["."] = "" } { files[$$2] = files[$$2] " " $$1; \
dash-0.5.8/Makefile:      $(AWK) '{ files[$$0] = 1; nonempty = 1; } \
glibc-2.23/Makefile:    AWK='$(AWK)' scripts/check-local-headers.sh \
re2c-0.16/Makefile:  $(AWK) 'BEGIN { files["."] = "" } { files[$$2] = files[$$2] " " $$1; \

```

+   Make 和 Shell 可悲地存在着**语法冲突**。例如：在 Make 中，`$@` 表示规则的 *输出文件*，但在 Shell 中，它是 *参数数组*。

+   典型的 `Makefile` 中，80% 的行都是**纯粹的 shell 代码**，或者是容易用 shell 表达的变量赋值。很容易想象扩展 shell 语法以支持 Make 的语义。

+   Make 和 shell 都是用分隔字符串来[模拟数组](06.html)，因此存在引用问题。

至于 awk：

+   `bash` 和 `zsh` 在 shell 中添加了**关联数组**和**正则表达式支持**，使它们在语义上与 `awk` 接近。

+   Awk 只有浮点**算术**，而 bash 只有整数算术。很容易想象一种同时具有两者的语言。

+   Awk 和 shell 在一个有趣的方面是相似的：它们都缺少**垃圾回收**。这导致数组既不能嵌套也不能从函数中返回。

这三种语言共同之处在于：

## 结论

尽管它们的起源不同，但现在这三种语言在语义上更相似而不是不同的。明天我们会看到它们的语法是多么地狂野而又毫无必要地不同。

尽管我在没有代码的情况下讨论计划时总是很谨慎，但我想说我希望 `oil` 也能替代 `awk` 和 `make`。我预计这个主题将会占据更多的博客文章。
