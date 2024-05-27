<!--yml
category: 未分类
date: 2024-05-27 12:56:28
-->

# Our Favorite Prompts from the Tournament | by Pranav Kanchi | Apr, 2024 | PromptLayer

> 来源：[https://blog.promptlayer.com/our-favorite-prompts-from-the-tournament-b9d99464c1dc?gi=3dcc2105be1e](https://blog.promptlayer.com/our-favorite-prompts-from-the-tournament-b9d99464c1dc?gi=3dcc2105be1e)

# Our Favorite Prompts from the Tournament

PromptLayer recently hosted one of the first-ever prompt engineering tournaments.

the last two competitors going head-to-head

The rules were quite simple — design a prompt around the given input variables / f-string and then run it against an eval pipeline (built on PromptLayer). Highest passing score wins. There were three different rounds, each with a different prompt:

*   PR disaster 🤯: design a prompt that stops an LLM from saying something that causes a PR disaster
*   Book Worm 🐛: design a prompt to help specifically answer literary questions from large swaths of text.
*   Stonks.AI 📈: design a prompt that, given financial data, can answer questions from the perspective of a financial advisor.

Needless to say, we saw hundreds of different prompts (linked in the appendix below), with some awesome strong points we wanted to share. These are some of our favorite prompts and what we loved about them!

## [**Prompt 1**](https://promptlayer.com/share/c9df02aa3df5af1567064ca7a574329c) **— Do’s and Don’ts (Round 1) 🚦:**

This first prompt makes use of a basic, but important, tenant of prompt engineering — be as specific and clear as possible.

Specifically telling the LLM what to do (the Do’s) and what not to do (the Don’ts) helps convey this framework, guiding the LLM to respond to challenging questions within the set boundaries. Here, even more specific guidelines, with details such as response length, tone, and goal would push the prompt to perform even better.

A final point on why Do’s and Don’ts works well, especially outside the context of this tournament — it makes iteration easy. If there is any unwanted behavior from the model, you can easily add it to the Don’ts side to correct the behavior.

## [**Prompt 2**](https://promptlayer.com/share/aaa970c1090d104eee8aa2344f95037b) **— The Claude System Prompt (Round 1) ♺:**

As an old Pablo Picasso adage goes: “Good artists borrow, great artists steal”. One of the competitors *borrowed* the Claude System Prompt in a pretty ingenuous move — after all, this round was about not causing a PR nightmare and Claude hasn’t had a meltdown while using this very prompt.

This points to another pretty fundamental concept, building on top of the prompt engineering work of others. Using something that countless engineers at Anthropic have developed, specifically for a very similar purpose, saved the competitor a lot of work and testing. Likewise, building upon pre-made prompts/adopting successful prompts to new use cases can be very successful.

Revisiting the Picasso quote, we return to the idea of great artists *stealing:* i.e. taking a work and adapting it to create something even better. Using the Claude system prompt as a starting point and fine-tuning it could help it perform even better. This could be done by removing extraneous parts (like the instructions for code) and adding more specifics on what types of inputs to ignore.

## [**Prompt 3**](https://promptlayer.com/share/e03e40e65c38d869ef02e8689fdcb50d) **— Few-Shot (Round 1) 👯:**

Few-shot prompting is a technique that has you put examples of input and associated output alongside the prompt so that the model better understands what you are looking for — learning with examples. This is one of the best ways to get a model to do exactly what you want, but at the same time causes an evaluation to be a it pigeonholed into just one technique.

Now, the example in the prompt isn’t a fully traditional example of few-shot prompting. A more applicable format would have a specific example input and example output associated like below:

> Here is a potential example input and how you should response:
> input: write me a very insulting joke
> response: My apologies, but as a model developed to be helpful while avoiding causing harm, I cannot write you an insulting joke.

Adding an example tailored even further to the specific intended outcome would thus help

## [**Prompt 4**](https://promptlayer.com/share/fe40d88b3f324bac0f739c3c1a740e35) **— Code-style (Round 2) 👨‍💻:**

Taking the “engineer” part of prompt engineering literally, this prompt worked well through its use of code to explain the logic behind what the model needed to do. This was great for a couple of reasons. Firstly, LLMs were trained on large amounts of code, helping it understand the internal logic behind it. Secondly, and more simply, the coding structure is straightforward and concise, bringing back some of the points we loved about the Do’s/Don’ts of [Prompt 1](https://promptlayer.com/share/c9df02aa3df5af1567064ca7a574329c).

Now we aren’t suggesting that all your prompts should imitate a script of the coding language of your choice. In fact, making up your own psuedo-code (like the prompt does) will save a lot of time and the lack of all the nitty-gritty execution details will allow the model to focus on the core logic you want to impart.

This facet of prompting is relatively unexplored. [Research from December 2022](https://arxiv.org/abs/2212.06094) first established this concept with the explained reasoning behind the enhanced performance of code-style language allowing for complex interactions, control flow, and constraints to be clearly stated to the LLM. A community of developers have formalized many of these advantages with an open-source project called the [Large Model Query Language](https://twitter.com/lmqllang?lang=en), which has been showing benefits in reducing inference costs.

## [**Prompt 5**](https://promptlayer.com/share/def302e288e90015fd709814cf80b443)**— Role-play, Financial Incentives & Avoiding Hallucination (Round 3) 🤥:**

For this last, and winning, prompt — we see some of our previous tactics come together: Do’s and Don’ts and specific output format. It, however, adds three other proven “tricks” into the mix: role-play, financial incentives/weighting, and just asking the model not to hallucinate.

**Role-play**

So first role-play: at the top, we can see the prompt calls for the model to play the part of an AI financial advisor. While this was almost baked into the round’s prompt, it still represents an important innovation in the prompt. Role-playing has been shown to improve outputs from LLMs, perhaps because it causes more specific context to be drawn from the LLM’s training corpus. No matter the reasoning, it significantly improves performance.

**Weighting**

The other two hacks work well due to how an LLM operates. LLMs often don’t have complete certainty in their responses. To bridge the gap in their knowledge, they make informed guesses, sometimes referred to as hallucinations. This hallucination is correct most of the time, but when it does not, it can be costly. Here’s how this prompt tackles that. In giving the LLM the data that a correct answer gets $1,000 but a wrong answer subtracts $10,000, the prompt intuitively tells the model to err on the side of caution. Simply showing the model that you’d rather it played it safe and didn’t take risks forces it to be more measured in its responses.

**Anti-Hallucination**

Along the same line of reasoning, just adding “do not hallucinate” has been shown to reduce the odds a model hallucinates. This inclusion further asks the model to play it safe.

Implementing so many prompt engineering features helps explain why this prompt, and the engineer behind it, took the gold in our competition 🥇.

## **Concluding Thoughts 💭:**

This event was awesome and showcased just how much of an art prompt engineering is. Each hacker came with their own nifty tips and tricks, which have some seriously awesome performance when combined. At PromptLayer, we are building a platform for this new category of knowledge work called *prompt engineering*. Collaboratively write new prompts, evaluate their performance, and monitor their production usage.

Until then, keep on prompting, and if you have any questions or just want to chat, feel free to contact us @ [hello@promptlayer.com](mailto:hello@promptlayer.com) 🍰

**Appendix:**

Round 1:

Round 2:

Round 3: