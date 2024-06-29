<!--yml

category: 未分类

date: 2024-05-27 14:49:47

-->

# Linux 6.8 发布，这些是新特性 - OMG! Ubuntu

> 来源：[https://www.omgubuntu.co.uk/2024/03/linux-kernel-6-8-new-features](https://www.omgubuntu.co.uk/2024/03/linux-kernel-6-8-new-features)

**经过几个月的稳定开发，Linux 6.8 内核已正式发布。**

对 Ubuntu 用户来说，这个版本尤为重要，因为它是选择用于 Ubuntu 24.04 LTS 的版本 - 即 GA 内核，因此在发布期间得到支持。

宣布 Linux 内核 6.8 版本在官方 Linux 内核邮件列表（LKML）上发布，Linux 创始人 [Linus Torvalds 表示](https://lkml.org/lkml/2024/3/10/243)：

*“这不是历史上像 6.7 那样的大版本发布 - 在过去几年里，我们似乎回到了一个相当平均的发布大小。”*

补充：*“你可以在整体的差异统计中看到它 - 从各方面来看，这看起来像是一个平均发布，我们没有（例如）任何明显的新的大型文件系统或架构。我认为 6.8 中最大的单一新功能可能是新的 Xe drm 驱动程序，但老实说，大多数的变化只是各种随机的更新和修复。”*

那么究竟有什么新的呢？

## Linux 6.8：新特性

正如你所预期的那样，Linux 内核 6.8 包括了大量的硬件准备、启用和早期支持，大多数我们目前都没有使用的硬件和硬件功能。

这包括了 Linus 在发布公告中提到的实验性 Intel Xe DRM 驱动程序，以及对 AMD Zen 5 和其他即将推出的 AMD 硬件的进一步支持，还有 Qualcomm Snapdragon 8 Gen 3（以及相关 SoCs）的初始代码，等等。

> 别忘了为即将推出的硬件进行启用 - 对我来说，令人兴奋的内核变化是我今天能够受益的那些！

但对我来说，真正令人兴奋的 Linux 内核变化是我现在可以感受到、受益于或者自己能够利用的那些变化。

幸运的是，Linux 6.8 带来了一大堆这些变化！

**Linux 6.8 将树莓派 5 的支持添加到 V3D DRM 驱动程序中**，并引入了 GPUTop 和 FDINFO。任何将 Mesa 23.3 与 Linux 6.8 配对的发行版现在在树莓派 5 上无需额外的内核补丁即可提供稳定的图形体验。

~~此更改应确保 Ubuntu 24.04 LTS 在 [树莓派 5](https://www.omgubuntu.co.uk/2023/09/raspberry-pi-5-officially-announced) 上运行*如丝般顺滑*。~~ 结果证明 Ubuntu 已经因为基于官方的 [树莓派内核](https://github.com/raspberrypi/linux) 补丁集而运行如丝般顺滑）。但这个主线添加是其他 Linux 发行版将从中受益的地方。

至于这个内核版本，[zswap](https://docs.kernel.org/admin-guide/mm/zswap.html) 子系统现在能够在内存压力过大时将冷页强制移到真正的交换空间（对于不希望使用此功能的人可以选择退出）。还有一个**新的 zswap 模式可以完全禁用写回到交换空间**。

Linux 内核 6.8 可以防止对已安装文件系统的块设备进行直接写入（目前 Btrfs 除外）。开发人员表示，对已挂载设备的写入可能导致文件系统损坏和崩溃。此功能默认情况下处于禁用状态，但可以理解大多数 Linux 发行版将选择启用它。

根据调整英特尔 P-State CPU 频率缩放驱动程序的调整，使用英特尔“Meteor Lake”CPU（去年年底发布）的设备在 Linux 下将会达到宣传的“提升”速度。在之前的内核版本中，[发现](https://www.spinics.net/lists/kernel/msg5059502.html)它们运行速度约低于宣传的100MHz。

> 英特尔 Core Ultra Mobile 处理器终于在 Linux 内核 6.8 中达到宣传的提升速度

如果你在 *联想 ThinkPad X1 Carbon（第12代）*、*宏碁 Swift Go 14*、*华硕 Expertbook B5* 或者其他任何使用英特尔 Core Ultra Mobile 处理器的笔记本上使用 Linux，预期在高峰负载期间会有**更多的性能提升**。

关于便携设备，AMD Ryzen 7000（以及即将推出的 Ryzen 8000）笔记本电脑在无线网络和 GPU 内存时钟方面遭遇到射频干扰（RFI）。Linux 6.8 包括了 **AMD RFI 缓解**（WBRF）来解决这个问题。

与网络相关：Linux 6.8 包含提升网络缓存效率的网络增强功能。这被认为[可以提高](https://git.kernel.org/pub/scm/linux/kernel/git/torvalds/linux.git/commit/?id=3e7aeb78ab01)*<q>与多个并发连接的 TCP 性能达到 40% 的提升</q>* — 虽然大多数用户将从中受益的程度尚不清楚。

如果你是 Linux 游戏玩家？如果是的话，请注意 Linux 6.8 增加了对以下内容的支持：

+   **任天堂 Switch 在线控制器**

+   **Powkiddy X5 和 RK2023 手持游戏机**

+   **Adafruit Mini I2C 游戏手柄**

+   **联想 Legion Go 手柄**

+   **Steam Deck 上的颜色管理**

还包括了官方 Steam 控制器的驱动程序修复。

除了上述内容，Linux 6.8 还有一些其他的亮点选择：

+   **新的** `statmount()` **和** `listmount()` **系统调用**

+   **新的[截止服务器](https://lwn.net/Articles/934415/)机制**

+   **Rust 内核支持 LoongArch CPU**

+   **可以更改跟踪子缓冲区的大小**

+   **KVM 的 Guest-first 内存特性**

+   **用于自动调优内核同页合并子系统的 KSM 顾问**

+   **IBM Z 上 sys 调用入口性能提高了约 11%**

+   **用 Rust 编写的新 PHY 网络驱动程序**

+   **英特尔 Trust Domain Extensions（TDX）主机端支持**

+   **英特尔 IAA 压缩加速器**

+   **dmesg 信息显示引导时是否禁用了32位支持**

+   `perf` 工具现在支持数据类型的性能分析

+   **支持 Apple M1 Thunderbolt DART**

+   **Bcachefs 增加了初步的在线文件系统检查和修复功能**

+   **AppArmor 切换到 SHA-256 用于策略哈希验证**

最后，虽然我们很少（如果有的话）在 RISC-V 板上运行 Linux，但无可否认这种开源处理器架构有着非常光明的未来。

更亮眼的是，Linux 6.8 还支持 AMD 的 MicroBlaze V 软核 RISC-V CPU；增加了 XIP 内核特性和 `riscv_hwprobe()` 系统调用；在 RISC-V 上支持 **挂起到 RAM**，当存在 SUSP SBI 扩展时；为 StarFive SoC 提供了新的摄像头子系统驱动程序。

对于以上所有细节，我（一如既往地）建议阅读两部分的 LWN 合并摘要（[第一部分](https://lwn.net/Articles/957188/) & [第二部分](https://lwn.net/Articles/958178/)），这些摘要提供了简明、无废话的概述，以及相关的深入和更多信息的链接。

## 获取 Linux 内核 6.8

至于获取 Linux 6.8，你可以立即下载源代码手动编译内核。有点繁琐；你最好等待你的 Linux 发行版正式打包 Linux 内核 6.8，并将其作为软件更新推送给你。

下个月你可以安装或升级到 Ubuntu 24.04 LTS，它默认包含 Linux 6.8（这将被回溯到 Ubuntu 22.04 LTS 在下一个 HWE/ Ubuntu 22.04.5 LTS 中）。

尽管其他 Linux 博客建议如此，**不建议使用 Canonical 的主线内核构建**（至少因为它们未签名，可能在某些情况下无法启动，不会获得安全更新，不包含某些补丁等）。

话虽如此，一些人确实在 Ubuntu 中安装了 Canonical 的主线内核构建（尽管有些人仅用于临时测试）。如果你迫不及待地想要 Linux 内核 6.8，那么这些预打包的 DEB 包是*一种*选择，但不建议使用——自行承担风险！
