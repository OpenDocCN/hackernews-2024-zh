<!--yml
category: 未分类
date: 2024-05-29 12:42:46
-->

# Researchers gave AI an 'inner monologue' and it massively improved its performance | Live Science

> 来源：[https://www.livescience.com/technology/artificial-intelligence/researchers-gave-ai-an-inner-monologue-and-it-massively-improved-its-performance](https://www.livescience.com/technology/artificial-intelligence/researchers-gave-ai-an-inner-monologue-and-it-massively-improved-its-performance)

Giving artificial intelligence (AI) systems an "inner monologue" makes them considerably better at reasoning, new research shows.

The method trains AI systems to think before they respond to prompts, just as many people consider what we should say next before we speak. This is different from the way scientists have trained mainstay AI chatbots, like ChatGPT, which don't "think" about what they write or anticipate different possibilities for the next steps in a conversation.

Dubbed "Quiet-STaR," the new method instructs an AI system to generate many inner rationales in parallel before responding to a conversational prompt. When the AI answers prompts, it generates a mixture of these predictions with and without a rationale, printing the best answer — which can be verified by a human participant depending on the nature of the question.

Finally, it learns by discarding rationales that proved incorrect. In effect, the training method gives AI agents the capacity to anticipate future conversations and learn from ongoing ones.

**Related:** [**AI singularity may come in 2027 with artificial 'super intelligence' sooner than we think, says top scientist**](https://www.livescience.com/technology/artificial-intelligence/ai-agi-singularity-in-2027-artificial-super-intelligence-sooner-than-we-think-ben-goertzel)

The researchers applied the Quiet-STaR algorithm to Mistral 7B, an open-source large language model (LLM), and posted the results March 14 to the pre-print database [arXiv](https://arxiv.org/pdf/2403.09629.pdf). (The paper has not yet been peer-reviewed.)

The Quiet-STaR-trained version of Mistral 7B scored 47.2% on a reasoning test versus 36.3% before any training. It still flunked a school math test, earning a score of 10.9%. But that was nearly double the starting score of 5.9% in the vanilla version.

Get the world’s most fascinating discoveries delivered straight to your inbox.

Models like ChatGPT and Gemini are built from neural networks — collections of machine learning algorithms arranged in a way that mimics the structure and learning patterns of the [human brain](https://www.livescience.com/29365-human-brain.html). However, systems built using this architecture are abysmal at common sense reasoning or contextualization — and AI chatbots do not have genuine "understanding."

Past attempts to improve the reasoning capabilities of LLMs have been highly domain-specific and could not be applied to different types of AI models. 

The self-taught reasoner (STaR) algorithm, which the researchers used as a basis for their work, is one example of such a training algorithm — but is held back by these limitations.

The scientists who developed Quiet-STaR named it that because the principles of STaR can be applied quietly in the background and generally over several different types of LLM, independent of the original training data. Now they want to investigate how techniques like theirs can reduce the gap between neural network-based AI systems and human-like reasoning capabilities.