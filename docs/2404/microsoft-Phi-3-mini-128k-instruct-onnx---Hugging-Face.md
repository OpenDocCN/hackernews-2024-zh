<!--yml
category: 未分类
date: 2024-05-27 13:27:36
-->

# microsoft/Phi-3-mini-128k-instruct-onnx · Hugging Face

> 来源：[https://huggingface.co/microsoft/Phi-3-mini-128k-instruct-onnx](https://huggingface.co/microsoft/Phi-3-mini-128k-instruct-onnx)

# [](#phi-3-mini-128k-instruct-onnx-models)Phi-3 Mini-128K-Instruct ONNX models

This repository hosts the optimized versions of [Phi-3-mini-128k-instruct](https://aka.ms/phi3-mini-128k-instruct) to accelerate inference with ONNX Runtime.

Phi-3 Mini is a lightweight, state-of-the-art open model built upon datasets used for Phi-2 - synthetic data and filtered websites - with a focus on very high-quality, reasoning dense data. The model belongs to the Phi-3 model family, and the mini version comes in two variants: 4K and 128K which is the context length (in tokens) it can support. The model underwent a rigorous enhancement process, incorporating both supervised fine-tuning and direct preference optimization to ensure precise instruction adherence and robust safety measures.

Optimized Phi-3 Mini models are published here in [ONNX](https://onnx.ai) format to run with [ONNX Runtime](https://onnxruntime.ai/) on CPU and GPU across devices, including server platforms, Windows, Linux and Mac desktops, and mobile CPUs, with the precision best suited to each of these targets.

[DirectML](https://aka.ms/directml) support lets developers bring hardware acceleration to Windows devices at scale across AMD, Intel, and NVIDIA GPUs. Along with DirectML, ONNX Runtime provides cross platform support for Phi-3 Mini across a range of devices for CPU, GPU, and mobile.

To easily get started with Phi-3, you can use our newly introduced ONNX Runtime Generate() API. See [here](https://aka.ms/generate-tutorial) for instructions on how to run it.

## [](#onnx-models)ONNX Models

Here are some of the optimized configurations we have added:

1.  ONNX model for int4 DML: ONNX model for AMD, Intel, and NVIDIA GPUs on Windows, quantized to int4 using [AWQ](https://arxiv.org/abs/2306.00978).
2.  ONNX model for fp16 CUDA: ONNX model you can use to run for your NVIDIA GPUs.
3.  ONNX model for int4 CUDA: ONNX model for NVIDIA GPUs using int4 quantization via RTN.
4.  ONNX model for int4 CPU and Mobile: ONNX model for your CPU and Mobile, using int4 quantization via RTN. There are two versions uploaded to balance latency vs. accuracy. Acc=1 is targeted at improved accuracy, while Acc=4 is for improved perf. For mobile devices, we recommend using the model with acc-level-4.

More updates on AMD, and additional optimizations on CPU and Mobile will be added with the official ORT 1.18 release in early May. Stay tuned!

## [](#hardware-supported)Hardware Supported

The models are tested on:

*   GPU SKU: RTX 4090 (DirectML)
*   GPU SKU: 1 A100 80GB GPU, SKU: Standard_ND96amsr_A100_v4 (CUDA)
*   CPU SKU: Standard F64s v2 (64 vcpus, 128 GiB memory)
*   Mobile SKU: Samsung Galaxy S21

Minimum Configuration Required:

*   Windows: DirectX 12-capable GPU and a minimum of 4GB of combined RAM
*   CUDA: NVIDIA GPU with [Compute Capability](https://developer.nvidia.com/cuda-gpus) >= 7.0

### [](#model-description)Model Description

*   **Developed by:** Microsoft
*   **Model type:** ONNX
*   **Language(s) (NLP):** Python, C, C++
*   **License:** MIT
*   **Model Description:** This is a conversion of the Phi-3 Mini-4K-Instruct model for ONNX Runtime inference.

## [](#additional-details)Additional Details

## [](#how-to-get-started-with-the-model)How to Get Started with the Model

To make running of the Phi-3 models across a range of devices and platforms across various execution provider backends possible, we introduce a new API to wrap several aspects of generative AI inferencing. This API make it easy to drag and drop LLMs straight into your app. For running the early version of these models with ONNX Runtime, follow the steps [here](http://aka.ms/generate-tutorial).

For example:

```
python model-qa.py -m /*{YourModelPath}*/onnx/cpu_and_mobile/phi-3-mini-4k-instruct-int4-cpu -k 40 -p 0.95 -t 0.8 -r 1.0 
```

```
*Input:*  <|user|>Tell me a joke<|end|><|assistant|>

*Output:*  Why don't scientists trust atoms?
           Because they make up everything!

This joke plays on the double meaning of "make up." In science, atoms are the fundamental building blocks of matter, literally making up everything. However, in a colloquial sense, "to make up" can mean to fabricate or lie, hence the humor. 
```

## [](#performance-metrics)Performance Metrics

Phi-3 Mini-128K-Instruct performs better in ONNX Runtime than PyTorch for all batch size, prompt length combinations. For FP16 CUDA, ORT performs up to 5X faster than PyTorch, while with INT4 CUDA it's up to 9X faster than PyTorch.

The table below shows the average throughput of the first 256 tokens generated (tps) for FP16 and INT4 precisions on CUDA as measured on [1 A100 80GB GPU, SKU: Standard_ND96amsr_A100_v4](https://learn.microsoft.com/en-us/azure/virtual-machines/ndm-a100-v4-series).

| Batch Size, Prompt Length | ORT FP16 CUDA | PyTorch Eager FP16 CUDA | FP16 CUDA Speed Up (ORT/PyTorch) |
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

| Batch Size, Prompt Length | PyTorch Eager INT4 CUDA | INT4 CUDA Speed Up (ORT/PyTorch) |
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

Note: PyTorch compile and Llama.cpp currently do not support the Phi-3 Mini-128K-Instruct model.

### [](#package-versions)Package Versions

| Pip package name | Version |
| --- | --- |
| torch | 2.2.0 |
| triton | 2.2.0 |
| onnxruntime-gpu | 1.18.0 |
| onnxruntime-genai | 0.2.0 |
| onnxruntime-genai-cuda | 0.2.0 |
| onnxruntime-genai-directml | 0.2.0 |
| transformers | 4.39.0 |
| bitsandbytes | 0.42.0 |

## [](#appendix)Appendix

### [](#activation-aware-quantization)Activation Aware Quantization

AWQ works by identifying the top 1% most salient weights that are most important for maintaining accuracy and quantizing the remaining 99% of weights. This leads to less accuracy loss from quantization compared to many other quantization techniques. For more on AWQ, see [here](https://arxiv.org/abs/2306.00978).

## [](#model-card-contact)Model Card Contact

parinitarahi, kvaishnavi, natke

## [](#contributors)Contributors

Kunal Vaishnavi, Sunghoon Choi, Yufeng Li, Akshay Sonawane, Sheetal Arun Kadam, Rui Ren, Edward Chen, Scott McKay, Ryan Hill, Emma Ning, Natalie Kershaw, Parinita Rahi, Patrice Vignola, Chai Chaoweeraprasit, Logan Iyer, Vicente Rivera, Jacques Van Rhyn