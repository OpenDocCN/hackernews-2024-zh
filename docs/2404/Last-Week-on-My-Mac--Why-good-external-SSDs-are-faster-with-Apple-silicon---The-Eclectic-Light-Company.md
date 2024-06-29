<!--yml

类别：未分类

日期：2024-05-27 13:13:42

-->

# 上周在我的 Mac 上：为什么好的外部 SSD 在使用苹果硅时更快 – The Eclectic Light Company

> 来源：[https://eclecticlight.co/2024/04/14/last-week-on-my-mac-why-good-external-ssds-are-faster-with-apple-silicon/](https://eclecticlight.co/2024/04/14/last-week-on-my-mac-why-good-external-ssds-are-faster-with-apple-silicon/)

两个两个字母的词可以涵盖多种性能问题。*最高* 几乎无一例外地设定了一个永远无法实现的目标，而一个规范的 *最高* 可能与另一个规范的完全不同。我在想雷电和USB4，它们都声称 *最高* 40 Gb/s，但却以不同的方式失败，并且在不同类型的 Mac 上表现也不同。要理解这一点，我们需要仔细阅读详细说明。

#### 不同的 Mac

查看苹果公司原始 [Mac mini M1](https://support.apple.com/111894) 规格中的细则。它具有两个支持以下内容的“雷电 / USB 4 端口”：

+   雷电 3（最高 40 Gb/s）

+   USB 4（最高 40 Gb/s）

+   USB 3.1 Gen 2（最高 10 Gb/s）。

将其与最后一批英特尔 Mac 之一进行比较，例如 [MacBook Pro（16 英寸，2019）](https://support.apple.com/111932)，其支持以下内容：

+   雷电（最高 40 Gb/s）

+   USB 3.1 Gen 2（最高 10 Gb/s）

虽然没有提到 USB4，但仍声称相同的 *最高* 40 Gb/s。

#### 不同的 *最高速度*

根据英特尔的说法，[雷电 3](https://www.thunderbolttechnology.net/sites/default/files/18-241_Thunder7000Controller_Brief_FIN_HI.pdf)和 4 提供了多达 4 条 PCI Express 3.0 总计 32.4 Gb/s 的通用数据传输通道，以及 4 条 DisplayPort 1.4 HBR3 用于视频。因此，当使用外部 SSD 传输数据时，我们可以从英特尔 Mac 的雷电端口中期望的最快速度是 32.4 Gb/s。根据长期且有时痛苦的经验，这意味着读写速度最高可达 3 GB/s，通常远低于此速度。所以雷电的 *最高* 40 Gb/s 说法并不那么靠谱。

相反，[USB4](https://en.wikipedia.org/wiki/USB4) 可支持技术上称为 USB4 Gen 3×2，以 USB 40Gbps 销售，名义传输速率为 40 Gb/s 或最高 4.8 GB/s。由于协议开销，实际上这意味着适当快速的 NVMe SSD 应该能够达到超过 3 GB/s 的读写速度，甚至可能接近 4 GB/s。这就是 USB4 的 *最高* 40 Gb/s。

因此，您在使用英特尔 Mac 的雷电端口时最快可以看到的速度是最高 3 GB/s，而适用于任何苹果硅 Mac 的 USB4 外壳如果确实支持 USB 40Gbps，则应轻松超过这一速度。

#### 下降至 10 Gb/s

预制的外置 SSD 和硬盘盒依赖于不同的芯片提供雷电和 USB4 的支持，尽管很少有供应商费心明确说明这一点。更昂贵的认证雷电模型应该使用英特尔芯片，但其中一些不能达到 40 Gb/s，只能达到 20 Gb/s。更近期的硬件可能选择两个芯片，一个支持雷电，另一个为那些被困在某种 USB 3 变体的计算机提供安慰奖。

最新的可能会将 USB4 定位在 40 Gb/s，当不可用时回退到 USB 3.1 Gen 2。因此，这些在 Apple 芯片的 Mac 上支持 USB4 的设备速度可达 3 GB/s 以上，但在连接到 Intel Mac 上的端口时，只有 USB 3 的 1 GB/s。在描述其产品时，更好的供应商会清楚地表明这一点，但当你购买一个之前没见过的带有临时名称的品牌时，可能直到连接 SSD 到你的 Mac 才意识到这一点。

#### 集线器？

但是，如果你通过“雷电 4”集线器连接了一个 USB 40Gbps 的硬盘盒会发生什么呢？这将取决于集线器可以以哪些模式运行，既可以对硬盘盒，也可以回到 Mac，尽管到目前为止，我见过的集线器没有一个能清楚地说明这一点。

在实践中，通过我的 Satechi 雷电 4 薄型集线器，我可以得到 3.0 GB/s 的写入速度和 3.2 GB/s 的读取速度，与一个 USB4 硬盘盒相比，这仍然超过了直接连接的没有 USB4 支持的雷电 3 SSD 的速度。关于其他最近的“雷电 4”集线器，如 CalDigit Element 和 Plugable Thunderbolt 4，是否也是如此，我不清楚。

#### 完毕

经过几天测试，我已经改变了对最佳外置 SSD 的推荐，选择了来自 OWC 的最新 Express 1M2 硬盘盒。此前，我选择了相对可靠的雷电 3 *高达* 3 GB/s，尽管很少有驱动器似乎能达到 *高达* 这一速度。如果你仍然需要与 Intel Mac 的良好性能，这是有道理的。

但是，如果你需要与 Apple 芯片的 Mac 获得最佳性能，你最好选择一个高质量的 USB 40Gbps 硬盘盒，比如 OWC 的 Express 1M2，即使通过兼容的集线器也应可靠地返回超过 3 GB/s。我更喜欢“超过”这个词而不是“高达”。
