<!--yml

category: 未分类

date: 2024-05-27 15:02:49

-->

# 从慢到 SIMD：一个 Go 优化故事

> 来源：[https://sourcegraph.com/blog/slow-to-simd](https://sourcegraph.com/blog/slow-to-simd)

所以，有这个函数。它被频繁调用。更重要的是，所有这些调用都在关键用户交互的关键路径上。让我们谈谈如何让它变快。

据说：它是一个点积。

在 Sourcegraph，我们正在开发一个名为 [Cody](https://sourcegraph.com/cody) 的 Code AI 工具。为了让 Cody 能够很好地回答问题，我们需要为其提供足够的[上下文](https://about.sourcegraph.com/blog/cheating-is-all-you-need)。我们做到这一点的一种方式是利用[嵌入](https://platform.openai.com/docs/guides/embeddings)。

对于我们的目的，[嵌入](https://developers.google.com/machine-learning/crash-course/embeddings/video-lecture)是文本块的向量表示。它们被构造成这样，以便语义上相似的文本片段具有更相似的向量。当 Cody 需要更多信息来回答查询时，我们会对嵌入进行相似性搜索，以获取一组相关的代码块，并将这些结果提供给 Cody 以提高结果的相关性。

与此博客文章相关的部分是相似度度量标准，即确定两个向量相似程度的函数。对于相似性搜索，一个常见的度量标准是[余弦相似度](https://en.wikipedia.org/wiki/Cosine_similarity)。然而，对于归一化的向量（具有单位大小的向量），[点积](https://en.wikipedia.org/wiki/Dot_product)产生的排名与[余弦相似度等价](https://developers.google.com/machine-learning/clustering/similarity/measuring-similarity)。为了进行搜索，我们计算数据集中每个嵌入的点积，并保留前几个结果。由于我们在获得必要的上下文之前不能开始执行 LLM，因此优化此步骤至关重要。

你可能会想：为什么不只使用一个索引向量 DB 呢？除了添加我们需要管理的另一个基础设施外，索引的构建还会增加延迟并增加资源需求。此外，标准的最近邻居索引只提供近似检索，这与更容易解释的详尽搜索相比增加了另一层模糊性。鉴于此，我们决定在我们手工制作的解决方案上投入一点时间，看看我们能推动到哪里。

## [](#the-target)目标

这是一个简单的 Go 实现，用于计算两个向量的点积。我的目标是概述我优化这个函数所经历的过程，并分享一些我在这个过程中学到的工具。

```
func DotNaive(a, b []float32) float32 {
 sum := float32(0)
 for i := 0; i < len(a) && i < len(b); i++ {
 sum += a[i] * b[i]
 }
 return sum
}
```

除非另有说明，所有基准测试都在 Intel Xeon Platinum 8481C 2.70GHz CPU 上运行。这是一个 `c3-highcpu-44` GCE VM。此博客文章中的代码都可以在[此处](https://github.com/camdencheek/simd_blog)找到并运行。

## [](#loop-unrolling)循环展开

现代 CPU 有一种叫做[*指令流水线*](https://chadaustin.me/2009/02/latency-vs-throughput/)的东西，如果它发现它们之间没有数据依赖关系，它可以同时运行多条指令。数据依赖意味着一个指令的输入取决于另一个指令的输出。

在我们简单的实现中，我们的循环迭代之间存在数据依赖关系。实际上有几个。`i` 和 `sum` 每次迭代都有一个读/写对，这意味着一个迭代在前一个迭代完成之前无法开始执行。

在像这样的情况下，从我们的 CPU 中挤出更多性能的常见方法被称为[*循环展开*](https://en.wikipedia.org/wiki/Loop_unrolling)。基本思想是重新编写我们的循环，以便更多的相对高延迟的乘法指令可以同时执行。此外，它将固定的循环成本（增量和比较）分摊到多个操作中。

```
func DotUnroll4(a, b []float32) float32 {
 sum := float32(0)
 for i := 0; i < len(a); i += 4 {
 s0 := a[i] * b[i]
 s1 := a[i+1] * b[i+1]
 s2 := a[i+2] * b[i+2]
 s3 := a[i+3] * b[i+3]
 sum += s0 + s1 + s2 + s3
 }
 return sum
}
```

在我们展开的代码中，乘法指令之间的依赖关系被移除，使 CPU 能够更充分地利用流水线。这使我们的吞吐量比我们的天真实现提高了 37%。

`DotNaive`

0.94M 个向量/秒

`DotUnroll4`

1.3M 个向量/秒

请注意，我们可以通过调整我们展开的迭代次数稍微改进这一点。在基准机器上，8 似乎是最佳的，但在我的笔记本电脑上，4 的表现最佳。但是，改进非常依赖平台，并且相当小，因此在本文的其余部分，我将保持展开深度为 4 以提高可读性。

## [](#bounds-checking-elimination)边界检查消除

为了防止越界切片访问成为安全漏洞（例如著名的[心脏出血](https://en.wikipedia.org/wiki/Heartbleed)漏洞利用），Go 编译器在每次读取之前插入检查。您可以在[生成的汇编代码](https://go.godbolt.org/z/qT3M7nPGf)中查看（寻找`runtime.panic`）。

编译后的代码看起来像我们写了这样的东西：

```
func DotUnroll4(a, b []float32) float32 {
 sum := float32(0)
 for i := 0; i < len(a); i += 4 {
 if i >= cap(b) {
 panic("out of bounds")
 }
 s0 := a[i] * b[i]
 if i+1 >= cap(a) || i+1 >= cap(b) {
 panic("out of bounds")
 }
 s1 := a[i+1] * b[i+1]
 if i+2 >= cap(a) || i+2 >= cap(b) {
 panic("out of bounds")
 }
 s2 := a[i+2] * b[i+2]
 if i+3 >= cap(a) || i+3 >= cap(b) {
 panic("out of bounds")
 }
 s3 := a[i+3] * b[i+3]
 sum += s0 + s1 + s2 + s3
 }
 return sum
}
```

在像这样的热循环中，即使使用现代分支预测，每次迭代额外的分支也会导致相当大的性能损失。在我们的情况下尤其如此，因为插入的跳转限制了我们能够利用流水线的程度。

如果我们能说服编译器这些读取永远不会超出边界，它就不会插入这些运行时检查。这种技术称为“边界检查消除”，相同的模式也适用于[Go 之外的语言](https://github.com/Shnatsel/bounds-check-cookbook/)。

理论上，我们应该能够在循环外执行所有检查，并且编译器能够确定所有切片索引都是安全的。但是，我找不到合适的检查组合来说服编译器我正在做的是安全的。我最终选择了一种方式，断言长度相等并将所有边界检查移动到循环的顶部。这足以接近无边界检查版本的速度。

```
func DotBCE(a, b []float32) float32 {
 if len(a) != len(b) {
 panic("slices must have equal lengths")
 }

 if len(a)%4 != 0 {
 panic("slice length must be multiple of 4")
 }

 sum := float32(0)
 for i := 0; i < len(a); i += 4 {
 aTmp := a[i : i+4 : i+4]
 bTmp := b[i : i+4 : i+4]
 s0 := aTmp[0] * bTmp[0]
 s1 := aTmp[1] * bTmp[1]
 s2 := aTmp[2] * bTmp[2]
 s3 := aTmp[3] * bTmp[3]
 sum += s0 + s1 + s2 + s3
 }
 return sum
}
```

减少边界检查带来了 9% 的改进。一直不为零，但也没什么好写家的。

`DotNaive`

0.94M vec/s

`DotUnroll4`

1.3M vec/s

`DotBCE`

1.4M vec/s

这种技术在许多内存安全的编译语言中效果很好，比如[Rust](https://nnethercote.github.io/perf-book/bounds-checks.html)。

读者的练习：为什么我们要像 `a[i:i+4:i+4]` 这样切片，而不只是 `a[i:i+4]`？

## [](#quantization)量化

我们目前已将单核搜索吞吐量提高了约50%，但现在我们遇到了一个新的瓶颈：内存使用。我们的向量是 1536 维。每个向量有 4 字节的元素，这意味着每个向量占用 6KiB，我们大约每 GiB 代码生成一百万个向量。这很快就会累积起来。我们有一些客户给我们带来了一些庞大的单库，我们希望减少我们的内存使用量，以便更便宜地支持这些情况。

一种可能的缓解措施是将向量移动到磁盘上，但在搜索时从磁盘加载可能会增加[显著的延迟](https://colin-scott.github.io/personal_website/research/interactive_latency.html)，特别是在慢速磁盘上。相反，我们选择了用 `int8` 量化来压缩我们的向量。

有[很多种方法](https://en.wikipedia.org/wiki/Dimensionality_reduction)可以压缩向量，但我们将讨论相对简单但有效的[*整数量化*](https://huggingface.co/docs/optimum/concept_guides/quantization)。其思想是通过将 4 字节的 `float32` 向量元素转换为 1 字节的 `int8` 来降低精度。

我不会详细讨论我们如何决定在`float32`和`int8`之间进行转换，因为那是一个相当[深入的话题](https://huggingface.co/docs/optimum/concept_guides/quantization)，但可以说我们的函数现在看起来像下面这样：

```
func DotInt8BCE(a, b []int8) int32 {
 if len(a) != len(b) {
 panic("slices must have equal lengths")
 }

 sum := int32(0)
 for i := 0; i < len(a); i += 4 {
 aTmp := a[i : i+4 : i+4]
 bTmp := b[i : i+4 : i+4]
 s0 := int32(aTmp[0]) * int32(bTmp[0])
 s1 := int32(aTmp[1]) * int32(bTmp[1])
 s2 := int32(aTmp[2]) * int32(bTmp[2])
 s3 := int32(aTmp[3]) * int32(bTmp[3])
 sum += s0 + s1 + s2 + s3
 }
 return sum
}
```

这个改变导致内存使用量减少了 4 倍，但牺牲了一些准确性（我们进行了仔细的测量，但这与本篇博文无关）。

不幸的是，重新运行基准测试显示我们的搜索速度从这个改变中有所倒退。通过使用 `go tool compile -S` 查看生成的汇编代码，可以看到一些新的指令用于将 `int8` 转换为 `int32`，这可能解释了差异。我没有深入研究，因为到目前为止我们的所有性能改进在下一节中都变得无关紧要。

`DotNaive`

0.94M vec/s

`DotUnroll4`

1.3M vec/s

`DotBCE`

1.4M vec/s

`DotInt8BCE`

1.2M vec/s

## [](#simd)SIMD

到目前为止，速度的改善很好，但对于我们最大的客户来说还不够。所以我们开始尝试更加戏剧性的方法。

我总是喜欢借口去玩SIMD。而这个问题似乎是用那个锤子打的完美的钉子。

对于不熟悉的人来说，SIMD 代表"单指令多数据"。就像它的名字一样，它让您可以用单个指令对一堆数据执行操作。举个例子，要逐个元素地将两个`int32`向量相加，我们可以使用`ADD`指令逐个将它们相加，或者我们可以使用`VPADDD`指令一次性添加64对元素，具有[相同](https://uops.info/html-instr/ADD_01_R32_R32.html)的[延迟](https://uops.info/html-instr/VPADDD_YMM_YMM_M256.html)(取决于架构)。

但是我们有一个问题。Go 不像[C](https://www.intel.com/content/www/us/en/docs/intrinsics-guide/index.html)或[Rust](https://doc.rust-lang.org/beta/core/simd/index.html)那样暴露SIMD内联函数。我们有两个选择：用C编写并使用Cgo，或者为Go的汇编器手动编写。我尽量避免使用Cgo，因为有[许多并非原创的原因](https://dave.cheney.net/2016/01/18/cgo-is-not-go)，但其中一个原因是Cgo会对性能造成影响，而这段代码的性能至关重要。此外，玩一些汇编听起来很有趣，所以我会这么做。

我希望这个例程具有相当的可移植性，因此我将限制自己仅使用 AVX2 指令，这些指令在大多数当前的`x86_64`服务器CPU上都受支持。我们可以使用[运行时特性检测](https://sourcegraph.com/github.com/sourcegraph/sourcegraph@3ac2170c6523dd074835919a1804f197cf86e451/-/blob/internal/embeddings/dot_amd64.go?L17-21)在纯 Go 中退回到一个较慢的选项。

<details><summary>`DotAVX2`的完整代码</summary>

```
#include "textflag.h"
 TEXT ·DotAVX2(SB), NOSPLIT, $0-52
 // Offsets based on slice header offsets.
 // To check, use `GOARCH=amd64 go vet`
 MOVQ a_base+0(FP), AX
 MOVQ b_base+24(FP), BX
 MOVQ a_len+8(FP), DX
 XORQ R8, R8 // return sum
 // Zero Y0, which will store 8 packed 32-bit sums
 VPXOR Y0, Y0, Y0
 // In blockloop, we calculate the dot product 16 at a time
blockloop:
 CMPQ DX, $16
 JB reduce
 // Sign-extend 16 bytes into 16 int16s
 VPMOVSXBW (AX), Y1
 VPMOVSXBW (BX), Y2
 // Multiply words vertically to form doubleword intermediates,
 // then add adjacent doublewords.
 VPMADDWD Y1, Y2, Y1
 // Add results to the running sum
 VPADDD Y0, Y1, Y0
 ADDQ $16, AX
 ADDQ $16, BX
 SUBQ $16, DX
 JMP blockloop
 reduce:
 // X0 is the low bits of Y0.
 // Extract the high bits into X1, fold in half, add, repeat.
 VEXTRACTI128 $1, Y0, X1
 VPADDD X0, X1, X0
 VPSRLDQ $8, X0, X1
 VPADDD X0, X1, X0
 VPSRLDQ $4, X0, X1
 VPADDD X0, X1, X0
 // Store the reduced sum
 VMOVD X0, R8
 end:
 MOVL R8, ret+48(FP)
 VZEROALL
 RET
```</details>

实现的核心循环依赖于三个主要指令：

+   [`VPMOVSXBW`](https://www.felixcloutier.com/x86/pmovsx)，它将`int8`加载到向量`int16`中

+   [`VPMADDWD`](https://www.felixcloutier.com/x86/pmaddwd)，它将两个`int16`向量逐元素相乘，然后将相邻对组合在一起以生成一个`int32`向量

+   [`VPADDD`](https://www.felixcloutier.com/x86/paddb:paddw:paddd:paddq)，它将结果`int32`向量累加到我们的运行总和

`VPMADDWD` 在这里起到了重要作用。通过将乘法和加法步骤合并为一步，它不仅节省了指令，还通过同时将结果扩展为`int32`来帮助我们避免溢出问题。

让我们看看这给我们带来了什么。

`DotNaive`

0.94M vec/s

`DotUnroll4`

1.3M vec/s

`DotBCE`

1.4M vec/s

`DotInt8BCE`

1.2M vec/s

`DotAVX2`

7.0M vec/s

哇，这比我们之前的最佳结果提高了530％！SIMD 胜利 🚀

现在，不是一切都是阳光和彩虹的。在 Go 中手写汇编很奇怪。它使用一个 [自定义汇编器](https://go.dev/doc/asm)，这意味着它的汇编语言看起来与您通常在线找到的汇编片段有一点点不同，足以让人感到困惑。它有一些奇怪的怪癖，比如 [改变指令操作数的顺序](https://www.quasilyte.dev/blog/post/go-asm-complementary-reference/#operands-order) 或 [使用不同的指令名称](https://www.quasilyte.dev/blog/post/go-asm-complementary-reference/#mnemonics)。在 go 汇编器中，有些指令甚至 *没有* 名称，只能通过它们的 [二进制编码](https://go.dev/doc/asm#unsupported_opcodes) 使用。厚颜无耻的推广：我发现 [sourcegraph.com](https://sourcegraph.com/search) 对于查找 Go 汇编示例非常有用。

尽管如此，与 Cgo 相比，还是有一些好处的。调试仍然很有效，汇编代码可以逐步执行，并且可以使用 `delve` 检查寄存器。没有额外的构建步骤（不需要设置 C 工具链）。很容易设置一个纯 Go 的后备，以便跨平台编译仍然有效。常见问题由 `go vet` 捕获。

## [](#simdbut-bigger)SIMD...但更大

以前，我们局限于 AVX2，但如果我们 *不* 这样做呢？ AVX-512 的 VNNI 扩展添加了 [`VPDPBUSD`](https://www.felixcloutier.com/x86/vpdpbusd) 指令，它计算 `int8` 向量而不是 `int16`。这意味着我们可以在一条指令中处理四倍数量的元素，因为我们不必先转换为 `int16`，并且我们的矢量宽度在 AVX-512 中加倍！

唯一的问题是指令要求一个向量为有符号字节，另一个为 *无符号* 字节。我们的两个向量都是有符号的。我们可以采用 [英特尔开发者指南中的技巧](https://www.intel.com/content/www/us/en/docs/onednn/developer-guide-reference/2023-0/nuances-of-int8-computations.html#DOXID-DEV-GUIDE-INT8-COMPUTATIONS-1DG-I8-COMP-S12) 来帮助我们。给定两个 `int8` 元素，`a[n]` 和 `b[n]`，我们按元素计算为 `a[n]* (b[n] + 128) - a[n] * 128`。`a[n] * 128` 项是从添加 128 以将 `b[n]` 增加到 `u8` 范围的超额。我们单独跟踪它，并在最后减去它。该表达式中的每个操作都可以矢量化。

<details><summary>`DotVNNI` 的完整代码</summary>

```
#include "textflag.h"
 // DotVNNI calculates the dot product of two slices using AVX512 VNNI
// instructions The slices must be of equal length and that length must be a
// multiple of 64.
TEXT ·DotVNNI(SB), NOSPLIT, $0-52
 // Offsets based on slice header offsets.
 // To check, use `GOARCH=amd64 go vet`
 MOVQ a_base+0(FP), AX
 MOVQ b_base+24(FP), BX
 MOVQ a_len+8(FP), DX
 ADDQ AX, DX // end pointer
 // Zero our accumulators
 VPXORQ Z0, Z0, Z0 // positive
 VPXORQ Z1, Z1, Z1 // negative
 // Fill Z2 with 128
 MOVD $0x80808080, R9
 VPBROADCASTD R9, Z2
 blockloop:
 CMPQ AX, DX
 JE reduce
 VMOVDQU8 (AX), Z3
 VMOVDQU8 (BX), Z4
 // The VPDPBUSD instruction calculates of the dot product 4 columns at a
 // time, accumulating into an i32 vector. The problem is it expects one
 // vector to be unsigned bytes and one to be signed bytes. To make this
 // work, we make one of our vectors unsigned by adding 128 to each element.
 // This causes us to overshoot, so we keep track of the amount we need
 // to compensate by so we can subtract it from the sum at the end.
 //
 // Effectively, we are calculating SUM((Z3 + 128) · Z4) - 128 * SUM(Z4).
 VPADDB Z3, Z2, Z3   // add 128 to Z3, making it unsigned
 VPDPBUSD Z4, Z3, Z0 // Z0 += Z3 dot Z4
 VPDPBUSD Z4, Z2, Z1 // Z1 += broadcast(128) dot Z4
 ADDQ $64, AX
 ADDQ $64, BX
 JMP blockloop
 reduce:
 // Subtract the overshoot from our calculated dot product
 VPSUBD Z1, Z0, Z0 // Z0 -= Z1
 // Sum Z0 horizontally. There is no horizontal sum instruction, so instead
 // we sum the upper and lower halves of Z0, fold it in half again, and
 // repeat until we are down to 1 element that contains the final sum.
 VEXTRACTI64X4 $1, Z0, Y1
 VPADDD Y0, Y1, Y0
 VEXTRACTI128 $1, Y0, X1
 VPADDD X0, X1, X0
 VPSRLDQ $8, X0, X1
 VPADDD X0, X1, X0
 VPSRLDQ $4, X0, X1
 VPADDD X0, X1, X0
 // Store the reduced sum
 VMOVD X0, R8
 end:
 MOVL R8, ret+48(FP)
 VZEROALL
 RET
```</details>

这个实现产生了另外 21% 的改进。还不错！

`DotNaive`

0.94M 向量/秒

`DotUnroll4`

1.3M 向量/秒

`DotBCE`

1.4M 向量/秒

`DotInt8BCE`

1.2M 向量/秒

`DotAVX2`

7.0M 向量/秒

`DotVNNI`

8.8M 向量/秒

# [](#whats-next)接下来是什么？

嗯，我对吞吐量增加了 9.3 倍和内存使用减少了 4 倍非常满意，所以我可能会就此结束。

在这里真实的答案可能是“使用索引”。有很多关于使最近邻搜索快速的优秀工作，而且有很多内置电池的矢量数据库，使得部署变得相当容易。

*然而*，如果你想要一些有趣的思考食物，我的一个同事构建了一个概念验证的 [GPU 上的点积](https://github.com/sourcegraph/sourcegraph/compare/simd-post-gpu-embeddings~3...simd-post-gpu-embeddings)。

## [](#bonus-material)奖励材料

+   如果你还没有使用 [benchstat](https://pkg.go.dev/golang.org/x/perf/cmd/benchstat)，你应该试试。它很棒。对基准测试结果进行超级简单的统计比较。

+   别错过 [编译器浏览器](https://go.godbolt.org/z/qT3M7nPGf)，这是一个非常有用的工具，可以深入研究编译器代码生成。

+   还有那个让我被 [nerd sniped](https://twitter.com/sluongng/status/1654066471230636033) 去实现 [带有 ARM NEON 的版本](https://github.com/camdencheek/simd_blog/blob/main/dot_arm64.s) 的时刻，这引发了一些有趣的比较。

+   如果你还没有接触过，[Agner Fog 指令表](https://www.agner.org/optimize/) 是一些用于低级优化的极好的参考资料。在这项工作中，我用它们来帮助理解不同指令的延迟以及为什么有些管道比其他管道更好。
