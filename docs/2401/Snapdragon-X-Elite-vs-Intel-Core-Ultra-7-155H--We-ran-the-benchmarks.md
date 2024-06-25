<!--yml

类别：未分类

日期：2024-05-27 14:53:46

-->

# Snapdragon X Elite vs Intel Core Ultra 7 155H：我们进行了基准测试

> 来源：[https://www.xda-developers.com/snapdragon-x-elite-vs-intel-core-ultra-7-155h/](https://www.xda-developers.com/snapdragon-x-elite-vs-intel-core-ultra-7-155h/)

### 主要观点

+   在基准测试中，高通的Snapdragon X Elite在单核和多核性能方面均优于英特尔的Core Ultra芯片。

+   Snapdragon X Elite在图形性能方面也超过了Core Ultra，在诸如Wild Life Extreme和Aztec Ruins之类的测试中，具有更高的FPS得分。

+   但是，就生产力任务而言，英特尔仍然具有优势，这可以从其在PCMark 10应用程序测试中的表现中看出。

[CES](https://www.xda-developers.com/ces-2024/)是一年中最重要的计算展会，长期以来，英特尔和AMD在那里宣布他们的最新产品，然后是每个OEM的设备发布。通常，这些新芯片和设备比它们的前身快一点，这些改进在五年的升级周期中累积起来。

今年，由于其在Core Ultra上的改进，英特尔有点抢了风头，其中包括神经处理单元（NPU）和大幅改进的GPU。这些芯片经过了优化，用于设备端的AI任务，虽然高通去年推出了Snapdragon X Elite，但英特尔有两个关键优势：它拥有大量的软件合作伙伴，并且正在发货。

为了不被落后，高通在CES上展示了Snapdragon X Elite仍然值得等待。该公司允许我携带一台Core Ultra笔记本电脑，并在Snapdragon X Elite参考设计旁边对其进行基准测试。当然，我已经对[华硕Swift Go 14](https://www.xda-developers.com/acer-swift-go-14-2024-review/)和[HP Spectre x360 14](https://www.xda-developers.com/hp-spectre-x360-14-2024-review/)进行了审查，而英特尔刚刚向我提供了一台华硕Zenbook 14，所以我已经有了Core Ultra芯片的经验。但是这两款芯片如何比较呢？

## 测试单元

这并不是我第一次对高通的Snapdragon X Elite参考设计进行基准测试。第一次是在十月份的Snapdragon峰会上，在那个时候，[我对它进行了测试](https://www.xda-developers.com/snapdragon-x-elite-benchmarks/) ，并将其与苹果的M2、英特尔的第13代CPU以及高通的上一代产品，Snapdragon 8cx Gen 3进行了比较。当时，高通实际上有两个型号，一个是23W的系统TDP，另一个是80W的系统TDP。然而，在CES上，它只带来了80W的型号，这实际上似乎并不公平。当然，我们也将在这里包含那些23W机器的分数。

总的来说，我在以下几台电脑上进行了这些测试：

+   Snapdragon X Elite B（23W）

+   Snapdragon X Elite A（80W）

+   HP Spectre x360 14（Core Ultra 7 155H）

+   华硕Zenbook 14 OLED（Core Ultra 7 155H）

+   MacBook Air（M2）

## Geekbench 6.2

[Geekbench](https://www.xda-developers.com/geekbench/)可能是最常见的CPU测试，所以你可能听过这个名字。你会发现，Snapdragon X Elite两台设备在单核和多核性能上都胜过Core Ultra。

有趣的是，当我第一次写关于Snapdragon X Elite基准测试时，我说高通处于一个很好的位置，因为它比英特尔第13代领先得多，Meteor Lake可能不会追赶上来，但事实上，英特尔Core Ultra在单线程性能上得分更低。英特尔也承认了这一点，因为它牺牲了单核性能以换取更好的多核性能。

## Cinebench 2024

Cinebench是我们将要运行的最新测试之一，旨在模拟实际的CPU使用情况。这又是一个CPU测试，尽管Cinebench 2024现在带有一个GPU测试。你需要专用图形才能运行它。

两台英特尔单核得分均为100，因此两台Snapdragon设备的得分比它们高出24%和32%。由于Snapdragon X Elite中的12个强大核心，高通的多核性能确实击败了竞争对手。当然，Core Ultra 7 155H的设计是不同的，它使用英特尔的混合技术，拥有六个大核心和八个小核心。单核性能与其余部分大致相当，但Snapdragon在多核方面表现卓越。

## Wild Life Extreme

[Wild Life Extreme](https://www.xda-developers.com/3dmark-wild-life-extreme/)是3DMark测试套件的一部分，但它是少数真正跨平台的测试之一，意味着它支持x86 Windows、Arm64 Windows和macOS。大多数3DMark测试仅支持x86。这也是列表中第一个考虑到图形性能的测试。

再次，Snapdragon X Elite的23W和80W单位都胜出。有趣的是，华硕Zenbook 14和惠普Spectre x360的得分分别为33.28和34.12FPS，比80W Snapdragon X Elite单位低了10FPS以上，比23W型号低了5FPS。

## Aztec Ruins（1080p）

Aztec Ruins是GFXBench的一部分，正如你从名称中可以猜到的那样，它是关于测试图形性能的。设置使用的是Vulkan，除了MacBook Air，它使用的是Metal。

再次，Snapdragon X Elite轻松击败了英特尔Core Ultra，这次是超过100FPS。80W单位击败了超过160FPS。

## PCMark 10应用程序

最后一个测试是PCMark 10应用程序，这是PCMark 10中唯一运行在Arm上的部分，因为它依赖于Office和Edge。它不运行在macOS上。

这是一项Qualcomm硬件落后的测试。事实上，英特尔擅长为生产力制造芯片。

## 结论

正如我在开头所说的，英特尔目前的最大优势是它今天就能发货，并且一切都在它上面运行。高通在Snapdragon X Elite中的新自定义Oryon硅在几乎每个基准测试中都轻松击败了英特尔Core Ultra，而Apple当前MacBook Air中的M2则位于中间位置。

高通希望你等待骁龙X Elite笔记本电脑，预计将于今年六月左右到达。无论你在等待哪款芯片，我们都知道今年将是计算机的大年，而2024年无疑是一个良好的开端。
