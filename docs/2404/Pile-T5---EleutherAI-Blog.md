<!--yml

类别：未分类

日期：2024-05-27 13:13:01

-->

# Pile-T5 | EleutherAI 博客

> 来源：[https://blog.eleuther.ai/pile-t5/](https://blog.eleuther.ai/pile-t5/)

T5 模型（Raffel 等，2019年）在 NLP 社区中被广泛使用。其基础模型已经从 Hugging Face 下载了数百万次，毫无疑问，这些模型是社区的最爱。然而，T5 的分词器省略了重要的与代码相关的标记，随后发布的预训练数据集经过了更高质量的过滤和更多领域的多样化。在本文中，我们介绍了一个新版本的 T5，旨在解决这些弱点：**Pile-T5**，在 Pile（Gao 等，2020年）上进行了训练，并使用了 LLaMA 分词器（Touvron 等，2023年）。

## 模型描述[#](#model-description)

我们的替代版本用 Pile 替换了预训练数据集，并使用 LLaMA 分词器替换了原始 T5 分词器。Pile-T5 在总共 2 万步或 2 万亿令牌的情况下进行了训练 - 是原始 T5 模型训练量的两倍。我们使用原始跨度破坏方法进行训练，并观察到在适用于用户的下游任务微调中的改进。我们发现，即使在令牌匹配的设置中，我们的模型也明显优于最广泛使用的 T5 模型（称为 T5-v1.1）。特别是在代码任务上，Pile-T5 的表现要好得多。我们发布的模型使用与原始 T5 相同的超参数，利用 [T5x](https://github.com/google-research/t5x)。我们在 [这里](https://github.com/EleutherAI/improved-t5) 发布我们的实验脚本。

这些模型可以从 EleutherAI 的 [Hugging Face 页面](https://huggingface.co/collections/EleutherAI/pile-t5-65a76a0d0022dd270b385a66) 访问。与原始 T5 的一个显著区别是使用了来自 [umT5](https://huggingface.co/docs/transformers/model_doc/umt5)（Chung, Constant, Garcia 等，2023年）的变压器实现，因为 T5x 中采用了可伸缩的实现。受 Pythia（Biderman 和 Schoelkopf 等，2023年）启发，我们发布了每 10,000 步一个的 [中间检查点](https://huggingface.co/collections/EleutherAI/pile-t5-65a76a0d0022dd270b385a66)，旨在赋予希望研究我们模型演变的研究人员力量。这些模型在其对应的 Hugging Face 页面的 `main` 分支包含 200 万步版本，部分训练的检查点可以在其他分支找到。此外，我们还发布了这些检查点的 T5x 版本 [这里](https://huggingface.co/collections/EleutherAI/pile-t5-t5x-checkpoints-660aaab3e8c24412c5f69a6a)。

## 超越 1 万亿令牌[#](#going-beyond-1-trillion-tokens)

Pile-T5 模型在 SuperGLUE、CodeXGLUE、以及 MMLU 和 Bigbench Hard 上进行了评估。Pile-T5 模型与 [T5v1.1](https://github.com/google-research/text-to-text-transfer-transformer/blob/main/released_checkpoints.md) 进行了比较，两者均在相同数量的 tokens 上进行了微调。我们还将 Pile-T5 模型与 Flan-T5 模型在 MMLU 和 BBH 上进行了宽泛比较。所有评估都是使用 [LM-Evaluation Harness](https://github.com/EleutherAI/lm-evaluation-harness)（Gao et al, 2023）来报告模型在这里提出的基准测试中的性能。我们发布了 [Base](https://huggingface.co/collections/lintang/pile-t5-base-finetuned-models-65d307353ecda975d828ebb8)、[Large](https://huggingface.co/collections/lintang/pile-t5-large-finetuned-models-65d307ec0545ab7c1153b804)、[XL](https://huggingface.co/collections/lintang/pile-t5-xl-finetuned-models-65d30bbcf5a15aa421099b4e) 和 [XXL](https://huggingface.co/collections/lintang/pile-t5-xxl-finetuned-models-65a76bbc4908b2676c6b8a94) 的微调检查点。

### 在 SuperGLUE 上的表现[#](#performance-on-superglue)

为了评估在 SuperGLUE 上的性能，我们使用批量大小为 128 对 Pile-T5（包括万亿令牌版本和最终的两万亿令牌版本）和 T5v1.1 模型进行了263K步的微调，与原始 T5 论文保持一致。除了 Large 模型外，我们观察到所有模型都有显著的性能提升。请注意，Pile-T5 (1T) 已经优于 T5-v1.1，并且通过进一步的训练进一步提高了性能。

| 大小 | 变体 | 平均 | boolq | cb |  | copa | multirc |  | record |  | rte | wic | wsc |
| --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
|  |  |  | 准确率 | F1 分数 | 准确率 | 准确率 | F1 分数 | 精确匹配 | 精确匹配 | F1 分数 | 准确率 | 准确率 | 准确率 |
| Base | T5-v1.1 | 71.33 | 79.36 | 83.63 | 87.5 | 63 | 73.45 | 33.26 | 69.7 | 68.75 | 78.34 | 65.83 | **75.96** |
|  | Pile-T5 (1T) | 74.85 | 81.46 | 93.69 | **94.64** | 65 | **77.75** | **40.50** | 76.97 | **76.49** | 80.86 | **67.39** | 74.03 |
|  | **Pile-T5** | **76.13** | **82.45** | **96.07** | **94.64** | **72** | **77.74** | 39.56 | **77.64** | **76.88** | **83.03** | **67.24** | 73.08 |
| Large | **T5-v1.1** | **81.11** | **85.96** | **93.21** | **96.43** | **82** | 81.71 | 48.37 | 82.18 | 81.71 | **85.92** | **71.47** | **81.73** |
|  | Pile-T5 (1T) | 79.18 | 83.70 | 91.85 | 94.64 | 79 | 82.36 | 47.85 | 82.72 | 82.14 | 83.03 | 65.2 | **81.73** |
|  | Pile-T5 | 79.67 | 85.71 | 88.96 | 94.64 | 74 | **82.60** | **50.47** | **84.1** | **83.70** | 85.19 | 68.49 | **81.73** |
| XL | T5-v1.1 | 81.76 | 86.79 | 81.18 | 91.07 | 84 | 84.03 | 52.89 | 83.92 | 83.5 | 90.25 | 73.04 | 81.73 |
|  | Pile-T5 (1T) | 86.09 | 89.76 | 90.6 | 94.64 | **96** | 88.17 | 63.90 | 91.58 | 91.36 | **93.50** | 72.73 | 86.54 |
|  | **Pile-T5** | **89.00** | **90.4** | **93.1** | **96.43** | **96** | **88.63** | **65.16** | **92.21** | **91.96** | 92.78 | **75.24** | **96.15** |
| XXL | T5-v1.1 | 82.43 | 88.29 | 93.61 | 94.64 | 86 | 75.22 | 51.00 | 84.67 | 84.55 | 89.17 | 72.41 | 81.73 |
|  | Pile-T5 (1T) | 87.11 | 90.46 | 94.3 | 96.43 | 93 | 80.81 | 56.77 | 91.36 | 91.18 | 92.42 | 70.38 | 95.19 |
|  | **Pile-T5** | **90.08** | **90.98** | **98.68** | **98.21** | **95** | **89.28** | **67.68** | **93.04** | **92.7** | **93.5** | **75.24** | **96.15** |

### CodeXGLUE性能[#](#performance-on-codexglue)

作为我们的主要目标之一是提高模型理解代码的能力，我们在CodeXGLUE的Code-to-Text子任务上进行了评估（Su等人，2021）。所有模型在每种编程语言变体上进行了10个时期的微调，使用与[原始存储库](https://github.com/microsoft/CodeXGLUE/tree/main/Code-Text/code-to-text)相同的方法。

| Size | Version | Average | Python | PHP | Go | Java | JavaScript | Ruby |
| --- | --- | --- | --- | --- | --- | --- | --- | --- |
| Base | T5-v1.1 | 14.34 | 15.55 | 21.72 | 14.71 | 14.89 | 9.25 | 9.90 |
|  | Pile-T5 (1T) | 15.90 | 17.20 | 22.90 | **16.75** | 16.24 | 11.23 | 11.10 |
|  | **Pile-T5** | **16.37** | **17.78** | **23.12** | **16.70** | **16.68** | **11.89** | **12.06** |
| Large | T5-v1.1 | 11.53 | 12.18 | 14.17 | 12.37 | 12.30 | 8.85 | 9.32 |
|  | Pile-T5 (1T) | 15.74 | 17.09 | 22.80 | **17.16** | 16.33 | 10.75 | 10.31 |
|  | **Pile-T5** | **16.28** | **17.72** | **22.95** | 17.07 | **16.41** | **12.05** | **11.45** |
| XL | T5-v1.1 | 16.17 | 17.36 | 21.91 | 16.69 | 17.74 | 11.08 | 12.25 |
|  | Pile-T5 (1T) | 18.01 | 18.61 | 23.75 | 19.04 | 18.43 | 14.27 | 13.93 |
|  | **Pile-T5** | **18.68** | **19.25** | **24.37** | **19.42** | **19.15** | **15.1** | **14.81** |
| XXL | T5-v1.1 | 17.67 | 17.89 | 23.21 | 18.54 | 19.17 | 13.85 | 13.33 |
|  | Pile-T5 (1T) | 18.55 | 19.53 | 24.11 | 19.27 | 18.52 | **15.11** | 14.75 |
|  | **Pile-T5** | **18.72** | **19.27** | **24.49** | **19.60** | **18.96** | **15.10** | **14.92** |

由于Pile包含基于代码的数据和LLaMA标记器包含在代码中经常使用的字符，我们观察到性能显著提高。请注意，尽管Pile-T5-Large在一般情况下表现不如T5-v1.1，但在这些编程基准测试中，它的表现明显优于T5-v1.1-base！相比之下，Pile-T5-Large的表现与Pile-T5-base相似。

## 使用Flan指导调优[#](#using-flan-instruction-tuning)

我们继续通过在Flan上对Pile-T5模型进行微调（Chung, Hou, Longpre等人，2022），使用相同的训练超参数，并在MMLU（Hendrycks等人，2021）和BigBench Hard（Suzgun等人，2022）上进行评估。

与Flan-T5模型相比，我们发现Pile-T5在数量上略有不足，但意义重大。在与作者跟进后，我们了解到用于生成Flan-T5的微调数据并未全部公开发布，这可能解释了性能差异。

为了进行更公正的比较，我们还使用了与Pile-T5模型相同的过程和数据对T5-v1.1检查点进行了微调。我们特别使用了Pile-T5的2万亿令牌版本，以便与T5-v1.1的比较反映出训练数据规模的增加以及数据和分词器的变化。

### Held-In[#](#performance-on-held-in)上的性能

我们观察到在保留任务上表现竞争力（这些任务包含在Flan指令调整数据集中），大型变体的性能有所下降，类似于在SuperGLUE中观察到的行为。

| 大小 | 版本 | 平均 | ANLI R1 | ANLI R2 | ANLI R3 | Arc Easy | Arc Challenge | BoolQ | RTE |
| --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
| 基础 | T5-v1.1 | **46.50** | **39.90** | 34.96 | 37.33 | **38.12** | 28.23 | 70.26 | **76.73** |
|  | Pile-T5 | 46.37 | 39.32 | **35.28** | **37.53** | 36.61 | **30.67** | **71.87** | 73.28 |
| 大型 | T5-v1.1 | **54.90** | **52.46** | **39.67** | **42.53** | **50.60** | **39.99** | **78.56** | **80.50** |
|  | Pile-T5 | 36.97 | 33.00 | 33.03 | 32.98 | 29.27 | 21.21 | 56.36 | 52.95 |
| XL | T5-v1.1 | 56.40 | 53.82 | 40.22 | 41.01 | 56.31 | 39.08 | 80.66 | **83.71** |
|  | Pile-T5 | **64.41** | **64.36** | **48.02** | **49.18** | **66.56** | **58.28** | **85.16** | 79.30 |
| XXL | T5-v1.1 | **69.99** | **71.63** | 55.81 | **57.41** | **75.56** | **62.30** | 86.53 | 80.71 |
|  | Pile-T5 | 69.21 | 71.16 | **55.92** | 55.19 | 70.85 | 59.82 | **87.55** | **83.96** |

### MMLU[#](#performance-on-mmlu)上的性能

评估模型时使用了两种不同版本的提示：原始提示（Hendrycks等人，2021年）和Flan提示（Chung，Hou，Longpre等人，2022年）。

MMLU提示：

```
The following are multiple choice questions (with answers) about abstract algebra.

Find the degree for the given field extension Q(sqrt(2), sqrt(3), sqrt(18)) over Q.
A. 0
B. 4
C. 2
D. 6
Answer: 
```

Flan提示：

```
The following are multiple choice questions (with answers) about abstract algebra.

Q: Find the degree for the given field extension Q(sqrt(2), sqrt(3), sqrt(18)) over Q.
(A) 0 (B) 4 (C) 2 (D) 6
A: 
```

当使用Pile-T5时，我们观察到性能提升。对于MMLU，最高对数似然和生成生成都用于评估。我们观察到对数似然评估主要受益于零次提示；贪婪生成经常在输出良好结构的响应方面遇到困难，包括生成完整答案而非被严格评估者拒绝的单个字母。

通过使用五次提示来改进贪婪生成的性能，这为模型提供了正确响应格式的示例。需要注意的是，性能可能因提示格式而显著变化。平均来看，Pile-T5在v1.1上有所改进，并与Flan-T5变体竞争。

| 大小 | 变体 | 平均 | 最高对数似然 |  |  |  | 贪婪生成 |  |  |  |
| --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
|  |  |  | 原始提示 |  | Flan提示 |  | 原始提示 |  | Flan提示 |  |
|  |  |  | 0-样本 | 5-样本 | 0-样本 | 5-样本 | 0-样本 | 5-样本 | 0-样本 | 5-样本 |
| XL | Flan-T5 | **42.45** | **47.37** | **49.17** | **47.83** | **49.43** | 6.63 | **48.8** | **40.98** | **49.39** |
|  | T5-v1.1 | 36.58 | 38.59 | 39.52 | 40.64 | 39.79 | **25.95** | 38.84 | 29.66 | 39.67 |
|  | Pile-T5 | 40.82 | 46.04 | 48.71 | 47.13 | 48.18 | 3.61 | 48.58 | 35.53 | 48.77 |
| XXL | Flan-T5 | 46.94 | **51.47** | **54.28** | **53.31** | **53.85** | 2.69 | **53.93** | **52.15** | **53.85** |
|  | T5-v1.1 | 45.76 | 51.03 | 51.15 | 46.72 | 50.77 | 31.00 | 50.72 | 33.90 | 50.78 |
|  | Pile-T5 | **48.27** | 50.88 | 53.35 | 52.22 | 53.06 | **35.8** | 53.13 | 33.85 | **53.84** |

### BigBench Hard (BBH)的表现[#](#performance-on-bigbench-hard-bbh)

Pile-T5在BBH的零样本和少样本设置上表现明显优于T5v1.1，并且与Flan-T5竞争力强。

| 大小 | 变体 | 贪婪生成 |  |
| --- | --- | --- | --- |
|  |  | 零样本 | 少样本 |
| XL | Flan-T5 | 24.71 | 40.36 |
|  | T5-v1.1 | 28.67 | 33.06 |
|  | Pile-T5 | **29.98** | **41.49** |
| XXL | Flan-T5 | **43.06** | 44.72 |
|  | T5-v1.1 | 35.14 | 39.84 |
|  | Pile-T5 | 41.61 | **46.71** |

## 结论[#](#conclusion)

我们观察到在微调基准上的改进，例如SuperGLUE、CodeXGLUE、MMLU和BBH。Pile-T5在超过T5v1.1上表现出色，尽管Pile-T5在Flan混合微调上仍然被Flan-T5超越。我们得出结论，Pile-T5非常适合未来的多任务微调和其他受益于编码器-解码器架构的任务。由于Pile-T5 Large在SuperGLUE和Flan Held-In等基准测试中表现不佳，我们认为在训练过程中可能存在错误，并建议在使用时谨慎。最后，我们希望发布中间检查点对研究社区在可解释性和其他努力方面有所帮助。

## 致谢[#](#acknowledgments)

我们感谢Stability AI提供的计算资源来训练这些模型，以及TRC计划提供的计算资源来微调部分模型。感谢Stella Biderman和@philpax调整博客文章。

## 引用[#](#citation)

```
@misc{2024PileT5,
  author  = {Lintang Sutawika and Aran Komatsuzaki and Colin Raffel},
  title   = {Pile-T5},
  year    = {2024},
  url     = {https://blog.eleuther.ai/pile-t5/},
  note    = {Blog post},
} 
```

## 参考文献[#](#references)

1.  斯特拉·比德曼, 海莉·舒尔科普夫, 昆汀·安东尼, 赫比·布拉德利, 凯尔·奥布莱恩, 埃里克·哈拉汉, 穆罕默德·阿夫拉·汗, 等人。*Pythia: 用于分析大型语言模型在训练和扩展过程中的套件*. arXiv [Cs.CL], 2023\. arXiv. [http://arxiv.org/abs/2304.01373](http://arxiv.org/abs/2304.01373).

1.  钟贤元, 诺亚·康斯坦, 阿维尔·加西亚, 亚当·罗伯茨, 易·泰, 沙兰·纳兰, 和奥尔汉·菲拉特。*UniMax: 更公平更有效的大规模多语言预训练语言采样*. arXiv [Cs.CL], 2023\. arXiv. [http://arxiv.org/abs/2304.09151](http://arxiv.org/abs/2304.09151).

1.  Chung, Hyung Won, Le Hou, Shayne Longpre, Barret Zoph, Yi Tay, William Fedus, Yunxuan Li, 等人。*Scaling Instruction-Finetuned Language Models*. arXiv [Cs.LG], 2022\. arXiv. [http://arxiv.org/abs/2210.11416](http://arxiv.org/abs/2210.11416).

1.  Gao, Leo, Stella Biderman, Sid Black, Laurence Golding, Travis Hoppe, Charles Foster, Jason Phang, 等人。‘The Pile: An 800GB Dataset of Diverse Text for Language Modeling’. arXiv [Cs.CL], 2020\. arXiv. [http://arxiv.org/abs/2101.00027](http://arxiv.org/abs/2101.00027).

1.  Gao, Leo, Jonathan Tow, Baber Abbasi, Stella Biderman, Sid Black, Anthony DiPofi, Charles Foster, 等人。‘A Framework for Few-Shot Language Model Evaluation’. Zenodo, 12 2023\. [https://doi.org/10.5281/zenodo.10256836](https://doi.org/10.5281/zenodo.10256836).

1.  Hendrycks, Dan, Collin Burns, Steven Basart, Andy Zou, Mantas Mazeika, Dawn Song, 以及 Jacob Steinhardt。‘Measuring Massive Multitask Language Understanding’. arXiv [Cs.CY], 2021\. arXiv. [http://arxiv.org/abs/2009.03300](http://arxiv.org/abs/2009.03300).

1.  Longpre, Shayne, Le Hou, Tu Vu, Albert Webson, Hyung Won Chung, Yi Tay, Denny Zhou, 等人。*The Flan Collection: Designing Data and Methods for Effective Instruction Tuning*. arXiv [Cs.AI], 2023\. arXiv. [http://arxiv.org/abs/2301.13688](http://arxiv.org/abs/2301.13688).

1.  Lu, Shuai, Daya Guo, Shuo Ren, Junjie Huang, Alexey Svyatkovskiy, Ambrosio Blanco, Colin Clement, 等人。‘CodeXGLUE: A Machine Learning Benchmark Dataset for Code Understanding and Generation’. arXiv [Cs.SE], 2021\. arXiv. [http://arxiv.org/abs/2102.04664](http://arxiv.org/abs/2102.04664).

1.  Suzgun, Mirac, Nathan Scales, Nathanael Schärli, Sebastian Gehrmann, Yi Tay, Hyung Won Chung, Aakanksha Chowdhery, 等人。‘Challenging BIG-Bench Tasks and Whether Chain-of-Thought Can Solve Them’. arXiv [Cs.CL], 2022\. arXiv. [http://arxiv.org/abs/2210.09261](http://arxiv.org/abs/2210.09261).

1.  Touvron, Hugo, Thibaut Lavril, Gautier Izacard, Xavier Martinet, Marie-Anne Lachaux, Timothée Lacroix, Baptiste Rozière, 等人。‘LLaMA: Open and Efficient Foundation Language Models’. arXiv [Cs.CL], 2023\. arXiv. [http://arxiv.org/abs/2302.13971](http://arxiv.org/abs/2302.13971).
