<!--yml

category: 未分类

date: 2024-05-27 13:13:00

-->

# WizardLM 2

> 来源：[https://wizardlm.github.io/WizardLM2/](https://wizardlm.github.io/WizardLM2/)

为了全面展示 WizardLM-2 的性能，我们对我们的模型与多种基线进行了人工和自动评估。如下所示的主要实验结果表明，与领先的专有作品相比，WizardLM-2 展示了极具竞争力的性能，并始终优于所有现有的开源模型。更多相关细节和思考将在我们即将发表的论文中呈现。

**人类偏好评估**

我们精心收集了一个包含现实世界指令的复杂和具有挑战性的集合，其中包括人性的主要要求，如写作、编码、数学、推理、代理和多语言。我们对 WizardLM-2 和基线进行了盲目的两两比较。对每个标注者，所有模型的响应都被随机打乱以隐藏它们的来源。我们报告了胜负比率而没有平局：

+   WizardLM-2 8x22B 仅稍逊于 GPT-4-1106-preview，并且比 Command R Plus 和 GPT4-0314 强大得多。

+   WizardLM-2 70B 比 GPT4-0613、Mistral-Large 和 Qwen1.5-72B-Chat 都更优秀。

+   WizardLM-2 7B 与 Qwen1.5-32B-Chat 相当，并超过了 Qwen1.5-14B-Chat 和 Starling-LM-7B-beta。

通过这项人类偏好评估，WizardLM-2 的能力非常接近领先的专有模型，如 GPT-4-1106-preview，并且显著领先于所有其他开源模型。
