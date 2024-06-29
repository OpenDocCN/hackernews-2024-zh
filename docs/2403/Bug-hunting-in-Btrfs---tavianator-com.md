<!--yml

category: 未分类

date: 2024-05-29 12:31:28

-->

# Btrfs的Bug追踪 - tavianator.com

> 来源：[https://tavianator.com/2024/btrfs_bug.html](https://tavianator.com/2024/btrfs_bug.html)

<main>

前几天我在 [`bfs`](../projects/bfs.html) 中实现了多线程的 `stat()` 调用。在运行了一些基准测试后，我发现了一些令人激动的事情：

```
$ tailfin run bench/bench.sh --build=main --default
...
bfs: error: bench/corpus/chromium/v8/src/heap/heap.cc: Structure needs cleaning 
```

如果你不熟悉，"structure needs cleaning" 是 `EUCLEAN` 的人类可读描述，通常表示文件系统损坏。担心情况严重，我检查了 `dmesg` 并看到

```
$ dmesg | tail
...
BTRFS critical (device dm-2): corrupted node, root=518 block=16438782945911875046 owner mismatch, have 4517169229596899607 expect [256, 18446744073709551360] 
```

我立即重启进入live环境并运行了 `btrfs check`，但令我惊讶的是，并没有发现错误。`btrfs scrub` 也没有发现任何损坏的证据。事实上，错误消息中的文件完全没有问题。无论问题是什么，它似乎已经自行消失了。

我在Google上搜索了那个 "corrupted node" 消息，发现最近也发生在了别人身上：[Linus Torvalds](https://lore.kernel.org/linux-btrfs/CAHk-=whNdMaN9ntZ47XRKP6DBes2E5w7fi-0U3H2+PS18p+Pzw@mail.gmail.com/)，就在合并Btrfs的一个pull请求后。这不是我和Linus第一次碰到相同的bug。我们的台式机都使用相同的CPU。上次这导致了我一次 [最喜欢的bug修复](https://git.kernel.org/pub/scm/linux/kernel/git/torvalds/linux.git/commit/?id=a51ab63b297ce9e26e3ffb9be896018a42d5f32f)。

但我和Linus之间的不同之处在于，我可以（半）可靠地重现这个错误。虽然不是每次，但可能有10%的时间我在大型目录树上并行运行 `bfs` 的 `stat()` 调用时，我会看到 "structure needs cleaning" 错误。我将我的 [发现](https://lore.kernel.org/linux-btrfs/20240206033807.15498-1-tavianator@tavianator.com/) 报告给了Btrfs邮件列表，尽管这引发了关于潜在原因的讨论，但我们并没有找到根本原因。

基于日志记录，Btrfs 开发人员认为问题涉及到 [`struct extent_buffer`](https://git.kernel.org/pub/scm/linux/kernel/git/torvalds/linux.git/tree/fs/btrfs/extent_io.h?id=a51ab63b297ce9e26e3ffb9be896018a42d5f32f#n76) 的分配及其内存页面。我并不是Btrfs内部专家，但我了解 `extent_buffer` 用于元数据I/O，即读取和写入 [B-tree](https://en.wikipedia.org/wiki/B-tree) 节点。`extent_buffer` 关联着一个 `struct folio`/`struct page` 数组（取决于内核版本），这些页面持有读取或写入的extent的实际内容。这种抽象有助于处理与系统页面大小不同的文件系统块大小。

分配`extent_buffer`及其页面比你想象的[更复杂](https://git.kernel.org/pub/scm/linux/kernel/git/torvalds/linux.git/tree/fs/btrfs/extent_io.c?id=a51ab63b297ce9e26e3ffb9be896018a42d5f32f#n3681)。每个`struct extent_buffer`包含一个`struct folio`数组，这些folios有一个私有引用指向`extent_buffer`。对于任何特定extent，应该始终只有一个`extent_buffer`。

为了减少分配路径上的锁定，`extent_buffer`和相应的folios是分开分配的，然后链接在一起，然后插入到以extent起始偏移量为键的基数树中。只有在插入时，如果发现其他人已经比我们提前分配了相同的extent，我们才会额外引用该`extent_buffer`并释放我们刚刚分配的（及其页面）。

这段代码路径最近经历了一些变动，包括[更改了竞争检测/处理策略](https://git.kernel.org/pub/scm/linux/kernel/git/torvalds/linux.git/commit/?id=09e6cef19c9fc0e10547135476865b5272aa0406)和[从`struct page`转换为`struct folio`](https://git.kernel.org/pub/scm/linux/kernel/git/torvalds/linux.git/commit/?id=082d5bb9b336d533b7b968f4f8712e7755a9876a)，因此有人怀疑可能潜伏了竞争或引用计数错误。但是额外的调试日志没有显示出问题。看起来`extent_buffer`和`folio`的引用计数都是正确的，但是内存本身却不是正确的磁盘内容，而是垃圾数据。Btrfs在读取这些块后会进行一些健全性检查，而这些检查失败了，导致了错误。但磁盘上的块本身是正确的，在下次读取时一切正常。

Linux内核中最好的bug调试工具之一是[`git bisect`](https://git-scm.com/docs/git-bisect)。不幸的是，像这样的二分法调试bug非常具有挑战性。对于间歇性问题，很容易知道何时标记一个提交为有问题，但更难确定一个提交是否好——也许bug只是还没有触发*而已*。你还需要一个已知好的提交来开始，但这很难找到。起初我以为bug是在Linux 6.7中新出现的，但我仍然能在6.6和6.5上重现它。Btrfs的开发人员建议我从6.0开始。

二分法调试也可能很危险，因为你会运行任意旧的中间开发周期内核提交，这些提交可能在官方发布版本中已经修复了bug。从虚拟机中进行会更安全，但到目前为止我无法在虚拟机上重现这个bug。因此我开始在我的实际桌面上进行二分法调试。

通常情况下，我会因为害怕在我日常使用的实际计算机上对文件系统损坏 bug 进行二分查找而望而却步。但这似乎不像是“真正”的损坏，再加上我有备份，所以我冒了险。不幸的是，经过几轮尝试后，我遇到了[实际的磁盘文件系统损坏](https://lore.kernel.org/linux-btrfs/CABg4E-=u7m_g3HCFUYHS-+RC==pefkUZXiTT2Aor86jruHSF9Q@mail.gmail.com/)。2073625 个不可纠正的读取错误对我来说是一个新记录，所以我太害怕继续进行二分查找了。我从备份中尽可能地恢复了数据，然后继续了我的生活。

几周后，Btrfs 开发者曲文若通过一些[额外的调试补丁](https://lore.kernel.org/linux-btrfs/c7241ea4-fcc6-48d2-98c8-b5ea790d6c89@gmx.com/)回复了我，我决定再试一次。我在裸机上运行了他的补丁，但是在应用调试补丁后无法重现这个 bug。调试竞争条件时就是这样的生活！

我还尝试在虚拟机中再次复现这个问题。我将其配置得更接近我的实际系统配置：多个带有[dm-crypt](https://wiki.archlinux.org/title/dm-crypt/Device_encryption)全磁盘加密的磁盘，在 Btrfs RAID 0 阵列中合并在一起。我将它们模拟成 NVME 驱动而不是 VirtIO。这样做起作用了：我现在可以在虚拟机中复现这个问题，尽管需要大约 30 次运行我的重现器，而不是实际生活中的大约 10 次。但这次即使在应用了调试补丁后我仍然可以复现这个问题。

经过几次运行和调整跟踪消息以便于交叉引用，我注意到了一些奇怪的地方。仅查看与“损坏”错误消息对应的跟踪行时，我看到了这个：

```
$ grep 'eb=15302115328' dmesg.log
BTRFS critical (device dm-0): corrupted leaf, root=258 block=15302115328 owner mismatch, have 13709377558419367261 expect [256, 18446744073709551360] eb=15302115328
iou-wrk--173727   15..... 2649295481us : alloc_extent_buffer: alloc_extent_buffer: alloc eb=15302115328 len=16384
kworker/-322      15..... 2649295735us : end_bbio_meta_read: read done, eb=15302115328 page_refs=3 eb level=0 fsid=b66a67f0-8273-4158-b7bf-988bb5683000
kworker/-5095     31..... 2649295941us : end_bbio_meta_read: read done, eb=15302115328 page_refs=8 eb level=0 fsid=b66a67f0-8273-4158-b7bf-988bb5683000 
```

唯一相关的操作似乎是分配这个`extent_buffer`，然后读取它……两次。我认为读取一次应该足够了，所以我更仔细地查看了实际读取它们的代码，并设法发现了这个错误。

就像多个线程可能同时尝试分配同一个`extent_buffer`一样，它们也可能同时尝试读取它。为了确保读取仅发生一次，使用了自定义的锁定协议：

```
int read_extent_buffer_pages(struct extent_buffer *eb, /* ... */)
{
    if (test_bit(EXTENT_BUFFER_UPTODATE, &eb->bflags))
        return 0;

    /* ... */

    /* Someone else is already reading the buffer, just wait for it. */
    if (test_and_set_bit(EXTENT_BUFFER_READING, &eb->bflags))
        goto done;

    /* ... */
    bbio = btrfs_bio_alloc(INLINE_EXTENT_BUFFER_PAGES,
                   REQ_OP_READ | REQ_META, eb->fs_info,
                   end_bbio_meta_read, eb);
    /* ... */
    btrfs_submit_bio(bbio, mirror_num);

done:
    if (wait == WAIT_COMPLETE) {
        wait_on_bit_io(&eb->bflags, EXTENT_BUFFER_READING, TASK_UNINTERRUPTIBLE);
        if (!test_bit(EXTENT_BUFFER_UPTODATE, &eb->bflags))
            return -EIO;
    }

    return 0;
}

static void end_bbio_meta_read(struct btrfs_bio *bbio)
{
    /* ... */
    bool uptodate = !bbio->bio.bi_status;
    /* ... */

    if (uptodate) {
        set_extent_buffer_uptodate(eb);
    } else {
        clear_extent_buffer_uptodate(eb);
        set_bit(EXTENT_BUFFER_READ_ERR, &eb->bflags);
    }

    /* ... */
    clear_bit(EXTENT_BUFFER_READING, &eb->bflags);
    smp_mb__after_atomic();
    wake_up_bit(&eb->bflags, EXTENT_BUFFER_READING);
    /* ... */
} 
```

逻辑设计的目的是这样的：

```
read_extent_buffer_pages() {
 if (test_bit(UPTODATE)) return 0;   if (test_and_set_bit(READING)) goto done;   btrfs_submit_bio();   done:
 wait_on_bit_io(READING); return 0; }   end_bbio_meta_read() {
 set_bit(UPTODATE); clear_bit(READING); wake_up_bit(READING); } 
```

```
read_extent_buffer_pages() {
 if (test_bit(UPTODATE)) return 0;   if (test_and_set_bit(READING)) goto done;   btrfs_submit_bio();   done:
 wait_on_bit_io(READING); return 0; } 
```

不幸的是，这个锁定协议存在一个错误：

```
if (test(UPTODATE))
 return 0;   if (test_and_set(READING))
 goto done;   btrfs_submit_bio();   done:
 wait_on(READING); return 0;   set(UPTODATE); clear(READING); wake_up(READING); 
```

```
if (test(UPTODATE))
 return 0;   if (test_and_set(READING))
 goto done;   btrfs_submit_bio();   done:
 wait_on(READING); return 0; 
```

```
if (test(UPTODATE))
 return 0;   if (test_and_set(READING))
 goto done;   btrfs_submit_bio();   done:
 wait_on(READING); return 0; 
```

如果第一个线程在第二个线程检查`UPTODATE`位和`READING`位之间启动并完成 I/O，第二个线程将开始完全不必要的读取，即使我们已经读取了 extent。更糟糕的是，由于`UPTODATE`位已经设置，第三个线程将在等待读取完成之前返回。然后该线程会认为可以安全地访问`extent_buffer`的页面，尽管它们目前正在进行 I/O，从而导致数据竞争！

解决方法是使用[双重检查锁定](https://en.wikipedia.org/wiki/Double-checked_locking)：在设置了`READING`位后，再次检查`UPTODATE`，以防止我们启动更多无用（甚至有害）的 I/O。

```
if (test(UPTODATE))
 return 0;   if (test_and_set(READING))
 goto done;   if (test(UPTODATE))
 goto wake;   btrfs_submit_bio();   done:
 wait_on(READING); return 0;   end_bbio:
 set(UPTODATE); wake:
 clear(READING); wake_up(READING); 
```

```
if (test(UPTODATE))
 return 0;   if (test_and_set(READING))
 goto done;   if (test(UPTODATE))
 goto wake;   btrfs_submit_bio();   done:
 wait_on(READING); return 0;   end_bbio:
 set(UPTODATE); wake:
 clear(READING); wake_up(READING); 
```

```
if (test(UPTODATE))
 return 0;   if (test_and_set(READING))
 goto done;   if (test(UPTODATE))
 goto wake;   btrfs_submit_bio();   done:
 wait_on(READING); return 0;   end_bbio:
 set(UPTODATE); wake:
 clear(READING); wake_up(READING); 
```

我提交了[一个补丁](https://lore.kernel.org/linux-btrfs/1ca6e688950ee82b1526bb3098852af99b75e6ba.1710551459.git.tavianator@tavianator.com/)来添加这个缺失的检查，以及几个[清理补丁](https://lore.kernel.org/linux-btrfs/cover.1710769876.git.tavianator@tavianator.com/T/)。通过这个修复，我可以在过夜运行重现器时不再触发任何错误。

也许你会感到惊讶，这种竞争并不无害。毕竟，我们正在将某些磁盘块读入内存，而这些内存已经包含了同一磁盘块的内容。从技术上讲，同时读取和写入内存仍然属于数据竞争，即使写入不修改内存，但通常在实践中是不可观察的。

```
uninitialized memory uninitialized memory uninitialized memory uninitialized memory uninitialized memory 
```

```
metadata metadata metadata metadata metadata metadata metadata metadata metadata metadata metadata metad 
```

```
metadata metadata metadata metadata metadata metadata metadata metadata metadata metadata metadata metad 
```

我能够观察到的原因已经在上面提到过：全盘加密。当 dm-crypt 处理磁盘读取时，它会将*加密*内容读入内存，然后就地解密。

```
uninitialized memory uninitialized memory uninitialized memory uninitialized memory uninitialized memory 
```

```
encrypted gibberish encrypted gibberish encrypted gibberish encrypted gibber encrypted gibberish encrypt 
```

```
metadata metadata metadata metadata metadata metadata metadata metadata metadata metadata metadata metad 
```

```
encrypted gibberish encrypted gibberish encrypted gibberish encrypted gibber encrypted gibberish encrypt 
```

```
metadata metadata metadata metadata metadata metadata metadata metadata metadata metadata metadata metad 
```

很明显，这会导致问题：有时竞争线程会看到有效的元数据，但有时会看到看似完全随机的加密字节。我甚至捕获了一条跟踪，在该错误信息中显示范围逐渐填充：

```
[  807.737576] BTRFS critical (device dm-0): corrupted node, root=266 block=3178470954007437077 owner mismatch, have 6155297726051334641 expect [256, 18446744073709551360] eb=ffff8ac915d022d0 start=12738904064
[  807.751913] BTRFS critical (device dm-0): corrupted leaf, root=266 block=12738904064 owner mismatch, have 6155297726051334641 expect [256, 18446744073709551360] eb=ffff8ac915d022d0 start=12738904064
[  807.800898] BTRFS critical (device dm-0): corrupted leaf, root=266 block=12738904064 owner mismatch, have 6155297726051334641 expect [256, 18446744073709551360] eb=ffff8ac915d022d0 start=12738904064 
```

在这里你可以看到*同一个块*从内部节点切换到叶子节点，并且块号在大约0.02秒内从一个随机数变为范围的开始。

</main>
