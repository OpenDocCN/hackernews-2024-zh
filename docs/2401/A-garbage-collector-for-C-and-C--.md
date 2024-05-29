<!--yml
category: 未分类
date: 2024-05-27 14:59:37
-->

# A garbage collector for C and C++

> 来源：[https://hboehm.info/gc/](https://hboehm.info/gc/)

# A garbage collector for C and C++

[ This is an updated version of the page formerly at `http://www.hpl.hp.com/personal/Hans_Boehm/gc`, and before that at `http://reality.sgi.com/boehm/gc.html` and before that at `ftp://parcftp.xerox.com/pub/gc/gc.html`. ]

The [Boehm](http://www.hboehm.info)-[Demers](http://www.cs.cornell.edu/annual_report/00-01/bios.htm#demers)-[Weiser](http://www-sul.stanford.edu/weiser/) conservative garbage collector can be used as a garbage collecting replacement for C `malloc` or C++ `new`. It allows you to allocate memory basically as you normally would, without explicitly deallocating memory that is no longer useful. The collector automatically recycles memory when it determines that it can no longer be otherwise accessed. A simple example of such a use is given [here](simple_example.html).

The collector is also used by a number of programming language implementations that either use C as intermediate code, want to facilitate easier interoperation with C libraries, or just prefer the simple collector interface. For a more detailed description of the interface, see [here](gcinterface.html).

Alternatively, the garbage collector may be used as a [leak detector](leak.html) for C or C++ programs, though that is not its primary goal.

The arguments for and against conservative garbage collection in C and C++ are briefly discussed in [issues.html](issues.html). The beginnings of a frequently-asked-questions list are [here](faq.html).

Empirically, this collector works with most unmodified C programs, simply by replacing `malloc` with `GC_malloc` calls, replacing `realloc` with `GC_realloc` calls, and removing free calls. Exceptions are discussed in [issues.html](issues.html).

Many versions are available in the [`gc_source`](gc_source) subdirectory. Currently a recent stable version is [`gc-8.2.4.tar.gz`](gc_source/gc-8.2.4.tar.gz). Note that this uses the new version numbering scheme and may occasionally require a separate `libatomic_ops` download (see below).

If that fails, try the latest explicitly numbered version in [`gc_source`](gc_source). Later versions may contain additional features, platform support, or bug fixes, but are likely to be less well tested.

Version 7.3 and later require that you download a corresponding version of libatomic_ops, which should be available in [`https://github.com/ivmai/libatomic_ops/wiki/Download`](https://github.com/ivmai/libatomic_ops/wiki/Download). The current (stable) version is also available as [`gc_source/libatomic_ops-7.8.0.tar.gz`](gc_source/libatomic_ops-7.8.0.tar.gz). You will need to place that in a `libatomic_ops` subdirectory.

Previously it was best to use corresponding versions of gc and libatomic_ops, but for recent versions, it should be OK to mix and match them.

Starting with 8.0, libatomic_ops is only required if the compiler does not understand gcc's C atomic intrinsics.

The development version of the GC source code now resides on github, along with the downloadable packages. The GC tree itself is at [`https://github.com/ivmai/bdwgc/`](https://github.com/ivmai/bdwgc/). The `libatomic_ops` tree required by the GC is at [`https://github.com/ivmai/libatomic_ops/`](https://github.com/ivmai/libatomic_ops/).

To build a working version of the collector, you will need to do something like the following, where `D` is the absolute path to an installation directory:

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

This will require that you have C and C++ toolchains, `git`, `automake`, `autoconf`, and `libtool` already installed.

Historical versions of the source can still be found on the SourceForge site (project "bdwgc") [here](http://bdwgc.cvs.sourceforge.net/bdwgc/).

The garbage collector code is copyrighted by [Hans-J. Boehm](http://hboehm.info), Alan J. Demers, [Xerox Corporation](http://www.xerox.com), [Silicon Graphics](http://www.sgi.com), and [Hewlett-Packard Company](http://www.hp.com). It may be used and copied without payment of a fee under minimal restrictions. See the README file in the distribution or the [license](license.txt) for more details. **IT IS PROVIDED AS IS, WITH ABSOLUTELY NO WARRANTY EXPRESSED OR IMPLIED. ANY USE IS AT YOUR OWN RISK**.

The collector is not completely portable, but the distribution includes ports to most standard PC and UNIX/Linux platforms. The collector should work on Linux, *BSD, recent Windows versions, MacOS X, HP/UX, Solaris, Tru64, Irix and a few other operating systems. Some ports are more polished than others. There are [instructions](porting.html) for porting the collector to a new platform.

Irix pthreads, Linux threads, Win32 threads, Solaris threads (old style and pthreads), HP/UX 11 pthreads, Tru64 pthreads, and MacOS X threads are supported in recent versions.

### Separately distributed ports

For MacOS 9/Classic use, Patrick Beard's latest port were once available from `http://homepage.mac.com/pcbeard/gc/`.

Several Linux and BSD versions provide prepacked versions of the collector. The Debian port can be found at [https://packages.debian.org/sid/libgc-dev](https://packages.debian.org/sid/libgc-dev).

Kenjiro Taura, Toshio Endo, and Akinori Yonezawa developed a parallel collector based on this one. It was once available from `http://www.yl.is.s.u-tokyo.ac.jp/gc/` Their collector takes advantage of multiple processors during a collection. Starting with collector version 6.0alpha1 we also do this, though with more modest processor scalability goals. Our approach is discussed briefly in [`scale.html`](scale.html). The collector uses a [mark-sweep](complexity.html) algorithm. It provides incremental and generational collection under operating systems which provide the right kind of virtual memory support. (Currently this includes SunOS[45], IRIX, OSF/1, Linux, and Windows, with varying restrictions.) It allows [*finalization*](finalization.html) code to be invoked when an object is collected. It can take advantage of type information to locate pointers if such information is provided, but it is usually used without such information. ee the README and `gc.h` files in the distribution for more details.

For an overview of the implementation, see [here](gcdescr.html).

The garbage collector distribution includes a C string ([*cord*](gc_source/cordh.txt)) package that provides for fast concatenation and substring operations on long strings. A simple curses- and win32-based editor that represents the entire file as a cord is included as a sample application.

Performance of the nonincremental collector is typically competitive with malloc/free implementations. Both space and time overhead are likely to be only slightly higher for programs written for malloc/free (see Detlefs, Dosser and Zorn's [Memory Allocation Costs in Large C and C++ Programs](ftp://ftp.cs.colorado.edu/pub/techreports/zorn/CU-CS-665-93.ps.Z).) For programs allocating primarily very small objects, the collector may be faster; for programs allocating primarily large objects it will be slower. If the collector is used in a multithreaded environment and configured for thread-local allocation, it may in some cases significantly outperform malloc/free allocation in time.

We also expect that in many cases any additional overhead will be more than compensated for by decreased copying etc. if programs are written and tuned for garbage collection.

**The beginnings of a frequently asked questions list for this collector are [here](faq.html)**.

**The following provide information on garbage collection in general**:

Paul Wilson's [garbage collection ftp archive](ftp://ftp.cs.utexas.edu/pub/garbage) and [GC survey](ftp://ftp.cs.utexas.edu/pub/garbage/gcsurvey.ps).

The Ravenbrook [Memory Management Reference](http://www.memorymanagement.org/).

David Chase's [GC FAQ](http://www.iecc.com/gclist/GC-faq.html).

Richard Jones' [GC page](http://www.ukc.ac.uk/computer_science/Html/Jones/gc.html) and [his book](//www.cs.kent.ac.uk/people/staff/rej/gcbook/gcbook.html).

**The following papers describe the collector algorithms we use and the underlying design decisions at a higher level.**

(Some of the lower level details can be found [here](gcdescr.html).)

The first one is not available electronically due to copyright considerations. Most of the others are subject to ACM copyright.

Boehm, H., "Dynamic Memory Allocation and Garbage Collection", *Computers in Physics 9*, 3, May/June 1995, pp. 297-303\. This is directed at an otherwise sophisticated audience unfamiliar with memory allocation issues. The algorithmic details differ from those in the implementation. There is a related letter to the editor and a minor correction in the next issue.

Boehm, H., and [M. Weiser](http://www.ubiq.com/hypertext/weiser/weiser.html), ["Garbage Collection in an Uncooperative Environment"](../spe_gc_paper), *Software Practice & Experience*, September 1988, pp. 807-820.

Boehm, H., A. Demers, and S. Shenker, ["Mostly Parallel Garbage Collection"](papers/pldi91.ps.Z), Proceedings of the ACM SIGPLAN '91 Conference on Programming Language Design and Implementation, *SIGPLAN Notices 26*, 6 (June 1991), pp. 157-164.

Boehm, H., ["Space Efficient Conservative Garbage Collection"](papers/pldi93.ps.Z), Proceedings of the ACM SIGPLAN '93 Conference on Programming Language Design and Implementation, *SIGPLAN Notices 28*, 6 (June 1993), pp. 197-206.

Boehm, H., "Reducing Garbage Collector Cache Misses", *Proceedings of the 2000 International Symposium on Memory Management* . [Official version.](http://portal.acm.org/citation.cfm?doid=362422.362438) [Technical report version.](http://www.hpl.hp.com/techreports/2000/HPL-2000-99.html) Describes the prefetch strategy incorporated into the collector for some platforms. Explains why the sweep phase of a "mark-sweep" collector should not really be a distinct phase.

M. Serrano, H. Boehm, "Understanding Memory Allocation of Scheme Programs", *Proceedings of the Fifth ACM SIGPLAN International Conference on Functional Programming*, 2000, Montreal, Canada, pp. 245-256. [Official version.](http://www.acm.org/pubs/citations/proceedings/fp/351240/p245-serrano/) [Earlier Technical Report version.](http://www.hpl.hp.com/techreports/2000/HPL-2000-62.html) Includes some discussion of the collector debugging facilities for identifying causes of memory retention.

Boehm, H., "Fast Multiprocessor Memory Allocation and Garbage Collection", [HP Labs Technical Report HPL 2000-165](http://www.hpl.hp.com/techreports/2000/HPL-2000-165.html). Discusses the parallel collection algorithms, and presents some performance results.

Boehm, H., "Bounding Space Usage of Conservative Garbage Collectors", *Proceeedings of the 2002 ACM SIGPLAN-SIGACT Symposium on Principles of Programming Languages*, Jan. 2002, pp. 93-100. [Official version.](http://portal.acm.org/citation.cfm?doid=503272.503282) [Technical report version.](http://www.hpl.hp.com/techreports/2001/HPL-2001-251.html) Includes a discussion of a collector facility to much more reliably test for the potential of unbounded heap growth.

**The following papers discuss language and compiler restrictions necessary to guaranteed safety of conservative garbage collection.**

We thank John Levine and JCLT for allowing us to make the second paper available electronically, and providing PostScript for the final version.

Boehm, H., [``Simple Garbage-Collector-Safety''](papers/pldi96.ps.gz), Proceedings of the ACM SIGPLAN '96 Conference on Programming Language Design and Implementation.

Boehm, H., and D. Chase, [``A Proposal for Garbage-Collector-Safe C Compilation''](papers/boecha.ps.gz), *Journal of C Language Translation 4*, 2 (Decemeber 1992), pp. 126-141.

**Other related information:**

The Detlefs, Dosser and Zorn's [Memory Allocation Costs in Large C and C++ Programs](ftp://ftp.cs.colorado.edu/pub/techreports/zorn/CU-CS-665-93.ps.Z). This is a performance comparison of the Boehm-Demers-Weiser collector to malloc/free, using programs written for malloc/free.

Joel Bartlett's [mostly copying conservative garbage collector for C++](ftp://ftp.digital.com/pub/DEC/CCgc).

John Ellis and David Detlef's [Safe Efficient Garbage Collection for C++](ftp://parcftp.xerox.com/pub/ellis/gc/gc.ps) proposal.

Henry Baker's [paper collection](http://home.pipeline.com/~hbaker1/).

Slides for Hans Boehm's [Allocation and GC Myths](myths.ps) talk.

**This section has not been updated recently.**

Known current users of some variant of this collector include:

The runtime system for [GCJ](http://gcc.gnu.org/java), the static GNU java compiler.

[W3m](http://w3m.sourceforge.net/), a text-based web browser.

Some versions of the Xerox DocuPrint printer software.

The [Mozilla](http://www.mozilla.org) project, as leak detector.

The [Mono](http://www.go-mono.com) project, an open source implementation of the .NET development framework.

The [DotGNU Portable.NET project](http://www.gnu.org/projects/dotgnu/), another open source .NET implementation.

The [Irssi IRC client](http://irssi.org/).

[The Berkeley Titanium project](http://titanium.cs.berkeley.edu/).

[The NAGWare f90 Fortran 90 compiler](http://www.nag.co.uk/nagware_fortran_compilers.asp).

Elwood Corporation's [Eclipse](http://www.elwood.com/eclipse-info/index.htm) Common Lisp system, C library, and translator.

The [Bigloo Scheme](http://www-sop.inria.fr/mimosa/fp/Bigloo/) and [Camloo ML compilers](http://kaolin.unice.fr/~serrano/camloo.html) written by Manuel Serrano and others.

Brent Benson's [libscheme](http://ftp.cs.indiana.edu/pub/scheme-repository/imp/).

The [MzScheme](http://www.cs.rice.edu/CS/PLT/packages/mzscheme/index.html) scheme implementation.

The [University of Washington Cecil Implementation](http://www.cs.washington.edu/research/projects/cecil/www/cecil-home.html).

[The Berkeley Sather implementation](http://www.icsi.berkeley.edu:80/Sather/).

[The Berkeley Harmonia Project](http://www.cs.berkeley.edu/~harmonia/).

The [Toba](http://www.cs.arizona.edu/sumatra/toba/) Java Virtual Machine to C translator.

The [Gwydion Dylan compiler](http://www.gwydiondylan.org/).

The [GNU Objective C runtime](http://gcc.gnu.org/onlinedocs/gcc/Objective-C.html).

[Macaulay 2](http://www.math.uiuc.edu/Macaulay2), a system to support research in algebraic geometry and commutative algebra.

The [Vesta](http://www.vestasys.org/) configuration management system.

[Visual Prolog 6](http://www.visual-prolog.com/vip6).

[Asymptote LaTeX-compatible vector graphics language.](http://asymptote.sf.net)

[A simple illustration of how to build and use the collector.](simple_example.html).

[Description of alternate interfaces to the garbage collector.](gcinterface.html)

[Slides from an ISMM 2004 tutorial about the GC.](04tutorial.pdf)

[A FAQ (frequently asked questions) list.](faq.html)

[How to use the garbage collector as a leak detector.](leak.html)

[Some hints on debugging garbage collected applications.](debugging.html)

[An overview of the implementation of the garbage collector.](gcdescr.html)

[Instructions for porting the collector to new platforms.](porting.html)

[The data structure used for fast pointer lookups.](tree.html)

[Scalability of the collector to multiprocessors.](scale.html)

[Directory containing garbage collector source.](gc_source)

[An attempt to establish a bound on space usage of conservative garbage collectors.](bounds.html)

[Mark-sweep versus copying garbage collectors and their complexity.](complexity.html)

[Pros and cons of conservative garbage collectors, in comparison to other collectors.](conservative.html)

[Issues related to garbage collection vs. manual memory management in C/C++.](issues.html)

[An example of a case in which garbage collection results in a much faster implementation as a result of reduced synchronization.](example.html)

[Slide set discussing performance of nonmoving garbage collectors.](nonmoving)

[Slide set discussing *Destructors, Finalizers, and Synchronization* (POPL 2003).](http://hboehm.info/popl03/web)

[Paper corresponding to above slide set.](http://portal.acm.org/citation.cfm?doid=604131.604153) ( [Technical Report version](http://www.hpl.hp.com/techreports/2002/HPL-2002-335.html).)

[A Java/Scheme/C/C++ garbage collection benchmark.](gc_bench.html)

[Slides for talk on memory allocation myths.](myths.ps)

[Slides for OOPSLA 98 garbage collection talk.](gctalk.ps)

[Related papers.](papers)

For current updates on the collector and information on reporting issues, please see the [bdwgc github page](https://github.com/ivmai/bdwgc#feedback-contribution-questions-and-notifications).

There used to be a pair of mailing lists for GC discussions. Those are no longer active. Please see the above page for pointers to the archives.

Some now ancient discussion of the collector took place on the gcc java mailing list, whose archives appear [here](http://gcc.gnu.org/ml/java/), and also on [gclist@iecc.com](http://lists.tunes.org/mailman/listinfo/gclist).

Comments and bug reports may also be sent to ([boehm@acm.org](mailto:boehm@acm.org)), but reporting issues on [github](https://github.com/ivmai/bdwgc/issues) is strongly preferred.