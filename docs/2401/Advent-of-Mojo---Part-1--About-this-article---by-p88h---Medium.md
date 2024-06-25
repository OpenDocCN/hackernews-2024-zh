<!--yml

类别：未分类

日期：2024 年 5 月 27 日 14:33:05

-->

# Mojo 之旅：第一部分。关于本文 | 作者：p88h | Medium

> 来源：[`medium.com/@p88h/advent-of-mojo-part-1-c1bcaa367fcb`](https://medium.com/@p88h/advent-of-mojo-part-1-c1bcaa367fcb)

# 关于本文

这是关于在《冒险代码》中使用 Mojo 的系列文章的第一部分。

点击这里查看介绍和目录。

# 性能概览

让我们从头开始。Mojo 真的像公司宣传的那样快吗？嗯，这要看情况。当你花一些时间进行优化时，它的速度很快。在弄清楚什么有效和什么无效之后，我能够在*每一天*的挑战中取得更好的性能（请参见底部的基准测试），但是增益并不是非常一致，而且有一些注意事项。

我还没有看到任何接近“68,000 倍性能”提升的东西。《冒险代码》简直没有这种规模的问题。我猜想，那些增益可能需要大规模并行化，而虽然在 Mojo 中这相当简单，但由于问题规模较小，开销通常会吞噬掉任何收益。而且开销也不是很可预测。目前似乎没有办法对代码进行性能分析，所以这有点凭直觉。

# 向量代码

```
# Python 'dot product'
def dot(mat, vec):
    acc = 0
    K = len(mat[0])
    for row in mat:
        for i in range(K):
            acc += vec[i] * row[K - i - 1]
    return acc

# Mojo 'equivalent'
    fn dot(self, mult: SIMD[DType.int32, 32]) -> Int:
        var acc = SIMDDType.int32, 32
        for i in range(0, self.rows, 32):
            acc = self.data.aligned_simd_load32, 32.fma(mult, acc)
        return acc.reduce_add().to_int()
```

我能够实现的最高改进大约是 Python 的 1000 倍 —— 这与 Mojo 的使用案例类似 —— 低级矩阵操作。使用融合乘法和加法编写 SIMD 代码，同时使用类似 Python 的语法比 Rust 或 C++ 中的类似原生解决方案更简单。核心 SIMD 数据类型被很好地集成，并提供全面但基本的操作。如果那个特定的解决方案是可并行化的（并且要大得多），那么在某些拥有 100 个核心的怪物 CPU 上，可能会产生 10,000 倍甚至更多的改进。

还有其他优化 Python 的方法。虽然相对于 Python 的平均改进是 >100 倍（注意这包括并行化），但对于 PyPy3，相同的度量指标只有‘仅’38 倍。而且这个解决方案中的一些部分也可以使用 numpy。

# 独特的字符串

Mojo 中有很多怪癖，这些怪癖使这种速度优势很快消失。我的代码通常不使用任何动态内存分配（这可能很慢），而是选择分配大缓冲区；并且尽可能简化输入解析，以字符串切片为例（请参阅可用性部分中有关字符串的更多信息）。

```
# Reasonably fast string split.. to perform adequately against s.split(' ')
fn parsesep: Int8:
    var start: Int = 0
    for i in range(self.ptr_size):
        if self.contents[i] == sep:
            if i != start or not skip_empty:
                self.rows.push_back(StringSlice(self.contents.offset(start), i - start))
            start = i + 1
    if start < self.ptr_size or (start == self.ptr_size and not skip_empty):
        self.rows.push_back(StringSlice(self.contents.offset(start), self.ptr_size - start))
```

说到这一点，字符串本身速度较慢。在精神上相当于 Rust 的*slice*的东西是*StringRef*，但它的用途非常有限。我实现了自己的切片来完成 AOC，但这忽略了很多日常字符串处理需求，比如 UTF。当上面的解析器使用*String.find()*实现时，它比 PyPy 慢 2–3 倍，并且接近于 Python。

一些其他相关的操作很奇怪地慢。例如，*ord(‘A’)* 是我期望编译器会自动优化为常量的东西。它并不是，但如果通过使用 *alias* 强制它成为常量，那么它运行得很好，只是让代码有点奇怪。

# 简单线程

Mojo 中没有线程原语，但有更高级别的并行化。想想看。真正使某物并行的唯一方法是将其分解为一个索引化的工作函数，就像这样：

```
let A = DTypePointer[DType.int32].aligned_alloc(16, 1024)
var S = AtomicDType.int32

@parameter
fn worker(id: Int):
    var M : Int32 = 0
    for i in range(256):
        M = max(M, A[id * 256 + i])
     S.max(M)        

fn parallel_max() -> Int32:
    parallelizeworker
    return S.value
```

注意：从线程中访问可变变量是很慢的。但如果这些变量不可变（比如例子中的 A 指针），那么速度会快得多。有时候。还要注意，指针虽然是不可变的，但仍然允许改变它指向的内存内容。

似乎没有任何显式的同步（嗯，至少有原子操作）或线程通信 —— 我考虑过实现一个无锁队列，但最终 AOC 中线程的使用并不多。

对于这种简单的东西，诸如线程之类的基准测试显示出了显著的操作开销。第一天第一部分不使用线程时需要 7 微秒，使用线程时需要十倍以上。然而，第二天使用线程时能够在 40 微秒内完成，并且使用的线程数量增加了一倍。对于更长的工作项，以毫秒及以上为单位更具可预测性。我猜这与从“不插手”的方法所能期望的相当，并且即使对低级线程进行了非常小心的处理，也有一些东西是无法优化掉的。

# 总结

总体而言，Mojo 提出了一个**非常**引人注目的性能方案。我对与 C++ 的比较不多，但我做了一个例子，Mojo 非常接近。编程模型暗示着更多潜在的改进，比如将代码编译为 GPU 目标，并在假装是单个程序运行时运行，但现在还没有真正实现，很难预测实现这一点需要做出什么妥协。

在我的解决方案中，一些收益可能更多地是由于 Mojo 缺乏标准库，而不是更好的编译技术所迫使的。例如，我不会在 Python 中实现自己的低级字符串处理。当处理矩阵操作时，我可能会使用 NumPy，而不是自己编写代码。我在第 24 天就是这样做的——Python 解决方案归结为运行一个线性代数求解器（我相信它使用的是 LAPACK 的 GESV，它使用高斯消元法对矩阵进行因式分解）。我的 Mojo 代码直接实现了高斯求解器，这稍微简单一些——但该求解器针对这个 _ 具体的 _ 问题，我不能用它同时解决多个方程，而 LAPACK 可以。结果代码的运行速度比 Python 快 30 倍（这是 PyPy 比 Python 更慢的那些晦涩案例之一）。Python 中的高斯求解器甚至比 NumPy 还要慢，所以我猜这仍然是一个有效的比较，但我不确定一个 _ 通用 _ 解决方案或 Mojo 的 NumPy 等效性会有多快。

当然，ML 编程的某些方面可能会受益于这一点——也许是自定义张量操作符或激活函数，而 Modular 可能正在关注这些关键性能痛点，并且 AOC 中的一些任务支持这一假设——这意味着你可以期望这确实会提高你想要用它运行的任何训练或推断工作负载的性能，假设这些工作负载依赖于在 GPU/TPU 和 CPU 之间来回切换进行计算，而 CPU 端的计算主要是 Python。它可能在其他一些领域也很有用，前提是语言成熟一点。

是否足够受益是一个非常棘手的问题。这是一种不同的语言，而且切换是一个巨大的成本，特别是考虑到要从这些性能收益中受益，你确实必须重写代码。当然，承诺是存在的，但目前其他因素才是真正的阻碍因素——可用性、安全性和稳定性，这些是下一部分的主题。

# 基准测试

```
Task             Python      PyPy3       Mojo       parallel    * speedup
Day01 part1     1.16 ms     0.23 ms     7.71 μs     0.07 ms     * 30 - 150
Day01 part2     4.97 ms     0.51 ms     0.07 ms     0.11 ms     * 7 - 72
Day02 part1     0.41 ms     0.16 ms     0.12 ms     0.04 ms     * 3 - 9
Day02 part2     0.41 ms     0.16 ms     0.12 ms     0.04 ms     * 3 - 9
Day03 parse     0.97 ms     0.16 ms     0.03 ms     nan         * 4 - 29
Day03 part1     0.50 ms     0.06 ms     7.85 μs     nan         * 7 - 63
Day03 part2     1.22 ms     0.09 ms     0.02 ms     nan         * 5 - 64
Day04 parse     0.64 ms     0.34 ms     0.02 ms     nan         * 13 - 25
Day04 part1     0.11 ms     0.07 ms     0.36 μs     nan         * 185 - 310
Day04 part2     0.16 ms     0.07 ms     0.68 μs     nan         * 100 - 234
Day05 parse     0.10 ms     0.06 ms     0.02 ms     nan         * 2 - 4
Day05 part1     0.08 ms     5.98 μs     4.94 μs     nan         * 1 - 15
Day05 part2     0.35 ms     0.13 ms     8.04 μs     nan         * 16 - 43
Day06 part1     1.89 μs     0.35 μs     0.28 μs     nan         * 1 - 6
Day06 part2     0.81 μs     0.29 μs     0.10 μs     nan         * 2 - 7
Day07 part1     1.75 ms     0.73 ms     0.19 ms     nan         * 3 - 9
Day07 part2     1.79 ms     0.76 ms     0.19 ms     nan         * 4 - 9
Day08 parse     0.26 ms     0.11 ms     2.87 μs     nan         * 37 - 89
Day08 part1     0.78 ms     0.10 ms     0.03 ms     nan         * 3 - 25
Day08 part2     7.15 ms     1.92 ms     0.26 ms     0.08 ms     * 23 - 89
Day09 parse     0.31 ms     0.11 ms     0.07 ms     nan         * 1 - 4
Day09 part1     1.93 ms     0.21 ms     4.76 μs     nan         * 43 - 406
Day09 part2     2.00 ms     0.20 ms     4.56 μs     nan         * 44 - 439
Day09sym parse  0.54 ms     0.14 ms     0.07 ms     nan         * 1 - 7
Day09sym part1  0.17 ms     6.77 μs     0.19 μs     nan         * 36 - 924
Day09sym part2  0.23 ms     7.45 μs     0.19 μs     nan         * 39 - 1214
Day10 part1     2.62 ms     1.09 ms     0.13 ms     nan         * 8 - 19
Day10 part2     1.77 ms     0.74 ms     0.04 ms     0.02 ms     * 36 - 87
Day11 part1     1.93 ms     0.32 ms     0.04 ms     nan         * 8 - 50
Day11 part2     1.97 ms     0.14 ms     0.04 ms     nan         * 3 - 51
Day11hv part1   0.55 ms     0.18 ms     0.01 ms     nan         * 13 - 42
Day11hv part2   0.55 ms     0.18 ms     0.01 ms     nan         * 13 - 42
Day12 part1     8.99 ms     2.06 ms     0.64 ms     0.17 ms     * 12 - 53
Day12 part2     0.20 s      0.05 s      3.58 ms     0.74 ms     * 66 - 268
Day13 parse     1.56 ms     0.09 ms     0.03 ms     nan         * 2 - 50
Day13 part1     0.23 ms     9.38 μs     1.58 μs     nan         * 5 - 143
Day13 part2     0.30 ms     0.02 ms     2.81 μs     nan         * 8 - 106
Day14 parse     1.59 ms     0.33 ms     0.03 ms     nan         * 10 - 50
Day14 part1     0.40 ms     0.07 ms     7.93 μs     nan         * 8 - 49
Day14 part2     0.28 s      0.05 s      8.02 ms     nan         * 6 - 34
Day15 part1     1.38 ms     0.16 ms     0.02 ms     nan         * 7 - 67
Day15 part2     1.74 ms     0.47 ms     0.05 ms     nan         * 9 - 33
Day16 part1     3.29 ms     2.05 ms     0.05 ms     nan         * 39 - 63
Day16 part2     0.66 s      0.42 s      0.01 s      1.85 ms     * 226 - 356
Day17 part1     0.11 s      0.02 s      2.10 ms     nan         * 9 - 53
Day17 part2     0.29 s      0.06 s      4.50 ms     nan         * 13 - 63
Day18 part1     0.13 ms     0.03 ms     3.35 μs     nan         * 9 - 40
Day18 part2     0.23 ms     0.07 ms     3.59 μs     nan         * 19 - 64
Day19 parse     1.10 ms     0.43 ms     0.04 ms     nan         * 9 - 25
Day19 part1     0.15 ms     0.06 ms     6.54 μs     nan         * 9 - 23
Day19 part2     0.40 ms     0.07 ms     7.36 μs     nan         * 9 - 54
Day20 parse     0.05 ms     0.02 ms     2.20 μs     nan         * 7 - 22
Day20 part1     0.02 s      2.12 ms     0.25 ms     nan         * 8 - 71
Day20 part2     0.07 s      8.27 ms     0.45 ms     nan         * 18 - 157
Day21 part1     5.67 ms     1.61 ms     0.10 ms     nan         * 16 - 56
Day21 part2     0.23 s      0.06 s      2.10 ms     nan         * 26 - 110
Day22 parse     0.81 ms     0.82 ms     0.17 ms     nan         * 4 - 4
Day22 part1     0.03 s      5.44 ms     0.19 ms     nan         * 28 - 139
Day22 part2     6.56 ms     0.74 ms     0.11 ms     nan         * 6 - 59
Day23 part1     0.08 s      0.05 s      2.24 ms     nan         * 20 - 36
Day23 part2     5.85 s      1.79 s      0.19 s      0.04 s      * 43 - 140
Day24 parse     0.27 ms     0.12 ms     0.10 ms     nan         * 1 - 2
Day24 part1     0.01 s      3.40 ms     0.18 ms     nan         * 19 - 80
Day24 part2     8.70 μs     0.05 ms     0.29 μs     nan         * 29 - 170
Day25 parse     0.84 ms     0.49 ms     0.01 ms     nan         * 36 - 62
Day25 part1     1.75 ms     1.23 ms     0.08 ms     nan         * 14 - 21

Total        7871.34 ms  2523.14 ms   224.09 ms    66.32 ms     * 38 - 118
```
