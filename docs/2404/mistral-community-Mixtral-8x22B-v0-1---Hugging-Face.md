<!--yml

category: 未分类

date: 2024-05-27 13:05:06

-->

# mistral-community/Mixtral-8x22B-v0.1 · Hugging Face

> 来源：[https://huggingface.co/mistral-community/Mixtral-8x22B-v0.1](https://huggingface.co/mistral-community/Mixtral-8x22B-v0.1)

# [](#mixtral-8x22b)Mixtral-8x22B

> MistralAI 已经将权重上传到他们的组织中，分别是 [mistralai/Mixtral-8x22B-v0.1](https://huggingface.co/mistralai/Mixtral-8x22B-v0.1) 和 [mistralai/Mixtral-8x22B-Instruct-v0.1](https://huggingface.co/mistralai/Mixtral-8x22B-Instruct-v0.1) 。
> 
> 感谢 [@v2ray](https://huggingface.co/v2ray) 将检查点转换并上传为 `transformers` 兼容格式。请关注他们！

使用脚本 [这里](https://huggingface.co/v2ray/Mixtral-8x22B-v0.1/blob/main/convert.py) 将其转换为 HuggingFace Transformers 格式。

Mixtral-8x22B 大语言模型 (LLM) 是一个预训练的生成稀疏专家混合体。

## [](#run-the-model)运行模型

```
from transformers import AutoModelForCausalLM, AutoTokenizer

model_id = "mistral-community/Mixtral-8x22B-v0.1"
tokenizer = AutoTokenizer.from_pretrained(model_id)

model = AutoModelForCausalLM.from_pretrained(model_id)

text = "Hello my name is"
inputs = tokenizer(text, return_tensors="pt")

outputs = model.generate(**inputs, max_new_tokens=20)
print(tokenizer.decode(outputs[0], skip_special_tokens=True)) 
```

默认情况下，transformers 将以完整精度加载模型。因此，您可能有兴趣通过我们在 HF 生态系统中提供的优化进一步减少内存需求来运行模型：

### [](#in-half-precision)使用半精度

仅支持 `float16` 精度在 GPU 设备上运行

<details><summary>展开查看</summary>

```
+ import torch
from transformers import AutoModelForCausalLM, AutoTokenizer

model_id = "mistral-community/Mixtral-8x22B-v0.1"
tokenizer = AutoTokenizer.from_pretrained(model_id)

+ model = AutoModelForCausalLM.from_pretrained(model_id, torch_dtype=torch.float16).to(0)

text = "Hello my name is"
+ inputs = tokenizer(text, return_tensors="pt").to(0)

outputs = model.generate(**inputs, max_new_tokens=20)
print(tokenizer.decode(outputs[0], skip_special_tokens=True)) 
```</details>

### [](#lower-precision-using-8-bit--4-bit-using-bitsandbytes)使用 `bitsandbytes` 进行 8 位和 4 位的低精度

<details><summary>展开查看</summary>

```
+ import torch
from transformers import AutoModelForCausalLM, AutoTokenizer

model_id = "mistral-community/Mixtral-8x22B-v0.1"
tokenizer = AutoTokenizer.from_pretrained(model_id)

+ model = AutoModelForCausalLM.from_pretrained(model_id, load_in_4bit=True)

text = "Hello my name is"
+ inputs = tokenizer(text, return_tensors="pt").to(0)

outputs = model.generate(**inputs, max_new_tokens=20)
print(tokenizer.decode(outputs[0], skip_special_tokens=True)) 
```</details>

### [](#load-the-model-with-flash-attention-2)加载具有 Flash Attention 2 的模型

<details><summary>展开查看</summary>

```
+ import torch
from transformers import AutoModelForCausalLM, AutoTokenizer

model_id = "mistral-community/Mixtral-8x22B-v0.1"
tokenizer = AutoTokenizer.from_pretrained(model_id)

+ model = AutoModelForCausalLM.from_pretrained(model_id, use_flash_attention_2=True)

text = "Hello my name is"
+ inputs = tokenizer(text, return_tensors="pt").to(0)

outputs = model.generate(**inputs, max_new_tokens=20)
print(tokenizer.decode(outputs[0], skip_special_tokens=True)) 
```</details>

## [](#notice)注意

Mixtral-8x22B-v0.1 是一个预训练的基础模型，因此不具备任何调节机制。

# [](#the-mistral-ai-team)Mistral AI 团队

Albert Jiang, Alexandre Sablayrolles, Alexis Tacnet, Antoine Roux, Arthur Mensch, Audrey Herblin-Stoop, Baptiste Bout, Baudouin de Monicault,Blanche Savary, Bam4d, Caroline Feldman, Devendra Singh Chaplot, Diego de las Casas, Eleonore Arcelin, Emma Bou Hanna, Etienne Metzger, Gianna Lengyel, Guillaume Bour, Guillaume Lample, Harizo Rajaona, Jean-Malo Delignon, Jia Li, Justus Murke, Louis Martin, Louis Ternon, Lucile Saulnier, Lélio Renard Lavaud, Margaret Jennings, Marie Pellat, Marie Torelli, Marie-Anne Lachaux, Nicolas Schuhl, Patrick von Platen, Pierre Stock, Sandeep Subramanian, Sophia Yang, Szymon Antoniak, Teven Le Scao, Thibaut Lavril, Timothée Lacroix, Théophile Gervet, Thomas Wang, Valera Nemychnikova, William El Sayed, William Marshall.

# [](#open-llm-leaderboard-evaluation-results)Open LLM 排行榜评估结果

详细结果可以在 [这里](https://huggingface.co/datasets/open-llm-leaderboard/details_mistral-community__Mixtral-8x22B-v0.1) 找到

| 指标 | 值 |
| --- | --: |
| 平均值 | 74.46 |
| AI2 推理挑战 (25-Shot) | 70.48 |
| HellaSwag (10-Shot) | 88.73 |
| MMLU (5-Shot) | 77.81 |
| TruthfulQA (0-shot) | 51.08 |
| Winogrande (5-shot) | 84.53 |
| GSM8k (5-shot) | 74.15 |
