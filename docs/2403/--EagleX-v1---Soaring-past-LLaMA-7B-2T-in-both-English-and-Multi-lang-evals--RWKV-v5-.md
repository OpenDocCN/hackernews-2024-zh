<!--yml
category: 未分类
date: 2024-05-27 15:00:14
-->

# 🦅 EagleX v1 : Soaring past LLaMA 7B 2T in both English and Multi-lang evals (RWKV-v5)

> 来源：[https://substack.recursal.ai/p/eaglex-17t-soaring-past-llama-7b](https://substack.recursal.ai/p/eaglex-17t-soaring-past-llama-7b)

An eagle, flying past llama

> If you are fine-tuning, we recommend waiting for the full EagleX 2T model coming out later this month instead, unless you are doing so for research purpose.
> 
> This model is released for research purposes, as it represents the major checkpoint that surpasses LLaMA2 7B, as part of our current training to 2T tokens and beyond.

EagleX 1.7T is a early research release of our 7.52B parameter model training that:

We are releasing RWKV-v5 EagleX 1.7T, [licensed under Apache 2.0](https://blog.rwkv.com/p/rwkv-joins-the-linux-foundation-as), which can be used personally or commercially without restrictions.

It is a definitely a very big claim to say you have caught up and pass the “Gold Standard” of the 7B weight class from scratch, which nearly every other major open access model is built on (allegedly even Mistral). Even more so given that this is done with a comparatively lower dataset token count of 1.7 trillion token (vs. 2 trillion tokens).

As this is a entirely different model, trained from scratch, there will be evals that we win and we lose, which we are fully transparent about, in showing how we are ahead of LLaMA 7B on average.

Instead of simply cherry picking 14 different evals which we won and calling it a day with a victory, we ran ALL the benchmarks in EleutherAI `[lm-eval-harness](https://github.com/EleutherAI/lm-evaluation-harness)`, at commit `f78e2da` that we could do, with the following limitations:

*   It has to complete in under 30 minutes on 8x4090 (we were running lots of evals)

*   We excluded all the personality / alignment evals

*   Eval has to be executable across a wide variety of models, via lm-eval-harness

*   All evals are 0 shot (no 5 shot-ing an MCQ question)

*   We limited comparison to other models within the 7B weight class

These resulted into running 60+ major eval groups, which generated over 1,000+ data points per model. A data point count so high, that we had to drop standard error deviations, just to ensure the raw CSV file can be loaded in MacOS numbers.

What it takes to fit 184 english eval data point onto the screen.

Whew, that’s a crazy number of data points to digest. Let me break it down to more digestible parts:

All data shown here is made available in the Google Sheet over here:

> We included explanations of what several of the evals mean, which you can keep in mind in future eval results you see (demystify what those numbers mean!)

* * *

We start with basics: Perplexity. This is the loss value against the test dataset (lower score = better), i.e. how good the model is with next token prediction.

In general, with the perplexity improvements, the EagleX model outperforms LLaMA2-7b, ranking between Falcom/LLaMA2-7b and Mistral.

**Why do experts care about perplexity?**
Eval in general can be very subjective, and opinion driven, and commonly gives mixed results. Perplexity in a way gives the TLDR summary for most experts to start with

EagleX maintains the lead for best in class multi-lingual performance, with the incremental improvements we’re making to the Eagle line of models.

Most of the tasks here are common sense reasoning tests of wide variety of formats, across languages including [23 of the world’s most widely used languages.](https://blog.rwkv.com/i/141130059/multi-lingual-performance-details)

For the remaining languages, we urge the community to test and judge it themselves, over a 100+ languages was trained. Over time, we would want more languages to be added into evals.

**Why is multi-lingual perf important?**
The goal of the RWKV project & Eagle line of models, is to build **inclusive** AI for everyone regardless of their language. Our mission is to build AI models not just made for English, but also for the 83% of the world’s population using a non-English language everyday.

Nevertheless, English is still important. We reduced the evals down to 21 of the argubly most popular English evals, such as Lambada, Glue, Swag, Winogrande, TruthfulQA, MMLU:

Narrowing it down to the 4 models that most of us actually care about - LLaMA, Mistral, EagleX and Eagle-7b - the new EagleX model outperforms LLaMA-2-7b on average across the 21 evals, and lags not far behind Mistral.

Keep in mind that this average shown, is across all 21 evals

* * *

Now, let’s look at where our model is blowing the rest of the models out of the water.

First, the big stand out is the first 6 evals, which even our small 1.7T trained model beats out even Mistral 2T++ trained model (sciq, glue, anli, mmnli, swag), across multiple tasks focused around either contextual based simple Q&A with common sense reasoning, or deductive logic. EagleX also performs better than LLaMA-2-7b in wingrade and wnli evals, which also involves contextual common sense reasoning as well. This implies that the EagleX model would be applicable in RAG use cases, which are mainly contextual Q&A, with the right prompt engineering.

Finally, for truthfulqa, while it outperforms LLaMA, but in my opinion, this is still indicative of how vulnerable all models are from learning common human misconceptions from the web, seeing how bad the scores are across all models.
(to be fair, this is hard for most humans as well)

> PS: The jump for glue/mnli was high enough, that we needed to check the dataset specifically for contamination. Which we were not be able to find any. This jump is currently being attributed to multiple training datasets, along with data augmented / machine rewritten instruct dataset following a similar structure.

Strong common sense reasoning over context,
has very strong applicable use cases for multiple RAG use cases

Next: the eval sets with mixed results. Here, we have very similar evals with 2 major variants. The results between EagleX and LLaMA are close enough, that it’s hard to say which model is clearly better between the two for these evals.

What’s interesting, is that even though logiqa can be seen as form of “common sense” reasoning test, the EagleX model scored much lower compared to the 6 evals (sciq, glue, anli, mmnli, swag). This could mean that while the model is better at reasoning given a context, but it lacks the depth of knowledge compared to other models with more token training.

* * *

These are the evals the EagleX performs worse on compared to both Mistral and LLaMA. However, for the evals that we’ve lost to LLaMA, it’s by a narrow margin. But we’ll be keeping track of these as we train past 2T tokens.

Let’s look what went really badly: Math.

The results for arithmetic eval sank drastically, like a rock, even compared to our original Eagle model.

What went wrong?

We dug through the dataset we used for training, and realized we missed out the entire math dataset (along with a few others) due to an error. Oops.

This emphasize the importance of maintaining the dataset composition over the training run. We’re adding math back for future runs.

> We expect overall math score to rise back up as the training continue, however realistically IMO - no one should be depending on a 7B model for math (just saying)

* * *

We do not simply want to cherry pick 9 or 21 evals and claim victory over LLaMA, or even Mistral. So, let’s zoom out, and look at it holistically across 183 English evals.

[You can view the full results here](https://docs.google.com/spreadsheets/d/1PFELH3u8yQlr-bGs9D5lBYXCXqSFZw2O0vfW084jbgI/edit?usp=sharing)

Although using the overall averages across all the evals does have a bias the results towards larger eval sets (due to double counting, e.g. mmlu overall and many indivudall mmlu test), it does not change the ranking among the EagleX, Mistral, LLaMA and the original Eagle models.

However these results is extremely useful for smaller insights, for example

The EagleX model lost to LLaMA2 when it comes to US history, but won in world history. This makes sense, given the broader approach we took to making the dataset from a more inclusive, more global view, instead of a US centric one.

The detailed insights will be used by our dataset team to iterate and improve on our future datasets.

How the model answer, is a reflection of the dataset experiences it has learnt
How much resources the model consumes, is a reflection of its architecture

One of the biggest change we did was to change the dataset for the current 1T tokens, which now uses a cleaner filtered set of data with **careful considerations to ensure permissible licensed content sources used**.

There are also huge implications on the fact, the model crossed the llama2 line earlier then the plan schedule. That either the architecture is more efficient in training, or that the improvements in dataset quality has a large impact in model performance.

The following is a summary of the dataset used, its public release will be made available next month after the current 2T training is completed.

```
## 15% Code 

Contains code/programming related topics
- the-stack
- codeparrot
- devopedia
- mdn

## 15% Multi lang

Generally multi-lang webtext
- sea-lion (Singapore)
- madlad
- culturax
- multi lang wiki

## The giant soup

Creative content
- fandom (only sites with permissive licenses, and low spam)
- scp-foundation

Wikipedia
- Various Permissively licensed wikis.
- wikipedia

Papers:
- Mainly arxiv (Permissive Licenses) and pes2o

Books:
All the books contained in out train sets are public domains books.
- gutenberg, 
- standardebooks

Webtext
- webtext
- refinedweb (Note: This chunk made the model worse, we recommend against refinedweb in future trains)
- slimpajama
- europarl
- eurlex.
- stackexchange

Various
- aya (multilang convo)
- some system prompt, instruct
- long list of sub 100B training datasets on HF
- rewritten text !!! (splicing in, to replicate the rewritten web paper)
```

* * *

We are over a 100x more scalable then the transformer architecture.
Transformers became the most prominent architecture in AI, not because it was the best, but it was the first to successfully scale to billion of parameters in training.

Till today

CUDA computational cost, for RWKV-based architecture vs transformer models - that quadratic-vs-linear really scales!

* * *

In overall, the release of this model marks an important milestone and transition for many of us, within both the commercial team within Recursal AI, and the open source team in the RWKV group.

*   Its the first major training done by the Recursal AI team, in partnership with AWS as our main compute provider

*   This model is being released under Apache 2 licensing

*   The fully trained 2T model will be released under the RWKV group, under the Linux Foundation

*   The first Non-Transformer Architecture to pass LLaMA2 in evals

*   The strongest Linear Transformer to date

*   Proof you can have both strong multi-lingual and english performance

* * *

Similar to the original Eagle 7B announcements, the following is the revised goals for the model training

*   [April 2024] Completion of the 2T Eagle 7B models

*   [March-May 2024] Training of our v6 “Finch”line of models

*   [June 2024] v6 MoE model, for GPT 3.5 class performance

> Disclaimer: All dates are approximate, and is heavily subjected to compute availability from our sponsors/compute-provider/investors

If you want find more about the RWKV opensource Project at

If you like to try the model today, you can do so on our platform at [recursal.ai](https://recursal.ai) - the best place host, run, and create finetunes of the Eagle line of RWKV models.