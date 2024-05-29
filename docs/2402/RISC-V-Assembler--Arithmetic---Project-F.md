<!--yml
category: 未分类
date: 2024-05-27 14:29:40
-->

# RISC-V Assembler: Arithmetic - Project F

> 来源：[https://projectf.io/posts/riscv-arithmetic/](https://projectf.io/posts/riscv-arithmetic/)

> “RISC architecture is going to change everything.” — [Acid Burn](https://tvtropes.org/pmwiki/pmwiki.php/Film/Hackers)

In the last few years, we’ve seen an explosion of RISC-V CPU designs, especially on FPGA. Thankfully, RISC-V is ideal for assembly programming with its compact, easy-to-learn instruction set. This series will help you learn and understand 32-bit RISC-V instructions (RV32) and the RISC-V ABI. The first part looks at load immediate, addition, and subtraction. We’ll also cover sign extension and pseudoinstructions.

Share your thoughts with @WillFlux on [Mastodon](https://mastodon.social/@WillFlux) or [Twitter](https://twitter.com/WillFlux). If you like what I do, [sponsor me](https://github.com/sponsors/WillGreen). 🙏

## RISC-V Instruction Sets

RISC-V handles processors of different sizes with separate but consistent instruction sets:

*   **RV32** - 32-bit RISC-V with 32 general-purpose registers
*   **RV64** - 64-bit RISC-V with 32 general-purpose registers
*   **RV32E** - Reduced 32-bit RISC-V with 16 general-purpose registers

I’ll be focusing on RV32, but the other instruction sets work in a similar way.

The base **integer** instruction set for RV32 is **RV32I** ("**I**" stands for integer). RV32I contains the essential RISC-V instructions, such as arithmetic, memory access, and branching.

## CPU Registers

RV32I has 32 general-purpose registers: **x0** to **x31**. These registers are 32 bits wide.

**x0** is hard-wired to **0** (zero). You can use the other registers as you see fit, but there is an **ABI** (application binary interface) to make life easier for programmers and allow code from different developers to interoperate. My examples use the temporary registers **t0-t6**. I’ll cover all the ABI registers in my post on [functions](/posts/riscv-jump-function#rv32-abi-registers), but you don’t need to worry about them for now.

With that briefest of introductions out of the way, let’s get started on the instructions.

It’s simple to load an immediate (constant) value into a register with load immediate **li**:

**rd** is the destination register, and **imm** is a 32-bit immediate.

Load immediate examples:

```
li t0, 2 li t1, 42 li t2, -1 li t3, 0x100000 li t4, 4100 li t5, 0xFACE 
```

*ProTip: Hexadecimal literals are prefixed with **0x**.*

### Pseudoinstructions

RISC-V registers are 32 bits wide. RISC-V instructions are 32 bits wide. An instruction needs room for an opcode and registers, so it can’t hold a 32-bit immediate. How does **li** manage it? Load immediate is not a RISC-V instruction but a **pseudoinstruction**.

Pseudoinstructions are translated into one or more real instructions by the assembler. Pseudoinstructions are syntactic sugar that makes assembly code easier to write *and* easier to understand. We’ll see many examples in this series, and you can see a complete list in [standard RISC-V pseudoinstructions](https://github.com/riscv-non-isa/riscv-asm-manual/blob/master/riscv-asm.md#-a-listing-of-standard-risc-v-pseudoinstructions).

Don’t worry if this doesn’t make sense right now. We’ll come back to load immediate once we’ve discussed RISC-V arithmetic.

## Addition

RISC-V has two add instructions: for adding registers together and adding an immediate to a register.

```
add  rd, rs1, rs2  # rd = rs1 + rs2 addi rd, rs, imm   # rd = rs + imm 
```

Where **rd** is the destination register, **rs rs1 rs2** are source registers, and **imm** is a 12-bit immediate.

Examples of adding registers:

```
li  t0, 2       # t0 =  2 li  t1, 46      # t1 = 46 li  t2, 10      # t2 = 10  add t3, t0, t0  # t3 =  2 +  2 =  4 add t4, t0, t1  # t4 =  2 + 46 = 48 add t4, t4, t2  # t4 = 48 + 10 = 58  ; t4 is a source and the destination 
```

Examples of adding an immediate to a register:

```
li   t0, 48       # t0 = 48  addi t1, t0,   1  # t1 = 48 + 1  = 49  ; increment addi t2, t0,  -1  # t2 = 48 - 1  = 47  ; decrement  addi t3, t0,  12  # t3 = 48 + 12 = 60 addi t4, t0, -12  # t4 = 48 - 12 = 36 addi t4, t4,  32  # t4 = 36 + 32 = 68  ; t4 is a source and destination 
```

**addi** can add an immediate value in the range -2048 to 2047\. RISC-V has no increment or decrement instructions; **addi** handles them too.

Not content with addition and subtraction, **addi** is also behind two common pseudoinstructions:

```
mv  rd, rs  # rd = rs nop         # no operation 
```

The **mv** (move) instruction copies one register to another. **mv** is **addi** with an immediate value of 0, so nothing is added. The move instruction can only copy between registers, it can’t access memory.

Move example (note the destination is the first register given):

```
# these two examples generate the same machine code mv   t0, t1     # t0 = t1 addi t0, t1, 0  # t0 = t1 + 0 
```

The **nop** instruction advances the program counter but makes no other changes. We have a standard **nop** encoding to make programmer intent clear and avoid the instruction being optimised away.

### Sign Extension

RISC-V immediates are sign-extended. The most significant bit (MSB) fills the remaining bits to create a 32-bit value. A 12-bit RISC-V immediate can represent -2048 to 2047 inclusive.

Sign extension of -2047 decimal (MSB=1):

`1000 0000 0000 -> 1111 1111 1111 1111 1111 1000 0000 0000`

Sign extension of 1033 decimal (MSB=0):

`0100 0000 1001 -> 0000 0000 0000 0000 0000 0100 0000 1001`

Using a signed immediate, we can add and subtract with **addi** (see examples above) or jump forwards or backwards in our code (discussed in [Jump and Function](/posts/riscv-jump-function/)).

RISC-V’s designers made several small decisions that have an oversized impact on the simplicity and power of the instruction set: sign extending immediates is one of them.

## Subtraction

The **sub** instruction subtracts registers. Subtracting an immediate is handled by **addi** (above).

```
sub rd, rs1, rs2  # rd = rs1 - rs2 neg rd, rs        # rd = -rs (pseudoinstruction) 
```

The **neg** pseudoinstruction negates a register value: positive numbers become negative and vice-versa. Negate only takes one source register because it uses **sub** with the zero register **x0** as the first source.

Subtraction examples:

```
li  t0, 2       # t0 =  2 li  t1, 46      # t1 = 46 li  t2, 10      # t2 = 10  sub t3, t1, t0  # t3 = 46 -  2 = 44 sub t4, t0, t2  # t4 =  2 - 10 = -8  # these two examples generate the same machine code neg t5, t0      # t5 = -2 sub t6, x0, t0  # t6 = 0 - 2 = -2  ; The x0 register is always zero 
```

*ProTip: the destination register comes first in RISC-V assembler.*

> Looking for more arithmetic? Check out my post on RISC-V [multiplication and division](/posts/riscv-multiply-divide/).

Load upper immediate sets the upper 20 bits of a register with an immediate value and zeros the lower 12 bits. Another way of looking at **lui** is that it left shifts the immediate by 12 bits.

```
lui rd, imm      # rd = imm << 12 
```

This is best seen with some examples:

```
lui t0, 1       # t0 = 1 << 12 = 0x1000 = 4096 lui t1, 3       # t1 = 3 << 12 = 0x3000 = 12288 lui t2, 0x100   # t2 = 0x100 << 12 = 0x100000 = 1048576 
```

**lui** accepts immediates in the range 0x00000-0xFFFFF.

Your assembler will error if you use numbers outside this range, including negative numbers.

```
lui t0, 0x100000  # 21-bit value is out of range! lui t1, -1        # negative value is out of range! 
```

If out of range, GNU assembler returns `Error: lui expression not in range 0..1048575`

Now we’ve met **addi** and **lui**, we’re ready to deconstruct **li**.

Load upper immediate **lui** sets the upper 20 bits and add immediate **addi** adds a 12-bit immediate. Together these two instructions can load a 32-bit immediate into a register.

Let’s see how the assembler does this for our earlier **li** examples:

```
li t0, 2 li t1, 42 li t2, -1 li t3, 0x100000 li t4, 4100 li t5, 0xFACE 
```

The first three examples fit into 12 bits, so they only need **addi**:

```
addi t0, x0, 2 addi t1, x0, 42 addi t2, x0, -1 
```

Remember that the **x0** register is hard-wired to 0 (zero).

The **t3** (0x100000) example fits in the upper 20 bits, so it only needs **lui**:

The **t4** (4100) example is a little too large for 12 bits (2^(12) + 4 = 4100), so we need **lui** then **addi**:

```
lui  t4, 0x1 addi t4, t4, 4 
```

The **t5** (0xFACE) example is a sneaky one:

```
lui  t5, 0x10 addi t5, t5, -1330 
```

The obvious answer of adding 0xACE to 0xF won’t work because **addi** sign extends the 12-bit immediate. Looking at 0xACE in binary, we see the most significant bit is 1:

`1010 1100 1110 -> 1111 1111 1111 1111 1111 1010 1100 1110`

The result of the sign extension is negative: -1330 (-0x532).

`0xF000 - 0x532 = 0xEACE`

To correct for the sign extension, we need to add 1 to the **lui** immediate: `0xF + 0x1 = 0x10`.

Any 12-bit immediate whose most significant bit is 1 will suffer from this issue.

However, the solution is simple: use **li** and the assembler will take care of it for you. :)

## What’s Next?

If you enjoyed this post, please [sponsor me](https://github.com/sponsors/WillGreen). Sponsors help me create more FPGA and RISC-V projects for everyone, *and* they get early access to blog posts and source code. 🙏

The second part of *RISC-V Assembler* covers **[Logical Instructions](/posts/riscv-logical)**.

Other parts of this series include: [Load Store](/posts/riscv-load-store/) and [Branch Set](/posts/riscv-branch-set/). Or check out all my [FPGA & RISC-V Tutorials](/tutorials/) and my series on early [Macintosh History](https://systemtalk.org/post/macintosh-history-8510/).

### Acknowledgements

Thanks to [jtruk](https://mastodon.social/@jtruk) and [Daniel Mangum](http://danielmangum.com) for suggestions and corrections.

### References