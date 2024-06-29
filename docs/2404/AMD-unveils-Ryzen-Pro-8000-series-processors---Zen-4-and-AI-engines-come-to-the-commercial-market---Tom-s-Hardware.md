<!--yml

category: 未分类

date: 2024-05-27 13:15:05

-->

# AMD推出Ryzen Pro 8000系列处理器 — Zen 4和AI引擎进入商业市场 | Tom's Hardware

> 来源：[https://www.tomshardware.com/pc-components/cpus/amd-unveils-ryzen-pro-8040-hawk-point-and-8000-series-phoenix-processors-ai-engines-come-to-the-commercial-market](https://www.tomshardware.com/pc-components/cpus/amd-unveils-ryzen-pro-8040-hawk-point-and-8000-series-phoenix-processors-ai-engines-come-to-the-commercial-market)

今天，AMD宣布其Ryzen Pro系列产品，将['Hawk Point' 8040系列](https://www.tomshardware.com/pc-components/cpus/the-refresh-that-wasnt-amd-announces-hawk-point-ryzen-8040-series-with-zen-4-rdna3-and-xdna-teases-strix-point)扩展到商用笔记本电脑和工作站用户，并同时推出其[Ryzen 8000 'Phoenix' APU型号](https://www.tomshardware.com/pc-components/cpus/amd-launches-ryzen-8000g-phoenix-apus-brings-ai-to-the-desktop-pc-reveals-zen-4c-clocks-for-the-first-time)用于商用台式电脑。与过去看到的情况一样，Pro系列基于AMD现有的面向消费者的处理器型号，但配备了额外的功能，专为商业市场量身定制。通过这些处理器的消费者版本，AMD成为第一个在移动和台式PC市场引入其AI处理神经处理单元（NPU）的x86公司。现在，这些同样的AI加速功能将面向商业用户，使AMD声称是首家为笔记本电脑和工作站配备NPU的专业CPU厂商。

AMD内置的XNDA引擎驱动NPU，并且公司宣称其移动处理器在16 TOPS的NPU性能方面领先于英特尔的竞争对手Core Ultra处理器，超越了Meteor Lake NPU的11 TOPS。AMD在总体系统TOPS方面也保持着微弱的领先优势，包括CPU和GPU的AI处理能力，以39 TOPS领先于英特尔的34 TOPS。AMD的桌面Ryzen 8000 APU也配备了内置的NPU引擎，提供16 TOPS的性能，而英特尔尚未推出集成AI引擎的桌面PC处理器。

值得注意的是，无论是 AMD 还是 Intel 的芯片都未满足[Microsoft](https://www.tomshardware.com/tag/microsoft)下一代[AI PC](https://www.tomshardware.com/tag/ai-pc)要求，即从 NPU 获得 45 TOPS 的性能，尽管两家公司都表示其下一代芯片，分别是 [Strix Point 和 Lunar Lake](https://www.tomshardware.com/pc-components/cpus/intel-says-lunar-lake-will-have-100-tops-of-ai-performance-45-tops-from-the-npu-alone-meeting-requirement-for-next-gen-ai-pcs)，将达到该标准。45 TOPS 的要求旨在使 Microsoft 能够在本地运行 Copilot 的 AI 元素，目前尚不清楚未来的 Windows 更新如何与当前世代的 AMD 和 Intel 处理器配合。[高通的骁龙 X Elite](https://www.tomshardware.com/laptops/i-went-hands-on-with-two-different-qualcomm-snapdragon-x-elite-chips-as-the-company-claims-it-will-beat-intels-core-ultra) Arm 芯片将于年中推出，其 NPU 从中获得 45 TOPS 的性能。苹果的 M3 处理器提供 18 TOPS 的 NPU 性能，但显然不受 Microsoft 要求的影响。

在商业领域中，本地运行 AI 工作负载的能力有助于缓解关键的隐私问题。它还为 AI 应用程序提供了延迟、性能和电池寿命优势（使用 NPU 的功耗效率比在 GPU 上运行相同的 AI 工作负载显著提高）。然而，释放这种性能需要紧密耦合的软件解决方案，并且在很大程度上，AI 主导权的早期战争将取决于开发者的合作伙伴关系。

AI 生态系统仍处于起步阶段，开发人员正努力利用 NPU 加速其产品，但 AMD、Microsoft 和其他公司正在构建基础，以支持软件和 Windows 中广泛应用的 AI 加速功能。AMD 引用称，到 2024 年将拥有 150 多个 ISV（独立软件供应商）合作伙伴，开发基于 AI 的解决方案，而[英特尔正在扩展的 AI 开发者计划](https://www.tomshardware.com/pc-components/cpus/intel-shares-new-ai-pc-definition-launches-ai-pc-acceleration-programs-and-core-ultra-meteor-lake-nuc-developer-kits-at-ai-conference)则已有 100 个 ISV 和 100 多个 IHV（集成硬件供应商）合作伙伴。

当然，释放本地运行 AI 的能力始于硬件。让我们更仔细地看看硅片。

## AMD Ryzen Pro 8040 系列处理器

如前所述，Ryzen Pro 8040系列处理器基于面向消费者的“Hawk Point”处理器 — 您可以在[这里了解](https://www.tomshardware.com/pc-components/cpus/the-refresh-that-wasnt-amd-announces-hawk-point-ryzen-8040-series-with-zen-4-rdna3-and-xdna-teases-strix-point)Zen 4驱动的细节。U系列处理器适用于15-28W TDP范围，而HS变体则分别适用于20-28W和35-54W TDP范围，涵盖了Ryzen 9、7和5产品线，45W型号主要面向工作站。

核心数从六核12线程的Ryzen 5到八核16线程的Ryzen 7和9不等。峰值时钟频率与消费型号相同。值得注意的是，Ryzen 5 Pro 8540U没有集成NPU，就像消费型号一样。所有其他关键标准，如时钟速度、iGPU和缓存，均保持不变。

获取Tom's Hardware最佳新闻和深度评测，直接发送到您的收件箱。

AMD分享了与Intel竞争芯片的[基准测试](https://www.tomshardware.com/tag/benchmark)，但像所有供应商提供的基准测试一样，我们应该持保留态度。AMD声称在15W功耗下，Ryzen 7 Pro 8840U在多任务处理、渲染、生产力、图形和内容创作等工作负载中比15W Intel Core Ultra 7 165U平均提升30%。该公司还宣称在28W Core Ultra 7 165H上平均提升18%。

AMD还表示，其45W Ryzen 9 Pro 8945HS工作站处理器在Topaz Labs AI视频应用中比45W Intel Core Ultra 9 185H具有50%以上的优势，并且在LLama 2 LLM的首个令牌上比Intel更快77%。AMD还与更广泛的AI模型分享了基准测试，如面部和物体识别等。声称的性能优势在各类更一般的CPU中心应用中范围从5%到23%不等。

该公司还声称在Teams工作流程中，与Intel Core Ultra 7 165H相比，电池寿命优势达到46%，并且在与Apple M3 Pro CPU的比较中略有优势。然而，AMD在所有直接性能比较中都没有提到Apple处理器。

## AMD Ryzen Pro 8000系列APU

桌面PC用的AMD Ryzen Pro 8000系列建立在强大的消费市场8000G APU系列的基础上，您可以在[这里](https://www.tomshardware.com/pc-components/cpus/amd-launches-ryzen-8000g-phoenix-apus-brings-ai-to-the-desktop-pc-reveals-zen-4c-clocks-for-the-first-time)和[这里](https://www.tomshardware.com/pc-components/cpus/amd-ryzen-7-8700g-cpu-review)详细了解。所有这些型号都配备了内置的NPU AI加速引擎。

所有相关规格与消费者型号保持完全一致，选择范围从四核八线程到八核十六线程，涵盖了 Ryzen Pro 3、5 和 7 系列。标准的 8000G 型号具有可配置的 45-65W TDP 范围，而 GE 型号则提供更低功耗的 35W TDP。

总体而言，AMD 声称相比英特尔竞争的 Core i7-14700，平均提升了 19%，在 Time Spy 图形基准测试中的峰值达到了 3 倍的优势。考虑到 Zen 4 CPU 和 RDNA 3 图形引擎强大的组合，这并不奇怪。AMD 还宣称，在各种配置下，功耗降低了从 33% 到 76% 不等。

## AMD Ryzen Pro 技术

集成的 AMD Pro 技术套件是 AMD Pro 型号与其标准消费级芯片之间的关键区别。该硬件和软件套件包括诸如 AMD Pro [安全性](https://www.tomshardware.com/tag/security) 的功能，其中包括利用专有 OEM 解决方案和内置的 [Windows 11](https://www.tomshardware.com/tag/windows-11) 功能、AMD memory guard、[Microsoft](https://www.tomshardware.com/tag/microsoft) Pluton 和 AMD 安全处理器的多层 [安全性](https://www.tomshardware.com/tag/security)。8000 系列家族标志着首批集成 Microsoft Pluton 安全功能和基于云的远程管理能力的台式机。

AMD Pro 可管理性功能简化了配置、系统镜像和部署任务，而 AMD Pro 商业就绪套件确保了稳定性，并包含质量和可靠性保证。AMD 指出，它在所有处理器型号上提供完整的 Pro 技术套件，而英特尔为了分割市场支持则有所不同。

AMD 的 OEM 平台推出了一系列基于 8040 和 8000 系列处理器的刷新产品。Ryzen Pro 系列现已提供给 AMD 的 OEM 合作伙伴。
