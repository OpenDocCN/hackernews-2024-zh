<!--yml

类别：未分类

日期：2024-05-27 14:48:24

-->

# OpenZFS开放加密错误（公共RO）- Google Drive

> 来源：[https://docs.google.com/spreadsheets/d/1OfRSXibZ2nIE9DGK6swwBZXgXwdCPKgp4SbPZwTexCg/htmlview](https://docs.google.com/spreadsheets/d/1OfRSXibZ2nIE9DGK6swwBZXgXwdCPKgp4SbPZwTexCg/htmlview)

快速备注：这只是对每一个看起来涉及加密的未解决问题进行的快速检查。并不意味着其具有权威性或者完全了解情况，只是第一次检查。

Bug #状态标题简短总结严重性难度

数据丢失/损坏？

重复计数涉及的域首次报告的注释[11679](https://www.google.com/url?q=https://github.com/openzfs/zfs/issues/11679&sa=D&source=editors&ust=1716796099392925&usg=AOvVaw32M97dEhKOZGWQU1BM3IRF)OpenZFS on Linux 空指针解引用我最理解的一个。 

看起来当加密缓冲区存在时，dnodes存在引用计数错误，因此在某些不寻常的情况下（我怀疑至少在某些内存碎片化边缘情况下会触发，基于在非x86平台上发生内存碎片化情况），我们最终会遇到一个头部中有一个认为其引用计数为0但实际上仍在某处被引用的缓冲区，它被释放并重新分配，然后最终要么它被一个引用丢弃并且另一个尝试访问它并导致崩溃或损坏。5相当困难有时候6ARC，发送|接收0.8.0-rc[mumble][10523](https://www.google.com/url?q=https://github.com/openzfs/zfs/issues/10523&sa=D&source=editors&ust=1716796099393678&usg=AOvVaw3eb7TbBOntoWmlmiLyJSQd)关闭在加密数据集上进行原始发送时，将快照复制回去时不起作用发送-w从src到dst； 

对dst进行更改和新的快照； 

尝试再次增量接收src，现在src数据集无法卸载，直到回滚为止3？是的4发送|接收???有人曾试图修补过一次，但发现修复未考虑到从未稳定发布的加密设置，因此被撤销了。[8758](https://www.google.com/url?q=https://github.com/openzfs/zfs/issues/8758&sa=D&source=editors&ust=1716796099394332&usg=AOvVaw30H6PYVlKAi8og2jofoPNw)Open不能在创建远程端快照后发送原始增量发送-send -w src@snap1 | recv dst；snap dst@newsnap在销毁dst@newsnap之前会破坏send -w -i src@snap1 src@snap2?否1发送|接收0.8.0-rc5[11433](https://www.google.com/url?q=https://github.com/openzfs/zfs/issues/11433&sa=D&source=editors&ust=1716796099394829&usg=AOvVaw0cI55MFGeaJNcjAEox4Bj7)Open断言失败：

zpl_get_file_info()中的坏sa_magic看起来像是用户对象计算中不知怎么搞的一些坏的元数据最终出现在内存或磁盘上 - 到目前为止，所有报告者都在使用加密，不清楚是否有关2????1userobj_accounting2.0.0[8722](https://www.google.com/url?q=https://github.com/openzfs/zfs/issues/8722&sa=D&source=editors&ust=1716796099395310&usg=AOvVaw0ExVGiTLFkI3kU9GycV5dy)OpenNULL dereference receiving (非原始) 到 加密的数据集NULL in zio_crypt_copy_dnode_bonus on recv - 这个怀疑是#11679的最早报告实例导致dupe查看[#11679](https://www.google.com/url?q=https://github.com/openzfs/zfs/issues/11679&sa=D&source=editors&ust=1716796099395535&usg=AOvVaw0iVAPbYlggsj1gjkI-unzf)dupe0ARC,send|recv0.8.0-rc4[10275](https://www.google.com/url?q=https://github.com/openzfs/zfs/issues/10275&sa=D&source=editors&ust=1716796099395854&usg=AOvVaw0zC5iys8TMekx_C0DEU3AY)OpenCannot zfs recv raw incremental encrypted stream after snapshot on destinationDupe of #8758dupeSee [#8758](https://www.google.com/url?q=https://github.com/openzfs/zfs/issues/8758&sa=D&source=editors&ust=1716796099396054&usg=AOvVaw1IIEqyiIy4rIdabUXrDSwa)dupe0send|recv0.8.3[10603](https://www.google.com/url?q=https://github.com/openzfs/zfs/issues/10603&sa=D&source=editors&ust=1716796099396327&usg=AOvVaw3gjfwqIPq9NHqhGxgmbUZy)Openudev not creating device file for cloned zvol看起来很简单。

可能已经通过#12271修复了？2？No0clones0.8.4[10787](https://www.google.com/url?q=https://github.com/openzfs/zfs/issues/10787&sa=D&source=editors&ust=1716796099396753&usg=AOvVaw1lD9Pmr62NDRjNJz0TWv0c)OpenOn zfs recv: 

assertlibzfs_sendrecv.c:2798：recv_fix_encryption_hierarchy()处出现故障发送/接收之间的一个缺陷（未加密的数据集，使用-w，-h）？

可能已经不再相关？1Probably lowNo0send|recv (userland)0.8.4[11294](https://www.google.com/url?q=https://github.com/openzfs/zfs/issues/11294&sa=D&source=editors&ust=1716796099397194&usg=AOvVaw0OThuPUkUwHRqRpBBM3_uL)ClosedHEAD issues mounting encrypted datasets/corrupting metadata由于加密数据集与从未在发行版中出现的奇特配置结合在一起，导致补丁被还原 - 修复错误导致补丁被损坏，目前尚未修复。？

困难，显然是这样。

不可取，未丢失0git d1d47691c[11405](https://www.google.com/url?q=https://github.com/openzfs/zfs/issues/11405&sa=D&source=editors&ust=1716796099397658&usg=AOvVaw2yfwYGaGz236EVmSfj5LOx)在使用原始标志发送加密数据集时会触发OpenPANIC

看起来是TrueNAS特定补丁引起的错误0???No02.0.0*Another TrueNAS SCALE report.[11490](https://www.google.com/url?q=https://github.com/openzfs/zfs/issues/11490&sa=D&source=editors&ust=1716796099398156&usg=AOvVaw26D8BeRN5nj_Vz75s6IZyx)OpenVERIFY(zfs_refcount_count(&dck->dck_holds) == 0)失败了在dsl_crypt.c:503:dsl_crypto_key_free()出现PANIC也许在接收过程中丢失了某个被占用的锁？

I don't know this portion of the code.3????0send|recv2.0.0*It's also on TrueNAS SCALE, may or may not be relevant to upstream?[11688](https://www.google.com/url?q=https://github.com/openzfs/zfs/issues/11688&sa=D&source=editors&ust=1716796099398791&usg=AOvVaw0CEJv-RQgCcMm6c8s-W4XU)Openpermanent errors (ereport.fs.zfs.authentication) reported after syncoid snapshot/send workload#11679 with [witticism], I suspect - random corruption in encrypted snapshots sure fits the bill, though I can't be certain until it's fixed and we see if it still happens...dupeSee [#11679](https://www.google.com/url?q=https://github.com/openzfs/zfs/issues/11679&sa=D&source=editors&ust=1716796099399028&usg=AOvVaw2Sj33ALW6hJA8n4KP5BDUS)dupe0send|recv2.0.3aerusso may have a different opinion than me, but I'd probably suspect many instances of mysterious corruption on-disk when encryption (esp.

encrypted send/recv) is involved of being #11679 until proven otherwise[11983](https://www.google.com/url?q=https://github.com/openzfs/zfs/issues/11983&sa=D&source=editors&ust=1716796099399316&usg=AOvVaw0e75Xqe4KqOCjy0Wyb5GIb)ClosedInput/Output Error when sending an encrypted incremental dataset back to it's source#10523 with metal paint and a chip on its shoulderdupesee [#10523](https://www.google.com/url?q=https://github.com/openzfs/zfs/issues/10523&sa=D&source=editors&ust=1716796099399548&usg=AOvVaw0BbweRsyRwrRZSWEIcbZu2)dupe0send|recv2.0.4[12000](https://www.google.com/url?q=https://github.com/openzfs/zfs/issues/12000&sa=D&source=editors&ust=1716796099399944&usg=AOvVaw00O5rfcAAFOtmbKqPrEfpV)OpenRepair encryption hierarchy of 'send -Rw | recv -d' datasets that do not bring their encryption rootSeems like #10523 in a dollar store wig, possibly; 

comes with a reproducer!dupesee [#10523](https://www.google.com/url?q=https://github.com/openzfs/zfs/issues/10523&sa=D&source=editors&ust=1716796099400232&usg=AOvVaw0UZKu3Nf_alw8AR__N4iv3)dupe0send|recv0.8.3[12001](https://www.google.com/url?q=https://github.com/openzfs/zfs/issues/12001&sa=D&source=editors&ust=1716796099400531&usg=AOvVaw1LEQFZMpwhlMdqenTvR1Tv)OpenZFS hangs on kernel error: 

VERIFY3(0 == dmu_bonus_hold_by_dnode同12732号，可能只是11679号的复制参见[#11679](https://www.google.com/url?q=https://github.com/openzfs/zfs/issues/11679&sa=D&source=editors&ust=1716796099400735&usg=AOvVaw1ZMxdJM5IxIkhsRdDJXJy5)dupe0send|recv2.0.3[12014](https://www.google.com/url?q=https://github.com/openzfs/zfs/issues/12014&sa=D&source=editors&ust=1716796099400989&usg=AOvVaw0idjTb3voSZmL2iuYpyrRN)OpenZFS与2.0.x升级后的快照相关的损坏#11679有一个坏的假发参见[#11679](https://www.google.com/url?q=https://github.com/openzfs/zfs/issues/11679&sa=D&source=editors&ust=1716796099401182&usg=AOvVaw1RXuq3mOsDpSqDOrFz43gD)dupe02.0.3[12123](https://www.google.com/url?q=https://github.com/openzfs/zfs/issues/12123&sa=D&source=editors&ust=1716796099401484&usg=AOvVaw0vEZG_BIxMAOdXGO5ZoK5L)Open在复制加密数据集并在目标数据集上执行密钥继承（change-key -i）后，下一个增量快照将破坏数据集/卷。正如名称所示 - 在原始接收加密数据集的树后更改子密钥会在某些情况下中断挂载 - 可能与10523等同5见[#10523](https://www.google.com/url?q=https://github.com/openzfs/zfs/issues/10523&sa=D&source=editors&ust=1716796099401778&usg=AOvVaw2sXLY1FRX1Y0A6nsg81B8Y)Sometimes0send|recv，密钥管理2.0.4由于在这种情况下，您可能明确地不再拥有旧的快照，而10523则明确地从活动副本发送到冗余副本[12270](https://www.google.com/url?q=https://github.com/openzfs/zfs/issues/12270&sa=D&source=editors&ust=1716796099402161&usg=AOvVaw3muR-_ci3ioATJb6wkOJiN)Open在ZFS接收期间发生SPL Panic与我认为都是只是#11679，当你在代码的不同点上输掉了比赛时，它们都是同一个11679号的长衣。参见[#11679](https://www.google.com/url?q=https://github.com/openzfs/zfs/issues/11679&sa=D&source=editors&ust=1716796099402427&usg=AOvVaw3kmOw63hQHClyv3YSd8iOo)dupe0ARC，send|recv2.0.4[12418](https://www.google.com/url?q=https://github.com/openzfs/zfs/issues/12418&sa=D&source=editors&ust=1716796099402766&usg=AOvVaw1iULKyCoz_PLfnbz7eeeA7)Openzfs挂载加密数据集时默默失败似乎与#12614类似 - 在子项上更改密钥似乎会破坏父项和子项的挂载，尽管至少可以手动触发子项稍后挂载4???Sometimes0密钥管理git 1b50749ce9（2021年7月）不清楚原始包装密钥是否被抹除或什么。

如果所涉及的包装密钥是=prompt，您可以在编写工具绕过检查并重新编写元数据之后简单地重新生成它们。[12439](https://www.google.com/url?q=https://github.com/openzfs/zfs/issues/12439&sa=D&source=editors&ust=1716796099403380&usg=AOvVaw0UFdfBl5e0Q69tnP43eq3Z)OpenPANIC at dsl_crypt.c:2441:dsl_crypto_populate_key_nvlist()给予send -w一个完整的文件系统而不仅仅是一个快照对于某些数据集可能会导致崩溃，但并非总是？

用于复制的样品池已被封闭。1???No0send|recv2.0.3之所以评分为1分，是因为这基本上只会发生在你做了一些愚蠢的事情时； 

如果有人有一个不是用户错误的真实用例，它会完成这个，我会提高它。[12594](https://www.google.com/url?q=https://github.com/openzfs/zfs/issues/12594&sa=D&source=editors&ust=1716796099404002&usg=AOvVaw2LaHYRPDTf51Jbu0E1TpXF)已关闭有时在加密数据集上进行原始发送时，当将快照复制回去时，此操作不起作用#10523 with a fake mustache； 

有一个复制脚本！dupesee [#10523](https://www.google.com/url?q=https://github.com/openzfs/zfs/issues/10523&sa=D&source=editors&ust=1716796099404298&usg=AOvVaw0oCWtuPrO5XV4k98kH22w-)dupe0send|recv2.1.1[12614](https://www.google.com/url?q=https://github.com/openzfs/zfs/issues/12614&sa=D&source=editors&ust=1716796099404807&usg=AOvVaw3TEGjxs_0MNK66xA-jWTVm)已打开复制加密的子数据集 + 更改密钥 + 增量接收会覆盖副本的主密钥，导致重新挂载时权限被拒绝zfs send -Rw src/encrypted@1 | zfs recv dst/encrypted； 

zfs change-key dst/encrypted;zfs send -iw src/encrypted/a@{1,2} | zfs recv dst/encrypted/a； 

这将使用旧的包装密钥覆盖磁盘上的新密钥，下次卸载时，数据就会丢失5???Sometimes0send|recv，密钥管理2.0.5Attila提到他认为你只需要在那种情况下阻止接收，而不是更好地/只是在期间更好地处理它/忽略接收的包装密钥。 

如果这是因为我不明显的原因而不能解决的一个严格的限制，那对我来说不清楚。

对我来说不清楚加密的增量为什么每时每刻都要发送它...或者为什么接收方不能只是忽略它或通过实际进行适当的更改密钥来处理它。[12659](https://www.google.com/url?q=https://github.com/openzfs/zfs/issues/12659&sa=D&source=editors&ust=1716796099405482&usg=AOvVaw0xerxw49Tmq_u22LCxqXiS)已打开内核恐慌VERIFY3（sa.sa_magic == SA_MAGIC）失败的dupe 11433dupesee [#11433](https://www.google.com/url?q=https://github.com/openzfs/zfs/issues/11433&sa=D&source=editors&ust=1716796099405805&usg=AOvVaw0tAjUMloJfH3GlyN89aoHb)dupe0 

userobj_accounting, send|recv 

2.0.6[12659](https://www.google.com/url?q=https://github.com/openzfs/zfs/issues/12659&sa=D&source=editors&ust=1716796099406383&usg=AOvVaw0_ixZVpHPvHGY29Ewp-yHa)Open无法接收加密原始发送，接收端出现“必须升级内核模块”的错误，但双方均为2.0.6，所以？？？3？？？不0发送|接收2.0.6[12732](https://www.google.com/url?q=https://github.com/openzfs/zfs/issues/12732&sa=D&source=editors&ust=1716796099407021&usg=AOvVaw1ydTr-j4WMLNBKSdQ1p2Ce)OpenPANIC在dmu_recv.c上，接收快照到加密文件系统#11679，我认为又是一个dupe看[#11679](https://www.google.com/url?q=https://github.com/openzfs/zfs/issues/11679&sa=D&source=editors&ust=1716796099407338&usg=AOvVaw164BBzgZezFaFERuU8KUpi)dupe0发送|接收2.0.6[12785](https://www.google.com/url?q=https://github.com/openzfs/zfs/issues/12785&sa=D&source=editors&ust=1716796099407762&usg=AOvVaw313McGVccA28Qa2PqVk9Jm)Open在同一池中两个加密数据集之间的增量发送/接收时发生内核恐慌，目标正在使用zstd-19#12001，我怀疑可能仅仅是#11679dupe看[#11679](https://www.google.com/url?q=https://github.com/openzfs/zfs/issues/11679&sa=D&source=editors&ust=1716796099408078&usg=AOvVaw3hiELettXDMxKNVr1pgOXO)dupe0发送|接收2.0.313067Closed使用较大的ashift向池进行原始发送导致无法挂载的文件系统12762的家族？PR open我认为是这样的0发送|接收，ashift2.1.2[13038](https://www.google.com/url?q=https://github.com/openzfs/zfs/issues/13038&sa=D&source=editors&ust=1716796099408984&usg=AOvVaw1Vt2TFP0N1QSM_VnlQzmz6)ClosedFreeBSD：zfskeys_enable：未为自动启动导入的池中的文件系统加载加密密钥1？？？no0git ???13033Closed无法将未加密数据集接收为加密数据集的子级（“无法接收新文件系统流：必须加载继承的密钥”）1？？？no012869Openztest在l2arc_apply_transforms中失败，比较计算的MAC1？？？？？？0ztestgit ???9046Closed在运行zpool_create_crypt_combos测试时在abd_verify中出现panic再次，仅仅是11679，可怜而未修复dupe看#11679？？？013445OpenZFS接收加密增量数据流导致PANIC相对于13067的问题不清楚是否在2.1.4+1？？？？？？0发送|接收2.1.213477Open在zfs_znode.c zfs_znode_sa_init()中的PANIC - 10971关闭问题中的回归？11679的亲属，或另一个SA相关的恐慌2.1.413491Open在接收和加密数据集时发生Panic是的，那个堆栈跟踪完全是11679dupe看#116790发送|接收2.1.213521Open无法挂载文件系统：输入/输出错误也许只是更改密钥的错误，也许是#13709，谁知道30发送|接收0.8.3-ubuntumumble13859Open检测到以下文件中的永久错误，经过干净的擦洗和没有其他错误

dupesee #13709013709Open升级后挂载加密文件系统时出现I/O错误哦，这只是 #11294 再次出现dupesee #112942send|recv2.1.314083Open无法解密使用 v2.1.0 创建的 v2.1.6 下的数据集看起来是 macOS 特定的 bug，只是类似于 1370910send|recv2.1.613926Open内核：PANIC <pool> blkptr at <address> has invalid CHECKSUM 013922Open[sparc64] 加密数据集出现 "无法处理内核空指针解引用错误"13699Open每个数据集的大块特性检测在发送原始加密大块数据集时不可靠13634Open无法读取、写入或删除损坏的目录13561Openzpool_import_errata4 PANICs slow sparc64 machine14330Open另一个加密 bug："未加密块在加密对象集中"看起来你可能会在加密数据集上出现嵌入式数据记录。

??? 无法重现它...

no0send|recv2.1.4如果将代码中的某些条件语句删除，则易于重现，尚未找到不涉及此条件的重现方式...14245Open增量流接收导致进程挂起
