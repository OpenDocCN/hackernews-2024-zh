<!--yml

分类：未分类

日期：2024-05-27 13:41:07

-->

# Ubuntu 24.04、Fedora 40、EndeavourOS 和 TrueNAS 24.04在这里 • The Register

> 来源：[https://www.theregister.com/2024/04/29/ubuntu_2404_fed_40_et_al/](https://www.theregister.com/2024/04/29/ubuntu_2404_fed_40_et_al/)

FOSS摘要 上周对开源社区来说非常繁忙：EndeavourOS 和 TrueNAS Scale 在星期二到达，Fedora 在星期三，Ubuntu 在星期四。

上周见证了一连串的新发行版发布，但它们之间存在相当大的重叠。它们都包括最新的[Linux kernel 6.8](https://www.theregister.com/2024/03/11/linux_6_8_arrives/)，所有发行版都提供最新的GNOME 46桌面环境，其[beta版本我们在二月份进行了检查](https://www.theregister.com/2024/02/21/gnome_46_beta_released/)，而大部分还包括最新的[KDE Plasma 6](https://www.theregister.com/2024/02/29/kde_plasma_60_released/)桌面环境。

第一个发布的是[EndeavourOS Gemini](https://endeavouros.com/news/plasma-6-with-wayland-or-x11-option-and-qt-6-ported-calamares-meet-gemini/)。我们最后一次在2022年六月查看了[EndeavourOS](https://www.theregister.com/2022/06/29/endeavouros_artemis_226/)，那是Artemis版本。它是基于Arch Linux的轻量级、易于安装的发行版，由于是滚动发布，新版本意味着更新的安装媒体，因此安装后不需要花费时间更新。此次发布包含KDE Plasma 6.0.4，提供X11或Wayland会话选择，以及来自上游的新nVidia二进制驱动程序包，改进的安装和更新工具。然而，有一件事它*不*包括，那就是面向Arm处理器的版本：由于人手不足，该项目[放弃了Arm支持](https://endeavouros.com/news/goodbye-endeavouros-arm/)。

### 大紫帽下是什么？

第二天，[Fedora 40发布](https://www.redhat.com/en/blog/announcing-fedora-linux-40)，正如我们在[查看发行候选版时预测的](https://www.theregister.com/2024/04/22/fedora_40_at_rc_stage/)。我们当时报道了其亮点，但也值得注意的是一些从这个版本中移除的内容，包括Delta RPMs、OpenSSL 1.1和Python 3.7。Fedora 40现在的默认版本是Python 3.12，因为3.7已经终止生命周期 — 而虽然它也已过期，但Python 3.6*仍然*可用，以便针对RHEL 8。

Fedora 不仅仅是**一个**发行版，而是一个完整的发行版家族，拥有多个不同版本。该家族有五个主要变种：[Fedora Workstation](https://fedoraproject.org/workstation/) 是旗舰 GNOME 桌面，还有十个 [Fedora Spins](https://fedoraproject.org/spins/) 提供不同的桌面环境，另外还有四个 [Atomic Desktops](https://fedoraproject.org/atomic-desktops/)，是不可变的图形化发行版。其中两个，[Fedora KDE Plasma Desktop](https://fedoraproject.org/spins/kde/) 及其不可变版本 [Fedora Kinoite](https://fedoraproject.org/atomic-desktops/kinoite/)，包括最新的 KDE Plasma 6 桌面，默认只支持 Wayland 会话。

此外，还有 [Fedora Server](https://fedoraproject.org/server/)，[Fedora IoT](https://fedoraproject.org/iot/) 专为 "边缘" 设备设计，[Fedora Cloud](https://fedoraproject.org/cloud/) 用于部署到本地或远程虚拟机，以及 [Fedora CoreOS](https://fedoraproject.org/coreos/)，是一个极简、不可变、自更新的容器主机。

我们总计有19个不同版本，因此总有一款适合每个人。尽管 Fedora 没有 LTS 发布，但它是略显备受诟病的 [CentOS Stream](https://www.theregister.com/2021/12/10/centos_stream_9/) 的上游，后者是红帽的较慢免费企业发行版。许多红帽用户更青睐 Stream 的前身 CentOS Linux，因为它更接近 RHEL 的兼容性，但即便如此，CentOS Stream 也拥有一些非常大规模的用户。例如，[2022年南加州Linux博览会](https://www.socallinuxexpo.org/scale/19x)的 [Building the Future](https://www.socallinuxexpo.org/scale/19x/presentations/building-future-centos-stream) 演讲证实，Facebook 母公司 Meta 广泛使用 CentOS Stream。Fedora 40 将成为未来 CentOS Stream 10 的基础，进而意味着有一天它将成为 RHEL 10。

### 在橙色国度

刚刚 Fedora 40 发布后，[Ubuntu 24.04 "Noble Numbat"](https://canonical.com/blog/canonical-releases-ubuntu-24-04-noble-numbat)也随之而来。这是一个长期支持版本，因此有些人至少会使用到2026年左右 - 尽管目前的 LTS 版本，[22.04 "Jammy Jellfish"](https://www.theregister.com/2022/04/21/ubuntu_22_04/)，在24.04.1推出之前不会收到升级提醒，计划于8月15日发布。如果你使用的是23.10版本，可能是为了避免收到关于 Ubuntu Pro 的烦人提醒，那么你将会更早收到提示。

Canonical 成立大约二十年了，因此24.04也是该公司的第十个 LTS 发布版 - 第一个是 Ubuntu 6.06 "Dapper Drake"，*Reg* [开始报道这个新兴的发行版](https://www.theregister.com/Print/2006/07/25/ubuntu_goes_mainstream/)，之前只是[简单提及](https://www.theregister.com/2006/01/31/db2_express/)过5.04 "Hoary Hedgehog"。

为了对即将到来的情况有所了解，我们几周前[查看了beta版本](https://www.theregister.com/2024/04/15/ubuntu_24_04_belated_beta/)。与其他版本不同的是，这个版本*不包含*KDE Plasma 6或[LXQt 2](https://www.theregister.com/2024/04/20/lxqt2_updates_to_qt6/)，尽管[基于Qt的风味将稍后提供更新](https://www.theregister.com/2024/04/19/qt_ubuntu_2404_betas/)。

除了光鲜的桌面外，一些新特性包括支持[新的bcachefs文件系统](https://www.theregister.com/2024/01/10/linux_kernel_67/)，内核将默认包含低延迟支持。它还默认启用了[帧指针](https://ubuntu.com/blog/ubuntu-performance-engineering-with-frame-pointers-by-default)，这个特性的性能开销非常小，不到百分之一，但可以使用[bpftrace](https://opensource.com/article/19/8/introduction-bpftrace)等工具进行性能分析。有关更多信息，Noble内核有自己的[发布说明](https://discourse.ubuntu.com/t/introducing-kernel-6-8-for-the-24-04-noble-numbat-release/41958)。

首次，服务器和主要桌面版本都使用新的Subiquity安装程序，具有改进的安装自动化支持。 "Noble"提供了TPM支持的全盘加密的实验性支持，以及对ZFS的引导安装支持。如果主机设备具有[适当的Intel芯片](https://www.theregister.com/2022/02/25/intel_xeon_d_openvino/)，Noble还支持Intel的QuickAssist压缩和加密加速功能。此外，还有一些红蒙特风格的集成，包括.NET 8，以及由于[ADSys GPO支持](https://canonical-adsys.readthedocs-hosted.com/en/stable/)，自动加入Microsoft Active Directory。

公司有[完整的发布说明](https://discourse.ubuntu.com/t/noble-numbat-release-notes/39890/1)，尽管它将各个风格的说明留给了各自的项目。 *Reg*开源桌面还没有完全查看所有的风味。两个风味推出了各自桌面的新版本。[Ubuntu Kylin 24.04](https://www.ubuntukylin.com/news/ubuntukylin2404-en.html)提供了[UKUI 4](https://www.ukui.org/news/shownews.php?id=129&lang=en)，尽管在西方被低估，但它是目前较为吸引人的Linux桌面之一。[Ubuntu Cinnamon 24.04](https://ubuntucinnamon.org/?p=1310)包括来自最新LinuxMint的[Cinnamon 6桌面](https://www.theregister.com/2023/12/01/cinnamon_6_kde_6b/)。除了我们已经涵盖的Lubuntu和Kubuntu的区别外，Xubuntu和Ubuntu Unity也有有趣的变化，我们计划在不久的将来回顾。

### 进入龙鱼（Enter the Dragon(fish)）

尽管 TrueNAS Scale 24.04 也在[周二发布](https://forums.truenas.com/t/truenas-scale-24-04-0-now-released/2485)，我们将它放在最后，因为它是一种与早期更为通用的发行版不同的产品。

上个月，我们提到 iXsystems 正将其基于FreeBSD的NAS发行版调整为维护状态，尽管公司向*Reg*的姐妹站[Blocks and Files](https://blocksandfiles.com/)保证这一变化不会使任何人陷入困境。这意味着 iXsystems NAS 操作系统的未来是基于[Debian Linux 的 TrueNAS Scale](https://www.theregister.com/2023/02/23/truenas_update/)。我们顺便提到当时它的版本是[23.10](https://www.truenas.com/docs/scale/23.10/gettingstarted/scalereleasenotes/)。

随着新版本 TrueNAS 24.04 "Dragonfish" 的发布，情况发生了变化。根据其[发布说明](https://www.truenas.com/docs/scale/24.04/gettingstarted/scalereleasenotes/)，这个版本仍然基于 Debian 12 和[OpenZFS 版本 2.3.3](https://www.theregister.com/2024/02/28/new_vers_bcachefs_zfs/)，所以变化不是很大。尽管如此，公司已经更新了重要组件，包括基于[最新 LTS 内核发布的更新内核，版本 6.6.20](https://www.theregister.com/2023/12/14/linux_kernel_of_the_beast/)。

Web GUI 中的SMB和NFS状态页面得到了改进，主仪表板页面现在有一个可选的备份状态小部件，UI 的其他部分也进行了简化。新增了一个[auditing logging](https://www.truenas.com/docs/scale/24.04/scaletutorials/systemsettings/auditingscale/)功能，用于跟踪管理员何时修改了哪些设置。"Dragonfish"可以与[FreeIPA server](https://www.freeipa.org/About.html)通信，进行LDAP认证。

用户社区贡献了两个其他特性：TrueNAS Scale 现在提供基于`systemd-nspawn`的Linux容器称为[Sandboxes](https://www.truenas.com/docs/scale/24.04/scaletutorials/apps/sandboxes/)，还新增了一个[Developer mode](https://www.truenas.com/docs/scale/24.04/scaletutorials/systemsettings/advanced/developermode/)，帮助定制操作系统，通常情况下是被锁定的。

这次发布中有一个[完整的更改日志](https://ixsystems.atlassian.net/issues/?filter=10541)，列出了本次更新的几十个修复。我们的印象是 TrueNAS Scale 的发展路径仍然保持着强烈的企业定位，这足以将其与已经拥挤的NAS发行版市场区分开来，其中许多更专注于家庭用户、媒体服务器等。
