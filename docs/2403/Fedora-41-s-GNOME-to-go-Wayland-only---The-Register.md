<!--yml

category: 未分类

date: 2024-05-27 14:55:01

-->

# Fedora 41的GNOME版将完全转向Wayland • The Register

> 来源：[https://www.theregister.com/2024/03/13/fedora_41_drops_x_gnome/](https://www.theregister.com/2024/03/13/fedora_41_drops_x_gnome/)

Fedora开发团队正在讨论在Fedora 41中取消GNOME的X11会话，意味着旗舰版本将仅支持Wayland。

这一变化源自[正在进行的在线讨论](https://pagure.io/fedora-workstation/issue/414#comment-899128)，计划在Fedora 41中实施，即在接下来的版本中，预计约七个月后发布。目前仍在讨论中，尚未定案，但看起来可能性很大。

这项改变已经酝酿了一段时间 —— 我们在去年十月就[报导了这一提议](https://www.theregister.com/2023/10/13/gnome_proposes_dropping_x11/)。这也与Fedora的母公司及主要赞助商Red Hat的宣布保持一致，他们去年宣布[RHEL 10将只支持Wayland](https://www.theregister.com/2023/11/29/rhel_10_dropping_x11/)。

请注意，这不适用于即将发布的Fedora 40，预计下个月发布。但在Fedora 40中可能会被视为此路径的一步 *将* 发生。Fedora 40的KDE版本将包含[新发布的KDE Plasma 6](https://www.theregister.com/2024/02/29/kde_plasma_60_released/)，并且团队已经[声明](https://fedoraproject.org/wiki/Changes/KDE_Plasma_6)，Fedora的KDE版本将默认为Wayland-only：

这项提议的改变意味着GNOME版本的默认设置将与KDE版本保持一致。

自2016年第25版以来，Fedora的GNOME版本已默认使用Wayland，而自2021年第34版以来，KDE版本也是如此 —— 但是X11服务器仍安装在系统中，用户可以在登录界面上选择“GNOME on X.org”或“KDE Plasma on X.org”使用它。

这一改变并不意味着不再支持使用X.org。它只意味着从Fedora 40开始，希望使用X.org的KDE用户将需要手动安装它，而从Fedora 41开始，GNOME用户也是如此。

即使现在还有一些理由希望继续使用X11：例如，如果您使用某些辅助工具或者需要某些只有X.org支持的图形驱动功能。一个非常常见的例子就是，如果您在VirtualBox下运行Fedora，并且希望利用Guest Additions提供的图形驱动程序的额外功能，比如动态调整客户机显示大小。

对于那些拥有最新Nvidia显卡的用户来说，这不应该是个问题。最新版本的Nvidia专有驱动现在已经完全兼容Wayland。Fedora贡献者Neal Gompa告诉*Reg*：

Fedora 40 预计会采用 [新发布的 6.8 内核](https://www.theregister.com/2024/03/11/linux_6_8_arrives/)。这意味着使用较老的需要 [“遗留” 驱动程序之一](https://www.nvidia.com/en-us/drivers/unix/legacy-gpu/)（即 470 版本或更旧）的旧 Nvidia 卡的用户可能注定运气不佳。在我们的测试中，我们发现 390 版本的遗留驱动程序在 6.5 或更新版本的内核上拒绝编译和安装。

尽管如此，这算是一个分水岭时刻。Wayland 多年来一直在发展，但 2024 年可能成为它主导的一年。可以说，GNOME 和 KDE 是自由开源软件世界中使用最广泛的两个主流桌面环境，而 Fedora 既是最高知名度的发行版之一，也是最前沿的稳定发布周期之一。尽管 Fedora 有 [多种衍生版本](https://fedoraproject.org/spins/) 使用其他桌面环境，其中大多数尚未完全支持 Wayland，但这标志着 Wayland 采用的一个转折点。 ®
