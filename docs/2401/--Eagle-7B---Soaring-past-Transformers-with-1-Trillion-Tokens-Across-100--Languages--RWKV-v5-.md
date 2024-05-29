<!--yml
category: 未分类
date: 2024-05-27 15:19:48
-->

# 🦅 Eagle 7B : Soaring past Transformers with 1 Trillion Tokens Across 100+ Languages (RWKV-v5)

> 来源：[https://blog.rwkv.com/p/eagle-7b-soaring-past-transformers](https://blog.rwkv.com/p/eagle-7b-soaring-past-transformers)

An eagle, flying past a transformer-looking robot

Eagle 7B is a 7.52B parameter model that:

We are releasing RWKV-v5 Eagle 7B, [licensed as Apache 2.0 license, under the Linux Foundation](https://blog.rwkv.com/p/rwkv-joins-the-linux-foundation-as), and can be used personally or commercially without restrictions

We performed multi-lingual performance across the following benchmarks: [xLAMBDA](https://github.com/EleutherAI/lm-evaluation-harness?tab=readme-ov-file#advanced-usage-tips), [xStoryCloze](https://huggingface.co/datasets/Muennighoff/xstory_cloze), [xWinograd](https://huggingface.co/datasets/Muennighoff/xwinograd), [xCopa](https://huggingface.co/datasets/xcopa)

Across a total of 23 languages

Most of these benchmarks cover common sense reasoning, in their respective languages. And show a huge overall jump in multi-lingual performance for RWKV v4-to-v5 architecture. And the v2 world dataset.

It should also be noted, that there is a lack of multi-lingual benchmarks, as the above covers approximately the top 23 languages.

This makes it hard to evaluate model language performance directly over the remaining 75+ languages, over the total 100+ trained languages. A shortcoming we hope to improve in future models.

English performance was measured across 12 separate benchmarks, across commonsense reasoning, and world knowledge

Once again we see a huge overall jump from RWKV v4-to-v5 architecture. And the v2 world dataset.

Where v4 previously lost out to MPT-7b, the top model in the 1T token tier.

v5 begins trading blows in benchmarks, in some cases even coming on top in certain benchmarks ( LAMBADA, StoryCloze16, WinoGrande, HeadQA_en, Sciq ) over Falcon, or even llama2.

In addition, v5 performance starts to fall in line with the expected transformer performance level, with its given approximate token training count.

With Mistral-7B maintaining its lead with its rumored 2~7 Trillion token training.

We expect to narrow the gap, as we train an additional 1T token, to cross the llama2 line and hopefully reach the mistral line.

Alternatively, as a base model, which is lightly tuned (really small instruct set mixed in), we are eager to see how the various community and instruct-tuned variants

* * *

A notable observation was that our checkpoints near the 300 Billion token point, show similar performance to pythia-6.9b

This is consistent with previous pile-based experiments on our RWKV-v4 architecture, that linear transformers like RWKV scale similarly in performance levels to transformers, with the same token count training.

If so, it does repeat the question. If the exact architecture, matter less than the data for the model eval performance?

CUDA computational cost, for RWKV-based architecture vs transformer models - that quadratic-vs-linear really scales!

If true, perhaps we should seek more efficient and scalable architecture, to increase accessibility, drive the cost of AI downwards for everyone, and [lessen the impact on our environment.](https://blog.rwkv.com/p/the-worlds-greenest-ai-model-rwkvs)

* * *

A common feedback we receive for the RWKV multi-lingual approach is

And for most parts, we agree on both points.

But we have no plans on changing this, as we are building AI for the world - which is not just an English world.

[In 2023, only 17% of the world's population speaks English](https://preply.com/en/blog/english-language-statistics/#:~:text=Current%20research%20suggests%20that%20the,widely%20spoken%20language%20in%202022%3F)
[( 1.3 billion people )](https://preply.com/en/blog/english-language-statistics/#:~:text=Current%20research%20suggests%20that%20the,widely%20spoken%20language%20in%202022%3F)

World Map showing the distribution of regions and people who are fluent in English (source: [Wikipedia](https://en.wikipedia.org/wiki/List_of_countries_by_English-speaking_population))

However, by ensuring support for the top 25 languages in the world and beyond, we can cover approximately [4 billion people, or 50% of the world](https://en.wikipedia.org/wiki/List_of_languages_by_number_of_native_speakers#Top_languages_by_population)

Flawed map, highlighting where the eagle language model will support entirely or partially - the goal is to be able paint the whole map green with confidence

This aligns well with the team’s common goal, of getting AI to support everyone, not just by allowing it to run cheaply and affordably even on lower-end hardware. But by supporting their language.

Over time, we intend to grow the multi-lingual dataset, to support a wider variety of languages, and to slowly grow that coverage to 100% of the world - to ensure no language gets left behind.

A major example of this in our community is the [Indonesian-NLP discord group](https://discord.gg/dy9YWXjV), which finetunes an Indonesian language model from the RWKV line of base models.

Allowing them to build strong language-specific models - on a cheap affordable basis (ie. single node), without needing to do half a million dollars of pre-training.

* * *

This release marks the release of the strongest linear transformer (in terms of eval benchmarks) to date.

While it may not have succeeded in passing LLaMA2 and Mistral. It provides strong evidence of the following

*   The RWKV-v5 model architecture scales similarly to transformer performance with a similar token count

*   You can achieve a near LLaMA2-like level of performance, with a substantially lower inference cost

*   While supporting multi-lingual levels of performance

We plan to follow by pushing further ahead with

*   [Feb 2024] An updated RWKV v5: Eagle paper, where we will go deeper in-depth on the architecture changes since v4, and the model benchmarks and evals

*   [Feb 2024] A further 1T token in training (2T total), for direct comparisons with the LLaMA2 7B model

*   [Mar 2024] An MoE model based on the v5 Eagle 2T model

*   [Mar 2024] RWKV-v6: “Finch” 1.5B, 3B world models

> Disclaimer: All dates are approximate, and is heavily subjected to compute avaliability from our sponsors/provider

Find more about the RWKV Project at

* * *

We are grateful and would like to thank the following key groups:

Along with the various developers, working on the growing collection of [RWKV-related projects](https://wiki.rwkv.com).