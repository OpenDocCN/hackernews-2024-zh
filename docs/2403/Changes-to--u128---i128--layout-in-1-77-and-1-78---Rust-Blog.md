<!--yml

类别：未分类

日期：2024-05-29 12:50:16

-->

# 变更到 `u128`/`i128` 布局在 1.77 和 1.78 版本 | Rust 博客

> 来源：[https://blog.rust-lang.org/2024/03/30/i128-layout-update.html](https://blog.rust-lang.org/2024/03/30/i128-layout-update.html)

Rust 长期以来与 C 在 x86-32 和 x86-64 架构上关于 128 位整数对齐的不一致性问题。这个问题最近已经得到解决，但解决方案带来了一些值得注意的影响。

作为用户，你大多数情况下无需担心这些变更，除非你是：

1.  假设 `i128`/`u128` 的对齐而不使用 `align_of`

1.  忽略 `improper_ctypes*` 的 lints，并在 FFI 中使用这些类型

除了 x86-32 和 x86-64 架构外，其他架构没有变化。如果你的代码大量使用 128 位整数，你可能会注意到运行时性能提高，但可能会增加额外的内存使用成本。

这篇文章记录了问题的本质，解决方案的变更以及变更的预期影响。如果你已经熟悉这个问题，只想找兼容性矩阵，请跳到 [兼容性](#compatibility) 部分。

# 背景

数据类型有两个内在值，它们与它们在内存中的布局有关；大小和对齐。类型的大小是它在内存中占用的空间量，它的对齐指定了它允许放置在哪些地址上。

像原始类型这样的简单类型的大小通常是明确的，即它们代表的数据的确切大小，没有填充（未使用的空间）。例如，一个 `i64` 的大小始终为 64 位或 8 字节。

然而，对齐可能会有所不同。一个 8 字节整数 *可能* 存储在任何内存地址上（1 字节对齐），但大多数 64 位计算机如果存储在 8 的倍数地址上会获得最佳性能（8 字节对齐）。所以，就像其他语言一样，Rust 中的原始类型默认具有最高效的对齐方式。这种效果在创建复合类型时可以看到（[playground 链接](https://play.rust-lang.org/?version=beta&mode=debug&edition=2021&gist=52f349bdea92bf724bc453f37dbd32ea)）：

```
use core::mem::{align_of, offset_of};

#[repr(C)]
struct Foo {
    a: u8,  // 1-byte aligned
    b: u16, // 2-byte aligned
}

#[repr(C)]
struct Bar {
    a: u8,  // 1-byte aligned
    b: u64, // 8-byte aligned
}

println!("Offset of b (u16) in Foo: {}", offset_of!(Foo, b));
println!("Alignment of Foo: {}", align_of::<Foo>());
println!("Offset of b (u64) in Bar: {}", offset_of!(Bar, b));
println!("Alignment of Bar: {}", align_of::<Bar>()); 
```

输出：

```
Offset of b (u16) in Foo: 2
Alignment of Foo: 2
Offset of b (u64) in Bar: 8
Alignment of Bar: 8 
```

我们看到在结构体内部，类型始终会被放置在其偏移量是其对齐倍数的位置上 - 即使这意味着未使用的空间（当未使用 `repr(C)` 时，Rust 默认会尽量减少这种情况）。

这些数字并非随意的；应用二进制接口（ABI）定义了它们应该是什么。在 x86-64 [psABI](https://www.uclibc.org/docs/psABI-x86_64.pdf)（处理器特定 ABI）中的 System V（Unix 和 Linux）中，*图 3.1：标量类型* 告诉我们原始类型应该如何表示：

| C 类型 | Rust 对等类型 | `sizeof` | 对齐（字节） |
| --- | --- | --- | --- |
| `char` | `i8` | 1 | 1 |
| `unsigned char` | `u8` | 1 | 1 |
| `short` | `i16` | 2 | 2 |
| **`unsigned short`** | **`u16`** | **2** | **2** |
| `long` | `i64` | 8 | 8 |
| **`unsigned long`** | **`u64`** | **8** | **8** |

ABI仅指定了C类型，但Rust对于兼容性和性能优势遵循相同的定义。

# 不正确的对齐问题

如果两个实现在数据类型的对齐上存在分歧，它们将无法可靠地共享包含该类型的数据。Rust对于128位类型的对齐不一致：

```
println!("alignment of i128: {}", align_of::<i128>()); 
```

```
// rustc 1.76.0
alignment of i128: 8 
```

```
printf("alignment of __int128: %zu\n", _Alignof(__int128)); 
```

```
// gcc 13.2
alignment of __int128: 16

// clang 17.0.1
alignment of __int128: 16 
```

（[Godbolt链接](https://godbolt.org/z/h94Ge1vMW)）回顾一下[psABI](https://www.uclibc.org/docs/psABI-x86_64.pdf)，我们可以看到Rust在这里的对齐是错误的：

| C类型 | Rust等价类型 | `sizeof` | 对齐（字节） |
| --- | --- | --- | --- |
| `__int128` | `i128` | 16 | 16 |
| `unsigned __int128` | `u128` | 16 | 16 |

结果表明，这并不是因为Rust主动做错了什么：基本类型的布局来自于LLVM代码生成后端，它由Rust和Clang等语言使用，并且将`i128`的对齐硬编码为8字节。

Clang仅因为一个变通方法而使用了正确的对齐，其中在将类型传递给LLVM之前手动设置了16字节的对齐。这修复了布局问题，但也引发了一些其他小问题。Rust没有进行这样的手动调整，因此在[https://github.com/rust-lang/rust/issues/54341](https://github.com/rust-lang/rust/issues/54341)上报告了问题。

# 调用约定问题

还有一个额外的问题：LLVM在将128位整数作为函数参数传递时并不总是做正确的事情。这在LLVM中是一个已知问题，在其[GitHub问题](https://github.com/llvm/llvm-project/issues/41784)中被发现与Rust的相关性。

在调用函数时，参数被传递到寄存器（CPU内的特殊存储位置），直到没有更多的槽位，然后它们被“spill”到堆栈（程序的内存）。ABI也告诉我们在这里要做什么，在第*3.2.3参数传递*部分：

> 类型为`__int128`的参数提供了与INTEGER相同的操作，但它们不能放入一个通用寄存器，而是需要两个寄存器。为了分类目的，`__int128`被视为：
> 
> ```
> typedef struct {
>     long low, high;
> } __int128; 
> ```
> 
> 除了存储在内存中的类型为`__int128`的参数必须对齐在16字节边界上。

我们可以通过手动实现调用约定来尝试这个。在下面的C示例中，内联汇编用于调用`foo(0xaf, val, val, val)`，其中`val`为`0x11223344556677889900aabbccddeeff`。

x86-64使用寄存器`rdi`、`rsi`、`rdx`、`rcx`、`r8`和`r9`按顺序传递函数参数（你猜对了，这也是ABI中的一部分）。每个寄存器可以容纳一个字（64位），超出部分将被`push`到堆栈中。

```
/* full example at <https://godbolt.org/z/5c8cb5cxs> */

/* to see the issue, we need a padding value to "mess up" argument alignment */
void foo(char pad, __int128 a, __int128 b, __int128 c) {
    printf("%#x\n", pad & 0xff);
    print_i128(a);
    print_i128(b);
    print_i128(c);
}

int main() {
    asm(
        /* load arguments that fit in registers */
        "movl    $0xaf, %edi \n\t"                /* 1st slot (edi): padding char (`edi` is the
                                                   * same as `rdi`, just a smaller access size) */
        "movq    $0x9900aabbccddeeff, %rsi \n\t"  /* 2rd slot (rsi): lower half of `a` */
        "movq    $0x1122334455667788, %rdx \n\t"  /* 3nd slot (rdx): upper half of `a` */
        "movq    $0x9900aabbccddeeff, %rcx \n\t"  /* 4th slot (rcx): lower half of `b` */
        "movq    $0x1122334455667788, %r8  \n\t"  /* 5th slot (r8):  upper half of `b` */
        "movq    $0xdeadbeef4c0ffee0, %r9  \n\t"  /* 6th slot (r9):  should be unused, but
                                                   * let's trick clang! */

        /* reuse our stored registers to load the stack */
        "pushq   %rdx \n\t"                       /* upper half of `c` gets passed on the stack */
        "pushq   %rsi \n\t"                       /* lower half of `c` gets passed on the stack */

        "call    foo \n\t"                        /* call the function */
        "addq    $16, %rsp \n\t"                  /* reset the stack */
    );
} 
```

使用GCC运行上述内容将打印出以下预期的输出：

```
0xaf
0x11223344556677889900aabbccddeeff
0x11223344556677889900aabbccddeeff
0x11223344556677889900aabbccddeeff 
```

但在Clang 17运行时，会打印：

```
0xaf
0x11223344556677889900aabbccddeeff
0x11223344556677889900aabbccddeeff
0x9900aabbccddeeffdeadbeef4c0ffee0
//^^^^^^^^^^^^^^^^ this should be the lower half
//                ^^^^^^^^^^^^^^^^ look familiar? 
```

惊喜！

这说明了第二个问题：LLVM希望在可能的情况下将`i128`一半存入寄存器，一半存入栈，但是这不符合ABI的规定。

由于这种行为来自LLVM并且没有合理的解决方法，这在Clang和Rust中都是一个问题。

# 解决方案

解决这些问题是许多人长时间努力的结果，始于2017年的编译器团队成员Simonas Kazlauskas的一次补丁：[D28990](https://reviews.llvm.org/D28990)。 不幸的是，这次被撤销了。 后来由LLVM贡献者Harald van Dijk在2023年10月实现的[D86310](https://reviews.llvm.org/D86310)再次尝试了这一点，这是最终实现的版本。

大约在同一时间，Nikita Popov通过[D158169](https://reviews.llvm.org/D158169)修复了调用规约的问题。 这些改变都已经进入了LLVM 18，这意味着所有相关的ABI问题将在使用这一版本的Clang和Rust（Clang 18和Rust 1.78在使用捆绑的LLVM时）中得到解决。

但是，`rustc`也可以使用系统上安装的LLVM版本，而不是打包的版本，后者可能较老。为了减少出现相同`rustc`版本的对齐不一致而导致问题的几率， [提议](https://github.com/rust-lang/compiler-team/issues/683)引入了手动修正对齐方式的方法，就像Clang一直在做的一样。 这是由Matthew Maurer在[#116672](https://github.com/rust-lang/rust/pull/116672/)中实现的。

自这些改变以来，Rust现在产生了正确的对齐方式：

```
println!("alignment of i128: {}", align_of::<i128>()); 
```

```
// rustc 1.77.0
alignment of i128: 16 
```

如上所述，ABI规定数据类型的对齐方式的部分原因是因为在该架构上更有效率。我们实际上已经亲眼见证了这一点：手动调整对齐方式后，[初始性能运行](https://github.com/rust-lang/rust/pull/116672/#issuecomment-1858600381)显示出对编译器性能的显著改进（这主要依赖于对整数文字进行处理的128位整数）。增加对齐方式的缺点是复合类型不一定总是能完美地放在一起，导致使用增加。不幸的是，这意味着一些性能提升需要牺牲以避免增加内存占用。

# 兼容性

最重要的问题是这些修复导致的兼容性如何改变。简而言之，使用LLVM 18的Rust的`i128`和`u128`（从1.78版本开始的默认版本）将与任何版本的GCC完全兼容，并且与Clang 18及以上版本（2024年3月发布）完全兼容。其他所有组合都有一些不兼容的情况，总结在下表中：

| 编译器 1 | 编译器 2 | 状态 |
| --- | --- | --- |
| Rust ≥ 1.78 with bundled LLVM (18) | GCC（任意版本） | 全兼容 |
| Rust ≥ 1.78 with bundled LLVM (18) | Clang ≥ 18 | 全兼容 |
| Rust ≥ 1.77 with LLVM ≥ 18 | GCC（任意版本） | 全兼容 |
| Rust ≥ 1.77 with LLVM ≥ 18 | Clang ≥ 18 | 全兼容 |
| Rust ≥ 1.77 with LLVM ≥ 18 | Clang < 18 | 存储兼容，存在调用错误 |
| Rust ≥ 1.77 with LLVM < 18 | GCC（任意版本） | 存储兼容，存在调用错误 |
| Rust ≥ 1.77 with LLVM < 18 | Clang（任意版本） | 存储兼容，存在调用错误 |
| Rust < 1.77 | GCC（任意版本） | 不兼容 |
| Rust < 1.77 | Clang（任意版本） | 不兼容 |
| GCC（任意版本） | Clang ≥ 18 | 完全兼容 |
| GCC（任意版本） | Clang < 18 | 存储兼容，存在调用错误 |

# 影响与未来步骤

正如介绍中所述，除非您已经对这些类型进行了某些可疑操作，否则大多数用户不会注意到此更改的任何影响。

从Rust 1.77开始，使用128位整数进行FFI实验将是相当安全的，随着1.78中LLVM更新的确定性更加明显。有关[持续讨论](https://github.com/rust-lang/lang-team/issues/255)计划在即将发布的版本中取消警告，但我们希望谨慎行事，避免对那些使用较旧LLVM构建的Rust编译器的用户引入潜在破坏。
