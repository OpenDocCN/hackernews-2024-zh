<!--yml
category: 未分类
date: 2024-05-29 12:28:51
-->

# X.ai's Grok-1 Model is Officially Open-Source and Larger Than Expected

> 来源：[https://synthedia.substack.com/p/xais-grok-1-model-is-officially-open](https://synthedia.substack.com/p/xais-grok-1-model-is-officially-open)

X.ai announced today that the Grok 1 large language model (LLM) is officially available under the open-source Apache 2.0 license. This permissive license allows users royalty-free access to the source code for commercial and private uses. According to [X.ai](https://x.ai/blog/grok-os):

> This is the raw base model checkpoint from the Grok-1 pre-training phase, which concluded in October 2023\. This means that the model is not fine-tuned for any specific application, such as dialogue.
> 
> We are releasing the weights and the architecture under the Apache 2.0 license.

The GitHub repo adds:

> The code and associated Grok-1 weights in this release are licensed under the Apache 2.0 license. The license only applies to the source files in this repository and the model weights of Grok-1.

That clarification means users of Grok-1 are not entitled to access other X.ai models using the same name or in the same model family.

The announcement is limited to the Grok-1 pre-training model, which is the model state before it went through the instruction phase, which optimized it for dialogue. It also means that entrepreneurs won’t be able to simply download the model and wrap it with a new brand to create a Grok competitor. Additional training would be required for this. Because of this, companies looking to create a new conversational chatbot might be better off using one of Meta’s Llama 2 Instruct or Mistral Instruct models.

X.ai did provide some new details about the model. In addition to making the model weights available (which are always withheld from proprietary models and often withheld in other open-source models), X.ai also revealed that Grok-1 is a 314 billion-parameter model. At the time of the Grok-1 announcement in [November 2023](https://synthedia.substack.com/p/what-is-grok-xais-chatbot-on-twitter), the company did not disclose this detail but instead mentioned that Grok-0 was a 33 billion parameter model.

This confirms that Grok-1 is a very large model that is larger than GPT-3/3.5 but likely smaller than GPT-4\. It is also more than four times larger than Meta’s Llama 2 70B model. Parameter count is not a strict correlation to performance but often provides better results when paired with large, high-quality training data sets and architectures. Benchmarks [published](https://synthedia.substack.com/p/what-is-grok-xais-chatbot-on-twitter) by X.ai in November suggested Grok-1 showed stronger performance than GPT-3.5 and Llama 2\.

X.ai also commented that Grok-1’s architecture is based on a Mixture-of-experts (MoE) design. This may be significant. GPT-4 has long been rumored to have an MoE architecture, and Mistal’s Mixtral 8x7B model also employed this approach.

There is a growing group of researchers who believe the MoE models are a more efficient method to scale to higher performance than loading up parameter counts. Instead of a single large model that processes all queries, MoE models are comprised of multiple “expert” models that are specialized by task. Examples could be expert sub-models for reasoning, language translation, language generation, summarization, or mathematics.

Unlike unified models such as GPT-3, MoE models also have arbitration features. Arbitration assigns tasks to the submodels based on the request and rates the responses before delivery to the user. By not activating the entire LLM for every query, the models can often reduce computing costs and latency.

It is widely known that Elon Musk originally collaborated with OpenAI CEO Sam Altman with the express objective of creating open-source AI models. That was the origin story behind the company name.

OpenAI eventually reversed course when it decided the capital requirements to develop technology similar to GPT-4 or the aspirational AGI objective would far exceed the level that a non-profit could raise independently. That doesn’t fully explain why OpenAI decided not to maintain an open-source posture or produce a more permissive open model license similar to Meta, but the decisions were made concurrently.

Musk says that is why he left OpenAI in 2018, though it has subsequently come to light that his break with the company came after it rejected an acquisition offer by Musk-owned Tesla. Musk [filed a lawsuit](https://www.cnbc.com/2024/03/01/elon-musk-sues-openai-and-ceo-sam-altman-over-contract-breach.html) against the company in March 2024, suggesting that OpenAI had broken its original founding agreements—essentially a breach of contract with early backers. A few days later, OpenAI issued a rejoinder in the form of a [blog post](https://openai.com/blog/openai-elon-musk).

Technology luminaries such as Jeff Bezos posted earlier this month on X that X.ai and Grok were really no different from OpenAI, as both only offered proprietary models. That has now changed … somewhat. Grok-1 is open-source in stark contrast to OpenAI’s proprietary-only offerings. Granted, Grok-1 is not the same as the model used for the Grok AI assistant. You could argue that X.ai has both open-source and proprietary models at the moment.

Interstingly, Mistral launched into the market as an open-source LLM and followed up with another open-source model. More recently, the company launched [two proprietary LLMs](https://synthedia.substack.com/p/mistral-introduces-two-new-high-performance). Google, which only provided proprietary models in the past, now has an [open small language model](https://synthedia.substack.com/p/google-goes-big-on-context-with-gemini) as part of its offering. Note that I used the term open model for Google and Meta and not open-source. Open models are made available for use but include additional restrictions beyond traditional open-source licenses.

It will be interesting to see the developer community’s reaction. For now, open-source developers have another ally in generative AI innovation. This may cause some other companies in the space to consider supporting open-source developers as well.

[Share](https://synthedia.substack.com/p/xais-grok-1-model-is-officially-open?utm_source=substack&utm_medium=email&utm_content=share&action=share)