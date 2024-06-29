<!--yml

category: 未分类

date: 2024-05-27 14:49:30

-->

# Linux 6.9 将是首个超过一千万 Git 对象的版本 • The Register

> 来源：[https://www.theregister.com/2024/03/11/linux_6_8_arrives/](https://www.theregister.com/2024/03/11/linux_6_8_arrives/)

Linus Torvalds 发布了 Linux 内核的 6.8 版本。

“所以这次提交计数的下降时间比我倾向于的要长一些，” Torvalds 在 Linx 内核邮件列表上写道，链接在这里：[https://lkml.iu.edu/hypermail/linux/kernel/2403.1/01820.html](https://lkml.iu.edu/hypermail/linux/kernel/2403.1/01820.html)，这是在周日，“但其中很多似乎是各种自测更新（特别是网络）而不是任何实际问题的真正迹象。”

“而且过去的两周相当安静，所以我觉得没有真正的理由延迟 6.8 版本。”

因此，他交付了它，结束了他自己对这个内核版本可能需要第八个发布候选版本的[推测](https://www.theregister.com/2024/03/04/linux_6_8_rc_7/)。

Torvalds 发现时间来记录他所描述的“一些随机的 git 数字学”，当这个版本的内核工作结束时，用于跟踪它的 git 仓库包含了 9.996 百万个对象。

“这是最后一个少于一千万 git 对象的主线内核”，Torvalds 写道。“当然，除了一个漂亮的圆数之外，它没有任何特别之处。Git 不关心，”他补充道。

说得对 — 尤其是指出其他树（如 linux-next）已经真正超过了一千万个对象。

Torvalds 将新的 Xe drm 驱动程序的添加视为这个内核版本最重要的新增功能。该驱动程序支持英特尔 GPU，无论是集成在 CPU 中还是独立设备，但被视为实验性。

内核的另一个新增功能是支持亚马逊 Web 服务的“Nitro”，这家云巨头的隔离技术将安全和网络任务转移到智能网卡上。一个新的驱动程序将 Nitro 硬件暴露给内核，允许 Linux 客户端访问其提供的服务。

bcachefs 文件系统，是 Linux 6.7 的重要新增之一，经历了一些修复以改善性能。另一个增强功能将支持 Rust 的内核引入到使用中国龙芯架构的 CPU 中。

树莓派 5 的图形硬件得到了更多支持，任天堂的 Switch Online 控制器现在也受到 Linux 的支持。

Torvalds 的帖子以对内核开发者测试 6.8 版内核的呼吁结束，感谢那些在其合并窗口开启之前已经提交了 6.9 版本的拉取请求。®
