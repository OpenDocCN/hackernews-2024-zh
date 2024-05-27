<!--yml
category: 未分类
date: 2024-05-27 13:28:34
-->

# Developing Rapidly with Generative AI

> 来源：[https://discord.com/blog/developing-rapidly-with-generative-ai](https://discord.com/blog/developing-rapidly-with-generative-ai)

## **Prototyping AI applications: From Idea to MVP**

The product requirements we define then play into our selection of which off-the-shelf LLM we'll use for our prototype. We generally lean towards picking more advanced commercial LLMs to quickly validate our ideas and obtain early feedback from users. Although they may be expensive, the general idea is that if problems can't be adequately solved with state-of-the-art foundational models like GPT-4, then more often than not, those problems may not be addressable using current generative AI tech. If an off-the-shelf LLM can address our problem, then we can step into the learning stage and concentrate on iterating on our product rather than diverting engineering resources towards building and maintaining machine learning infrastructure.

### Evaluating Prompts

The key step at this stage is to create the right prompt. We start with a basic prompt that tells ChatGPT (or whatever LLM we selected for our prototype) what we want it to do. Then, we make adjustments to this prompt, changing the wording to make the task clearer. However, after a lot of adjustments, it's often difficult to tell if these changes are actually improving our results. That's where evaluating the prompts becomes crucial. By using metrics to guide our changes, we know we are moving the needle on the quality of our results.

To do this, we employ a technique known as [AI-assisted evaluation](https://arize.com/blog-course/llm-evaluation-the-definitive-guide/), alongside traditional metrics for measuring performance. This helps us pick the prompts that lead to better quality outputs, making the end product more appealing to users. AI-assisted evaluation uses best-in-class LLMs (like GPT-4) to automatically critique how well the AI's outputs match what we expected or how they score against a set of criteria. This method uses GPT-4 in a way that’s similar to the critic model found in the [actor-critic algorithm](https://www.tensorflow.org/tutorials/reinforcement_learning/actor_critic) in reinforcement learning where a separate model is used to evaluate how well the model used for inference performed. Automating evaluation allows us to quickly see what's working well and what needs to be tweaked in our prompts, without having to manually check everything. When evaluating, we design prompts that ask for simple yes or no answers or rate the outputs on a scale, making the evaluation process straightforward.

AI-assisted evaluation consists of 2 separate prompts: one for your task and another to evaluate your results. The task prompt is passed to the inference model whereas the critic prompt is passed to the more advanced critic model.

### Launch and Learn

Once we are sufficiently confident in the quality of the results our prompt generates, we roll out a limited release (e.g. A/B test) of our product and observe the system’s performance in situ. The exact metrics we use depend on the application — our main goal is to understand how users use the feature and quickly make improvements to better meet their needs. For internal applications, this might mean measuring efficiency and sentiment. For consumer-facing applications, we similarly focus on measures of user satisfaction - direct user feedback, user engagement measures, etc.  This feedback is critical to identify areas for improvement, including highlighting incorrect answers or instances where LLM hallucinations might be causing a strange user experience.

Beyond user satisfaction, we also pay attention to system health metrics, such as response speed (latency), throughput (tokens per second), and error rates. LLMs sometimes have trouble generating output in a consistently structured format, which is crucial for minimizing data parsing errors and ensuring the output is robustly usable in our services. Insights here can inform how much post-hoc processing might be needed to fully productionize this capability at scale.

Keeping an eye on costs is equally important for understanding how much we will spend when we fully scale up the feature. We look at how many tokens per second we're using in our initial limited release to predict the costs of a complete launch if we were to use the same technology that’s powering our prototype. 

All of the above information is critical to understanding if our product is working as intended and providing value to users.  If it is, then we can proceed to the next step: deploying at scale.  If not, then we look to take our learnings, iterate on the system, and try again.