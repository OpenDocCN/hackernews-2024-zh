<!--yml
category: 未分类
date: 2024-05-27 13:32:10
-->

# Oracle Shared Pool Internals: Allocated Chunk Status Indicators in Heap Dumps | Tanel Poder Consulting

> 来源：[https://tanelpoder.com/posts/oracle-shared-pool-chunk-status-indicators-in-heapdump/](https://tanelpoder.com/posts/oracle-shared-pool-chunk-status-indicators-in-heapdump/)

<main>

Over time, Oracle has been adding more contextual information into each allocated heap memory chunk, to make it easier to see what for your heap memory is used. This instrumentation is used for private (PGA,UGA,etc) heaps too, but this article focuses only on shared pool heaps.

A few examples from past are:

1.  [Library cache object hash values added to allocated shared pool chunk names](/2010/11/04/a-little-new-feature-for-shared-pool-geeks/) (2010)
2.  The above technique is used for some other allocation types too, like `KKSSP^NNN` allocations that are used for “session pages” that hold various tiny memory structures like library cache lock (and pin in older DB versions) that a session must allocate when accessing library cache objects. The NNN is in this case is not showing some hash value, but the SID of the allocating session.
3.  Additionally, you can use shared pool dumps for listing chunk location in the *recurrent* end of the heap LRU replacement list after the [SEPARATOR marker in shared pool heap dumps](/posts/oracle-shared-pool-list-chunk-position-in-lru-list/) (2020). That would tell you which existing cursors have been executed more than once (pinned at least 3 times - once for loading the object into cache + at least 2 executions).

Newer Oracle versions print out even more info about chunk status indicators in heap dumps, for example:

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

Recreatable chunks have 2 charaters indicating their status.

First character:

*   **P** - chunk is pinned
*   **U** - chunk is unpinned

Second character:

*   **C** - chunk is just created (pinned only once for KGL parent objects, twice for child objects)
*   **T** - chunk is in the *transient* end of the LRU list (loaded and used only once, heap is pinned only twice)
*   **R** - chunk is in the *recurrent* end of the LRU list (loaded and used twice or more, heap is pinned at least 3 times)

The 3-digit number in `recrPC001` is the CON_ID of the allocating container/PDB, if you’re using multitenant database.

You might wonder what’s the difference between C and T above. It looks like Oracle immediately uses the transient flag (T) for objects like library cache object handle (KGLHD - the entry point to any library cache object structure, data dictionary cache objects KQR PO/SO), but for actual library cache object “payload”, like cursors, PL/SQL objects, it first uses the (C) flag, which then gets changed to (R) if executed for more than once.

Query executed only once:

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

The same query executed twice:

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

In the above output, you see how `recrUC001` turned into `recrUR001` and the chunk was moved below the `SEPARATOR` special marker in the LRU list.

Here’s an example of a few library cache object handle (KGLHD) objects, that don’t even have the object name hash value included in the allocations (you can find the right ones using `v$db_object_cache.addr` or `v$sql.address/child_address`) in a heapdump:

```
  Chunk        14779e1a8 sz=      504    freeable  "KGLDA          "
  Chunk        14779e3a0 sz=      560    recrPT001 "KGLHD          "
  Chunk        14779e5d0 sz=      816    recrPT001 "KGLHD          "
  Chunk        14779e900 sz=      816    recrPT001 "KGLHD          "
  Chunk        14779ec30 sz=      504    freeable  "KGLDA          "
  Chunk        14779ee28 sz=      560    recrPT001 "KGLHD          "

```

For such allocations, Oracle seems to immediately mark chunks as (R) or (T) and not use the (C) flag at all.

Nowadays, Oracle tries make library cache’s heap allocations use standardized/fixed sizes of individual chunks (*_kgl_fixed_extents* = 4096) to reduce heap fragmentation. Thus, you’ll see a lot of `freeable` chunks if your cursor (or PL/SQL object) requires more than 4kB of memory for its heaps:

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

These *freeable* chunks are not linked into the LRU list that shared pool manager searches through. Oracle doesn’t free random freeable chunks on their own - and it doesn’t see them anyway as they are not linked into the LRU list.

Shared pool LRU list scan must find an unpinned *recreatable* chunk (the first chunk in an allocated heap) and it frees this leading recreatable chunk first (the `ds=0x13e3d9d48` points to the beginning of the heap). Then it goes on to freeing the remaining chunks of this heap, it will walk through a linked list of any further allocations “chained” to the leading chunk (marked as freeable) and frees these as a result. The heap manager frees entire subheaps when it finds its leading recreatable chunk to be unpinned - there’s no point in freeing only the leading 4kB of a 100kB SQL Area heap and leaving the rest of it in some partially allocated unusable state.

</main>