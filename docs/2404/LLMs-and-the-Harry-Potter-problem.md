<!--yml
category: 未分类
date: 2024-05-27 13:27:25
-->

# LLMs and the Harry Potter problem

> 来源：[https://www.pyqai.com/blog/llms-and-the-harry-potter-problem](https://www.pyqai.com/blog/llms-and-the-harry-potter-problem)

### Introduction

Long-context LLMs are all the rage these days - it feels like every new one increases its context window by about an order of magnitude over the competition. They even do well on typical benchmarks, like the [Needle In A Haystack](https://arstechnica.com/information-technology/2024/03/claude-3-seems-to-detect-when-it-is-being-tested-sparking-ai-buzz-online/). Unfortunately, in our opinion, they still have some key issues: we call it the Harry Potter problem.

In this article, we will begin with a description of the problem and its practical implications. In response to our proofreaders (thanks all), we provide an explanation for why agents, out-of-the-box RAG and fine-tuning aren’t adequate solutions to this problem. For more technical readers, we also provide some reading and data to chew further on.

> "Large language models may have big context windows, but they still aren't good enough at using the information in big contexts, especially in high value use-cases."

#### The problem

Imagine you give an LLM (note: not an agent - explanation for why we care about the LLM’s ability later in this article) a chapter of Harry Potter and ask it to count how many times the word “wizard” is mentioned. GPT4, Claude 3 Opus, Gemini Ultra and Mixtral - arguably representative of SOTA as of this writing - all fail at this task despite having big context windows.

This test admittedly has a split focus: it is measuring the model’s in-context recall and its ability to count, both of which are weak points. For the purpose of this discussion, we will focus on in-context recall because that’s more closely related to the point of long-context models.

Here are some summary statistics that help describe the issue. Read till the end for all the details on our tests and methodology:

1.  **GPT4 Turbo: 55% accuracy** on documents over >=64k tokens
2.  **Claude 3 Opus: 65% accuracy** on documents over >=64k tokens
3.  **Mixtral 8x7b Instruct: 17.5% accuracy** on documents over >=64k tokens
4.  **Gemini 1.5 Pro: 45% accuracy** on documents over >=64k tokens

Evidently, none do especially well in long contexts. More details about our methodology at the end of this post.

#### Why should I care?

In short, this problem can really hurt LLM accuracy and is very hard to catch. Harry Potter is an innocent example, but this problem is far more costly when it comes to higher value use-cases. For example, we analyze insurance policies. They’re 70-120 pages long, very dense and expect the reader to create logical links between information spread across pages (say, a sentence each on pages 5 and 95). So, answering a question like “what is my fire damage coverage?” means you have to read:

1.  Page 2 (the premium)
2.  Page 3 (the deductible and limit)
3.  Page 78 (the fire damage exclusions)
4.  Page 94 (the legal definition of “fire damage”)

This is hard enough for a human expert, and it’s nearly impossible for any existing LLM. The greater issue is that any failure of the above process can materially affect the correctness of the answer in a high-value use case. To make matters worse, the model can (and will) confidently BS. Some other situations where this might be an issue include: reviewing lengthy legal cases, understanding codebases, reviewing medical records for anyone older than 2, and more.

#### Doesn't RAG solve this?

Not quite. Traditional RAG - we’re talking LangChain + embeddings model + OpenAI + semantic/hybrid search - does not take into account the structure and informational hierarchy of a document. That means the retrieved chunk(s) and prompt ignore the rest of the information found near them which might be relevant. For example: if you were looking for “fire coverages” in an insurance policy and pulled a chunk with the definition of “fire coverage,” it could either be the types of fires covered or excluded,and the information describing it as such was located several pages before this chunk. And if you really did want to keep the entire structure, you’d end up increasing your chunk size, retrieving a large context and end up right where you started.

Metadata filtering is a step closer, but it also does not completely solve the problem. You still limit the retrieval to some arbitrary number of chunks and might miss some context.

#### Doesn't fine-tuning solve this?

Only partially. LoRA and other “quick” fine-tuning methods do not fix the fundamental issue with how LLMs tend to digest context, since they have been shown to pay more attention to the beginnings and ends of documents than to information located near the middle [1 - “Lost in the middle”]. A series of carefully planned full fine-tunes, in a certain order, can help long context performance [3 - LongRoPE]. The core issues here are cost and scarcity of long texts (especially industry-specific ones).

#### Don't agents solve this?

As of this writing, agents have the potential to solve this problem but aren’t there (yet). OpenAI with tool-calling can solve the Harry Potter problem, but it still misses out on the greater context of the document, which is the point of this discussion. For it to work, an agent would need to autonomously digest the entire document, develop an ontology that addresses your use-case, and figure out a way to parse said document with all of its complexities. This would be a revelation and we hope it happens. Unfortunately our experience with agents has not yet yielded satisfactory results.

#### Ok, so what’re you doing about it?

We’ve found that the best way to solve this is to have an opinionated view of what each long document should look like, the information it should contain, and how the information within the document is interconnected. This is a difficult undertaking and it does not generalize, so unfortunately there isn’t really a hack here. For instance, we’ve spent months studying every insurance policy we can get our hands on to develop an ontology that we try to fit the document to, and then built an ingestion and retrieval pipeline around it.

Basically, we made a tradeoff: in exchange for great insurance policy understanding, we gave up the ability to understand anything else. We had to modify our general-purpose retrieval engine to work with certain categories of queries that (to our knowledge) do not exist outside of the industry, the LLM to be able to properly understand industry terminology, and our document ingestion to handle faxes (yup that’s a thing too). Knowledge graphs can be helpful with the right approach, as can experimenting with the many recently announced chunking techniques.

Another thing to consider is treating the document like an encyclopedia - build a table of contents, glossary and list of citations that an LLM can consult before retrieving chunks from it. This is also useful during ingestion.

#### How do I do this for my own documents?

Start by picking a category of document you wish to understand, and accept that you’ll have to repeat this process for each new category until either:

1.  The Harry Potter problem is fixed, or
2.  Someone comes up with a generalizable way to build a knowledge graph out of a document that actually works

Side-note: we would love to hear about work done on b, especially given the uptick in the number of long-context LLMs.

Next, develop an opinion of the information that you think all variants of this document must have. Try to list them, their types and their relationships to each other (Graph Theory is helpful in this respect).

Finally, and much easier said than done, experiment with as many examples as you can find. Chances are that you will need to iterate on this several times before you see good results. For the right business problem, however, the juice is worth the squeeze.

#### Descriptive statistics

This is not meant to be comprehensive or academic, however hopefully these statistics will help convey the point. You can try all of them yourself using the prompts and information below.

*Accuracy of top 4 LLMs when given the Harry Potter problem*

*Accuracy of top 4 LLMs when tested on a simple factual query that requires in-context recall*

*Accuracy of top 4 LLMs when asked to find multiple needles in a haystack*

*Accuracy of top 4 LLMs when asked to list all insurance policy coverage types*

##### Prompts & Methodology

We ran each of the following prompts using the same input and various temperatures 10 times to minimize situations where the model got unlucky. They were evaluated by a combination of another LLM (GPT3.5) and human review. Incomplete answers as well as overly verbose ones containing irrelevant information are considered wrong. The results are an aggregate of all the runs.

Prompts:

1.  How many times is the word ‘wizard’ mentioned in the following excerpt? <chapter 1 of The Half Blood Prince here>
2.  What is section 523 of the Dodd-Frank act about based on the information below: <[Section-wise summary of the Dodd-Frank act, US Congress](https://www.congress.gov/bill/111th-congress/house-bill/4173)>?
3.  List out the first 5 numbered sections and summaries of Title V, Subtitle B in the document below: <[Section-wise summary of the Dodd-Frank act, US Congress](https://www.congress.gov/bill/111th-congress/house-bill/4173)>
4.  List out all of the coverage types from the insurance policy below. Some examples of coverage types include: Professional Liability, Commercial General Liability, Employee Benefits Liability, and Cyber and Privacy. <insurance policy redacted for privacy reasons, but use any commercial policy you want>

‍

#### Technical reading

1.  [Lost in the middle](https://arxiv.org/abs/2307.03172): A pretty good review of the fundamental issue with LLMs and long contexts
2.  [RULER](https://arxiv.org/abs/2404.06654): A new framework proposed by researchers at NVidia to replace the needle in a haystack test. This demonstrates the Harry Potter problem far more rigorously.
3.  [ALiBi](https://arxiv.org/abs/2108.12409) and [LongRoPE](https://arxiv.org/abs/2402.13753) (an extension of the original RoPE design most popularly seen in the T5 models): How we can get larger context windows without training on equivalently large bodies of text
4.  [Long-context LLMs Struggle with Long In-context Learning](https://huggingface.co/papers/2404.02060)

*Want to analyze and compare long, complicated insurance policies? Get in touch with us by booking a call using the link above!*