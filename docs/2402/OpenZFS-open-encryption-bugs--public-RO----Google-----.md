<!--yml

category: 未分类

date: 2024-05-29 13:05:06

-->

# OpenZFS开放加密漏洞（公共RO）- Google ドライブ

> 来源：[https://docs.google.com/spreadsheets/d/1OfRSXibZ2nIE9DGK6swwBZXgXwdCPKgp4SbPZwTexCg/htmlview](https://docs.google.com/spreadsheets/d/1OfRSXibZ2nIE9DGK6swwBZXgXwdCPKgp4SbPZwTexCg/htmlview)

Quick note: this was just a rapid pass over every open bug that looked like encryption was involved. It is not intended to be taken as authoritative or completely well informed, just a first pass

Bug #StatusTitleBrief summarySeverityDifficulty

Data loss/corruption?

Dupe CountDomains touchedFirst ReportedComments[11679](https://www.google.com/url?q=https://github.com/openzfs/zfs/issues/11679&sa=D&source=editors&ust=1716962701643370&usg=AOvVaw044Ra-17I0VznV4qjeC4PY)OpenZFS on Linux null pointer dereferenceThe one I understand best. 

It seems like there's a refcount bug with dnodes when encrypted buffers are around, such that in some unusual case (I suspect it's at least triggerable with some memory fragmentation edge case, based on seeing that happen under heavy mem frag on non-x86 platforms), we wind up with a buffer in a header that thinks it has refcount 0 but it's still referenced somewhere, it gets freed and reallocated, and then eventually either it gets destroyed by one of the references dropping it and then the other tries accessing it and a panic or corruption ensues.5Quite hardSometimes6ARC, send|recv0.8.0-rc[mumble][10523](https://www.google.com/url?q=https://github.com/openzfs/zfs/issues/10523&sa=D&source=editors&ust=1716962701643919&usg=AOvVaw2HOwyi8gnmS1UwWfG025-F)ClosedRaw send on encrypted datasets does not work when copying snapshots backsend -w from src to dst; 

make changes and new snaps on dst;

尝试在源上再增量接收，现在源数据集不可卸载直到回滚3？Yes4send|recv???有人曾试图修补过一次，但发现修补未考虑到从未在稳定版本中发布的加密设置，因此被撤销。[8758](https://www.google.com/url?q=https://github.com/openzfs/zfs/issues/8758&sa=D&source=editors&ust=1716962701644428&usg=AOvVaw2xdAev2akI9Yu5hfygmyT5)Opencannot send raw incremental send after creation of snapshot at remote endsend -w src@snap1 | recv dst;snap dst@newsnap breaks send -w -i src@snap1 src@snap2 until dst@newsnap is destroyed2?No1send|recv0.8.0-rc5[11433](https://www.google.com/url?q=https://github.com/openzfs/zfs/issues/11433&sa=D&source=editors&ust=1716962701644932&usg=AOvVaw1FpHIRCZ_iuJUBtYCzor-A)Openassertion failure:

zpl_get_file_info() 中的坏 sa_magic。看起来是由于用户对象 accounting 中的一些坏元数据导致的，目前为止报告的报告者都在使用加密，不清楚是否相关2????1userobj_accounting2.0.0[8722](https://www.google.com/url?q=https://github.com/openzfs/zfs/issues/8722&sa=D&source=editors&ust=1716962701645445&usg=AOvVaw0xPx-N4Xx1GVDYXpdPmCF_)OpenNULL dereference receiving (non-raw) to encrypted datasetNULL in zio_crypt_copy_dnode_bonus on recv - suspect it's earliest reported instance of #11679dupeSee [#11679](https://www.google.com/url?q=https://github.com/openzfs/zfs/issues/11679&sa=D&source=editors&ust=1716962701645701&usg=AOvVaw0G1mWVCwUyxnS--5FiTpIG)dupe0ARC,send|recv0.8.0-rc4[10275](https://www.google.com/url?q=https://github.com/openzfs/zfs/issues/10275&sa=D&source=editors&ust=1716962701646052&usg=AOvVaw0r2SoibSK3IaD5ns7ZF-i9)OpenCannot zfs recv raw incremental encrypted stream after snapshot on destinationDupe of #8758dupeSee [#8758](https://www.google.com/url?q=https://github.com/openzfs/zfs/issues/8758&sa=D&source=editors&ust=1716962701646298&usg=AOvVaw0PJAml-vTbOn7PIAixrMkY)dupe0send|recv0.8.3[10603](https://www.google.com/url?q=https://github.com/openzfs/zfs/issues/10603&sa=D&source=editors&ust=1716962701646613&usg=AOvVaw3eD83bigNV1fXOkwH-yxW8)Openudev not creating device file for cloned zvolSeems simple.

可能已经通过 #12271 修复？2？No0clones0.8.4[10787](https://www.google.com/url?q=https://github.com/openzfs/zfs/issues/10787&sa=D&source=editors&ust=1716962701647034&usg=AOvVaw27HzoBFZ00W79n21HadkqO)OpenOn zfs recv：

ASSERT at libzfs_sendrecv.c:2798:recv_fix_encryption_hierarchy()看起来是在使用 -w、-h 进行 send/recv 时出现的问题？

可能不再相关了？1Probably lowNo0send|recv (userland)0.8.4[11294](https://www.google.com/url?q=https://github.com/openzfs/zfs/issues/11294&sa=D&source=editors&ust=1716962701647574&usg=AOvVaw1TJiB1AEDyVIY2n_JyWYa1)ClosedHEAD issues mounting encrypted datasets/corrupting metadata因补丁被还原而原因 - 加密数据集与未发行的配置的奇怪组合破坏了错误修复，因此被还原，尚未修复？

难度较大，显然。

不可访问，未丢失0git d1d47691c[11405](https://www.google.com/url?q=https://github.com/openzfs/zfs/issues/11405&sa=D&source=editors&ust=1716962701648091&usg=AOvVaw21GfNV0Pe510DacT6_57Ps)Open发送加密数据集时 PANIC。

看起来是 TrueNAS 特定的补丁引起的问题0???No02.0.0*另一个 TrueNAS SCALE 报告。[11490](https://www.google.com/url?q=https://github.com/openzfs/zfs/issues/11490&sa=D&source=editors&ust=1716962701648587&usg=AOvVaw03lqFxN7deG42kDieGi7AQ)OpenVERIFY(zfs_refcount_count(&dck->dck_holds) == 0) 失败，在 dsl_crypt.c:503:dsl_crypto_key_free() 处 PANIC。也许是在接收过程中错过了某个锁？

I don't know this portion of the code.3????0send|recv2.0.0*It's also on TrueNAS SCALE, may or may not be relevant to upstream?[11688](https://www.google.com/url?q=https://github.com/openzfs/zfs/issues/11688&sa=D&source=editors&ust=1716962701649107&usg=AOvVaw0RktCTcu2IuS8XfkqGrJE7)Openpermanent errors (ereport.fs.zfs.authentication) reported after syncoid snapshot/send workload#11679 with [witticism], I suspect - random corruption in encrypted snapshots sure fits the bill, though I can't be certain until it's fixed and we see if it still happens...dupeSee [#11679](https://www.google.com/url?q=https://github.com/openzfs/zfs/issues/11679&sa=D&source=editors&ust=1716962701649374&usg=AOvVaw1DTn3Y_DOxHmrkg5KIMlDu)dupe0send|recv2.0.3aerusso may have a different opinion than me, but I'd probably suspect many instances of mysterious corruption on-disk when encryption (esp.

encrypted send/recv) 涉及到#11679号问题，除非另有证据，否则[11983](https://www.google.com/url?q=https://github.com/openzfs/zfs/issues/11983&sa=D&source=editors&ust=1716962701649690&usg=AOvVaw3ukUheqU3SpmqQMeaiPzxM)已关闭的输入/输出错误，当发送加密增量数据集回到它的源#10523使用金属漆和肩膀上的芯片dupesee [#10523](https://www.google.com/url?q=https://github.com/openzfs/zfs/issues/10523&sa=D&source=editors&ust=1716962701649926&usg=AOvVaw3uAh9Nc6EbC3RxYTtb78nE)dupe0send|recv2.0.4[12000](https://www.google.com/url?q=https://github.com/openzfs/zfs/issues/12000&sa=D&source=editors&ust=1716962701650208&usg=AOvVaw0_RAzeA_kFuFDqtjLTO12I)OpenRepair encryption hierarchy of 'send -Rw | recv -d' datasets that do not bring their encryption rootSeems like #10523 in a dollar store wig, possibly; 

comes with a reproducer!dupesee [#10523](https://www.google.com/url?q=https://github.com/openzfs/zfs/issues/10523&sa=D&source=editors&ust=1716962701650403&usg=AOvVaw3Y9aV1S9VMTDTJ2QCkupk_)dupe0send|recv0.8.3[12001](https://www.google.com/url?q=https://github.com/openzfs/zfs/issues/12001&sa=D&source=editors&ust=1716962701650701&usg=AOvVaw1-ZJug-CBJB7DWOOnNCIEO)OpenZFS hangs on kernel error: 

VERIFY3(0 == dmu_bonus_hold_by_dnode与＃12732相同，可能只是＃11679dupe参见＃11679（https://www.google.com/url?q=https://github.com/openzfs/zfs/issues/11679&sa=D&source=editors&ust=1716962701650902&usg=AOvVaw0hfWQIMBsPpvLO8_Kb6mLQ）dupe0send|recv2.0.3[12014](https://www.google.com/url?q=https://github.com/openzfs/zfs/issues/12014&sa=D&source=editors&ust=1716962701651175&usg=AOvVaw1IVDyG-MXXcXn_jJin-ZHb)与快照升级后的 OpenZFS 损坏相关#11679，带有错误的假发dupe请参阅＃11679（https://www.google.com/url?q=https://github.com/openzfs/zfs/issues/11679&sa=D&source=editors&ust=1716962701651377&usg=AOvVaw1_WNYkIrzuH4-bd7cf5un-）dupe02.0.3[12123](https://www.google.com/url?q=https://github.com/openzfs/zfs/issues/12123&sa=D&source=editors&ust=1716962701651725&usg=AOvVaw20SYNPOeLJKzesujI2iOo5)在复制加密数据集并在目标数据集上执行密钥继承（更改密钥 -i）之后，下一个增量快照将破坏数据集/卷。顾名思义 - 在原始接收加密数据集的树之后更改子密钥会在某些情况下破坏挂载 - 可能与 10523 等情况相同等＃10523（https://www.google.com/url?q=https://github.com/openzfs/zfs/issues/10523&sa=D&source=editors&ust=1716962701652009&usg=AOvVaw3GugpUWXsUaXU8i1OdSjca）有时0send|recv，密钥管理2.0.4更高评级，因为在这种情况下，您可能明确不再具有旧的快照，而 10523 明确是从活动到冗余副本发送[12270](https://www.google.com/url?q=https://github.com/openzfs/zfs/issues/12270&sa=D&source=editors&ust=1716962701652350&usg=AOvVaw1o0xnrf5YCFzIIBxIODyFg)在 ZFS 接收期间发生 SPL 恐慌与＃12732相同，我认为这两者都只是在不同点处于被编码dupe时失败＃11679请参阅＃11679（https://www.google.com/url?q=https://github.com/openzfs/zfs/issues/11679&sa=D&source=editors&ust=1716962701652590&usg=AOvVaw00_t4-QXd0Qz8c1M66qt80）dupe0ARC，send|recv2.0.4[12418](https://www.google.com/url?q=https://github.com/openzfs/zfs/issues/12418&sa=D&source=editors&ust=1716962701652947&usg=AOvVaw3nd0mKZ0RqfeCJjxuQ2jBV)在加密数据集上默默失败的 ZFS 挂载似乎与＃12614 的概念类似 - 在子级上更改密钥似乎会破坏父级和子级的挂载，尽管至少可以稍后手动触发子级挂载4???有时0密钥管理git 1b50749ce9（2021 年 7 月）不清楚原始包装密钥是否被消除或其他。

假设包装密钥涉及=提示，您可以编写工具绕过检查并重新生成它们，然后重新写入元数据。[12439](https://www.google.com/url?q=https://github.com/openzfs/zfs/issues/12439&sa=D&source=editors&ust=1716962701653452&usg=AOvVaw3lnQzY9ltRCvRmxjpGuuul)在 dsl_crypt.c 的 OpenPANIC：2441：dsl_crypto_populate_key_nvlist()在一些数据集上，为整个文件系统发送 -w，而不仅仅是一个快照，可能会导致崩溃，但并非总是如此？

包含可以复制的示例池。1???No0send|recv2.0.3Rated 1 because it basically only happens if you do something dumb;

如果有真实的使用情况不是用户错误而会出现这种情况，我会将其提升。[12594](https://www.google.com/url?q=https://github.com/openzfs/zfs/issues/12594&sa=D&source=editors&ust=1716962701654010&usg=AOvVaw12EfZ5fV76M39G9HuTt9cO)ClosedSometimes raw send on encrypted datasets does not work when copying snapshots back#10523 with a fake mustache; 

有一个复制脚本！dupesee [#10523](https://www.google.com/url?q=https://github.com/openzfs/zfs/issues/10523&sa=D&source=editors&ust=1716962701654256&usg=AOvVaw1B5YDXAz49L5frbLHbuokJ)dupe0send|recv2.1.1[12614](https://www.google.com/url?q=https://github.com/openzfs/zfs/issues/12614&sa=D&source=editors&ust=1716962701654582&usg=AOvVaw10SfAOQVcRfmDbUKNiA0HA)OpenReplicating encrypted child dataset + change-key + incremental receive overwrites master key of replica, causes permission denied on remountzfs send -Rw src/encrypted@1 | zfs recv dst/encrypted; 

zfs change-key dst/encrypted;zfs send -iw src/encrypted/a@{1,2} | zfs recv dst/encrypted/a; 

会覆盖磁盘上的新包装密钥，并且下次卸载时，如果密钥未加载5???Sometimes0send|recv，密钥管理2.0.5Attila提到他认为在这种情况下你只需要阻塞接收，而不是更好地处理/如果在中间更改了接收包装密钥，则忽略接收的包装密钥。 

对我来说不明确是否有硬性限制不明显的原因。 

对我来说不明确为什么加密的增量需要每次发送，或者为什么接收方不能简单地忽略它或通过实际执行适当的change-key处理它。[12659](https://www.google.com/url?q=https://github.com/openzfs/zfs/issues/12659&sa=D&source=editors&ust=1716962701655286&usg=AOvVaw1sfa4-bFMjzDjiDed8gjzn)OpenKernel panic VERIFY3(sa.sa_magic == SA_MAGIC) faileddupe of 11433dupesee [#11433](https://www.google.com/url?q=https://github.com/openzfs/zfs/issues/11433&sa=D&source=editors&ust=1716962701655552&usg=AOvVaw0wem6xj6y45KDcwW9SwAyN)dupe0

userobj_accounting, send|recv

dupesee #13709013709Open升级后挂载加密文件系统时出现I/O错误哦，这只是再次出现了#11294dupesee #112942send|recv2.1.314083Open无法解密使用v2.1.0创建的数据集在v2.1.6下看起来像是一个macOS特定的错误，只是类似于1370910send|recv2.1.613926Openkernel: PANIC <pool> blkptr at <address> has invalid CHECKSUM 013922Open[sparc64] 加密数据集出现"无法处理内核空指针解引用错误"13699Open每个数据集的大块功能检测在发送原始加密大块数据集时不可靠13634Open无法读取、写入或删除损坏的目录13561Openzpool_import_errata4 PANICs slow sparc64 machine14330Open另一个加密bug："在加密对象集中出现未加密块"看起来你可能会在加密数据集上出现一个embedded_data记录。

??? 没能复现它...

no0send|recv2.1.4易于在代码中删除一些条件后重现，没有找到不涉及此条件的重现方法...14245Open增量接收流导致进程挂起
