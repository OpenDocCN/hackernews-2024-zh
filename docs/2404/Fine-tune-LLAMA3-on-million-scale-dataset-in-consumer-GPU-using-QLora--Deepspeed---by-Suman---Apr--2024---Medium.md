<!--yml

category: 未分类

日期：2024-05-27 13:32:39

-->

# 在消费级GPU上使用QLora，Deepspeed对百万规模数据集进行LLAMA3微调 | 作者：苏曼 | 2024年4月 | Medium

> 来源：[https://medium.com/@sumandas0/fine-tune-llama3-on-million-scale-dataset-in-consumer-gpu-using-qlora-deepspeed-3ae8ad75299a](https://medium.com/@sumandas0/fine-tune-llama3-on-million-scale-dataset-in-consumer-gpu-using-qlora-deepspeed-3ae8ad75299a)

# 在消费级GPU上使用QLora，Deepspeed对百万规模数据集进行LLAMA3微调

# Highlights,

**模型**：LLAMA-8b-instruct

**数据集**：Openhermes-2.5（700k训练，300k测试）

**GPU**：4 RTX 4090，24GB

# 一点关于我的背景，

我是一名全职软件工程师2，在我们平台团队的核心。在我稀少的空闲时间里，我探索机器学习世界的各个方面，对表格数据、自然语言处理和声音感兴趣。我在这里分享的都是从互联网各处收集汇总而来的碎片。我有训练小型NLP模型的经验，并在Kaggle竞赛中使用DeBERTa v3提交了解决方案，得分足以进入前50%，但我从未尝试过与大型语言模型合作。这是我的第一次尝试，请告诉我是否有任何疏忽。是的，这是我的第一篇博客文章。写这篇文章肯定会对我有所帮助，希望对任何读者也有用。

# LLama

谁不知道这种长颈骆驼般的生物从诞生开始就在革新人工智能领域。开玩笑的话，全OSS动力的LLM发行的羊驼引爆了这场看起来在不久的将来不会停止的革命。

要深入了解羊驼和技术，请查看此[文章|LinkedIn](https://www.linkedin.com/posts/ujamil_llama-explained-kv-cache-rotary-positional-activity-7100620274642866176-XaKO/)，这是我在互联网上找到的最简化的技术解释之一。他们在他们的架构中实施了一些很酷的东西，如分组多查询注意力、KV缓存、旋转位置嵌入（RoPE）。这些不在本文的讨论范围之内。他们继续发布LLama的版本，最新版本几天前发布。这一次将大量数据压缩到几GB参数中。

[Meta揭示Llama 3-10关于先进LLM的关键事实（forbes.com）](https://www.forbes.com/sites/janakirammsv/2024/04/19/meta-unveils-llama-310-key-facts-about-the-advanced-llm/)

# Deepspeed

> *DeepSpeed是一个深度学习优化库，使分布式训练和推断变得简单、高效和有效。*
> 
> [*https://github.com/microsoft/DeepSpeed*](https://github.com/microsoft/DeepSpeed)

我将使用从 [vast.ai](http://vast.ai/) 租来的四块 RTX 4090 GPU 来训练这个模型，因此我们需要采取一些步骤来跨多个 GPU 进行模型训练。与单个 GPU 训练相比，多 GPU 训练是一项复杂的任务。为什么？当我们在单个 GPU 上训练时，优化器状态、参数和梯度都驻留在一个系统中，这有助于在一个 GPU 上迭代模型。

现在，如果我们再添加一个 GPU，会有两个系统来训练模型，每个系统都有自己的状态（优化器状态、参数和梯度）。经过一个 epoch 或几个步骤后，我们希望获得一个单一的结果。现在想象两个系统并行训练两个数据批次；它们需要通信其状态，并以最小的数据损失汇聚结果。有多种方式可以利用多个 GPU：我们可以复制参数、梯度和优化器状态到所有 GPU，或者我们可以仅分片优化器状态，或者优化器状态和梯度。DeepSpeed 有助于在多个 GPU 上分布负载而没有任何问题。而 Huggingface 的 accelerate 包让我们做到这一点就像小菜一碟。

我们将使用阶段 3，它将分片所有参数、梯度和优化器状态，从而让我们在内存需求较少的情况下进行训练。

更多细节请参阅他们的博客，[ZeRO & DeepSpeed：新系统优化使训练模型的参数超过 1000 亿 — 微软研究](https://www.microsoft.com/en-us/research/blog/zero-deepspeed-new-system-optimizations-enable-training-models-with-over-100-billion-parameters/)

# QLoRA

在我写关于 QLoRA 的内容之前，请看看这篇博客以获取更多技术背景 [什么是 QLoRA？| QLoRA — Weights & Biases (wandb.ai)](https://wandb.ai/sauravmaheshkar/QLoRA/reports/What-is-QLoRA---Vmlldzo2MTI2OTc5)，基本上 70B/8B 模型在尺寸上非常大，这意味着当你进行微调时，你将无法完全使用任何 GPU 在正常人的预算内，因此我们尝试使用非常低的资源进行微调，并且 LoRA 帮助我们只需训练低秩参数并与原始权重合并，然后出现 QLoRA，它通过将预训练的 LLM 量化为 4 位精度来进一步减少内存消耗，量化本身就是一个独立的主题，因此不再深入讨论。

还请看看这篇文章 [LoRA 微调和超参数解释（用简单英语）| Entry Point AI](https://www.entrypointai.com/blog/lora-fine-tuning/)

# 让我们开始对 LLamA 3 进行微调

我们将对 llama3 instruct 模型进行微调 [meta-llama/Meta-Llama-3–8B-Instruct · Hugging Face](https://huggingface.co/meta-llama/Meta-Llama-3-8B-Instruct)，使用 teknium 提供的 [openhermes](https://huggingface.co/teknium/OpenHermes-2.5-Mistral-7B) 数据集。

# 数据准备

Meta 有他们自己的聊天格式，所以试图遵循他们提供的格式，并阅读他们在他们的 llama3 仓库中的编码算法。

**加载数据集**

```
from datasets import load_dataset

dataset = load_dataset("teknium/OpenHermes-2.5")
```

**我从 [**llama3 repo**](https://github.com/meta-llama/llama3/blob/af6eedf7042fb51d00b2b26d8ef1ceaab73e1670/llama/tokenizer.py#L202) 取得编码实用程序的灵感,**

```
def _return_header(message)-> str:
    role = message["from"]
    header = ""
    if role == "system":
        header = "system"
    elif role == "gpt":
        header = "assistant"
    elif role == "human":
        header = "user"
    return header

def encode_header(message):
    text = ''
    text = text + "<|start_header_id|>"
    header = _return_header(message)
    text = text + header
    text = text + "<|end_header_id|>"
    text = text + "\n\n"
    return text

def encode_message(message)->str:
    text = encode_header(message)
    text = text + message["value"].strip()
    text = text + "<|eot_id|>"
    return text

def encode_dialog_prompt(dialog):
    text = ''
    text = text + "<|begin_of_text|>"
    for message in dialog:
        text = text + encode_message(message)
    return text
```

```
ds = dataset.map(lambda x: {"content":encode_dialog_prompt(x['conversations'])}, num_proc=10)
```

删除冗余列并将其拆分为训练和验证集

```
ds = ds.remove_columns(['custom_instruction', 'topic', 'model_name', 'model', 'skip_prompt_formatting', 'category', 'conversations', 'views', 'language', 'id', 'title', 'idx', 'hash', 'avatarUrl', 'system_prompt', 'source'])
train_test_split = ds["train"].train_test_split(test_size=0.3)
```

**并将其推送到hub,**

```
train_test_split.push_to_hub("sumandas/openhermes-2.5-llama3")
```

结果数据集，[sumandas/openhermes-2.5-llama3 · Datasets at Hugging Face](https://huggingface.co/datasets/sumandas/openhermes-2.5-llama3)，示例文本

```
<|begin_of_text|><|start_header_id|>system<|end_header_id|> You are an AI assistant. Provide a detailed answer so user don’t need to search outside to understand the answer.<|eot_id|><|start_header_id|>user<|end_header_id|> Instructions: Given a sentence, generate what should be the most likely next statement. The next statement should be reasonable and logically correct. Input: The screen is full of white bubbles and words, while a pair of hands plays the piano. The bubbles and words disappear and it Output:<|eot_id|><|start_header_id|>assistant<|end_header_id|> Output: becomes apparent that the hands are creating a visual representation of the music being played, captivating the audience with this unique sensory experience.<|eot_id|>
```

# 现在是训练LLama3的时候了

所有资源已经在互联网上可用，我只是根据我的设置和需求对其进行了精细调整，

**先决条件,**

1.  安装cuda开发工具包 `conda install cuda` 或访问 [developer.nvidia.com/cuda-downloads?target_os=Linux](https://developer.nvidia.com/cuda-downloads?target_os=Linux)

1.  安装deepspeed

1.  安装闪烁-关注 *pip install flash-attn — no-build-isolation*

1.  安装这些库，我使用 [uv](https://github.com/astral-sh/uv) 进行更快的依赖项解决，

```
git+https://github.com/huggingface/transformers
git+https://github.com/huggingface/accelerate
git+https://github.com/huggingface/peft
git+https://github.com/huggingface/trl
huggingface-hub
bitsandbytes
evaluate
datasets
einops
wandb
tiktoken
xformers
sentencepiece
deepspeed
torch==2.2.2
```

**训练代码**

这是瑞士军刀培训代码，您可以根据自己的方便模式进行多模式训练，发现此代码库 [pacman100/LLM-Workshop: LLM Workshop by Sourab Mangrulkar (github.com)](https://github.com/pacman100/LLM-Workshop)，

> `training.py` 文件是我们将使用加速器和适当配置启动的文件，这里只是放置 training.py 要点，[https://gist.github.com/sumandas0/0483db8514ea43e45cc5e5f5525914ab](https://gist.github.com/sumandas0/0483db8514ea43e45cc5e5f5525914ab)

此训练代码使用了来自huggingface的SFTTrainer，更多细节 [Supervised Fine-tuning Trainer (huggingface.co)](https://huggingface.co/docs/trl/en/sft_trainer)

您可以用此做多种事情，可以使用loftq、unsloth、FFT、普通lora进行训练，但我只会使用带有Deepspeed ZerO阶段3的QloRa。

**首先定义加速配置以使用deepspeed**

> 注意，如果增加GPU更新号，请在 *num_processes* 中更新

现在让我们运行加速命令来开始训练，

```
accelerate launch --config_file "deepspeed_config.yaml"  train.py \
--seed 100 \
--model_name_or_path "meta-llama/Meta-Llama-3-8B-Instruct" \
--dataset_name "sumandas/openhermes-2.5-llama3" \
--chat_template_format "none" \
--add_special_tokens False \
--append_concat_token False \
--splits "train,test" \
--max_seq_len 2048 \
--num_train_epochs 1 \
--logging_steps 5 \
--log_level "info" \
--logging_strategy "steps" \
--evaluation_strategy "epoch" \
--save_strategy "steps" \
--push_to_hub \
--hub_private_repo True \
--report_to "wandb" \
--hub_strategy "every_save" \
--bf16 True \
--packing True \
--learning_rate 1e-4 \
--lr_scheduler_type "cosine" \
--weight_decay 1e-4 \
--warmup_ratio 0.0 \
--max_grad_norm 1.0 \
--output_dir "llama3-openhermes-2.5" \
--per_device_train_batch_size 4\
--per_device_eval_batch_size 4\
--gradient_accumulation_steps 2 \
--gradient_checkpointing True \
--use_reentrant True \
--dataset_text_field "content" \
--use_flash_attn True \
--use_peft_lora True \
--lora_r 8 \
--lora_alpha 16 \
--lora_dropout 0.1 \
--lora_target_modules "all-linear" \
--use_4bit_quantization True \
--use_nested_quant True \
--bnb_4bit_compute_dtype "bfloat16" \
--bnb_4bit_quant_storage_dtype "bfloat16"
```

**注意,**

1.  首先设置环境变量 HF_HUB_ENABLE_HF_TRANSFER=1

1.  输出目录也将是在huggingface中创建的仓库，所有检查点将默认每500步创建一个

1.  我将聊天模板格式设置为 `none`，因为我已经按自己的方式格式化了它们，如果您有其他格式，请使用例如chatml，zephyr

1.  `lora_target_modules` 被设置为全线性，这是QLoRa特有的，他们发表了一篇论文显示，微调所有线性层使我们的结果可与全面微调相媲美。

1.  要设置LoRa的超参数，请查看此精彩博客 [LoRA Fine-tuning & Hyperparameters Explained (in Plain English) | Entry Point AI](https://www.entrypointai.com/blog/lora-fine-tuning/)

1.  设置WANDB_API_KEY=<key>，如果要报告给wandb，则删除 `report_to='wandb'`

这应该就是全部了，您的训练应该在全力运行中，请查看GPU利用率。

# 观察

仅运行了1个epoch的精细调整，大约花费了15小时。损失曲线

fig: 训练损失 [train/loss (24/04/25 02:44:11) | huggingface — Weights & Biases (wandb.ai)](https://wandb.ai/sumandas0/huggingface/reports/train-loss-24-04-25-02-44-11---Vmlldzo3Njg1NzIw?accessToken=hinzctjy4lbm48zwjoamnmhxs5r56zp8l88iqpss2jb0xo2w2bu049jkiqd59btj)

**WandB 摘要**

```
{
  "train/learning_rate": 0.00004551803455482833,
  "eval/steps_per_second": 0.893,
  "_wandb.runtime": 51487,
  "_runtime": 51480.36651659012,
  "_timestamp": 1713698971.6200776,
  "train/epoch": 1.0571428571428572,
  "train/grad_norm": 0.14189070214353952,
  "train/global_step": 8325,
  "eval/samples_per_second": 7.141,
  "_step": 1665,
  "eval/loss": 0.963840126991272,
  "train/loss": 0.9674,
  "eval/runtime": 7532.9797
}
```

# 最后的步骤，

在微调之后，您将获得一个小的适配器模型而不是完整的模型，您现在可以立即开始使用，我们需要将适配器添加到原始的 meta llama3 权重中，

载入 PEFT 适配器模型，

```
from peft import PeftModel
from transformers import AutoModelForCausalLM

base_model = AutoModelForCausalLM.from_pretrained("meta-llama/Meta-Llama-3-8B-Instruct",torch_dtype=torch.bfloat16, device_map="auto")
adapter_model = PeftModel.from_pretrained(base_model, "sumandas/llama3-openhermes-2.5")
adapter_model.merge_and_unload()
```

现在将适配器模型保存到 hf 中，

```
adapter_model.push_to_hub("sumandas/llama3-openhermes-2.5")
```

# 推断

```
import torch
import transformers
from transformers import pipeline

pipe = pipeline("text-generation", model="sumandas/llama3-openhermes-2.5",model_kwargs={"torch_dtype": torch.bfloat16},
    device="cuda")

generate_kwargs = {
    "do_sample": True,
    "temperature": 0.7,
    "max_new_tokens": 35,
} 
```

```
pipe("<|begin_of_text|><|start_header_id|>user<|end_header_id|> Describe the food of Thailand<|eot_id|><|start_header_id|>assistant<|end_header_id|>",max_length=2048)
```

回复

```
Thai cuisine is a blend of bold flavors and aromatic ingredients that create a harmonious balance of sweet, sour, salty, bitter, and spicy taste. The food in Thailand varies between different regions, but some common ingredients found throughout include rice, noodles, vegetables, herbs, and spices. Rice is a staple of Thai cuisine and is served with almost every meal, whether it is stir-fried, boiled, or used to make congee or fried rice. Noodles, made from rice or wheat, are also very popular and can be found served with various broths, sauces, and toppings in many Thai dishes. Thai food is known for its use of fresh herbs and spices such as lemongrass, galangal, coriander, basil, kaffir lime leaves, ginger, and garlic, which give the dishes their distinct flavors. Thai chilies are also widely used to add varying levels of heat to the food. Some popular Thai dishes include Tom Yum soup (hot and sour soup with lemongrass, kaffir lime leaves, and chilies), Pad Thai (stir-fried rice noodles with vegetables, peanuts, and a tangy sauce), and green curry (a spicy curry made with green chilies, coconut milk, and Thai basil). Many Thai dishes are also accompanied by a variety of sauces and condiments, including fish sauce, soy sauce, chili paste, and tamarind sauce. Fresh fruits like mango, papaya, and pineapple are also commonly enjoyed as a sweet ending to a meal. Overall, Thai food is a vibrant and flavorful cuisine that combines traditional ingredients and cooking techniques with a balance of flavors that tantalize the taste buds.<|eot_id|>
```

如果我的模型和数据集有任何价值，请给予一些爱 :)

[sumandas/openhermes-2.5-llama3 · 在 Hugging Face 的数据集](https://huggingface.co/datasets/sumandas/openhermes-2.5-llama3)

[sumandas/llama3-openhermes-2.5 · Hugging Face](https://huggingface.co/sumandas/llama3-openhermes-2.5)
