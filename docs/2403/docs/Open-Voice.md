<!--yml
category: 未分类
date: 2024-05-29 12:46:57
-->

# Open Voice

> 来源：[https://research.myshell.ai/open-voice](https://research.myshell.ai/open-voice)

# OpenVoice: Versatile Instant Voice Cloning

We introduce OpenVoice, a versatile instant voice cloning approach that requires only a short audio clip from the reference speaker to replicate their voice and generate speech in multiple languages. OpenVoice enables granular control over voice styles, including emotion, accent, rhythm, pauses, and intonation, in addition to replicating the tone color of the reference speaker. OpenVoice also achieves zero-shot cross-lingual voice cloning for languages not included in the massive-speaker training set. OpenVoice is also computationally efficient, costing tens of times less than commercially available APIs that offer even inferior performance. The technical report and source code can be found at [https://arxiv.org/pdf/2312.01479.pdf](https://arxiv.org/pdf/2312.01479.pdf) and [https://github.com/myshell-ai/OpenVoice](https://github.com/myshell-ai/OpenVoice)

<https://cdn.myshell.ai/video/opensource/tts_introduce.mp4>

## Accurate Tone Color Cloning

OpenVoice can accurately clone the reference tone color and generate speech in multiple languages and accents.

## Flexible Voice Style Control

OpenVoice enables granular control over voice styles, such as emotion and accent, as well as other style parameters including rhythm, pauses, and intonation. Here we demonstrate the control over emotion and accent of the generated voice.

Generated - Indian Accent

Generated - British Accent

Generated - Australian Accent

## Zero-shot Cross-lingual Voice Cloning

The reference voice and the generated voice can be in any languages outside the massive-speaker multi-lingual dataset. We use “U” to denote the unseen languages in the following examples.

Generated – Mixed Lingual (U)

## Comparison with State-of-the-Arts