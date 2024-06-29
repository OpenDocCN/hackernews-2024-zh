<!--yml

类别：未分类

日期：2024-05-27 12:57:21

-->

# OpenBSD 7.5

> 来源：[https://www.openbsd.org/75.html](https://www.openbsd.org/75.html)

熟悉 OpenBSD 快速安装程序的信息，并使用 "[disklabel](https://man.openbsd.org/disklabel.8) -E" 命令。如果在安装 OpenBSD 时感到困惑，请阅读上述相关的 INSTALL.* 文件！

### OpenBSD/alpha:

如果您的设备可以从 CD 启动，则可以将 *install75.iso* 或 *cd75.iso* 写入 CD 并从中引导。有关更多详情，请参考 INSTALL.alpha。

### OpenBSD/amd64:

如果您的设备可以从 CD 启动，则可以将 *install75.iso* 或 *cd75.iso* 写入 CD 并从中引导。您可能需要先调整 BIOS 选项。

如果您的设备可以从 USB 启动，则可以将 *install75.img* 或 *miniroot75.img* 写入 USB 闪存驱动器并从中引导。

如果无法从 CD、软盘或 USB 启动，请按照附带的 INSTALL.amd64 文档中描述的方式通过 PXE 在网络上安装。

如果计划将 OpenBSD 与另一个操作系统双引导，您需要阅读 INSTALL.amd64。

### OpenBSD/arm64:

如果您的设备可以从 CD 启动，则可以将 *install75.iso* 或 *cd75.iso* 写入 CD 并从中引导。

要从磁盘引导，请将 *install75.img* 或 *miniroot75.img* 写入磁盘，并在连接到串行控制台后从中引导。有关更多详情，请参考 INSTALL.arm64。

### OpenBSD/armv7:

将系统专用 miniroot 写入 SD 卡，并在连接到串行控制台后从中引导。有关更多详情，请参考 INSTALL.armv7。

### OpenBSD/hppa:

按照 INSTALL.hppa 或 [hppa platform page](hppa.html#install) 中的说明在网络上引导。

### OpenBSD/i386:

如果您的设备可以从 CD 启动，则可以将 *install75.iso* 或 *cd75.iso* 写入 CD 并从中引导。您可能需要先调整 BIOS 选项。

如果您的设备可以从 USB 启动，则可以将 *install75.img* 或 *miniroot75.img* 写入 USB 闪存驱动器并从中引导。

如果无法从 CD、软盘或 USB 启动，请按照附带的 INSTALL.i386 文档中描述的方式通过 PXE 在网络上安装。

如果计划与另一个操作系统双引导 OpenBSD，则需要阅读 INSTALL.i386。

### OpenBSD/landisk:

将 *miniroot75.img* 写入 CF 或硬盘的开头，并正常引导。

### OpenBSD/loongson:

将 *miniroot75.img* 写入 USB 闪存驱动器并从中引导 bsd.rd，或者通过 tftp 引导 bsd.rd。有关更多详情，请参考 INSTALL.loongson 中的说明。

### OpenBSD/luna88k:

将 'boot' 和 'bsd.rd' 复制到 Mach 或 UniOS 分区，并从 PROM 引导加载引导程序，然后从引导程序加载 bsd.rd。有关更多详情，请参考 INSTALL.luna88k 中的说明。

### OpenBSD/macppc:

将映像从镜像站点刻录到 CDROM 上，并在开机时按住*C*键，直到显示器亮起并显示*OpenBSD/macppc boot*。

或者在 Open Firmware 提示符处输入 *boot cd:,ofwboot /7.5/macppc/bsd.rd*

### OpenBSD/octeon:

连接串行端口后，通过DHCP/tftp网络引导bsd.rd。有关更多详细信息，请参阅INSTALL.octeon中的说明。

### OpenBSD/powerpc64:

要安装，请将*install75.img*或*miniroot75.img*写入USB存储设备，将其插入机器，并在Petitboot中选择*OpenBSD install*菜单项。有关更多详细信息，请参阅INSTALL.powerpc64中的说明。

### OpenBSD/riscv64:

要安装，请将*install75.img*或*miniroot75.img*写入USB存储设备，并在插入该驱动器的情况下引导。确保还插入了随HiFive Unmatched板卡一起发货的microSD卡。有关更多详细信息，请参阅INSTALL.riscv64中的说明。

### OpenBSD/sparc64:

从镜像站点将映像刻录到CDROM中，从中引导，并输入*boot cdrom*。

如果这不起作用，或者您没有CDROM驱动器，您可以将*floppy75.img*或*floppyB75.img*（取决于您的机器）写入软盘，并使用*boot floppy*引导。有关详细信息，请参阅INSTALL.sparc64。

确保使用没有坏块的正确格式化软盘，否则您的安装很可能会失败。

您还可以将*miniroot75.img*写入磁盘上的交换分区，并使用*boot disk:b*引导。

如果什么都不起作用，您可以按照INSTALL.sparc64中描述的方式通过网络引导。

### 端口树

提供了一个端口树存档。要提取：

> ```
> # cd /usr
> # tar xvfz /tmp/ports.tar.gz
> 
> ```

如果您此时对端口一无所知，请阅读[ports](faq/ports/index.html)页面。此文本不是端口使用手册，而是一组旨在启动OpenBSD端口系统用户的注释。

*ports/*目录表示我们端口的CVS检出。与我们的完整源代码树一样，我们的端口树可以通过[AnonCVS](anoncvs.html)获得。因此，为了跟上稳定分支，您必须在可读写的介质上使*ports/*树可用，并使用如下命令更新树：

> ```
> # cd /usr/ports
> # cvs -d anoncvs@server.openbsd.org:/cvs update -Pd -rOPENBSD_7_5
> 
> ```

[当然，您必须将此处的服务器名称替换为附近的anoncvs服务器。]

请注意，大多数端口作为我们镜像上的软件包提供。如果出现问题，将提供7.5版本的更新端口。

如果您有兴趣添加端口，想要帮忙或只是想了解更多信息，请知晓邮件列表[ports@openbsd.org](mail.html)是一个很好的去处。
