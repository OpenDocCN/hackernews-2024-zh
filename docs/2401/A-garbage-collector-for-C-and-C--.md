<!--yml

分类：未分类

日期：2024年05月27日14:59:37

-->

# 一个用于C和C++的垃圾收集器。

> 来源：[https://hboehm.info/gc/](https://hboehm.info/gc/)

# 一个用于C和C++的垃圾收集器。

[ 这是该页面更新后的版本，前身分别是位于`http://www.hpl.hp.com/personal/Hans_Boehm/gc`、`http://reality.sgi.com/boehm/gc.html`以及`ftp://parcftp.xerox.com/pub/gc/gc.html`。]

[Boehm](http://www.hboehm.info)-[Demers](http://www.cs.cornell.edu/annual_report/00-01/bios.htm#demers)-[Weiser](http://www-sul.stanford.edu/weiser/) 保守垃圾收集器可以用作C的`malloc`或C++的`new`的垃圾收集替代。它允许您基本上像通常一样分配内存，而无需显式释放不再有用的内存。当收集器确定无法再访问内存时，它会自动回收内存。有一个这样的用法的简单示例在[这里](simple_example.html)。

该收集器还被许多编程语言实现所使用，这些语言要么使用C作为中间代码，要么希望更轻松地与C库进行交互，要么只是更喜欢简单的收集器界面。有关接口的更详细描述，请参见[这里](gcinterface.html)。

或者，垃圾收集器可以用作C或C++程序的[泄漏检测器](leak.html)，尽管这不是其主要目标。

简要讨论了在C和C++中支持保守垃圾收集的利弊，见[issues.html](issues.html)。[这里](faq.html)有一个常见问题列表的开头。

根据经验，通过仅将`malloc`替换为`GC_malloc`调用，将`realloc`替换为`GC_realloc`调用，并删除free调用，该收集器可以与大多数未经修改的C程序一起使用。有关例外情况，请参阅[issues.html](issues.html)。

许多版本可在[`gc_source`](gc_source)子目录中找到。当前一个最新的稳定版本是[`gc-8.2.4.tar.gz`](gc_source/gc-8.2.4.tar.gz)。请注意，这使用了新的版本编号方案，可能偶尔需要单独下载`libatomic_ops`（请参见下文）。

如果失败，请尝试[`gc_source`](gc_source)中的最新明确编号版本。后续版本可能包含其他功能、平台支持或错误修复，但可能测试不够充分。

版本7.3及更高版本要求您下载相应版本的libatomic_ops，可在[`https://github.com/ivmai/libatomic_ops/wiki/Download`](https://github.com/ivmai/libatomic_ops/wiki/Download)找到。当前（稳定）版本也可作为[`gc_source/libatomic_ops-7.8.0.tar.gz`](gc_source/libatomic_ops-7.8.0.tar.gz)获得。您需要将其放置在`libatomic_ops`子目录中。

以前最好使用相应版本的gc和libatomic_ops，但对于最近的版本，混合使用它们应该没问题。

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

垃圾回收器代码版权归[Hans-J. Boehm](http://hboehm.info)，Alan J. Demers，[Xerox Corporation](http://www.xerox.com)，[Silicon Graphics](http://www.sgi.com)和[Hewlett-Packard Company](http://www.hp.com)所有。它可以在最小限制下免费使用和复制，无需支付费用。请参阅分发中的 README 文件或[许可证](license.txt)以获取更多详细信息。**它按原样提供，绝对没有明示或暗示的任何保证。任何使用都是自担风险**。

回收器并不是完全可移植的，但分发版包括对大多数标准 PC 和 UNIX/Linux 平台的端口。回收器应该可以在 Linux，*BSD，最新版本的 Windows，MacOS X，HP/UX，Solaris，Tru64，Irix 和其他一些操作系统上工作。某些端口比其他端口更加完善。有[说明](porting.html)可以将回收器移植到新平台上。

Irix pthreads，Linux threads，Win32 threads，Solaris threads（旧版和pthreads），HP/UX 11 pthreads，Tru64 pthreads 和 MacOS X threads 在最新版本中受支持。

### 单独分发的端口

对于 MacOS 9/Classic 版本的使用，Patrick Beard 的最新版本曾经可以从`http://homepage.mac.com/pcbeard/gc/`获取。

几个 Linux 和 BSD 版本提供了垃圾回收器的预打包版本。Debian 版本可在[https://packages.debian.org/sid/libgc-dev](https://packages.debian.org/sid/libgc-dev)找到。

田浦健次郎、遠藤利郎和米澤彰則基于此开发了一个并行收集器。曾经可以从`http://www.yl.is.s.u-tokyo.ac.jp/gc/`获取。他们的收集器在收集过程中利用多个处理器。从收集器版本6.0alpha1开始，我们也采取了这种方法，尽管目标是更为温和的处理器可伸缩性。我们的方法在[`scale.html`](scale.html)中简要讨论。该收集器使用[标记-清除](complexity.html)算法。在提供正确类型的虚拟内存支持的操作系统下，它提供增量和分代收集。 （目前包括SunOS[45]，IRIX，OSF/1，Linux和Windows，具体取决于不同的限制。）它允许在对象被收集时调用[*finalization*](finalization.html)代码。如果提供了此类信息，它可以利用类型信息来定位指针，但通常在没有此类信息的情况下使用。有关更多详细信息，请参阅分发中的README和`gc.h`文件。

要了解实现概述，请参见[这里](gcdescr.html)。

垃圾收集器分发包含一个C字符串（[*cord*](gc_source/cordh.txt)）包，该包提供了对长字符串的快速连接和子字符串操作。包含一个简单的基于curses和win32的编辑器，该编辑器将整个文件表示为一个cord，并作为示例应用程序包含在内。

非增量收集器的性能通常与malloc/free实现竞争力相当。对于为malloc/free编写的程序，空间和时间开销可能略高。 （请参阅Detlefs，Dosser和Zorn的[大型C和C++程序的内存分配成本](ftp://ftp.cs.colorado.edu/pub/techreports/zorn/CU-CS-665-93.ps.Z)。）对于主要分配非常小对象的程序，收集器可能更快；对于主要分配大对象的程序，它将更慢。如果在多线程环境中使用收集器，并配置为线程本地分配，则在某些情况下，它的性能可能显著优于malloc/free分配。

我们还期望在许多情况下，通过为垃圾收集编写和调整程序，任何额外的开销都将被减少的复制等所补偿。

**针对此收集器的常见问题列表的开端在[此处](faq.html)**。

**以下提供了垃圾收集的一般信息**：

Paul Wilson的[垃圾回收ftp存档](ftp://ftp.cs.utexas.edu/pub/garbage)和[GC调查](ftp://ftp.cs.utexas.edu/pub/garbage/gcsurvey.ps)。

Ravenbrook的[内存管理参考](http://www.memorymanagement.org/)。

David Chase的[GC常见问题解答](http://www.iecc.com/gclist/GC-faq.html)。

Richard Jones的[GC页面](http://www.ukc.ac.uk/computer_science/Html/Jones/gc.html)和[他的书](//www.cs.kent.ac.uk/people/staff/rej/gcbook/gcbook.html)。

**以下论文描述了我们使用的收集器算法和更高层次的基本设计决策。**

(一些较低级别的细节可以在[gcdescr.html](gcdescr.html)中找到。)

第一个由于版权问题无法电子获取。其他大部分都受到ACM版权保护。

Boehm, H., "动态内存分配和垃圾回收", *《物理学中的计算机》9期*, 1995年5月/6月, 第297-303页。这是针对一个对内存分配问题不熟悉的复杂受众的。算法细节与实现中的不同。下一期有相关的编辑信和一个小的修正。

Boehm, H., 和 [M. Weiser](http://www.ubiq.com/hypertext/weiser/weiser.html), ["不合作环境中的垃圾回收"](../spe_gc_paper), *《软件实践与经验》*, 1988年9月, 第807-820页。

Boehm, H., A. Demers, 和 S. Shenker, ["大部分并行垃圾收集"](papers/pldi91.ps.Z), 《ACM SIGPLAN '91程序设计语言设计与实现会议论文集》，*SIGPLAN通知 26期*, 6 (1991年6月), 第157-164页。

Boehm, H., ["空间高效的保守式垃圾收集"](papers/pldi93.ps.Z), 《ACM SIGPLAN '93程序设计语言设计与实现会议论文集》，*SIGPLAN通知 28期*, 6 (1993年6月), 第197-206页。

Boehm, H., "减少垃圾收集器缓存未命中", *2000年国际内存管理研讨会论文集*。[官方版本。](http://portal.acm.org/citation.cfm?doid=362422.362438) [技术报告版本。](http://www.hpl.hp.com/techreports/2000/HPL-2000-99.html) 描述了一些平台上收集器中所包含的预取策略。解释了为什么“标记-清除”收集器的清扫阶段实际上不应该是一个单独的阶段。

M. Serrano, H. Boehm, "理解Scheme程序的内存分配", *第五届ACM SIGPLAN国际函数式编程会议论文集*, 2000年, 加拿大蒙特利尔, 第245-256页。[官方版本。](http://www.acm.org/pubs/citations/proceedings/fp/351240/p245-serrano/) [较早的技术报告版本。](http://www.hpl.hp.com/techreports/2000/HPL-2000-62.html) 包括一些关于识别内存保留原因的收集器调试设施的讨论。

Boehm, H., "快速多处理器内存分配和垃圾收集", [惠普实验室技术报告 HPL 2000-165](http://www.hpl.hp.com/techreports/2000/HPL-2000-165.html)。讨论了并行收集算法，并呈现了一些性能结果。

Boehm, H., "保守式垃圾收集器空间使用的限制", *2002年ACM SIGPLAN-SIGACT编程语言原理研讨会论文集*, 2002年1月, 第93-100页。[官方版本。](http://portal.acm.org/citation.cfm?doid=503272.503282) [技术报告版本。](http://www.hpl.hp.com/techreports/2001/HPL-2001-251.html) 包括一个关于更可靠地测试潜在的无限堆增长可能性的收集器设施的讨论。

**以下论文讨论了确保保守垃圾收集器安全所需的语言和编译器限制。**

我们感谢John Levine和JCLT允许我们提供第二篇论文的电子版，并为最终版本提供PostScript。

Boehm, H., [``简单的垃圾收集器安全性''](papers/pldi96.ps.gz)，ACM SIGPLAN '96会议论文集上的文章，讨论了编程语言设计和实现。

Boehm, H., 和D. Chase, [``用于垃圾收集器安全的C编译提案''](papers/boecha.ps.gz)，*C语言翻译杂志4*，2（1992年12月），第126-141页。

**其他相关信息：**

Detlefs、Dosser和Zorn的[大型C和C++程序中的内存分配成本](ftp://ftp.cs.colorado.edu/pub/techreports/zorn/CU-CS-665-93.ps.Z)。这是对Boehm-Demers-Weiser收集器与malloc/free的性能比较，使用了为malloc/free编写的程序。

Joel Bartlett的[主要是用于C++的复制保守型垃圾收集器](ftp://ftp.digital.com/pub/DEC/CCgc)。

John Ellis和David Detlef的[用于C++的安全高效垃圾收集](ftp://parcftp.xerox.com/pub/ellis/gc/gc.ps)提案。

Henry Baker的[论文集](http://home.pipeline.com/~hbaker1/)。

Hans Boehm的[Allocation and GC Myths](myths.ps)演讲幻灯片。

**这一部分最近没有更新。**

目前已知使用某个变体的收集器的用户包括：

[GCJ](http://gcc.gnu.org/java)的运行时系统，即静态GNU Java编译器。

[W3m](http://w3m.sourceforge.net/)，一个基于文本的网页浏览器。

一些版本的施乐DocuPrint打印机软件。

[Mozilla](http://www.mozilla.org)项目，作为泄漏检测器。

[Mono](http://www.go-mono.com)项目，一个.NET开发框架的开源实现。

[DotGNU Portable.NET项目](http://www.gnu.org/projects/dotgnu/)，另一个开源的.NET实现。

[Irssi IRC客户端](http://irssi.org/)。

[伯克利钛项目](http://titanium.cs.berkeley.edu/)。

[NAGWare f90 Fortran 90编译器](http://www.nag.co.uk/nagware_fortran_compilers.asp)。

Elwood Corporation的[Eclipse](http://www.elwood.com/eclipse-info/index.htm) Common Lisp系统、C库和翻译器。

由Manuel Serrano和其他人编写的[Bigloo Scheme](http://www-sop.inria.fr/mimosa/fp/Bigloo/)和[Camloo ML编译器](http://kaolin.unice.fr/~serrano/camloo.html)。

Brent Benson的[libscheme](http://ftp.cs.indiana.edu/pub/scheme-repository/imp/)。

[MzScheme](http://www.cs.rice.edu/CS/PLT/packages/mzscheme/index.html)方案实现。

[华盛顿大学Cecil实现](http://www.cs.washington.edu/research/projects/cecil/www/cecil-home.html)。

[伯克利Sather实现](http://www.icsi.berkeley.edu:80/Sather/)。

[伯克利Harmonia项目](http://www.cs.berkeley.edu/~harmonia/)。

[Toba](http://www.cs.arizona.edu/sumatra/toba/) Java虚拟机到C语言的翻译器。

[Gwydion Dylan 编译器](http://www.gwydiondylan.org/)。

[GNU Objective C 运行时](http://gcc.gnu.org/onlinedocs/gcc/Objective-C.html)。

[Macaulay 2](http://www.math.uiuc.edu/Macaulay2)，一个支持代数几何和交换代数研究的系统。

[Vesta](http://www.vestasys.org/)配置管理系统。

[Visual Prolog 6](http://www.visual-prolog.com/vip6)。

[Asymptote LaTeX 兼容的矢量图形语言](http://asymptote.sf.net)。

[构建和使用收集器的简单示例](simple_example.html)。

对[垃圾收集器的替代接口](gcinterface.html)的描述。

[ISMM 2004教程关于GC的幻灯片](04tutorial.pdf)。

[FAQ（常见问题）列表](faq.html)。

如何将[垃圾收集器用作泄漏检测器](leak.html)。

关于调试垃圾收集应用程序的[一些提示](debugging.html)。

[垃圾收集器实现概述](gcdescr.html)。

将收集器[移植到新平台的说明](porting.html)。

用于快速指针查找的[数据结构](tree.html)。

收集器[扩展到多处理器的可扩展性](scale.html)。

包含[垃圾收集器源代码](gc_source)的目录。

尝试建立保守垃圾收集器空间使用量上限的[一项尝试](bounds.html)。

[标记-清除与复制式垃圾收集器及其复杂性](complexity.html)的对比。

[与其他收集器相比，保守型垃圾收集器的优缺点](conservative.html)。

关于C/C++中[垃圾收集与手动内存管理的问题](issues.html)。

在某些情况下，由于减少同步，[垃圾收集导致实现速度大大提升的案例](example.html)。

讨论[非移动式垃圾收集器性能](nonmoving)的幻灯片集。

讨论*Destructors, Finalizers, and Synchronization*（POPL 2003）的[幻灯片集](http://hboehm.info/popl03/web)。

对应上述幻灯片集的[论文](http://portal.acm.org/citation.cfm?doid=604131.604153)（ [技术报告版本](http://www.hpl.hp.com/techreports/2002/HPL-2002-335.html)）。

一个Java/Scheme/C/C++垃圾收集基准测试[（benchmark）](gc_bench.html)。

[关于内存分配神话的演讲幻灯片](myths.ps)。

[OOPSLA 98 垃圾收集演讲的幻灯片集](gctalk.ps)。

[相关论文](papers)。

有关收集器的最新更新和问题报告信息，请参阅[bdwgc github 页面](https://github.com/ivmai/bdwgc#feedback-contribution-questions-and-notifications)。

曾经有一对用于GC讨论的邮件列表。这些现已不再活跃。请参阅上述页面获取存档的指针。

一些关于收集器的讨论曾在gcc java邮件列表上进行，其存档在[这里](http://gcc.gnu.org/ml/java/)，还在[gclist@iecc.com](http://lists.tunes.org/mailman/listinfo/gclist)。

评论和错误报告也可以发送至([boehm@acm.org](mailto:boehm@acm.org))，但强烈建议在[github](https://github.com/ivmai/bdwgc/issues)上报告问题。
