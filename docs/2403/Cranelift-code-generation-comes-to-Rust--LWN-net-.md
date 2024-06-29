<!--yml

分类：未分类

日期：2024-05-27 15:05:35

-->

# Cranelift代码生成进入Rust[LWN.net]

> 来源：[https://lwn.net/SubscriberLink/964735/8b795f23495af1d4/](https://lwn.net/SubscriberLink/964735/8b795f23495af1d4/)

| **本文由LWN订阅者支持**LWN.net的订阅者使本文及其周边一切成为可能。如果您欣赏我们的内容，请[购买订阅](/subscribe/)，支持我们制作更多的文章。 |
| --- |

作者：**达洛克·奥尔登**

2024年3月15日

[Cranelift](https://cranelift.dev/)是作为[WebAssembly](https://webassembly.org/)运行时的一部分开发的Apache-2.0许可的代码生成后端。2023年10月，Rust项目将Cranelift作为其每夜构建工具链的可选组件提供给用户。现在用户可以将Cranelift作为Rust项目调试构建的代码生成后端，这是查看Cranelift不同之处的一个适当时机。Cranelift旨在通过仅优先考虑最重要的优化来生成比现有编译器更快的代码。

快速编译时间是用户从他们的编程语言中想要的许多功能之一。尽管Rust和LLVM项目持续稳定进展，编译时间长期以来一直是关于Rust（以及[使用LLVM的其他语言](/Articles/959915/)）的抱怨来源。此外，一个能够快速生成代码的编译器在当前更合理使用解释器的应用程序中有潜在的可行性。所有这些因素都表明，一个专注于编译速度而不是生成代码速度的编译器可能非常有价值。

Cranelift首次用作Wasmtime的即时（JIT）编译器的后端。现在许多语言都配备了JIT编译器，这些编译器通常使用专门的技巧来快速编译孤立的函数。例如，Python [最近添加了一个复制和补丁的JIT](/Articles/958350/)，它通过在内存中获取每个Python字节码的预编译代码段并将它们拼接在一起来工作。JIT编译器通常使用诸如推测性优化之类的技术，这使得在其原始上下文之外重用编译器变得困难，因为它们对设计时的特定语言进行了许多假设。

Cranelift的开发人员选择使用更通用的架构，这意味着Cranelift可在WebAssembly之外的环境中使用。最初设计项目时，考虑了在Wasmtime、Rust和Firefox的[SpiderMonkey JavaScript解释器](https://spidermonkey.dev/)中使用Cranelift。SpiderMonkey项目目前决定暂不使用Cranelift，但Cranelift项目仍具有旨在轻松融入其他程序的设计。

Cranelift采用了称为CLIF的[自定义中间表示](https://github.com/bytecodealliance/wasmtime/blob/main/cranelift/docs/ir.md)，并直接为目标架构生成机器代码。与许多其他即时编译器不同，Cranelift不生成依赖于能够在假设失效时返回使用解释器的代码。这使其适合用于非与WebAssembly相关的项目中采用。

#### Cranelift的优化

尽管Cranelift专注于快速代码生成，但它确实通过多种方式优化生成的代码。Cranelift的优化流水线基于[等价图](https://zh.wikipedia.org/wiki/%E7%AD%89%E4%BB%B7%E5%9B%BE)（或称E图），这是一种有效表示等价中间表示集合的数据结构。在传统编译器中，优化器通过获取解析生成的程序表示，然后对其应用一系列传递以生成优化版本。优化传递的执行顺序对生成的代码质量有很大影响，因为某些传递需要其他传递简化后的简化才能应用。选择正确的优化顺序称为[阶段排序问题](https://ieeexplore.ieee.org/document/1611550)，已成为大量学术研究的来源。

在Cranelift中，每个优化的一部分识别特定结构的更简单或更快表示方法，与最终选择应使用的表示方法分离开来。每个优化通过在内部表示中找到特定模式，然后注释它等效于某些简化版本来工作。E图数据结构通过允许两个内部表示的副本共享它们共同拥有的节点，并允许CLIF中的节点引用其他节点的等效类，而不是特定的其他节点，有效地表示了这一点。这产生了一个密集的结构，在其中添加程序特定部分的替代形式是便宜的。

因为优化只在 E-图上运行，以新注释的形式添加信息，所以优化的顺序不会改变结果。只要编译器继续运行优化，直到它们不再有任何新的匹配（这个过程被称为[等饱和](https://arxiv.org/abs/1012.1802)），E-图将包含等同于传统优化 passes 等效顺序产生的表示，以及许多不那么高效的表示。E-图比直接存储每个可能的替代方案更高效（平均占用 O(log n) 的空间），但它们仍比传统的中间表示花费更多的内存。根据所讨论的程序和所使用的优化集合，一个完全饱和的 E-图可能会非常大。在实践中，Cranelift 设置了对图执行多少次操作的限制，以防止它变得过大。

当从 E-图中提取最终表示以用于代码生成时，E-图的简单性和优化性也付出了代价。从 E-图中提取最快的表示是一个[NP完全](https://effect.systems/blog/egraph-extraction.html)问题。Cranelift 使用一组启发式方法快速提取一个足够好的表示。

将一个 NP 完全问题（选择一组 passes 的最佳顺序）换成另一个可能看起来并不像是一个很大的好处，但对于一个较小的项目来说是有意义的。优化 passes 的顺序主要由编写优化的程序员设定，因为它需要领域知识来选择合理的顺序。另一方面，从 E-图中提取高效的表示是一个通用的搜索问题，可以根据应用程序允许的计算时间多少来进行。Cranelift 的启发式方法并不提取最高效的表示，但确实能快速提取一个不错的表示。

以这种方式表示优化也使得 Cranelift 的维护者更容易理解和调试现有的优化及其效果，并使得编写新的优化稍微简单些。Cranelift 使用一个[定制的领域特定语言](https://github.com/bytecodealliance/wasmtime/blob/522f9711ad57e3c00f394691fbc5cde0fdf8017d/cranelift/isle/README.md)（ISLE）来内部指定优化。

虽然Cranelift没有按阶段组织其优化，但它在自己的ISLE文件中定义了十个不同的相关优化集，这允许与GCC和LLVM进行大致比较。LLVM在其文档中[列出了](https://llvm.org/docs/Passes.html)96个优化过程，而GCC有[372个](https://github.com/gcc-mirror/gcc/blob/master/gcc/passes.def)。Cranelift现有的优化包括常量传播、位操作简化、矢量化、浮点操作优化以及比较的归一化。通过从E图中提取表示来隐式地进行死代码消除。

[2020年的一篇论文](https://arxiv.org/pdf/2011.13127.pdf)显示，Cranelift比LLVM快一个数量级，但在某些基准测试中生成的代码大约慢两倍。然而，Cranelift仍然比论文作者的自定义复制和补丁JIT编译器慢。

#### 适用于Rust的Cranelift

Cranelift可能是为了成为Rust的备用后端而设计的，但要使其可用实际上需要付出了巨大的努力。Rust编译器有一个称为[mid-level IR](https://blog.rust-lang.org/2016/04/19/MIR.html)的内部表示(IR)，用于表示类型检查的程序。通常情况下，编译器将其转换为LLVM IR，然后发送到LLVM代码生成后端。为了使用Cranelift，编译器需要[另一个库](https://github.com/rust-lang/rustc_codegen_cranelift)，该库接受mid-level IR并生成CLIF。

那个库主要由Rust编译器团队成员"bjorn3"编写，他为Rust的Cranelift后端贡献了大约4000次提交中的超过3000次。他撰写了一系列详细描述其工作的[进展报告](https://bjorn3.github.io/)。开发始于2018年，并跟随Rust自身的快速发展步伐。到2023年，后端被认为足够稳定，作为Rust nightly的可选工具链组件之一发布。

现在，人们可以使用`rustup`和`cargo`尝试Cranelift后端：

```
    $ rustup component add rustc-codegen-cranelift-preview --toolchain nightly
    $ export CARGO_PROFILE_DEV_CODEGEN_BACKEND=cranelift
    $ cargo +nightly build -Zcodegen-backend

```

给定的`rustup`命令将Cranelift后端的动态库添加到要下载并在本地保持更新的工具链组件集中。设置`CARGO_PROFILE_DEV_CODEGEN_BACKEND`环境变量指示`cargo`在调试构建中使用Cranelift，并且最终的`cargo`调用构建当前目录中的任何Rust项目，使用了启用的替代代码生成后端功能。来自[bjorn3的最新进展报告](https://bjorn3.github.io/2023/10/31/progress-report-oct-2023.html)包含了有关如何配置Cargo默认使用新后端的额外细节，而无需复杂的命令行操作。

Cranelift 本身是用 Rust 编写的，这使得它可以作为一个基准来与 LLVM 进行比较。在我的电脑上，使用 Cranelift 后端对 Cranelift 本身进行完整的调试构建花费了 29.6 秒，而使用 LLVM 则需要 37.5 秒（墙钟时间减少了 20%）。然而，这些墙钟时间并未完全反映实际情况，因为构建系统中存在并行性。使用 Cranelift 编译花费了 125 CPU 秒，而 LLVM 则需要 211 CPU 秒，差异为 40%。增量构建 —— 仅重新构建 Cranelift 本身，而不重新构建其依赖项 —— 使用两种后端都更快。CPU 时间为 66ms，相比之下 LLVM 为 90ms。

Cranelift 是否能改善 Rust 用户对编译时间慢的担忧尚待观察，但初步迹象是令人鼓舞的。无论如何，Cranelift 展示了一种不同的编译器设计方法的有趣案例。

* * *

(

[登录](https://lwn.net/Login/?target=/Articles/964735/)

发表评论)
