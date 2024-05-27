<!--yml
category: 未分类
date: 2024-05-27 12:54:12
-->

# Yi-34B, Llama 2, and common practices in LLM training: a fact check of the New York Times | EleutherAI Blog

> 来源：[https://blog.eleuther.ai/nyt-yi-34b-response/](https://blog.eleuther.ai/nyt-yi-34b-response/)

On February 21 2024, the New York Times published [“China’s Rush to Dominate A.I. Comes With a Twist: It Depends on U.S. Technology.”](https://www.nytimes.com/2024/02/21/technology/china-united-states-artificial-intelligence.html) The authors claim that Yi-34B, a recent large language model by the Chinese startup 01.AI, is fundamentally indebted to Meta’s Llama 2:

> There was just one twist: Some of the technology in 01.AI’s system [came](https://archive.is/o/krRqo/https://huggingface.co/01-ai/Yi-34B/discussions/11) from Llama. Mr. Lee’s start-up then built on Meta’s technology, training its system with new data to make it more powerful.

This assessment is based on a misreading of the [cited Hugging Face issue](https://huggingface.co/01-ai/Yi-34B/discussions/11). While we make no claims about the overall state of US-China AI competition, we want to explain why what 01.AI did is unremarkable and well within the bounds of common machine learning practices.

In short, all modern large language models (LLMs) are made from the same algorithmic building blocks. The architectural differences between Llama 2 and the original 2017 Transformer were not invented by Meta, and are all public owing to open access publishing being the norm in computer science. So, even though Yi-34B adopts Llama 2's architecture, Meta's model did not give 01.AI access to any previously inaccessible innovation.

In November 2023, a Hugging Face user asked for two components of Yi-34B to be renamed so that the model would be compatible with third-party codebases developed for Llama 2 by default. Training data is the main reason LLMs differ from one another, and in that regard Meta disclosed no useful details; 01.AI developed and described their own English-Chinese dataset. The similarities between Yi-34B and Llama 2 do not support the argument that Chinese AI firms simply rely on American open models, because all LLMs have extremely similar architecture. Moreover, we note that most of the architectural innovations in Llama 2 came from implementing algorithms that were already known and described in research literature.

### An overview of the Hugging Face discussion for non-coders[#](#an-overview-of-the-hugging-face-discussion-for-non-coders)

Like in other pieces of software, language model developers assign names to software components in order to reference them later. For example, the first layer of a neural network is often named `layer_one`. Importantly, the actual name has no semantic content. It's just a symbol to refer to a particular piece of the software. However, these names are very important for interoperability between developers. If two people train neural networks but one calls the first layer `layer_one` and the other calls the first layer `layer_1`, then it can be very inconvenient for a third-party piece of code to interact with both of them, because the third-party code needs to be able to refer to the components *by name*.

When 01.AI uploaded Yi-34B to the Hugging Face Hub, they used a different naming convention than Meta. This meant that third party code designed to interact with Llama 2 didn't work with 01.AI’s model. This can be fixed by each third party developer tweaking their code, but across all third party developers this would cumulatively result in a lot of work. The purpose of the HF issue was to request 01.AI rename the components to avoid this overhead.

These kinds of compatibility issues are common in open source releases, and are not evidence of nefarious intent nor of 01.AI relying on Llama to train their model. EleutherAI has experienced it with many of our early model releases, and now we do compatibility checks as part of our standard pre-release review.

Nevertheless, a number of [media](https://technode.com/2023/11/15/kai-fu-lees-ai-large-language-model-yi-used-metas-llama-architecture-without-namechecking-its-source/) [outlets](https://www.scmp.com/tech/tech-trends/article/3241680/chinese-tech-unicorn-01ai-admits-oversight-changing-name-ai-model-built-meta-platforms-llama-system) blew the Hugging Face issue out of proportion, claiming in November that 01.AI had willfully concealed the connection between its leading model and Llama 2\. These hyperbolic claims received little attention in the machine learning community.

### How all LLMs are similar[#](#how-all-llms-are-similar)

At a high level, every existing large language model is currently trained using the same basic techniques and methodologies. The developer starts with a large dataset of text on the web, separated out into individual word-like components called "tokens." Then, a deep learning architecture (almost always a basic Transformer, as introduced by Attention Is All You Need [(Vaswani et al. 2017)](https://arxiv.org/abs/1706.03762) is trained to fit to the dataset by optimizing a loss function causing the neural network to predict the most likely next token that comes next in the dataset. This process is referred to as “pretraining.”

This basic recipe, and the building blocks used in it, have not fundamentally changed since the Transformer was introduced by Google Brain in 2017, and slightly tweaked to today’s left-to-right language models by OpenAI in [GPT-1](https://s3-us-west-2.amazonaws.com/openai-assets/research-covers/language-unsupervised/language_understanding_paper.pdf) and [GPT-2](https://d4mucfpksywv.cloudfront.net/better-language-models/language_models_are_unsupervised_multitask_learners.pdf). 

For example, the neural network architecture of Meta’s Llama 2 model series only adopts a few changes that differentiate it from the original Transformer in 2017, or the Transformer as used in GPT-2 or [GPT-3](https://arxiv.org/abs/2005.14165). These are the following:

    - [The SwiGLU Activation Function](https://arxiv.org/abs/2002.05202) is a minor change to the transformer architecture that was observed to provide a slight quality improvement. It was introduced by Noam Shazeer at Google Brain in 2020, and subsequently adopted by Google’s PaLM models. It is now **de facto** used by most new language models, although not universally.

    - [Rotary Positional Embeddings (RoPE)](https://arxiv.org/abs/2104.09864) are a method introduced by a group of Chinese researchers from Zhuiyi Technology Co., Ltd. in 2021, that allows language models to more cleanly keep track of the relative “positions” in a piece of text of two different tokens. The technique was subsequently popularized among the Western research community in part [by EleutherAI](%5Bhttps://blog.eleuther.ai/rotary-embeddings/%5D(https://blog.eleuther.ai/rotary-embeddings/)) and, due to its simplicity and several useful quality-of-life enhancements it provides, has become the standard for the vast majority of current LLMs.

    - A [“Pre-norm” architecture](%5Bhttps://arxiv.org/abs/2002.04745%5D(https://arxiv.org/abs/2002.04745)) is a re-ordering of the transformer model’s operations which makes it mathematically cleaner to reliably train. This correction was proposed by a number of groups, including a collaboration between Chinese academics and Microsoft Research Asia. As an interesting historical aside, this adjustment was discovered and used by Vaswani et al. (2017) in the original transformer paper, but was reported inconsistently.

    - [Multi-query Attention](https://arxiv.org/abs/1911.02150) and [Grouped-Query Attention](https://arxiv.org/abs/2305.13245) were both introduced by Google. Multi-Query Attention (MQA) is an improvement to the efficiency of running many inputs through a language model simultaneously for downstream use. It was first presented by Noam Shazeer of Google Brain in 2019, and was adopted by [Google’s PaLM model](https://arxiv.org/abs/2204.02311) at scale. Grouped-Query Attention (GQA) was proposed relatively recently, in 2023, by Google Research, as an extension of Multi-Query Attention which retained its benefits. Grouped-Query Attention’s adoption in Llama 2 has helped it become a new baseline for many recent language model architectures.

While it does not reduce Meta’s achievements in training Llama 2 at all, it is important to note that none of these advances were invented by Meta, and many have existed for several years. These techniques had become accepted as de facto best practices at the time Llama 2 was created and released.

We hope that this illustrates the commonalities between LLMs today, many tracing back several years to the original Transformer paper and first transformer-based models. 

### So what makes Yi-34B different from Llama 2?[#](#so-what-makes-yi-34b-different-from-llama-2)

Although the Llama 2 models’ weights, and therefore their architectural details, are available (this is necessary for the models to be run by other developers or users in their code), it is useful to look to the components of Llama 2’s creation that were **not** disclosed as an indication of the true differentiators.

Llama 2’s [technical report](https://arxiv.org/abs/2307.09288) says only the following about the exact contents of their dataset:

> Our training corpus includes a new mix of data from publicly available sources, which does not include data from Meta’s products or services. We made an effort to remove data from certain sites known to contain a high volume of personal information about private individuals.

No further details about data sources are given. This is because, since all current language models mostly use similar architecture, Meta’s key competitive advantage comes from their training dataset. 

Additionally, while Meta provides a [file](https://github.com/meta-llama/llama/blob/main/llama/model.py) defining the Llama architecture that can be used to load their models, they also do **not** release their infrastructure or codebase which was used to train the Llama models from scratch. To actually train one’s own language model requires assembling one’s own dataset and building this training infrastructure on one’s own computing clusters, which further requires an engineering team with expertise in high performance computing (HPC). Releasing the Llama 2 model weights does not enable other teams to develop a competitor for free any more than they already could, especially considering that advances like SwiGLU or GQA were already publicly known through open access academic publishing. In addition, Llama 2 also gave Meta concrete benefits: the Hugging Face discussion positions Llama as the current standard for open-weight LLM software, such that not using its architecture and naming convention is a disadvantage.

Two weeks after the New York Times article was published, 01.AI released the [Yi technical report](https://arxiv.org/abs/2403.04652), which mentions these factors. Perhaps in response to negative media coverage, they explicitly state that a “standard” architecture is sufficient, and that far more effort must be spent on curating and processing a high-quality dataset. For Yi-34B, this included work to ensure performance in Chinese, and they had to build out this data processing pipeline as well as model pretraining infrastructure from scratch. 01.AI thus had to solve the same problems as those faced by other language model training companies, and is not “built on” Llama 2 any more than it is built on other language models, including models that were never released, such as Google’s PaLM.

### Conclusion[#](#conclusion)

We hope this post illustrates how Yi-34B fits into larger trends in language model training. We want to emphasize that the NYT writers were not negligent and that the error is completely understandable, as interpreting this [Hugging Face issue](https://huggingface.co/01-ai/Yi-34B/discussions/11) clearly requires a substantial amount of context about model development and the current open AI ecosystem. Journalists cannot be expected to have this level of machine learning know-how. Our goal here is to educate, because we believe public discussion of AI needs to be firmly grounded in facts. In this case, 01.AI’s founder was absolutely correct when he stated that the company followed standard industry practices.