<!--yml

分类：未分类

日期：2024-05-27 15:01:34

-->

# 现代硬件算法 - Algorithmica

> 来源：[https://en.algorithmica.org/hpc/](https://en.algorithmica.org/hpc/)

这是即将发布的高性能计算书籍《现代硬件算法》，作者是[Sergey Slotin](http://sereja.me/)。

它的预期读者是所有性能工程师和实用算法研究人员，以及刚刚完成高级算法课程并希望了解更多实用方法来加速程序的计算机科学本科生，方法不仅仅局限于从$O(n \log n)$到$O(n \log \log n)$。

所有书籍材料都托管在GitHub上，代码在[单独的存储库](https://github.com/sslotin/scmm-code)中。这不是一个协作项目，但非常欢迎任何贡献和反馈。

### 常见问题解答

**Bug/typo fixes.** 如果你在任何页面上发现错误，请按以下任一方法之一处理 — 依次优先：

+   立即修复它，方法是点击任何页面右上角的铅笔图标（打开[Prose](https://prose.io/)编辑器），或者更传统地，直接在GitHub上修改页面（源链接也在右上角）；

+   在GitHub上[创建一个问题](https://github.com/algorithmica-org/algorithmica/issues);

+   [直接告诉我](http://sereja.me/)。

或在讨论它的其他网站上留下评论 — 我会阅读大部分被标记的[HackerNews](https://news.ycombinator.com/from?site=algorithmica.org)、[CodeForces](https://codeforces.com/profile/sslotin)和[Twitter](https://twitter.com/sergey_slotin)讨论帖。

**发布日期。** 本书分为几个部分，我计划顺序完成，并在其中长时间休息。第一部分，性能工程，截至2022年3月，完成约75%，希望到今年夏天时完成超过95%。

对于这样一本开源书籍，“发布”基本上意味着：

+   完成所有基本部分并填充所有的TODOs，

+   大部分时间冻结目录（除了案例研究），

+   进行最后一轮重度剪辑（希望能得到专业编辑的帮助 — 我还没搞清楚逗号在英语中怎么用），

+   制作插图（目前显示的大部分都是我偷来的），

+   制作适合打印的PDF并找出最佳的分发方式。

之后，我将主要修复错误，只做一些反映技术变化或新算法进展的小修订。电子书/印刷版很可能以“随你付费”的方式销售，无论如何，网络版本始终完全在线上。

**预订/财务支持这本书。** 由于我的不幸国籍和出生地，你不能 — 也就是说，直到我找到既符合国际制裁，又不资助[战争](https://en.wikipedia.org/wiki/2022_Russian_invasion_of_Ukraine)，并且不会因逃税而将我送进监狱的方法。

所以，请不要烦恼。如果你想支持这本书，只需分享它并帮忙修复错别字 — 那就足够了。

**翻译。** 该网站有一个单独的功能用于创建和管理翻译 — 我已经被一些愿意将这本书翻译成意大利语和汉语的友好人士联系过（我也将亲自翻译其中的一部分成为我的母语俄语）。

但是，由于这本书还在不断发展中，现在开始翻译它至少要等到第一部分完成可能并不是最好的主意。尽管如此，非常鼓励你翻译任何文章并在你的博客上发布 — 只需把链接发给我，我们可以在集中翻译开始时将其合并回来。

**“翻译”俄文版本。** 托管在[ru.algorithmica.org/cs/](https://ru.algorithmica.org/cs/)上的文章不是关于高级性能工程，而主要是关于经典计算机科学算法 — — 没有讨论如何超越渐近复杂性加速它们。那里的大部分信息并不独特，已经存在于互联网上的其他地方，例如类似精神的[cp-algorithms.com](https://cp-algorithms.com/)。

**大学中的性能工程教学。** 我写这本书的一个目标是改变计算机科学 — 具体说来是算法设计在大学中的教学方式。让我详细解释一下。

计算机科学课程建立在两本极具影响力的教科书上。这两本书毫无疑问都是杰出的，但[其中一本](https://en.wikipedia.org/wiki/The_Art_of_Computer_Programming)已有50年历史，[另一本](https://en.wikipedia.org/wiki/Introduction_to_Algorithms)则有30年历史，而[计算机](/hpc/complexity/hardware)自那时以来发生了很大的变化。渐近复杂性不再是唯一的决定因素。在现代实际算法设计中，你选择的方法更多地利用了硬件上可用的不同类型并行性，而不是理论上在银河规模输入上执行更少的原始操作。

然而，大多数大学的计算机科学课程完全忽视了这一变革。虽然有一些旨在纠正这一问题的优秀课程，比如来自麻省理工学院的“[软件系统性能工程](https://ocw.mit.edu/courses/electrical-engineering-and-computer-science/6-172-performance-engineering-of-software-systems-fall-2018/)”，阿尔托大学的“[并行计算机编程](https://ppc.cs.aalto.fi/)”，以及像Denis Bakhvalov的非学术课程“[性能忍者](https://github.com/dendibakh/perf-ninja)”等，但大多数计算机科学毕业生仍然把现代硬件看作是上世纪90年代的东西。

我真正想要实现的是，在算法介绍之后立即开始教授性能工程。编写这门学科的第一本全面教材是其中的一部分，这也是我赶在夏季完成它的原因，以便大学可以在下一个学年推广它。但要创建一门新课程需要更多的内容：你需要一个平衡的课程设置，课程基础设施，讲座幻灯片，实验作业……因此，在完成主要著作之后的一段时间里，我将致力于课程材料和*教学*性能工程工具的开发——我期待与其他有意推动这一现实的人进行合作。

### 第一部分：性能工程

第一部分涵盖了计算机体系结构的基础知识和单线程算法的优化。

它详细介绍了主要的CPU优化主题，如缓存，SIMD和流水线，并提供了C++的简短示例，然后是大案例研究，在这些案例中，我们通常可以比STL算法或数据结构显著加快速度。

计划目录：

```
0\. Preface
1\. Complexity Models
 1.1\. Modern Hardware
 1.2\. Programming Languages
 1.3\. Models of Computation
 1.4\. When to Optimize
2\. Computer Architecture
 1.1\. Instruction Set Architectures
 1.2\. Assembly Language
 1.3\. Loops and Conditionals
 1.4\. Functions and Recursion
 1.5\. Indirect Branching
 1.6\. Machine Code Layout
 1.7\. System Calls
 1.8\. Virtualization
3\. Instruction-Level Parallelism
 3.1\. Pipeline Hazards
 3.2\. The Cost of Branching
 3.3\. Branchless Programming
 3.4\. Instruction Tables
 3.5\. Instruction Scheduling
 3.6\. Throughput Computing
 3.7\. Theoretical Performance Limits
4\. Compilation
 4.1\. Stages of Compilation
 4.2\. Flags and Targets
 4.3\. Situational Optimizations
 4.4\. Contract Programming
 4.5\. Non-Zero-Cost Abstractions
 4.6\. Compile-Time Computation
 4.7\. Arithmetic Optimizations
 4.8\. What Compilers Can and Can't Do
5\. Profiling
 5.1\. Instrumentation
 5.2\. Statistical Profiling
 5.3\. Program Simulation
 5.4\. Machine Code Analyzers
 5.5\. Benchmarking
 5.6\. Getting Accurate Results
6\. Arithmetic
 6.1\. Floating-Point Numbers
 6.2\. Interval Arithmetic
 6.3\. Newton's Method
 6.4\. Fast Inverse Square Root
 6.5\. Integers
 6.6\. Integer Division
 6.7\. Bit Manipulation
(6.8\. Data Compression)
7\. Number Theory
 7.1\. Modular Inverse
 7.2\. Montgomery Multiplication
(7.3\. Finite Fields)
(7.4\. Error Correction)
 7.5\. Cryptography
 7.6\. Hashing
 7.7\. Random Number Generation
8\. External Memory
 8.1\. Memory Hierarchy
 8.2\. Virtual Memory
 8.3\. External Memory Model
 8.4\. External Sorting
 8.5\. List Ranking
 8.6\. Eviction Policies
 8.7\. Cache-Oblivious Algorithms
 8.8\. Spacial and Temporal Locality
(8.9\. B-Trees)
(8.10\. Sublinear Algorithms)
(9.13\. Memory Management)
9\. RAM & CPU Caches
 9.1\. Memory Bandwidth
 9.2\. Memory Latency
 9.3\. Cache Lines
 9.4\. Memory Sharing
 9.5\. Memory-Level Parallelism
 9.6\. Prefetching
 9.7\. Alignment and Packing
 9.8\. Pointer Alternatives
 9.9\. Cache Associativity
 9.10\. Memory Paging
 9.11\. AoS and SoA
10\. SIMD Parallelism
 10.1\. Intrinsics and Vector Types
 10.2\. Moving Data
 10.3\. Reductions
 10.4\. Masking and Blending
 10.5\. In-Register Shuffles
 10.6\. Auto-Vectorization and SPMD
11\. Algorithm Case Studies
 11.1\. Binary GCD
(11.2\. Prime Number Sieves)
 11.3\. Integer Factorization
 11.4\. Logistic Regression
 11.5\. Big Integers & Karatsuba Algorithm
 11.6\. Fast Fourier Transform
 11.7\. Number-Theoretic Transform
 11.8\. Argmin with SIMD
 11.9\. Prefix Sum with SIMD
 11.10\. Reading Decimal Integers
 11.11\. Writing Decimal Integers
(11.12\. Reading and Writing Floats)
(11.13\. String Searching)
 11.14\. Sorting
 11.15\. Matrix Multiplication
12\. Data Structure Case Studies
 12.1\. Binary Search
 12.2\. Static B-Trees
(12.3\. Search Trees)
 12.4\. Segment Trees
(12.5\. Tries)
(12.6\. Range Minimum Query)
 12.7\. Hash Tables
(12.8\. Bitmaps)
(12.9\. Probabilistic Filters) 
```

我们将加速的一些酷东西包括：

+   2倍速度更快的最大公约数（与`std::gcd`相比）

+   8-15倍速度更快的二分查找（与`std::lower_bound`相比）

+   5-10倍速度更快的段树（与Fenwick树相比）

+   5倍速度更快的哈希表（与`std::unordered_map`相比）

+   2倍速度更快的popcount（与重复调用`popcnt`相比）

+   35倍速度更快的整数序列解析（与`scanf`相比）

+   ?倍速度更快的排序（与`std::sort`相比）

+   2倍速度更快的求和（与`std::accumulate`相比）

+   2-3倍速度更快的前缀和（与朴素实现相比）

+   10倍速度更快的argmin（与朴素实现相比）

+   10倍速度更快的数组搜索（与`std::find`相比）

+   15倍速度更快的搜索树（与`std::set`相比）

+   100倍速度更快的矩阵乘法（与“for-for-for”相比）

+   最优字长整数因式分解（每60位整数约0.4毫秒）

+   最优Karatsuba算法

+   最优FFT

页数：450-600页

发行日期：2022年第三季度

### 第二部分：并行算法

并发性，并行模型，上下文切换，绿色线程，并行运行时，缓存一致性，同步原语，OpenMP，归约，扫描，链表排名，图算法，无锁数据结构，异构计算，CUDA，内核，线程束，块，矩阵乘法，排序。

页数：150-200页

发行日期: 2023-2024？

### 第三部分: 分布式计算

网络，消息传递，Actor模型，通信受限算法，分布式基本功能，全归约，MapReduce，流处理，查询规划，存储，分片，压缩，分布式数据库，一致性，可靠性，调度，工作流引擎，云计算。

发行日期: ???（更有可能完成）

### 第四部分: 软件与硬件

LLVM IR，编译优化和后端，解释器，JIT编译，Cython，JAX，Numba，Julia，OpenCL，DPC++，oneAPI，XLA，（基本）Verilog，FPGAs，ASICs，TPUs以及其他人工智能加速器。

发行日期: ???（不太可能完成）

### 致谢

本书主要基于博客文章、研究论文、会议演讲以及其他许多人所著作的作品:

### 免责声明: 技术选择

本书中的示例使用了C++，GCC，x86-64，CUDA和Spark，尽管传达的底层原则并不特定于它们。

为了安抚我的良心，我对这些选择都不满意: 这些技术只是目前最普遍和稳定的，因此对读者更有帮助。 我本来会选择C / Rust / [Carbon?](https://github.com/carbon-language/carbon-lang), LLVM, arm, OpenCL和Dask；也许会有第二版，在其中一些技术栈会改变。
