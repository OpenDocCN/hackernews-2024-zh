<!--yml
category: 未分类
date: 2024-05-27 13:30:29
-->

# apple/OpenELM · Hugging Face

> 来源：[https://huggingface.co/apple/OpenELM](https://huggingface.co/apple/OpenELM)

# [](#openelm-an-efficient-language-model-family-with-open-training-and-inference-framework)OpenELM: An Efficient Language Model Family with Open Training and Inference Framework

*Sachin Mehta, Mohammad Hossein Sekhavat, Qingqing Cao, Maxwell Horton, Yanzi Jin, Chenfan Sun, Iman Mirzadeh, Mahyar Najibi, Dmitry Belenko, Peter Zatloukal, Mohammad Rastegari*

We introduce **OpenELM**, a family of **Open** **E**fficient **L**anguage **M**odels. OpenELM uses a layer-wise scaling strategy to efficiently allocate parameters within each layer of the transformer model, leading to enhanced accuracy. We pretrained OpenELM models using the [CoreNet](https://github.com/apple/corenet) library. We release both pretrained and instruction tuned models with 270M, 450M, 1.1B and 3B parameters.

Our pre-training dataset contains RefinedWeb, deduplicated PILE, a subset of RedPajama, and a subset of Dolma v1.6, totaling approximately 1.8 trillion tokens. Please check license agreements and terms of these datasets before using them.

See the list below for the details of each model:

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

## [](#usage)Usage

We have provided an example function to generate output from OpenELM models loaded via [HuggingFace Hub](https://huggingface.co/docs/hub/) in `generate_openelm.py`.

You can try the model by running the following command:

```
python generate_openelm.py --model [MODEL_NAME] --hf_access_token [HF_ACCESS_TOKEN] --prompt 'Once upon a time there was' --generate_kwargs repetition_penalty=1.2 
```

Please refer to [this link](https://huggingface.co/docs/hub/security-tokens) to obtain your hugging face access token.

Additional arguments to the hugging face generate function can be passed via `generate_kwargs`. As an example, to speedup the inference, you can try [lookup token speculative generation](https://huggingface.co/docs/transformers/generation_strategies) by passing the `prompt_lookup_num_tokens` argument as follows:

```
python generate_openelm.py --model [MODEL_NAME] --hf_access_token [HF_ACCESS_TOKEN] --prompt 'Once upon a time there was' --generate_kwargs repetition_penalty=1.2 prompt_lookup_num_tokens=10 
```

Alternatively, try model-wise speculative generation with an [assistive model](https://huggingface.co/blog/assisted-generation) by passing a smaller model through the `assistant_model` argument, for example:

```
python generate_openelm.py --model [MODEL_NAME] --hf_access_token [HF_ACCESS_TOKEN] --prompt 'Once upon a time there was' --generate_kwargs repetition_penalty=1.2 --assistant_model [SMALLER_MODEL_NAME] 
```

## [](#main-results)Main Results

### [](#zero-shot)Zero-Shot

### [](#llm360)LLM360

### [](#openllm-leaderboard)OpenLLM Leaderboard

| **Model Size** | **ARC-c** | **CrowS-Pairs** | **HellaSwag** | **MMLU** | **PIQA** | **RACE** | **TruthfulQA** | **WinoGrande** | **Average** |
| --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
| [OpenELM-270M](https://huggingface.co/apple/OpenELM-270M) | 27.65 | **66.79** | 47.15 | 25.72 | 69.75 | 30.91 | **39.24** | **53.83** | 45.13 |
| [OpenELM-270M-Instruct](https://huggingface.co/apple/OpenELM-270M-Instruct) | **32.51** | 66.01 | **51.58** | **26.70** | **70.78** | 33.78 | 38.72 | 53.20 | **46.66** |
| [OpenELM-450M](https://huggingface.co/apple/OpenELM-450M) | 30.20 | **68.63** | 53.86 | **26.01** | 72.31 | 33.11 | 40.18 | 57.22 | 47.69 |
| [OpenELM-450M-Instruct](https://huggingface.co/apple/OpenELM-450M-Instruct) | **33.53** | 67.44 | **59.31** | 25.41 | **72.63** | **36.84** | **40.48** | **58.33** | **49.25** |
| [OpenELM-1_1B](https://huggingface.co/apple/OpenELM-1_1B) | 36.69 | **71.74** | 65.71 | **27.05** | **75.57** | 36.46 | 36.98 | 63.22 | 51.68 |
| [OpenELM-1_1B-Instruct](https://huggingface.co/apple/OpenELM-1_1B-Instruct) | **41.55** | 71.02 | **71.83** | 25.65 | 75.03 | **39.43** | **45.95** | **64.72** | **54.40** |
| [OpenELM-3B](https://huggingface.co/apple/OpenELM-3B) | 42.24 | **73.29** | 73.28 | **26.76** | 78.24 | **38.76** | 34.98 | 67.25 | 54.35 |
| [OpenELM-3B-Instruct](https://huggingface.co/apple/OpenELM-3B-Instruct) | **47.70** | 72.33 | **76.87** | 24.80 | **79.00** | 38.47 | **38.76** | **67.96** | **55.73** |

See the technical report for more results and comparison.

## [](#evaluation)Evaluation

### [](#setup)Setup

Install the following dependencies:

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

### [](#evaluate-openelm)Evaluate OpenELM

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

## [](#bias-risks-and-limitations)Bias, Risks, and Limitations

The release of OpenELM models aims to empower and enrich the open research community by providing access to state-of-the-art language models. Trained on publicly available datasets, these models are made available without any safety guarantees. Consequently, there exists the possibility of these models producing outputs that are inaccurate, harmful, biased, or objectionable in response to user prompts. Thus, it is imperative for users and developers to undertake thorough safety testing and implement appropriate filtering mechanisms tailored to their specific requirements.

## [](#citation)Citation

If you find our work useful, please cite:

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