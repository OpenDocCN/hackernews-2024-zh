<!--yml

分类：未分类

日期：2024-05-27 14:50:34

-->

# 探索对象文件格式 | MaskRay

> 来源：[`maskray.me/blog/2024-01-14-exploring-object-file-formats`](https://maskray.me/blog/2024-01-14-exploring-object-file-formats)

我与 LLVM 项目的旅程始于对 lld 和二进制工具的深入研究。无数个小时都花在了解对象文件格式的复杂性和塑造 LLVM 相关组件上。尽管我的兴趣已经扩展，但对象文件格式仍然是我个人着迷的领域，经常让我卷入 LLVM 内潜在变化的讨论中。

本文比较了几种显著的对象文件格式，借鉴了我的经验和见解。

每种格式的核心都是对象的基本组件，如符号、节和重定位的表示。对于每个控制结构，我们将从 ELF 开始，这是一种广泛使用的格式，然后再进入其他显著格式的领域。

## 对象文件格式的历史

在深入技术方面之前，我将分享一些关于我的考古之旅的笔记。

### a.out

a.out 格式是为 PDP-11 设计的，并出现在 Unix 的第一个版本中。数量是 16 位的，但可以自然地扩展为 32 位或 64 位。

在 1990 年夏季 USENIX 会议的论文《ELF: An Object File to Mitigate Mischievous Misoneism》中，James Q. Arnold 提供了一些描述。

> 对于 32 位机器，a.out 格式在几个方面进行了扩展。最明显的是，16 位的量被扩展为 32 位值。符号表发生了变化，允许无限长度的名称。重定位条目也发生了显著变化。更大的程序和不同的重定位约定使得需要将重定位条目与显式地址关联起来，而不是依赖于程序段和重定位记录之间的隐式对应关系。

许多 Unix 和类 Unix 操作系统，包括 SunOS、HP-UX、BSD 和 Linux，在转换为 ELF 之前都使用了 a.out。

最显著的扩展是动态共享库支持。（此功能与静态共享库不同，静态共享库需要在地址空间中的每个共享库都有一个固定的地址。）有两种风格：

[FreeBSD a.out(5)](https://man.freebsd.org/cgi/man.cgi?a.out(5))提供了很好的描述。

如果你关注最近几年的 Linux 内核新闻，你会发现在 2022 年 Linux 最终[移除了对 a.out 的支持](https://git.kernel.org/linus/987f20a9dcce3989e48d87cff3952c095c994445)时有所讨论。

### COFF

a.out 支持三个固定的可加载节 TEXT、DATA 和 BSS，这太过限制性了。COFF 引入了自定义节支持，并允许最多 32767 节。ELF 论文中有一些评论：

> COFF（通用对象文件格式）主要设计用于支持电子交换系统（电话网络）。其特点是多个部分（文本、数据、未初始化内存、保留内存、叠加、等等）、对多个目标处理器的某些支持、为 C 语言量身定制的符号表和重定位的定义结构，以及调试信息。

根据 System V Release 2 中的 `scnhdr.h`，COFF 最迟于 1982 年设计。然后，System V Release 3 采用了 COFF，这激发了很多后续工作。

+   Windows 扩展了 COFF 至 [可移植可执行文件（PE）格式](https://learn.microsoft.com/en-us/windows/win32/debug/pe-format)。Symbian OS 在 9 之前也使用了 PE。

+   德州仪器为其 TI 工具集修改了 COFF，然后转换为 ELF。

+   Tru64 UNIX 使用的 ECOFF 改变了符号表示。

+   IBM 开发了 XCOFF（COFF 结合了 TOC 模块格式概念、CSECT 等），并将其用于 AIX。

主要缺点：

+   将专为 C 语言量身定制的调试信息硬编码到符号结构中是复杂的、空间效率低下的，而且丑陋的。

+   辅助符号记录设计不灵活且效率低下。

+   早期系统中不是 32 位对齐的符号和段结构导致了性能问题。

+   不支持弱符号。PE 实现了一种不灵活的弱定义机制，称为“弱外部”。

### Mach-O

卡内基梅隆大学开发了 Mach 内核作为微内核概念的证明。操作系统使用了从 a.out 派生的格式，并进行了微小的修改，命名为 Mach 对象文件格式。缩写 Mach-O 经常被使用。NeXTSTEP 操作系统，然后是 Darwin 采用了 Mach-O。

Mach-O 上的动态共享库支持晚于其他目标文件格式。在 1995 年发布的 NeXTSTEP 手册中，我可以找到 `MH_FVMLIB`（固定虚拟内存库，似乎是一种静态共享库方案），但找不到 `MH_DYLIB`（现代 macOS 用于 .dylib 文件的格式）。

主要缺点：

+   没有 COMDAT 支持。

+   重定位类型的稀缺性。

+   255 节点限制。`.subsections_via_symbols` 有一些缺点（稍后讨论）。

在我看来，Mach-O 在 Mach-O/PE/ELF 中是最受限制的。但我要承认某些创新特性，如 `.subsections_via_symbols` 和 `S_ATTR_LIVE_SUPPORT`。

### ELF

COFF 的挫败感和固有限制，再加上自我施加的字节顺序困境，AT&T 推出了一种开创性的格式：可执行和链接格式（ELF）。ELF 重新审视了先前目标文件格式中的固定内容和硬编码概念，删除了不必要的元素，并使控制结构更加灵活。

这个重大转变被 System V Release 4 所采纳，标志着目标文件格式设计的新时代。在 1990 年代，许多 Unix 和类 Unix 操作系统，包括 Solaris、IRIX、HP-UX、Linux 和 FreeBSD，都转向了 ELF。

## 符号

符号控制结构的最小值需要编码名称、节和值。我们可以要求每个符号都是相对于某个节定义的。我们可以使用节索引为零来表示未定义的符号。

在仅有少量硬编码节的最小对象文件格式中（a.out），节字段可以省略。可以使用类型字段来决定符号是否可以引用函数或数据对象。

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
 typedef struct {
 Elf32_Word	st_name;
 Elf32_Addr	st_value;
 Elf32_Word	st_size;
 unsigned char	st_info;
 unsigned char	st_other;
 Elf32_Half	st_shndx;
} Elf32_Sym;
 typedef struct {
 Elf64_Word st_name; 
 unsigned char st_info; 
 unsigned char st_other; 
 Elf64_Half st_shndx; 
 Elf64_Addr st_value;
 Elf64_Xword st_size;
} Elf64_Sym; 
```

|

符号名称表示为指向字符串表的 32 位索引。32 位整数足够了，而 16 位整数则太小了。

`st_shndx` 使用了一种节省空间的技巧。16 位成员编码一个节索引。如果成员是 `SHN_XINDEX`（0xffff），那么实际值包含在类型为 `SHT_SYMTAB_SHNDX` 的相关节中。这是一个非常好的技巧，因为节的数量几乎总是小于 0xff00。在病态情况下，可能会有更多的节，需要一个类型为 `SHT_SYMTAB_SHNDX` 的节。

`st_info` 指定了符号的类型（4 位）和绑定（4 位）属性。类型分配非常保守，通常意味着不同的链接器行为。对于符号类型的本质上不同的链接器行为并不那么多。所以 4 位看起来很小，但在实践中已经足够了。正如我们将要了解的那样，这明显比 COFF 的类型和存储类表示要小得多。符号的绑定是用于区分局部/弱/全局的。保留的 4 位可以容纳更多的值，但只有 GNU 保留了一个值 (`STB_GNU_UNIQUE`)（在我看来是一个错误的特性）。

在 COFF 中，函数符号可以使用辅助符号记录来编码函数的大小 (`x_fsize`；在 PE 中为 `TotalSize`)。在 ELF 中，`st_size` 是一个固定成员，用于复制重定位和符号化。如果我们消除复制重定位并且不需要符号化启发式方法，这个字段将变成垃圾。

如果我们去掉 `st_size`，这里是一个演示。

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
 struct Elf64_Sym_minimized {
 Elf64_Word st_name; 
 unsigned char st_info; 
 unsigned char st_other; 
 Elf64_Half st_shndx; 
 Elf64_Addr st_value;
} Elf64_Sym; 
```

|

### 符号（a.out）

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

```

|

```
 struct nlist {
 char n_name[8];
#if pdp11
 int n_type;
#else
 char n_type;
 char n_other;
 short n_desc;
#endif
 unsigned n_value;
}; 
```

|

a.out 使用 `nlist` 来表示符号表条目。在 PDP-11 的原始格式中，汇编器生成的符号最多为 7 字节。`n_name[8]` 可以保存带有 NUL 结尾的名称。Unix 对更短的标识符名称的欣赏与此相关:)

为了支持更长的名称，扩展在符号表之后添加了一个字符串表，并允许将 `n_name` 解释为一个索引 (`n_strx`) 到字符串表中。然后，这个成员通过将短名称（8 字节或更少）内联到结构中来节省空间。一些变体，如 binutils 的 64 位 a.out 格式，专门使用索引并移除了 `n_name`。

`n_type`分解为三个子字段，描述了符号是否已定义或未定义、外部或本地以及符号类型。FreeBSD 手册页中列出的值也用于 PDP-11。

对于已定义的符号，`n_type`描述了它是相对于 TEXT、DATA 还是 BSS。

### 符号（COFF）

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
 struct syment {
 union {
 char _n_name[SYMNMLEN]; 
 struct {
 long _n_zeroes; 
 long _n_offset; 
 } _n_n;
 } _n;
 unsigned long n_value; 
 short n_scnum; 
 unsigned short n_type; 
 char n_sclass; 
 char n_numaux; 
}; 
```

|

COFF 采用了 a.out 的方法来节省符号名称的空间。这在大多数符号较短时可能是合理的。然而，随着今天常常较长的符号名称，这种内联技术使得代码复杂化并增加了控制结构大小（从 4 到 8 字节）。

节号是一个 16 位有符号整数，支持最多 32,767 个节。正值表示节索引，而特殊值包括：

+   `N_UNDEF`（0）：未定义的符号（与 a.out 的`n_type`表示不同）。

+   `N_ABS`（-1）：符号具有绝对值。

+   `N_DEBUG`（-2）：特殊的调试符号（值无意义）。

COFF 的`n_type`和`n_sclass`编码了 C 语言的类型和存储类信息。PE 为这些类型和存储类分配了更长的名称，例如，`IMAGE_SYM_TYPE_CHAR/IMAGE_SYM_TYPE_SHORT`，`IMAGE_SYM_CLASS_AUTOMATIC/IMAGE_SYM_CLASS_EXTERNAL`。虽然大部分数值保持一致，但存在一些细微差异：

+   PE 的`IMAGE_SYM_TYPE_VOID`（1）与 System V Release 3 的`#define T_ARG 1 /* function argument (only used by compiler) */`不同。

+   PE 的`IMAGE_SYM_CLASS_WEAK_EXTERNAL`（105）与 System V Release 3 的`#define C_ALIAS 105 /* duplicate tag */`不同。

具有`C_EXT`（`IMAGE_SYM_CLASS_EXTERNAL`）的符号是全局的，并添加到链接器的全局符号表中，类似于 ELF 的`STB_GLOBAL`符号绑定。

System V 提供了一个符号调试器（sdb），它利用`n_type`和`n_sclass`。如果我们承认调试信息格式已经过时，`n_type`和`n_class`就成为 ELF 的`st_info`的冗余对应物。

`n_numaux`与辅助符号记录相关，允许额外的信息，但引入了不规则的符号表条目。虽然看似有益，但它们的用例有限，通常可以使用单独的节进行编码。在 PE 中，辅助符号记录可以表示弱定义，但不支持弱引用。它们还可以为节符号提供额外的信息。

ECOFF 定义了本地符号入口（SYMR）和外部符号入口（EXTR）。

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

```

|

```
typedef struct {
 coff_long value;
 coff_int iss;
 coff_uint st : 6;
 coff_uint sc : 5;
 coff_uint reserved : 1;
 coff_uint index : 20;
} SYMR, *pSYMR;
 typedef struct {
 SYMR asym;
 coff_uint jmptbl:1;
 coff_uint cobol_main:1;
 coff_uint weakext:1;
 coff_uint reserved:29;
 coff_int ifd;
} EXTR, *pEXTR; 
```

|

### 符号（Mach-O）

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

```

|

```
 struct nlist {
 uint32_t n_strx;
 uint8_t n_type;
 uint8_t n_sect;
 int16_t n_desc;
 uint32_t n_value;
};
 struct nlist_64 {
 uint32_t n_strx;
 uint8_t n_type;
 uint8_t n_sect;
 uint16_t n_desc;
 uint64_t n_value;
}; 
```

|

Mach-O 的`nlist`和`nlist_64`与 a.out 的差别不大，只是将`n_other`更改为`n_sect`以表示节索引。8 位的 n_sect 字段限制了可表示的节数为 255，不包括带外数据（稍后讨论）。如果我们将`n_sect`扩展为 32 位，结构大小将增加到 24 字节，与`Elf64_Sym`相同。

像 a.out 一样，`n_type`的`N_EXT`位表示外部符号。`N_PEXT`位表示私有外部符号。

`n_desc`中的关键位是`N_WEAK_DEF`、`N_WEAK_REF`和`N_ALT_ENTRY`。

## 节

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
24
25
26
27

```

|

```
 typedef struct {
 Elf32_Word	sh_name;
 Elf32_Word	sh_type;
 Elf32_Word	sh_flags;
 Elf32_Addr	sh_addr;
 Elf32_Off	sh_offset;
 Elf32_Word	sh_size;
 Elf32_Word	sh_link;
 Elf32_Word	sh_info;
 Elf32_Word	sh_addralign;
 Elf32_Word	sh_entsize;
} Elf32_Shdr;
 typedef struct {
 Elf64_Word	sh_name;
 Elf64_Word	sh_type;
 Elf64_Xword	sh_flags;
 Elf64_Addr	sh_addr;
 Elf64_Off	sh_offset;
 Elf64_Xword	sh_size;
 Elf64_Word	sh_link;
 Elf64_Word	sh_info;
 Elf64_Xword	sh_addralign;
 Elf64_Xword	sh_entsize;
} Elf64_Shdr; 
```

|

段名称表示为对字符串表的 32 位索引。如果我们使用 16 位整数，大量带有符号后缀的段名称（例如 `.text.foo` `.text.bar`）可能会导致索引溢出。

`sh_type` 对段的内容和语义进行分类。在许多情况下，它避免了在代码中硬编码魔术名称。从技术上讲，16 位类型可以很好地工作，但被认为对灵活性不足。

`sh_flags` 描述了各种属性，例如可写和可执行权限，以及该段是否应该出现在可加载段中。在 `Elf32_Shdr` 中，此成员为 32 位，而在 `Elf64_Shdr` 中为 64 位。实际上，没有任何架构定义了位于 32 到 63 的标志，因此此成员有点浪费。

位置和大小。`sh_offset` 给出了从文件开头到段中第一个字节的字节偏移量。为了支持大于 4GiB 的目标文件，此成员必须为 64 位。`sh_size` 给出了段的大小（以字节为单位）。`SHT_NOBITS` 类型的段在文件中不占用空间。为了支持大于 4GiB 的段，此成员必须为 64 位。

地址和对齐。`sh_addr` 描述了可执行文件或共享对象中段的第一个字节应该位于的地址。对于可重定位文件，应将其置零。`sh_addralign` 保存地址对齐。在实践中，即使通用 ABI 不要求，此成员必须是 2 的幂。在 ELF64 中，此成员为 64 位，允许对齐达到 `2**63`。实际上，大于页面大小（如果启用了大页，则为最大的大页大小）的对齐没有意义，并且 2**31 的最大值足够。因此，我们可以使用 log2 值来保存对齐。

连接信息。`sh_link` 保存段索引。`sh_info` 保存段索引或符号索引。如果您记得 `st_shndx` 为 16 位有很好的原因，您将知道这两个字段有点浪费。

对于固定大小条目的表，`sh_entsize` 以字节为单位保存条目大小。在某些用例中，此成员不是 2 的幂。实际上，一个字节就足够了。

虽然 ELF 的段头结构设计灵活，但通过在实际需要的基础上使用较小的数据类型来缩小 `sh_flags`、`sh_link`、`sh_info` 和 `sh_entsize` 的结构，我们可以使结构显着变小，而功能损失不大。

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
24
25
26
27

```

|

```
// 32 bytes
struct Elf32_Shdr_minimized {
 Elf32_Word	sh_name;
 Elf32_Word	sh_type;    // Making this uint16_t and reordering it can decrease the size to 28 bytes
 Elf32_Word	sh_flags;
 Elf32_Addr	sh_addr;
 Elf32_Off	sh_offset;
 Elf32_Word	sh_size;
 uint8_t	sh_addralign;
 uint8_t	sh_entsize;
 Elf32_Half	sh_link;
 Elf32_Half	sh_info;
};
 // 40 bytes
struct Elf64_Shdr_minimized {
 Elf64_Word sh_name;
 Elf64_Word sh_flags;
 Elf64_Addr sh_addr;
 Elf64_Off sh_offset;
 Elf64_Xword sh_size;
 Elf64_Half sh_type;
 uint8_t sh_addralign;
 uint8_t sh_entsize;
 Elf64_Half sh_link;
 Elf64_Half sh_info;
}; 
```

|

将 `sh_type` 缩减为 2 个字节会稍微减少灵活性。如果这被认为不够，我们可以从 `sh_addralign` 中取出 3 位（通过将其转换为位字段），并将它们给予 `sh_type`。

### 段（COFF）

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
24
25
26
27

```

|

```
// COFF (System V Release 3), 40 bytes, when sizeof(long) == 4
struct scnhdr {
 char            s_name[8];      /* section name */
 long            s_paddr;        /* physical address */
 long            s_vaddr;        /* virtual address */
 long            s_size;         /* section size */
 long            s_scnptr;       /* file ptr to raw data for section */
 long            s_relptr;       /* file ptr to relocation */
 long            s_lnnoptr;      /* file ptr to line numbers */
 unsigned short  s_nreloc;       /* number of relocation entries */
 unsigned short  s_nlnno;        /* number of line number entries */
 long            s_flags;        /* flags */
};
 // PE, 40 bytes
struct section {
 char Name[8];
 uint32_t VirtualSize;
 uint32_t VirtualAddress;
 uint32_t SizeOfRawData;
 uint32_t PointerToRawData;
 uint32_t PointerToRelocations;
 uint32_t PointerToLineNumbers;
 uint16_t NumberOfRelocations;
 uint16_t NumberOfLineNumbers;
 uint32_t Characteristics;
}; 
```

|

与 COFF 相比，PE 的段控制结构进行了轻微修改，`s_paddr => VirtualSize`。

当 `long` 为 4 字节时，所呈现的结构的大小为 40 字节。如果我们将 `s_paddr, s_vaddr, s_size, s_scnptr, s_relptr, s_lnnoptr` 扩展为 8 字节，则结构将为 64 字节。

节名称支持最多 8 个字节。更长的名称将需要类似于符号控制结构的扩展。

编码`s_paddr`和`s_vaddr`都是浪费的。ELF 将物理地址编码到段中，因此从其节结构中删除了该成员。

COFF 将重定位的位置和大小嵌入到节结构中。这实际上相当不错。16 位的`s_nreloc`可能看起来受限制，但对于可重定位文件是足够的。实际上，可重定位链接可以使单个节的重定位数量超过 65536。

`s_lnnoptr`和`s_nlnno`指向行号条目，将地址与源文件行号相关联。嵌入式的本质是不灵活的。

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

```

|

```
 struct lineno {
 union {
 long l_symndx; 
 long l_paddr; 
 } l_addr;
 unsigned short l_lnno ; 
}; 
```

|

这种简单的格式已经过时了。在 DWARF 中，行号信息中的特殊操作码可以更节省空间地编码信息，并提供更多信息，比如列号。

### 节（Mach-O）

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
 struct section_64 {
 char sectname[16];
 char segname[16];
 uint64_t addr;
 uint64_t size;
 uint32_t offset;
 uint32_t align;
 uint32_t reloff;
 uint32_t nreloc;
 uint32_t flags;
 uint32_t reserved1; 
 uint32_t reserved2; 
 uint32_t reserved3;
}; 
```

|

Mach-O 是如何得到如此巨大的节结构的？让我们找出原因...

Mach-O 二进制文件被划分为段，每个段包含一个或多个节。节结构编码了节名称和段名称，两者都可以长达 16 个字节。这种表示允许读取节名称而无需字符串表，但对于描述性名称来说限制较大。节语义是从名称派生的（不像 ELF）。

段名冗余地编码在节结构中。我们可以从节名和标志中派生段，例如，`S_ATTR_SOME_INSTRUCTIONS => __TEXT`，`S_ZEROFILL => ZeroFill __DATA`。

有一个严重的限制：由于`nlist::n_sect`是`uint8_t`，最多只能有 255 个部分。这显然太过严格了。幸运的是，一个创新的特性`.subsections_via_symbols`克服了这个限制。该特性使用一个整体部分，并使用“原子”将其划分为片段（子部分）。这比 ELF 的`-ffunction-sections -fdata-sections -fno-unique-section-names`更加节省空间。但是，这里有汇编器的限制、重定位处理的复杂性以及可能导致无法确保两个非局部符号不被重新排序的潜在损失。

像 COFF 一样，Mach-O 将重定位的位置和大小嵌入到节结构中。

`reserved1`和`reserved2`与 ELF 的连接信息类似。

`__TEXT,__stub`（类似于 ELF 的`.plt`）、`__TEXT,__got`（类似于 ELF 的`.got`）、`__TEXT,__la_symbol_ptr`（类似于 ELF 的`.got.plt`）和`__DATA,__thread_ptrs`将`reserved1`设置为间接符号表的索引（偏移量由`LC_DYSYMTAB`命令中的`indirectsymoff`指定）。

对于`__TEXT,__stub`，`reserved2`是一个条目的大小，例如，对于 x86-64 是 6（`jmpq *__la_symbol_ptr(%rip)`）。这类似于 ELF x86-64 的`DT_X86_64_PLTSZ`。对于其他节，`reserved2`为零。

## 重定位

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
 typedef struct {
 Elf32_Addr r_offset;
 Elf32_Word r_info;
} Elf32_Rel;
 typedef struct {
 Elf32_Addr r_offset;
 Elf32_Word r_info;
 Elf32_Sword r_addend;
} Elf32_Rela;
 typedef struct {
 Elf64_Addr r_offset;
 Elf64_Xword r_info;
} Elf64_Rel;
 typedef struct {
 Elf64_Addr r_offset;
 Elf64_Xword r_info;
 Elf64_Sxword r_addend;
} Elf64_Rela; 
```

|

`r_info` 指定必须进行重定位的符号表索引，以及要应用的重定位类型。

+   ELFCLASS32：8 位类型，24 位符号索引

+   ELFCLASS64：32 位类型，32 位符号索引

有两个变体，REL 和 RELA。让我们引用通用 ABI：

> 如前所述，只有 Elf32_Rela 和 Elf64_Rela 条目包含显式修正值。类型为 Elf32_Rel 和 Elf64_Rel 的条目在要修改的位置存储隐式修正值。根据处理器架构，一种形式或另一种形式可能是必要的或更方便的。因此，特定机器的实现可能仅使用一种形式，或根据上下文使用任一形式。

可重定位文件需要很多重定位类型，而可执行文件和共享对象只需要很少。前者通常称为静态重定位，而后者称为动态重定位。

在少数动态重定位类型中，大多数不需要修正值成员。lld 提供了一个选项 `-z rel` 来使用 `SHT_REL/DT_REL` 动态重定位。

如果我们忽略 REL 动态重定位方案，那么所有现代架构都仅使用 RELA。大多数架构仅用几位编码立即数，这对于许多可重定位文件的用途来说是不足够的。

ELFCLASS64，其具有 64 位成员，与 ELFCLASS32 的 32 位成员相比，大小翻倍。由于重定位通常占据对象文件的重要部分，因此这种大小差异可能会引起用户关注。然而，在实践中，即使在 64 位上下文中，24 位符号索引通常也足够。因此，如果 64 位架构的重定位类型要求小于 256，则 ELFCLASS32 可以是一个可行且更节省空间的选项。

在 2024 年 3 月，我提出了 CREL 作为另一种重定位格式。

### 重定位（a.out）

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
 struct relocation_info {
 int	   r_address;
 unsigned r_symbolnum : 24,
 r_pcrel : 1,
 r_length : 2,
 r_extern : 1,
 r_pad : 4;
}; 
```

|

`r_symbolnum` 映射 ELF 的 `ELF32_R_SYM`。

另外一组位域，类似于 ELF 的 `ELF32_R_TYPE`，但分为不同的字段：

为个别位保留专用语义可能会限制适应性。COFF 和 ELF 选择删除位字段，以提供更大的灵活性。

### 重定位（COFF）

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
 struct reloc {
 long r_vaddr;
 long r_symndx;
 unsigned short r_type;
}; 
```

|

此格式类似于 ELF 的 `Elf32_Rel`。

`r_vaddr` 给出要应用重定位操作的位置的虚拟地址。如果我们将 `r_vaddr` 解释为偏移量（如 PE 所做），并将节大小限制为 32 位，则我们可以将此结构重用于 64 位架构。

`r_symndx` 是一个 32 位符号表索引。

`r_type` 是一个 16 位重定位类型，与 ELF 相比数量有限。

COFF 通常支持的重定位类型比 ELF 少。System V Release 3 为每个架构定义了很少的重定位。在 binutils 中，`include/coff/*.h` 文件为更多的架构定义了重定位。

虽然 ELF 在可重定位文件和可执行文件中都使用 REL/RELA，但在 PE 映像文件中，导入地址表和基址重定位表（`.reloc`）是完全不同的设计。

### 重定位（Mach-O）

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

```

|

```
 struct relocation_info {
 int32_t	r_address; 
 uint32_t     r_symbolnum:24, 
 r_pcrel:1, 
 r_length:2, 
 r_extern:1, 
 r_type:4; 
};
 struct scattered_relocation_info {
#ifdef __BIG_ENDIAN__
 uint32_t r_scattered : 1, r_pcrel : 1, r_length : 2, r_type : 4,
 r_address : 24;
#else
 uint32_t r_address : 24, r_type : 4, r_length : 2, r_pcrel : 1,
 r_scattered : 1;
#endif
 int32_t r_value;
}; 
```

|

Mach-O 的重定位结构与 a.out 的紧密相似，并适应了`r_symbolnum`的含义。当`r_extern == 0`（本地）时，`r_symbolnum`成员引用节索引而不是符号索引。这是为了支持自定义节，打破传统 a.out 的三节限制（文本、数据和 bss）。

如前所述，为位域（`r_pcrel`、`r_length` 和 `r_scattered`）分配位数严重限制了重定位类型的数量。

与重定位类型限制相关的是，在数据段中的`.long foo - .`需要一对重定位，`SUBTRACTOR` 和`/UNSIGNED`。我在将 LLVM XRay 移植到 Apple 系统上有一些注释。

Mach-O 使用一些节在`__LINKEDIT`段中向 dyld 传递信息。

Dennis MacAlistair Ritchie 的[`A.OUT (V)` manpage](https://www.bell-labs.com/usr/dmr/www/man51.pdf)（1971）描述了原始的 a.out 格式。头部包含 6 个字。

+   一个“br .+14”指令（205(8)）

+   程序文本的大小

+   符号表的大小

+   重定位位区域的大小

+   数据区域的大小

+   一个零字词（目前未使用）

文本重定位是隐含的。

较新版本引入了新的魔数，分离了文本重定位和数据重定位，并添加了入口点（`a_entry`）。

## 大小比较

待办事项

## 减小体积的机会

ELFCLASS32 结构已经很紧凑，提供了有限的体积减小潜力。ELFCLASS64 结构虽然灵活，但可以通过牺牲一些灵活性（64 位数量）来进行优化。64 位符号控制结构很紧凑，但节和重定位非常浪费，如果我们可以牺牲一些灵活性。

正如 ELF 论文所承认的那样，“可重定位和可执行文件并不一定具有相同的约束，我们考虑使用两种文件格式。最终，我们决定这两种活动相似到足以只使用一种格式。”检查可执行文件的工具比可重定位文件更多。因此，自然地，我们可能只想更改可重定位文件。我们可以将 ELFCLASS32 可重定位文件用于 64 位架构吗？

好吧，x86-64 和 AArch64 对 ELFCLASS32 和 ELFCLASS64 有明显的区分。ELFCLASS32 适用于 ILP32（x32，aarch64_ilp32），而 ELFCLASS64 适用于 LP64。然而，被停用的 Itanium 架构为 ELFCLASS32 可以用于 LP64 程序树立了一个先例。引用它的 psABI（*Intel Itanium Processorspecific Application Binary Interface (ABI)*）。

> 对于 Itanium 架构 ILP32 可重定位（即类型 ET_REL）对象，e_ident[EI_CLASS]中的文件类值必须为 ELFCLASS32。对于 LP64 可重定位对象，文件类值可以是 ELFCLASS32 或 ELFCLASS64，并且符合规范的链接器必须能够处理其中一个或两个类。ET_EXEC 或 ET_DYN 对象文件类型必须对 ILP32 使用 ELFCLASS32，对 LP64 程序使用 ELFCLASS64。
> 
> 出现在 LP64 程序的 ELFCLASS32 可重定位对象中的地址被零扩展隐式扩展到 64 位。
> 
> 注意：在 LP64 程序中合法的一些构造，例如超出 32 位范围的绝对 64 位地址，可能需要使用 ELFCLASS64 可重定位对象文件。

鉴于先前的成果，当代码大小引起人们关注时，允许 ELFCLASS32 似乎是有希望的。理想情况下，应该有一个标记来区分使用 ILP32 和 LP64 的 ELFCLASS32 对象文件。

主要的变化在汇编器和链接器中。还要确保二进制操作程序（如 objcopy）和转储工具对它们满意。

进一步的优化潜力在于探索使用 `Elf32_Rel` 而不是 `Elf32_Rela`，以实现更小的重定位。

### 替换控制结构

这种方法与是否采用 ELFCLASS32 无关，可以应用于 ELFCLASS32 和 ELFCLASS64。ELF 论文清楚地指出，“ELF 允许扩展和重新定义其他控制结构”。但是，由于许多工具依赖于现有结构，因此需要谨慎。

一个有希望的例子是 `Elf32_Shdr_minimized`，一个定制结构，从标准的 `Elf32_Shdr` 的 40 字节减少到 32 字节。虽然我会有些紧张，但如果我们将 `sh_type` 缩小到一个 `uint16_t`，结构大小可以减少到 28 字节。

## stabs

早期的调试器使用了一种名为“stabs”的调试信息格式（缩写为符号表条目；至少可追溯到 1979 年的 UNIX/32V）。Stabs 是使用 a.out 对象文件格式中的额外符号表条目进行编码的。

|

```
1
2
3

```

|

```
.stabs "string",type,other,desc,value
.stabn type,other,desc,value
.stabd type,other,desc

```

|

Stabs 被移植到 COFF 上，用于 System V Release 2，在一些机器上使用。System V Release 4 切换到了 ELF，并放弃了 stabs，转而采用了一种新开发的格式叫做 DWARF。它的调试器 sdb 被重写以支持 DWARF，不再支持 stabs。（DWARF 的第一个版本后来于 1992 年 1 月由 UNIX 国际编程语言特别兴趣小组（SIG）发布。）

然而，stabs 继续在其他操作系统中使用，包括 *BSD、AIX 和 IRIX。例如，GNU 汇编器为 ELF 添加了 stabs 支持（`n_strx` 是 32 位）。

GCC 13 [移除了 stabs 支持](https://gcc.gnu.org/git/gitweb.cgi?p=gcc.git;h=7e0db0cdf01e9c885a29cb37415f5bc00d90c029)。

Stabs 比 DWARF 不那么高效。当编译一个非平凡程序（因此 DWARF 中的样板不那么重要）时，你可能会观察到 `.stab` 和 `.stabstr` 消耗的空间比 `.debug_*` 节更多，即使 DWARF 更具表现力且包含更多信息。

## 异构性和挑战

尽管操作系统和体系结构的多样性给应用程序开发者带来了复杂性，但对象文件格式的异构性为工具链开发提供了独特的挑战，可能对应用程序开发者和用户并不是非常明显。

集成诸如链接时间优化（LTO）、基于配置文件的优化（PGO）和消毒剂等功能存在着复杂性，因为目标文件格式特定的限制和细微差别。虽然大多数开发者主要关注特定的格式，但他们在开发过程中仍需要谨慎行事，以避免对其他平台造成干扰。

## 待办事项

WebAssembly
