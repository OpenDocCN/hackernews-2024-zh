<!--yml

类别：未分类

日期：2024-05-27 13:28:05

-->

# microsoft/Phi-3-mini-128k-instruct · Hugging Face

> 来源：[https://huggingface.co/microsoft/Phi-3-mini-128k-instruct](https://huggingface.co/microsoft/Phi-3-mini-128k-instruct)

## [](#model-summary)模型摘要

Phi-3-Mini-128K-Instruct 是一个38亿参数、轻量级、最先进的开放模型，使用了Phi-3数据集进行训练。该数据集包括合成数据和经过滤的公开可用网站数据，重点是高质量和推理密集属性。该模型属于Phi-3系列，Mini版本有两个变体 [4K](https://huggingface.co/microsoft/Phi-3-mini-4k-instruct) 和 [128K](https://huggingface.co/microsoft/Phi-3-mini-128k-instruct)，其支持的上下文长度（以令牌计）。

在初步训练后，模型经历了一个后训练过程，其中包括监督微调和直接偏好优化，以增强其遵循指令和遵守安全措施的能力。在针对测试常识、语言理解、数学、编码、长期上下文和逻辑推理的基准进行评估时，Phi-3 Mini-128K-Instruct 在少于130亿参数的模型中表现出了强大和最先进的性能。资源和技术文档：

## [](#intended-uses)预期用途

**主要用例**

该模型旨在用于英语的商业和研究用途。该模型为需要以下应用程序提供了使用：

1.  内存/计算受限环境

1.  延迟边界场景

1.  强大的推理能力（尤其是代码、数学和逻辑）

我们的模型旨在加速语言和多模型的研究，用作生成AI功能的构建模块。

**用例注意事项**

我们的模型并非专门设计或评估所有下游用途。开发人员在选择用例时应考虑语言模型的常见限制，并在使用特定下游用例之前评估和减轻其准确性、安全性和公平性，尤其是对于高风险场景。开发人员应注意并遵守适用的法律或法规（包括隐私、贸易合规法律等），这些法律或法规与他们的用例相关。

本模型卡中包含的任何内容都不应被解释为或视为限制或修改模型发布的许可证。

## [](#how-to-use)使用方法

Phi-3 Mini-128K-Instruct 已集成到 `transformers` 的开发版本（4.41.0.dev0）。在通过 `pip` 发布官方版本之前，请确保您正在执行以下操作之一：

+   加载模型时，请确保将`trust_remote_code=True`作为`from_pretrained()`函数的参数传递。

+   更新您的本地 `transformers` 到开发版本：`pip uninstall -y transformers && pip install git+https://github.com/huggingface/transformers`。前述命令是从源代码克隆和安装的替代方法。

您可以通过以下命令验证当前的 `transformers` 版本：`pip list | grep transformers`。

### [](#tokenizer)分词器

Phi-3 Mini-128K-Instruct 支持多达 `32064` 个标记的词汇量。[分词器文件](https://huggingface.co/microsoft/Phi-3-mini-128k-instruct/blob/main/added_tokens.json)已经提供了可用于下游微调的占位符标记，但它们也可以扩展到模型的词汇量。

### [](#chat-format)对话格式

鉴于训练数据的特性，Phi-3 Mini-128K-Instruct 模型最适合采用以下对话格式的提示。您可以将提示提供为以下通用模板的问题：

```
<|user|>\nQuestion<|end|>\n<|assistant|> 
```

例如：

```
<|user|>
How to explain Internet for a medieval knight?<|end|>
<|assistant|> 
```

在模型生成文本之后 `<|assistant|>`。在少量样本提示情况下，提示可以格式化如下：

```
<|user|>
I am going to Paris, what should I see?<|end|>
<|assistant|>
Paris, the capital of France, is known for its stunning architecture, art museums, historical landmarks, and romantic atmosphere. Here are some of the top attractions to see in Paris:\n\n1\. The Eiffel Tower: The iconic Eiffel Tower is one of the most recognizable landmarks in the world and offers breathtaking views of the city.\n2\. The Louvre Museum: The Louvre is one of the world's largest and most famous museums, housing an impressive collection of art and artifacts, including the Mona Lisa.\n3\. Notre-Dame Cathedral: This beautiful cathedral is one of the most famous landmarks in Paris and is known for its Gothic architecture and stunning stained glass windows.\n\nThese are just a few of the many attractions that Paris has to offer. With so much to see and do, it's no wonder that Paris is one of the most popular tourist destinations in the world."<|end|>
<|user|>
What is so great about #1?<|end|>
<|assistant|> 
```

### [](#sample-inference-code)示例推理代码

这些代码片段展示了如何快速启动在 GPU 上运行模型：

```
import torch
from transformers import AutoModelForCausalLM, AutoTokenizer, pipeline

torch.random.manual_seed(0)

model = AutoModelForCausalLM.from_pretrained(
    "microsoft/Phi-3-mini-128k-instruct", 
    device_map="cuda", 
    torch_dtype="auto", 
    trust_remote_code=True, 
)
tokenizer = AutoTokenizer.from_pretrained("microsoft/Phi-3-mini-128k-instruct")

messages = [
    {"role": "user", "content": "Can you provide ways to eat combinations of bananas and dragonfruits?"},
    {"role": "assistant", "content": "Sure! Here are some ways to eat bananas and dragonfruits together: 1\. Banana and dragonfruit smoothie: Blend bananas and dragonfruits together with some milk and honey. 2\. Banana and dragonfruit salad: Mix sliced bananas and dragonfruits together with some lemon juice and honey."},
    {"role": "user", "content": "What about solving an 2x + 3 = 7 equation?"},
]

pipe = pipeline(
    "text-generation",
    model=model,
    tokenizer=tokenizer,
)

generation_args = {
    "max_new_tokens": 500,
    "return_full_text": False,
    "temperature": 0.0,
    "do_sample": False,
}

output = pipe(messages, **generation_args)
print(output[0]['generated_text']) 
```

*一些应用/框架可能不包括对话开始的 BOS 标记 (`<s>`)。请确保它被包含，因为它能提供更可靠的结果。*

## [](#responsible-ai-considerations)负责任的 AI 考虑

与其他语言模型一样，Phi 系列模型可能表现出不公平、不可靠或令人反感的方式。一些需注意的限制行为包括：

+   服务质量：Phi 系列模型主要是在英文文本上进行训练的。除英语外的其他语言将会表现得更差。训练数据中较少代表性的英语语言变体可能比标准美式英语表现更差。

+   对于伤害的表达与刻板印象的延续：这些模型可能会过度或不足地代表某些群体，擦除某些群体的代表性，或者加强贬低或负面刻板印象。尽管安全后训练，由于不同群体的代表性水平或负面刻板印象的例子在训练数据中的普遍性和社会偏见的反映，这些限制可能仍然存在。

+   不恰当或冒犯性内容：这些模型可能生成其他类型的不恰当或冒犯性内容，这可能使其在没有针对具体使用情境的额外缓解措施的情况下不适合部署于敏感环境中。

+   信息可靠性：语言模型可以生成荒谬的内容或捏造可能听起来合理但不准确或过时的内容。

+   代码的使用范围有限：大部分Phi-3训练数据基于Python，并使用常见的包，例如"typing, math, random, collections, datetime, itertools"。如果模型生成利用其他包或其他语言的Python脚本，我们强烈建议用户手动验证所有API使用情况。

开发者应当应用负责任的人工智能最佳实践，并确保特定用例符合相关法律和法规（例如隐私、贸易等）。需要重点考虑的领域包括：

+   分配：模型可能不适用于可能对法律地位或资源分配或生活机会分配（例如住房、就业、信贷等）产生重大影响的场景，除非进一步评估和额外去偏见技术。

+   高风险场景：开发者应评估在高风险场景中使用模型的适宜性，其中不公平、不可靠或冒犯性输出可能会造成极大的成本或导致危害。这包括在敏感或专业领域提供建议，其中准确性和可靠性至关重要（例如法律或健康建议）。根据部署上下文，应在应用程序级别实施额外的安全保障措施。

+   误信息：模型可能生成不准确的信息。开发者应遵循透明度最佳实践，并告知最终用户他们正在与AI系统交互。在应用程序级别，开发者可以构建反馈机制和管道，将响应根据特定用例的上下文信息实时化，这是一种称为检索增强生成（RAG）的技术。

+   生成有害内容：开发者应评估输出的上下文，并使用适合其用例的安全分类器或自定义解决方案。

+   滥用：可能存在其他形式的滥用，如欺诈、垃圾邮件或恶意软件生产，并且开发者应确保其应用程序不违反适用的法律和法规。

## [](#training)训练

### [](#model)模型

+   架构：Phi-3 Mini-128K-Instruct有38亿参数，是一个密集的仅解码Transformer模型。模型经过监督微调（SFT）和直接偏好优化（DPO），以确保与人类偏好和安全准则的一致性。

+   输入：文本。最适合使用聊天格式进行提示。

+   上下文长度：128K标记

+   GPU：512个H100-80G

+   训练时间：7天

+   训练数据：3.3万亿标记

+   输出：对输入生成的文本

+   日期：我们的模型在2024年2月至4月之间进行了训练

+   状态：这是一个静态模型，使用截至2023年10月的离线数据集进行训练。随着模型的改进，可能会发布调优版本的未来模型。

### [](#datasets)数据集

我们的训练数据包括多种来源，总计3.3万亿标记，是高质量教育数据和代码的结合；

1.  经过严格过滤的公开文档，选择高质量的教育数据和代码；

1.  为了教授数学、编程、常识推理、世界通识（科学、日常活动、心理理论等），新创建了类似“教科书”的合成数据；

1.  高质量聊天格式的监督数据，涵盖各种主题，反映人类在不同方面（如指示-跟随、真实性、诚实和乐于助人）上的偏好。

### [](#fine-tuning)微调

提供了关于多GPU监督微调（SFT）的基本示例，包括TRL和Accelerate模块，可在[这里](https://huggingface.co/microsoft/Phi-3-mini-128k-instruct/resolve/main/sample_finetune.py)找到。

## [](#benchmarks)基准

我们报告Phi-3-Mini-128K-Instruct在衡量模型推理能力（常识推理和逻辑推理）的标准开源基准上的结果。我们与Phi-2、Mistral-7b-v0.1、Mixtral-8x7b、Gemma 7B、Llama-3-8B-Instruct和GPT-3.5进行比较。

所有报告的数字都是使用完全相同的流程生成的，以确保数字是可比较的。由于在评估中选择略有不同，这些数字可能与其他发布的数字有所不同。

正如现在的标准，我们使用少量示例来评估模型，在温度为0。提示和射击次数是微软内部工具的一部分，用于评估语言模型，特别是我们对Phi-3没有进行管道优化。更具体地说，我们不改变提示，选择不同的少量示例，改变提示格式，或进行任何其他形式的模型优化。

每个基准的k-shot示例数量列在括号内。

|  | Phi-3-Mini-128K-In 3.8b | Phi-3-Small 7b（预览） | Phi-3-Medium 14b（预览） | Phi-2 2.7b | Mistral 7b | Gemma 7b | Llama-3-In 8b | Mixtral 8x7b | GPT-3.5版本1106 |
| --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
| MMLU 5-Shot | 68.1 | 75.3 | 78.2 | 56.3 | 61.7 | 63.6 | 66.5 | 68.4 | 71.4 |
| HellaSwag 5-Shot | 74.5 | 78.7 | 83.2 | 53.6 | 58.5 | 49.8 | 71.1 | 70.4 | 78.8 |
| ANLI 7-Shot | 52.8 | 55.0 | 58.7 | 42.5 | 47.1 | 48.7 | 57.3 | 55.2 | 58.1 |
| GSM-8K 0-Shot; CoT | 83.6 | 86.4 | 90.8 | 61.1 | 46.4 | 59.8 | 77.4 | 64.7 | 78.1 |
| MedQA 2-Shot | 55.3 | 58.2 | 69.8 | 40.9 | 49.6 | 50.0 | 60.5 | 62.2 | 63.4 |
| AGIEval 0-Shot | 36.9 | 45.0 | 49.7 | 29.8 | 35.1 | 42.1 | 42.0 | 45.2 | 48.4 |
| TriviaQA 5-Shot | 57.1 | 59.1 | 73.3 | 45.2 | 72.3 | 75.2 | 67.7 | 82.2 | 85.8 |
| Arc-C 10-Shot | 84.0 | 90.7 | 91.9 | 75.9 | 78.6 | 78.3 | 82.8 | 87.3 | 87.4 |
| Arc-E 10-Shot | 95.2 | 97.1 | 98.0 | 88.5 | 90.6 | 91.4 | 93.4 | 95.6 | 96.3 |
| PIQA 5-Shot | 83.6 | 87.8 | 88.2 | 60.2 | 77.7 | 78.1 | 75.7 | 86.0 | 86.6 |
| SociQA 5-Shot | 76.1 | 79.0 | 79.4 | 68.3 | 74.6 | 65.5 | 73.9 | 75.9 | 68.3 |
| BigBench-Hard 0-Shot | 71.5 | 75.0 | 82.5 | 59.4 | 57.3 | 59.6 | 51.5 | 69.7 | 68.32 |
| WinoGrande 5-Shot | 72.5 | 82.5 | 81.2 | 54.7 | 54.2 | 55.6 | 65.0 | 62.0 | 68.8 |
| OpenBookQA 10-Shot | 80.6 | 88.4 | 86.6 | 73.6 | 79.8 | 78.6 | 82.6 | 85.8 | 86.0 |
| BoolQ 0-Shot | 78.7 | 82.9 | 86.5 | -- | 72.2 | 66.0 | 80.9 | 77.6 | 79.1 |
| CommonSenseQA 10-Shot | 78.0 | 80.3 | 82.6 | 69.3 | 72.6 | 76.2 | 79 | 78.1 | 79.6 |
| TruthfulQA 10-Shot | 63.2 | 68.1 | 74.8 | -- | 52.1 | 53.0 | 63.2 | 60.1 | 85.8 |
| HumanEval 0-Shot | 57.9 | 59.1 | 54.7 | 47.0 | 28.0 | 34.1 | 60.4 | 37.8 | 62.2 |
| MBPP 3-Shot | 62.5 | 71.4 | 73.7 | 60.6 | 50.8 | 51.5 | 67.7 | 60.2 | 77.8 |

## [](#software)软件

## [](#hardware)硬件

默认情况下，Phi-3-mini 模型使用闪存注意力，需要特定类型的 GPU 硬件才能运行。我们已在以下 GPU 类型上进行了测试：

+   NVIDIA A100

+   NVIDIA A6000

+   NVIDIA H100

如果您想在以下设备上运行该模型：

+   NVIDIA V100 或早期的 GPU：调用 AutoModelForCausalLM.from_pretrained() 并设置 attn_implementation="eager"

+   GPU、CPU 和移动设备的优化推断：使用 **ONNX** 模型 [128K](https://aka.ms/phi3-mini-128k-instruct-onnx)

## [](#cross-platform-support)跨平台支持

ONNX 运行时生态系统现在跨平台和硬件支持 Phi-3 Mini 模型。你可以在[这里](https://aka.ms/phi3-mini-128k-instruct-onnx)找到优化的 Phi-3 Mini-128K-Instruct ONNX 模型。

优化的 Phi-3 模型也以 ONNX 格式发布，可在 CPU 和 GPU 上使用 ONNX Runtime 在各种设备上运行，包括服务器平台、Windows、Linux 和 Mac 桌面以及移动 CPU，适应每个目标的最佳精度。DirectML 支持让开发人员在 AMD、Intel 和 NVIDIA GPU 上为 Windows 设备带来规模化的硬件加速。

除了 DirectML，ONNX Runtime 还为 Phi-3 在一系列设备上提供跨平台支持，包括 CPU、GPU 和移动设备。

这里是我们添加的一些优化配置：

1.  int4 DML 的 ONNX 模型：通过 AWQ 进行整数 4 位量化

1.  fp16 CUDA 的 ONNX 模型

1.  int4 CUDA 的 ONNX 模型：通过 RTN 进行整数 4 位量化

1.  int4 CPU 和移动设备的 ONNX 模型：通过 RTN 进行整数 4 位量化

## [](#license)许可证

该模型在 [MIT 许可证](https://huggingface.co/microsoft/Phi-3-mini-128k/resolve/main/LICENSE)下许可。

## [](#trademarks)商标

本项目可能包含项目、产品或服务的商标或标识。使用 Microsoft 商标或标识必须遵守并遵循 [Microsoft 商标和品牌指南](https://www.microsoft.com/en-us/legal/intellectualproperty/trademarks)。在此项目的修改版本中使用 Microsoft 商标或标识不得引起混淆或暗示 Microsoft 赞助。任何第三方商标或标识的使用需遵守该第三方的政策。
