<!--yml

category: 未分类

date: 2024-05-27 13:35:53

-->

# Clang的`-O0`输出：分支位移和大小增加 | MaskRay

> 来源：[https://maskray.me/blog/2024-04-27-clang-o0-output-branch-displacement-and-size-increase](https://maskray.me/blog/2024-04-27-clang-o0-output-branch-displacement-and-size-increase)

tl;dr Clang 19将在`-O0`时移除`-mrelax-all`的默认设置，显著减少了x86的文本段大小。

## 跨依赖指令

在汇编语言中，一些带有立即操作数的指令可以用两种（或更多）不同大小的形式编码。在x86-64上，直接的JMP/JCC指令可以用2字节编码，带有8位相对偏移量，或者用6字节编码，带有32位相对偏移量。短跳转更为常见，因为它占用的空间更小。然而，当跳转的目标太远（超出8位相对偏移的范围）时，必须使用长跳转。

|

```
1
2
3
4

```

|

```
ja foo    # jump short if above, 77 <rel8>
ja foo    # jump near if above, 0f 87 <rel32>
.nops 126
foo: ret

```

|

1978年Thomas G. Szymanski的一篇论文（"*Assembling Code for Machines with Span-Dependent Instructions*"）使用术语“跨依赖指令”来指称这些具有短和长形式的指令。汇编器需要解决选择这些指令最佳大小的挑战，这通常被称为“分支位移问题”，因为分支是最常见的类型。理解Szymanski工作的好资源是[*Assembling Span-Dependent Instructions*](https://www.complang.tuwien.ac.at/anton/assembling-span-dependent.html)。

## 从小开始，逐渐增长

今天仍然流行的汇编器通常倾向于采用“从小开始，逐渐增长”的方法，通常比Szymanski的“从大开始，逐渐缩小”方法多需一次通过。这种方法通常会导致更小的代码，并且可以处理额外的复杂性，如对齐指令。

在LLVM中，[MC库](https://blog.llvm.org/2010/04/intro-to-llvm-mc-project.html)（机器码）负责汇编、反汇编和目标文件格式。在MC中，“汇编器放松”处理跨依赖指令。这与[链接器放松](/blog/2021-03-14-the-dark-side-of-riscv-linker-relaxation)是不同的。

Eli Bendersky在[2013年的博客文章](https://eli.thegreenplace.net/2013/01/03/assembler-relaxation)中详细解释并突出了一个有趣的行为：

> 例如，在使用`-O0`编译时，LLVM汇编器简单地在首次遇到的所有跳转上放松。这允许它将所有指令立即放入数据片段中，从而确保整体片段数量较少，因此汇编过程更快，消耗的内存更少。

当启用`-O0`并且使用嵌入式汇编器时（通常为默认情况），[clangDriver](/blog/2021-03-28-compiler-driver-and-cross-compilation) 会将`-mrelax-all`标志传递给LLVM MC库。 这将在`MCTargetOptions`中设置`MCRelaxAll`标志，指示汇编器可能仅在X86目标上以长格式（近距离）开始JMP和JCC指令。 ADD/SUB/CMP和非x86架构的其他指令保持不受影响。

## `-mrelax-all`权衡

以下是一个示例：

|

```
1
2
3
4
5

```

|

```
void foo(int a) {

 if (a) bar();
}

```

|

汇编代码（`clang -S`）如下:

|

```
1
2
3
4
5
6
7
8
9
10
11
12
13
14
15

```

|

```
foo:                                    # @foo
# %bb.0:                                # %entry
 pushq   %rbp
 movq    %rsp, %rbp
 subq    $16, %rsp
 movl    %edi, -4(%rbp)
 cmpl    $0, -4(%rbp)
 je      .LBB0_2
# %bb.1:                                # %if.then
 movb    $0, %al
 callq   bar@PLT
.LBB0_2:                                # %if.end
 addq    $16, %rsp
 popq    %rbp
 retq

```

|

JE指令汇编后将成为短跳转（8位相对偏移）或近跳转（32位相对偏移）。

|

```
1
2
3
4
5
6
7
8
9
10
11
12
13

```

|

```
# -mrelax-all
MCSection
 MCDataFragment: empty
 MCAlignFragment: alignment=4
 MCDataFragment: instructions including JE (jump near if equal, 6 bytes)
 # -mno-relax-all
MCSection
 MCDataFragment: empty
 MCAlignFragment: alignment=4
 MCDataFragment: instructions before JE (push; mov; sub; mov; cmp)
 MCRelaxableFragment: JE (jump short if equal, 2 bytes). This JE could be expanded, but not in this case.
 MCDataFragment: instructions after JE (mov; call; add; pop; ret) 
```

|

`-mrelax-all`对文本部分大小的影响非常显著，特别是当存在多个分支指令时。 在lld的x86-64 release构建中，`-mrelax-all`使`.text`部分大小增加了7.9%。这导致VM大小增加了5.4%，整体文件大小增加了4.6%。在lld的RISC-V rv64gc release构建中，`-mrelax-all`使`.text`部分大小增加了13.7%。这导致VM大小增加了9.0%，整体文件大小增加了7.2%。

Dean Michael Berris在2016年提议[移除默认情况下的`-mrelax-all` for `-O0`](https://reviews.llvm.org/D21830)，但未获通过。`-mrelax-all`导致与RISC-V的[条件分支变换](https://reviews.llvm.org/D108961)产生不良交互问题，最近Craig Topper移除了RISC-V的`-O0`中的`-mrelax-all`（https://github.com/llvm/llvm-project/pull/88538）。

实际上，这表明了在2023年时，条件分支变换补丁引入了大小回归。

|

```
1
2
3
4
5
6
7
8
9
10
11
12
13

```

|

```
blt a1, a2, .Lfoo
beqz a1, .Lfoo
.Lfoo:
 # llvm-mc -filetype=obj -triple=riscv64 -mattr=+relax,+c -mc-relax-all
blt a1, a2, .Lfoo       # R_RISCV_BRANCH(.Lfoo), range: +-4KiB
c.beqz a1, .Lfoo        # R_RISCV_BRANCH(.Lfoo)
.Lfoo:
# llvm-mc -filetype=obj -triple=riscv64 -mattr=+relax,+c
bge a1, a2, .+8
jal zero, .Lfoo         # R_RISCV_JAL(.Lfoo), range: +-2MiB
c.bneq a1, .+8
jal zero, .Lfoo         # R_RISCV_JAL(.Lfoo) 
```

|

虽然`-mrelax-all`在过去可能带来了轻微的编译时间好处，但如今收益微乎其微。使用Clang的阶段2构建进行基准测试显示，`-mrelax-all`和`-mno-relax-all`之间没有可测量的差异。 在llvm-compile-time-tracker上运行llvm-test-suite/CTMark基准测试，编译时间实际上[略微增加](https://llvm-compile-time-tracker.com/compare.php?from=ef2ca97f48f1aee1483f0c29de5ba52979bec454&to=18376810f359dbd39d2a0aa0ddfc0f7f50eac199&stat=instructions%3Au)了0.62%，而文本部分大小则[减小](https://llvm-compile-time-tracker.com/compare.php?from=ef2ca97f48f1aee1483f0c29de5ba52979bec454&to=18376810f359dbd39d2a0aa0ddfc0f7f50eac199&stat=size-text)了4.44%。

惊人的是，在不同优化级别下进行汇编的差异是非常令人惊讶的。即使在`-O0`上，GCC/GNU汇编器也没有展示类似的JMP/JCC指令扩展。

这些论据加强了将`-mrelax-all`作为`-O0`的默认值。[我的补丁](https://github.com/llvm/llvm-project/pull/90013)已经取得进展，并将包含在下一个主要版本，LLVM 19.1中。我还改变了Clang，使其对汇编输入尊重 `-mrelax-all`：`clang -c --target=x86_64 -mrelax-all a.s`

## 了解编译时间差异

我曾研究过一个声名狼藉的巨大文件，`llvm/lib/Target/X86/X86ISelLowering.cpp`。

**片段计数**：生成的汇编片段数量存在显著差异：

+   `-mrelax-all`: 89633

+   `-mno-relax-all`: 143852

使用`-mrelax-all`后，`MCRelaxableFragment`的数量大幅减少（在构建Clang时减少到零）。这种减少很可能影响了编译时间的差异。

**固定点迭代**：`-mrelax-all`确保固定点迭代算法（几乎总是）在一次迭代中收敛。相比之下，使用`-mno-relax-all`时，大约有6%的部分需要额外的迭代。然而，这种差异可能不是影响编译时间的主要因素。

|

```
1
2
3
4
5
6
7
8
9

```

|

```
// -mrelax-all
1: 13919
2: 1
 // -mno-relax-all
1: 13103
2: 793
3: 23
4: 1 
```

|

## 为什么没有人抱怨代码大小的增加？

因为人们通常较少关注`-O0`代码的大小。

`-O0`经常与`-g`一起使用以包含调试信息。这些调试信息可能会掩盖`-mrelax-all`导致的大小增加。 (`-O1`或更高级别的优化会牺牲一些调试能力。)

此外，并非所有项目都能成功地使用`-O0`优化。这通常是由于程序非常庞大或者必须进行内联的行为等问题所致。

若要讨论ELF可重定位文件中的大小减少想法，请查阅我的博文关于[轻量ELF](/blog/2024-04-01-light-elf-exploring-potential-size-reduction)。

* * *

您可能还对[我的笔记](/blog/2023-05-08-assemblers)关于GNU汇编器和LLVM集成汇编器感兴趣。
