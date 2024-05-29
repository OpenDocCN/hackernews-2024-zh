<!--yml
category: 未分类
date: 2024-05-27 15:04:28
-->

# Chris's Wiki :: blog/linux/ZFSGlobalZILInformation

> 来源：[https://utcc.utoronto.ca/~cks/space/blog/linux/ZFSGlobalZILInformation](https://utcc.utoronto.ca/~cks/space/blog/linux/ZFSGlobalZILInformation)

The [ZFS Intent Log (ZIL)](/~cks/space/blog/solaris/ZFSTXGsAndZILs) is effectively ZFS's version of a filesystem journal, writing out hopefully brief records of filesystem activity to make them durable on disk before their full version is committed to the ZFS pool. What the ZIL is doing and how it's performing can be important for the latency (and thus responsiveness) of various operations on a ZFS filesystem, since operations like `fsync()` on an important file must wait for the ZIL to write out (*commit*) their information before they can return from the kernel. On Linux, [OpenZFS](https://openzfs.org/) exposes global information about the ZIL in `/proc/spl/kstat/zfs/zil`, but this information can be hard to interpret without some knowledge of ZIL internals.

(In OpenZFS 2.2 and later, each dataset also has per-dataset ZIL information in its kstat file, /proc/spl/kstat/zfs/<pool>/objset-0xXXX, for some hexadecimal '0xXXX'. There's no overall per-pool ZIL information the way there is a global one, but for most purposes you can sum up the ZIL information from all of the pool's datasets.)

The basic background here is [the flow of activity in the ZIL](/~cks/space/blog/solaris/ZFSZILActivityFlow) and also the comments in [zil.h](https://github.com/openzfs/zfs/blob/master/include/sys/zil.h) about the members of the `zil_stats` struct.

The (ZIL) data you can find in the "`zil`" file (and the per-dataset kstats in OpenZFS 2.2 and later) is as follows:

*   `zil_commit_count` counts how many times a ZIL commit has been requested through things like `fsync()`.
*   `zil_commit_writer_count` counts how many times the ZIL has actually committed. More than one commit request can be merged into the same ZIL commit, if two people `fsync()` more or less at the same time.
*   `zil_itx_count` counts how many *intent transactions* (itxs) have been written as part of ZIL commits. Each separate operation (such as a `write()` or a file rename) gets its own separate transaction; these are aggregated together into *log write blocks* (lwbs) when a ZIL commit happens.

When ZFS needs to record file data into the ZIL, it has three options, which it calls '`indirect`', '`copied`', and '`needcopy`' in ZIL metrics. Large enough amounts of file data are handled with an *indirect* write, [which writes the data to its final location in the regular pool](/~cks/space/blog/solaris/ZFSWritesAndZIL); the ZIL transaction only records its location, hence 'indirect'. In a *copied* write, the data is directly and immediately put in the ZIL transaction (itx), even before it's part of a ZIL commit; this is done if ZFS knows that the data is being written synchronously and it's not large enough to trigger an indirect write. In a *needcopy* write, the data just hangs around in RAM as part of ZFS's regular dirty data, and if a ZIL commit happens that needs that data, the process of adding its itx to the log write block will fetch the data from RAM and add it to the itx (or at least the lwb).

There are ZIL metrics about this:

*   `zil_itx_indirect_count` and `zil_itx_indirect_bytes` count how many indirect writes have been part of ZIL commits, and the total size of the indirect writes of file data (not of the 'itx' records themselves, per the comments in [zil.h](https://github.com/openzfs/zfs/blob/master/include/sys/zil.h)).

    Since these are indirect writes, the data written is not part of the ZIL (it's regular data blocks), although it is put on disk as part of a ZIL commit. However, unlike other ZIL data, the data written here would have been written even without a ZIL commit, as part of ZFS's regular transaction group commit process. A ZIL commit merely writes it out earlier than it otherwise would have been.

*   `zil_itx_copied_count` and `zil_itx_copied_bytes` count how many 'copied' writes have been part of ZIL commits and the total size of the file data written (and thus committed) this way.
*   `zil_itx_needcopy_count` and `zil_itx_needcopy_bytes` count how many 'needcopy' writes have been part of ZIL commits and the total size of the file data written (and thus committed) this way.

A regular system using ZFS may have little or no 'copied' activity. Our NFS servers all have significant amounts of it, presumably because some NFS data writes are done synchronously and so this trickles through to the ZFS stats.

In a given pool, the ZIL can potentially be written to either the main pool's disks or to a separate log device (a *slog*, which can also be mirrored). The ZIL metrics have a collection of `zil_itx_metaslab_*` metrics about data actually written to the ZIL in either the main pool ('normal' metrics) or to a slog (the 'slog' metrics).

*   `zil_itx_metaslab_normal_count` counts how many ZIL *log write blocks* (not ZIL records, itxs) have been committed to the ZIL in the main pool. There's a corresponding 'slog' version of this and all further zil_itx_metaslab metrics, with the same meaning.
*   `zil_itx_metaslab_normal_bytes` counts how many bytes have been 'used' in ZIL log write blocks (for ZIL commits in the main pool). This is a rough representation of how much space the ZIL log actually needed, but it doesn't necessarily represent either the actual IO performed or the space allocated for ZIL commits.

    As I understand things, this size includes the size of the intent transaction records themselves and also the size of the associated data for 'copied' and 'needcopy' data writes (because these are written into the ZIL as part of ZIL commits, and so use space in log write blocks). It doesn't include the data written directly to the pool as 'indirect' data writes.

If you don't use a slog in any of your pools, the 'slog' versions of these metrics will all be zero. I think that if you have only slogs, the 'normal' versions of these metrics will all be zero.

In ZFS 2.2 and later, there are two additional statistics for both normal and slog ZIL commits:

*   `zil_itx_metaslab_normal_write` counts how many bytes have actually been written in ZIL log write blocks. My understanding is that this includes padding and unused space at the end of a log write block that can't fit another record.
*   `zil_itx_metaslab_normal_alloc` counts how many bytes of space have been 'allocated' for ZIL log write blocks, including any rounding up to block sizes, alignments, and so on. I think this may also be the logical size before any compression done as part of IO, although I'm not sure if ZIL log write blocks are compressed.

You can see some additional commentary on these new stats (and the code) in [the pull request](https://github.com/openzfs/zfs/pull/14863) and [the commit itself](https://github.com/openzfs/zfs/commit/b6fbe61fa6a75747d9b65082ad4dbec05305d496).

PS: OpenZFS 2.2 and later has a currently undocumented '`zilstat`' command, and its 'zilstat -v' output may provide some guidance on what ratios of these metrics the ZFS developers consider interesting. In its current state it will only work on 2.2 and later because it requires the two new stats listed above.

### Sidebar: Some typical numbers

Here is the "zil" file from [my office desktop](/~cks/space/blog/linux/WorkMachine2017), which has been up for long enough to make it interesting:

> ```
> zil_commit_count                4    13840
> zil_commit_writer_count         4    13836
> zil_itx_count                   4    252953
> zil_itx_indirect_count          4    27663
> zil_itx_indirect_bytes          4    2788726148
> zil_itx_copied_count            4    0
> zil_itx_copied_bytes            4    0
> zil_itx_needcopy_count          4    174881
> zil_itx_needcopy_bytes          4    471605248
> zil_itx_metaslab_normal_count   4    15247
> zil_itx_metaslab_normal_bytes   4    517022712
> zil_itx_metaslab_normal_write   4    555958272
> zil_itx_metaslab_normal_alloc   4    798543872
> 
> ```

With these numbers we can see interesting things, such as that the average number of ZIL transactions per commit is about 18 and that my machine has never done any synchronous data writes.

Here's an excerpt from one of [our Ubuntu 22.04 ZFS fileservers](/~cks/space/blog/linux/ZFSFileserverSetupIII):

> ```
> zil_commit_count                4    155712298
> zil_commit_writer_count         4    155500611
> zil_itx_count                   4    200060221
> zil_itx_indirect_count          4    60935526
> zil_itx_indirect_bytes          4    7715170189188
> zil_itx_copied_count            4    29870506
> zil_itx_copied_bytes            4    74586588451
> zil_itx_needcopy_count          4    1046737
> zil_itx_needcopy_bytes          4    9042272696
> zil_itx_metaslab_normal_count   4    126916250
> zil_itx_metaslab_normal_bytes   4    136540509568
> 
> ```

Here we can see the drastic impact of NFS synchronous writes (the significant 'copied' numbers), and also of large NFS writes in general (the high 'indirect' numbers). This machine has written many times more data in ZIL commits as 'indirect' writes as it has written to the actual ZIL.