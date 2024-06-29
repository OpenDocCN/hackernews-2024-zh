<!--yml

类别：未分类

日期：2024-05-27 12:48:39

-->

# 轻 ELF：探索潜在的尺寸减小 | MaskRay

> 来源：[https://maskray.me/blog/2024-04-01-light-elf-exploring-potential-size-reduction](https://maskray.me/blog/2024-04-01-light-elf-exploring-potential-size-reduction)

ELF 的设计强调了 [其控制结构的自然大小和对齐指南](https://maskray.me/blog/2024-03-09-a-compact-relocation-format-for-elf)。虽然在旧日确保了高效处理，但可能导致更大的文件尺寸。我提出“轻 ELF”（`EV_LIGHT`，版本 2）——灵感来自托尔金的传奇中的光明精灵（他们曾在瓦林诺的两树之光下见证过光明）。

在轻 ELF 文件中，ELF 头的 `e_version` 成员被设为 2。`EV_CURRENT` 保持为 1 以保持向后兼容性。

|

```
1
2
3

```

|

```
#define EV_NONE 0
#define EV_CURRENT 1
#define EV_LIGHT 2

```

|

在链接程序时，传统 ELF（版本 1）和轻 ELF（版本 2）文件可以混合使用。

## 重定位

轻 ELF 利用 [CREL](/blog/2024-03-09-a-compact-relocation-format-for-elf) 进行重定位。传统 ELF 中的 [REL 和 RELA](/blog/2024-01-14-exploring-object-file-formats#relocations) 未被使用。

现有的惰性绑定方案依赖于对 `DT_JMPREL` 表内重定位条目的随机访问。由于 CREL 的顺序性质，保持惰性绑定需要一个内存分配来保存解码的 JUMP_SLOT 重定位。

传统 ELF（版本 1）[节头表](/blog/2024-01-14-exploring-object-file-formats#sections) 可能很大。轻 ELF 通过 ELF 头中 `e_shentsize == 0` 信号的紧凑节头表格式解决了这个问题。

[一个用于 ELF 的紧凑节头表](/blog/2024-03-31-a-compact-section-header-table-for-elf) 包含详细信息。其当前版本如下供您参考。

`nshdr` 表示节的数量（包括 `SHN_UNDEF`）。节头表（位于 `e_shoff` 处）以 `nshdr` 个 `Elf_Word` 值开头。这些值指定每个节头相对于 `e_shoff` 的偏移量。

在这些偏移量之后，`nshdr` 节头被编码。每个头部以一个 `presence` 字节开头，指示随后的每个 `Elf_Shdr` 成员使用显式值还是默认值：

+   `sh_name`，ULEB128 编码

+   `sh_type`，ULEB128 编码（如果 `presence & 1`），默认为 `SHT_PROGBITS`

+   `sh_flags`，ULEB128 编码（如果 `presence & 2`），默认为 0

+   `sh_addr`，ULEB128 编码（如果 `presence & 4`），默认为 0

+   `sh_offset`，ULEB128 编码

+   `sh_size`，ULEB128 编码（如果 `presence & 8`），默认为 0

+   `sh_link`，ULEB128 编码（如果 `presence & 16`），默认为 0

+   `sh_info`，ULEB128 编码（如果 `presence & 32`），默认为 0

+   `sh_addralign`，ULEB128 编码作为对数值（如果 `presence & 64`），默认为 1

+   `sh_entsize`，ULEB128 编码（如果 `presence & 128`），默认为 0

在传统 ELF 中，`sh_addralign` 可以为 0 或正整数的二次幂，其中 0 和 1 表示该节段没有对齐约束。虽然紧凑编码无法编码 `sh_addralign` 值为 0，但这并不影响一般性。

C++ 示例代码，解码特定节段头部的示例：

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
 const uint8_t *sht = base + ehdr->e_shoff;
const uint8_t *p = sht + ((Elf_Word*)sht)[i];
uint8_t presence = *p++;
Elf_Shdr shdr = {};
shdr.sh_name = readULEB128(p);
shdr.sh_type = presence & 1 ? readULEB128(p) : ELF::SHT_PROGBITS;
shdr.sh_flags = presence & 2 ? readULEB128(p) : 0;
shdr.sh_addr = presence & 4 ? readULEB128(p) : 0;
shdr.sh_offset = readULEB128(p);
shdr.sh_size = presence & 8 ? readULEB128(p) : 0;
shdr.sh_link = presence & 16 ? readULEB128(p) : 0;
shdr.sh_info = presence & 32 ? readULEB128(p) : 0;
shdr.sh_addralign = presence & 64 ? 1UL << readULEB128(p) : 1;
shdr.sh_entsize = presence & 128 ? readULEB128(p) : 0; 
```

|

尽管当前格式允许使用表开始处的偏移量进行节段头部的 O(1) 原地随机访问，但这种访问模式在实践中似乎不常见。至少在 llvm-project 代码库中，我没有遇到（或者记得）任何实例。因此，我考虑移除这个功能。

在 llvm-project 的发布构建 (`-O3 -ffunction-sections -fdata-sections -Wa,--crel`) 中，传统节段头表占 `.o` 文件大小的 16.4%，而紧凑节段头表将这一比率显著降低至 4.7%。

## 符号表

与其他节段类似，[符号表](/blog/2024-01-14-exploring-object-file-formats#symbols) 和字符串表节段 (`SHT_SYMTAB` 和 `SHT_STRTAB`) 可以通过 [`SHF_COMPRESSED`](/blog/2023-07-07-compressed-arbitrary-sections) 进行压缩。然而，不建议压缩动态符号表 (`.dynsym`) 及其相关的字符串表 (`.dynstr`)。

符号表节段具有非零的 `sh_entsize`，在压缩后保持不变。

字符串表，用于存储符号名称（也包括 LLVM 输出中的节段名称），通常比符号表本身要大得多。为了减小其大小，我们可以利用文本压缩算法。虽然压缩字符串表时，同时压缩符号表可能是合理的选择，但对符号表本身使用紧凑编码并不会带来显著的好处。

程序头部，虽然每个 `Elf64_Phdr` 都很大（每个为 56 字节），且不需要随机访问，但在可执行文件或共享对象中的数量通常有限。因此，它们的整体大小贡献相对较小。轻量 ELF 保留现有格式。

## 节段压缩

ELFCLASS64 尤其对于头部开销，压缩节段面临挑战。

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
typedef struct {
 Elf32_Word	ch_type;
 Elf32_Word	ch_size;
 Elf32_Word	ch_addralign;
} Elf32_Chdr;
 typedef struct {
 Elf64_Word	ch_type;
 Elf64_Word	ch_reserved;
 Elf64_Xword	ch_size;
 Elf64_Xword	ch_addralign;
} Elf64_Chdr; 
```

|

当与 `-ffunction-sections` 和 `-fdata-sections` 等生成许多较小节段的特性一起使用时，开销和 [对齐填充](/blog/2022-01-23-compressed-debug-sections#alignment) 限制了其有效性。例如，我发现大型 `Elf64_Chdr` 使得评估压缩的 `.rela.*` 节段变得困难。轻量 ELF 通过引入更小占地面积的头部格式来解决这一挑战：

+   `ch_type`，ULEB128 编码

+   `ch_size`，ULEB128 编码

+   `ch_addralign`，ULEB128 编码的对数值

这种方法允许轻量 ELF 仅用 3 字节表示较小节段的头部信息，而传统格式则需要 24 字节。内容不再保证字对齐，这是大多数压缩库无需的属性。

此外，允许使用带有`SHF_ALLOC`标志的[压缩段](https://groups.google.com/g/generic-abi/c/HUVhliUrTG0)。尽管如此，在非可重定位文件中使用它们需要谨慎。

## 实验

我开发了一个实现了紧凑段头表和CREL的Clang/lld原型（[https://github.com/MaskRay/llvm-project/tree/april-2024](https://github.com/MaskRay/llvm-project/tree/april-2024)）。

| 136012504 | 18284992 | -O3 |
| --- | --- | --- |
| 111583312 | 18284992 | -O3 -Wa,--crel |
| 97976973 | 4604341 | -O3 -Wa,--crel,--cshdr |
| 2174179112 | 260281280 | -g |
| 1763231672 | 260281280 | -g -Wa,--crel |
| 1577187551 | 74234983 | -g -Wa,--crel,--cshdr |

## 轻量级ELF：一个思想实验

到目前为止，你可能已经意识到这篇文章是一个笑话。虽然提升`e_version`并修改`Elf_Chdr`可能不可行，但考虑紧凑段头和压缩符号/字符串表的可能性还是很有趣的。也许这能引发一些有趣的讨论！
