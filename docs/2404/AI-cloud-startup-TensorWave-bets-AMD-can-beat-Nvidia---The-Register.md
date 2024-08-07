<!--yml

category: 未分类

date: 2024-05-27 13:15:19

-->

# AI云初创公司TensorWave押注AMD能击败Nvidia • The Register

> 来源：[https://www.theregister.com/2024/04/16/amd_tensorwave_mi300x/](https://www.theregister.com/2024/04/16/amd_tensorwave_mi300x/)

熟练运行热门和耗电量大的GPU及其他AI基础设施的专业云操作员正在涌现，尽管像CoreWeave、Lambda或Voltage Park这样的玩家已经用数万个Nvidia GPU构建了他们的集群，但也有一些玩家转向了AMD。

一个例子是比特谷初创公司TensorWave，本月初开始安装由AMD的Instinct MI300X驱动的系统，计划以低于访问Nvidia加速器成本的价格出租这些芯片。

TensorWave联合创始人Jeff Tatarchuk认为AMD的最新加速器有很多优点。首先，你可以真正地购买到它们。TensorWave已经获得了大量的部件分配。

到2024年底，TensorWave计划在两个设施部署2万个MI300X加速器，并计划明年启动更多的液冷系统。

AMD的最新AI硅片也比Nvidia的备受追捧的H100更快。"仅从原始规格来看，MI300x占据了H100的主导地位，" Tatarchuk说。

在AMD的“推进AI”活动中推出的MI300X是该芯片设计公司迄今为止最先进的加速器。该[750W芯片](https://www.theregister.com/2023/12/06/amd_mi300_gpu/)使用先进的封装技术将12个芯片（如果计算HBM3模块则为20个）组装成单个GPU，声称比Nvidia的H100快32%。

除了更高的浮点性能外，该芯片还拥有更大的192GB HBM3内存，能够提供5.3TB/s的带宽，而H100声称的带宽为80GB和3.35TB/s。

正如我们从Nvidia的H200（H100加入HBM3e提升版本）看到的那样，内存带宽是AI性能的[主要贡献者](https://www.theregister.com/2023/12/21/nvidia_amd_benchmarks/)，特别是在大型语言模型的推理中。

就像Nvidia的HGX和Intel的OAM设计一样，AMD最新GPU的标准配置要求每个节点配备八个加速器。

TensorWave 的配置正忙于安装和堆放。

"我们现在正在安装数百台设备，并计划未来几个月安装数千台，" Tatarchuk说道。

### 安装它们

在一张社交媒体上[发布的照片](https://twitter.com/TensorWaveCloud/status/1775018324373479654)，TensorWave团队展示了似乎是三台8U Supermicro AS-8125GS-TNMR2 [系统](https://www.supermicro.com/en/products/system/gpu/8u/as-8125gs-tnmr2)机架。这引发了我们对TensorWave的机架是否受到功率或热量限制的质疑，毕竟，这些系统在满负荷时往往会吸收超过10千瓦的电力。

原来，TensorWave 的人们还没有完成安装这些机器，该公司计划每个机架的四个节点，总容量约为每架 40 千瓦。这些系统将使用后门热交换器（RDHx）进行冷却。正如我们过去讨论的那样，这些是机架大小的散热器，通过其中流动的冷水冷却从传统服务器排出的热空气，使其降至可接受的水平。

这种冷却技术已成为数据中心运营商中的热门商品，他们希望支持更密集的 GPU 集群，并导致了一些供应链挑战，TensorWave 首席运营官 Piotr Tomasik 表示。

"当前数据中心周围的辅助设备存在许多容量问题，" 他说，特别是将 RDHx 视为痛点。"到目前为止，我们取得了成功，而且对我们部署它们的能力非常乐观。"

然而，长远来看，TensorWave 的目标是实现直接到芯片的冷却技术，这在原本没有设计用于安置 GPU 的数据中心中可能难以部署，Tomasik 说。"我们很高兴能在今年下半年部署直接到芯片的冷却技术。我们认为这样做会更好，也更容易实现高密度。"

### 性能焦虑

另一个挑战是对 AMD 性能的信心。据 Tatarchuk 介绍，尽管人们对 AMD 提供的替代品充满热情，但客户们不确定它们是否能够享受与 Nvidia 相同的性能。"人们也有很多 '我们并不100%确定它是否会像我们目前在 Nvidia 上所习惯的那样出色'，" 他说道。

为了尽快使系统运行起来，TensorWave 将使用 RoCE（RDMA over Converged Ethernet）推出其 MI300X 节点。这些裸金属系统将提供固定租赁期，据说每个 GPU 每小时仅需 $1。

### 扩展规模

随着时间的推移，该公司旨在引入一个更像云的编排层来提供资源。还计划采用基于 PCIe 5.0 的 GigaIO FabreX 技术，将多达 5,750 个 GPU 缝合在一个单一域中，具备超过一PB的高带宽内存。

所谓的 TensorNODE 是基于 GigaIO 的 SuperNODE 架构构建的，去年它在 [这里展示过](https://www.nextplatform.com/2023/08/15/crafting-a-dgx-alike-ai-server-out-of-amd-gpus-and-pci-switches/)，该架构使用了一对 PCIe 交换设备将多达 32 个 AMD MI210 GPU 连接在一起。理论上，这应该使单个 CPU 主节点能够处理远远超过当前 GPU 节点中通常见到的八个加速器。

这种方法与英伟达首选的设计有所不同，后者使用NVLink将多个超级芯片连接在一起形成一个大型GPU。尽管NVLink在其 [最新版本](https://www.theregister.com/2024/03/21/nvidia_dgx_gb200_nvk72/) 中的带宽速度可达1.8TB/s，远高于PCIe 5.0的128GB/s，但仅支持高达576个GPU的配置。

TensorWave将通过将其GPU作为大笔债务融资的抵押品来资助其数据中心建设，这种方法被其他数据中心运营商所采用。就在上周，Lambda [披露](https://www.theregister.com/2024/04/05/lambda_500m_loan/)已获得5亿美元贷款，用于部署“数以万计”的英伟达最快加速器。

与此同时，CoreWeave，一家最大的GPU租赁服务提供商之一，成功地 [获得](https://www.prnewswire.com/news-releases/coreweave-secures-2-3-billion-debt-financing-facility-led-by-magnetar-capital-and-blackstone-to-meet-surging-demand-and-ongoing-expansion-of-specialized-cloud-infrastructure-to-power-ai-301892706.html)了23亿美元的贷款，以扩展其数据中心的覆盖面。

“您会预计我们在今年晚些时候也会有类似的公告，” Tomasik说道。®
