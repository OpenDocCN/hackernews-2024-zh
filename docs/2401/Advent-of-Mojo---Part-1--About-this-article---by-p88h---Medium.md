<!--yml
category: 未分类
date: 2024-05-27 14:33:05
-->

# Advent of Mojo : Part 1\. About this article | by p88h | Medium

> 来源：[https://medium.com/@p88h/advent-of-mojo-part-1-c1bcaa367fcb](https://medium.com/@p88h/advent-of-mojo-part-1-c1bcaa367fcb)

# About this article

This is part I of a series of articles on using Mojo in Advent of Code.

Go [here](/@p88h/advent-of-mojo-6d6d0d00761b) for the introduction and table of contents.

# Performance overview

Let’s start with the headliner. Is Mojo actually as fast as the company advertises it to be? Well, it depends. It’s plenty fast when you spend some time optimizing. After getting the hang of what works and what doesn’t work, I was able to achieve better performance in *every* days challenge (see benchmarks at the bottom), however, the gains were not very consistent and there are some caveats.

I haven’t seen anything close to the ‘68.000 x performance’ gains. Advent of Code simply doesn’t have problems of this scale. My guess is those gains would require massive parallelization, and while that’s reasonably simple in Mojo, due to small problem size, the overheads often just eat up any gains. And the overheads are not very predictable, too. There doesn’t seem to be a way (yet) to profile the code, so it’s a bit of guesswork.

# Vector code

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
        var acc = SIMD[DType.int32, 32](0)
        for i in range(0, self.rows, 32):
            acc = self.data.aligned_simd_load[32, 32](i).fma(mult, acc)
        return acc.reduce_add().to_int()
```

The highest improvement I was able to achieve was about 1000x vs Python — and this was similar to the use cases that Mojo seems to be built for — low level matrix operations. Writing SIMD code using fused multiply & addition, while working in Python-ish syntax is simpler than similar native solutions in Rust or C++. The core SIMD datatypes are well integrated and offer comprehensive, if basic, operations. If that particular solution was parallelisable (and much, much larger) then maybe that would yield 10.000 improvement or so, or even more, on some monster CPU with 100 cores.

There are other ways to optimize Python, though. While the average improvement over Python was > 100x (note that this does include parallelization), the same metric for PyPy3 was ‘only’ 38 x. And some of this solution could have used numpy, too.

# Quirky strings

There are a lot of quirks in Mojo that make this speed advantage go away pretty quickly, though. My code typically doesn’t use any dynamic memory allocation (which can be slow), opting for allocating large buffers; and simplifies input parsing, wherever possible, to string slicing (see more on Strings in the Usability section).

```
# Reasonably fast string split.. to perform adequately against s.split(' ')
fn parse[sep: Int8](inout self, skip_empty: Bool = True):
    var start: Int = 0
    for i in range(self.ptr_size):
        if self.contents[i] == sep:
            if i != start or not skip_empty:
                self.rows.push_back(StringSlice(self.contents.offset(start), i - start))
            start = i + 1
    if start < self.ptr_size or (start == self.ptr_size and not skip_empty):
        self.rows.push_back(StringSlice(self.contents.offset(start), self.ptr_size - start))
```

On that note, Strings themselves are slow. There is an equivalent (in spirit) of Rust’s *slice* which is *StringRef*, but it has very limited use. I implemented my own slices to get through AOC, but that’s ignoring a lot of everyday string processing needs, like UTF. When the parser above was implemented using *String.find()* it was 2–3x slower than PyPy, and close to Python.

Some other related operations are weirdly slow. For example, *ord(‘A’)* is something I would expect the compiler to automatically optimize away as constant. It’s not, but if you force it to be by using *alias* then it works well, just makes the code weird a bit.

# Simple threads

There are no thread primitives in Mojo, but there is higher-level parallelization. Go figure. The only way to make something parallel, really, is to break it out into an indexed worker function, like so:

```
let A = DTypePointer[DType.int32].aligned_alloc(16, 1024)
var S = Atomic[DType.int32](0)

@parameter
fn worker(id: Int):
    var M : Int32 = 0
    for i in range(256):
        M = max(M, A[id * 256 + i])
     S.max(M)        

fn parallel_max() -> Int32:
    parallelize[worker](len, 4)
    return S.value
```

Note: accessing mutable variables from threads is slow. But if those variables are not mutable (like the A pointer in the example), then it gets much faster. Sometimes. Also, note that the pointer being immutable still allows mutating the contents of the memory it’s pointing at.

There doesn’t seem to be any explicit synchronization (well, there are Atomics, at least) or thread communication — I considered implementing a lockless queue, but in the end there isn’t that much use of threads in AOC.

Benchmarks for simple stuff like this show that threads have significant operational overheads. Day 1 Part 1 takes 7 microseconds without threads, and ten times more when using threading. Day 2, however is able to complete in 40 microseconds with threading, and uses twice as many threads. It’s more predictable with longer work items, in milliseconds and above. I guess this is on par with what could be expected from a ‘hands-off’ approach, and even with very careful handling of low-level threads there are certain things you cannot optimize away.

# Summary

Overall, Mojo presents a **very** compelling performance proposal. I don’t have a lot of comparisons against C++, but for the one example I did, Mojo came really close. The programming model hints at the promise of even more potential improvements, like compiling the code to GPU target, and running that while pretending to be a single program, but it’s not really there yet and it’s hard to predict what compromises will be needed to actually achieve that.

Some of the gains in my solutions are perhaps more forced by Mojo’s lack of standard library, rather than better compilation techniques. I wouldn’t implement my own low-level string handling in Python, for example. When dealing with matrix operations, I would probably use NumPy, rather than coding it myself. I did that in Day 24 — the Python solution boils down to running a linear algebra solver (which I believe uses GESV from LAPACK, that factorizes the matrix using Gaussian elimination). My mojo code implements a Gaussian solver directly, which is a bit simpler — but that solver is targetted towards this _specific_ problem, I couldn’t solve multiple equations at the same time with it, and LAPACK can. The resulting code runs 30 times faster than Python (and this is one of those obscure cases where PyPy is slower than Python, too). A Gaussian solver in Python was even slower than NumPy, so I guess this is still a valid comparison, but I’m not sure how fast a _generic_ solution, or a NumPy equivalent for Mojo would be.

There are certainly some aspects of ML programming that could benefit from this though — perhaps custom tensor operators or activations, and Modular is probably focusing on those key performance pain points, and some of the tasks in AOC do support this hypothesis — meaning you could expect this would really improve the performance of whatever training or inference workloads you would want to run with this, assuming these workloads rely on switching back and forth between GPU/TPU and CPU for computations, and the CPU-side computations are mostly Python. It might be useful in some other areas, too, provided the language matures a bit.

Whether that is a benefit enough is a very tough question. It is a different language, and switching is a huge cost, especially given that to benefit from these performance gains, you do have to rewrite the code significantly. The promise is there, certainly, but for now other factors are the real blockers — usability and safety and stability, the topics of the next parts.

# Benchmarks

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