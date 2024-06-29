<!--yml

category: 未分类

date: 2024-05-27 13:25:53

-->

# LLVM比我更聪明 - sulami的博客

> 来源：[https://blog.sulami.xyz/posts/llvm-is-smarter-than-me/](https://blog.sulami.xyz/posts/llvm-is-smarter-than-me/)

我最近在阅读[现代硬件算法](https://en.algorithmica.org/hpc/)，特别是[SIMD章节](https://en.algorithmica.org/hpc/simd/)时，对自动向量化印象深刻。基本思想是现代CPU具有所谓的SIMD指令，允许一次对多个值应用操作，比逐个执行等效的标量指令快得多。现代编译器可以检测到某些编程模式，其中对标量值重复执行操作，并将这些操作分组以利用更快的指令。

该书主要使用C++示例，但我很好奇相同的模式在Rust中是否也有效，而无需手动注释任何内容。所以我打开了[编译器资源管理器](https://godbolt.org/)，并输入了我的第一个测试，看起来像这样：

```
#[no_mangle] fn sum() -> u32 {
 (0..1000).sum() } 
```

为了确保编译器安全使用SIMD指令，我使用了`-C opt-level=3 -C target-cpu=skylake`，告诉它目标CPU支持这些指令。但是，我没有得到SIMD指令，而是得到了这段汇编代码：

```
sum:
 mov     eax, 499500 ret 
```

结果证明编译器比我聪明，利用常量折叠在编译时直接返回了最终计算得到的值。为了避免发生这种优化，我们可以向函数传递参数，这样编译器无法知道最终结果：

```
#[no_mangle] fn sum(n: u32) -> u32 {
 (0..n).sum() } 
```

这生成了以下汇编代码：

```
sum:
 test    edi, edi je      .LBB0_1 lea     eax, [rdi - 1] lea     ecx, [rdi - 2] imul    rcx, rax shr     rcx lea     eax, [rdi + rcx] dec     eax ret .LBB0_1:
 xor     eax, eax ret 
```

在这里还没有SIMD指令，通常以字母`v`开头，比如[`vpaddd`](https://www.felixcloutier.com/x86/paddb:paddw:paddd:paddq)，用于并行添加整数。为了比较，我用C++写了等效的程序：

```
unsigned int sum(unsigned int n) {
  unsigned int rv = 0;
  for (unsigned int i = 0; i < n; i++) {
 rv += i; }  return rv; } 
```

编译器资源管理器默认使用GCC编译C++。我使用了等效的`-O3 -march=skylake`，果然，我得到了106行汇编代码（为了简洁起见已省略），包括所需的SIMD指令。然后我切换到Clang，这是基于LLVM的C/C++编译器，也被Rust使用，它生成了与Rust版本完全相同的汇编代码。这告诉我们，区别不在于语言，而在于编译器。

一项快速的比较基准测试显示，LLVM版本比GCC的向量化循环要快得多，特别是对于较大的`n`值，这在查看指令时并不令人意外，大约10条标量指令将击败数百条循环向量指令。

关键在于从零到n的连续整数的和有一个闭合形式解，这是我在仔细查看汇编代码后才意识到的：

$$\sum^n_{i=0}{i} = \frac{n (n+1)}{2}$$

显然，LLVM检测到这正是我们试图做的事情，因此它可以完全摒弃循环，并在一步之内直接计算结果，将`sum`从O(n)改变为O(1)。让我印象深刻。

最后，更贴近书中示例的复制，我设法让LLVM为我自动向量化一个循环，使用以下代码：

```
#[no_mangle] fn sum(arr: &[u32]) -> u32 {
 arr.iter().sum() } 
```

这有效果是因为LLVM对数组的内容一无所知，因此它无法推断出比实际将它们全部加起来更有效的计算方式。

我在这里的收获是，LLVM在生成优化代码方面比我预期的要好得多，至少对于简单的例子是如此。我很高兴我可以继续编写惯用代码，而不必感觉像是在以可读性为代价来交换性能。
