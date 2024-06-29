<!--yml

category: 未分类

date: 2024-05-27 13:30:29

-->

# apple/OpenELM · Hugging Face

> 来源：[https://huggingface.co/apple/OpenELM](https://huggingface.co/apple/OpenELM)

# [](#openelm-an-efficient-language-model-family-with-open-training-and-inference-framework)OpenELM：一种具有开放训练和推断框架的高效语言模型系列

*Sachin Mehta, Mohammad Hossein Sekhavat, Qingqing Cao, Maxwell Horton, Yanzi Jin, Chenfan Sun, Iman Mirzadeh, Mahyar Najibi, Dmitry Belenko, Peter Zatloukal, Mohammad Rastegari*

我们引入**OpenELM**，这是一系列**开放** **高效** **语言** **模型**。OpenELM采用逐层缩放策略，在转换器模型的每一层内高效分配参数，从而提高精度。我们使用[CoreNet](https://github.com/apple/corenet)库预训练了OpenELM模型。我们发布了270M、450M、1.1B和3B参数的预训练和调整模型。

我们的预训练数据集包括RefinedWeb、去重的PILE、RedPajama的子集以及Dolma v1.6的子集，总共约1.8万亿个标记。请在使用这些数据集之前查看许可协议和条款。

请参阅以下列表，了解每个模型的详细信息：

```
 from transformers import AutoModelForCausalLM

openelm_270m = AutoModelForCausalLM.from_pretrained("apple/OpenELM-270M", trust_remote_code=True)
openelm_450m = AutoModelForCausalLM.from_pretrained("apple/OpenELM-450M", trust_remote_code=True)
openelm_1b = AutoModelForCausalLM.from_pretrained("apple/OpenELM-1_1B", trust_remote_code=True)
openelm_3b = AutoModelForCausalLM.from_pretrained("apple/OpenELM-3B", trust_remote_code=True)

openelm_270m_instruct = AutoModelForCausalLM.from_pretrained("apple/OpenELM-270M-Instruct", trust_remote_code=True)
openelm_450m_instruct = AutoModelForCausalLM.from_pretrained("apple/OpenELM-450M-Instruct", trust_remote_code=True)
openelm_1b_instruct = AutoModelForCausalLM.from_pretrained("apple/OpenELM-1_1B-Instruct", trust_remote_code=True)
openelm_3b_instruct = AutoModelForCausalLM.from_pretrained("apple/OpenELM-3B-Instruct", trust_remote_code=True) 
```

## [](#usage)使用方法

我们提供了一个示例函数，用于从通过[HuggingFace Hub](https://huggingface.co/docs/hub/)加载的OpenELM模型中生成输出，在`generate_openelm.py`中。

您可以通过运行以下命令来尝试该模型：

```
python generate_openelm.py --model [MODEL_NAME] --hf_access_token [HF_ACCESS_TOKEN] --prompt 'Once upon a time there was' --generate_kwargs repetition_penalty=1.2 
```

请参考[此链接](https://huggingface.co/docs/hub/security-tokens)获取您的Hugging Face访问令牌。

通过`generate_kwargs`参数可以向Hugging Face生成函数传递额外的参数。例如，为了加速推断过程，可以尝试通过传递`prompt_lookup_num_tokens`参数来进行[查找标记推测生成](https://huggingface.co/docs/transformers/generation_strategies)：

```
python generate_openelm.py --model [MODEL_NAME] --hf_access_token [HF_ACCESS_TOKEN] --prompt 'Once upon a time there was' --generate_kwargs repetition_penalty=1.2 prompt_lookup_num_tokens=10 
```

或者，通过将辅助模型传递给`assistant_model`参数，例如，可以尝试按模型进行推测生成：

```
python generate_openelm.py --model [MODEL_NAME] --hf_access_token [HF_ACCESS_TOKEN] --prompt 'Once upon a time there was' --generate_kwargs repetition_penalty=1.2 --assistant_model [SMALLER_MODEL_NAME] 
```

## [](#main-results)主要结果

### [](#zero-shot)零样本

### [](#llm360)LLM360

### [](#openllm-leaderboard)OpenLLM排行榜

| **模型大小** | **ARC-c** | **CrowS-Pairs** | **HellaSwag** | **MMLU** | **PIQA** | **RACE** | **TruthfulQA** | **WinoGrande** | **平均值** |
| --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
| [OpenELM-270M](https://huggingface.co/apple/OpenELM-270M) | 27.65 | **66.79** | 47.15 | 25.72 | 69.75 | 30.91 | **39.24** | **53.83** | 45.13 |
| [OpenELM-270M-Instruct](https://huggingface.co/apple/OpenELM-270M-Instruct) | **32.51** | 66.01 | **51.58** | **26.70** | **70.78** | 33.78 | 38.72 | 53.20 | **46.66** |
| [OpenELM-450M](https://huggingface.co/apple/OpenELM-450M) | 30.20 | **68.63** | 53.86 | **26.01** | 72.31 | 33.11 | 40.18 | 57.22 | 47.69 |
| [OpenELM-450M-Instruct](https://huggingface.co/apple/OpenELM-450M-Instruct) | **33.53** | 67.44 | **59.31** | 25.41 | **72.63** | **36.84** | **40.48** | **58.33** | **49.25** |
| [OpenELM-1_1B](https://huggingface.co/apple/OpenELM-1_1B) | 36.69 | **71.74** | 65.71 | **27.05** | **75.57** | 36.46 | 36.98 | 63.22 | 51.68 |
| [OpenELM-1_1B-Instruct](https://huggingface.co/apple/OpenELM-1_1B-Instruct) | **41.55** | 71.02 | **71.83** | 25.65 | 75.03 | **39.43** | **45.95** | **64.72** | **54.40** |
| [OpenELM-3B](https://huggingface.co/apple/OpenELM-3B) | 42.24 | **73.29** | 73.28 | **26.76** | 78.24 | **38.76** | 34.98 | 67.25 | 54.35 |
| [OpenELM-3B-Instruct](https://huggingface.co/apple/OpenELM-3B-Instruct) | **47.70** | 72.33 | **76.87** | 24.80 | **79.00** | 38.47 | **38.76** | **67.96** | **55.73** |

详见技术报告获取更多结果和比较。

## [](#evaluation)评估

### [](#setup)设置

安装以下依赖项：

```
 harness_repo="public-lm-eval-harness"
git clone https://github.com/EleutherAI/lm-evaluation-harness ${harness_repo}
cd ${harness_repo}

git checkout dc90fec
pip install -e .
cd ..

pip install datasets@git+https://github.com/huggingface/datasets.git@66d6242
pip install tokenizers>=0.15.2 transformers>=4.38.2 sentencepiece>=0.2.0 
```

### [](#evaluate-openelm)评估 OpenELM

```
 hf_model=apple/OpenELM-270M

tokenizer=meta-llama/Llama-2-7b-hf
add_bos_token=True
batch_size=1

mkdir lm_eval_output

shot=0
task=arc_challenge,arc_easy,boolq,hellaswag,piqa,race,winogrande,sciq,truthfulqa_mc2
lm_eval --model hf \
        --model_args pretrained=${hf_model},trust_remote_code=True,add_bos_token=${add_bos_token},tokenizer=${tokenizer} \
        --tasks ${task} \
        --device cuda:0 \
        --num_fewshot ${shot} \
        --output_path ./lm_eval_output/${hf_model//\//_}_${task//,/_}-${shot}shot \
        --batch_size ${batch_size} 2>&1 | tee ./lm_eval_output/eval-${hf_model//\//_}_${task//,/_}-${shot}shot.log

shot=5
task=mmlu,winogrande
lm_eval --model hf \
        --model_args pretrained=${hf_model},trust_remote_code=True,add_bos_token=${add_bos_token},tokenizer=${tokenizer} \
        --tasks ${task} \
        --device cuda:0 \
        --num_fewshot ${shot} \
        --output_path ./lm_eval_output/${hf_model//\//_}_${task//,/_}-${shot}shot \
        --batch_size ${batch_size} 2>&1 | tee ./lm_eval_output/eval-${hf_model//\//_}_${task//,/_}-${shot}shot.log

shot=25
task=arc_challenge,crows_pairs_english
lm_eval --model hf \
        --model_args pretrained=${hf_model},trust_remote_code=True,add_bos_token=${add_bos_token},tokenizer=${tokenizer} \
        --tasks ${task} \
        --device cuda:0 \
        --num_fewshot ${shot} \
        --output_path ./lm_eval_output/${hf_model//\//_}_${task//,/_}-${shot}shot \
        --batch_size ${batch_size} 2>&1 | tee ./lm_eval_output/eval-${hf_model//\//_}_${task//,/_}-${shot}shot.log

shot=10
task=hellaswag
lm_eval --model hf \
        --model_args pretrained=${hf_model},trust_remote_code=True,add_bos_token=${add_bos_token},tokenizer=${tokenizer} \
        --tasks ${task} \
        --device cuda:0 \
        --num_fewshot ${shot} \
        --output_path ./lm_eval_output/${hf_model//\//_}_${task//,/_}-${shot}shot \
        --batch_size ${batch_size} 2>&1 | tee ./lm_eval_output/eval-${hf_model//\//_}_${task//,/_}-${shot}shot.log 
```

## [](#bias-risks-and-limitations)偏见、风险和限制

开放ELM模型的发布旨在通过提供最先进的语言模型，增强和丰富开放研究社区的能力。这些模型是在公开可用的数据集上训练的，无需任何安全保证。因此，这些模型有可能在响应用户提示时产生不准确、有害、偏见或令人反感的输出。因此，对于用户和开发者来说，进行彻底的安全测试并实施适合其特定需求的适当过滤机制至关重要。

## [](#citation)引用

如果您觉得我们的工作有用，请引用：

```
@article{mehtaOpenELMEfficientLanguage2024,
    title = {{OpenELM}: {An} {Efficient} {Language} {Model} {Family} with {Open} {Training} and {Inference} {Framework}},
    shorttitle = {{OpenELM}},
    url = {https://arxiv.org/abs/2404.14619v1},
    language = {en},
    urldate = {2024-04-24},
    journal = {arXiv.org},
    author = {Mehta, Sachin and Sekhavat, Mohammad Hossein and Cao, Qingqing and Horton, Maxwell and Jin, Yanzi and Sun, Chenfan and Mirzadeh, Iman and Najibi, Mahyar and Belenko, Dmitry and Zatloukal, Peter and Rastegari, Mohammad},
    month = apr,
    year = {2024},
}

@inproceedings{mehta2022cvnets, 
     author = {Mehta, Sachin and Abdolhosseini, Farzad and Rastegari, Mohammad}, 
     title = {CVNets: High Performance Library for Computer Vision}, 
     year = {2022}, 
     booktitle = {Proceedings of the 30th ACM International Conference on Multimedia}, 
     series = {MM '22} 
} 
```
