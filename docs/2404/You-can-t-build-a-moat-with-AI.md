<!--yml
category: 未分类
date: 2024-05-27 13:06:27
-->

# You can't build a moat with AI

> 来源：[https://generatingconversation.substack.com/p/you-cant-build-a-moat-with-ai](https://generatingconversation.substack.com/p/you-cant-build-a-moat-with-ai)

Differentiating AI applications is a hot topic, and it’s hard. Is everything just a RAG application? (No.) If a company gets good enough at RAG in one domain, does it neatly transfer to another domain? (Maybe! We’re not sure.) Are all AI applications just going to be built by OpenAI? (Probably not.)

Managing technical and go-to-market differentiation is an eternal question for startups. Some believe it’s all in the technology, and others believe it’s all in the execution. Whatever the case may be, it’s become clear to us that there’s not very much you can do to differentiate yourself using just LLMs. **The real differentiation lies in your data you feed into your models.**

This might sound surprising at first; after all, it’s the LLMs that have kicked off the current AI hype cycle. If they don’t matter, then… what are we all doing here?

LLMs are obviously incredibly powerful, but we believe the core models and their use will effectively be commoditized across competitive products in the same space. In that world, having a clever use of LLMs ([especially if you’re relying on a single LLM call](https://generatingconversation.substack.com/publish/posts/detail/142618146)!) is simply not going to cut it. Instead, you’ll need to think carefully about what data you’re using to build your application. Here’s why:

**Everyone’s using the same model.** Regardless of what you believe the best LLM is — we mostly think it’s GPT-4 — there’s probably a model that outperforms its competitors for each major category of task. If you had to bet, that model is probably still GPT-4 for most use cases, but Claude is catching up quickly. Everyone today can easily switch between models and experiment with what works best today for their application. That means that whatever you’re building, the advantage you get by picking the right model is small.

Deviating from the norm is also dangerous — it might get you unexpectedly good answers in some cases and unexpectedly bad answers in other cases. When a lot of AI startups are working to build trust, that’s a risky proposition, and as a result, most teams default back to using the safe models. That means that what you put into the model matters more than ever.

**And your prompts aren’t IP.** It might feel like your applications’ prompts or prompt templates are a good form of differentiation. After all, your top-notch engineering team has invested days into tuning them to have the right response characteristics, tone, and output style. Of course, giving your competitors your prompts would probably accelerate their progress, but any good engineering team will figure out the right changes quickly. The main reason is that the experimentation (with the right evaluation data!) is quick and easy — trying a new prompt template isn’t much harder than writing it out. All it really takes is a little bit of patience, some creativity, and extra Azure OpenAI credits.

Your prompts might be better today, but we can almost guarantee you that advantage won’t last. You’ll have to look elsewhere.

**So (as always) it comes down to the data.** If the models are commoditized, and the prompts are commoditized, all that’s left to differentiate your AI application is the data you feed into your LLMs. **Thankfully, the data makes a huge difference**. Any application you build — especially if you’re working with enterprises — is going require using your customers’ data to drive your applications’ responses and decisions. Every team is going to have slightly different data and slightly different preferences about how it’s used.

The nice thing about LLMs is that the they do well with arbitrary, unstructured data. The catch is that your customers have a *ton* of arbitrary, unstructured data. Some of it will be absolute gold; some of it will be completely useless. As an application builder, it’s on you to figure out how to use that data.

**And of course… garbage in, garbage out.** LLMs as with any AI model (or any system?) are garbage in, garbage out. That puts a huge burden on your ability to find and use the right data. For reference, our team at RunLLM has spent roughly 70% of our engineering cycles in the last quarter on data engineering — everything from pre-processing data at ingestion time to implementing hybrid search to reranking results. Whenever we get a customer complaint or request, the first solution has typically been to improve the data engineering process.

The result is a fairly complex data pipeline that can account for customer priorities, process a wide variety of data types, and generate metadata for downstream complex reasoning tasks. It certainly isn’t something that can be reproduced with a little bit of experimentation. It’s directly driven by getting hands-on customer feedback. The result is what’s allowed us to consistently differentiate ourselves with straightforward, grounded responses, and minimal hallucinations.

* * *

We firmly believe the moat for AI application is in the data and the data engineering today. At some point, the process of building custom LLMs might get so fast and easy that we’ll all return to building our own models. That simply isn’t the case today.

What that also means is that you don’t need to be an AI genius to succeed in building applications. In fact, focusing on the AI might be a detriment to your differentiation at a certain point. With thoughtful software engineering and a focus on customer data, you’ll build a moat over time.