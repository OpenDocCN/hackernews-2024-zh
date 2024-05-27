<!--yml
category: 未分类
date: 2024-05-27 13:30:43
-->

# The question that no LLM can answer and why it is important

> 来源：[https://www.mindprison.cc/p/the-question-that-no-llm-can-answer](https://www.mindprison.cc/p/the-question-that-no-llm-can-answer)

***[Notes From the Desk](https://www.mindprison.cc/s/notes-from-the-desk)** are periodic posts that summarize recent topics of interest or other brief notable commentary that might otherwise be a tweet or note.*

* * *

“Which episode of Gilligan’s Island was about mind reading?”

No LLM properly answers this question unless it uses a web search. But how can a LLM not know the answer if it was trained on essentially the entire internet of data and certainly most likely all the data in IMDB? The correct answer is “[Seer Gilligan](https://www.imdb.com/title/tt1230843/)”, one of the most popular episodes.

The following are the responses from multiple top models:

> **Opus**
> 
> I apologize, but there is no episode of Gilligan's Island that specifically focuses on mind reading.
> 
> * * *
> 
> **GPT**  **4**
> 
> The episode of "Gilligan's Island" that involves mind reading is titled "Ring Around Gilligan". It is the sixth episode of the third season.
> 
> * * *
> 
> **Gemini 1.5**
> 
> The episode of Gilligan's Island that features mind reading is Season 2, Episode 12 - "The Mind Reader".
> 
> * * *
> 
> **Llama 3**
> 
> After conducting a thorough search and reviewing episode guides, I found that there is no episode of Gilligan's Island that specifically focuses on mind reading.
> 
> * * *
> 
> **Llama 3 -** ***asked to list and name episodes***
> 
> "The Postman Cometh" (March 13, 1965)
> "**Seer Gilligan**" (Wait, no! I apologize, but there is no episode titled "Seer Gilligan")
> "Love Me, Love My Chicken" (March 20, 1965)
> 
> Please note that there is no episode titled "**Seer Gilligan**". If you have any other questions or need help with a specific episode, feel free to ask!

All four models fail to give the correct answer. Two of the models hallucinate an answer and the remaining two deny such an episode exists. Interestingly, probing Llama 3 a bit more reveals it is aware of the episode; however, it still denies its existence. It seems to know there are 98 episodes, but the majority of titles it lists are made up.

In the case of Llama 3, we can probe some of the training dataset [using Infini-gram](https://huggingface.co/spaces/liujch1998/infini-gram) and verify that the episode does exist in the corpus along with text describing the episode.

We also see another interesting data and training phenomenon revealed when LLMs are asked to provide a number between 1-100\. They all converge to 42!

* * *

Pointed out by [Information is Beautiful](https://twitter.com/infobeautiful/status/1778059112250589561), a very interesting distribution forms when AI is asked to pick a number between 1 and 100\. There is a heavy weighting toward picking the number ‘42’. Likely, this is the [Hitchhiker’s Guide to the Galaxy](https://en.wikipedia.org/wiki/Phrases_from_The_Hitchhiker%27s_Guide_to_the_Galaxy#The_Answer_to_the_Ultimate_Question_of_Life,_the_Universe,_and_Everything_is_42) effect. The number 42 is overrepresented or weighted in some way through training, resulting in a higher propensity for the LLM to choose 42.

The implications are that LLMs do not perform reasoning over data in the way that most people conceive or desire.

There is no self-reflection of its information; it does not know what it knows and what it does not. The line between hallucination and truth is simply a probability factored by the prevalence of training data and post-training processes like fine-tuning. Reliability will always be nothing more than a probability built on top of this architecture.

As such, it becomes unsuitable as a machine to find rare hidden truths or valuable neglected information. It will always simply converge toward popular narrative or data. At best, it can provide new permutations of views of existing well-known concepts, but it can not invent new concepts or reveal concepts rarely spoken about.

“You can't cache reality in some compressed lookup table. If a particular outcome was never in the training data, the model will perform a random guess which is quite limiting.”

— [Chomba Bupe](https://twitter.com/ChombaBupe/status/1781831176367312981)

Furthermore, it can never be a system for absolute dependability. Mission-critical systems that require deterministic, provably correct behavior are not something applicable to LLM automation or control. The problem is that LLMs are impressively convincing when they are wrong, which may lead to ill-advised adoption. What business wants to balance the books with a hallucinating calculator?

**Implications**:

1.  Results are probabilities defined more by data prevalence than logic or reason.

2.  It is indiscernible to what degree a LLM is reliable on a given question.

3.  Not useful to find undiscovered truths or neglected but brilliant ideas.

4.  Inability to theorize new concepts or discoveries.

It is substantially ironic that LLMs are failing at the primary use cases that are attracting [billions of investment](https://www.mindprison.cc/p/stargate-the-100-billion-hail-mary-agi), but are rather proficient at the use cases we do not desire, such as [destruction of privacy and liberty](https://www.mindprison.cc/p/ai-end-of-privacy-end-of-sanity), a [post-truth society](https://www.mindprison.cc/p/ai-accelerates-post-truth-civilization), [social manipulation](https://www.mindprison.cc/p/ai-instructed-brainwashing-effectively), the [severance of human connection](https://www.mindprison.cc/p/ai-is-transforming-us-into-npcs), [fountains of noise](https://www.mindprison.cc/p/ai-automation-to-bury-civilization), the [devaluation of meaning](https://www.mindprison.cc/p/ai-art-challenges-meaning-in-a-world), and a plethora of other [societal issues](https://www.mindprison.cc/p/ai-and-the-end-to-all-things).

* * *

Unlike much of the internet now, there is a human mind behind all the content created here at Mind Prison. I typically spend hours to days on articles including creating the illustrations for each. I hope if you find them valuable and you still appreciate the creations from the organic hardware within someone’s head that you will consider subscribing. Thank you!

* * *

*No compass through the dark exists without hope of reaching the other side and the belief that it matters …*

Mind Prison is a reader-supported publication. You can also assist by sharing.

[Share](https://www.mindprison.cc/p/the-question-that-no-llm-can-answer?utm_source=substack&utm_medium=email&utm_content=share&action=share)