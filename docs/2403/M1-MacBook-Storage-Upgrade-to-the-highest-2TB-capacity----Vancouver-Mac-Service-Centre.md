<!--yml

类别：未分类

日期：2024-05-27 14:43:24

-->

# M1 MacBook存储升级至最高的2TB容量！ - 温哥华Mac服务中心

> 来源：[https://vancouvermac.ca/repair/macbook-storage-upgrade/](https://vancouvermac.ca/repair/macbook-storage-upgrade/)

## 背景

曾经，MacBook存储升级是一个简单的10分钟任务，涉及从计算机的存储驱动器插槽中更换标准的2.5英寸SATA驱动器，并安装任何常用的替换部件。2012年，苹果开始为其新的刀片SSD驱动器采用专有的PCIe连接器，进一步加剧了情况的复杂性。然而，交换驱动器仍然很简单，只需几分钟时间。2015年，苹果开始在其12英寸MacBook型号中将SSD集成到逻辑板中。

结果，传输速度提高了，MacBook甚至可以做得比以往更薄。然而，这给消费者带来了各种问题。最重要的是，如果逻辑板损坏，数据将不可能被检索，这在逻辑板故障的情况下导致数据丢失。

## 硬盘空间不足

曾经，MacBook存储升级是因为存储空间不足的常见程序，由于这些设计变化，消费者唯一的选择是将整个MacBook升级到具有更多内置存储容量的其他型号。幸运的是，由于先进的焊接技术，现在可以将最新的MacBook存储空间升级到其最高配置，价格低于苹果在购买时最初报价的价格。

## 液体损坏故障

在2018年后的MacBook型号上，电源传输系统特别容易在液体损坏事件中导致灾难性故障。首先，SSD通常放置在MacBook的外边框上，使其最容易受到液体侵入。其次，电源传输系统涉及由BGA IC控制的降压转换器电路。这意味着高电压与供电SSD芯片的敏感低电压线之间仅有毫米之隔。液体很容易造成短路，将更高电压发送到SSD，毁坏芯片并阻止任何数据恢复的可能性。这种故障模式在16英寸2019年MacBook Pro A2141上特别常见，因为其设计特性如此。

## 绕过自定义固件

可以在所有MacBook Air和MacBook Pro型号上执行MacBook存储升级，但是对于包含SoC芯片的MacBook型号（例如2018-2020年间配备T2芯片的MacBook），由于加载到SSD NAND芯片上的定制配置固件，存在额外的复杂性。这些NAND芯片（根据主板和容量在2-8之间）在主逻辑板上共同工作以创建SSD阵列，但其配置被编码到每个芯片中，因此芯片的位置不能更改。因此，在温哥华Mac服务中心使用诸如[JC P13](https://www.jcprogrammer.com/product/p13-nand-programmer-hard-disk-unbind-and-purple-screen-repair-for-ios-jcid/)之类的工具来将定制固件转移到替换的SSD阵列。

搭载Apple Silicon SoC的MacBook，如M1、M2或M3系列芯片，也具有类似于配备T2芯片的Mac的数组配置的自定义固件。但是，与配备T2芯片的Mac不同，Apple Configurator实用程序可以通过其恢复功能自动重新编程SSD。这极大地提高了MacBook存储升级的可修复性和可能性，因为不需要为每个主板和存储容量都提供自定义固件转储。

## MacBook存储升级程序

升级程序相对简单——必须移除主逻辑板，并替换掉NAND芯片与替换零件。下面的视频详细介绍了该过程，重点关注安全传输芯片所需的焊接技术。

更换芯片后，通过另一台Mac通过USB-C运行Apple Configurator来重新编程SSD配置，并正确链接CPU到新的存储驱动器，重新分配加密参数。

## 升级您的MacBook存储

如果您想要升级您的MacBook存储，请[联系我们](https://vancouvermac.ca/contact/)！
