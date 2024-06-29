<!--yml

category: 未分类

date: 2024-05-29 12:45:06

-->

# Nvidia 在 Llama 2 和 Stable Diffusion 的速度测试中表现出色 - IEEE Spectrum

> 来源：[https://spectrum.ieee.org/ai-benchmark-mlperf-llama-stablediffusion](https://spectrum.ieee.org/ai-benchmark-mlperf-llama-stablediffusion)

时代在变，基准测试也随之变化。现在我们已经牢固处于大规模生成式人工智能时代，是时候将两个这样的庞然大物 [Llama 2 70B](https://spectrum.ieee.org/llama-2-llm) 和 [Stable Diffusion](https://spectrum.ieee.org/tag/stable-diffusion) XL 加入 [MLPerf](https://mlcommons.org/) 的推理测试了。第 4.0 版基准测试包括来自 23 家提交组织的超过 8500 个结果。从一开始就是这样，拥有 [Nvidia](https://www.nvidia.com/en-us/ai-data-science/) GPU 的计算机表现最佳，尤其是其 H200 处理器。但是来自 [Intel](https://www.intel.com/content/www/us/en/artificial-intelligence/overview.html) 和 [Qualcomm](https://www.qualcomm.com/research/artificial-intelligence) 的 AI 加速器也参与其中。

MLPerf 从去年开始进入 LLM 领域，当时它加入了一个文本摘要的基准测试 [GPT-J](https://www.eleuther.ai/artifacts/gpt-j)（一个拥有 60 亿参数的开源模型）。而具有 700 亿参数的 Llama 2 则大了一个数量级。因此，组织者 [MLCommons](https://mlcommons.org/)，一个总部位于旧金山的人工智能联盟，称其需要“一种不同类别的硬件”。

“就模型参数而言，Llama-2 对推理套件中的模型进行了显著增加，” [Mitchelle Rasquinha](https://www.linkedin.com/in/mrasquinha/)，Google 软件工程师兼 MLPerf 推理工作组联合主席，在新闻发布会上如此说道。

Stable Diffusion XL，新的 [文本到图像生成](https://spectrum.ieee.org/ai-art-generator) 基准测试，参数为 26 亿，不到 GPT-J 的一半。去年修订的推荐系统测试比两者都要大。

MLPerf 的基准测试涵盖了各种规模，最新的如 Llama 2 70B，参数数量在数百亿级别。MLCommons

测试被分为适用于 [数据中心](https://mlcommons.org/benchmarks/inference-datacenter/) 使用和适用于设备使用在世界上，或者所谓的 “[边缘](https://mlcommons.org/benchmarks/inference-datacenter/)”。对于每个基准测试，计算机可以以所谓的离线模式或更现实的方式进行测试。在离线模式下，它尽可能快地运行测试数据以确定其最大吞吐量。更现实的测试旨在模拟来自智能手机摄像头的数据流、汽车中所有摄像头和传感器的多个数据流，或者作为数据中心设置中的查询，例如。此外，一些系统在任务期间的功耗也被跟踪。

## 数据中心推理结果

新一代生成式 AI 类别的最佳表现者是一台 Nvidia H200 系统，结合了八个 GPU 和两个 [英特尔](https://spectrum.ieee.org/tag/intel) Xeon CPU。它在稳定扩散中每秒处理近 14 个查询，在 Llama 2 70B 中每秒处理约 27,000 个标记。其最接近的竞争对手是 8-GPU H100 系统。对于稳定扩散，性能差异不大，约为每秒 1 个查询，但在 Llama 2 70B 中差异较大。

H200 和 H100 相同的 [霍普架构](https://spectrum.ieee.org/nvidias-next-gpu-shows-that-transformers-are-transforming-ai)，但高带宽内存增加了约 75%，内存带宽增加了 43%。据 Nvidia 的 [Dave Salvator](https://www.linkedin.com/in/davesalvator/) 称，内存在 LLMs 中尤为重要，如果能够完全容纳在芯片上，与其他关键数据一起，性能表现更佳。这种内存差异显示在 Llama 2 的结果中，H200 在性能上比 H100 快约 45%。

据该公司称，配备 H100 GPU 的系统，由于软件改进，比 [去年九月的结果](https://spectrum.ieee.org/mlperf) 的 H100 系统快 2.4-2.9 倍。

虽然 H200 是 Nvidia 基准测试秀的明星，但其最新的 GPU 架构，[布莱克韦尔](https://spectrum.ieee.org/nvidia-blackwell)，上周正式揭幕，却悄然背后。 Salvator 没有透露带有该 GPU 的计算机可能何时在基准表中亮相。

就 Intel 而言，它继续提供其 [Gaudi 2](https://habana.ai/products/gaudi2/) 加速器作为至少参与 MLPerf 推理基准测试的公司中唯一选择的选项。从原始性能来看，英特尔的 7 纳米芯片在稳定扩散 XL 的 8-GPU 配置中交付的性能略低于 5 纳米 H100，对于 Llama 2 70B，其 Gaudi 2 的结果接近 Nvidia 性能的三分之一。然而，英特尔认为，如果您衡量性能与价格的比例（这是他们自己进行的评估，而不是 MLPerf），Gaudi 2 和 H100 相当。对于稳定扩散，英特尔计算其性能比 H100 高约 25%。对于 Llama 2 70B，无论是服务器模式还是离线模式，它要么相当，要么差 21%。

**高迪 2 的继任者，高迪 3 预计将在今年晚些时候到来。**

**英特尔还宣称了几个仅使用CPU的条目，显示在缺少GPU的情况下也可以达到合理的推理性能，尽管没有在Llama 2 70B或Stable Diffusion上。这是英特尔第五代至MLPerf推理竞赛中首次亮相的Xeon CPU，并且该公司声称与2023年9月的第四代Xeon系统相比，性能提升范围为18%至91%。**

## **边缘推断结果**

**尽管Llama 2 70B如此大，但并未在边缘类别进行测试，但Stable Diffusion XL是。这里表现最佳的系统使用了两个Nvidia L40S GPU和一个英特尔Xeon CPU。性能在延迟和每秒样本数中进行衡量。这套系统由总部位于台北的云基础设施公司[Wiwynn](https://www.wiwynn.com/)提交，单流模式下在不到2秒内产生答案。在离线模式下，它每秒生成1.26个结果。**

## **功耗**

**在数据中心类别中，围绕能效的竞争是在Nvidia和高通之间进行的。后者自一年多以来推出[Cloud AI 100](https://www.qualcomm.com/products/technology/processors/cloud-artificial-intelligence/cloud-ai-100)处理器以来，专注于能效高的推理。高通去年底推出了加速器芯片新一代[Cloud AI 100 Ultra](https://www.qualcomm.com/news/onq/2023/11/introducing-qualcomm-cloud-ai-100-ultra)，其首次结果出现在边缘和数据中心性能基准测试中。与Cloud AI 100 Pro结果相比，Ultra在每片芯片消耗低于150瓦的情况下，产生了2.5到3倍的性能提升。**

**在边缘推断入口中，高通是唯一尝试稳定扩散XL的公司，使用578瓦管理0.6个样本每秒。**

**来自您网站的文章**

**相关文章周围**
