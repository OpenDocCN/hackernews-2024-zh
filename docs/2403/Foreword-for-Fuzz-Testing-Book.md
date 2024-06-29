<!--yml

类别：未分类

日期：2024-05-29 12:46:23

-->

# 模糊测试书前言

> 来源：[https://pages.cs.wisc.edu/~bart/fuzz/Foreword1.html](https://pages.cs.wisc.edu/~bart/fuzz/Foreword1.html)

## 模糊测试书前言

* * *

下面是我为一本关于模糊测试的书前言写的散文草稿：

* * *

那是一个黑夜暴风雨的夜晚。真的。

1988年秋天，我坐在麦迪逊的公寓里，密西西比地区有一场狂野的雷暴，雨水倾盆，深夜的天空被闪电照亮。那天晚上，我通过拨号电话线和一个1200波特的调制解调器登录到办公室的Unix系统上。由于雨势很大，电话线上传来噪音，干扰了我输入合理命令给终端和正在运行的程序的能力。在噪音压倒命令之前，我必须争分夺秒地输入命令行。

与嘈杂的电话线斗争并不奇怪。毕竟，在纠错调制解调器问世之前，这种情况并不罕见。让我惊讶的是，噪音似乎导致程序崩溃。更让我吃惊的是那些崩溃的程序 -- 我们每天都在使用的常见Unix实用程序。

我内心的科学家说，我们需要进行系统性调查，试图理解问题的程度和原因。

那个学期，我在威斯康星大学教授研究生高级操作系统课程。每学期，我们会提供一份建议主题列表供学生选择他们的课程项目。我把这个测试项目添加到了列表中；项目描述的文本出现在下面的突出框中。

在撰写项目描述的过程中，我需要给这种测试起个名字。我想要一个能唤起随机、非结构化数据感觉的名字。经过几次尝试后，我选定了术语***模糊测试***。

那学期，有三组尝试了模糊测试项目，其中两组未能取得任何崩溃结果。Lars Fredriksen和Bryan So组成了第三组，他们是更有才华的程序员，也是更谨慎的实验者；他们的成功远远超出了我的预期。正如我们在第一篇模糊测试论文中报道的那样[[1990](http://www.cs.wisc.edu/paradyn/papers/fuzz.pdf)]，他们在测试的七个Unix变体上能够使25-33%的实用程序崩溃或挂起。

然而，模糊测试项目不仅仅是找到程序故障的快速方法。找到每个故障的原因并分类这些故障赋予了结果更深刻的意义和更持久的影响。工具和脚本的源代码、原始测试结果以及建议的错误修复都是公开的。信任和可重复性是这项工作的关键基本原则。

在接下来的几年里，我们在更多和更多的 Unix 系统上重复了这些测试，针对更大的一组命令行实用程序进行了测试，并扩展了我们的测试到基于当时新的 X 窗口系统的 GUI 程序 [[1995](http://www.cs.wisc.edu/paradyn/papers/fuzz-revisited.pdf)]。几年后，Windows 也加入进来 [[2000](http://www.cs.wisc.edu/paradyn/papers/fuzz-nt.pdf)]，最近的是 MacOS [[2006](http://www.cs.wisc.edu/paradyn/papers/Fuzz-MacOS.pdf)]。在每种情况下，多年来，我们发现了许多漏洞，并且在每种情况下，我们诊断了这些漏洞并发布了我们所有的结果。

在我们最近的研究中，随着我们扩展到更多基于 GUI 的应用程序测试，我们发现了经典的 1983 年测试工具，“The Monkey”，它曾在早期的 Macintosh 计算机上使用 [引用 Hertzfeld 的书]。显然，这是一个超前于他们时代的团体。

在撰写我们早期的模糊测试论文的过程中，我们遇到了来自测试和软件工程社区的强烈抵制。缺乏正式的模型和方法论，以及测试的不纪律化方法常常会冒犯到这个领域有经验的从业者。事实上，我仍然经常遇到对这种“石斧与熊皮”（对不起，斯波克先生）测试方法持敌对态度的人。

我的回答总是很简单的：“我们只是在试图找出漏洞”。正如我多次所说的，模糊测试并不意味着取代更系统化的测试。它只是测试人员工具包中的又一个工具，尽管是一个极其容易使用的工具。

顺便提一下，需要注意的是，对我来说，模糊测试从未是一个资助的研究项目；这是一种研究爱好而不是职业。所有的辛勤工作都是由一系列有才华和积极性的研究生在我的高级操作系统课程的课程项目中完成的。这就是我们的乐趣所在。

Fuzz 测试已经发展成为研究和工程的一个重要子领域，新的研究结果使它远远超出了我们最初简单的工作。由于可靠性是安全的基础，所以它已经成为软件安全评估中至关重要的工具。因此，这本书的主题既及时又极其重要。每一个希望编写安全可靠软件的从业者都需要将这些技术加入到他们的工具包中。

Barton Miller

威斯康星州麦迪逊

2008 年 4 月

|

&#124;

```

**Operating System Utility Program Reliability - The Fuzz Generator**

The goal of this project is to evaluate the robustness of various UNIX utility
programs, given an unpredictable input stream.  This project has two parts.
First, you will build a "fuzz" generator.  This is a program that will output
a random character stream.  Second, you will take the fuzz generator and use
it to attack as many UNIX utilities as possible, with the goal of trying to
break them.  For the utilities that break, you will try to determine what type
of input cause the break.

THE PROGRAM

The fuzz generator will generate an output stream of random characters.
It will need several options to give you flexibility to test different
programs.  Below is the start for a list of options for features that
fuzz will support.  It is important when writing this program to use
good C and UNIX style, and good structure, as we hope to distribute
this program to others.

    -p          only the printable ASCII characters

    -a          all ASCII characters

    -0          include the null (0 byte) character

    -l          generate random length lines (\\n terminated strings)

    -f name     record characters in file ``name''

    -d nnn      delay nnn seconds following each character

    -r name     replay characters in file ``name'' to output

THE TESTING

The fuzz program should be used to test various UNIX utilities.  These
utilities include programs like vi, mail, cc, make, sed, awk, sort,
etc.  The goal is to first see if the program will break and second to
understand what type of input is responsible for the break.

```

&#124;

|

* * *
