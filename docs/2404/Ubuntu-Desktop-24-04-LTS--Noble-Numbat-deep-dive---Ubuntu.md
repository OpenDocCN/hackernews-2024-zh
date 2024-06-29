<!--yml

category: 未分类

date: 2024-05-27 13:35:26

-->

# Ubuntu Desktop 24.04 LTS：Noble Numbat深入剖析 | Ubuntu

> 来源：[https://ubuntu.com/blog/ubuntu-desktop-24-04-noble-numbat-deep-dive](https://ubuntu.com/blog/ubuntu-desktop-24-04-noble-numbat-deep-dive)

[https://www.youtube.com/embed/q5yM4ZYwB_s?feature=oembed](https://www.youtube.com/embed/q5yM4ZYwB_s?feature=oembed)

VIDEO

二十年的积淀。Ubuntu 24.04 LTS将Linux生态系统的最新进展汇聚在一起，致力于赋予开源开发者力量，并为未来十二年的创新提供支持。

迈向Noble Numbat的道路被证明是一个充满挑战的旅程，通过逐步增强的中间版本进行实验，尝试新的安全方法（并解决[最后一分钟的CVE](https://discourse.ubuntu.com/t/xz-liblzma-security-update-post-2/43801)），发展我们的核心桌面应用程序，并继续致力于性能和兼容性，支持全新Linux 6.8内核所支持的广泛硬件。

虽然每个LTS都是一个重要的里程碑，但绝不是最终目的地。我们期待在Ubuntu 24.04 LTS的生命周期内及未来发布中继续扩展和增强我们今天提供的内容，始终考虑如何实现[我们的使命](https://ubuntu.com/about)，以及Ubuntu Desktop的[价值观](https://ubuntu.com/blog/ubuntu-desktop-charting-a-course-for-the-future)。

让我们深入了解细节。

## 重新思考供应

解决“我如何在这台机器上安装Ubuntu？”这一基本问题仍然是我们的首要任务之一。尽管如今Ubuntu与戴尔、惠普和联想等OEM合作，在全球数百万台台式机、笔记本电脑和工作站上预装，但每年有超过十倍的用户自行安装操作系统。以下是我们为简化Ubuntu安装所添加的功能。

### 统一技术栈

在过去几个临时发布版本中，我们已经调整了桌面安装程序的基础技术堆栈，使用与Ubuntu服务器相同的[Subiquity](https://canonical-subiquity.readthedocs-hosted.com/en/latest/)后端，创建了跨平台的一致代码库，以提供功能平等和更易维护性。这得益于Flutter构建的全新前端，经过过去一年的大幅迭代，改进了辅助功能选项的访问、提高了用户体验的清晰度，实现了打磨和改进的体验。

### 附加加密选项

作为这一迁移的一部分，我们已经重新引入了ZFS引导安装作为文件系统选项，并添加了ZFS加密支持。我们还为双启动设置提供了改进的指导，特别是与BitLocker相关的设置。用户的一个主要需求是硬件支持的全磁盘加密，在Ubuntu 24.04 LTS中以实验形式首次亮相。在推出时，此实施具有一定的限制，限制其仅适用于仅需要通用内核且无第三方驱动程序或内核模块的设备，并且目前不支持固件升级。我们计划在此版本的生命周期内逐步扩展此功能的硬件兼容性，支持NVIDIA驱动作为我们的首要任务。

### 集成自动安装

其中最令人兴奋的新功能之一是[自动安装](https://ubuntu.com/server/docs/install/autoinstall)支持在图形安装程序中的展示。希望创建定制化、可重复、自动化安装流程的用户或企业现在可以提供本地或远程autoinstall.yaml文件的地址，然后让Subiquity接管安装流程。

查看这个[入门教程](https://blog.local-optimum.net/getting-started-with-autoinstall-on-ubuntu-desktop-24-04-lts-147a1defb2de)，看看如何轻松地自动创建用户、安装附加应用程序并配置适用于多台机器的文件系统格式。

这使我们更接近[零触摸配置](https://en.wikipedia.org/wiki/Zero-touch_provisioning)的长期目标，我们计划在以后的日期为企业环境中访问受保护的autoinstall文件添加额外的SSO身份验证支持。

## 新的核心应用

安装完Ubuntu桌面后新功能并未停止。新的应用中心（也基于flutter）是另一个显著亮点，通过更现代、更高效的新外观提升应用发现，提供更清晰的类别和应用管理功能。自其[初次发布](https://ubuntu.com/blog/ubuntu-desktop-23-10-mantic-minotaur-deep-dive)以来，应用中心现在包括一个新的评分服务，允许用户评价他们的应用质量并查看其他用户的综合评分。这些评分与从[Snap Store](https://snapcraft.io/)获得的其他丰富元数据结合，将使我们能够提供额外的发现机制，例如排行榜、最受欢迎的或最近更新的应用。

尽管应用中心默认为snap中心化视图，以便我们提供这些可用性功能，您仍然可以通过搜索切换器查找并安装deb包。

作为新应用中心开发的一部分，我们将固件更新分离为自己专用的应用。这不仅允许更丰富的管理固件体验，还提高了性能，因为旧的Ubuntu软件应用程序需要保持在后台运行以检查以前版本的新固件。

## GNEW GNOME

Ubuntu Desktop 24.04 LTS继续致力于使用最新和最好的GNOME，版本为46。此版本带来了大量性能和可用性改进，包括文件管理器搜索和性能优化，可扩展通知以及为了更便于访问而整合的设置选项。

与往常一样，Ubuntu在GNOME提供的出色基础上进行了多项扩展和增加。颜色选择器允许用户根据自己的喜好定制桌面高亮显示，三重缓冲改善了Intel和Raspberry Pi图形驱动的性能，而添加的Tiling Assistant扩展则实现了四分之一屏幕平铺支持，以改进工作区管理。

## 使用Netplan 1.0实现桌面和服务器的一致网络配置

在Ubuntu 23.10中，我们将Netplan作为桌面网络配置的默认工具，统一了自2016年以来在服务器和云端的默认堆栈。此变更使管理员能够在不考虑平台的情况下一致地配置他们的Ubuntu系统。随着Netplan 1.0的最新发布，所有平台也受益于围绕无线兼容性和可用性改进的新功能，如netplan status –diff。

值得注意的是，Netplan并不取代NetworkManager，并不会影响那些倾向于之前配置方法的工作流程。NetworkManager与Netplan具有双向集成，这意味着在任一配置中进行的更改都会更新并反映在两者中。

您可以在Lukas的[先前博客](https://ubuntu.com/blog/netplan-configuration-across-desktop-server-cloud-and-iot)中了解更多关于这种双向性的信息。要了解Netplan 1.0的新功能，请查看他的[最新公告](https://ubuntu.com/blog/introducing-netplan-v1)。

## 与Active Directory全面兼容的GPO支持

Ubuntu桌面在全球企业、学术和联邦机构的工程和数据科学团队中非常普及，而Windows则仍然是其他部门的企业操作系统首选。[Canonical的Landscape](https://ubuntu.com/landscape)在跨桌面、服务器和云端监视、管理和报告Ubuntu实例的合规性方面非常有效，但桌面IT管理员经常在寻找帮助他们管理混合Ubuntu和Windows设备的解决方案。

多年来，本地Active Directory一直是Windows管理员首选的管理工具，并且仍然代表了大多数组织的主要份额。在Linux上使用Active Directory进行用户身份验证已经成为一种标准，作为系统服务安全守护进程（SSSD）的一部分。然而，在Ubuntu 22.04 LTS中，我们[引入了额外的支持](https://ubuntu.com/engage/microsoft-active-directory)用于组策略对象（GPOs），允许进一步的合规配置。在我们的临时发布过程中，这种GPO支持已扩展到覆盖由Active Directory管理员请求的大多数设备和用户策略，包括：

+   权限管理和移除本地管理员

+   远程脚本执行

+   管理apparmor配置文件

+   配置网络共享

+   配置代理设置

+   自动证书颁发

除了[现有的政策](https://canonical-adsys.readthedocs-hosted.com/en/latest/reference/policies/)在Ubuntu 22.04 LTS上可用之外。这为希望通过Ubuntu桌面赋予其开发人员权力的管理员提供了一流的解决方案。

接下来，我们的注意力现在转向支持第三方基于云的身份提供商，这是在Ubuntu 23.04中实施Azure Active Directory注册概念验证后的发展方向。我们目前正在扩展该发布中提供的功能，作为新实施的一部分，并期待在不久的将来更多地讨论此事。

最后，对于那些由于内部政策要求而继续使用Windows的开发人员，我们正在继续投资于[Windows子系统上的Ubuntu](https://ubuntu.com/desktop/wsl)（WSL）的企业工具。 Ubuntu 24.04 LTS支持cloud-init [实例初始化](https://canonical-ubuntu-wsl.readthedocs-hosted.com/en/latest/tutorials/cloud-init/)，使管理员能够在开发者的机器上种植自定义配置文件，以创建标准化的Ubuntu环境。这比现有的导入/导出工作流程更为健壮，并代表了未来管理和合规工具的第一步。

## Ubuntu Desktop 24.04 LTS中的安全软件管理

在Ubuntu 24.04 LTS的内部，还包括了一些针对Ubuntu生态系统内开发和分发软件的安全改进。在Ubuntu 23.10中，我们引入了[软件源的新版本](https://discourse.ubuntu.com/t/improvements-to-ppa-management-in-23-10/35783)，改变了在Ubuntu上管理个人软件包存档（PPAs）的方式。

PPA（个人软件包存档）是开发、测试和定制的关键工具，允许用户安装官方Ubuntu存档之外的软件。这不仅允许了很大程度上的软件自由，但也由于它们获得了对您的操作系统的访问权，存在潜在的安全风险。在Ubuntu 24.04 LTS中，PPA现在以deb822格式化的.sources文件分发，其签名密钥直接嵌入文件的signed-by字段中。这建立了密钥与存储库之间的1：1关系，意味着一个密钥不能用于签署多个存储库，删除存储库也将删除其关联的密钥。此外，APT现在要求使用更强的公钥算法对存储库进行签名。

### 限制非特权用户命名空间

另一个重要的安全增强功能是限制非特权用户命名空间。这些是Linux内核的广泛使用功能，为构建自己的沙盒（例如浏览器用于执行不受信任的Web内容的应用程序）提供了额外的安全隔离。到目前为止一切顺利，然而创建非特权用户命名空间的能力可能会暴露Linux内核中的额外攻击面，并已被证明是大量攻击利用的一步。在Ubuntu 24.04 LTS中，AppAmor现在用于基于应用程序的需要选择性地控制对非特权用户命名空间的访问，因此只有有正当需求的应用程序可以利用这一功能。

您可以在安全团队的[深入探讨](https://ubuntu.com/blog/whats-new-in-security-for-ubuntu-24-04-lts)中了解有关这一变更以及最新Ubuntu版本的其他安全增强功能。

### 改进的建议存储区

建议的存储区用作在向更广泛的Ubuntu用户群发布之前对软件更新的暂存区。过去，这个存储区是一个全包或不包的体验，选择从建议中更新的用户需要接受所有可用的更新。因此，引入系统不稳定性的机会显著增加，使那些希望提前提供特定功能测试支持的用户失去了动力。

在Ubuntu 24.04 LTS中，我们降低了“建议”更新的默认apt优先级，以允许用户明确指定他们想要安装的软件包及其想要保持稳定的软件包。这一变更旨在增加那些希望在其一般发布之前测试特定功能的用户的信心。

## 共同构建未来

这让我们深入探讨了最新的Ubuntu桌面长期支持版本中一些功能背后的动机和决策。过去三个临时发布周期中，看到这些构建模块如何汇聚在一起，是一次具有挑战性和激动人心的经历。对于Ubuntu桌面24.04 LTS，我们的目标是打造一个经得起时间考验的平台，为您的下一个伟大开源项目奠定基础。

故事将继续。感谢您的加入。

## 今天就开始吧
