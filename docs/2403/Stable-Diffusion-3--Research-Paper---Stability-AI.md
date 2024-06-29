<!--yml

category: 未分类

date: 2024-05-27 14:38:37

-->

# 稳定扩散3：研究论文 — Stability AI

> 来源：[https://stability.ai/news/stable-diffusion-3-research-paper](https://stability.ai/news/stable-diffusion-3-research-paper)

我们已经将稳定扩散3的输出图像与其他各种开放模型（包括[SDXL](https://stability.ai/news/stable-diffusion-sdxl-1-announcement)，[SDXL Turbo](https://stability.ai/news/stability-ai-sdxl-turbo)，[稳定级联](https://stability.ai/news/introducing-stable-cascade)，Playground v2.5和Pixart-α）以及封闭源系统（如DALL·E 3，Midjourney v6和Ideogram v1）进行比较，以人类反馈评估性能。在这些测试中，人类评估员得到了来自每个模型的示例输出，并被要求根据模型输出如何紧随所给的背景（“prompt following”）、根据提示如何呈现文本（“typography”）以及哪张图像具有更高美学质量（“visual aesthetics”）来选择最佳结果。

从我们的测试结果来看，我们发现稳定扩散3在上述所有领域中要么与当前最先进的文本到图像生成系统相当，要么表现更好。

在消费者硬件上早期的、未经优化的推理测试中，我们最大的SD3模型具有8B参数，在RTX 4090的24GB VRAM中运行，使用50个采样步骤时，生成分辨率为1024x1024的图像需时34秒。此外，初始发布期间将会有多个稳定扩散3的变体，从8亿到80亿参数的模型，以进一步消除硬件限制。
