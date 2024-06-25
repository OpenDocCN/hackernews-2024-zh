<!--yml

分类：未分类

日期：2024-05-27 14:31:20

-->

# CultriX/MistralTrix-v1 · Hugging Face

> 来源：[`huggingface.co/CultriX/MistralTrix-v1`](https://huggingface.co/CultriX/MistralTrix-v1)

# [](#edit)编辑：

总是查看我的空间以获取最新的模型基准结果！

# [](#results)结果：

T: 🟦 模型：CultriX/MistralTrix-v1 📑 平均值：73.39 ARC：72.27 HellaSwag：88.33 MMLU：65.24 TruthfulQA：70.73 Winogrande：80.98 GSM8K：62.77

# [](#editdisclaimer)编辑/免责声明：

目前在 LLM 排行榜上排名第一的 7B LLM，哇！我根本没想到会有这样的结果，而且在 LLM 或者计算机科学方面我完全不是专业的，只是一个喜欢研究和摆弄的家伙。

对于那些想知道我是如何做到这一点的人，答案就是我简单地尝试了自己在这篇精彩文章中概述的技术：[`towardsdatascience.com/fine-tune-a-mistral-7b-model-with-direct-preference-optimization-708042745aac`](https://towardsdatascience.com/fine-tune-a-mistral-7b-model-with-direct-preference-optimization-708042745aac) 因此，基本上所有的功劳都归功于那个写这篇文章的家伙。他提供了我用来训练这个模型的确切 Colab 笔记本，以及一个非常好的 GitHub 页面，我希望他不介意我分享：[`github.com/mlabonne/llm-course/`](https://github.com/mlabonne/llm-course/) 所以非常感谢他分享他的知识，在这个过程中教了我一两件事情！

# [](#gguf)GGUF

我尝试了自己量化模型，再次强调，我对此几乎一无所知，但当我测试它们时，它们似乎运行良好：[`huggingface.co/CultriX/MistralTrix-v1-GGUF`](https://huggingface.co/CultriX/MistralTrix-v1-GGUF)

不过，我要再说一次：“我对这一切完全是个新手，所以如果这些结果不好别感到惊讶。”

你已经被警告了 :)

# [](#description)描述：

（在单个 Colab GPU 上训练，不到几个小时）

MistralTrix-v1 是一个 zyh3826/GML-Mistral-merged-v1 模型，经过使用 Intel 的数据集进行了进一步的直接偏好优化（DPO） fine-tune。它在几个基准测试中超越了原始模型（参见结果）。

它直接受到了 Intel/neural-chat-7b-v3-1 的作者描述的 RLHF 过程的启发，以提高性能。我使用了相同的数据集，并重新格式化了它以应用 ChatML 模板。

训练此模型的代码可在 Google Colab 和 GitHub 上获得。在 Google Colab A-1000 GPU 上，使用 40GB VRAM 的 fine-tuning 大约需要一个小时。

# [](#training-specifications)训练规范

> LoRA 配置 peft_config = LoraConfig( r=16, lora_alpha=16, lora_dropout=0.05, bias="none", task_type="CAUSAL_LM", target_modules=['k_proj', 'gate_proj', 'v_proj', 'up_proj', 'q_proj', 'o_proj', 'down_proj'] )
> 
> 通过以下代码 fine-tune 模型：`AutoModelForCausalLM.from_pretrained( model_name, torch_dtype=torch.float16, load_in_4bit=True )`，`model.config.use_cache = False`
> 
> 参考模型 ref_model = AutoModelForCausalLM.from_pretrained( model_name, torch_dtype=torch.float16, load_in_4bit=True )
> 
> 训练参数 training_args = TrainingArguments( per_device_train_batch_size=4, gradient_accumulation_steps=4, gradient_checkpointing=True, learning_rate=5e-5, lr_scheduler_type="cosine", max_steps=200, save_strategy="no", logging_steps=1, output_dir=new_model, optim="paged_adamw_32bit", warmup_steps=100, bf16=True, report_to="wandb", )
> 
> 创建 DPO 训练器 dpo_trainer = DPOTrainer( model, ref_model, args=training_args, train_dataset=dataset, tokenizer=tokenizer, peft_config=peft_config, beta=0.1, max_prompt_length=1024, max_length=1536, )
