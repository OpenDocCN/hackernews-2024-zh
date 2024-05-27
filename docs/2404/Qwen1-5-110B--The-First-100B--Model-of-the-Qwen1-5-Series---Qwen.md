<!--yml
category: 未分类
date: 2024-05-27 13:33:57
-->

# Qwen1.5-110B: The First 100B+ Model of the Qwen1.5 Series | Qwen

> 来源：[https://qwenlm.github.io/blog/qwen1.5-110b/](https://qwenlm.github.io/blog/qwen1.5-110b/)

[GITHUB](https://github.com/QwenLM/Qwen1.5) [HUGGING FACE](https://huggingface.co/Qwen) [MODELSCOPE](https://modelscope.cn/organization/qwen) [DEMO](https://huggingface.co/spaces/Qwen/Qwen1.5-110B-Chat-Demo) [DISCORD](https://discord.gg/yPEP2vHTu4)

# Introduction[#](#introduction)

Recently we have witnessed a burst of large-scale models with over 100 billion parameters in the opensource community. These models have demonstrated remarkable performance in both benchmark evaluation and chatbot arena. Today, we release the first 100B+ model of the Qwen1.5 series, Qwen1.5-110B, which achieves comparable performance with Meta-Llama3-70B in the base model evaluation, and outstanding performance in the chat evaluation, including MT-Bench and AlpacaEval 2.0.

# Model Features[#](#model-features)

Qwen1.5-110B is similar to other Qwen1.5 models and built with the same Transformer decoder architecture. It consists of grouped query attention (GQA) and it can be efficient in model serving. The model supports the context length 32K tokens, and the model is still multilingual, supporting a large number of languages including English, Chinese, French, Spanish, German, Russian, Korean, Japanese, Vietnamese, Arabic, etc.

# Model Quality[#](#model-quality)

We conduct a series of evaluations for the base language models, and we compare with Meta-Llama3-70B, the recent SOTA language model as well as Mixtral-8x22B.

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

The above results show that the new 110B model is at least competitive with the Llama-3-70B model in terms of base capabilities. In terms of this model, we did not change the pretraining and posttraining recipes drastically, and thus we believe that the performance improvement compared with 72B comes from increasing model size.

We also test the chat models on MT-Bench and AlpacaEval 2.0.

| Models | MT-Bench | AlpacaEval 2.0 |
| Avg. Score | LC Win Rate |
| Llama-3-70B-Instruct | 8.85 | 34.40 |
| Qwen1.5-72B-Chat | 8.61 | 36.60 |
| Qwen1.5-110B-Chat | 8.88 | 43.90 |

Compared with the previously released 72B model, on the two benchmark evaluation for chat models the 110B performs significantly better. The consistent improvement in the evaluation indicates that stronger and larger base language models can lead to better chat models even without changing the post-training recipes much.

# Develop with Qwen1.5-110B[#](#develop-with-qwen15-110b)

We advise you to read our blog for [Qwen1.5](https://qwenlm.github.io/blog/qwen1.5/) to figure out the usages with Transformers, vLLM, llama.cpp, Ollama, LMStudio, SkyPilot, Axolotl, LLaMA-Factory, etc.

# Conclusion[#](#conclusion)

The Qwen1.5-110B is the largest model in the Qwen1.5 series, and it is also the first one with over 100 billion parameters in the series. It demonstrates competitive performance against the very recently released SOTA model Llama-3-70B and it is significantly better than the 72B model. This tells us that there is still a lot of room in model size scaling for better performance. While the releease of Llama-3 indicates the significance of data scaling to an extremely large scale, we believe we can get the best of both worlds by scaling both data and model size in our future release. Stay tuned for Qwen2!

# Citation[#](#citation)

```
@misc{qwen1.5,
    title = {Introducing Qwen1.5},
    url = {https://qwenlm.github.io/blog/qwen1.5/},
    author = {Qwen Team},
    month = {February},
    year = {2024}
} 
```