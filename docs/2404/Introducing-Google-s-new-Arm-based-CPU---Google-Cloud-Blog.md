<!--yml

category: 未分类

date: 2024-05-27 13:04:06

-->

# 介绍 Google 的新 Arm 架构 CPU | Google Cloud 博客

> 来源：[https://cloud.google.com/blog/products/compute/introducing-googles-new-arm-based-cpu/](https://cloud.google.com/blog/products/compute/introducing-googles-new-arm-based-cpu/)

Axion 是谷歌自定义硅的最新产品。自 2015 年以来，我们已发布了五代 Tensor 处理单元（TPU）；2018 年，我们发布了第一代视频编码单元（VCU），实现了视频转码效率高达 [33x](https://dl.acm.org/doi/abs/10.1145/3445814.3446723)；2021 年，我们通过投资于“系统芯片”（SoC）设计加倍强化了自定义计算，并发布了移动设备的三代 Tensor 芯片的 [首款](https://blog.google/products/pixel/introducing-google-tensor/)。

尽管我们在计算加速器上的投资已经改变了客户的能力，但通用计算仍然是客户工作负载中的关键部分，也将保持如此。分析、信息检索以及机器学习的训练和服务都需要大量的计算能力。希望最大化性能、降低基础设施成本并达到可持续发展目标的客户和用户已经发现，CPU 改进的速度最近有所放缓。阿姆达尔定律表明，随着加速器的持续改进，通用计算将主导成本并限制我们基础设施的能力，除非我们进行相应的投资以跟上发展。

Axion 处理器将 Google 的硅技术与 Arm 最高性能的 CPU 核心结合，提供的实例性能比当今云中最快的通用 Arm 实例高出多达 30%，比当前一代 x86 实例高出多达 50% 的性能，并且能效比更高达 60%¹。这就是为什么我们已经开始在当前一代基于 Arm 的服务器上部署 Google 服务，如 BigTable、Spanner、BigQuery、Blobstore、Pub/Sub、Google Earth Engine 和 YouTube Ads 平台，并计划很快在 Axion 上部署和扩展这些服务以及更多服务。

### **无与伦比的性能和效率，以 Titanium 为基础**

使用 Arm Neoverse™ V2 CPU 构建的 Axion 处理器，在通用工作负载如 Web 和应用服务器、容器化微服务、开源数据库、内存缓存、数据分析引擎、媒体处理、基于 CPU 的 AI 训练和推理等方面实现了巨大的性能飞跃。

Axion is underpinned by [钛](https://cloud.google.com/titanium?e=48754805)，一个专为定制硅微控制器和分层扩展优化的系统。钛扩展负责平台操作，如网络和安全性，因此Axion处理器具有更多的容量和改进的性能，适用于客户的工作负载。钛还将存储I/O处理转移到[超级硬盘](https://cloud.google.com/compute/docs/disks/hyperdisks)，我们的新块存储服务，可以动态实时分配性能而不是实例大小。

*“Google’s announcement of the new Axion CPU marks a significant milestone in delivering custom silicon that is optimized for Google’s infrastructure, and built on our high-performance Arm Neoverse V2 platform. Decades of ecosystem investment, combined with Google’s ongoing innovation and open-source software contributions ensure the best experience for the workloads that matter most to customers running on Arm everywhere."* - Rene Haas, CEO, Arm

除了性能，客户还希望更高效地运行并实现可持续发展目标。Google Cloud数据中心的效率已经比行业平均水平高出1.5倍，并且与五年前相比，提供了3倍的计算能力，但电力消耗相同²。我们已经设定了雄心勃勃的[目标](https://sustainability.google/operating-sustainably/)，即在碳零排放的条件下全天候运行我们的办公室、校园和数据中心，并提供[工具](https://cloud.google.com/carbon-footprint)帮助您报告碳排放。使用Axion处理器，客户可以进一步优化能效。

### **Axion - 开箱即用的应用兼容性和互操作性**

Google还有丰富的贡献历史，为Arm生态系统做出了贡献。我们构建并开源了Android、Kubernetes、TensorFlow和Go语言，并与Arm及行业合作伙伴密切合作，优化了它们以适应Arm架构。

Axion基于标准的Armv9架构和指令集。最近，我们为SystemReady Virtual Environment (VE)做出了贡献，这是Arm的硬件和固件互操作性标准，确保常见操作系统和软件包可以无缝运行在基于Arm的服务器和虚拟机上，使客户更容易在Google Cloud上部署Arm工作负载，几乎无需重新编写代码。通过这种合作，我们可以访问已部署工作负载并利用来自数百个ISV和开源项目的Arm本地软件的数万个云客户的生态系统。

客户将能够在多个 Google Cloud 服务中使用 Axion，包括 Google Compute Engine、Google Kubernetes Engine、Dataproc、Dataflow、Cloud Batch 等。 Arm 兼容的软件和解决方案现在可以在 [Google Cloud Marketplace](https://cloud.google.com/marketplace?hl=en) 上找到，并且我们最近在 [Migrate to Virtual Machines](https://cloud.google.com/migrate/virtual-machines) 服务中推出了 Arm-based 实例迁移的预览支持。

### **我们的客户和合作伙伴的见证**

“我们很高兴能在 VMware Tanzu 库中增加基于新的 Axion Arm-based CPU 构建的应用程序包。这将为我们的用户提供显著改进的性能、有吸引力的性价比以及更好的可持续性。我们迫不及待地想要尝试新的 Google Axion 实例，并且进一步帮助客户简化部署，减少环境足迹。” - Mike Wookey，Tanzu 部门高级研发总监，Broadcom

“全球各地的组织依靠 CrowdStrike 和我们的单一平台、单一代理架构来阻止云端入侵。CrowdStrike 提供行业最佳的保护，同时部署速度最快，因此我们对测试 Google 的新处理器充满期待，以发现其在性能和效率上的提升。” – Daniel Bernard，首席商务官，CrowdStrike

“我们的客户要求我们提供不妥协的网络安全保护。我们对 Google Cloud 的新定制 Arm-based CPU 可能带来的功效和效率提升感到好奇，并计划验证其能力，以为我们的客户提供更好的威胁检测和响应能力。” - Tzach Segal，商业发展副总裁，Cybereason

“Datadog 是客户采用基于 Arm 的虚拟机的信任合作伙伴，也是 Arm 技术的早期采用者。我们对 Google Cloud 宣布的 Axion 处理器感到兴奋，并计划在我们的工作负载上评估它，帮助客户衡量在 Google Cloud Arm 实例上使用 Datadog 的成本和性能优势。” - Yrieix Garnier，产品管理副总裁，Datadog

“Elastic 致力于帮助客户高效解锁其所有结构化和非结构化数据的潜力。我们期待测试 Google Cloud 的新定制 Arm-based CPU，并希望它能帮助我们在 Google Cloud 平台上提供更好的 Elastic 客户体验。” - Steve Kearns，产品副总裁，Elastic

“多年来，我们与 Google Cloud 建立了紧密的合作关系，并且看到了在 GCP 基于 Arm 的 VM 上构建的好处。我们迫不及待地期待与新一代基于 Arm 的 Axion 处理器带来的显著改进。” - Joel Meyer，工程高级副总裁，OpenX

"Snap 让每个人都能表达自己，活在当下，了解世界，并共同享乐。我们不断优化我们的基础设施以提升性能和效率。Google 的新 Axion Arm 架构 CPU 承诺在这两个方面取得重大进展。在达成我们的可持续发展目标的同时，利用这些进展为我们的社区服务潜力巨大。我们期待 Axion 架构虚拟机推出后的好处。” - Korwin Smith，Snap 云基础设施高级工程总监

“WP Engine 现在为来自 150 个国家的超过 150 万客户提供网站服务。他们依赖 WP Engine 提供的性能、可靠性和安全性，并将这一责任视为重中之重。我们致力于创新和以客户为中心的思维方式，意味着我们始终在寻找方法来进一步提升客户在线性能和信心。我们很高兴测试 Google 的新自定义 Arm 架构处理器及其预期的性能和可持续性提升，并探索它们如何帮助我们的客户创建更加引人入胜的网站和应用。” - Ramadass Prabhakar，WP Engine 高级副总裁兼首席技术官

### **了解更多**

基于 Axion 处理器的虚拟机将在未来几个月内提供预览。[立即注册您的兴趣](https://docs.google.com/forms/d/e/1FAIpQLSdmFDDBNffCScti1FLlum71Q2V9kBANNKIy_2fd85iSgMcj9Q/viewform)！如果您参加了 Next ‘24，请了解更多关于 Axion 和其他计算相关的发布活动：

* * *

^(*1\. Google Cloud 内部数据，2024 年 3 月 31 日

2\. [Google 环境报告](https://www.gstatic.com/gumdrop/sustainability/google-2023-environmental-report.pdf)，2023 年，第 10 页。*)
