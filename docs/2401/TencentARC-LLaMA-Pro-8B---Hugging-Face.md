<!--yml

category: 未分类

date: 2024-05-27 14:33:03

-->

# TencentARC/LLaMA-Pro-8B · Hugging Face

> 来源：[`huggingface.co/TencentARC/LLaMA-Pro-8B`](https://huggingface.co/TencentARC/LLaMA-Pro-8B)

# [](#llama-pro-8b-model-card)LLaMA-Pro-8B 模型卡

## [](#model-description)模型描述

LLaMA-Pro 是原始 LLaMA 模型的进步版本，通过添加 Transformer 块进行增强。它专门用于整合一般语言理解和特定领域的知识，尤其是在编程和数学方面。

## [](#development-and-training)开发和培训

由腾讯 ARC 实验室开发的 LLaMA-Pro 是一个包含 83 亿个参数的模型。它是 LLaMA2-7B 的扩展，进一步在总计 800 亿个标记的代码和数学语料库上进行了训练。

## [](#intended-use)预期用途

该模型旨在进行广泛的 NLP 任务，重点关注编程、数学和一般语言任务。它适用于需要自然语言和编程语言集成的场景。

## [](#performance)性能

LLaMA-Pro 在各种基准测试中展示了先进的性能。它在处理多样化任务方面优于 LLaMA 系列中的现有模型，展示了其作为智能语言代理的能力。

### [](#overall-performance-on-languages-math-and-code-tasks)语言、数学和代码任务的总体性能

| 模型 | ARC | Hellaswag | MMLU | TruthfulQA | Winogrande | GSM8K | GSM8K-PoT | 人工评估 | MBPP | 平均 |
| :-: | :-: | :-: | :-: | :-: | :-: | :-: | :-: | :-: | :-: | :-: |
| LLAMA PRO (8B) | 54.10 | 77.94 | 47.88 | 39.04 | 73.95 | 17.89 | 25.42 | 28.66 | 33.20 | 44.2 |
| LLaMA2-7B | 53.07 | 78.59 | 46.87 | 38.76 | 74.03 | 14.48 | 17.68 | 13.05 | 20.09 | 39.62 |
| CodeLLaMA-7B | 39.93 | 60.80 | 31.12 | 37.82 | 64.01 | 5.16 | 25.20 | 33.50 | 41.40 | 37.66 |
| LLAMA PRO-INSTRUCT | 52.30 | 76.88 | 52.57 | 48.80 | 72.53 | 43.59 | 55.61 | 44.51 | 37.88 | 53.8 |

### [](#performance-on-gpt4-evaluation)在 GPT4 评估上的表现

| Model | MT Bench |
| :-: | :-: |
| 羊驼-13B | 4.53 |
| CodeLLaMA-7B-Instruct | 5.71 |
| 维库娜-7B | 6.17 |
| LLaMA2-7B-Chat | 6.27 |
| LLAMA PRO-INSTRUCT | 6.32 |

## [](#limitations)局限性

虽然 LLaMA-Pro 解决了系列先前模型的一些局限性，但仍可能遇到特定于高度专业领域或任务的挑战。

## [](#ethical-considerations)道德考虑

用户应该意识到模型中潜在的偏见，并负责任地使用它，考虑其对各种应用的影响。
