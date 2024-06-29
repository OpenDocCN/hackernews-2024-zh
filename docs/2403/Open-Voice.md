<!--yml

category: 未分类

date: 2024-05-29 12:46:57

-->

# Open Voice

> 来源：[https://research.myshell.ai/open-voice](https://research.myshell.ai/open-voice)

# OpenVoice：多功能即时语音克隆

我们推出了 OpenVoice，一种多才多艺的即时语音克隆方法，只需使用参考讲话者的短音频片段即可复制其语音，并能在多种语言中生成语音。OpenVoice 能够精细控制语音风格，包括情感、口音、节奏、停顿和语调，同时复制参考讲话者的语音色彩。OpenVoice 还能够实现零-shot 跨语言语音克隆，即使是未包含在大规模训练集中的语言。OpenVoice 在计算效率上也非常高效，成本仅为商业可用API的几十分之一，同时性能甚至更好。技术报告和源代码可在 [https://arxiv.org/pdf/2312.01479.pdf](https://arxiv.org/pdf/2312.01479.pdf) 和 [https://github.com/myshell-ai/OpenVoice](https://github.com/myshell-ai/OpenVoice) 找到。

<https://cdn.myshell.ai/video/opensource/tts_introduce.mp4>

## 精准的语音色彩克隆

OpenVoice 能够精确复制参考语音的语音色彩，并能在多种语言和口音中生成语音。

## 灵活的语音风格控制

OpenVoice 能够精细控制语音风格，如情感和口音，以及其他风格参数，包括节奏、停顿和语调。这里我们展示了对生成语音情感和口音的控制。

生成 - 印度口音

生成 - 英式口音

生成 - 澳大利亚口音

## 零-shot 跨语言语音克隆

参考语音和生成语音可以是多语种数据集之外的任何语言。在以下示例中，我们使用“U”表示看不见的语言。

生成 – 混合语言（U）

## 与最先进技术的比较
