<!--yml
category: 未分类
date: 2024-05-27 13:05:06
-->

# mistral-community/Mixtral-8x22B-v0.1 · Hugging Face

> 来源：[https://huggingface.co/mistral-community/Mixtral-8x22B-v0.1](https://huggingface.co/mistral-community/Mixtral-8x22B-v0.1)

# [](#mixtral-8x22b)Mixtral-8x22B

> MistralAI has uploaded weights to their organization at [mistralai/Mixtral-8x22B-v0.1](https://huggingface.co/mistralai/Mixtral-8x22B-v0.1) and [mistralai/Mixtral-8x22B-Instruct-v0.1](https://huggingface.co/mistralai/Mixtral-8x22B-Instruct-v0.1) too.

> Kudos to [@v2ray](https://huggingface.co/v2ray) for converting the checkpoints and uploading them in `transformers` compatible format. Go give them a follow!

Converted to HuggingFace Transformers format using the script [here](https://huggingface.co/v2ray/Mixtral-8x22B-v0.1/blob/main/convert.py).

The Mixtral-8x22B Large Language Model (LLM) is a pretrained generative Sparse Mixture of Experts.

## [](#run-the-model)Run the model

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

By default, transformers will load the model in full precision. Therefore you might be interested to further reduce down the memory requirements to run the model through the optimizations we offer in HF ecosystem:

### [](#in-half-precision)In half-precision

Note `float16` precision only works on GPU devices

<details><summary>Click to expand</summary>

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

### [](#lower-precision-using-8-bit--4-bit-using-bitsandbytes)Lower precision using (8-bit & 4-bit) using `bitsandbytes`

<details><summary>Click to expand</summary>

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

### [](#load-the-model-with-flash-attention-2)Load the model with Flash Attention 2

<details><summary>Click to expand</summary>

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

## [](#notice)Notice

Mixtral-8x22B-v0.1 is a pretrained base model and therefore does not have any moderation mechanisms.

# [](#the-mistral-ai-team)The Mistral AI Team

Albert Jiang, Alexandre Sablayrolles, Alexis Tacnet, Antoine Roux, Arthur Mensch, Audrey Herblin-Stoop, Baptiste Bout, Baudouin de Monicault,Blanche Savary, Bam4d, Caroline Feldman, Devendra Singh Chaplot, Diego de las Casas, Eleonore Arcelin, Emma Bou Hanna, Etienne Metzger, Gianna Lengyel, Guillaume Bour, Guillaume Lample, Harizo Rajaona, Jean-Malo Delignon, Jia Li, Justus Murke, Louis Martin, Louis Ternon, Lucile Saulnier, Lélio Renard Lavaud, Margaret Jennings, Marie Pellat, Marie Torelli, Marie-Anne Lachaux, Nicolas Schuhl, Patrick von Platen, Pierre Stock, Sandeep Subramanian, Sophia Yang, Szymon Antoniak, Teven Le Scao, Thibaut Lavril, Timothée Lacroix, Théophile Gervet, Thomas Wang, Valera Nemychnikova, William El Sayed, William Marshall.

# [](#open-llm-leaderboard-evaluation-results)Open LLM Leaderboard Evaluation Results

Detailed results can be found [here](https://huggingface.co/datasets/open-llm-leaderboard/details_mistral-community__Mixtral-8x22B-v0.1)

| Metric | Value |
| --- | --: |
| Avg. | 74.46 |
| AI2 Reasoning Challenge (25-Shot) | 70.48 |
| HellaSwag (10-Shot) | 88.73 |
| MMLU (5-Shot) | 77.81 |
| TruthfulQA (0-shot) | 51.08 |
| Winogrande (5-shot) | 84.53 |
| GSM8k (5-shot) | 74.15 |