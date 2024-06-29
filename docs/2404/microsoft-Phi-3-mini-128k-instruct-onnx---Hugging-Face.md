<!--yml

类别：未分类

日期：2024-05-27 13:27:36

-->

# microsoft/Phi-3-mini-128k-instruct-onnx · Hugging Face

> 来源：[https://huggingface.co/microsoft/Phi-3-mini-128k-instruct-onnx](https://huggingface.co/microsoft/Phi-3-mini-128k-instruct-onnx)

# [](#phi-3-mini-128k-instruct-onnx-models)Phi-3 Mini-128K-Instruct ONNX 模型

该存储库托管了优化版本的 [Phi-3-mini-128k-instruct](https://aka.ms/phi3-mini-128k-instruct)，以加速使用 ONNX Runtime 的推理过程。

Phi-3 Mini 是一个轻量级、最新开放模型，基于 Phi-2 使用的数据集构建 - 合成数据和经过筛选的网站 - 重点关注非常高质量、推理密集的数据。该模型属于 Phi-3 模型系列，迷你版本有两个变体：4K 和 128K（这是它能支持的上下文长度（以标记计））。该模型经过严格的增强过程，包括监督的微调和直接的偏好优化，以确保精确的指令遵循和稳健的安全措施。

优化的 Phi-3 Mini 模型以 [ONNX](https://onnx.ai) 格式发布，以便在包括服务器平台、Windows、Linux 和 Mac 桌面以及移动 CPU 在内的设备上通过 [ONNX Runtime](https://onnxruntime.ai/) 运行，每个目标都选择最适合的精度。

[DirectML](https://aka.ms/directml) 支持使开发人员能够在 Windows 设备上扩展硬件加速，覆盖 AMD、Intel 和 NVIDIA GPU。除 DirectML 外，ONNX Runtime 还为各种设备提供跨平台支持，适用于 CPU、GPU 和移动设备上的 Phi-3 Mini。

要轻松开始使用 Phi-3，您可以使用我们新推出的 ONNX Runtime Generate() API。请参阅 [此处](https://aka.ms/generate-tutorial) 获取有关如何运行的说明。

## [](#onnx-models)ONNX 模型

这里是我们添加的一些优化配置：

1.  int4 DML 的 ONNX 模型：适用于 AMD、Intel 和 NVIDIA GPU 上的 Windows，使用 [AWQ](https://arxiv.org/abs/2306.00978) 进行 int4 量化。

1.  fp16 CUDA 的 ONNX 模型：你可以用于 NVIDIA GPU 运行的 ONNX 模型。

1.  int4 CUDA 的 ONNX 模型：使用 RTN 进行 NVIDIA GPU 的 int4 量化。

1.  int4 CPU 和移动设备的 ONNX 模型：用于 CPU 和移动设备的 ONNX 模型，使用 RTN 进行 int4 量化。已上传两个版本，以平衡延迟和准确性。Acc=1 旨在提高准确性，而 Acc=4 则是为了提高性能。对于移动设备，我们建议使用具有 acc-level-4 的模型。

更多关于 AMD 的更新，以及在正式发布 ORT 1.18 版本时对 CPU 和移动设备的额外优化将会添加进来。请关注！

## [](#hardware-supported)硬件支持

这些模型已在以下设备上进行了测试：

+   GPU SKU：RTX 4090（DirectML）

+   GPU SKU：1 A100 80GB GPU，SKU：Standard_ND96amsr_A100_v4（CUDA）

+   CPU SKU：标准 F64s v2（64 个 vCPU，128 GiB 内存）

+   移动 SKU：三星 Galaxy S21

最低配置要求：

+   Windows：需要 DirectX 12 兼容 GPU 和至少 4GB 的组合内存

+   CUDA：具有[Compute Capability](https://developer.nvidia.com/cuda-gpus) >= 7.0的NVIDIA GPU

### [](#model-description)模型描述

+   **开发者：** Microsoft

+   **模型类型：** ONNX

+   **语言（NLP）：** Python，C，C++

+   **许可证：** MIT

+   **模型描述：** 这是Phi-3 Mini-4K-Instruct模型为ONNX Runtime推理转换的结果。

## [](#additional-details)附加细节

## [](#how-to-get-started-with-the-model)如何开始使用该模型

为了使Phi-3模型在多种设备和平台上使用各种执行提供程序后端变得可能，我们引入了一个新的API来包装生成式AI推理的几个方面。该API使得将LLM直接拖放到您的应用程序中变得非常容易。要在ONNX Runtime中运行这些模型的早期版本，请按照这里的步骤进行操作[here](http://aka.ms/generate-tutorial)。

例如：

```
python model-qa.py -m /*{YourModelPath}*/onnx/cpu_and_mobile/phi-3-mini-4k-instruct-int4-cpu -k 40 -p 0.95 -t 0.8 -r 1.0 
```

```
*Input:*  <|user|>Tell me a joke<|end|><|assistant|>

*Output:*  Why don't scientists trust atoms?
           Because they make up everything!

This joke plays on the double meaning of "make up." In science, atoms are the fundamental building blocks of matter, literally making up everything. However, in a colloquial sense, "to make up" can mean to fabricate or lie, hence the humor. 
```

## [](#performance-metrics)性能指标

Phi-3 Mini-128K-Instruct在所有批次大小和提示长度组合中，比PyTorch在ONNX Runtime中表现更好。对于FP16 CUDA，ORT的性能比PyTorch快了多达5倍，而对于INT4 CUDA，它比PyTorch快了多达9倍。

下表显示了在CUDA上使用FP16和INT4精度测量的前256个生成的平均吞吐量（tps），使用的硬件是[1 A100 80GB GPU，SKU：Standard_ND96amsr_A100_v4](https://learn.microsoft.com/en-us/azure/virtual-machines/ndm-a100-v4-series)。

| 批次大小，提示长度 | ORT FP16 CUDA | PyTorch Eager FP16 CUDA | FP16 CUDA加速比（ORT/PyTorch） |
| --- | --- | --- | --- |
| 1, 16 | 134.46 | 25.35 | 5.30 |
| 1, 64 | 132.21 | 25.69 | 5.15 |
| 1, 256 | 124.51 | 25.77 | 4.83 |
| 1, 1024 | 110.03 | 25.73 | 4.28 |
| 1, 2048 | 96.93 | 25.72 | 3.77 |
| 1, 4096 | 62.12 | 25.66 | 2.42 |
| 4, 16 | 521.10 | 101.31 | 5.14 |
| 4, 64 | 507.03 | 101.66 | 4.99 |
| 4, 256 | 459.47 | 101.15 | 4.54 |
| 4, 1024 | 343.60 | 101.09 | 3.40 |
| 4, 2048 | 264.81 | 100.78 | 2.63 |
| 4, 4096 | 158.00 | 77.98 | 2.03 |
| 16, 16 | 1689.08 | 394.19 | 4.28 |
| 16, 64 | 1567.13 | 394.29 | 3.97 |
| 16, 256 | 1232.10 | 405.30 | 3.04 |
| 16, 1024 | 680.61 | 294.79 | 2.31 |
| 16, 2048 | 350.77 | 203.02 | 1.73 |
| 16, 4096 | 192.36 | OOM |  |
| 批次大小，提示长度 | PyTorch Eager INT4 CUDA | INT4 CUDA加速比（ORT/PyTorch） |
| --- | --- | --- |
| 1, 16 | 25.35 | 8.89 |
| 1, 64 | 25.69 | 8.58 |
| 1, 256 | 25.77 | 7.69 |
| 1, 1024 | 25.73 | 6.34 |
| 1, 2048 | 25.72 | 5.24 |
| 1, 4096 | 25.66 | 2.97 |
| 4, 16 | 101.31 | 2.82 |
| 4, 64 | 101.66 | 2.77 |
| 4, 256 | 101.15 | 2.64 |
| 4, 1024 | 101.09 | 2.20 |
| 4, 2048 | 100.78 | 1.84 |
| 4, 4096 | 77.98 | 1.62 |
| 16, 16 | 394.19 | 2.52 |
| 16, 64 | 394.29 | 2.41 |
| 16, 256 | 405.30 | 2.00 |
| 16, 1024 | 294.79 | 1.79 |
| 16, 2048 | 203.02 | 1.81 |
| 16, 4096 | OOM |  |

注：PyTorch编译和Llama.cpp当前不支持Phi-3 Mini-128K-Instruct模型。

### [](#package-versions)包版本

| Pip软件包名称 | 版本 |
| --- | --- |
| torch | 2.2.0 |
| triton | 2.2.0 |
| onnxruntime-gpu | 1.18.0 |
| onnxruntime-genai | 0.2.0 |
| onnxruntime-genai-cuda | 0.2.0 |
| onnxruntime-genai-directml | 0.2.0 |
| transformers | 4.39.0 |
| bitsandbytes | 0.42.0 |

## [](#appendix)附录

### [](#activation-aware-quantization)激活感知量化

AWQ 通过识别对维持准确性最重要的前1%的显著权重，并对其余99%的权重进行量化来工作。与许多其他量化技术相比，这导致了从量化中损失的准确性较少。有关AWQ的更多信息，请参见[这里](https://arxiv.org/abs/2306.00978)。

## [](#model-card-contact)模型卡联系人

parinitarahi, kvaishnavi, natke

## [](#contributors)贡献者

Kunal Vaishnavi, Sunghoon Choi, Yufeng Li, Akshay Sonawane, Sheetal Arun Kadam, Rui Ren, Edward Chen, Scott McKay, Ryan Hill, Emma Ning, Natalie Kershaw, Parinita Rahi, Patrice Vignola, Chai Chaoweeraprasit, Logan Iyer, Vicente Rivera, Jacques Van Rhyn
