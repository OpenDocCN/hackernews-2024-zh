<!--yml

category: 未分类

date: 2024-05-27 13:32:10

-->

# Oracle共享池内部：堆转储中分配的块状态指示器 | Tanel Poder Consulting

> 来源：[https://tanelpoder.com/posts/oracle-shared-pool-chunk-status-indicators-in-heapdump/](https://tanelpoder.com/posts/oracle-shared-pool-chunk-status-indicators-in-heapdump/)

<main>

随着时间的推移，Oracle一直在每个已分配的堆内存块中添加更多的上下文信息，以便更容易看到堆内存的用途。这种工具也用于私有（PGA、UGA等）堆，但本文仅关注共享池堆。

过去的一些例子包括：

1.  [添加库缓存对象哈希值到已分配的共享池块名称](/2010/11/04/a-little-new-feature-for-shared-pool-geeks/) (2010)

1.  上述技术也适用于其他一些分配类型，例如用于“会话页面”的`KKSSP^NNN`分配，这些页面保存各种小内存结构，例如访问库缓存对象时会话必须分配的库缓存锁（在较旧的DB版本中还有固定）。在这种情况下，NNN并不显示某些哈希值，而是分配会话的SID。

1.  另外，您可以使用共享池转储来列出堆LRU替换列表末尾的*周期*结束后的[SEPARATOR标记共享池堆转储](/posts/oracle-shared-pool-list-chunk-position-in-lru-list/) (2020)。这将告诉您哪些现有游标已被执行多次（至少固定了3次-一次是为了将对象加载到缓存中+至少2次执行）。

新的Oracle版本在堆转储中打印出更多关于块状态指示器的信息，例如：

```
HEAP DUMP heap name="sga heap(7,0)"  desc=0x601f67d8
  Chunk        16e5a4b88 sz=     4096    recrPR001 "KGLH0^b9ac7ac5 "  B07:29:52
  Chunk        0fffb0df8 sz=     4096    recrUT001 "PLDIA^b9ac7ac5 "  B07:29:52
  Chunk        16e5a4b88 sz=     4096    recrPR001 "KGLH0^b9ac7ac5 "  B07:29:52
SEPARATOR
HEAP DUMP heap name="sga heap(7,3)"  desc=0x601fb0e0
  Chunk        0fffabdf8 sz=     4096    recrPC001 "KGLS^b9ac7ac5  "
  Chunk        0fffacdf8 sz=     4096    recrPC001 "PLMCD^b9ac7ac5 "
  Chunk        0fffaddf8 sz=     4096    freeableU "PLDIA^b9ac7ac5 "  ds=0x16e5a5108
  Chunk        0fffaedf8 sz=     4096    freeableU "PLDIA^b9ac7ac5 "  ds=0x16e5a5108
  Chunk        0fffafdf8 sz=     4096    freeableU "PLDIA^b9ac7ac5 "  ds=0x16e5a5108
  Chunk        0fffb0df8 sz=     4096    recrUT001 "PLDIA^b9ac7ac5 "  B07:29:52

```

可重建的块有2个字符表示其状态。

第一个字符：

+   **P** - chunk is pinned

+   **U** - chunk is unpinned

第二个字符：

+   **C** - chunk is just created (pinned only once for KGL parent objects, twice for child objects)

+   **T** - chunk is in the *transient* end of the LRU list (loaded and used only once, heap is pinned only twice)

+   **R** - chunk is in the *recurrent* end of the LRU list (loaded and used twice or more, heap is pinned at least 3 times)

在`recrPC001`中的三位数是分配容器/PDB的CON_ID，如果您正在使用多租户数据库。

您可能想知道上面的C和T之间有什么区别。看起来Oracle立即对像库缓存对象句柄（KGLHD - 任何库缓存对象结构的入口点，数据字典缓存对象KQR PO/SO）之类的对象使用瞬态标志（T），但对于实际的库缓存对象“有效载荷”，例如游标、PL/SQL对象，它首先使用（C）标志，然后如果执行超过一次则更改为（R）。

查询仅执行一次：

```
HEAP DUMP heap name="sga heap(6,0)"  desc=0x601e20b0
  Chunk        17d20e0b8 sz=     4096    recrUC001 "KGLH0^cae3710  "  C23:19:04 
  Chunk        17d320070 sz=     4096    recrPC001 "KGLH0^cae3710  "   
  Chunk        0aa4b9300 sz=     4096    recrUC001 "SQLA^cae3710   "  C23:19:02 
  Chunk        17d20e0b8 sz=     4096    recrUC001 "KGLH0^cae3710  "  C23:19:04 
SEPARATOR
HEAP DUMP heap name="sga heap(6,3)"  desc=0x601e69b8
  Chunk        0aa4b7300 sz=     4096    freeableU "SQLA^cae3710   "  ds=0x17d3208e8
  Chunk        0aa4b8300 sz=     4096    freeableU "SQLA^cae3710   "  ds=0x17d3208e8
  Chunk        0aa4b9300 sz=     4096    recrUC001 "SQLA^cae3710   "  C23:19:02 
SEPARATOR

```

相同的查询执行两次：

```
HEAP DUMP heap name="sga heap(6,0)"  desc=0x601e20b0
  Chunk        17d20e0b8 sz=     4096    recrPR001 "KGLH0^cae3710  "  C23:19:04 
  Chunk        17d320070 sz=     4096    recrPC001 "KGLH0^cae3710  "   
  Chunk        17d20e0b8 sz=     4096    recrPR001 "KGLH0^cae3710  "  C23:19:04 
SEPARATOR
  Chunk        0aa4b9300 sz=     4096    recrUR001 "SQLA^cae3710   "  C23:20:11 
HEAP DUMP heap name="sga heap(6,3)"  desc=0x601e69b8
  Chunk        0aa4b7300 sz=     4096    freeableU "SQLA^cae3710   "  ds=0x17d3208e8
  Chunk        0aa4b8300 sz=     4096    freeableU "SQLA^cae3710   "  ds=0x17d3208e8
  Chunk        0aa4b9300 sz=     4096    recrUR001 "SQLA^cae3710   "  C23:20:11 
SEPARATOR

```

在上述输出中，您可以看到`recrUC001`如何变成`recrUR001`，并且该块被移动到LRU列表中的`SEPARATOR`特殊标记下面。

这里有几个库缓存对象句柄（KGLHD）的示例，这些句柄在分配时甚至没有包含对象名称哈希值（你可以使用`v$db_object_cache.addr`或`v$sql.address/child_address`来找到正确的对象）在堆转储中：

```
  Chunk        14779e1a8 sz=      504    freeable  "KGLDA          "
  Chunk        14779e3a0 sz=      560    recrPT001 "KGLHD          "
  Chunk        14779e5d0 sz=      816    recrPT001 "KGLHD          "
  Chunk        14779e900 sz=      816    recrPT001 "KGLHD          "
  Chunk        14779ec30 sz=      504    freeable  "KGLDA          "
  Chunk        14779ee28 sz=      560    recrPT001 "KGLHD          "

```

对于这种分配，Oracle似乎立即将块标记为（R）或（T），而根本不使用（C）标志。

如今，Oracle试图使库缓存的堆分配使用标准化/固定大小的单个块（*_kgl_fixed_extents* = 4096），以减少堆碎片化。因此，如果你的游标（或PL/SQL对象）需要超过4kB的内存来分配其堆，则会看到许多`freeable`块：

```
$ grep -iE "HEAP DUMP|SEPARATOR|ED8894E3" LIN19M_ora_46680_0014.trc
HEAP DUMP heap name="sga heap"  desc=0x6017a390
HEAP DUMP heap name="sga heap(1,0)"  desc=0x6017bce8
  Chunk        13deffa20 sz=     4096    freeableU "KGLH0^ed8894e3 "  ds=0x13e3d9130
  Chunk        13df038a0 sz=     4096    recrUC001 "KGLH0^ed8894e3 "  B07:40:21 
  Chunk        13e3d94f0 sz=     4096    recrPC001 "KGLH0^ed8894e3 "   
  Chunk        09f576948 sz=     4096    recrUC001 "SQLA^ed8894e3  "  B07:40:21 
  Chunk        13df038a0 sz=     4096    recrUC001 "KGLH0^ed8894e3 "  B07:40:21 
SEPARATOR
HEAP DUMP heap name="sga heap(1,3)"  desc=0x601805f0
  Chunk        09f53f948 sz=     4096    freeableU "SQLA^ed8894e3  "  ds=0x13e3d9d48
  Chunk        09f540948 sz=     4096    freeableU "SQLA^ed8894e3  "  ds=0x13e3d9d48
  Chunk        09f541948 sz=     4096    freeableU "SQLA^ed8894e3  "  ds=0x13e3d9d48
  Chunk        09f542948 sz=     4096    freeableU "SQLA^ed8894e3  "  ds=0x13e3d9d48
  Chunk        09f543948 sz=     4096    freeableU "SQLA^ed8894e3  "  ds=0x13e3d9d48
[...tens of rows removed...]
  Chunk        09f56d948 sz=     4096    freeableU "SQLA^ed8894e3  "  ds=0x13e3d9d48
  Chunk        09f56e948 sz=     4096    freeableU "SQLA^ed8894e3  "  ds=0x13e3d9d48
  Chunk        09f573948 sz=     4096    freeableU "SQLA^ed8894e3  "  ds=0x13e3d9d48
  Chunk        09f574948 sz=     4096    freeableU "SQLA^ed8894e3  "  ds=0x13e3d9d48
  Chunk        09f575948 sz=     4096    freeableU "SQLA^ed8894e3  "  ds=0x13e3d9d48
  Chunk        09f576948 sz=     4096    recrUC001 "SQLA^ed8894e3  "  B07:40:21 
SEPARATOR

```

这些*可释放*块未链接到共享池管理器搜索的LRU列表中。Oracle不会自行释放随机的可释放块 - 而且它根本看不到它们，因为它们未链接到LRU列表中。

共享池LRU列表扫描必须找到一个未固定的*可重建*块（分配堆中的第一个块），并首先释放这个主导的可重建块（`ds=0x13e3d9d48`指向堆的开头）。然后继续释放这个堆的其余块，它会遍历连接到主导块的任何进一步分配的链表（标记为可释放），并作为结果释放这些块。当堆管理器发现其主导的可重建块未固定时，它会释放整个子堆 - 没有必要仅释放100kB SQL Area堆的开头4kB并使其余部分处于部分分配的不可用状态。

</main>
