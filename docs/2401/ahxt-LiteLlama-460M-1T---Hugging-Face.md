<!--yml

类别：未分类

日期：2024-05-27 14:34:21

-->

# ahxt/LiteLlama-460M-1T · Hugging Face

> 来源：[https://huggingface.co/ahxt/LiteLlama-460M-1T](https://huggingface.co/ahxt/LiteLlama-460M-1T)

# [](#litellama-reduced-scale-llama)LiteLlama：缩减规模的Llama

我们提供了[Meta AI的LLaMa 2](https://ai.meta.com/llama/)的开源复制。 然而，缩减后的模型尺寸[LiteLlama-460M-1T](https://huggingface.co/ahxt/LiteLlama-460M-1T)具有460M个参数，并使用了1T个标记进行训练。

## [](#dataset-and-tokenization)数据集和标记化

我们在部分[RedPajama](https://www.together.xyz/blog/redpajama)数据集上训练我们的模型。 我们使用[GPT2Tokenizer](https://huggingface.co/docs/transformers/v4.31.0/en/model_doc/gpt2#transformers.GPT2Tokenizer)对文本进行标记化。

## [](#training-details)训练详情

该模型使用了大约1T个标记（0.98T）。 标记数=步数*长度*批量大小=499679*1024*192=98240888832≈0.98T。

训练曲线在此[WandB项目](https://wandb.ai/ahxt/llama2_xs_460M_training_loss/reports/reduced_train_loss-23-09-05-20-25-43---Vmlldzo1MzIwNDUx?accessToken=x2ch3n30jo77p1x8y7q9js4h4d8zpjtz1tzot4xxullyefixp4jwt7au2q37k2q6)上。

### [](#using-with-huggingface-transformers)使用 HuggingFace Transformers

实验性检查点可直接由[Transformers](https://huggingface.co/transformers/)库加载。 以下代码片段显示了如何加载我们的实验性模型并使用它生成文本。

```
import torch
from transformers import AutoTokenizer, AutoModelForCausalLM

model_path = 'ahxt/LiteLlama-460M-1T'

model = AutoModelForCausalLM.from_pretrained(model_path)
tokenizer = AutoTokenizer.from_pretrained(model_path)
model.eval()

prompt = 'Q: What is the largest bird?\nA:'
input_ids = tokenizer(prompt, return_tensors="pt").input_ids
tokens = model.generate(input_ids, max_length=20)
print( tokenizer.decode(tokens[0].tolist(), skip_special_tokens=True) ) 
```

## [](#evaluation)评估

### [](#we-evaluate-our-models-on-the-mmlu-task)我们对MMLU任务上的模型进行评估。

| 模型 | #参数 | 零shot | 5-shot |
| --- | --- | --- | --- |
| llama | 7B | 28.46 | 35.05 |
| openllama | 3B | 24.90 | 26.71 |
| TinyLlama-1.1B-step-50K-105b | 1.1B | 19.00 | 26.53 |
| LiteLlama-460M-1T | 0.46B | 21.13 | 26.39 |

### [](#open-llm-leaderboard-evaluation-results)Open LLM领先评估结果

可在[此处](https://huggingface.co/datasets/open-llm-leaderboard/details_ahxt__llama2_xs_460M_experimental)找到详细结果

| 指标 | 值 |
| --- | --- |
| 平均 | 26.65 |
| ARC（25-shot） | 24.91 |
| HellaSwag（10-shot） | 38.47 |
| MMLU（5-shot） | 26.17 |
| TruthfulQA（0-shot） | 41.59 |
| Winogrande（5-shot） | 49.88 |
| GSM8K（5-shot） | 0.0 |
| DROP（3-shot） | 5.51 |

## [](#contact)联系方式

该模型是由得克萨斯农工大学数据实验室的[Xiaotian Han](https://ahxt.github.io/)在[胡霄（Xia "Ben" Hu）教授](https://cs.rice.edu/~xh37/index.html)的监督下开发的，并且该模型在MIT许可下发布。
