<!--yml
category: 未分类
date: 2024-05-27 13:13:00
-->

# WizardLM 2

> 来源：[https://wizardlm.github.io/WizardLM2/](https://wizardlm.github.io/WizardLM2/)

To present a comprehensive overview of the performance of WizardLM-2, we conduct both human and automatic evaluations between our models and diverse baselines. As indicated in the following main experimental results, WizardLM-2 demonstrates highly competitive performance compared to those leading proprietary works and consistently outperforms all the existing state-of-the-art opensource models. More associated details and thinking will be presented in our upcoming paper.

**Human Preferences Evaluation**
We carefully collected a complex and challenging set consisting of real-world instructions, which includes main requirements of humanity, such as writing, coding, math, reasoning, agent, and multilingual. We perform a blind pairwise comparison between WizardLM-2 and baselines. To each annotator, responses from all models are presented, which are randomly shuffled to hide their sources. We report the win:loss rate without tie:

*   WizardLM-2 8x22B is just slightly falling behind GPT-4-1106-preview, and significantly stronger than Command R Plus and GPT4-0314.
*   WizardLM-2 70B is better than GPT4-0613, Mistral-Large, and Qwen1.5-72B-Chat.
*   WizardLM-2 7B is comparable with Qwen1.5-32B-Chat, and surpasses Qwen1.5-14B-Chat and Starling-LM-7B-beta.

Through this human preferences evaluation, WizardLM-2's capabilities are very close to the cutting-edge proprietary models such as GPT-4-1106-preview, and significantly ahead of all the other open source models.