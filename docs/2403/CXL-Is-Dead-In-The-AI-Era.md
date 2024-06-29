<!--yml

类别：未分类

日期：2024-05-27 15:00:09

-->

# CXL在AI时代已经名存实亡

> 来源：[https://www.semianalysis.com/p/cxl-is-dead-in-the-ai-era](https://www.semianalysis.com/p/cxl-is-dead-in-the-ai-era)

如果我们回到两年前，在AI的快速崛起之前，数据中心硬件世界大多数是在追逐CXL。它被承诺为带来异构计算、内存池和可组合服务器架构的救世主。现有的参与者和一大批新创企业都在赶着将CXL集成到他们的产品中，或者创建基于CXL的新产品，例如内存扩展器、池化器和交换机。快进到2023年和2024年初，许多项目已经悄然搁置，许多超大规模和大型半导体公司几乎完全转向其他方向。

随着Astera Labs的即将上市和产品发布，CXL讨论至少在短期内重新引起关注。我们已经对技术进行了[广泛的介绍](https://www.semianalysis.com/p/cxl-deep-dive-future-of-composable)，它对[云服务提供商的成本节约潜力](https://www.semianalysis.com/p/cxl-enables-microsoft-azure-to-cut)，以及[生态系统和硬件堆栈](https://www.semianalysis.com/p/cxl-deep-dive-future-of-composable)。尽管在纸面上非常有前景，数据中心的格局已经发生了很大变化，但有一件事情仍然没有改变：像控制器和交换机这样的CXL硬件仍然没有大量发货。尽管如此，围绕CXL仍然存在大量噪音和研究，某些行业专业人士现在正推动CXL作为AI的“推动者”的叙述。

更广泛的CXL市场是否已准备好起飞并兑现其承诺？CXL能否成为AI应用的互连？在CPU附加扩展和池化中的作用是什么？我们将在本报告的订阅者部分回答这些问题。

简单来说，推动CXL用于AI的人完全错了。让我们先快速回顾一下CXL的主要用例和承诺。

CXL是建立在PCIe物理层之上的协议，实现设备间的缓存和内存一致性。利用广泛的PCIe接口可用性，CXL允许在各种硬件上共享内存：CPU、NIC和DPU、GPU及其他加速器、SSD和内存设备。

这样可以实现以下用例：

+   内存扩展：CXL可以帮助增加服务器的内存带宽和容量。

+   内存池：CXL可以创建内存池，将内存从CPU中分离，理论上可以大幅提高DRAM的利用率。从理论上讲，这可以为每个云服务提供商节省数十亿美元。

+   异构计算：ASIC比通用CPU效率高得多。CXL可以通过提供低延迟高速缓存一致性互连，将ASIC与通用计算整合，使应用更容易集成到现有代码库中。

+   可组合的服务器架构：服务器被分解成各种组件，并放置在可以动态分配工作负载的组中，从而改善资源的利用率，同时更好地匹配应用程序的需求，减少资源的浪费。

下图讲述了故事的一部分：CXL可以解决主系统内存和存储之间的延迟和带宽差距，从而实现一个新的内存层。

一些人现在预测到2028年，CXL的销售额将达到高达150亿美元，而目前仅为几百万美元，因此我们认为是时候对CXL市场进行适当的更新了，因为这是一个完全荒谬的说法。让我们首先讨论人工智能领域的CXL案例。

目前，CXL的可用性是主要问题，因为Nvidia的GPU不支持它，而AMD的技术仅限于MI300A。虽然MI300X理论上可以在硬件上支持CXL，但并未正确暴露。CXL IP的可用性将在未来得到改善，但存在使CXL在加速计算时代变得无关紧要的更深层次问题。

两个主要问题与PCIe SerDes、海滨或海岸线区域有关。芯片的IO通常必须来自芯片的边缘。下面的图片来自Nvidia，以卡通化的方式展示了H100。中心有所有的计算能力。顶部和底部完全专用于HBM。随着我们从H100过渡到B100，HBM的数量增加到8个，这需要更多的海岸线面积。Nvidia继续用其2片芯片包装的两侧完全用于HBM。

剩余的两侧专用于其他芯片到芯片的IO，并且这是标准和专有互连在芯片面积上竞争的地方。H100 GPU具有3种IO格式：PCIe、NVlink和C2C（用于连接到Grace）。Nvidia决定仅包括最少的16条PCIe通道，因为Nvidia更偏爱后者NVLink和C2C。请注意，诸如AMD的Genoa等服务器CPU可达到128条PCIe通道。

选择这一方案的主要原因是带宽。16通道的PCIe接口在每个方向上有64GB/s的带宽。Nvidia的NVlink为其他GPU带来了每个方向450 GB/s的带宽，大约高出7倍。Nvidia的C2C与Grace CPU也带来了每个方向450GB/s的带宽。公平地说，Nvidia为NVLink分配了更多的海滨区域，所以我们需要在这个方程式中包括硅面积；但即使如此，我们估计，在各种各样的SOC中，以太网风格的SerDes，如Nvidia NVLink，Google ICI等每平方毫米的带宽都是海岸线面积的3倍。

因此，如果您是处于带宽受限世界中的芯片设计师，选择使用PCIe 5.0而不是112G Ethernet风格的SerDes会使您的芯片大约差3倍。这种差距将随着下一代GPU和AI加速器采用224G SerDes而保持，维持与PCIe 6.0 / CXL 3.0的这种3倍差距。我们正处于一个受限于引脚的世界，放弃IO效率是一个不理智的权衡。

AI集群的主要扩展和扩展互连协议将是专有协议，如Nvidia NVlink和Google ICI，或者Ethernet和Infiniband。即使在扩展格式中也是如此，这是由于PCIe SerDes在固有的限制下。由于延迟目标的分歧，PCIe和Ethernet SerDes具有极其不同的比特错误率（BER）要求。

PCIe 6要求BER < 1e-12，而Ethernet要求1e-4。这种8个数量级的巨大差异是由于PCIe的严格延迟要求，需要一种极轻的前向纠错（FEC）方案。FEC在发送端数字添加冗余的奇偶校验位/信息，接收端用于检测和纠正错误（位翻转），类似于内存系统中的ECC。更重的FEC会增加更多的开销，在本应用于数据位的空间上占用。更重要的是，FEC会在接收端增加大量延迟。这就是为什么直到第六代PCIe，PCIe才避免使用任何FEC。

Ethernet风格的SerDes受到严格的PCIe规范限制较少，因此它可以更快速、具有更高的带宽。因此，NVlink具有更高的延迟，但在大规模并行工作负载的AI世界中，约100纳秒与约30纳秒并不是一个值得考虑的问题。

> 首先，MI300 AID主要利用其海滨区域进行PCIe SerDes而非Ethernet风格的SerDes。虽然这使得AMD在IFIS、CXL和PCIe连接方面具有更多的配置灵活性，但结果是总IO大约只有Ethernet风格SerDes的三分之一。如果AMD希望与Nvidia的B100竞争，他们需要立即摆脱PCIe风格的SerDes。我们相信他们将会在MI400上实现这一点。
> 
> [Nvidia的计划以碾压竞争为目标 – B100，“X100”，H200，224G SerDes，OCS，CPO，PCIe 7.0，HBM3E](https://www.semianalysis.com/p/nvidias-plans-to-crush-competition)

AMD缺乏高质量的SerDes严重限制了他们长期产品的竞争力。他们提出了Open xGMI / Open Infinity Fabric / Accelerated Fabric Link，因为CXL不是AI的正确协议。虽然它主要基于PCIe，但出于时间市场、性能、一致性和覆盖范围等原因，它放弃了PCIe 7.0和CXL的一些标准功能。

CXL 内存带宽扩展对 AI 有何影响？定制 AI 超级计算机芯片采用情况如何？其他供应商如 Marvell Google 推出的定制硅芯片又如何？我们将回答这些问题，并审视更经典的内存池化和内存扩展用例。

[团体订阅享 20% 折扣](https://www.semianalysis.com/subscribe?group=true&coupon=fe141654)
