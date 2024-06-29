<!--yml

类别：未分类

日期：2024-05-27 13:30:27

-->

# QEMU 版本 9.0.0 发布 - QEMU

> 来源：[https://www.qemu.org/2024/04/23/qemu-9-0-0/](https://www.qemu.org/2024/04/23/qemu-9-0-0/)

# QEMU 版本 9.0.0 已发布

2024 年 4 月 23 日

我们很高兴宣布 QEMU 9.0.0 版本的发布。此版本包含来自 220 位作者的 2700 多次提交。

您可以从我们的[下载页面](https://www.qemu.org/download/#source)获取 tarball。完整的变更列表在[更改日志](https://wiki.qemu.org/ChangeLog/9.0)中可用。

重点包括：

+   块设备：virtio-blk 现在支持多队列，单个磁盘的不同队列可以由不同的 I/O 线程处理

+   gdbstub：各种改进，如在用户模式下捕获系统调用，支持 fork-follow 模式，以及支持 siginfo:read

+   内存：在某些情况下，内存后端的预分配现在可以通过多线程并发处理

+   迁移：支持“mapped-ram”功能，允许更高效的虚拟机快照，改进了对零页检测的支持，并为 VFIO 提供了检查点重新启动支持

+   ARM：ECV（增强计数虚拟化）、NV（嵌套虚拟化）和 NV2（增强嵌套虚拟化）的架构特性支持

+   ARM：B-L475E-IOT01A IoT 节点的板支持，mp3-an536（MPS3 开发板 + AN536 固件），以及 raspi4b（树莓派 4B 型号）

+   ARM：为 Freescale i.MX6、Allwinner R40、Banana Pi、npcm7xxx 和 virt boards 添加额外的 IO/磁盘/USB/SPI/以太网控制器和定时器支持

+   HPPA：大量错误修复，并更新 SeaBIOS-hppa 固件至 16 版本

+   龙芯架构：KVM 加速支持，包括 LSX/LASX 向量扩展

+   RISC-V：支持 Zacas、amocas、RVA22 profile 的 ISA/扩展支持，还有 Zaamo、Zalrsc、Ztso 等

+   RISC-V：RISC-V 虚拟机的 SMBIOS 支持，SRAT、SLIT、AIA、PLIC 的 ACPI 支持，以及更新的 RHCT 表支持和众多修复

+   s390x：模拟支持 CVDG、CVB、CVBY 和 CVBG 指令，并修复了 LAE（扩展加载地址）的模拟

+   还有更多内容……

感谢所有为此版本做出贡献的人，无论是编写代码、报告错误、改进文档、进行测试，还是提供项目的 CI 资源。没有你们，我们无法完成这些工作！
