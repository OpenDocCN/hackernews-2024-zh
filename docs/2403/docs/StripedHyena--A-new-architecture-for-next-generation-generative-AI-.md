<!--yml
category: 未分类
date: 2024-05-27 14:33:01
-->

# StripedHyena: A new architecture for next-generation generative AI?

> 来源：[https://the-decoder.com/stripedhyena-a-new-architecture-for-next-generation-generative-ai/](https://the-decoder.com/stripedhyena-a-new-architecture-for-next-generation-generative-ai/)

**GPT-4 and other models rely on transformers. With StripedHyena, researchers present an alternative to the widely used architecture.**

With StripedHyena, the Together AI team presents a family of language models with 7 billion parameters. What makes it special: StripedHyena uses a new set of AI architectures that aim to improve training and inference performance compared to the widely used transformer architecture, used for example in [GPT-4](https://the-decoder.com/open-ai-gpt-4-announcement/).

The release includes StripedHyena-Hessian-7B (SH 7B), a base model, and StripedHyena-Nous-7B (SH-N 7B), a chat model. These models are designed to be faster, more memory efficient, and capable of processing very long contexts of up to 128,000 tokens. Researchers from HazyResearch, hessian.AI, Nous Research, MILA, HuggingFace, and the German Research Centre for Artificial Intelligence (DFKI) were involved.

## StripedHyena: an efficient alternative to transformers

According to Together AI, StripedHyena is the first alternative model that can compete with the best open-source transformers. The base model achieves comparable performance to [Llama-2](https://the-decoder.com/how-to-get-started-with-metas-llama-2-guide/), Yi, and [Mistral 7B](https://the-decoder.com/new-open-source-llm-mistral-7b-outperforms-larger-meta-llama-models/) on OpenLLM leaderboard tasks and outperforms them on long context summarization.

Ad

THE DECODER Newsletter

The most important AI news straight to your inbox.

✓ Weekly

✓ Free

✓ Cancel at any time

Ad

THE DECODER Newsletter

The most important AI news straight to your inbox.

✓ Weekly

✓ Free

✓ Cancel at any time

The core component of the StripedHyena models is a state-space model (SSM) layer. Traditionally, [SSMs](https://kevinkotze.github.io/ts-4-state-space/) have been used to model complex sequences and time series data. They are particularly useful for tasks where temporal dependencies need to be modeled. In the last two years, however, researchers have developed better and better ways to use SSMs for sequence models for language and other domains. The reason: they require less computing power.

The result: StripedHyena is more than 30 percent, 50 percent, and 100 percent faster than conventional transformers in the end-to-end training of sequences of 32,000 tokens, 64,000 tokens, and 128,000 tokens.

The main goal of the StripedHyena models is to push the boundaries of architectural design beyond transformers. In the future, the researchers plan to investigate larger models with longer contexts, multimodal support, further performance optimizations, and the integration of StripedHyena into retrieval pipelines to take full advantage of the longer context.