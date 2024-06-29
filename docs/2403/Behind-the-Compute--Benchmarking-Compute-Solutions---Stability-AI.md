<!--yml

category: 未分类

date: 2024-05-27 14:50:01

-->

# Behind the Compute: Benchmarking Compute Solutions — Stability AI

> 来源：[https://stability.ai/news/putting-the-ai-supercomputer-to-work](https://stability.ai/news/putting-the-ai-supercomputer-to-work)

在这种配置下，与A100-80GB GPU相比，Gaudi 2集群每秒处理的图像数量提高了3倍以上。考虑到A100具有非常优化的软件堆栈，这一点尤为令人印象深刻。

在稳定扩散3 8B参数模型的推断测试中，Gaudi 2芯片使用基础PyTorch提供了与Nvidia A100芯片类似的推断速度。然而，通过TensorRT优化，A100芯片的图像生成速度比Gaudi 2快40%。我们预计随着进一步的优化，Gaudi 2将很快在这个模型上超越A100。在早期对我们的SDXL模型进行的基于基础PyTorch的测试中，Gaudi 2在30步中以3.2秒生成1024x1024图像，而A100上的PyTorch为3.6秒，A100上的TensorRT为2.7秒。

Gaudi 2的更高内存和快速互连，加上其他设计考虑因素，使其在支持下一代媒体模型的扩散变压器架构方面具有竞争力。

**模型2：**

[**稳定Beluga 2.5 70B**](https://stability.ai/news/stable-beluga-large-instruction-fine-tuned-models)是我们对LLaMA 2 70B的优化版本，建立在第一个在特定基准中超越ChatGPT 3.5的开放模型[**稳定Beluga 2**](https://stability.ai/news/stable-beluga-large-instruction-fine-tuned-models)基础上。我们在256个Gaudi 2加速器上运行了这个训练基准。我们使用PyTorch代码直接运行，没有额外的优化，测得了令人印象深刻的平均总吞吐量为116,777 tokens/second。具体来说，这包括使用FP16数据类型，全局批大小为1024，梯度累积步骤为2，微批大小为2。

On inference tests with our 70B language model on Gaudi 2, it generates 673 tokens/second per accelerator, using an input token size of 128 and output token size of 2048\. In comparison to [TensorRT-LLM](https://nvidia.github.io/TensorRT-LLM/performance.html), Gaudi 2 appears to be 28% faster than the 525 tokens/second for the A100\. We also anticipate further speed improvements with FP8.

像我们这样的公司面临着对更强大和高效的计算解决方案日益增长的需求。我们的研究结果强调了像Gaudi 2这样的替代方案的必要性，它不仅提供比其他7nm芯片更优越的性能，还解决了关键市场需求，如可负担性、缩短交货时间以及卓越的性价比。最终，计算选项的多样性扩大了参与和创新的机会，从而使先进的AI技术更加普及。

Stay tuned for more insights in our next installment of "Behind the Compute."

要了解我们的进展，请关注我们的[Twitter](https://twitter.com/stabilityai)，[Instagram](https://www.instagram.com/stability.ai/)，[LinkedIn](https://www.linkedin.com/company/stability-ai)，并加入我们的[Discord社区](https://discord.gg/stablediffusion)。
