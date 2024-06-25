<!--yml

分类：未分类

日期：2024 年 05 月 27 日 14:59:37

-->

# 一个用于 C 和 C++的垃圾收集器。

> 来源：[`hboehm.info/gc/`](https://hboehm.info/gc/)

# 一个用于 C 和 C++的垃圾收集器。

[ 这是该页面更新后的版本，前身分别是位于`http://www.hpl.hp.com/personal/Hans_Boehm/gc`、`http://reality.sgi.com/boehm/gc.html`以及`ftp://parcftp.xerox.com/pub/gc/gc.html`。]

[Boehm](http://www.hboehm.info)-[Demers](http://www.cs.cornell.edu/annual_report/00-01/bios.htm#demers)-[Weiser](http://www-sul.stanford.edu/weiser/) 保守垃圾收集器可以用作 C 的`malloc`或 C++的`new`的垃圾收集替代。它允许您基本上像通常一样分配内存，而无需显式释放不再有用的内存。当收集器确定无法再访问内存时，它会自动回收内存。有一个这样的用法的简单示例在这里。

该收集器还被许多编程语言实现所使用，这些语言要么使用 C 作为中间代码，要么希望更轻松地与 C 库进行交互，要么只是更喜欢简单的收集器界面。有关接口的更详细描述，请参见这里。

或者，垃圾收集器可以用作 C 或 C++程序的泄漏检测器，尽管这不是其主要目标。

简要讨论了在 C 和 C++中支持保守垃圾收集的利弊，见 issues.html。这里有一个常见问题列表的开头。

根据经验，通过仅将`malloc`替换为`GC_malloc`调用，将`realloc`替换为`GC_realloc`调用，并删除 free 调用，该收集器可以与大多数未经修改的 C 程序一起使用。有关例外情况，请参阅 issues.html。

许多版本可在`gc_source`子目录中找到。当前一个最新的稳定版本是`gc-8.2.4.tar.gz`。请注意，这使用了新的版本编号方案，可能偶尔需要单独下载`libatomic_ops`（请参见下文）。

如果失败，请尝试`gc_source`中的最新明确编号版本。后续版本可能包含其他功能、平台支持或错误修复，但可能测试不够充分。

版本 7.3 及更高版本要求您下载相应版本的 libatomic_ops，可在[`https://github.com/ivmai/libatomic_ops/wiki/Download`](https://github.com/ivmai/libatomic_ops/wiki/Download)找到。当前（稳定）版本也可作为`gc_source/libatomic_ops-7.8.0.tar.gz`获得。您需要将其放置在`libatomic_ops`子目录中。

以前最好使用相应版本的 gc 和 libatomic_ops，但对于最近的版本，混合使用它们应该没问题。

从 8.0 版本开始，只有当编译器不理解 gcc 的 C 原子内在函数时才需要 libatomic_ops。

GC 源代码的开发版本现在存放在 github 上，以及可下载的软件包。GC 树本身位于 [`https://github.com/ivmai/bdwgc/`](https://github.com/ivmai/bdwgc/)。GC 所需的 `libatomic_ops` 树位于 [`https://github.com/ivmai/libatomic_ops/`](https://github.com/ivmai/libatomic_ops/)。

要构建垃圾回收器的工作版本，您需要执行类似以下的操作，其中 `D` 是安装目录的绝对路径：

```
cd D
git clone https://github.com/ivmai/libatomic_ops
git clone https://github.com/ivmai/bdwgc
ln -s  D/libatomic_ops D/bdwgc/libatomic_ops
cd bdwgc
autoreconf -vif
automake --add-missing
./configure
make

```

这将需要你已经安装了 C 和 C++ 的工具链，`git`，`automake`，`autoconf` 和 `libtool`。

历史版本的源代码仍然可以在 SourceForge 网站上找到（项目“bdwgc”）[这里](http://bdwgc.cvs.sourceforge.net/bdwgc/)。

垃圾回收器代码版权归[Hans-J. Boehm](http://hboehm.info)，Alan J. Demers，[Xerox Corporation](http://www.xerox.com)，[Silicon Graphics](http://www.sgi.com)和[Hewlett-Packard Company](http://www.hp.com)所有。它可以在最小限制下免费使用和复制，无需支付费用。请参阅分发中的 README 文件或许可证以获取更多详细信息。**它按原样提供，绝对没有明示或暗示的任何保证。任何使用都是自担风险**。

回收器并不是完全可移植的，但分发版包括对大多数标准 PC 和 UNIX/Linux 平台的端口。回收器应该可以在 Linux，*BSD，最新版本的 Windows，MacOS X，HP/UX，Solaris，Tru64，Irix 和其他一些操作系统上工作。某些端口比其他端口更加完善。有说明可以将回收器移植到新平台上。

Irix pthreads，Linux threads，Win32 threads，Solaris threads（旧版和 pthreads），HP/UX 11 pthreads，Tru64 pthreads 和 MacOS X threads 在最新版本中受支持。

### 单独分发的端口

对于 MacOS 9/Classic 版本的使用，Patrick Beard 的最新版本曾经可以从`http://homepage.mac.com/pcbeard/gc/`获取。

几个 Linux 和 BSD 版本提供了垃圾回收器的预打包版本。Debian 版本可在[`packages.debian.org/sid/libgc-dev`](https://packages.debian.org/sid/libgc-dev)找到。

田浦健次郎、遠藤利郎和米澤彰則基于此开发了一个并行收集器。曾经可以从`http://www.yl.is.s.u-tokyo.ac.jp/gc/`获取。他们的收集器在收集过程中利用多个处理器。从收集器版本 6.0alpha1 开始，我们也采取了这种方法，尽管目标是更为温和的处理器可伸缩性。我们的方法在`scale.html`中简要讨论。该收集器使用标记-清除算法。在提供正确类型的虚拟内存支持的操作系统下，它提供增量和分代收集。 （目前包括 SunOS[45]，IRIX，OSF/1，Linux 和 Windows，具体取决于不同的限制。）它允许在对象被收集时调用*finalization*代码。如果提供了此类信息，它可以利用类型信息来定位指针，但通常在没有此类信息的情况下使用。有关更多详细信息，请参阅分发中的 README 和`gc.h`文件。

要了解实现概述，请参见这里。

垃圾收集器分发包含一个 C 字符串（*cord*）包，该包提供了对长字符串的快速连接和子字符串操作。包含一个简单的基于 curses 和 win32 的编辑器，该编辑器将整个文件表示为一个 cord，并作为示例应用程序包含在内。

非增量收集器的性能通常与 malloc/free 实现竞争力相当。对于为 malloc/free 编写的程序，空间和时间开销可能略高。 （请参阅 Detlefs，Dosser 和 Zorn 的大型 C 和 C++程序的内存分配成本。）对于主要分配非常小对象的程序，收集器可能更快；对于主要分配大对象的程序，它将更慢。如果在多线程环境中使用收集器，并配置为线程本地分配，则在某些情况下，它的性能可能显著优于 malloc/free 分配。

我们还期望在许多情况下，通过为垃圾收集编写和调整程序，任何额外的开销都将被减少的复制等所补偿。

**针对此收集器的常见问题列表的开端在此处**。

**以下提供了垃圾收集的一般信息**：

Paul Wilson 的垃圾回收 ftp 存档和 GC 调查。

Ravenbrook 的[内存管理参考](http://www.memorymanagement.org/)。

David Chase 的[GC 常见问题解答](http://www.iecc.com/gclist/GC-faq.html)。

Richard Jones 的[GC 页面](http://www.ukc.ac.uk/computer_science/Html/Jones/gc.html)和他的书。

**以下论文描述了我们使用的收集器算法和更高层次的基本设计决策。**

(一些较低级别的细节可以在 gcdescr.html 中找到。)

第一个由于版权问题无法电子获取。其他大部分都受到 ACM 版权保护。

Boehm, H., "动态内存分配和垃圾回收", *《物理学中的计算机》9 期*, 1995 年 5 月/6 月, 第 297-303 页。这是针对一个对内存分配问题不熟悉的复杂受众的。算法细节与实现中的不同。下一期有相关的编辑信和一个小的修正。

Boehm, H., 和 [M. Weiser](http://www.ubiq.com/hypertext/weiser/weiser.html), "不合作环境中的垃圾回收", *《软件实践与经验》*, 1988 年 9 月, 第 807-820 页。

Boehm, H., A. Demers, 和 S. Shenker, "大部分并行垃圾收集", 《ACM SIGPLAN '91 程序设计语言设计与实现会议论文集》，*SIGPLAN 通知 26 期*, 6 (1991 年 6 月), 第 157-164 页。

Boehm, H., "空间高效的保守式垃圾收集", 《ACM SIGPLAN '93 程序设计语言设计与实现会议论文集》，*SIGPLAN 通知 28 期*, 6 (1993 年 6 月), 第 197-206 页。

Boehm, H., "减少垃圾收集器缓存未命中", *2000 年国际内存管理研讨会论文集*。[官方版本。](http://portal.acm.org/citation.cfm?doid=362422.362438) [技术报告版本。](http://www.hpl.hp.com/techreports/2000/HPL-2000-99.html) 描述了一些平台上收集器中所包含的预取策略。解释了为什么“标记-清除”收集器的清扫阶段实际上不应该是一个单独的阶段。

M. Serrano, H. Boehm, "理解 Scheme 程序的内存分配", *第五届 ACM SIGPLAN 国际函数式编程会议论文集*, 2000 年, 加拿大蒙特利尔, 第 245-256 页。[官方版本。](http://www.acm.org/pubs/citations/proceedings/fp/351240/p245-serrano/) [较早的技术报告版本。](http://www.hpl.hp.com/techreports/2000/HPL-2000-62.html) 包括一些关于识别内存保留原因的收集器调试设施的讨论。

Boehm, H., "快速多处理器内存分配和垃圾收集", [惠普实验室技术报告 HPL 2000-165](http://www.hpl.hp.com/techreports/2000/HPL-2000-165.html)。讨论了并行收集算法，并呈现了一些性能结果。

Boehm, H., "保守式垃圾收集器空间使用的限制", *2002 年 ACM SIGPLAN-SIGACT 编程语言原理研讨会论文集*, 2002 年 1 月, 第 93-100 页。[官方版本。](http://portal.acm.org/citation.cfm?doid=503272.503282) [技术报告版本。](http://www.hpl.hp.com/techreports/2001/HPL-2001-251.html) 包括一个关于更可靠地测试潜在的无限堆增长可能性的收集器设施的讨论。

**以下论文讨论了确保保守垃圾收集器安全所需的语言和编译器限制。**

我们感谢 John Levine 和 JCLT 允许我们提供第二篇论文的电子版，并为最终版本提供 PostScript。

Boehm, H., ``简单的垃圾收集器安全性''，ACM SIGPLAN '96 会议论文集上的文章，讨论了编程语言设计和实现。

Boehm, H., 和 D. Chase, ``用于垃圾收集器安全的 C 编译提案''，*C 语言翻译杂志 4*，2（1992 年 12 月），第 126-141 页。

**其他相关信息：**

Detlefs、Dosser 和 Zorn 的大型 C 和 C++程序中的内存分配成本。这是对 Boehm-Demers-Weiser 收集器与 malloc/free 的性能比较，使用了为 malloc/free 编写的程序。

Joel Bartlett 的主要是用于 C++的复制保守型垃圾收集器。

John Ellis 和 David Detlef 的用于 C++的安全高效垃圾收集提案。

Henry Baker 的[论文集](http://home.pipeline.com/~hbaker1/)。

Hans Boehm 的 Allocation and GC Myths 演讲幻灯片。

**这一部分最近没有更新。**

目前已知使用某个变体的收集器的用户包括：

[GCJ](http://gcc.gnu.org/java)的运行时系统，即静态 GNU Java 编译器。

[W3m](http://w3m.sourceforge.net/)，一个基于文本的网页浏览器。

一些版本的施乐 DocuPrint 打印机软件。

[Mozilla](http://www.mozilla.org)项目，作为泄漏检测器。

[Mono](http://www.go-mono.com)项目，一个.NET 开发框架的开源实现。

[DotGNU Portable.NET 项目](http://www.gnu.org/projects/dotgnu/)，另一个开源的.NET 实现。

[Irssi IRC 客户端](http://irssi.org/)。

[伯克利钛项目](http://titanium.cs.berkeley.edu/)。

[NAGWare f90 Fortran 90 编译器](http://www.nag.co.uk/nagware_fortran_compilers.asp)。

Elwood Corporation 的[Eclipse](http://www.elwood.com/eclipse-info/index.htm) Common Lisp 系统、C 库和翻译器。

由 Manuel Serrano 和其他人编写的[Bigloo Scheme](http://www-sop.inria.fr/mimosa/fp/Bigloo/)和[Camloo ML 编译器](http://kaolin.unice.fr/~serrano/camloo.html)。

Brent Benson 的[libscheme](http://ftp.cs.indiana.edu/pub/scheme-repository/imp/)。

[MzScheme](http://www.cs.rice.edu/CS/PLT/packages/mzscheme/index.html)方案实现。

[华盛顿大学 Cecil 实现](http://www.cs.washington.edu/research/projects/cecil/www/cecil-home.html)。

[伯克利 Sather 实现](http://www.icsi.berkeley.edu:80/Sather/)。

[伯克利 Harmonia 项目](http://www.cs.berkeley.edu/~harmonia/)。

[Toba](http://www.cs.arizona.edu/sumatra/toba/) Java 虚拟机到 C 语言的翻译器。

[Gwydion Dylan 编译器](http://www.gwydiondylan.org/)。

[GNU Objective C 运行时](http://gcc.gnu.org/onlinedocs/gcc/Objective-C.html)。

[Macaulay 2](http://www.math.uiuc.edu/Macaulay2)，一个支持代数几何和交换代数研究的系统。

[Vesta](http://www.vestasys.org/)配置管理系统。

[Visual Prolog 6](http://www.visual-prolog.com/vip6)。

[Asymptote LaTeX 兼容的矢量图形语言](http://asymptote.sf.net)。

构建和使用收集器的简单示例。

对垃圾收集器的替代接口的描述。

ISMM 2004 教程关于 GC 的幻灯片。

FAQ（常见问题）列表。

如何将垃圾收集器用作泄漏检测器。

关于调试垃圾收集应用程序的一些提示。

垃圾收集器实现概述。

将收集器移植到新平台的说明。

用于快速指针查找的数据结构。

收集器扩展到多处理器的可扩展性。

包含垃圾收集器源代码的目录。

尝试建立保守垃圾收集器空间使用量上限的一项尝试。

标记-清除与复制式垃圾收集器及其复杂性的对比。

与其他收集器相比，保守型垃圾收集器的优缺点。

关于 C/C++中垃圾收集与手动内存管理的问题。

在某些情况下，由于减少同步，垃圾收集导致实现速度大大提升的案例。

讨论非移动式垃圾收集器性能的幻灯片集。

讨论*Destructors, Finalizers, and Synchronization*（POPL 2003）的[幻灯片集](http://hboehm.info/popl03/web)。

对应上述幻灯片集的[论文](http://portal.acm.org/citation.cfm?doid=604131.604153)（ [技术报告版本](http://www.hpl.hp.com/techreports/2002/HPL-2002-335.html)）。

一个 Java/Scheme/C/C++垃圾收集基准测试（benchmark）。

关于内存分配神话的演讲幻灯片。

OOPSLA 98 垃圾收集演讲的幻灯片集。

相关论文。

有关收集器的最新更新和问题报告信息，请参阅[bdwgc github 页面](https://github.com/ivmai/bdwgc#feedback-contribution-questions-and-notifications)。

曾经有一对用于 GC 讨论的邮件列表。这些现已不再活跃。请参阅上述页面获取存档的指针。

一些关于收集器的讨论曾在 gcc java 邮件列表上进行，其存档在[这里](http://gcc.gnu.org/ml/java/)，还在[gclist@iecc.com](http://lists.tunes.org/mailman/listinfo/gclist)。

评论和错误报告也可以发送至(boehm@acm.org)，但强烈建议在[github](https://github.com/ivmai/bdwgc/issues)上报告问题。
