<!--yml

分类：未分类

日期：2024-05-27 14:29:40

-->

# RISC-V 汇编器：算术 - 项目 F

> 来源：[https://projectf.io/posts/riscv-arithmetic/](https://projectf.io/posts/riscv-arithmetic/)
> 
> “RISC 架构将改变一切。” — [酸性烧伤](https://tvtropes.org/pmwiki/pmwiki.php/Film/Hackers)

在过去几年里，我们看到了 RISC-V CPU 设计的爆炸式增长，特别是在 FPGA 上。幸运的是，RISC-V 以其紧凑、易学的指令集非常适合汇编编程。本系列将帮助您学习和理解 32 位 RISC-V 指令（RV32）和 RISC-V ABI。第一部分将介绍加载立即数、加法和减法。我们还将涵盖符号扩展和伪指令。

与 @WillFlux 在 [Mastodon](https://mastodon.social/@WillFlux) 或 [Twitter](https://twitter.com/WillFlux) 分享您的想法。如果您喜欢我的工作，请 [赞助我](https://github.com/sponsors/WillGreen)。 🙏

## RISC-V 指令集

RISC-V 使用单独但一致的指令集处理不同大小的处理器：

+   **RV32** - 32 位 RISC-V，具有 32 个通用寄存器

+   **RV64** - 64 位 RISC-V，具有 32 个通用寄存器

+   **RV32E** - 精简版 32 位 RISC-V，具有 16 个通用寄存器

我将专注于 RV32，但其他指令集的工作方式类似。

RV32 的基本**整数**指令集为 RV32I（“I” 代表整数）。RV32I 包含了基本的 RISC-V 指令，如算术、内存访问和分支。

## CPU 寄存器

RV32I 具有 32 个通用寄存器：**x0** 到 **x31**。这些寄存器宽度为 32 位。

**x0** 被硬编码为 **0**（零）。您可以自由使用其他寄存器，但有一个应用程序二进制接口（ABI），使得程序员生活更加轻松，并允许来自不同开发者的代码互操作。我的示例使用临时寄存器 **t0-t6**。我将在我的关于 [函数](/posts/riscv-jump-function#rv32-abi-registers) 的文章中涵盖所有 ABI 寄存器，但现在您不需要担心它们。

介绍结束后，让我们开始学习这些指令。

使用加载立即指令 **li** 将常量值加载到寄存器中非常简单：

**rd** 是目标寄存器，**imm** 是 32 位立即数。

立即加载示例：

```
li t0, 2 li t1, 42 li t2, -1 li t3, 0x100000 li t4, 4100 li t5, 0xFACE 
```

*专业提示：十六进制字面量以 **0x** 为前缀。*

### 伪指令

RISC-V 寄存器宽度为 32 位。RISC-V 指令也是 32 位宽。指令需要空间来存放操作码和寄存器，因此无法容纳 32 位立即数。那么 **li** 如何处理呢？加载立即数不是一个 RISC-V 指令，而是一个**伪指令**。

伪指令由汇编器翻译成一个或多个真实指令。伪指令是语法糖，使汇编代码更易于编写和理解。在本系列中，我们将看到许多示例，您可以在[标准RISC-V伪指令](https://github.com/riscv-non-isa/riscv-asm-manual/blob/master/riscv-asm.md#-a-listing-of-standard-risc-v-pseudoinstructions)中查看完整列表。

如果现在不理解，不要担心。一旦我们讨论了RISC-V算术，我们将回到加载立即数的问题。

## 加法

RISC-V有两个加法指令：用于将寄存器相加和将立即数加到寄存器中。

```
add  rd, rs1, rs2  # rd = rs1 + rs2 addi rd, rs, imm   # rd = rs + imm 
```

在这里，**rd**是目标寄存器，**rs rs1 rs2**是源寄存器，**imm**是一个12位的立即数。

寄存器相加的示例：

```
li  t0, 2       # t0 =  2 li  t1, 46      # t1 = 46 li  t2, 10      # t2 = 10  add t3, t0, t0  # t3 =  2 +  2 =  4 add t4, t0, t1  # t4 =  2 + 46 = 48 add t4, t4, t2  # t4 = 48 + 10 = 58  ; t4 is a source and the destination 
```

添加立即数到寄存器的示例：

```
li   t0, 48       # t0 = 48  addi t1, t0,   1  # t1 = 48 + 1  = 49  ; increment addi t2, t0,  -1  # t2 = 48 - 1  = 47  ; decrement  addi t3, t0,  12  # t3 = 48 + 12 = 60 addi t4, t0, -12  # t4 = 48 - 12 = 36 addi t4, t4,  32  # t4 = 36 + 32 = 68  ; t4 is a source and destination 
```

**addi**可以在-2048到2047的范围内添加立即值。RISC-V没有增加或减少指令；**addi**也处理它们。

对于不满足于加法和减法的内容，**addi**也是两个常见伪指令背后的原因之一：

```
mv  rd, rs  # rd = rs nop         # no operation 
```

**mv**（移动）指令将一个寄存器的内容复制到另一个寄存器。**mv**实际上是带有立即值0的**addi**，因此什么也不添加。移动指令只能在寄存器之间复制，不能访问内存。

移动示例（请注意目的寄存器是给出的第一个寄存器）：

```
# these two examples generate the same machine code mv   t0, t1     # t0 = t1 addi t0, t1, 0  # t0 = t1 + 0 
```

**nop**指令使程序计数器前进，但不做其他改变。我们有一个标准的**nop**编码，以明确程序员的意图，并避免优化掉该指令。

### 符号扩展

RISC-V的立即数是带符号扩展的。最高有效位（MSB）填充剩余位，形成一个32位的值。一个12位的RISC-V立即数可以表示从-2048到2047的范围。

-2047十进制的符号扩展（MSB=1）：

`1000 0000 0000 -> 1111 1111 1111 1111 1111 1000 0000 0000`

1033十进制的符号扩展（MSB=0）：

`0100 0000 1001 -> 0000 0000 0000 0000 0000 0100 0000 1001`

使用有符号立即数，我们可以通过**addi**（参见上面的示例）进行加减运算，或在代码中向前或向后跳转（在[跳转和函数](/posts/riscv-jump-function/)中讨论）。

RISC-V的设计者做出了几个小决策，这些决策对指令集的简洁性和功能有着超常的影响：符号扩展即为其中之一。

## 减法

**sub**指令用于寄存器相减。对立即数的减法操作由上面的**addi**处理。

```
sub rd, rs1, rs2  # rd = rs1 - rs2 neg rd, rs        # rd = -rs (pseudoinstruction) 
```

**neg**伪指令对寄存器值取反：正数变负数，反之亦然。Negate仅使用一个源寄存器，因为它使用**sub**，并将零寄存器**x0**作为第一个源。

减法示例：

```
li  t0, 2       # t0 =  2 li  t1, 46      # t1 = 46 li  t2, 10      # t2 = 10  sub t3, t1, t0  # t3 = 46 -  2 = 44 sub t4, t0, t2  # t4 =  2 - 10 = -8  # these two examples generate the same machine code neg t5, t0      # t5 = -2 sub t6, x0, t0  # t6 = 0 - 2 = -2  ; The x0 register is always zero 
```

*专业提示：在RISC-V汇编语言中，目标寄存器先出现。*

> 想了解更多算术内容？查看我的关于RISC-V [乘法和除法](/posts/riscv-multiply-divide/) 的文章。

加载上位立即数将一个寄存器的前20位设置为立即值并将低12位清零。从另一个角度看**lui**，它将立即数左移12位。

```
lui rd, imm      # rd = imm << 12 
```

这在一些例子中最为明显：

```
lui t0, 1       # t0 = 1 << 12 = 0x1000 = 4096 lui t1, 3       # t1 = 3 << 12 = 0x3000 = 12288 lui t2, 0x100   # t2 = 0x100 << 12 = 0x100000 = 1048576 
```

**lui**接受范围在0x00000-0xFFFFF的立即数。

如果使用超出此范围的数字，包括负数，你的汇编器会报错。

```
lui t0, 0x100000  # 21-bit value is out of range! lui t1, -1        # negative value is out of range! 
```

如果超出范围，GNU汇编器会返回`Error: lui expression not in range 0..1048575`

现在我们已经了解了**addi**和**lui**，我们准备好解构**li**了。

加载上位立即数**lui**设置前20位并且加上立即数**addi**。这两条指令一起可以将一个32位立即数加载到一个寄存器中。

让我们看看汇编器如何处理我们之前的**li**例子：

```
li t0, 2 li t1, 42 li t2, -1 li t3, 0x100000 li t4, 4100 li t5, 0xFACE 
```

前三个例子适合12位，因此只需要**addi**：

```
addi t0, x0, 2 addi t1, x0, 42 addi t2, x0, -1 
```

记住，**x0**寄存器是硬连接到0（零）的。

**t3**（0x100000）的例子适合前20位，因此只需要**lui**：

**t4**（4100）的例子稍微大了一点超出了12位（2^(12) + 4 = 4100），因此我们需要**lui**然后**addi**：

```
lui  t4, 0x1 addi t4, t4, 4 
```

**t5**（0xFACE）的例子很棘手：

```
lui  t5, 0x10 addi t5, t5, -1330 
```

很明显，将0xACE加上0xF的答案是不行的，因为**addi**会对12位立即数进行符号扩展。查看0xACE的二进制表示，我们可以看到最高位是1：

`1010 1100 1110 -> 1111 1111 1111 1111 1111 1010 1100 1110`

符号扩展的结果为负数：-1330（-0x532）。

`0xF000 - 0x532 = 0xEACE`

要校正符号扩展，我们需要将**lui**的立即数加1：`0xF + 0x1 = 0x10`。

任何最高有效位为1的12位立即数都会遇到这个问题。

然而，解决方案很简单：使用**li**，汇编器会为你处理它。:)

## 接下来做什么？

如果你喜欢这篇文章，请[赞助我](https://github.com/sponsors/WillGreen)。赞助者帮助我为大家创建更多的FPGA和RISC-V项目，*并且*他们可以提前访问博客文章和源代码。🙏

*RISC-V Assembler*的第二部分涵盖了**[逻辑指令](/posts/riscv-logical)**。

本系列的其他部分包括：[加载存储](/posts/riscv-load-store/)和[分支集](/posts/riscv-branch-set/)。或查看所有我的[FPGA和RISC-V教程](/tutorials/)以及早期的[Macintosh历史](https://systemtalk.org/post/macintosh-history-8510/)系列。

### 致谢

感谢[jtruk](https://mastodon.social/@jtruk)和[Daniel Mangum](http://danielmangum.com)提供的建议和修正。

### 参考资料
