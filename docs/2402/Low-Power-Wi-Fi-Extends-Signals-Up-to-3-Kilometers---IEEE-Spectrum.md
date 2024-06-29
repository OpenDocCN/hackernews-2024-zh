<!--yml

category: 未分类

date: 2024-05-27 14:33:33

-->

# 低功耗 Wi-Fi 将信号延伸到 3 公里 - IEEE Spectrum

> 来源：[https://spectrum.ieee.org/wi-fi-halow](https://spectrum.ieee.org/wi-fi-halow)

大多数人可能经历过 Wi-Fi 信号弱的挫败感。即使是覆盖一个相当小的房子的每个角落也可能是一个挑战。对于澳大利亚初创企业[Morse Micro](https://www.morsemicro.com/)开发的 Wi-Fi 技术来说，这不是问题，他们刚刚展示了具有 3 公里范围的 Wi-Fi 信号。

**Morse Micro 开发了一种名为[片上系统](https://spectrum.ieee.org/tag/system-on-chip) (SoC) 设计，使用了名为[Wi-Fi HaLow](https://www.wi-fi.org/discover-wi-fi/wi-fi-certified-halow)的无线协议，基于[IEEE 802.11ah 标准](https://ieeexplore.ieee.org/document/7920364)。该协议通过使用传播范围比传统 Wi-Fi 频率更远的低频无线电信号显著增强了范围。它还具有低功耗特性，并专为[物联网](https://spectrum.ieee.org/iot-for-arson-forensics) (IoT) 应用提供连接。**

**为了展示技术的潜力，Morse Micro 最近在旧金山海滨区的海洋海滩进行了一次[测试](https://www.youtube.com/watch?v=2xlUijXucoM)。他们展示了两台平板电脑通过 HaLow 网络能够在保持大约 1 兆位每秒的速度的情况下通信的距离达到 3 公里，足以支持略显粗糙的视频通话。**

**Wi-Fi HaLow 以 3 公里的距离打破了限制 Morse/Micro/[youtube](https://www.youtube.com/watch?v=2xlUijXucoM)**

**“这个范围相当前所未见”，Morse Micro 的市场营销和产品管理副总裁[普拉卡什·古达](https://www.linkedin.com/in/prakashguda/)说道。“不仅可以发送 ping，还可以发送实际的兆位数据。”**

**古达表示，HaLow 协议与传统 Wi-Fi 工作方式基本相同，唯一的区别是它在 900 兆赫频段工作，而不是 2.4 吉赫频段。低频信号能够传播更远，更能穿透实体物体，这是 HaLow 可以实现更长距离的秘密。**

**“这个范围相当前所未见。不仅可以发送 ping，还可以发送实际的兆位数据。” —— 普拉卡什·古达，Morse Micro**

**该协议还允许通道带宽最窄至 1 MHz，而标准 Wi-Fi 通常为 20 MHz。古达表示，这使得可以拥有更多不互相干扰的专用通道，非常适用于 IoT 应用中需要将多个设备连接到同一网络的情况。**

**古达说，这种折衷是较低的吞吐量，但正如他们的测试所示，HaLow仍然可以支持视频等数据密集型应用。这项技术也比传统Wi-Fi要低得多，因为它具有内置功能，允许设备长时间保持休眠状态，只有在需要传输数据时才唤醒。这使得用电池供电的HaLow设备的使用成为可能。古达表示，莫斯迈克罗正在与合作伙伴合作，建造需要长时间运行的电池供电物联网设备，在这些情况下，HaLow不会很快耗尽电池。**

**但物联网连接是一个竞争激烈的领域，意大利特里耶斯特国际理论物理中心（ICTP）的科学顾问[埃尔曼诺·皮埃特罗塞莫利](https://www.ictp.it/member/ermanno-pietrosemoli)表示。他专门从事无线网络领域。他说：“[HaLow]是一个有竞争力的对手，但在这个领域有很多参与者。”**

**与物联网专用技术如[ZigBee](https://spectrum.ieee.org/busy-as-a-zigbee)或[蓝牙](https://spectrum.ieee.org/apptricity-beams-bluetooth-signals-over-20-miles)相比，该技术具有相当范围的优势。而培德罗塞莫利表示，虽然[LoRa标准](https://spectrum.ieee.org/loras-bid-to-rule-the-internet-of-things)可以实现数十公里的距离，但其吞吐量过低，无法实现视频甚至实时语音。HaLow的高数据传输速率对于更多数据密集型应用可能会很有用，例如无线连接大量安全摄像头。**

**尽管HaLow标准于2017年获得认可，但皮埃特罗塞莫利表示，该技术的采用率非常低，这可能会让开发人员产生疑虑，他们担心这项技术是否会继续得到支持。相比之下，类似HaLow原理的竞争技术例如[SigFox](https://spectrum.ieee.org/stopping-a-wildfire-with-a-lowcost-sensor-network)和[窄带物联网](https://spectrum.ieee.org/italy-launches-a-new-wireless-network-for-the-internet-of-things)（NB-IoT）——一种低功率蜂窝技术——已经得到广泛部署。**

**甘特纳研究副总裁，分析师[比尔·雷](https://www.gartner.com/analyst/61747)则更加谨慎。他表示，虽然莫斯迈克罗公司已经开发出了一些令人印象深刻的技术，但HaLow在行业中并未取得重大进展。他补充说，高通公司早期曾是该技术的早期支持者，但目前对其失去兴趣，这是一个很有代表性的迹象。**

**“即使它的扩展范围可达3公里，我们也对这个标准并不乐观，”雷说。“蓝牙提供更大的互操作性，而LoRa提供更大的覆盖范围，将HaLow置于一个有限的应用场景。”**

**古达对此有不同的看法。他说，通过结合高数据传输速率和远距离，这项技术可以为整个物联网应用提供服务。他说：“这是两全其美。”**

**他承认采用率不高确实引起了关于互操作性和对单一供应商过度依赖的担忧，但他表示情况正在改变。总部位于加利福尼亚的[Newracom](https://newracom.com/)现在也生产HaLow SoC，并且在2021年，Wi-Fi联盟[推出了认证计划](https://www.wi-fi.org/news-events/newsroom/wi-fi-certified-halow-delivers-long-range-low-power-wi-fi)，该计划据Guda称将提高互操作性。**

**“如果你正在基于新技术制作一些设备，你希望确保有互操作性并且有多个供应商，”他说。“这是我们现在拥有的关键质量。”**

***本文出现在2024年4月的印刷版中，标题为“Wi-Fi HaLow信号延伸至3公里”。***

**来自您网站上的文章**

**网络相关文章**
