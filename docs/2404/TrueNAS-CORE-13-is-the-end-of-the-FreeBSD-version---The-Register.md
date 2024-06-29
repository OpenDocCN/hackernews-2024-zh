<!--yml

分类: 未分类

date: 2024-05-27 13:07:35

-->

# TrueNAS CORE 13 是 FreeBSD 版本的终结 • The Register

> 来源：[https://www.theregister.com/2024/03/18/truenas_abandons_freebsd/](https://www.theregister.com/2024/03/18/truenas_abandons_freebsd/)

来自 BSD 领域的坏消息 – 最古老的 BSD 系统供应商正在改变方向，远离 FreeBSD 转向 Linux。

今年，NAS 供应商 iXsystems 非常忙碌，但除了在在线用户社区发表了一些声明外，他们并未提及大新闻。早在 2022 年，我们报道了基于 FreeBSD 的即插即用 NAS 服务器操作系统 TrueNAS CORE 13 的新版本，同时在那篇文章中提到了它的新产品，基于 Debian 的 TrueNAS SCALE，旨在为 Kubernetes 用户提供存储服务。

现在看来，公司正在押注其未来的基于 Linux 的产品，这意味着 FreeBSD 版本的终结即将到来。

TrueNAS SCALE 显然正在蓬勃发展。Gartner 最近授予 iXsystems [顾客选择奖](https://www.ixsystems.com/blog/truenas-enterprise-customers-choice-for-primary-storage-in-2024/)。由于一些 VMware 用户对 Broadcom 收购的不安，正在评估替代方案，许多 TrueNAS 用户向公司表示他们正在考虑 KVM。

公司同时发布了基于 Debian 的 TrueNAS SCALE 的 [23.10 版本](https://www.truenas.com/docs/scale/23.10/gettingstarted/scalereleasenotes/)。甚至还抽出时间向 FOSS 社区捐赠了其快速去重（Fast DeDup）功能，详情请见 [此链接](https://www.ixsystems.com/blog/ixsystems-and-klara-systems-celebrate-valentines-day-with-a-heartfelt-donation-of-fast-dedupe-to-openzfs-and-truenas/)。

iXsystems 为 TrueNAS SCALE 非常骄傲。但是，除此之外，如果你允许我打个双关语，对于它的 *核心* 产品如何？它之所以称为 TrueNAS CORE 13，是因为它基于 [FreeBSD 13](https://www.freebsd.org/releases/13.0R/announce/) … 但那是在 2021 年 4 月发布的，本月早些时候开发者发布了最新更新版本 [FreeBSD 13.3](https://www.freebsd.org/releases/13.3R/announce/)。TrueNAS CORE 13 的用户，包括笔者在内，仍在等待 13.1 版本的发布，而在公司的 [软件发布页面](https://www.truenas.com/docs/truenasupgrades/)上仍标记为“不稳定构建”，而在 [状态页面](https://www.truenas.com/software-status/) 上也没有任何暗示。

TrueNAS CORE [演变自旧版的FreeNAS](https://www.theregister.com/2016/10/18/truenas_review/)。FreeNAS [首次亮相](https://web.archive.org/web/20080313190212/http://sourceforge.net/potm/potm-2007-01.php) 是在2005年，基于用PHP编写的[m0n0wall防火墙](https://m0n0.ch/wall/index.php)的Web GUI。在iXsystems的领导下，FreeNAS [在版本11中获得全新的用户界面](https://www.theregister.com/2017/06/15/freenas_11/)，至今仍在使用。原始FreeNAS的一个分支，通过更新的FreeBSD组件继续作为[XigmaNAS项目](https://xigmanas.com/xnaswp/)。

PC-BSD的创始人Kris Moore现在仍然在[iXsystems任职](https://www.ixsystems.com/our-team/)，他现在是工程高级副总裁。他还在Reddit上使用[/u/kmoore134](https://www.reddit.com/user/kmoore134/)这个用户名进行活跃。[去年12月](https://www.reddit.com/r/truenas/comments/18hfwcr/comment/kd7l36h/)，他告诉TrueNAS的Subreddit：

这看起来已经有些过时了。[FreeBSD 13.1](https://www.freebsd.org/releases/13.1R/announce/)于2022年5月发布，并在2023年7月达到了[生命周期的终点](https://www.freebsd.org/security/unsupported/)，被[FreeBSD 13.2](https://www.freebsd.org/releases/13.2R/announce/)所取代。

Moore 继续说道：

这听起来不妙。我们询问了公司是否属实，市场副总裁Mario Blandini向*The Reg*确认：

CORE至少会有一个更新：

[13.3版本的愿望清单](https://www.truenas.com/community/threads/truenas-13-3-wishlist.116359/)包含了更多期待的更新细节。

存在升级路径。由于TrueNAS的工作方式，操作系统被保留在自己的专用驱动器上 —— 公司建议使用快速SSD —— 不与网络共享，实际上也无法共享。通过ZFS快照，可以进行[就地迁移](https://www.truenas.com/docs/scale/gettingstarted/migrate/migratingfromcore/)，尽管这是一个单向过程。FreeBSD和Linux版均使用相同的OpenZFS文件系统，因此新操作系统可以从旧系统中获取并迁移设置。但无法迁移的是虚拟机设备。TrueNAS CORE的[插件](https://www.truenas.com/docs/core/coretutorials/jailspluginsvms/plugins/)运行在FreeBSD的Jails中或[bhyve虚拟化器](https://bhyve.org/)下，而您可以在[TrueNAS SCALE](https://www.truenas.com/apps/)中创建的虚拟机则运行在Linux内核的KVM中。这两者不可互换。

这是一则重要且令人悲伤的消息，因为 iXsystems 起源于最初的 BSD 供应商 [Berkeley Software Design, Inc.](https://en.wikipedia.org/wiki/Berkeley_Software_Design)，该公司销售了第一个基于通用硬件的商业 BSD 操作系统 [BSD/386](http://gunkies.org/wiki/BSD/386)。当我们 [审视 FreeBSD 13.1](https://www.theregister.com/2022/05/20/freebsd_131/) 时，我们描述了 BSD 的早期历史。该公司有着 [复杂的历史](https://www.ixsystems.com/history/)，涉及多次并购、出售和重新收购，包括 Walnut Creek CD-ROM。

Unix 市场几十年来一直动荡不安，而这家公司在不同名称下有着长时间的出售、收购和产品停产历史。与 Walnut Creek 合并后，它将其操作系统部门 [出售给了](https://www.eetimes.com/wind-river-buys-into-open-source-but-avoids-linux/) Wind River，后者随后将其 FreeBSD 业务 [出售给](https://www.macworld.com/article/152246/freebsd.html) FreeBSD Mall，并在 2007 年 [被 iXsystems 收购](https://www.ixsystems.com/blog/ixsystems-announces-acquisition-of-freebsd-mall/)，这发生在该公司 [2006 年收购](https://web.archive.org/web/20170701063157/http://www.onlamp.com/pub/a/bsd/2006/10/23/ixsystems-pc-bsd.html) 桌面版 FreeBSD 发行版 PC-BSD 之后。

2009 年，FreeNAS 的开发者转向了基于 Linux 的系统，这发展成了 [Open Media Vault](https://www.openmediavault.org/)。iXsystems 加大了力度，并 [接管了 FreeNAS 的开发](https://lwn.net/Articles/366886/)。它还继续在其桌面版上工作，2016 年将其改名为 TrueOS。2020 年，它 [停止了 TrueOS](https://www.truenas.com/trueos-discontinuation/)，并 [将 FreeNAS 与其 TrueNAS 系列合并](https://www.truenas.com/blog/freenas-truenas-unification/)。

对于公司而言，这一举措无疑是商业上明智的。虽然 *The Reg* FOSS 专栏仍对大多数用户是否真的 *需要* Kubernetes 持怀疑态度，但它仍然很受欢迎且继续增长。许多公司已将未来寄托于这一趋势，并因此获利。对于 iXsystems，对我们来说，这就像一条小鱼选择跳进大池塘，我们担心如果微服务狂热的浪潮到来并消退时会发生什么。

FreeNAS 的一个优点是它简单而节约资源。像 XigmaNAS 一样，它可以安装在 USB 键盘上，这意味着升级只需写入新的镜像到新的键盘。在与公司的简报会上，我们反复提醒 iXsystems，并要求恢复该功能，但该公司的代表似乎不理解为什么这是可取的。

十一年前，我们曾经写过关于如何以企业客户为目标，导致 [Novell 丢失了其小企业市场的核心](https://www.theregister.com/Print/2013/07/16/netware_4_anniversary/)。我们担心 iXsystems 也面临同样的命运。

ZFS 在 Linux 上的一个问题是，由于它不是内核的一部分，所以它的缓存必须与 Linux 内核自己的缓存保持分离，这对于[bcachefs](https://www.theregister.com/2024/01/10/linux_kernel_67/)来说是一个关键优势。我们至少有三台旧的 HP 微型服务器，它们总共拥有 22 GB 的内存。这意味着 TrueNAS SCALE 对我们几乎没用 —— 我们怀疑，对许多其他小型部署也是如此。

已经有[关于分叉的传言](https://old.reddit.com/r/truenas/comments/1baju4f/any_updates_on_core_131/)，我们将密切关注这一领域。®
