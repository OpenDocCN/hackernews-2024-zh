<!--yml

category: 未分类

date: 2024-05-27 13:39:58

-->

# PCIe 5.0 几乎已有四年历史了，在游戏 PC 中仍然几乎毫无价值 | PC Gamer

> 来源：[https://www.pcgamer.com/hardware/pcie-50-is-nearly-four-years-old-and-its-still-virtually-worthless-in-gaming-pcs/](https://www.pcgamer.com/hardware/pcie-50-is-nearly-four-years-old-and-its-still-virtually-worthless-in-gaming-pcs/)

Nick Evanson, 硬件作家

（图片来源：未来）

**本月我一直在测试：** 其实并没有测试太多！嗯，Asus 的一些电缆隐藏技术和 Gigabyte 的最新笔记本，但我主要是在测试新的显示面板色度计。看起来你看显示器时眼睛会误导你多少是令人惊讶的。

PCI Express 已经成为在计算机内部发送数据和指令的标准系统已有二十年之久。其规范经过定期更新，每次更新都提供前一版本性能的两倍。PCIe 5.0 几乎在四年前发布，是目前任何游戏 PC 中可以找到的最新版本。但由于支持它的硬件令人失望，它几乎没有价值。

要使计算机支持 PCIe 5.0，至少需要以下两者之一，最好是两者都具备。首先是 CPU —— AMD 和 Intel 最新一代处理器均内置 PCIe 5.0 控制器。[Ryzen 7000 系列](https://www.pcgamer.com/amd-zen-4-ryzen-7000-series-announcement/) 芯片拥有 28 条 Gen 5 PCIe，这些通道分配如下：前 16 条分配给图形卡插槽，8 条专用于 M.2 NVMe 插槽，剩余的 4 条用于与主板芯片组通信（稍后详述这一混乱情况）。

Intel 的 [第 14 代酷睿](https://www.pcgamer.com/intels-new-14th-gen-chips-are-almost-a-carbon-copy-of-its-13th-gen-fortunately-including-the-price/) 处理器与 AMD 的同样拥有相同数量的通道，但分配方式有所不同。其中只有十六条是 PCIe 5.0，其余是较慢的 4.0 规格；然而，8 条通道用于主板芯片组，留出空间让单个 SSD 直接连接到 CPU。但不完全是这样，因为这 16 条 Gen 5 通道可以切换为 8+8 模式，一半用于图形卡，一半用于 SSD（AMD 的芯片也可以这样做）。

然而，在理论上，AMD 在 PCIe 5.0 支持方面彻底压倒 Intel。但是，对于 [Ryzen](https://www.pcgamer.com/tag/ryzen/) 处理器来说，情况变得更加复杂，因为要访问完整的 Gen 5 通道范围，你需要一个使用 [最新芯片组的 E 版本](https://www.amd.com/en/products/processors/chipsets/am5.html#specs) 的主板。例如，X670 主板将拥有一个 PCIe 4.0 图形卡插槽，最多一个 PCIe 5.0 SSD 插槽。切换到 X670E 主板，GPU 插槽将变成 5.0，并且可能会有两个 Gen 5 M.2 插槽（尽管通常只有一个）。

AMD 的芯片提供了大量的 PCIe 5 轨道，但是否能全部利用又是另一回事。（图片来源：Future）

如果这有点令人困惑，那我可以向您保证，这比 Intel 目前的 PCIe 5.0 支持状态要好。以 [ASRock Z790 Nova](https://pg.asrock.com/mb/Intel/Z790%20Nova%20WiFi/index.asp) 主板为例。这是一块精美的主板，做工精良，并配备了六个 M.2 SSD 插槽。其中两个直接连接到 CPU，以获取最佳性能，最接近处理器的一个支持 PCIe 5.0 SSD。不幸的是，如果您将 *任何* SSD 插入该插槽，CPU 将自动将显卡插槽切换为八轨道（即 PCIe x8）。

尽管还剩下其他八条第五代轨道，它们不能进一步分割，并且由于 M.2 插槽总是使用四条轨道，因此通过使用第一个 SSD 插槽，你会浪费四条 PCIe 5.0 轨道。即使使用第四代甚至第三代 SSD 也是如此。并非所有 Intel 主板都会这样做，但是那些使用 Z790 芯片组并配备第五代 SSD 插槽的主板将有此限制。

但问题并不仅仅在于 Intel 的 SSD 插槽。单个 PCIe 5.0 轨道的带宽略低于 4 GB/s，因此八个轨道聚合后总带宽达到 31.5 GB/s（我们暂且称为 32 GB/s）。这与 16 轨道的第四代相同，因此理论上，将第五代显卡插入八轨道插槽不应该成为问题。

保持与由 PC Gamer 团队挑选的最重要的故事和最佳优惠的更新。

除非没有任何第五代显卡——它们至多还在使用 PCIe 4.0。来自 AMD、Intel 和 Nvidia 的下一轮新 GPU 可能会采用更新的规范，但考虑到它们刚刚在最新一代的显卡中切换到第四代，它们很可能不会这样做。

使用那个 'Gen 5' SSD 插槽，你的 GPU 插槽带宽将减半。（图片来源：ASRock/Intel Corporation）

因此，如果你像我一样拥有 ASRock Z790 主板，无论使用什么 GPU，如果在第一个 M.2 插槽中插入 SSD，该卡都将被迫以 PCIe 4.0 x8 模式运行（与第三代 x16 相同）。幸运的是，没有任何游戏能接近 PCI Express 显卡连接的带宽上限，但谁能说这不会很快发生呢？

如果 PCIe 5.0 SSD 表现出色并且值得购买，所有这些挫折都将是可以原谅的。事实是，它们是浪费钱的。当然，合成测试中的峰值性能使老款第四代驱动器看起来慢得多，但在实际使用中，你几乎不会注意到差别。

那可能看起来很奇怪，因为很少有 [值得考虑的 PCIe 5.0 SSD](https://www.pcgamer.com/hardware/ssds/teamgroup-z540-2tb-nvme-ssd-review/) 其顶峰顺序读/写速度远超过 10,000 MB/s —— 基本的 Gen 4 驱动器只有一半的速度。原因在于，游戏、应用程序甚至整个操作系统都是围绕着移动少量数据并且仅在需要时才移动设计的，因为大多数 PC 的存储速度并不快。Gen 5 SSD 只能在非常特定的场景中展示其真正的极限，而这些场景在游戏或一般 PC 使用中很少遇到。

然后是价格和发热问题雪上加霜。去看看像 Newegg 这样的网站，看看 [一款正宗的 Gen 5 SSD 的价格](https://click.linksynergy.com/deeplink?id=kXQk6%2AivFEQ&mid=44583&u1=pcg-us-4905622311240705561&murl=https%3A%2F%2Fwww.newegg.com%2Fp%2Fpl%3FN%3D100011693%2520601193224%2520601412649%2520601412650%2520600414920%2520601362178%26d%3Dpcie%2B5.0%2Bssd%26Order%3D1%26isdeptsrh%3D1)，即完全利用连接性能的产品。你看的是 1TB PCIe 5.0 SSD，价格在 $150 或更高，而用这些钱，你可以购买 [最好的 2TB Gen 4 驱动器之一](https://www.pcgamer.com/best-ssd-for-gaming/#section-the-best-gaming-ssd) 并且还有零钱。

> Gen 5 真的遇到了早期采用者的问题，还是厂商们根本不愿意全面支持它？

所有的 Gen 5 SSD 都比 Gen 4 的运行温度更高，而且不只是几度。你 *必须* 使用一个良好的冷却系统，无论是专用的散热器和风扇，还是主板上的大型金属散热器来吸收和散热。就前一种情况而言，这是要考虑的额外成本，而且根据您的显卡大小，您可能没有空间来安装它。

有人可能会说这只是“早期采用者”的问题，随着技术的进步，情况会好转。虽然这是正确的，但值得注意的是，PCIe 5.0 规范几乎是四年前发布的，AMD 的 Ryzen 7000 系列于 2022 年下半年上市。所以，Gen 5 真的遇到了早期采用者的问题，还是厂商们根本不愿意全面支持它？

老实说，这两者都有点。每个 PCIe Express 的后续修订版都会使每个通道的带宽加倍，直到版本 5.0，这是通过将总线时钟加快两倍来实现的。试着将您的 CPU 或 GPU 时钟加倍，看看会发生什么。好吧，这不是同一回事，但即使对于像 PCI Express 这样的相对简单的系统来说，大幅提高时钟速度也不是一件容易的事情。电气公差必须非常严格，这增加了所有东西的复杂性和成本。

对于 SSD 来说尤为如此。要完全符合 PCIe 5.0 标准，控制器芯片需要比 Gen 4 的运行速度更快，NAND 闪存芯片也需要以更高的速率读写数据。目前，能够做到这一切的公司非常少，而且不幸的是，它们生成大量热量。与 Gen 4 驱动相比，Gen 5 SSD 的市场要小得多，需求更少，因此相对成本也更高。

Gen 5 SSD — 快速、热、昂贵。嗯，不用了。 （图片来源：未来）

尽管如此，AMD 和 Intel 在实际实施 PCIe 5.0 方面并没有做得最好——在前者的情况下，Gen 5 显卡插槽被浪费了，因为没有人在销售 Gen 5 显卡，而在 [E 和非 E 主板之间的意识形态限制](https://www.pcgamer.com/amd-has-actually-created-only-one-chipset-for-zen-4-not-three/) 只增加了混乱。这完全是不必要的，因为在 B650 和 B650E 主板上使用的芯片物理上没有任何区别。

显然，Intel 对 Gen 5 PCIe 并不是很感兴趣，因为它在第 14 代桌面 CPU 和 700 系列芯片组中的实现只包括一个 PCIe 5.0 x16 插槽，最多一个 Gen 5 M.2 插槽，以及一堆令人恼火的配置限制。看起来 Intel 在 [Arrow Lake](https://www.pcgamer.com/intels-arrow-lake-specs-break-cover-say-hello-to-ddr5-6400-wave-goodbye-to-ddr4-and-possibly-hyper-threading/) 中也不会提供任何显著更好的东西。

PCIe 5.0 目前基本上是没什么用处的，或者至少说，用得不多，如果这样说有意义的话。在未来的短期内支持它的情况看起来也不会好到哪里去，但幸运的是，Gen 4 已经足够好了。虽然这项技术现在已经接近七年的历史，但我们仍然远未达到它成为游戏限制的点。在某个时候，每台台式电脑将全面采用 Gen 5，从顶部到底部，不过考虑到 PCI Express 的实施速度，我们在很长一段时间内不需要担心这个问题。

希望我们不会再次遇到与 Gen 7 或类似的问题！
