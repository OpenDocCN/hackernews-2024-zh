<!--yml

category: 未分类

date: 2024-05-27 13:33:57

-->

# Qwen1.5-110B：Qwen1.5系列的第一个100B+模型 | Qwen

> 来源：[https://qwenlm.github.io/blog/qwen1.5-110b/](https://qwenlm.github.io/blog/qwen1.5-110b/)

[GITHUB](https://github.com/QwenLM/Qwen1.5) [HUGGING FACE](https://huggingface.co/Qwen) [MODELSCOPE](https://modelscope.cn/organization/qwen) [DEMO](https://huggingface.co/spaces/Qwen/Qwen1.5-110B-Chat-Demo) [DISCORD](https://discord.gg/yPEP2vHTu4)

# Introduction[#](#introduction)

最近我们在开源社区中见证了一波超过1000亿参数的大型模型的涌现。这些模型在基准评估和聊天机器人领域表现出色。今天，我们发布了Qwen1.5系列的第一个100B+模型，Qwen1.5-110B，在基础模型评估中与Meta-Llama3-70B有着可比较的表现，在包括MT-Bench和AlpacaEval 2.0在内的聊天评估中表现出色。

# Model Features[#](#model-features)

Qwen1.5-110B与其他Qwen1.5模型类似，采用了相同的Transformer解码器架构。它包括分组查询注意力（GQA），在模型服务中可以高效运行。该模型支持32K令牌的上下文长度，仍然是多语言的，支持包括英语、中文、法语、西班牙语、德语、俄语、韩语、日语、越南语、阿拉伯语等多种语言。

# Model Quality[#](#model-quality)

我们对基础语言模型进行了一系列评估，并与最近的SOTA语言模型Meta-Llama3-70B以及Mixtral-8x22B进行了比较。

|  | Qwen1.5-110B | Qwen1.5-72B | Llama-3-70B | Mixtral-8x22B |
| --- | --- | --- | --- | --- |
| MMLU | 80.4 | 77.5 | 79.5 | 77.8 |
| TheoremQA | 34.9 | 29.3 | 32.0 | 35.9 |
| GPQA | 35.9 | 36.3 | 36.4 | 34.3 |
| Hellaswag | 87.5 | 86.0 | 88.0 | 88.7 |
| BBH | 74.8 | 65.5 | 76.6 | 69.2 |
| ARC-C | 69.6 | 65.9 | 68.8 | 70.7 |
| GSM8K | 85.4 | 79.5 | 79.2 | 78.6 |
| MATH | 49.6 | 34.1 | 41.0 | 41.7 |
| HumanEval | 52.4 | 41.5 | 45.7 | 45.1 |
| MBPP | 58.1 | 53.4 | 55.1 | 71.2 |

上述结果显示，新的110B模型至少在基础能力方面与Llama-3-70B模型相媲美。在该模型中，我们没有大幅改变预训练和后训练配方，因此我们相信与72B相比的性能提升来自增加模型大小。

我们还在MT-Bench和AlpacaEval 2.0上测试了聊天模型。

| Models | MT-Bench | AlpacaEval 2.0 |
| --- | --- | --- |
| 平均分数 | LC胜率 |
| Llama-3-70B-Instruct | 8.85 | 34.40 |
| Qwen1.5-72B-Chat | 8.61 | 36.60 |
| Qwen1.5-110B-Chat | 8.88 | 43.90 |

与之前发布的72B模型相比，在两个聊天模型的基准评估中，110B表现显著更好。评估中的持续改进表明，更强大更大的基础语言模型即使在不大幅改变训练后配方的情况下，也能导致更好的聊天模型。

# 使用Qwen1.5-110B进行开发[#](#develop-with-qwen15-110b)

我们建议您阅读我们的博客，了解有关使用Transformers、vLLM、llama.cpp、Ollama、LMStudio、SkyPilot、Axolotl、LLaMA-Factory等内容的[Qwen1.5](https://qwenlm.github.io/blog/qwen1.5/)用法。

# 结论[#](#conclusion)

Qwen1.5-110B是Qwen1.5系列中最大的模型，也是该系列中首个具有超过1000亿参数的模型。它展示了与最近发布的SOTA模型Llama-3-70B的竞争性表现，并且明显优于72B模型。这告诉我们，在模型尺寸扩展方面还有很大的优化空间。虽然Llama-3的发布表明数据扩展至极大规模的重要性，但我们相信通过在未来的发布中同时扩展数据和模型尺寸，我们可以兼顾两者的优势。敬请关注Qwen2的发布！

# 引用[#](#citation)

```
@misc{qwen1.5,
    title = {Introducing Qwen1.5},
    url = {https://qwenlm.github.io/blog/qwen1.5/},
    author = {Qwen Team},
    month = {February},
    year = {2024}
} 
```
