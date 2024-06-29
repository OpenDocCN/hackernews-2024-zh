<!--yml

类别：未分类

日期：2024-05-27 14:37:05

-->

# 介绍

> 来源：[https://anaseto.codeberg.page/goal/chap-intro.html](https://anaseto.codeberg.page/goal/chap-intro.html)

[主页](/)。关于[Goal](https://codeberg.org/anaseto/goal)的进行中文档。*最近更新：2024-05-25*。

# 1 介绍

Goal是一种可嵌入的[数组编程语言](https://en.wikipedia.org/wiki/Array_programming)，具有Go编写的字节码解释器。命令行解释器可以执行脚本或在交互模式下运行。有关安装，请参阅项目存储库中的`README.md`文件（[链接](https://codeberg.org/anaseto/goal)）。您还可以在浏览器上[尝试Goal](https://anaseto.codeberg.page/try-goal/)。当前文档的EPUB版本可在[此处](https://anaseto.codeberg.page/misc/goal/goal-docs-gen.epub)获取。

与大多数数组编程语言一样，Goal的内置函数对不可变数组进行矢量化操作，并通过简单的动态类型系统支持控制和数据转换的功能风格，具有少量抽象和可变变量（但无可变值）。

您可以在FAQ的[起源部分](chap-FAQ.html#origins)阅读有关Goal起源和影响的内容。如果您已了解K语言，可能需要阅读[与K语言的区别](chap-from-k.html)章节。

Goal在数组语言中的主要特点可以总结如下：

+   字典和表功能（类似大多数K方言）

+   原子字符串："ab" "bc"="bc" → 0 1

+   Unicode感知的字符串基元：`_"AB" "Π" → "ab" "π"`

+   类似Perl的引用构造和字符串内插：`qq/$var\n or ${var}/`和`rq#raw strings#`

+   格式化字符串："%.2g"$1 4%3 → "0.33" "1.3"

+   专用正则表达式语法：`rx/\s+/`

+   错误值

+   可嵌入和可扩展的Go

Goal还支持标准功能，如I/O、CSV和时间处理。

Goal在处理列数据或文本处理等常见脚本任务中表现最为突出。Goal也适用于探索性编程。

[下一章](chap-tutorial.html)将介绍Goal的特性，并展示语言在几个实际示例中的应用。
