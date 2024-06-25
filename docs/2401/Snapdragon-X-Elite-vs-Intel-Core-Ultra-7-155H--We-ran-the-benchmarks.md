<!--yml

类别：未分类

日期：2024-05-27 14:53:46

-->

# Snapdragon X Elite vs Intel Core Ultra 7 155H：我们进行了基准测试

> 来源：[`www.xda-developers.com/snapdragon-x-elite-vs-intel-core-ultra-7-155h/`](https://www.xda-developers.com/snapdragon-x-elite-vs-intel-core-ultra-7-155h/)

### 主要观点

+   在基准测试中，高通的 Snapdragon X Elite 在单核和多核性能方面均优于英特尔的 Core Ultra 芯片。

+   Snapdragon X Elite 在图形性能方面也超过了 Core Ultra，在诸如 Wild Life Extreme 和 Aztec Ruins 之类的测试中，具有更高的 FPS 得分。

+   但是，就生产力任务而言，英特尔仍然具有优势，这可以从其在 PCMark 10 应用程序测试中的表现中看出。

[CES](https://www.xda-developers.com/ces-2024/)是一年中最重要的计算展会，长期以来，英特尔和 AMD 在那里宣布他们的最新产品，然后是每个 OEM 的设备发布。通常，这些新芯片和设备比它们的前身快一点，这些改进在五年的升级周期中累积起来。

今年，由于其在 Core Ultra 上的改进，英特尔有点抢了风头，其中包括神经处理单元（NPU）和大幅改进的 GPU。这些芯片经过了优化，用于设备端的 AI 任务，虽然高通去年推出了 Snapdragon X Elite，但英特尔有两个关键优势：它拥有大量的软件合作伙伴，并且正在发货。

为了不被落后，高通在 CES 上展示了 Snapdragon X Elite 仍然值得等待。该公司允许我携带一台 Core Ultra 笔记本电脑，并在 Snapdragon X Elite 参考设计旁边对其进行基准测试。当然，我已经对[华硕 Swift Go 14](https://www.xda-developers.com/acer-swift-go-14-2024-review/)和[HP Spectre x360 14](https://www.xda-developers.com/hp-spectre-x360-14-2024-review/)进行了审查，而英特尔刚刚向我提供了一台华硕 Zenbook 14，所以我已经有了 Core Ultra 芯片的经验。但是这两款芯片如何比较呢？

## 测试单元

这并不是我第一次对高通的 Snapdragon X Elite 参考设计进行基准测试。第一次是在十月份的 Snapdragon 峰会上，在那个时候，[我对它进行了测试](https://www.xda-developers.com/snapdragon-x-elite-benchmarks/) ，并将其与苹果的 M2、英特尔的第 13 代 CPU 以及高通的上一代产品，Snapdragon 8cx Gen 3 进行了比较。当时，高通实际上有两个型号，一个是 23W 的系统 TDP，另一个是 80W 的系统 TDP。然而，在 CES 上，它只带来了 80W 的型号，这实际上似乎并不公平。当然，我们也将在这里包含那些 23W 机器的分数。

总的来说，我在以下几台电脑上进行了这些测试：

+   Snapdragon X Elite B（23W）

+   Snapdragon X Elite A（80W）

+   HP Spectre x360 14（Core Ultra 7 155H）

+   华硕 Zenbook 14 OLED（Core Ultra 7 155H）

+   MacBook Air（M2）

## Geekbench 6.2

[Geekbench](https://www.xda-developers.com/geekbench/)可能是最常见的 CPU 测试，所以你可能听过这个名字。你会发现，Snapdragon X Elite 两台设备在单核和多核性能上都胜过 Core Ultra。

有趣的是，当我第一次写关于 Snapdragon X Elite 基准测试时，我说高通处于一个很好的位置，因为它比英特尔第 13 代领先得多，Meteor Lake 可能不会追赶上来，但事实上，英特尔 Core Ultra 在单线程性能上得分更低。英特尔也承认了这一点，因为它牺牲了单核性能以换取更好的多核性能。

## Cinebench 2024

Cinebench 是我们将要运行的最新测试之一，旨在模拟实际的 CPU 使用情况。这又是一个 CPU 测试，尽管 Cinebench 2024 现在带有一个 GPU 测试。你需要专用图形才能运行它。

两台英特尔单核得分均为 100，因此两台 Snapdragon 设备的得分比它们高出 24%和 32%。由于 Snapdragon X Elite 中的 12 个强大核心，高通的多核性能确实击败了竞争对手。当然，Core Ultra 7 155H 的设计是不同的，它使用英特尔的混合技术，拥有六个大核心和八个小核心。单核性能与其余部分大致相当，但 Snapdragon 在多核方面表现卓越。

## Wild Life Extreme

[Wild Life Extreme](https://www.xda-developers.com/3dmark-wild-life-extreme/)是 3DMark 测试套件的一部分，但它是少数真正跨平台的测试之一，意味着它支持 x86 Windows、Arm64 Windows 和 macOS。大多数 3DMark 测试仅支持 x86。这也是列表中第一个考虑到图形性能的测试。

再次，Snapdragon X Elite 的 23W 和 80W 单位都胜出。有趣的是，华硕 Zenbook 14 和惠普 Spectre x360 的得分分别为 33.28 和 34.12FPS，比 80W Snapdragon X Elite 单位低了 10FPS 以上，比 23W 型号低了 5FPS。

## Aztec Ruins（1080p）

Aztec Ruins 是 GFXBench 的一部分，正如你从名称中可以猜到的那样，它是关于测试图形性能的。设置使用的是 Vulkan，除了 MacBook Air，它使用的是 Metal。

再次，Snapdragon X Elite 轻松击败了英特尔 Core Ultra，这次是超过 100FPS。80W 单位击败了超过 160FPS。

## PCMark 10 应用程序

最后一个测试是 PCMark 10 应用程序，这是 PCMark 10 中唯一运行在 Arm 上的部分，因为它依赖于 Office 和 Edge。它不运行在 macOS 上。

这是一项 Qualcomm 硬件落后的测试。事实上，英特尔擅长为生产力制造芯片。

## 结论

正如我在开头所说的，英特尔目前的最大优势是它今天就能发货，并且一切都在它上面运行。高通在 Snapdragon X Elite 中的新自定义 Oryon 硅在几乎每个基准测试中都轻松击败了英特尔 Core Ultra，而 Apple 当前 MacBook Air 中的 M2 则位于中间位置。

高通希望你等待骁龙 X Elite 笔记本电脑，预计将于今年六月左右到达。无论你在等待哪款芯片，我们都知道今年将是计算机的大年，而 2024 年无疑是一个良好的开端。
