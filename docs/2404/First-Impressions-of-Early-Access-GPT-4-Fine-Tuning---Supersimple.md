<!--yml
category: 未分类
date: 2024-05-27 13:20:06
-->

# First Impressions of Early-Access GPT-4 Fine-Tuning | Supersimple

> 来源：[https://www.supersimple.io/blog/gpt-4-fine-tuning-early-access](https://www.supersimple.io/blog/gpt-4-fine-tuning-early-access)

A few weeks ago we finally got access to the GPT-4 fine-tuning API (in limited early access), and were super excited to check out how well it works. We’d been a user of OpenAI’s fine-tuned models since fine-tuning the original GPT-3 davinci model first became available.

Unsurprisingly, a fine-tuned GPT-4 greatly outperforms fine-tuned GPT-3.5 (more than 50% improvement for our use case!). We’ll go deeper into how we use these models below, but let’s start by comparing several of the OpenAI-based models we use at Supersimple:

*   Fine tuned (FT) GPT-3 Davinci model, which we used at the beginning
*   GPT-3.5 and GPT-4 base models
*   GPT-3.5 and GPT-4, fine-tuned models using a custom proprietary data set

Relative model performance based on our test set; FT signifies a fine-tuned model

The models in question were fine-tuned for a domain-specific use case of asking data questions in natural language. The LLMs are tasked with putting together the appropriate reports and underlying queries for the user’s question.

These evaluations were performed using a proprietary test data set we use at Supersimple; Davinci’s (GPT-3) performance is taken as a baseline. To give more insight into where we see these models working well, let’s give a bit more context on what we actually do. We’ll also include more comparative data, including cost and latency, below!

## Context & our LLM usage at Supersimple

Supersimple is a data analytics platform that allows users to dive deep into their data, to answer complex ad-hoc questions in seconds and to uncover deep data insights.

Our users can ask natural language (plain English) questions about their data, and get answers in the form of tables and visualizations. The AI explains its work using a few no-code steps, and the user can always continue to go even deeper – either using natural language or a few clicks on our data platform.

We use LLMs to answer our users’ natural language questions, with the goal of providing a great starting point for further deep dives into the data. The LLM takes the user's question, relevant parts of their [semantic data model](https://www.supersimple.io/blog/semantic-layers), existing reports and dashboards, and generates an exploration that answers the question with the most context possible.

‍

This feature is also particularly useful as an onboarding tool, which showcases the capabilities of our platform and teaches users to use our data exploration UI. For long-time users, plain English is at times still the fastest way to answer questions, but at times it’s just as quick for them to just click a few buttons.

## Implementation

Technically speaking, the LLM outputs a custom high-level markup language (DSL) that we designed for our specific use case, that’s token-efficient and well-aligned with pre-trained language models. This DSL gets compiled into JSON, that we in turn compile into one or more database queries to be executed. Finally, we render an interactive data exploration to the user based on the report structure dictated by the DSL.

It's important to highlight that this task is quite different from the traditional text-to-SQL problem for a number of reasons:

*   The output is a no-code exploration, which is transparent and explainable to the user. This means that the model directly interacts with our data platform. We believe that generating & showing code or SQL is a terrible way to prove/explain reports to users – even if those users are technical, but especially if they aren’t.
*   The resulting report/exploration is easily editable in the UI, and is meant to serve as a starting point for further deep dives. When exploring data, you won’t want to think about refactoring SQL queries.
*   The complex output is broken into individual blocks that represent logical steps in the reasoning process.
*   We offload the complexity of constructing correct SQL queries with many joins, CTEs and subqueries to the platform’s “query engine”, which deterministically compiles the semantic steps output by the model into valid and performant SQL. This means the LLM doesn’t need to worry about an entire class of problems, allowing it to perform better overall.
*   When generating output, the model takes into account existing dashboards and user-defined concepts.

Due to the complexity of the task and the need for a custom output format (a DSL that the AI uses to effectively construct app UI state), we found that fine-tuned models tend to perform significantly better than the base ones.

To reiterate: we never generate SQL using LLMs – our LLMs only interact with our app directly.

## **Fine-tuning and benchmarking**

For fine-tuning, we use a proprietary dataset that contains tens of millions of tokens worth of question-answering examples, with combinations of data models, questions and the perfect output reports to answer these questions.

GPT-3.5 (gpt-3.5-turbo-0125) and GPT-4 (gpt-4-0613, the only fine-tunable GPT-4 model available at the moment) were fine-tuned for 3 epochs.

For both of the base models here (gpt-3.5-turbo-0125 and gpt-4-0125-preview), we used the same prompt, containing 8-shot examples. Performance of all models except davinci was evaluated in the beginning of March 2024 while the performance of davinci was last evaluated back in August 2023.

The performance comparison is shown in the next table. For our test set, fine-tuned (FT) GPT-4 **outperformed GPT-3.5 by 56%**, a significant, albeit slightly smaller improvement than the jump from Davinci FT to GPT-3.5 FT (96%). 

Fine-tuned GPT-4 is slightly slower than fine-tuned GPT-3.5 (21.6 tokens/s vs 31 tokens/s).

*Note: the original version of this article showed significantly worse latency stats for GPT-4: our first benchmarks only achieved 5.8 tokens/s, but OpenAI has since greatly improved the service's stability and speed.*

While cost is significantly higher for GPT-4 (15x higher for inference, 11x higher for training, compared to GPT-3.5), it might not be an important factor for you depending on your use case.

We benchmarked models on our in-house test set of 100 diverse questions. Examples of questions and model performance are shown in the table below:

| Question | Davinci FT | GPT-3.5 | GPT-4 | GPT-3.5 FT | GPT-4 FT |
| User with id 342122 | ✅ | ✅ | ✅ | ✅ | ✅ |
| Companies that are using the analytics dashboard? | ✅ | ❌ | ✅ | ✅ | ✅ |
| What's our ARR and how is it changing over time? | ❌ | ❌ | ❌ | ✅ | ✅ |
| Companies whose subscriptions are expiring this month | ❌ | ❌ | ❌ | ✅ | ✅ |
| QoQ change in feature usage on account level | ❌ | ❌ | ❌ | ❌ | ✅ |
| Churn trends this year vs last year | ❌ | ❌ | ❌ | ❌ | ✅ |
| What are the main blockers in our onboarding funnel? | ❌ | ❌ | ❌ | ❌ | ❌ |

‍

Despite the performance enhancements, models continue to struggle with broad and open-ended queries when trying to answer with a single completion.

Worryingly, there is an observable trend of diminishing returns from fine-tuning. While fine-tuned Davinci showed marked improvement over its base model, fine-tuned GPT-3.5 offered lesser gains, and the progress achieved by fine-tuning GPT-4 was even smaller.

To address these limitations, in production, we seldom rely on a single model call. Instead, we employ a mix of various specialized models, prompts, and heuristics to improve both accuracy and response time, achieving better performance than any individual model.

## **Conclusion**

Having used fine-tuned LLMs in production from the early days of GPT-3-davinci being publicly available, we were extremely excited to get to try out fine-tuning GPT-4\. On the one hand, we were blown away by the performance gains over the previously-best GPT-3.5-FT.

On the other hand, we actually expected this, as the gap between the base versions of GPT-3.5 and GPT-4 was already immense.

Fine-tuned GPT-4 significantly outperforms both fine-tuned GPT-3.5 and vanilla GPT-4 on our test set. We saw comparable improvements to those that we saw when first switching from GPT-3-FT to GPT-3.5-FT last year.

The main issue at the moment with the largest models is higher latency. As our goal is to make human interactions with data seamless and effortless, we can't have users waiting behind an AI for more than a few seconds.

Because of the higher latency (and cost), we only use GPT-4-FT for a certain subset of questions and for some of the most critical reasoning steps. For the rest, we use various other models, including GPT-3.5-FT.

With current state-of-the-art models, we believe that:

1.  A single model with a single completion (or API call if using hosted LLMs) is not sufficient for many real-world applications that require non-trivial levels of reasoning capabilities
2.  In a work context, it’s critical for any AI to accurately explain its work to users (people don’t even believe other humans before seeing proof!)

Read more about our [natural language question-answering AI](https://www.supersimple.io/platform/natural-language-ai) on our website, or get started with our next-generation business intelligence platform today.