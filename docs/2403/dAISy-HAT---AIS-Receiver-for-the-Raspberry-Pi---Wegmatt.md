<!--yml

category: 未分类

date: 2024-05-27 14:45:19

-->

# dAISy HAT - 树莓派AIS接收器 – Wegmatt

> 来源：[https://shop.wegmatt.com/products/daisy-hat-ais-receiver](https://shop.wegmatt.com/products/daisy-hat-ais-receiver)

dAISy HAT是您的[树莓派项目](https://www.partmarine.com/blog/wireless_ais_howto/)和[嵌入式应用](http://mdbuoyproject.wixsite.com/default/single-post/2017/03/02/V3-Buoy-Introduction)的完美AIS接收器。这款双通道海洋AIS接收器与OpenCPN、Kplex和其他接受串行数据输入的软件完美配合。它还非常适合将本地船舶交通报告给MarineTraffic等服务。

对于嵌入式应用，还请查看我们的新款[dAISy FeatherWing](https://shop.wegmatt.com/products/daisy-featherwing-ais-receiver "dAISy FeatherWing AIS接收器")和[dAISy Mini](https://shop.wegmatt.com/products/daisy-mini-ais-receiver "dAISy Mini AIS接收器")型号。

我们还有具有USB连接功能的AIS接收器[这里](https://shop.wegmatt.com/products/daisy-2-dual-channel-ais-receiver-with-nmea-0183 "dAISy 2+双通道AIS接收器，带有NMEA 0183输出")和[这里](https://shop.wegmatt.com/products/daisy-ais-receiver "dAISy AIS接收器")。

还可以查看[配件](https://shop.wegmatt.com/collections/accessories)部分，了解与[Pi 3的外壳](https://shop.wegmatt.com/products/raspberry-pi-case-for-daisy-hat "树莓派3用外壳")和[Pi 4的外壳](https://shop.wegmatt.com/collections/accessories/products/raspberry-pi-4-case-for-daisy-hat "树莓派4用外壳")匹配的[外壳](https://shop.wegmatt.com/products/uputronics-filtered-preamplifier-for-ais "Uputronics过滤前置放大器")和[RF适配器和电缆](https://shop.wegmatt.com/collections/accessories "RF适配器和电缆")。

### 规格

+   真正的双通道接收器，连续接收AIS通道A（161.975 MHz）和B（162.025 MHz）

+   与其他低成本AIS接收器相比，具有更高的灵敏度

+   低功耗，在接收模式下少于200mW（5V时<40mA）

+   以行业标准NMEA格式（AIVDM）输出38400波特率的串行输出

+   通过UART0（serial0）与树莓派通信

+   适用于树莓派1（仅限A+/B+）、树莓派2、树莓派3和4（见下面的说明），以及树莓派Zero

+   形状和尺寸符合树莓派HAT标准

+   为2个独立的TTL串行输出提供分离的分频板，3.3伏和5伏电源，以及树莓派I2C端口

+   SMA天线连接器

+   包括SMA到BNC适配器和六角支架

注意：在树莓派3和4上，UART0被内置的蓝牙电台占用。使用dAISy HAT时，蓝牙可能不可用。

### dAISy非常适合OpenCPN和OpenPlotter

[OpenCPN](http://opencpn.org/)，一款开源的海图绘制和导航软件，可与dAISy等接收器一起使用，跟踪地图上的船只。树莓派上运行的任何接受来自串行输入的AIS数据的软件都适用于dAISy。

我们的几位客户基于树莓派、OpenCPN和dAISy构建了图表绘图仪。详细的逐步教程可以在[这里](https://mvcesc.wordpress.com/2017/04/12/a-year-later-opencpn-on-a-raspberry-pi-3/)找到。

使用[Kplex](http://www.stripydog.com/kplex/examples/index.html)，树莓派可以设置为一个NMEA服务器，将AIS数据分发到网络中的其他设备，例如通过WiFi传输到iPad或Android平板上运行的导航应用程序。

**免责声明：我们不建议仅依赖dAISy进行导航和避碰！**

### 向MarineTraffic报告

dAISy HAT非常适合向AIS跟踪网站和像[MarineTraffic](http://www.marinetraffic.com/)、[FleetMon](https://www.fleetmon.com/)、[AISHub](http://www.aishub.net/)或[Pocket Mariner](http://pocketmariner.com/ais-ship-tracking/cover-your-area/)等服务报告附近的船舶交通情况。

最简单的方法是使用dAISy与[Kplex](http://www.stripydog.com/kplex/examples/index.html)。Kplex是一个NMEA-0183复用器，允许将AIS数据流发送到多个目标，包括像MarineTraffic和AISHub这样的服务。

高级用户可以使用几行Python代码，甚至[基本的Linux命令](http://forum.43oh.com/topic/4833-potm-daisy-a-simple-ais-receiver/?p=49778)直接将dAISy的串行输出提交给AIS跟踪服务。

### Breakout Pads

dAISy HAT包括便捷的breakout pads以访问AIS数据流。通过一点点的焊接，高级用户可以将dAISy与蓝牙模块结合起来，在无线设备上接收AIS，连接数据记录器如[SparkFun OpenLog](https://github.com/sparkfun/OpenLog/wiki)，或任何其他可以处理串行数据的设备。

+   Serial 1将发送到树莓派的串行数据进行镜像，速率为38400波特率。此输出始终启用。

+   Serial 2默认情况下处于禁用状态，可以配置为不同速率（4800、9600和38400波特率）的输入或输出。

+   TX/RX电压级别为3.3V。

+   5V接口连接到树莓派的5V电源线，支持的最大负载取决于电源供应。

+   3V3 pads连接到dAISy的3.3V电源线，支持的最大负载约为150mA。

+   I2C breakout pads直接连接到树莓派，便于向项目中添加传感器。

### 独立操作

当通过5V或3.3V的断开的pads供电时，dAISy HAT也可以在没有树莓派的情况下工作。使用串行断开的pads将其连接到TTL串口转USB电缆、蓝牙模块或其他设备。

### dAISy的比较方式

通过[我们的开源工作](https://github.com/astuder/dAISy)，我们开创了一种围绕[SiLabs EZRadioPro](http://www.silabs.com/products/wireless/ezradiopro/pages/si4362.aspx)单芯片无线电IC设计的新类别的AIS接收器。这种架构对于价格非常低廉、体积小和低功耗至关重要。其缺点是较长的获取时间、较低的灵敏度和比传统AIS接收器更少的范围。

采用传统双通道AIS接收器进行的现场测试显示，在良好条件下，dAISy HAT可以媲美某些型号（例如SmartRadio SR162），但仍然被其他型号（例如advanSEA RX-100）所超越。然而，在我们迄今为止测试的所有低于100美元的接收器中，dAISy显然表现出色（Quark-elec，MarineGadget）。

我们的观察是，配备良好天线的 dAISy 非常适合监控当地船舶交通。如果您需要最佳性能并且愿意为此付费，请选择来自传统品牌的接收器。

### 应保修，发货等。

我们的AIS接收器在美国进行专业组装。我们为产品提供12个月的保修（不包括物理或水损坏）。如果您的AIS接收器提前出现故障或遇到任何其他技术问题，请[与我们联系](https://shop.wegmatt.com/pages/contact-us "联系我们")。

我们使用美国邮政优先邮递服务、UPS和DHL快递进行发货。优先邮件递送非常可靠，但仅适用于[少数几个国家](https://pe.usps.com/text/imm/immc2_022.htm#ep2899642 "具有端到端跟踪的国家")，如果您有特殊的发货要求，请与我们联系。

**注意：对于国际运输，您当地的海关可能会征收增值税和其他进口或处理费用。这些是收件人的责任。我们将始终声明购买价格不包括运费。**

**在欧盟境内进行履约**

根据过往经验，我们强烈建议我们的[欧洲经销商](https://wegmatt.com/wheretobuy.html "经销商")供应商给德国、葡萄牙和希腊的客户。
