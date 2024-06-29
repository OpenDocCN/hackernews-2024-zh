<!--yml

category: 未分类

date: 2024-05-27 15:04:44

-->

# [无 MMU 系统和 FDPIC | MaskRay](https://maskray.me/blog/2024-02-20-mmu-less-systems-and-fdpic)

> 来源：[https://maskray.me/blog/2024-02-20-mmu-less-systems-and-fdpic](https://maskray.me/blog/2024-02-20-mmu-less-systems-and-fdpic)

本文描述了没有内存管理单元（MMU）系统的 ABI 和工具链考虑因素。我们将重点讨论 FDPIC 和在开发中的 RISC-V FDPIC ABI，随着对这一主题的深入研究进行更新。

嵌入式系统通常缺乏 MMU，依赖于实时操作系统（RTOS）如 VxWorks 或特殊的 Linux 配置 (`CONFIG_MMU=n`)。在这些系统中，文本段和数据段之间的偏移量通常在编译时不可知。因此，通常会将专用寄存器设置到数据段的某个位置，并且可写数据相对于该寄存器访问。

为什么在编译时不知道偏移量？主要有两个原因。

首先，eXecute in Place (XIP) 指的是代码存放在 ROM 中，数据段复制到 RAM 中。因此，在编译时通常不知道文本段和数据段之间的偏移量。

其次，所有进程在没有 MMU 的情况下共享相同的地址空间。但仍希望这些进程共享文本段。因此需要一种机制，让代码找到其相应的数据。

## 编译器支持未知文本数据段偏移

### `-msep-data`

GCC 的 m68k 版本 [添加了 `-msep-data`](https://gcc.gnu.org/pipermail/gcc-patches/2003-September/113699.html) 在 2003-10。

> 为 uClinux 添加 -msep-data 和 -mid-shared-library 支持。这两个特殊的 PIC 变体允许在 ROM 文件系统中执行 Linux 应用程序，而无需在内存中加载额外的副本（XIP）。
> 
> 使用 -msep-data，全局数据的引用通过寄存器 A5 进行，该寄存器加载一个指向 RAM 中分配的数据/bss 段起始地址的指针。
> 
> -mid-shared-library 选项允许使用特殊的共享库版本，为每个进程分配一个独立的数据/bss 段，而无需在库和应用程序中重新定位代码。

`-msep-data` 只适用于 PIC，并将 `-fno-pic` 更新为 `-fPIE`。在此模式下，a5 是只读的，保存 `_GLOBAL_OFFSET_TABLE_` 的地址。如果不与 `-mid-shared-library` 一起使用，`-fPIC -msep-data` 是不必要的。只需使用 `-fPIE -msep-data`。

### `-mid-shared-library`

[此选项](https://gcc.gnu.org/onlinedocs/gcc/M680x0-Options.html#:~:text=mid) 在 GCC 的 m68k 版本中与 `-msep-data` 一起添加。文档中写道：

> 生成支持通过库 ID 方法的共享库的代码。这允许在没有虚拟内存管理的环境中进行执行和共享库。此选项隐含了 -fPIC。

`-mid-shared-library` 仅适用于PIC，并将 `-fno-pic` 更新为 `-fPIE`。使用 `-mid-shared-library -mshared-library-id=n` 编译源文件，函数将附加到库ID n。函数入口时，a5 指向一个数组，该数组将库ID映射到相应的GOT基地址。编译器生成 `move.l -(n+1)*4(%a5),%a5` 来获取实际的GOT基地址。然后使用a5来访问相应的数据段。

`gcc/config/bfin` 在2006年添加了 `-msep-data`。

### `-mno-pic-data-is-text-relative`

这个ARM选项类似于 `-msep-data` ，仅在 `-fpie`/`-fpic` 时才有意义。在2013年，从ARM [VxWorks RTP](https://gcc.gnu.org/pipermail/gcc-patches/2007-May/217111.html) 移植中概括出来的 `-mno-pic-data-is-text-relative` 被 [添加](https://inbox.sourceware.org/gcc-patches/000001cedf74%24bd1bf710%243753e530%24@arm.com/) ，假设文本段和数据段没有固定的位移。对于非VxWorks-RTP目标，`-mno-pic-data-is-text-relative` 暗示 [`-msingle-pic-base`](https://gcc.gnu.org/onlinedocs/gcc/ARM-Options.html#index-msingle-pic-base) ：

> 将用于PIC寻址的寄存器视为只读，而不是在每个函数的序言中加载它。运行时系统负责在执行开始前使用适当的值初始化此寄存器。

在位置无关数据模型中，r9 被用作静态基地址 (`arm_pic_register`) ，用于访问数据段。由于r9不会改变，动态链接似乎不支持，因为DSO需要不同的数据段。

GCC的s390x端口在2017年为kpatch [添加了 `-mno-pic-data-is-text-relative`](https://github.com/dynup/kpatch/commit/10002f5aa671de2878252aaa48f585457d39638a) （动态内核修补）。

### `-fropi` 和 `-frwpi`

Clang ARM 的 `-fropi` 和 `-frwpi` 是特殊的 `-fno-pic` 变体，仅用于静态链接。而常规的 `-fno-pic` 假定代码数据均使用绝对寻址，`-fropi` 和 `-frwpi` 通过根据特定的重定位假设强制执行相对寻址，为此增加了一些变化。这两个选项考虑了编译时未知的文本-数据段偏移量。

+   `-fropi` 假定代码和只读数据将在运行时重新定位，因此绝对寻址不适用。而是使用PC相对寻址。`.ARM.attributes` 部分包含 `Tag_ABI_PCS_RO_data: 1` ，类似 `-fpic`。

+   `-frwpi` 假定可写数据将在运行时重新定位，因此绝对寻址不适用。而是相对于静态基址寄存器访问可写数据。`.ARM.attributes` 部分包含 `Tag_ABI_PCS_RW_data: 2` 。

可以同时使用 `-fropi` 和 `-frwpi` 来要求代码和数据的相对寻址。与 `-fno-pic -frwpi` 相比，`-fno-pic -fropi -frwpi` 需要额外的一条指令来检索函数地址。

从语义上讲，我认为 `-fno-pic -fropic -frwpic` 和 `-fpie -mno-pic-data-is-text-relative` 在使用隐藏可见性声明时是相同的。在实践中，GCC `-fpie -mno-pic-data-is-text-relative` 使用GOT相对重定位（`R_ARM_GOT_BREL`），而不是MOVW/MOVT指令。

### `-mfdpic`

我们稍后将详细讨论此问题。

### 编译器选项总结

`-msep-data` 和 `-mno-pic-data-is-text-relative` 是相同的，依赖于 `-fpie/-fpic` 语义来强制文本段的相对寻址。`-fropi` 和 `-frwpi` 提供更精细的控制。您可以选择仅对文本段使用相对寻址（`-fropi`），仅对数据段使用（使用 `-frwpi`），或同时使用两者。

`-msep-data` 和 `-fropi -frwpi` 都不支持共享库。`-msep-data` 的变种 `-mid-shared-library` 提供了基于库ID的共享库，适用于某些情况，但不灵活。

* * *

现在，让我们回顾操作系统的支持。虽然我不是RTOS专家，让我们探讨Linux的可执行文件加载器，并查看它们如何处理没有MMU的情况。

## Linux二进制格式加载器

`fs/Kconfig.binfmt` 定义了一些加载器。

+   `BINFMT_ELF` 默认为y，并依赖于 `MMU`。

+   `BINFMT_ELF_FDPIC` 在未选择 `BINFMT_ELF` 时默认为y。一些架构支持 `BINFMT_ELF_FDPIC` 用于NOMMU。即使带有MMU，ARM也支持FDPIC。

+   `BINFMT_FLAT` 为几种架构提供支持。

因此，`BINFMT_ELF_FDPIC` 和 `BINFMT_FLAT` 可用于没有MMU的系统。`BINFMT_FLAT` 是一个非常古老的解决方案，不允许动态链接，而 `BINFMT_ELF_FDPIC` 支持动态链接。

顺便说一下，`BINFMT_AOUT` 在2022年被移除，曾支持alpha/arm/x86-32。

## 二进制平面格式

Linux的 `BINFMT_FLAT` 是指由[μClinux](https://en.wikipedia.org/wiki/%CE%9CClinux)使用的对象文件格式：Binary Flat format (BFLT)。[https://myembeddeddev.blogspot.com/2010/02/uclinux-flat-file-format.html](https://myembeddeddev.blogspot.com/2010/02/uclinux-flat-file-format.html) 有一个介绍。BFLT仅用于可执行文件格式，不用于可重定位文件。可执行文件通常使用 [elf2flt](https://github.com/uclinux-dev/elf2flt) 从ELF转换。`ld-elf2flt` 是一个ld包装器，当看到选项 `-elf2flt` 时调用 `elf2flt`。

Linux的 `BINFMT_FLAT` 支持版本2 (`OLD_FLAT_VERSION`) 和版本4。版本4支持在ROM中执行（XIP），其中代码驻留在ROM中，而数据段被复制到RAM中。因此，文本段和数据段之间的偏移通常在编译时未知。

Greg在2003年为 [`-mid-shared-library`](#mid-shared-library) 添加了[基于ID的共享库支持](https://git.kernel.org/pub/scm/linux/kernel/git/tglx/history.git/commit/?id=3d97dc2d349e6630bced9ced2ca7d0c7b52e49bc)，这在2022年4月被[移除](https://git.kernel.org/linus/70578ff3367dd4ad8f212a9b5c05cffadabf39a8)。该代码支持一个可执行文件和最多3个共享库。

用于共享库支持的工具似乎称为eXtended FLAT（XFLAT）。这是一种限制的共享库方案，不允许全局变量共享。引用[XFLAT FAQ](https://xflat.sourceforge.net/XFlatFAQ.html)：

> XFLAT提供了一种替代机制，用于通过插入在每个模块间函数调用之间的Thunk层来绑定和重定位函数。然而，没有GOT，无法绑定和重定位数据。简而言之，没有GOT，XFLAT无法支持程序和共享库模块之间的全局变量共享。

## FDPIC

FDPIC可以看作是一种扩展的`-mno-pic-data-is-text-relative`模式，它利用函数描述符支持动态链接的PIC寄存器更改。FDPIC可执行文件可以使用常规的Linux ELF加载器加载到MMU系统或使用`fs/binfmt_elf_fdpic.c`加载到无MMU系统中。自2002年以来，`fs/binfmt_elf_fdpic.c`一直[可用](https://git.kernel.org/pub/scm/linux/kernel/git/tglx/history.git/commit/?id=be3e0ada7b264efa8714b1ff6203c2c29fcc61c5)。它支持MMU和NOMMU配置，但不支持NOMMU模式下的`ET_EXEC`可执行文件。支持FDPIC的每个架构都定义了一个要由加载器检查的`EI_OSABI`值。

几种架构定义了FDPIC ABI。

这里是一个摘要。

可以共享的只读段通常称为“文本段”，而可写段是非共享的，通常称为“数据段”。函数和某些数据符号（`.rodata`）驻留在文本段中，而其他数据符号和GOT驻留在数据段中。特殊条目称为“规范函数描述符”也驻留在GOT中。

一个被调用破坏的寄存器作为FDPIC寄存器保留，用于访问数据段。进入函数时，FDPIC寄存器保存了`_GLOBAL_OFFSET_TABLE_`的地址。文本段可以使用PC相对寻址引用。包括GOT在内的数据段使用间接FDPIC寄存器相对寻址引用。稍后我们将看到，有时不确定一个不可抢占符号驻留在文本段还是数据段，此时必须使用带有FDPIC寄存器的GOT间接寻址。

函数调用称为外部调用，如果目标可能驻留在另一个模块中，该模块具有不同的数据段，因此需要不同的FDPIC寄存器值。因此，外部函数调用需要更新FDPIC寄存器以及改变程序计数器（PC）。如果调用者需要稍后引用数据段，则FDPIC寄存器可以溢出到堆栈插槽或调用保存的寄存器。FDPIC寄存器被调用破坏，以[允许外部尾调用](/blog/2021-09-19-all-about-procedure-linkage-table#got-setup-is-expensive-without-pc-relative-addressing)，并避免PLT保存该寄存器。

调用函数指针，包括调用PLT条目，也会设置FDPIC寄存器和PC。当获取函数的地址时，获取的是它的规范函数描述符的地址，而不是入口点的地址。描述符位于GOT中，包含指向函数入口点和其FDPIC寄存器值的指针。这两个GOT条目通过类型为`R_*_FUNCDESC_VALUE`（例如[`R_FRV_FUNCDESC_VALUE`](https://www.fsfla.org/~lxoliva/writeups/FR-V/FDPIC-ABI.txt#:~:text=The%20R_FRV_FUNCDESC_VALUE%20relocation%20is%20used%20to)）的动态重定位进行重新定位。

如果符号可抢占，代码序列会加载一个GOT条目。当符号是一个函数时，GOT条目会被动态重定位`R_*_FUNCDESC`重新定位，并将包含函数描述符地址。

让我们看一下获取函数和变量地址的示例。

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
__attribute__((visibility("hidden"))) void hidden_fun();
void fun();
__attribute__((visibility("hidden"))) extern int hidden_var;
extern int var;
__attribute__((visibility("hidden"))) const int ro_hidden_var = 42;
 void *addr_hidden_fun() { return hidden_fun; }
void *addr_fun() { return fun; }
void *addr_hidden_var() { return &hidden_var; }
void *addr_var() { return &var; }
const int *addr_ro_hidden_var() { return &ro_hidden_var; }
int read_hidden_var() { return hidden_var; }
int read_var() { return var; } 
```

|

### 在FDPIC中的函数访问

规范函数描述符存储在GOT中，它们的访问取决于引用的函数是否可抢占。

+   对于不可抢占的函数：描述符的地址通过将偏移添加到FDPIC寄存器直接计算而得。

+   对于可抢占的函数：首先加载一个GOT条目。这个条目通过`R_*_FUNCDESC`动态重定位进行重新定位，保存函数描述符的最终地址。

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
16

```

|

```
// arm-linux-gnueabihf-gcc -c -fpic -mfdpic -Wa,--fdpic
addr_hidden_fun: // non-preemptible function
 ldr     r0, .L3            // r0 = &.got[n] - FDPIC
 add     r0, r0, r9         // r0 = &.got[n]; the address of the canonical function descriptor
 ...
.L3:
// Linker resolves this to &.got[n] - FDPIC. .got[n], relocated by R_ARM_FUNCDESC_VALUE, is the canonical function descriptor.
 .word   hidden_fun(GOTOFFFUNCDESC) // R_ARM_GOTOFFFUNCDESC(hidden_fun)
 addr_fun: // preemptible function
 ldr     r3, .L6            // r3 = &.got[n] - FDPIC
 ldr     r0, [r9, r3]       // r0 = &.got[n]; the address of the canonical function descriptor
 ...
.L6:
// Linker resolves this to &.got[n] - FDPIC. .got[n], relocated by R_ARM_FUNCDESC, will contain the address of the canonical function descriptor.
 .word   fun(GOTFUNCDESC)   // R_ARM_GOTFUNCDESC(fun) 
```

|

遗憾的是，在链接DSO时，引用隐藏符号的`R_ARM_GOTOFFFUNCDESC`重定位会导致链接器错误。这个错误可能是由于生成的`R_ARM_FUNCDESC_VALUE`动态重定位需要一个动态符号引起的。虽然可以使用`STB_LOCAL STT_SECTION`动态符号来实现这一点，但GNU ld目前不支持这种方法。

|

```
1
2
3
4

```

|

```
% arm-linux-gnueabihf-gcc -fpic -mfdpic -O2 -Wa,--fdpic q.c -shared
/tmp/ccxpnij8.o: in function `addr_hidden_fun':
q.c:(.text+0x10): dangerous relocation: no dynamic index information available
collect2: error: ld returned 1 exit status

```

|

让我们尝试sh4。`sh4-linux-gnu-gcc -fpic -mfdpic -O2 q.c -shared -nostdlib`允许获取隐藏函数的地址，但不允许获取受保护函数的地址（我的[pending fix](https://sourceware.org/pipermail/binutils/2024-February/132519.html)）。

然后，让我们看看由函数地址和C++虚表初始化的全局变量。

|

```
1
2
3

```

|

```
struct A { virtual void foo(); };
void call(A *a) { a->foo(); }
auto *var_call = call;

```

|

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
16
17
18
19

```

|

```
// arm-linux-gnueabihf-g++ -c -fpic -mfdpic -Wa,--fdpic
 ldr     r3, [r0]           // load vtable
...
 ldr     r3, [r3]           // load vtable entry `.word _ZN1A3fooEv(FUNCDESC)`
 ldr     r9, [r3, #4]       // load FDPIC register value
 ldr     r3, [r3]           // load foo's entry point
 blx     r3
 .section        .data.rel,"aw"
var_call:
// Function descriptor address, relocated by R_ARM_FUNCDESC dynamic relocation
 .word   _Z4callP1A(FUNCDESC) // R_ARM_FUNCDESC
 .section        .data.rel.ro,"aw"
_ZTV1A:
 .word   0
 .word   _ZTI1A
// Function descriptor address, relocated by R_ARM_FUNCDESC dynamic relocation
 .word   _ZN1A3fooEv(FUNCDESC) // R_ARM_FUNCDESC 
```

|

TODO：`-fexperimental-relative-c++-abi-vtables`

### 在FDPIC中的数据访问

在两种情况下需要使用GOT间接寻址来访问数据符号：

+   可抢占符号：传统的GOT需求。

+   具有潜在数据段放置的不可抢占符号：这包括

    +   可写数据符号：这涵盖了本地声明的（`int var;`）和外部声明的（`extern int var;`）非const变量。

    +   潜在的动态初始化：`const A a; extern const int var;`

    +   某些保证的常量初始化：`extern constinit const int *const extern_const;`。常量初始化可能需要进行重定位，例如`constinit const int *const extern_const = &var;`

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
addr_hidden_var: // non-preemptible data with potential data segment placement
 ldr     r3, .L9             // r3 = &.got[n] - FDPIC
 ldr     r0, [r9, r3]
 ...
.L9:
// Linker resolves this to &.got[n] - FDPIC. .got[n], relocated by R_ARM_RELATIVE, will contain the address of hidden_var.
 .word   hidden_var(GOT)    // R_ARM_GOT_BREL
 addr_var: // preemptible data
 ldr     r3, .L12
 ldr     r0, [r9, r3]
 ...
.L12:
// Linker resolves this to &.got[n] - FDPIC. .got[n], relocated by R_ARM_GLOB_DAT, will contain the address of var.
 .word   var(GOT)           // R_ARM_GOT_BREL 
```

|

动态重定位`R_*_RELATIVE`和`R_*_GLOB_DAT`未使用标准的`+ load_base`语义。看起来musl fdpic不支持特殊的`R_*_RELATIVE`。

如果引用的数据符号是非可抢占的，并且保证在文本段中，我们可以使用基于 PC 的地址。 然而，在实践中，这种情况非常罕见。 最可能的用例如下：

|

```
1
2
3

```

|

```
const int ro_array[] = {1, 2, 3, 4}; 
 int read_ro_array_elem(int i) { return ro_array[i]; } 
```

|

GCC 的 ARM 端口似乎没有使用基于 PC 的地址。 我们可以尝试 GCC 的 SuperH 端口：

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
16
17
18

```

|

```
// sh4-linux-gnu-gcc -S -fpic -mfdpic -O2 q.c
addr_hidden_var: // non-preemptible data
 mov.l   .L12,r0
 rts
 add     r12,r0
.L13:
 .align 2
.L12:
 .long   hidden_var@GOTOFF
 addr_ro_hidden_var: // non-preemptible data
 mov.l   .L18,r0
 rts
 mov.l   @(r0,r12),r0
.L19:
 .align 2
.L18:
 .long   ro_hidden_var@GOT 
```

|

它优化了 `addr_hidden_var`，但没有优化 `addr_ro_hidden_var`。

### FDPIC 中的线程本地存储

*ARM FDPIC ABI* 定义了静态 TLS 重定位 `R_ARM_TLS_GD32_FDPIC, R_ARM_TLS_LDM32_FDPIC, R_ARM_TLS_IE32_FDPIC` 相对于 GOT，与其非-FDPIC 对应物相对于 PC 不同。

### FDPIC 中的 PLT

PLT 条目需要更新 FDPIC 寄存器以及更改程序计数器（PC）。 binutils 的 ARM 端口使用以下代码序列。

|

```
1
2
3
4
5
6

```

|

```
foo@plt:
ldr r12, .L1
add r12, r12, r9
ldr r9, [r12, #4]
ldr pc, [r12]
.L1: .word foo(GOTOFFFUNCDESC)

```

|

如果架构不允许两个字的原子更新，懒绑定就很难实现。 binutils 的 ARM 端口只是禁用了懒绑定。

让我们检查一个涉及连续函数调用的例子。

|

```
1
2
3
4

```

|

```
void f0(void);
void f1(void);
void f2(void);
void g() { f0(); f1(); f2(); }

```

|

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
g:
 push    {r4, lr}
 mov     r4, r9
 bl      f(PLT)
 mov     r9, r4
 bl      f(PLT)
 mov     r9, r4
 pop     {r4, lr}
 b       f(PLT)

```

|

如果 GCC 实现了 `-fno-plt`，可以使用以下代码序列：

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
16
17
18
19
20
21
22
23

```

|

```
g:
 push    {r4, lr}
 mov     r4, r9
 // call f0
 ldr     r12, .L0
 add     r12, r12, r4
 ldr     r9, [r12, #4]
 ldr     pc, [r12]
 // call f1
 ldr     r12, .L1
 add     r12, r12, r4
 ldr     r9, [r12, #4]
 ldr     pc, [r12]
 // tail call f2
 ldr     r12, .L2
 add     r12, r12, r4
 ldr     r9, [r12, #4]
 pop     {r4, lr}
 ldr     pc, [r12]
 .L0: .word f0(GOTOFFFUNCDESC)
.L1: .word f1(GOTOFFFUNCDESC)
.L2: .word f2(GOTOFFFUNCDESC) 
```

|

### 相对重定位和 `.rofixup` 部分

不像标准的`R_*_RELATIVE`重定位那样使用"*loc += load_base"语义，FDPIC 模式中的加载地址取决于包含段的情况。 下面从musl改编的代码展示了这种行为：

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
static void *laddr(const struct dso *p, size_t v) {
 size_t j=0;
 for (; v-p->loadmap->segs[j].p_vaddr >= p->loadmap->segs[j].p_memsz; j++);
 return (void *)(v - p->loadmap->segs[j].p_vaddr + p->loadmap->segs[j].addr);
}

```

|

在 `-pie` 和 `-shared` 链接中，存在动态部分，并且非可抢占函数和数据指针通过 `R_*_FUNCDESC_VALUE` 和 `R_*_RELATIVE` 动态重定位进行重定位。 对于 `-no-pie` 链接，情况各异：

+   动态链接：存在动态部分。 我们仍然可以使用动态重定位。

+   静态链接：没有动态部分。 在非-FDPIC 中，甚至没有重定位（除了 musl/uclibc-ng 不支持的 `R_*_IRELATIVE`）。

`ET_EXEC` 类型的 FDPIC 可执行文件面临独特的挑战：虽然文本段有固定地址，但数据段在链接时具有未知地址并且需要重定位。 为解决此问题，链接器创建了一个名为 `.rofixup` 的部分，在第一个 FDPIC ABI（FR-V）中引入，后来被其他 FDPIC ABI 接受。

`.rofixup` 拥有非可抢占函数和数据指针，具有 `R_*_RELATIVE` 语义。 `.rofixup` 的最后一个条目是特殊的，并保存了 `_GLOBAL_OFFSET_TABLE_` 的地址。 在 `-pie` 或 `-shared` 链接中，`.rofixup` 只有一个条目。 `__ROFIXUP_LIST__` 和 `__ROFIXUP_END__` 被定义为 `.rofixup` 的封装符号。

在运行时，加载器将 FDPIC 寄存器设置为重新定位后的 `_GLOBAL_OFFSET_TABLE_` 值，然后将控制传递到可执行文件的入口点。

这是一个例子：

|

```
1
2
3
4
5
6
7

```

|

```
.globl fun; fun: bx lr
.section .rodata,"a"
.globl var; var: .long 0
 .section .data.rel.ro,"aw"
.long fun(FUNCDESC)          // R_ARM_FUNCDESC_VALUE or two .rofixup entries
.long var                    // R_ARM_RELATIVE or one .rofixup entry 
```

|

## FDPIC 的反思

FDPIC 可以被视为：

+   一种扩展的 `-msep-data`/`-mno-pic-data-is-text-relative` 模式利用函数描述符来支持动态链接的 PIC 寄存器更改。

+   固定的[PPC64 ELFv1函数描述符ABI](/blog/2023-02-26-linker-notes-on-power-isa#ppc64-elfv1-function-descriptors)。但是，PPC64 ELFv1中`st_value`引用函数描述符的技巧优于现有的FDPIC ABI（sh、arm）。

FDPIC类似于PPC64 ELFv2 TOC，其中FDPIC寄存器由调用者而不是被调用者设置，避免了全局/局部入口和尾调用复杂性。

+   `-fno-pic -mfdpic`与隐藏可见性声明可以替代`-fno-pic -fropi -frwpi`，尽管跨函数调用时破坏的r9有轻微的开销。

+   `-fPIE -mfdpic`与隐藏可见性声明可以替代`-fPIE -msep-data`，尽管设置调用时破坏的FDPIC寄存器有轻微的开销。

`-mfdpic`在PC相对寻址昂贵的体系结构上通常生成比`-mno-fdpic`更小的代码。这包括：

+   sh4：完全缺乏PC相对寻址。

+   arm：需要使用带有`.word _GLOBAL_OFFSET_TABLE_ - (.LPIC0 + 4)`的`LDR`，这是昂贵的。

由于FDPIC甚至在具有MMU系统上也能有效工作，因此可能完全替代标准调用ABI的想法显得非常有趣。

`-mfdpic`启用了FDPIC代码生成。GCC的sh端口在2015年获得了[FDPIC支持](https://gcc.gnu.org/git/gitweb.cgi?p=gcc.git;h=1e44e857e05c165f6f01aeb56a7a43ee765bfc99)。`-mfpic`意味着`-fPIE`，因此`-fno-pic -mfdpic`和`-fPIE -mfdpic`具有相同的代码生成行为。`-fPIC -mfdpic`可能有不同的生成代码，因为它额外设置了`flag_shlib`。

cfgexpand pass调用`sh_get_fdpic_reg_initial_val`从伪寄存器中检索FDPIC寄存器值，并将伪寄存器注册为第一次调用时的硬寄存器r12。在ira（集成寄存器分配器）pass开始时，`allocate_initial_values`将伪寄存器初始化为函数入口点的硬寄存器r12。sh是唯一定义`TARGET_ALLOCATE_INITIAL_VALUE`的端口。

在GCC的ARM端口中，`-fno-pic -mfdpic`生成的代码无法工作。

此外，外部函数调用会保存和恢复r9。

gas的ARM端口需要`--fdpic`来组装与FDPIC相关的重定位类型。使用配置为`arm*-*-uclinuxfdpiceabi`目标的GCC利用`arm/uclinuxfdpiceabi.h`，在组装文件时将`-mfdpic`转换为`--fdpic`。对于其他目标，需要`-Wa,--fdpic`来组装输出。[[PATCH] arm: Support -mfdpic for more targets](https://gcc.gnu.org/pipermail/gcc-patches/2024-February/646436.html)将使`-Wa,--fdpic`不再需要。

`-mfdpic -mtls-dialect=gnu2`不受支持。ARM FDPIC ABI使用`ldr`来加载嵌入在文本段中的32位常量。偏移量用于实现GOT条目（规范函数描述符、规范函数描述符的地址或数据的地址）的地址。

您可以使用`--target=arm-unknown-uclinuxfdpiceabi`配置binutils以获取支持FDPIC仿真的BFD链接器。

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

```

|

```
% ~/Dev/binutils-gdb/out/arm-fdpic/ld/ld-new -V
GNU ld (GNU Binutils) 2.42.50.20240222
 Supported emulations:
 armelf_linux_eabi
 armelfb_linux_eabi
 armelf_linux_fdpiceabi
 armelfb_linux_fdpiceabi
% ~/Dev/binutils-gdb/out/arm-fdpic/ld/ld-new -m armelf_linux_fdpiceabi -shared a.o -o a.so

```

|

GNU ld的ARM端在引用隐藏函数符号时失败（[PR31408](https://sourceware.org/bugzilla/show_bug.cgi?id=31408)）。

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
% cat a.c
__attribute__((visibility("hidden"))) void fun_hidden();
void *fun_hidden_addr() { return fun_hidden; }
% ./bin/ld-new -m armelf_linux_fdpiceabi a.o
[1]    3819239 segmentation fault  ./bin/ld-new a.o
% ./bin/ld-new -m armelf_linux_fdpiceabi -shared a.o
./bin/ld-new: BFD (GNU Binutils) 2.42.50.20240224 internal error, aborting at ../../../bfd/elf32-arm.c:16466 in allocate_dynrelocs_for_symbol
 ./bin/ld-new: Please report this bug. 
```

|

在`-no-pie`模式下，某些需要`.rofixup`条目的非函数引用会导致段错误（[PR31407](https://sourceware.org/bugzilla/show_bug.cgi?id=31407)）。

由`R_ARM_FUNCDESC`引用的全局/弱非隐藏符号被不必要地导出（[PR31409](https://sourceware.org/bugzilla/show_bug.cgi?id=31407)）。

## RISC-V FDPIC

存在几个提议用于定义类似于FDPIC的ABI以在没有MMU的系统上工作。

毫无疑问，GP应作为FDPIC寄存器使用。

将常量加载到代码附近（类似于ARM）并不高效。相反，考虑使用两条指令的序列：

+   使用hi20和lo12指令生成相对于GP寄存器的偏移量。

+   使用`c.add a0, gp`来计算GOT条目的地址。

马杰克的代码序列支持通过间接GP相对寻址访问函数和数据。我们可以通过添加`R_RISCV_RELAX`来增强它，以启用链接器放松并提高性能。此外，为了与x86-64和AArch64上类似的标记保持一致（"gotpcrel"），让我们采用"gotgprel"标记。

|

```
1
2
3
4

```

|

```
.L0:
lui rX, %gotgprel_hi20(sym)  # R_RISCV_?(sym); R_RISCV_RELAX
c.add rX, gp                 # R_RISCV_?(.L0)
ld rY, %gotgprel_lo12(sym)   # R_RISCV_?(.L0); rY = address

```

|

对于数据访问，代码序列后跟随类似以下的指令：

函数描述符和数据具有不同的语义，需要两种重定位类型。斯蒂芬·奥里尔建议：

+   `R_RISCV_FUNCDESC_GOTGPREL_HI`：为规范函数描述符找到或创建两个GOT条目。

+   `R_RISCV_GOTGPREL_HI`：为符号找到或创建一个GOT，并返回相对于FDPIC寄存器的偏移量。

借鉴ARM FDPIC，需要两种额外的重定位类型用于TLS。这导致了一个4种类型的方案。

### RISC-V FDPIC：优化

处理性能问题至关重要。斯蒂芬建议采用“间接到相对优化和放松方案”：

+   `R_RISCV_PIC_ADD`：标记`c.add rX, gp`以启用优化

+   `R_RISCV_INTERMEDIATE_LOAD`：标记`ld rY, <gotgprel_lo12>(rX)`以启用优化

在特定条件下，间接GP相对寻址可以优化为直接GP相对寻址：

+   非可抢占函数

+   数据段中的非可抢占数据

|

```
1
2
3
4

```

|

```
# Indirect GP-relative to direct GP-relative
lui rX, <gprel_hi20>
c.add rX, gp
addi rY, rX, <gprel_lo12>

```

|

GOT间接寻址可以针对文本段中的非可抢占数据进行PC相对寻址优化。

|

```
1
2
3
4

```

|

```
# Indirect GP-relative to PC-relative
auipc rX, %pcrel_hi20(sym)
c.nop                               # deletable
addi rY, rX, %pcrel_lo12(sym)

```

|

GOT间接寻址可以针对文本段中的非可抢占数据进行优化为绝对寻址。

|

```
1
2
3
4

```

|

```
# Indirect GP-relative to absolute
lui rX, %hi20(sym)
c.nop                               # deletable
addi rY, rX, %lo12(sym)

```

|

这可以用于`SHN_ABS`和未解析的未定义弱符号。在`-no-pie`链接时，常规符号也有资格进行此优化。然而，链接器可能选择不实现这一点，因为增加的复杂性可能超过了好处。

### RISC-V FDPIC：线程本地存储

为了处理TLSDESC，我们引入了一个新的重定位类型：`R_RISCV_TLSDESC_GPREL_HI`。这种类型指示链接器查找或创建两个GOT条目，除非优化为本地执行或初始执行。组合的hi20和lo12偏移量计算到第一个GOT条目的GP相对偏移量。

|

```
1
2
3
4
5
6

```

|

```
label:
lui rX, %tlsdesc_gprel_hi(sym)      # R_RISCV_TLSDESC_GPREL_HI(sym); R_RISCV_RELAX
c.add a0, gp                        # R_RISCV_PIC_ADD(label)
ld rY, rX, %tlsdesc_load_lo(label)  # R_RISCV_TLSDESC_LOAD_LO12(label)
addi a0, rX, %tlsdesc_add_lo(label) # R_RISCV_TLSDESC_ADD_LO12(label)
jalr t0, rY, %tlsdesc_call(label)   # R_RISCV_TLSDESC_CALL(label)

```

|

现有的重定位类型，`R_RISCV_TLSDESC_LOAD_LO12`和`R_RISCV_TLSDESC_ADD_LO12`，被扩展以适用于`R_RISCV_TLSDESC_GPREL_HI`。

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

```

|

```
# TLSDESC to initial-exec optimization
lui a0, <gottpoff_gprel_hi20>
c.add a0, gp
ld a0, <gottpoff_gprel_lo12>(a0)
 # TLSDESC to local-exec optimization
lui a0, <tpoff_hi20>
addi a0, a0, <tpoff_lo12> 
```

|

对于初始执行TLS模型，我们需要一个新的伪指令，比如`la.tls.ie.fd rX, sym`。它扩展为：

|

```
1
2
3

```

|

```
lui rX, 0                    # R_RISCV_TLS_GOTGPREL_HI20(sym)
c.add rX, gp                 # R_RISCV_PIC_ADD(label)
ld rX, 0(rX)                 # R_RISCV_PIC_LO12_I(label)

```

|

斯蒂凡的方案将`R_RISCV_PIC_LO12_I`定义为`R_RISCV_PCREL_LO12_I`的别名。由于符号是相对于GP而不是PC的，避免在重定位类型名称中使用`PCREL`是合理的。

斯蒂凡的11种方案将`R_RISCV_PIC_ADDR_LO12_I`与`ld rX, 0(rX)`关联起来。我还没有弄清楚其理由。

### RISC-V FDPIC：`-fno-plt`

普通的[`-fno-plt`](/blog/2021-09-19-all-about-procedure-linkage-table#fno-plt)代码使用PC相对寻址加载`.got.plt`条目，并执行间接跳转。 FDPIC `-fno-plt`变体需要加载FDPIC寄存器和目标地址。

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
lui rX, 0
c.add rX, gp
ld gp, 8(rX)
ld rX, 0(rX)
c.jr rX

```

|

## libc实现支持FDPIC。

+   uclibc-ng支持AArch32、Blackfin和FR-V。

+   musl支持SuperH。
