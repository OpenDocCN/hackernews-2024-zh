<!--yml

类别：未分类

日期：2024-05-27 14:49:47

-->

# 理解 x86_64 分页 - zolutal 的博客

> 来源：[https://zolutal.github.io/understanding-paging/](https://zolutal.github.io/understanding-paging/)

我花了相当多的时间研究 x86_64 页表，理解地址转换并不容易，当我开始学习时，我觉得关于它是如何工作的材料很难理解。所以在这篇博客文章中，我将尝试提供一种“我在学习分页时希望拥有的东西”的形式。

快速说明，我只会讨论 PML4（页面映射级别 4）的分页上下文，因为它目前是主流的 x86_64 分页方案，而且可能会在相当长的一段时间内保持主导地位。

## 环境

这并不是必需的，但我建议您准备好具有 QEMU + gdb 的 Linux 内核调试设置以跟随操作。如果您从未尝试过这样做，也许可以尝试这个仓库：[easylkb](https://github.com/deepseagirl/easylkb)（我从未使用过，但我听说过好评），或者如果您想避免自己设置环境，[pwn.college](https://pwn.college/) 上的任何内核安全挑战的实践模式也可以（`vm connect` 和 `vm debug` 是要知道的命令）。

我建议这样做是因为我认为在你自己运行相同的命令并能够根据在 gdb 中看到的内容执行页面漫游是一个很好的理解测试。

## 一个页面是什么玩意儿

在 x86_64 上，一个页是内存的 0x1000 字节切片，它是 0x1000 字节对齐的。

这就是为什么如果您曾经查看过 /proc/<pid>/maps，您会发现所有地址范围都以一个以 0x000 结尾的地址开始和结束，因为在 x86_64 上内存映射的最小大小是页面大小（0x1000 字节），而且页面需要‘页面对齐’（最后 12 位必须为零）。

‘虚拟页’将通过您的 MMU 解析为一个单独的‘物理页’（又称‘页面框’），尽管许多虚拟页可能指向同一个物理页。

## 一个虚拟地址中有什么

PML4，正如人们可能猜到的，有四级分页结构，这些分页结构称为‘页表’。一个页表是一个页大小的内存区域，其中包含 512 个 8 字节的页表条目。每个页表的条目将引用下一级的页表或虚拟地址解析到的最终物理地址。

用于地址转换的页表条目是基于内存访问的虚拟地址的。每一级有 512 个条目，这意味着在每一级上，有 9 位虚拟地址被用于索引到相应的页表中。

假设我们有这样一个地址：

`0x7ffe1c9c9000`

这个地址的最后 12 位表示物理页内的偏移量：

`0x7ffe1c9c9000 & 0xfff = 0x0`

这意味着一旦我们确定了虚拟地址解析到的页的物理地址，我们将在结果中加上零以得到最终的物理地址。

在最后的12位之后，即再次只是最终页面内的偏移量，虚拟地址由指向页表的索引组成。如上所述，每个分页级别使用虚拟地址的9位，因此，分页结构的最低级别，即页表，由地址的下一个9位索引（通过位掩码`& 0x1ff`对移位值）索引。对于接下来的级别，我们只需每次向右移动另外九位，再次掩盖最低的九位作为我们的索引。对于上面的地址，这样做给我们这些索引：

```
Level 1, Page Table (PT):
Index = (0x7ffe1c9c9000 >> 12) & 0x1ff = 0x1c9

Level 2, Page Middle Directory (PMD):
Index = (0x7ffe1c9c9000 >> 21) & 0x1ff = 0x0e4

Level 3, Page Upper Directory (PUD):
Index = (0x7ffe1c9c9000 >> 30) & 0x1ff = 0x1f8

Level 4, Page Global Directory (PGD):
Index = (0x7ffe1c9c9000 >> 39) & 0x1ff = 0x0ff 
```

## 所有你的基地

现在我们知道如何索引页表和它们大致包含什么，那么它们实际上在哪里呢？

好吧，你CPU的每个线程都有一个称为`cr3`的页表基址寄存器。

`cr3`保存了分页结构的最高级别，也就是页面全局目录（PGD）的物理地址。

从gdb，在调试内核时，你可以像这样读取`cr3`的内容：

```
gef➤  p/x $cr3
$1 = 0x10d664000 
```

`cr3`寄存器可以保存一些额外的信息，而不仅仅是PGD地址，这取决于正在使用的处理器功能，因此从`cr3`寄存器获取PGD的物理地址的更一般方法是像这样屏蔽其内容的低12位：

```
gef➤  p/x $cr3 & ~0xfff
$2 = 0x10d664000 
```

## 页表条目

让我们看看从`cr3`中获取的物理地址在gdb中是什么。通过QEMU Monitor向gdb公开的`monitor xp/...`命令让我们打印出vm的物理内存，执行`monitor xp/512gx ...`将打印出由`cr3`引用的PGD的所有512个条目的整个内容：

```
gef➤  monitor xp/512gx 0x10d664000
...
000000010d664f50: 0x0000000123fca067 0x0000000123fc9067
000000010d664f60: 0x0000000123fc8067 0x0000000123fc7067
000000010d664f70: 0x0000000123fc6067 0x0000000123fc5067
000000010d664f80: 0x0000000123fc4067 0x0000000123fc3067
000000010d664f90: 0x0000000123fc2067 0x000000000b550067
000000010d664fa0: 0x000000000b550067 0x000000000b550067
000000010d664fb0: 0x000000000b550067 0x0000000123fc1067
000000010d664fc0: 0x0000000000000000 0x0000000000000000
000000010d664fd0: 0x0000000000000000 0x0000000000000000
000000010d664fe0: 0x0000000123eab067 0x0000000000000000
000000010d664ff0: 0x000000000b54c067 0x0000000008c33067 
```

这产生了大量输出，其中大部分是零，所以我只在这里包含了输出的尾部。

这个输出可能对你来说还没有意义，但是我们可以观察数据中的一些模式，例如很多8字节的条目以`0x67`结尾。

## 解码PGD条目

从上面的PGD输出中，让我们以值为`0x0000000123fca067`的`0x000000010d664f50`处的PGD条目为例，看看如何解码一个条目。

让我们用那个条目值的二进制表示来做一下：

```
gef➤  p/t 0x0000000123fca067
$6 = 100100011111111001010000001100111 
```

这里有一个小图示，显示条目中的每个位表示什么：

```
~ PGD Entry ~                                                   Present ──────┐
                                                            Read/Write ──────┐|
                                                      User/Supervisor ──────┐||
                                                  Page Write Through ──────┐|||
                                               Page Cache Disabled ──────┐ ||||
                                                         Accessed ──────┐| ||||
                                                         Ignored ──────┐|| ||||
                                                       Reserved ──────┐||| ||||
┌─ NX          ┌─ Reserved                             Ignored ──┬──┐ |||| ||||
|┌───────────┐ |┌──────────────────────────────────────────────┐ |  | |||| ||||
||  Ignored  | ||               PUD Physical Address           | |  | |||| ||||
||           | ||                                              | |  | |||| ||||
0000 0000 0000 0000 0000 0000 0000 0001 0010 0011 1111 1100 1010 0000 0110 0111
       56        48        40        32        24        16         8         0 
```

下面是这些标签的含义：

+   NX（不可执行）- 如果设置了此位，则不会执行此PGD条目的任何内存映射的后代。

+   保留-这些值必须为零。

+   PUD物理地址-与此PGD条目关联的PUD的物理地址。

+   已访问-如果由该条目或其后代引用的任何页面，此位将由MMU设置，并且可以由OS清除。

+   页面高速缓存禁用（PCD）- 此PGD条目的后代页面不应进入CPU的高速缓存层次结构，有时也称为“不可缓存”（UC）位。

+   页面写穿（WT）- 写入此PGD条目的后代页面应立即写入RAM，而不是在最终更新RAM之前缓冲写入CPU缓存。

+   当此位未设置时，用户/监督员无法访问此PGD的后代页，除非处于监督模式。

+   读/写 - 当此位未设置时，此PGD的后代页无法被写入。

+   存在 - 如果此位未设置，则处理器不会使用此条目进行地址转换，其他位也不会应用。

在这里我们真正关心的位是存在位，表示下一级分页结构的物理地址的位，PUD物理地址位，以及权限位：NX、用户/监督员和读/写。

+   存在位非常重要，因为如果没有设置，那么条目的其余部分将被忽略。

+   PUD物理地址让我们通过告诉我们下一级分页结构的物理地址位于哪里来继续页行走。

+   权限位都适用于作为PGD条目后代的页面，确定这些页面可以如何被访问。

其余位对我们的目的并不那么重要：

+   如果条目在转换内存访问中被使用，则访问位被设置，对于页行走而言并不重要。

+   页面缓存禁用和页面写入缓存对于正常内存映射不会被使用，也不会影响页面转换或权限，因此让我们忽略它们。

因此，解码此条目，我们了解到：

PUD存在：

```
gef➤  p/x 0x0000000123fca067 & 0b0001
$18 = 0x1 
```

PUD及以下的映射可能是可写的：

```
gef➤  p/x 0x0000000123fca067 & 0b0010
$19 = 0x2 
```

PUD及以下的映射可能可以被用户访问：

```
gef➤  p/x 0x0000000123fca067 & 0b0100
$20 = 0x4 
```

PUD的物理地址（位（51:12]）是`0x123fca000`：

```
gef➤  p/x 0x0000000123fca067 & ~((1ull<<12)-1) & ((1ull<<51)-1)
$21 = 0x123fca000 
```

PUD及以下的映射可能是可执行的：

```
gef➤  p/x 0x0000000123fca067 & (1ull<<63)
$22 = 0x0 
```

## 解码所有级别的条目

现在我们已经看到如何解码PGD条目，解码其他级别并没有太大区别，至少在常见情况下是这样。

在所有这些图表中，“X”表示该位可以是零或一，否则，如果某位设置为特定值，则该值要么是架构所要求的，要么是图表所示编码要求的。

### PGD

```
~ PGD Entry ~                                                   Present ──────┐
                                                            Read/Write ──────┐|
                                                      User/Supervisor ──────┐||
                                                  Page Write Through ──────┐|||
                                               Page Cache Disabled ──────┐ ||||
                                                         Accessed ──────┐| ||||
                                                         Ignored ──────┐|| ||||
                                                       Reserved ──────┐||| ||||
┌─ NX          ┌─ Reserved                             Ignored ──┬──┐ |||| ||||
|┌───────────┐ |┌──────────────────────────────────────────────┐ |  | |||| ||||
||  Ignored  | ||               PUD Physical Address           | |  | |||| ||||
||           | ||                                              | |  | |||| ||||
XXXX XXXX XXXX 0XXX XXXX XXXX XXXX XXXX XXXX XXXX XXXX XXXX XXXX XXXX 0XXX XXXX
       56        48        40        32        24        16         8         0 
```

这一点我们已经看过了，在前一节中我进行了详细描述，但没有填写特定的PGD条目。

### PUD

```
~ PUD Entry, Page Size unset ~                                  Present ──────┐
                                                            Read/Write ──────┐|
                                                      User/Supervisor ──────┐||
                                                  Page Write Through ──────┐|||
                                               Page Cache Disabled ──────┐ ||||
                                                         Accessed ──────┐| ||||
                                                         Ignored ──────┐|| ||||
                                                      Page Size ──────┐||| ||||
┌─ NX          ┌─ Reserved                             Ignored ──┬──┐ |||| ||||
|┌───────────┐ |┌──────────────────────────────────────────────┐ |  | |||| ||||
||  Ignored  | ||               PMD Physical Address           | |  | |||| ||||
||           | ||                                              | |  | |||| ||||
XXXX XXXX XXXX 0XXX XXXX XXXX XXXX XXXX XXXX XXXX XXXX XXXX XXXX XXXX 0XXX XXXX
       56        48        40        32        24        16         8         0 
```

如您在上面的PUD图表中所见，与PGD的图表非常相似，唯一的区别是引入了“页大小”位。设置页大小位会改变我们如何解释PUD条目。对于此图表，我们假设未设置，这是最常见的情况。

### PMD

```
~ PMD Entry, Page Size unset ~                                  Present ──────┐
                                                            Read/Write ──────┐|
                                                      User/Supervisor ──────┐||
                                                  Page Write Through ──────┐|||
                                               Page Cache Disabled ──────┐ ||||
                                                         Accessed ──────┐| ||||
                                                         Ignored ──────┐|| ||||
                                                      Page Size ──────┐||| ||||
┌─ NX          ┌─ Reserved                             Ignored ──┬──┐ |||| ||||
|┌───────────┐ |┌──────────────────────────────────────────────┐ |  | |||| ||||
||  Ignored  | ||                PT Physical Address           | |  | |||| ||||
||           | ||                                              | |  | |||| ||||
XXXX XXXX XXXX 0XXX XXXX XXXX XXXX XXXX XXXX XXXX XXXX XXXX XXXX XXXX 0XXX XXXX
       56        48        40        32        24        16         8         0 
```

同样，PMD图表与上一个图表非常相似，就像PUD条目一样，我们现在忽略了"页大小"位。

### PT

```
~ PT Entry ~                                                    Present ──────┐
                                                            Read/Write ──────┐|
                                                      User/Supervisor ──────┐||
                                                  Page Write Through ──────┐|||
                                               Page Cache Disabled ──────┐ ||||
                                                         Accessed ──────┐| ||||
┌─── NX                                                    Dirty ──────┐|| ||||
|┌───┬─ Memory Protection Key              Page Attribute Table ──────┐||| ||||
||   |┌──────┬─── Ignored                               Global ─────┐ |||| ||||
||   ||      | ┌─── Reserved                          Ignored ───┬─┐| |||| ||||
||   ||      | |┌──────────────────────────────────────────────┐ | || |||| ||||
||   ||      | ||            4KB Page Physical Address         | | || |||| ||||
||   ||      | ||                                              | | || |||| ||||
XXXX XXXX XXXX 0XXX XXXX XXXX XXXX XXXX XXXX XXXX XXXX XXXX XXXX XXXX XXXX XXXX
       56        48        40        32        24        16         8         0 
```

在页表条目中，情况变得更有趣，出现了一些以前级别中不存在的新字段/属性。

这些新字段/属性包括：

+   内存保护密钥（MPK或PK）：这是一个x86_64扩展，允许为具有特定密钥的所有页面分配4位密钥，用于配置所有具有该密钥的页面的内存权限。

+   全局：这与 TLB（转换后备缓冲区，MMU 用于虚拟地址到物理地址转换的缓存）如何缓存页面的翻译有关，设置此位意味着页面不会在上下文切换时从 TLB 中刷新，这通常在内核页面上启用以减少 TLB 未命中。

+   页面属性表（PAT）：如果设置，MMU 在确定页面的‘内存类型’时应查阅页面属性表 MSR，例如，此页面是否为‘不可缓存’、‘写穿透’或其他一些内存类型之一。

+   脏：这个位类似于已访问位，如果此页面被写入，则 MMU 将设置该位，并且必须由操作系统复位。

这些都实际上不会影响地址转换本身，但是内存保护键的配置可能意味着由该条目引用的页面的预期内存访问权限可能比条目本身编码的严格。

与前几个级别不同，由于这是最后一级，所以条目保存了与正在转换的虚拟地址相关联的页面的最终物理地址。一旦您应用位掩码以获取物理地址字节并添加原始虚拟地址的最后 12 位（页面内部的偏移量），您就获得了物理地址！

希望这听起来并不那么糟糕，页面遍历的一般情况只需几个步骤：

+   通过移位地址和应用位掩码将虚拟地址转换为索引和页面偏移量

+   读取 `cr3` 以获取 PGD 的物理地址。

+   对于每个级别直到最后一个：

    +   使用从虚拟地址计算出的索引来知道从页面表中使用哪个条目

    +   将位掩码应用于条目，以获取下一级的物理地址。

+   在最终级别上，再次找到与虚拟地址相对应的条目。

+   将位掩码应用于虚拟地址，以获取与虚拟地址相关联的页面的物理地址。

+   将虚拟地址中的偏移量添加到页面的物理地址，以获得页面内部的偏移量。

+   完成！

## 巨化

如前所述，PUD 和 PMD 的先前图表是针对普通情况的，即当页大小位未设置时。

那么，当设置时呢？

当设置时，这实际上告诉 MMU，收拾好了，我们在这里完成了，不要继续遍历页面，当前条目保存了我们正在寻找的页面的物理地址。

但不仅仅是这样，当页面大小位被设置时，条目中页面的物理地址不是正常的 4KB（0x1000 字节）页面，而是‘巨大页面’，有两种变体：1GB 巨大页面和 2MB 巨大页面。

当 PUD 条目设置了页面大小位时，它指的是 1GB 的巨大页面，而当 PMD 设置了页面大小位时，它指的是 2MB 的巨大页面。

但是 1GB 和 2MB 的数字是从哪里来的呢？

每个页表级别最多可容纳 512 个条目，这意味着 PT 最多可以指向 512 页，`512 * 4KB = 2MB`。因此，PMD 级别的巨大页面有效地意味着该条目指向与完整 PT 大小相同的页面。

将此扩展到 PUD 级别，我们再次乘以 512 得到具有完整 PT 的完整 PMD 的大小：`512 * 512 * 4KB = 1GB`。

### 巨大页面 PUD

```
~ PUD Entry, Page Size set ~                                     Present ─────┐
                                                             Read/Write ─────┐|
                                                       User/Supervisor ─────┐||
                                                   Page Write Through ─────┐|||
                                                Page Cache Disabled ─────┐ ||||
                                                          Accessed ─────┐| ||||
                                                            Dirty ─────┐|| ||||
┌─── NX                                                Page Size ─────┐||| ||||
|┌───┬─── Memory Protection Key                         Global ─────┐ |||| ||||
||   |┌──────┬─── Ignored                             Ignored ───┬─┐| |||| ||||
||   ||      | ┌─── Reserved           Page Attribute Table ───┐ | || |||| ||||
||   ||      | |┌────────────────────────┐┌───────────────────┐| | || |||| ||||
||   ||      | || 1GB Page Physical Addr ||      Reserved     || | || |||| ||||
||   ||      | ||                        ||                   || | || |||| ||||
XXXX XXXX XXXX 0XXX XXXX XXXX XXXX XXXX XX00 0000 0000 0000 000X XXXX 1XXX XXXX
       56        48        40        32        24        16         8         0 
```

当页面大小位被设置时，注意到 PUD 条目看起来更像是 PT 条目而不是普通的 PUD 条目，这是有道理的，因为它也是指向页面而不是页表。

与 PT 条目有一些区别：

1.  页面大小位就是 PT 上的 Page Attribute Table (PAT) 位所在的位置，因此 PAT 位被重新定位到位 12。

1.  1GB 巨大页面的物理地址需要在物理内存中具有 1GB 对齐，这就是新的保留位存在以及为什么位 12 能够被重新用作 PAT 位的原因。

总的来说，这里没有太多新东西，处理巨大页面时唯一的其他区别实际上是需要对地址应用不同的位掩码以获取页面的物理地址位，此外，1GB 对齐意味着在计算页面内虚拟地址的物理地址时，我们需要使用基于 1GB 对齐而不是 4KB 对齐的掩码。

### 巨大页面 PMD

```
~ PMD Entry, Page Size set ~                                     Present ─────┐
                                                             Read/Write ─────┐|
                                                       User/Supervisor ─────┐||
                                                   Page Write Through ─────┐|||
                                                Page Cache Disabled ─────┐ ||||
                                                          Accessed ─────┐| ||||
                                                            Dirty ─────┐|| ||||
┌─── NX                                                Page Size ─────┐||| ||||
|┌───┬─── Memory Protection Key                         Global ─────┐ |||| ||||
||   |┌──────┬─── Ignored                             Ignored ───┬─┐| |||| ||||
||   ||      | ┌─── Reserved         Page Attribute Table ─────┐ | || |||| ||||
||   ||      | |┌───────────────────────────────────┐┌────────┐| | || |||| ||||
||   ||      | ||     2MB Page Physical Address     ||Reserved|| | || |||| ||||
||   ||      | ||                                   ||        || | || |||| ||||
XXXX XXXX XXXX 0XXX XXXX XXXX XXXX XXXX XXXX XXXX XXX0 0000 000X XXXX 1XXX XXXX
       56        48        40        32        24        16         8         0 
```

这与设置了页面大小位的 PUD 条目非常相似，唯一变化的是，由于此级别的 2MB 页面的对齐更小，因此设置的保留位较少。

2MB 对齐意味着在巨大页面内的偏移量应使用基于 2MB 对齐的掩码来计算。

## 开始遍历

上一节有很多图表，这一节让我们看看如何在 gdb 中手动执行页面遍历。

### 准备工作

使用启动的虚拟机和附加了 gdb 的情况下，我首先会选择一个地址进行页面遍历，作为示例，我将使用内核中运行时的当前堆栈指针：

```
gef➤  p/x $rsp
$42 = 0xffffffff88c07da8 
```

现在我们有了要遍历的地址，让我们也从 `cr3` 中获取 PGD 的物理地址：

```
gef➤  p/x $cr3 & ~0xfff
$43 = 0x10d664000 
```

我将使用这个小小的 Python 函数来从虚拟地址中提取页表偏移量：

```
def get_virt_indicies(addr):
    pageshift = 12
    addr = addr >> pageshift
    pt, pmd, pud, pgd = (((addr >> (i*9)) & 0x1ff) for i in range(4))
    return pgd, pud, pmd, pt 
```

这样输出：

```
In [2]: get_virt_indicies(0xffffffff88c07da8)
Out[2]: (511, 510, 70, 7) 
```

### PGD

我们根据虚拟地址得到的 PGD 的索引是 511，将 511 乘以 8 将使我们得到 PGD 条目的字节偏移量，该偏移量是我们虚拟地址的 PGD 条目起始位置处：

```
gef➤  p/x 511*8
$44 = 0xff8 
```

将该偏移量添加到 PGD 的物理地址上得到 PGD 条目的物理地址：

```
gef➤  p/x 0x10d664000+0xff8
$45 = 0x10d664ff8 
```

读取该地址处的物理内存得到 PGD 条目本身：

```
gef➤  monitor xp/gx 0x10d664ff8
000000010d664ff8: 0x0000000008c33067 
```

看起来条目的最后三位（存在、用户和可写）已设置，并且顶部位（NX）未设置，这意味着到目前为止与此虚拟地址关联的页面的权限没有任何限制。

对位 [12, 51) 进行掩码运算得到 PUD 的物理地址：

```
gef➤  p/x 0x0000000008c33067 & ~((1<<12)-1) & ((1ull<<51) - 1)
$46 = 0x8c33000 
```

### PUD

根据虚拟地址得到的 PUD 索引是 510，将 510 乘以 8 将让我们得到 PUD 的字节偏移量，即我们的虚拟地址所在的 PUD 条目开始的地方：

```
gef➤  p/x 510*8
$47 = 0xff0 
```

将该偏移添加到 PUD 的物理地址上，我们就得到了 PUD 条目的物理地址：

```
gef➤  p/x 0x8c33000+0xff0
$48 = 0x8c33ff0 
```

读取该地址处的物理内存将得到 PUD 条目本身：

```
gef➤  monitor xp/gx 0x8c33ff0
0000000008c33ff0: 0x0000000008c34063 
```

在此级别，我们需要开始注意大小位（位 7），因为如果是 1GB 页，我们将在此处停止分页。

```
gef➤  p/x 0x0000000008c34063 & (1<<7)
$49 = 0x0 
```

看起来此条目未设置，因此我们将继续进行分页。

还要注意 PUD 条目结尾是 0x3 而不是像前一级别那样是 0x7，底部两位（存在、可写）仍然被设置，但第三位，用户位现在未设置。这意味着属于此 PUD 条目的页面的用户模式访问将因访问的权限检查失败而导致页面故障。

NX 位仍未设置，因此属于此 PUD 的页面仍可执行。

对位 [12, 51) 进行屏蔽得到了 PMD 的物理地址：

```
gef➤  p/x 0x0000000008c34063 & ~((1ull<<12)-1) & ((1ull<<51)-1)
$50 = 0x8c34000 
```

### PMD

根据虚拟地址得到的 PMD 索引是 70，将 70 乘以 8 将让我们得到 PMD 的字节偏移量，即我们的虚拟地址所在的 PMD 条目开始的地方：

```
gef➤  p/x 70*8
$51 = 0x230 
```

将该偏移添加到 PMD 的物理地址上，我们就得到了 PMD 条目的物理地址：

```
gef➤  p/x 0x8c34000+0x230
$52 = 0x8c34230 
```

读取该地址处的物理内存将得到 PMD 条目本身：

```
gef➤  monitor xp/gx 0x8c34230
0000000008c34230: 0x8000000008c001e3 
```

同样，在此级别，我们需要注意大小位，因为如果是 2MB 页，我们将在此处停止分页。

```
gef➤  p/x 0x8000000008c001e3 & (1<<7)
$53 = 0x80 
```

看起来我们的虚拟地址指向一个 2MB 巨大页面！因此，此 PMD 条目中的物理地址是该巨大页面的物理地址。

此外，查看权限位，看起来页面仍然存在且可写，并且用户位仍未设置，因此此页面仅从超级用户模式（ring-0）可访问。

与前几个级别不同，最高位，NX 位，被设置为 1：

```
gef➤  p/x 0x8000000008c001e3 & (1ull<<63)
$54 = 0x8000000000000000 
```

因此，这个巨大页面不是可执行内存。

对位 [21:51) 进行位掩码操作将得到巨大页面的物理地址：

```
gef➤  p/x 0x8000000008c001e3 & ~((1ull<<21)-1) & ((1ull<<51)-1)
$56 = 0x8c00000 
```

现在我们需要根据 2MB 页对齐来对虚拟地址应用位掩码以获得对巨大页面的偏移量。

2MB 等于 `1<<21`，因此应用位掩码 `(1ull<<21)-1` 将得到偏移量：

```
gef➤  p/x 0xffffffff88c07da8 & ((1ull<<21)-1)
$57 = 0x7da8 
```

现在将此偏移添加到 2MB 巨大页面的基地址上将得到与我们起始的虚拟地址相关联的物理地址：

```
gef➤  p/x 0x8c00000 + 0x7da8
$58 = 0x8c07da8 
```

看起来虚拟地址：`0xffffffff88c07da8` 的物理地址是：`0x8c07da8`！

### 检查自己

有几种方法可以测试我们是否正确地进行了分页，一个简单的检查是只需转储虚拟地址和物理地址处的内存并进行比较，如果它们看起来相同，我们可能是正确的：

物理：

```
gef➤  monitor xp/10gx 0x8c07da8
0000000008c07da8: 0xffffffff810effb6 0xffffffff88c07dc0
0000000008c07db8: 0xffffffff810f3685 0xffffffff88c07de0
0000000008c07dc8: 0xffffffff8737dce3 0xffffffff88c3ea80
0000000008c07dd8: 0xdffffc0000000000 0xffffffff88c07e98
0000000008c07de8: 0xffffffff8138ab1e 0x0000000000000000 
```

虚拟：

```
gef➤  x/10gx 0xffffffff88c07da8
0xffffffff88c07da8:	0xffffffff810effb6	0xffffffff88c07dc0
0xffffffff88c07db8:	0xffffffff810f3685	0xffffffff88c07de0
0xffffffff88c07dc8:	0xffffffff8737dce3	0xffffffff88c3ea80
0xffffffff88c07dd8:	0xdffffc0000000000	0xffffffff88c07e98
0xffffffff88c07de8:	0xffffffff8138ab1e	0x0000000000000000 
```

看起来不错！

另一种检查方法是使用 QEMU Monitor 提供的 `monitor gva2gpa`（客户虚拟地址到客户物理地址）命令：

```
gef➤  monitor gva2gpa 0xffffffff88c07da8
gpa: 0x8c07da8 
```

假设 QEMU 正确执行地址转换（这可能是一个合理的假设），那么看起来我们已经双重确认我们的页表遍历成功了！

## 结束

希望到此结束时，你已经对 x86_64 系统上的分页是如何工作有了相当扎实的理解。我希望在这篇文章中包含了大量信息，所以需要一些思考来确定如何组织所有内容，但我仍然不确定这是否是一个很好的方法。

无论如何，我觉得分页挺不错的，我认为这是一种一旦掌握就能轻松掌握的东西，但要达到那个水平可能需要一些时间和在 gdb 中一些折腾。

我还想提一下，我为本文制作的各种页表条目的图表灵感来自于 [blink](https://github.com/jart/blink/) 项目的文档：[blink/machine.h](https://github.com/jart/blink/blob/46d82a0ced97c0df1fc645c5d81a88f0d142fbfd/blink/machine.h#L61)。

感谢阅读！