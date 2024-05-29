<!--yml
category: 未分类
date: 2024-05-27 14:50:01
-->

# Behind the Compute: Benchmarking Compute Solutions — Stability AI

> 来源：[https://stability.ai/news/putting-the-ai-supercomputer-to-work](https://stability.ai/news/putting-the-ai-supercomputer-to-work)

In this configuration, the Gaudi 2 cluster processed over 3x more images per second, compared to A100-80GB GPUs. This is particularly impressive considering that the A100s have a very optimized software stack. 

On inference tests with the Stable Diffusion 3 8B parameter model the Gaudi 2 chips offer inference speed similar to Nvidia A100 chips using base PyTorch. However, with TensorRT optimization, the A100 chips produce images 40% faster than Gaudi 2\. We anticipate that with further optimization, Gaudi 2 will soon outperform A100s on this model. In earlier tests on our SDXL model with base PyTorch, Gaudi 2 generates a 1024x1024 image in 30 steps in 3.2 seconds, versus 3.6 seconds for PyTorch on A100s and 2.7 seconds for a generation with TensorRT on an A100. 

The higher memory and fast interconnect of Gaudi 2, plus other design considerations, make it competitive to run the Diffusion Transformer architecture that underpins this next generation of media models.

### 
**Model 2:**

[**Stable Beluga 2.5 70B**](https://stability.ai/news/stable-beluga-large-instruction-fine-tuned-models)is our fine-tuned version of LLaMA 2 70B, building on the [Stable Beluga 2](https://stability.ai/news/stable-beluga-large-instruction-fine-tuned-models) model which was the first open model to best ChatGPT 3.5 in select benchmarks. We ran this training benchmark on 256 Gaudi 2 accelerators. Running our PyTorch code out of the box, with no extra optimizations, we measured an impressive total average throughput of 116,777 tokens/second. More specifically, this involves using a FP16 datatype, a global batch size of 1024, gradient accumulation steps of 2, and micro batch size of 2.

On inference tests with our 70B language model on Gaudi 2, it generates 673 tokens/second per accelerator, using an input token size of 128 and output token size of 2048\. In comparison to [TensorRT-LLM](https://nvidia.github.io/TensorRT-LLM/performance.html), Gaudi 2 appears to be 28% faster than the 525 tokens/second for the A100\. We also anticipate further speed improvements with FP8.

Companies like ours face an increasing demand for more powerful and efficient computing solutions. Our findings underscore the need for alternatives like the Gaudi 2, which not only offers superior performance to other 7nm chips, but also addresses critical market needs such as affordability, reduced lead times, and superior price-to-performance ratios. Ultimately, the opportunity for choice in computing options broadens participation and innovation, thereby making advanced AI technologies more accessible to all.

Stay tuned for more insights in our next installment of "Behind the Compute." 

To stay updated on our progress follow us on [Twitter](https://twitter.com/stabilityai), [Instagram](https://www.instagram.com/stability.ai/), [LinkedIn](https://www.linkedin.com/company/stability-ai), and join our [Discord Community](https://discord.gg/stablediffusion).