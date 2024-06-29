<!--yml

category: 未分类

date: 2024-05-27 14:40:07

-->

# GPS天线改装使Starlink终端免疫干扰器 | Hackaday

> 来源：[https://hackaday.com/2024/03/06/gps-antenna-mods-make-starlink-terminal-immune-to-jammers/](https://hackaday.com/2024/03/06/gps-antenna-mods-make-starlink-terminal-immune-to-jammers/)

Starlink接收器需要定位和精准时间信息才能正常运行，目前获取该信息的最佳方式是使用全球导航卫星系统（GNSS）如GPS。不幸的是，用于这种次级卫星连接的天线留下了些许遗憾。当然，当解决Starlink问题时，没有人比[Oleg Kutkov]更擅长，他的责任是修复和改进在乌克兰使用的Starlink终端——特别是当具体问题是GPS频段受到入侵军方干扰时，[你最好相信改进即将到来](https://olegkutkov.me/2023/11/07/connecting-external-gps-antenna-to-the-starlink-terminal/)。

[Oleg]为我们描述了Starlink终端GPS电路的演变过程。然后，他向我们展示了您可以做的最简单的改装，比如用改进的被动天线代替当前使用的芯片天线进行焊接。接着，他更进一步，展示了您可以如何通过使用偏置模块安装主动天线，这种改装肯定不仅仅对这个设备有奇效！然后，他展示了测试结果表格——结果差异显著，具有主动天线改装的Starlink终端能够在存在主动干扰的区域获取GPS信号，而未改装的终端则不能。

这篇文章非常通俗易懂，对于任何对客户可访问设备中GPS天线接收问题感到困惑的人来说都是必读的。这不是我们看到的唯一一个由[Oleg]制作的Starlink硬件改装，我们刚刚报道了他关于在新型号中详细修复以及[修复以太网连接疏忽](https://hackaday.com/2024/02/28/restoring-starlinks-missing-ethernet-ports/)的Starlink以太网端口修复旅程的博客文章，博客上还有一篇关于[无需PoE供电的Starlink终端](https://olegkutkov.me/2023/06/09/how-to-power-starlink-terminal-from-12v-without-poe-injector-and-dc-dc-converters/)的文章，所以如果您想了解更多，请务必查看！
