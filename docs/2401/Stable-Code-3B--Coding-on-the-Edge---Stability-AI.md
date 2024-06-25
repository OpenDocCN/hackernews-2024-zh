<!--yml

分类：未分类

日期：2024-05-27 14:49:25

-->

# Stable Code 3B：边缘编码 — Stability AI

> 来源：[`stability.ai/news/stable-code-2024-llm-code-completion-release`](https://stability.ai/news/stable-code-2024-llm-code-completion-release)

今天，我们宣布我们 2024 年的第一个大型语言模型发布：[Stable Code 3B](https://huggingface.co/stabilityai/stable-code-3b)。这个新的大型语言模型是我们之前发布的 [Stable Code Alpha 3B](https://stability.ai/news/stablecode-llm-generative-ai-coding) 的后续版本，也是第一个重要的 Stable Code 发布，提供了一个新的设计用于代码完成的最新模型，并具有多个额外功能。

相对于 CodeLLaMA 7b，Stable Code 3B 的体积减小了 60%，同时在各种编程语言中保持了类似的高性能水平。基于我们预先存在的 [Stable LM 3B](https://huggingface.co/stabilityai/stablelm-3b-4e1t) 基础模型，该模型在 4 万亿个自然语言数据令牌上进行了训练，Stable Code 进一步在软件工程特定数据上进行了训练，包括代码。该模型紧凑的体积使其能够在现代笔记本电脑上即使没有专用 GPU 也能够在边缘实时运行。

Stable Code 3B 提供了更多功能，并且在多种语言中的性能显著提升，还增加了支持填充功能 (FIM) 和扩展上下文大小等额外优势。Stable Code 作为基础模型的序列训练长度为最多 16,384 个标记，但是遵循类似于 CodeLlama 的方法实现了 Rotary Embeddings，可选地允许修改旋转基数达到 1,000,000，进一步扩展模型的上下文长度达到 100k 个标记。

Stable Code 在 18 种编程语言上进行了训练（根据 [2023 年 StackOverflow 开发者调查](https://survey.stackoverflow.co/2023/#section-most-popular-technologies-programming-scripting-and-markup-languages) 进行选择），并在多种测试的编程语言中的 MultiPL-E 指标上表现出同类大小模型的最新性能。

**性能比较**
