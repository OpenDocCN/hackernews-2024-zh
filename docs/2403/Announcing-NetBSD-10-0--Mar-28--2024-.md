<!--yml

类别：未分类

日期：2024-05-29 12:47:37

-->

# 宣布NetBSD 10.0（2024年3月28日）

> 来源：[https://www.netbsd.org/releases/formal-10/NetBSD-10.0.html](https://www.netbsd.org/releases/formal-10/NetBSD-10.0.html)

# 宣布NetBSD 10.0（2024年3月28日）

NetBSD项目很高兴地宣布NetBSD 10.0，这是NetBSD操作系统的第十八个主要发布版。

它代表自2019年NetBSD 9.x最初分支以来对操作系统的累积改进。

可通过引导安装映像并选择升级选项来升级现有安装。

如果您使用其他升级方法，请先升级内核和模块，然后重新启动并升级用户空间。您需要调整任何软件包存储库的URL并更新所有第三方软件包。还请注意新增的`gpufw`集合，可能需要单独安装，使用[sysinst(8)](//man.NetBSD.org/NetBSD-10.0/sysinst.8)进行操作。

如果您从早期版本升级，请特别注意*系统行为和兼容性的更改*。

### 性能和可伸缩性

[NetBSD 10的基准测试](https://mail-index.NetBSD.org/current-users/2020/11/07/msg039815.html)显示相对于NetBSD 9.x，在多处理器和多核系统上，特别是计算和文件系统密集型应用中，性能和可伸缩性有了巨大提升。改进的领域包括：

+   将内核的文件路径查找缓存切换为更快的每目录红黑树。

+   提高了调度程序的性能，包括更好地在慢速和快速核心（例如 big.LITTLE Arm CPU）的混合环境中分配负载能力。

+   为机器独立虚拟内存系统进行了各种优化：

    +   切换到更快的基数树算法进行内存页查找。

    +   改进了对大文件上[fsync(2)](//man.NetBSD.org/NetBSD-10.0/fsync.2)的跟踪：几个数量级的加速。

    +   改进了并行化：重写了页面分配器以了解CPU拓扑，用每CPU计数器替换了全局计数器，并减少了锁竞争。

+   提高了[select(2)](//man.NetBSD.org/NetBSD-10.0/select.2)和[poll(2)](//man.NetBSD.org/NetBSD-10.0/poll.2)系统调用的性能。

+   提高了`tmpfs`的性能。实现了atime/mtime的延迟更新。

+   针对依赖于架构的x86和AArch64代码进行了各种优化，大幅改进了aarch64的网络和I/O吞吐量。

+   各种引导速度改进。

### 安全性和质量保证

+   与WireGuard兼容：

    +   新接口，[wg(4)](//man.NetBSD.org/NetBSD-10.0/wg.4)，提供了与WireGuard规范兼容的VPN隧道。*驱动程序处于实验阶段，需要进一步测试。*

    +   包含一个使用rump内核服务器的用户空间实现，详见[wg-userspace(8)](//man.NetBSD.org/NetBSD-10.0/wg-userspace.8)

    +   NetBSD实现与商业VPN提供者、Android、Linux等使用的WireGuard实现兼容。

+   更强、更快的加密技术：

    +   添加了Adiantum密码的实现，用于在没有AES加速的系统上通过[cgd(4)](//man.NetBSD.org/NetBSD-10.0/cgd.4)进行高效磁盘加密。

    +   添加了对[cgdconfig(8)](//man.NetBSD.org/NetBSD-10.0/cgdconfig.8)的共享密钥支持，允许在多个驱动器上使用多个派生子密钥。

    +   将默认密码哈希算法切换为Argon2id，这是密码哈希竞赛的获胜者。该算法的硬度随系统性能自动调节。将Argon2id支持添加到[cgdconfig(8)](//man.NetBSD.org/NetBSD-10.0/cgdconfig.8)以用于基于密码的磁盘加密。

    +   内核现在利用x86和Arm上常见的加密算法的CPU加速和矢量化，包括AES和ChaCha。所有内核中的AES实现现在在所有架构上都是恒定时间的。

    +   现在使用`vm.swap_encrypt=1` [sysctl(8)](//man.NetBSD.org/NetBSD-10.0/sysctl.8)变量自动进行交换加密。

+   支持新的Armv8-A安全功能：

    +   *特权访问绝不* - 帮助防止内核无意中访问用户空间内存。

    +   *指针认证* - 帮助防御针对缓冲区溢出的返回导向编程攻击。

    +   *分支目标识别* - 限制分支指令可以跳转的位置。

+   更多的消毒剂，测试能力和质量保证：

    +   *内核并发性检查器* - 在运行时检测内核中的竞态条件。

    +   *内核内存消毒剂* - 在运行时检测内核中未初始化的内存。

    +   新的虚拟USB主机驱动程序（vHCI）允许在用户空间中模糊测试和检测USB驱动程序中的错误，即使开发人员无法访问硬件。

    +   添加了2000多个新的测试用例。

    +   完成了对内部API使用的各种全面的内核范围审计：[membar_ops(3)](//man.NetBSD.org/NetBSD-10.0/membar_ops.3)，[autoconf(9)](//man.NetBSD.org/NetBSD-10.0/autoconf.9)，设备分离...

    +   添加了对硬链接创建的限制到[secmodel_extensions(9)](//man.NetBSD.org/NetBSD-10.0/secmodel_extensions.9)。

+   对Arm的支持改进：

    +   支持Allwinner V3s SoC，例如在Lichee Pi Zero中找到。

    +   支持Amlogic G12 SoC，例如在ODROID-N2+中找到。

    +   [Apple M1](https://wiki.NetBSD.org/ports/evbarm/apple/) SoC支持，例如M1 Mac Mini。

    +   NXP i.MX 8M SoC支持，例如在HummingBoard Pulse中找到。

    +   NXP i.MX 6SoloX SoC支持，例如在UDOO Neo Full中找到。

    +   支持Raspberry Pi 4。使用[EDK II UEFI固件](https://github.com/pftf/RPi4)从USB引导到SD卡，或将EDK II复制到`/boot`分区。

    +   Rockchip RK356X支持，例如在PINE64 Quartz64（安装了[EDK II UEFI固件](https://github.com/jaredmcneill/quartz64_uefi)）中找到。

    +   Rockchip RK3588支持，例如在Orange Pi 5（安装了[EDK II UEFI固件](https://github.com/edk2-porting/edk2-rk3588)）中找到。

    +   支持Rockchip RK3288，例如在Asus Tinker Board中找到。

    +   添加了在大端模式下引导Raspberry Pi 0-3的支持。

    +   添加对ACPI协作处理器性能控制的支持，用于服务器硬件的CPU性能调整。

    +   对AArch64上的[compat_linux(8)](//man.NetBSD.org/NetBSD-10.0/compat_linux.8)添加支持，使在`/etc/modules.conf`中启用`compat_linux`模块时可以运行Linux用户空间程序。

    +   添加对Rockchip RK3328上spiflash的支持。

    +   将对Xilinx Zynq-7000的支持移入`GENERIC` evbarm内核（基于FDT）。

    +   在PINE64 Rock64和NanoPi R2S上启用`rkv1crypto`驱动程序。

    +   UEFI引导加载程序改进：支持其他端ian FFS文件系统，从[raid(4)](//man.NetBSD.org/NetBSD-10.0/raid.4)卷启动，ISO9660（.iso文件系统）支持，`boot.cfg`支持，`gop`命令用于更改视频模式，直接从引导加载程序加载内核模块。

+   新的驱动程序：

    +   [aht20temp(4)](//man.NetBSD.org/NetBSD-10.0/aht20temp.4) - 用于Aosong AHT20温湿度传感器的驱动程序。

    +   [eqos(4)](//man.NetBSD.org/NetBSD-10.0/eqos.4) - 用于DesignWare以太网服务质量控制器的驱动程序。

    +   genet - 用于树莓派4上的Broadcom GENETv5以太网控制器的驱动程序。

    +   [ixl(4)](//man.NetBSD.org/NetBSD-10.0/ixl.4) - 用于英特尔以太网700系列10/25/40千兆以太网适配器的驱动程序。

    +   [iavf(4)](//man.NetBSD.org/NetBSD-10.0/iavf.4) - 用于英特尔以太网自适应虚拟功能的驱动程序。

    +   [mcommphy(4)](//man.NetBSD.org/NetBSD-10.0/mcommphy.4) - 用于Motorcomm YT8511C / YT8511H千兆以太网收发器的驱动程序。

    +   [mos(4)](//man.NetBSD.org/NetBSD-10.0/mos.4) - 用于MosChip MCS7730/7830/7832 USB以太网设备的驱动程序。

    +   [nct(4)](//man.NetBSD.org/NetBSD-10.0/nct.4) - 用于PC Engines APU系统上的Nuvoton NCT5104D GPIO控制器的驱动程序。

    +   pcf8574 - 用于某些SPARC64硬件上LED和指示器的GPIO驱动程序。

    +   [qat(4)](//man.NetBSD.org/NetBSD-10.0/qat.4) - 用于英特尔QuickAssist加密加速器的驱动程序。

    +   [rge(4)](//man.NetBSD.org/NetBSD-10.0/rge.4) - 用于Realtek 8125 2.5千兆以太网适配器的驱动程序。

    +   [scmd(4)](//man.NetBSD.org/NetBSD-10.0/scmd.4) - 用于Sparkfun串行控制电机的驱动程序。

    +   [sgp40mox(4)](//man.NetBSD.org/NetBSD-10.0/sgp40mox.4) - 用于Sensirion SGP40 MOx气体传感器的驱动程序。

    +   [sht3xtemp(4)](//man.NetBSD.org/NetBSD-10.0/sht3xtemp.4) - 用于Sensirion SHT30/SHT31/SHT35湿度/温度传感器的驱动程序。

    +   [sht4xtemp(4)](//man.NetBSD.org/NetBSD-10.0/sht4xtemp.4) - 用于Sensirion SHT40/SHT41/SHT45湿度/温度传感器的驱动程序。

    +   [uintuos(4)](//man.NetBSD.org/NetBSD-10.0/uintuos.4) - 用于Wacom Intuos绘图板笔的驱动程序。

    +   [wwanc(4)](//man.NetBSD.org/NetBSD-10.0/wwanc.4) - 用于英特尔XMM7360 LTE调制解调器的驱动程序。

+   改进的驱动程序：

    +   将内核中的GPU驱动程序与Linux 5.6同步，为英特尔（通过*i915*）、Nvidia（通过*nouveau*）和AMD（通过*amdgpu*和*radeon*）图形处理器带来大量新的硬件支持。

    +   [acpi(4)](//man.NetBSD.org/NetBSD-10.0/acpi.4) - 添加了用于访问ACPI表的字符设备`/dev/acpi`。[acpidump(8)](//man.NetBSD.org/NetBSD-10.0/acpidump.8)不再需要`options INSECURE`。

    +   amdsmn, amdzentemp - 添加了对AMD Family 17h/Axh、17h/6xh、19h/6xh、19h/7xh的支持。

    +   [aq(4)](//man.NetBSD.org/NetBSD-10.0/aq.4) - 为Aquantia 2.5/5/10 Gigabit以太网接口添加了硬件TCP/UDP接收校验和卸载支持，并支持Marvell AQC113 10G适配器。

    +   [bge(4)](//man.NetBSD.org/NetBSD-10.0/bge.4) - 移除了对大内核锁的要求（支持`NET_MPSAFE`内核选项，适用于Broadcom 10/100/Gigabit以太网接口）。

    +   [ciss(4)](//man.NetBSD.org/NetBSD-10.0/ciss.4) - 为支持的HP Smart Array RAID控制器添加了PERFORMANT模式和MSI/MSI-X支持。

    +   [ichsmb(4)](//man.NetBSD.org/NetBSD-10.0/ichsmb.4) - 添加了对Intel 700系列、Alder Lake-N和Snow Ridge设备的支持。

    +   [itesio(4)](//man.NetBSD.org/NetBSD-10.0/itesio.4) - 添加了对IT8625E传感器的支持。

    +   [ixv(4)](//man.NetBSD.org/NetBSD-10.0/ixv.4) - 添加了对在vmware ESXi上使用的邮箱API版本1.5的支持。

    +   [mcx(4)](//man.NetBSD.org/NetBSD-10.0/mcx.4) - 添加了硬件校验和卸载、硬件VLAN标记以及对Mellanox ConnectX多千兆以太网接口的多接收队列支持。

    +   [onewire(4)](//man.NetBSD.org/NetBSD-10.0/onewire.4), [owtemp(4)](//man.NetBSD.org/NetBSD-10.0/owtemp.4) - 减少了CPU开销，提高了可靠性。

    +   [pci(4)](//man.NetBSD.org/NetBSD-10.0/pci.4) - 添加了对Cavium ThunderX基础板上增强分配的支持。

    +   [pci(4)](//man.NetBSD.org/NetBSD-10.0/pci.4) - 添加了对更多PCIe 5.x解码支持。

    +   [pms(4)](//man.NetBSD.org/NetBSD-10.0/pms.4) - 改进了对Synaptics触摸板的支持。

    +   [tty(4)](//man.NetBSD.org/NetBSD-10.0/tty.4) - 从控制台tty中移除了使用“大内核锁”，提高了多核系统的响应速度。

    +   [u3g(4)](//man.NetBSD.org/NetBSD-10.0/u3g.4) - 添加了对ZTE MF112和D-Link DWM222 3G USB调制解调器的支持。

    +   [udl(4)](//man.NetBSD.org/NetBSD-10.0/udl.4) - 在刷新大多数静态USB显示器时提高了性能。

    +   [urndis(4)](//man.NetBSD.org/NetBSD-10.0/urndis.4) - 为OnePlus 5T添加了设备特性。

    +   [urtwn(4)](//man.NetBSD.org/NetBSD-10.0/urtwn.4) - 添加了对TRENDnet TEW-648UBM微型无线USB适配器的支持。

    +   [wm(4)](//man.NetBSD.org/NetBSD-10.0/wm.4) - 添加了对Intel Tiger Lake及更新设备的支持（I219V 15-V9和LM 16-19）。

    +   [xhci(4)](//man.NetBSD.org/NetBSD-10.0/xhci.4) - 添加了对等时通道管道的初始支持（例如用于USB 3.x网络摄像头）。

+   改进了对MIPS的支持：

    +   引导映像`octeon.img.gz`现在适用于Cavium OCTEON MIPS64板，例如Ubiquiti ERLite-3。 `ERLITE`内核配置已重命名为`OCTEON`。

    +   添加了对[ofctl(8)](//man.NetBSD.org/NetBSD-10.0/ofctl.8)和`/dev/openfirm`的支持，已在Cavium Octeon核心上启用。

    +   在 Cavium Octeon 上添加了平整设备树、USB 3 和 CPU 核心支持。

    +   添加了对内核模块的支持。

    +   移植了 [dtrace(1)](//man.NetBSD.org/NetBSD-10.0/dtrace.1) 和 [crash(8)](//man.NetBSD.org/NetBSD-10.0/crash.8) 内核调试器。

    +   增加了 O32 二进制文件的最大文本大小从 64MB 到 128MB。

+   改进了对复古硬件的支持：

    +   alpha：许多性能和 MP 稳定性改进。在 `GENERIC` 内核中默认启用了多处理器支持。

    +   amiga：支持 Kickstart 3.2（来自 2020 年的发布）。

    +   amiga：`loadbsd` 引导加载器现在将内核加载到最高优先级内存段，而不是最大段。

    +   atari：为 `ite(4)` 帧缓冲驱动添加了绘制字符支持。

    +   evbppc：为 DHT Walnut 405GP 板添加了支持。

    +   evbppc：为 Nintendo Wii 添加了支持。

    +   hp300：为 HP9000/425e 上的 EVRX 帧缓冲添加了位图操作支持。

    +   hp300：在所有 `punits` 上的 HPDisk 上添加了对多个 `rd(4)` 磁盘的支持。

    +   hppa：在 `GENERIC` 中启用了对内核模块的支持。

    +   luna68k：通过 [wskbd(4)](//man.NetBSD.org/NetBSD-10.0/wskbd.4) 添加了对键盘 LED 和蜂鸣器控制的支持。

    +   luna68k：为 YM2149 PSG/SSG 声音芯片添加了 `psgpam(4)` 驱动程序。

    +   luna68k：改进了帧缓冲图形和文本控制台性能。

    +   macppc：改进了 iMac G5 的兼容性：添加了对 GeForce 帧缓冲、CPU 温度和风扇传感器的支持。

    +   mac68k：在 Quadra/Centris AV 型号的 esp(4) SCSI 驱动程序中添加了同步传输支持。

    +   next68k：修复了许多问题以使端口重新工作。

    +   sparc：[wsdisplay(4)](//man.NetBSD.org/NetBSD-10.0/wsdisplay.4) 性能改进。

    +   sparc64：为 Sun Enterprise 250 添加了环境监控支持。

    +   x68k：在单体 X 服务器中添加了 Emulate3Buttons 支持。

    +   x68k：为 `ite(4)` 帧缓冲驱动添加了绘制字符支持。

    +   vax：从旧版本的 OpenBSD 移植了 VAXstation 3100 的 `gpx(4)` 和 `smg(4)` 帧缓冲驱动程序。

    +   vax：支持在仅有 8MB 至 512MB RAM 的机器上启动。

### 虚拟化改进

+   对 Xen 支持进行了许多改进：

    +   添加了对 Xen PVH 的支持。

    +   添加了对 Xen PV 驱动程序在 HVM 客户机下的支持。

    +   添加了对巨型帧和 paravirtualized 网络接口的支持。

    +   Dom0 内核现在启用了多处理器支持。

    +   Xen 内核现在使用与本地内核相同的内核模块。

    +   现在可以利用内核并行化的 paravirtualized 网络设备（[xennet(4)](//man.NetBSD.org/NetBSD-10.0/xennet.4)）、块设备（[xbd(4)](//man.NetBSD.org/NetBSD-10.0/xbd.4)）是 `MPSAFE`。

+   对 HyperV 支持进行了许多改进：

    +   在 vmbus 和 [hvn(4)](//man.NetBSD.org/NetBSD-10.0/hvn.4) 中添加了多通道支持。

    +   添加了在 [hvn(4)](//man.NetBSD.org/NetBSD-10.0/hvn.4) 中更改 MTU 和 TX 聚合的支持。

    +   改进了 VLAN 和 IP 校验和卸载支持。

+   VirtIO 驱动程序增强：

    +   将 [virtio(4)](//man.NetBSD.org/NetBSD-10.0/virtio.4) 驱动程序升级到支持版本 1.0，之前支持的是版本 0.9。

    +   新的 [vio9p(4)](//man.NetBSD.org/NetBSD-10.0/vio9p.4) 驱动程序允许挂载由 VM 主机导出的 VirtIO 9P 文件系统。

    +   新的 [viocon(4)](//man.NetBSD.org/NetBSD-10.0/viocon.4) 串行驱动程序。

+   NetBSD 虚拟机监视器 ([nvmm(4)](//man.NetBSD.org/NetBSD-10.0/nvmm.4)) 改进：

    +   允许主机在虚拟机运行时挂起和恢复。

    +   增加对 REP CMPS x86 指令的支持。

+   将对 QEMU 的虚拟 "mipssim" 机器的支持添加到 NetBSD/evbmips，包括 [virtio(4)](//man.NetBSD.org/NetBSD-10.0/virtio.4) 的扩展。

+   添加对在 QEMU 中运行 NetBSD/alpha 的支持。

+   增加对 VMware ESXi-Arm 和 Oracle Cloud 的 NetBSD/aarch64 支持。

### 功能和一般改进

+   网络堆栈改进：

    +   在内核的网络堆栈中实现了 RFC 7048，放宽了 IPv6 Neighbor Discovery 重传的规则。IPv6 邻居检测现在与地址无关，并且被 ARP 使用。

    +   [ipsec(4)](//man.NetBSD.org/NetBSD-10.0/ipsec.4) - 添加 net.key.allow_different_idtype [sysctl(7)](//man.NetBSD.org/NetBSD-10.0/sysctl.7) 选项，以提高与某些 VPN 设备的互联性。

    +   [lagg(4)](//man.NetBSD.org/NetBSD-10.0/lagg.4) - 新的可扩展链路聚合和链路故障转移接口，取代 [agr(4)](//man.NetBSD.org/NetBSD-10.0/agr.4)。

    +   [vether(4)](//man.NetBSD.org/NetBSD-10.0/vether.4) - 新的虚拟以太网接口，可配置地址用作 [bridge(4)](//man.NetBSD.org/NetBSD-10.0/bridge.4) 端点，在某些情况下取代 [tap(4)](//man.NetBSD.org/NetBSD-10.0/tap.4)。

+   文件系统和存储改进：

    +   向 FFS 添加 POSIX.1e 访问控制列表的支持，通过扩展属性（来自 FreeBSD）。引入了新的文件系统类型 FFSv2ea 用于此目的。用户可以使用 [fsck_ffs(8)](//man.NetBSD.org/NetBSD-10.0/fsck_ffs.8) 将文件系统升级为支持扩展属性。为了与之前的版本兼容，FFSv2ea 尚未成为默认值。

    +   各种 UDF 更改以实现与 Windows 10 上 UDF 文件系统的错误兼容性。

    +   [fstat(1)](//man.NetBSD.org/NetBSD-10.0/fstat.1) - 添加基本的 ZFS 支持。

    +   [refuse(3)](//man.NetBSD.org/NetBSD-10.0/refuse.3) - 现在支持从 FUSE 1.1 到 FUSE 3.10 的所有 FUSE API 变体。

    +   [raid(4)](//man.NetBSD.org/NetBSD-10.0/raid.4) - 添加对交换字节序配置的支持。

    +   [blkdiscard(8)](//man.NetBSD.org/NetBSD-10.0/blkdiscard.8) - 新的前端用于 [fdiscard(2)](//man.NetBSD.org/NetBSD-10.0/fdiscard.2) 手动 TRIM 磁盘。

    +   [raidctl(8)](//man.NetBSD.org/NetBSD-10.0/raidctl.8) - 添加 `-t` 选项以测试配置文件的有效性。

    +   [newfs_udf(8)](//man.NetBSD.org/NetBSD-10.0/newfs_udf.8) - 添加对 UDF 2.50 格式化的支持，包括元数据分区。

    +   [scan_ffs(8)](//man.NetBSD.org/NetBSD-10.0/scan_ffs.8) - 添加 `SIGINFO` 支持，按下 Ctrl+T 显示扫描状态。

+   新用户空间程序：

    +   [aiomixer(1)](//man.NetBSD.org/NetBSD-10.0/aiomixer.1) - 基于[curses(3)](//man.NetBSD.org/NetBSD-10.0/curses.3)的控制台音频混合器。

    +   [realpath(1)](//man.NetBSD.org/NetBSD-10.0/realpath.1) - 从相对路径打印绝对路径，包括解析符号链接。

    +   [tradcpp(1)](//man.NetBSD.org/NetBSD-10.0/tradcpp.1) - K&R风格的C宏处理器，用于基础程序中使用C预处理器宏的配置文件，但无需安装C编译器也能正常工作。

    +   [ioctlprint(1)](//man.NetBSD.org/NetBSD-10.0/ioctlprint.1) - 打印描述性`ioctl`值。

    +   [testpat(6)](//man.NetBSD.org/NetBSD-10.0/testpat.6) - 显示彩色测试模式。

    +   [warp(6)](//man.NetBSD.org/NetBSD-10.0/warp.6) - 经典的BSD太空战争游戏（版权由Larry Wall捐赠给NetBSD基金会）。

    +   [fsck_udf(8)](//man.NetBSD.org/NetBSD-10.0/fsck_udf.8) - 用于修复通用磁盘格式文件系统的损坏的新命令，使UDF成为适合跨系统共享磁盘的可靠读写选择。

+   用户空间程序的改进：

    +   [audioplay(1)](//man.NetBSD.org/NetBSD-10.0/audioplay.1) - 添加了解码64位和32位IEEE浮点RIFF WAVE文件的能力。

    +   [date(1)](//man.NetBSD.org/NetBSD-10.0/date.1) - 添加了`-f`选项来设置时间。

    +   [date(1)](//man.NetBSD.org/NetBSD-10.0/date.1) - 添加了`-R`选项，以RFC 5322格式显示时间，类似于GNU date。

    +   [df(1)](//man.NetBSD.org/NetBSD-10.0/df.1) - 添加了`-b`（输出单位：块；512）、`-H`（使用SI单位的`-h`）、`-N`（抑制头部行）和`-f`（仅显示空闲空间）选项。

    +   [env(1)](//man.NetBSD.org/NetBSD-10.0/env.1) - 添加了`-u`标志以移除环境变量，并添加了`-0`以允许用NUL字符分隔的变量输入。

    +   [ftp(1)](//man.NetBSD.org/NetBSD-10.0/ftp.1) - 添加了SSL/TLS证书验证。

    +   [ftp(1)](//man.NetBSD.org/NetBSD-10.0/ftp.1) - 跟随重定向到相对的HTTP URL。

    +   [ftp(1)](//man.NetBSD.org/NetBSD-10.0/ftp.1) - 增加了SSL连接设置超时，使用`-q QUITTIME`，默认为60秒。

    +   [grep(1)](//man.NetBSD.org/NetBSD-10.0/grep.1) - 使用`-r`而无文件参数时，搜索当前目录。

    +   [indent(1)](//man.NetBSD.org/NetBSD-10.0/indent.1) - 支持更新的标准C语法。

    +   [ldd(1)](//man.NetBSD.org/NetBSD-10.0/ldd.1) - 添加了`-v`选项以增加详细信息，并显示所有可执行文件处理错误。

    +   [make(1)](//man.NetBSD.org/NetBSD-10.0/make.1) - 添加了`randomize-targets`选项以帮助调试。

    +   [make(1)](//man.NetBSD.org/NetBSD-10.0/make.1) - 添加了`.break`关键字以提前终止`.for`循环。

    +   [mv(1)](//man.NetBSD.org/NetBSD-10.0/mv.1) - 添加了`-h`选项，以原子方式替换指向目录的符号链接。

    +   [netstat(1)](//man.NetBSD.org/NetBSD-10.0/netstat.1) - 添加了各种新的包计数器。

    +   [nbperf(1)](//man.NetBSD.org/NetBSD-10.0/nbperf.1) - 各种优化；将内存占用减少了30%。

    +   [patch(1)](//man.NetBSD.org/NetBSD-10.0/patch.1) - 添加了对具有过长行的文件打补丁的支持。

    +   [pgrep(1)](//man.NetBSD.org/NetBSD-10.0/pgrep.1) - 添加了`-q`标志以静默输出，如[grep(1)](//man.NetBSD.org/NetBSD-10.0/grep.1)中的操作。

    +   [pmap(1)](//man.NetBSD.org/NetBSD-10.0/pmap.1) - 添加了`-t`标志以将pmap打印为底层RB树。

    +   [ps(1)](//man.NetBSD.org/NetBSD-10.0/ps.1) - 添加了`-G`标志，以接受单个组参数，符合POSIX.2的要求。

    +   [rcp(1)](//man.NetBSD.org/NetBSD-10.0/rcp.1) - 添加了`SIGINFO`（ctrl+t状态）支持。

    +   [sh(1)](//man.NetBSD.org/NetBSD-10.0/sh.1), [ksh(1)](//man.NetBSD.org/NetBSD-10.0/ksh.1), [csh(1)](//man.NetBSD.org/NetBSD-10.0/csh.1) - 添加了`jobs -Z`以设置进程标题，就像zsh一样。

    +   [sh(1)](//man.NetBSD.org/NetBSD-10.0/sh.1) - 添加了命令自动完成。

    +   [sh(1)](//man.NetBSD.org/NetBSD-10.0/sh.1) - 添加了`-l`选项以强制创建登录Shell。

    +   [sh(1)](//man.NetBSD.org/NetBSD-10.0/sh.1) - 为`read`命令添加了`-d ''`选项，以配合即将发布的POSIX版本的`find -print0`和`xargs -0`兼容性。

    +   [script(1)](//man.NetBSD.org/NetBSD-10.0/script.1) - 正确回放[curses(3)](//man.NetBSD.org/NetBSD-10.0/curses.3)会话。

    +   [vacation(1)](//man.NetBSD.org/NetBSD-10.0/vacation.1) - 除了检查RFC 2076的'Precedence:'邮件头外，还检查'Auto-Submitted:'（RFC 3834）邮件头，并在'Precedence:'之外设置'Auto-Submitted:'。

    +   [vmstat(1)](//man.NetBSD.org/NetBSD-10.0/vmstat.1) - 添加了基于快速[sysctl(7)](//man.NetBSD.org/NetBSD-10.0/sysctl.7)的内核哈希统计生成。

    +   [worms(6)](//man.NetBSD.org/NetBSD-10.0/worms.6) - 添加了更多类型的worms，`-C`以使用颜色，`-S`选项以设置随机数生成器种子。

    +   [cgdconfig(8)](//man.NetBSD.org/NetBSD-10.0/cgdconfig.8) - 添加了`-T`选项以打印所有生成的密钥。

    +   [crash(8)](//man.NetBSD.org/NetBSD-10.0/crash.8) - 添加了对PowerPC和MIPS的支持。

    +   [httpd(8)](//man.NetBSD.org/NetBSD-10.0/httpd.8) - 添加了[blocklistd(8)](//man.NetBSD.org/NetBSD-10.0/blocklistd.8)支持。

    +   [httpd(8)](//man.NetBSD.org/NetBSD-10.0/httpd.8) - 添加了`-q`标志以禁用日志消息。

    +   [inetd(8)](//man.NetBSD.org/NetBSD-10.0/inetd.8) - 添加了`-f`标志以在前台运行。

    +   [iostat(8)](//man.NetBSD.org/NetBSD-10.0/iostat.8) - 添加了`-z`标志以抑制非活动设备的输出。

    +   [sysinst(8)](//man.NetBSD.org/NetBSD-10.0/sysinst.8) - 添加了对配置Wi-Fi设备的支持。

    +   [sysinst(8)](//man.NetBSD.org/NetBSD-10.0/sysinst.8) - 当启用多播DNS时，自动配置查找`.local`域在[nsswitch.conf(5)](//man.NetBSD.org/NetBSD-10.0/nsswitch.conf.5)中。

    +   [tprof(8)](//man.NetBSD.org/NetBSD-10.0/tprof.8) - 添加了`top`子命令，实时显示硬件分析结果。

    +   [tprof(8)](//man.NetBSD.org/NetBSD-10.0/tprof.8) - 添加了`count`子命令以输出原始性能事件计数器。

    +   [tprof(8)](//man.NetBSD.org/NetBSD-10.0/tprof.8) - 添加了对AMD第19h家族（Zen 3和Zen 4）、英特尔Comet Lake的性能分析支持。

    +   [wsfontload(8)](//man.NetBSD.org/NetBSD-10.0/wsfontload.8) - 添加了`-l`标志来列出所有加载的和内置的字体。

    +   [wsfontload(8)](//man.NetBSD.org/NetBSD-10.0/wsfontload.8) - 支持新的带有嵌入式元数据的字体格式。

    +   [wsmoused(8)](//man.NetBSD.org/NetBSD-10.0/wsmoused.8) - 添加了对绝对鼠标位置事件的支持，例如触摸屏。

+   新的和扩展的API：

    +   [eventfd(2)](//man.NetBSD.org/NetBSD-10.0/eventfd.2), [timerfd(2)](//man.NetBSD.org/NetBSD-10.0/timerfd.2) - 新的与Linux兼容的本地系统调用，也用于[compat_linux(8)](//man.NetBSD.org/NetBSD-10.0/compat_linux.8)。

    +   [fexecve(2)](//man.NetBSD.org/NetBSD-10.0/fexecve.2) - 新的系统调用，用于从文件描述符执行文件，符合The Open Group扩展API Set 2规范。

    +   [kqueue(2)](//man.NetBSD.org/NetBSD-10.0/kqueue.2) - 添加了`EVFILT_USER`用于用户建立的事件。

    +   [ppoll(2)](//man.NetBSD.org/NetBSD-10.0/ppoll.2) - 作为与其他操作系统兼容的本地系统调用`pollts`的别名。

    +   [curses(3)](//man.NetBSD.org/NetBSD-10.0/curses.3) - 添加了存根鼠标函数和`curses_version()`以与ncurses兼容。

    +   [curses(3)](//man.NetBSD.org/NetBSD-10.0/curses.3) - 改进了Unicode支持。

    +   [fetch(3)](//man.NetBSD.org/NetBSD-10.0/fetch.3) - 为TLS连接启用了Server Name Indication。

    +   [getentropy(3)](//man.NetBSD.org/NetBSD-10.0/getentropy.3) - 新的libc函数，用于从内核获取随机数据，类似于`KERN_ARND`，与其他各种操作系统兼容。

    +   [math(3)](//man.NetBSD.org/NetBSD-10.0/math.3) - 添加了更多长双精度函数的定义。

    +   [ossaudio(3)](//man.NetBSD.org/NetBSD-10.0/ossaudio.3) - 添加了对OSSv4混音器API的实现。

    +   [hosts_access(3)](//man.NetBSD.org/NetBSD-10.0/hosts_access.3) - 添加了[blocklistd(8)](//man.NetBSD.org/NetBSD-10.0/blocklistd.8)支持，使所有使用libwrap的程序能够阻止来自被拒绝主机的访问。

    +   [regex(3)](//man.NetBSD.org/NetBSD-10.0/regex.3) - 添加了本地语言支持，并支持GNU扩展（默认情况下关闭；来自FreeBSD）。

+   各种改进：

    +   将BSD许可的Spleen位图字体添加到X11集和`/usr/share/wscons/fonts`，并将它们设置为[ctwm(1)](//man.NetBSD.org/NetBSD-10.0/ctwm.1)的默认字体。

    +   将Terminus控制台字体添加到`/usr/share/wscons/fonts`。

    +   [wskbd(4)](//man.NetBSD.org/NetBSD-10.0/wskbd.4) - 添加了法国BPO和德国Neo 2布局的定义。

    +   [wsmouse(4)](//man.NetBSD.org/NetBSD-10.0/wsmouse.4) - 添加了与OpenBSD兼容的“精准滚动”事件类型，并在触摸板驱动程序和[Xorg(1)](//man.NetBSD.org/NetBSD-10.0/Xorg.1)中使用它们。

    +   [compat_linux(8)](//man.NetBSD.org/NetBSD-10.0/compat_linux.8) - 添加了`eventfd`、`timerfd`、POSIX定时器、`preadv`和`pwritev`。

### 系统行为和兼容性的变化

+   使用 [tap(4)](//man.NetBSD.org/NetBSD-10.0/tap.4) 作为 [bridge(4)](//man.NetBSD.org/NetBSD-10.0/bridge.4) 端点的网络设置必须更新为使用 [vether(4)](//man.NetBSD.org/NetBSD-10.0/vether.4)，因为现在 [tap(4)](//man.NetBSD.org/NetBSD-10.0/tap.4) 的链接状态基于应用程序是否已打开它。

+   由于安全原因，默认禁用了 [compat_linux(8)](//man.NetBSD.org/NetBSD-10.0/compat_linux.8)。要在启动时加载它，请将 `compat_linux` 添加到 `/etc/modules.conf`。

+   新安装的默认软件包数据库已更改为 `/usr/pkg/pkgdb`，以保持与其他 pkgsrc 平台的一致性，取代了 `/var/db/pkg`。

+   许多先前可能正常的系统现在可能会向内核消息缓冲区打印 **not enough entropy** 警告。这是因为现在只有经过审查的硬件源计入内核的熵估计。请参阅 [entropy(7)](//man.NetBSD.org/NetBSD-10.0/entropy.7)。

+   IEEE 802.11（Wi-Fi）设备现在需要配置 SSID 才能关联到开放接入点。

+   `blacklistd(8)`，一个可以根据需要阻止和释放端口以避免 DoS 滥用的守护程序，已重命名为 [blocklistd(8)](//man.NetBSD.org/NetBSD-10.0/blocklistd.8)。

+   将 `toor` 用户的默认 shell 更改为 `/rescue/sh`，以确保默认安装中存在具有静态链接 shell 的用户，以防出现问题。

+   现在 Xorg 根据 wscons 配置确定默认键盘布局，而不再使用 `/etc/xorg.conf`。要覆盖默认设置，请使用 [setxkbmap(1)](//man.NetBSD.org/NetBSD-10.0/setxkbmap.1)。

+   禁用了内核模块的自动卸载 - 现在内核模块必须选择自动卸载。

+   将 midi 和 sequencer 内核模块合并为单一的 `midi_seq` 模块。

+   arm：由于增加了 sdio 设备，ROCKPro64 的 [ld(4)](//man.NetBSD.org/NetBSD-10.0/ld.4) 磁盘设备顺序已更改。

+   arm：切换到 XZ 压缩集以用于 AArch64（请相应更新您的脚本）。

+   x86：HDMI 和 DisplayPort 音频已在 `GENERIC` 内核配置中启用。如果用户希望继续使用非 HDMI/DP 音频，可能需要使用 [audiocfg(1)](//man.NetBSD.org/NetBSD-10.0/audiocfg.1) 更改默认音频设备。

+   [crunchgen(1)](//man.NetBSD.org/NetBSD-10.0/crunchgen.1) - 删除了多个特殊变量处理标志，并替换为 `-V`。

+   [pam(8)](//man.NetBSD.org/NetBSD-10.0/pam.8) - `/etc/pam.d` 中默认禁用了 [pam_krb5(8)](//man.NetBSD.org/NetBSD-10.0/pam_krb5.8) 和 [pam_ksu(8)](//man.NetBSD.org/NetBSD-10.0/pam_ksu.8)。

+   [kqueue(2)](//man.NetBSD.org/NetBSD-10.0/kqueue.2) - 由于与其他 BSD 兼容性，`udata` 类型已从 `intptr_t` 更改为 `void *`。

+   [curses(3)](//man.NetBSD.org/NetBSD-10.0/curses.3) - 将默认颜色对更改为 0，与其他 curses 实现一致。

+   [proplib(3)](//man.NetBSD.org/NetBSD-10.0/proplib.3) - 各种API变更和新增。已被替换的旧API现在会产生弃用警告。

+   [iconv(3)](//man.NetBSD.org/NetBSD-10.0/iconv.3) - 输入参数更改为非const以符合当前POSIX标准，之前为了与其他标准（例如SUSv2）兼容而为const。

+   [resolver(3)](//man.NetBSD.org/NetBSD-10.0/resolver.3) - 默认更改为检查名称（参见[resolv.conf(5)](//man.NetBSD.org/NetBSD-10.0/resolv.conf.5)），这意味着包含无效字符的主机名将无法解析。

### 移除的过时组件

许多过时的组件被移除，旨在使网络堆栈和内核更易维护，并使未来的系统范围改进（例如改进的SMP）更加容易。在某些情况下，由于缺乏可用的硬件和兴趣，或包含严重的长期错误，无法测试已移除的驱动程序。

+   大部分被以太网取代的网络技术的驱动程序和支持：HIPPI、FDDI和TokenRing。

+   部分旧evbarm板的驱动程序和支持（需要自定义内核的那些），作为支持每个evbarm设备的`GENERIC`内核移动的一部分。已删除的硬件包括除了OMAP3530和AM335x之外的TI OMAP设备，如Gumstix和Hawkboard。

+   内核中的SMBFS - `nsmb(4)`和`mount_smbfs(8)`。这不支持现代版本的SMB协议，而用户空间实现更为功能强大。

+   内核中的IPv6路由广播处理 - 现在由用户空间的[dhcpcd(8)](//man.NetBSD.org/NetBSD-10.0/dhcpcd.8)处理。

+   `azalia(4)` - 曾在过去的版本中被[hdaudio(4)](//man.NetBSD.org/NetBSD-10.0/hdaudio.4)替换的驱动程序。

+   `de(4)` - 曾在过去的版本中被[tlp(4)](//man.NetBSD.org/NetBSD-10.0/tlp.4)替换的驱动程序。

+   `strip(4)` - 用于Metricom Ricochet数据收发的驱动程序。

+   `urio(4)` - 用于Diamond Multimedia Rio500 MP3播放器的驱动程序。

+   `uscanner(4)` - 用于非常老旧的USB扫描仪的驱动程序，请改用[ugen(4)](//man.NetBSD.org/NetBSD-10.0/ugen.4)和SANE。

+   `uyap(4)` - 用于USB YAP手机固件加载器的驱动程序。

+   `uyurex(4)` - 用于2008年Maywa-denki艺术团体制造的新奇设备的驱动程序。

+   `sup(1)` - 用于CMU软件升级协议的客户端。它在pkgsrc中可用。

+   在[umass(4)](//man.NetBSD.org/NetBSD-10.0/umass.4)中支持ISD的非标准ATA协议，用于早期Archos MP3播放器中的存储访问。

+   来自[queue(3)](//man.NetBSD.org/NetBSD-10.0/queue.3)的`CIRCLEQ`，由于指针别名违规，在NetBSD 7中已被弃用。

+   X11分发中的几个库：`libXTrap`、`libXevie`和`libglut`（虽然GLUT在现代X服务器中仍然有用，但建议libglut用户切换到pkgsrc中可用的FreeGLUT）。如有必要，可以继续通过pkgsrc安装`compat90`来使用已移除的库。

更新了NetBSD基本系统中包含的各种第三方组件：

完整的变更列表可以在NetBSD 10.0发行版树顶层目录中的[CHANGES](https://cdn.NetBSD.org/pub/NetBSD/NetBSD-10.0/CHANGES)和[CHANGES-10.0](https://cdn.NetBSD.org/pub/NetBSD/NetBSD-10.0/CHANGES-10.0)文件中找到。

NetBSD 10.0的完整源码和二进制文件可以在全球许多站点下载。您可以从我们的[主CDN](https://cdn.NetBSD.org/pub/NetBSD/NetBSD-10.0/)下载NetBSD 10.0，或者使用靠近您的[镜像站点](https://www.NetBSD.org/mirrors/)。由[NetBSD安全官员的PGP密钥](//cdn.NetBSD.org/pub/NetBSD/security/PGP/security-officer@netbsd.org.asc)签名的散列列表在[此文件](https://cdn.NetBSD.org/pub/NetBSD/security/hashes/NetBSD-10.0_hashes.asc)中可用于NetBSD 10.0发行版。

NetBSD是免费的。所有代码都采用非限制性许可证，并且可以无需向任何人支付版税而使用。我们通过邮件列表和网站提供免费支持服务。商业支持可从多个来源获得。有关NetBSD的更详细信息，请访问我们的网站：

## NetBSD 10.0支持的系统家族

NetBSD 10.0发行版为以下系统提供支持的二进制发行版：

发行版中包含但不完全支持或功能不全的端口：

NetBSD 10.0致力于已故的清水涼的记忆。

ryo@对NetBSD、我们的社区、ARM和网络的贡献（事实上，对这个发行版的贡献）是巨大的。我们对这位出色的技术贡献者和好朋友的失去感到深切悲伤。

NetBSD基金会要感谢所有多年来为我们的服务器提供代码、硬件、文档、资金、合作服务、网页和其他文档、发布工程以及其他资源的人员。有关使NetBSD成为可能的人员的更多信息，请访问：

我们还要感谢哥伦比亚大学计算机科学系的Tasty Lime和网络安全实验室提供的当前合作服务。感谢[Fastly](https://www.fastly.com/)提供CDN服务。

NetBSD是一个免费、快速、安全且高度可移植的类Unix开源操作系统。它适用于各种平台，从大型服务器和强大的桌面系统到手持设备和嵌入式设备。其清晰的设计和先进的特性使其在生产和研究环境中都表现出色，源代码在商业友好的许可证下免费提供。NetBSD由一个庞大而充满活力的国际社区开发和支持。许多应用程序可以通过[NetBSD软件包集pkgsrc](//pkgsrc.org)轻松获取。

## 关于NetBSD基金会

[NetBSD 基金会](../../foundation/) 成立于 1995 年，负责监督核心 NetBSD 项目服务，推广项目在行业和开源社区中的影响，并持有 NetBSD 代码库的知识产权。项目的日常运作由志愿者负责。

作为一家没有商业支持的非营利组织，NetBSD 基金会依赖于其用户的捐赠，我们希望您考虑向 [NetBSD 基金会捐款](../../donations/)，以支持我们优秀操作系统的持续开发。您的慷慨捐赠将特别受欢迎，用于持续升级和维护，以及支持 NetBSD 基金会的运营开支。

捐款可以通过 PayPal 转至 `<[paypal@NetBSD.org](mailto:paypal@NetBSD.org)>`，或通过 Google Checkout 进行，捐款在美国完全可抵税。有关更多信息，请访问 [www.NetBSD.org/donations/](//www.NetBSD.org/donations/#how-to-donate)，或直接联系 `<[finance-exec@NetBSD.org](mailto:finance-exec@NetBSD.org)>`。

* * *

返回

*[NetBSD 10.x 正式版本发布](./)*
