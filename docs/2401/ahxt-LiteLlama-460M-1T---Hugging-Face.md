<!--yml
category: 未分类
date: 2024-05-27 14:34:21
-->

# ahxt/LiteLlama-460M-1T · Hugging Face

> 来源：[https://huggingface.co/ahxt/LiteLlama-460M-1T](https://huggingface.co/ahxt/LiteLlama-460M-1T)

# [](#litellama-reduced-scale-llama)LiteLlama: Reduced-Scale Llama

We present an open-source reproduction of Meta AI's [LLaMa 2](https://ai.meta.com/llama/). However, with significantly reduced model sizes, [LiteLlama-460M-1T](https://huggingface.co/ahxt/LiteLlama-460M-1T) has 460M parameters trained with 1T tokens.

## [](#dataset-and-tokenization)Dataset and Tokenization

We train our models on part of [RedPajama](https://www.together.xyz/blog/redpajama) dataset. We use the [GPT2Tokenizer](https://huggingface.co/docs/transformers/v4.31.0/en/model_doc/gpt2#transformers.GPT2Tokenizer) to tokenize the text.

## [](#training-details)Training Details

The model was trained with ~1T tokens (0.98T). num of tokens = steps*length*batch_size=499679*1024*192=98240888832≈0.98T.

The training curve is at this [WandB project](https://wandb.ai/ahxt/llama2_xs_460M_training_loss/reports/reduced_train_loss-23-09-05-20-25-43---Vmlldzo1MzIwNDUx?accessToken=x2ch3n30jo77p1x8y7q9js4h4d8zpjtz1tzot4xxullyefixp4jwt7au2q37k2q6).

### [](#using-with-huggingface-transformers)Using with HuggingFace Transformers

The experimental checkpoints can be directly loaded by [Transformers](https://huggingface.co/transformers/) library. The following code snippet shows how to load the our experimental model and generate text with it.

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

## [](#evaluation)Evaluation

### [](#we-evaluate-our-models-on-the-mmlu-task)We evaluate our models on the MMLU task.

| Models | #parameters | zero-shot | 5-shot |
| --- | --- | --- | --- |
| llama | 7B | 28.46 | 35.05 |
| openllama | 3B | 24.90 | 26.71 |
| TinyLlama-1.1B-step-50K-105b | 1.1B | 19.00 | 26.53 |
| LiteLlama-460M-1T | 0.46B | 21.13 | 26.39 |

### [](#open-llm-leaderboard-evaluation-results)Open LLM Leaderboard Evaluation Results

Detailed results can be found [here](https://huggingface.co/datasets/open-llm-leaderboard/details_ahxt__llama2_xs_460M_experimental)

| Metric | Value |
| --- | --- |
| Avg. | 26.65 |
| ARC (25-shot) | 24.91 |
| HellaSwag (10-shot) | 38.47 |
| MMLU (5-shot) | 26.17 |
| TruthfulQA (0-shot) | 41.59 |
| Winogrande (5-shot) | 49.88 |
| GSM8K (5-shot) | 0.0 |
| DROP (3-shot) | 5.51 |

## [](#contact)Contact

This model was developed by [Xiaotian Han](https://ahxt.github.io/) from Texas A&M University at the DATA Lab under the supervision of Prof. [Xia "Ben" Hu](https://cs.rice.edu/~xh37/index.html), and the model is released under MIT License.