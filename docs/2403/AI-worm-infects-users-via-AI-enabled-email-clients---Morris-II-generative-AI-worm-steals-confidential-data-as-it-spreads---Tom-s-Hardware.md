<!--yml
category: 未分类
date: 2024-05-27 14:35:58
-->

# AI worm infects users via AI-enabled email clients — Morris II generative AI worm steals confidential data as it spreads | Tom's Hardware

> 来源：[https://www.tomshardware.com/tech-industry/artificial-intelligence/ai-worm-infects-users-via-ai-enabled-email-clients-morris-ii-generative-ai-worm-steals-confidential-data-as-it-spreads](https://www.tomshardware.com/tech-industry/artificial-intelligence/ai-worm-infects-users-via-ai-enabled-email-clients-morris-ii-generative-ai-worm-steals-confidential-data-as-it-spreads)

A group of researchers created a first-generation AI worm that can steal data, spread malware, spam others via an email client, and spread through multiple systems. This worm was developed and successfully functions in test environments using popular LLMs. The team shared [research papers](https://sites.google.com/view/compromptmized) and published a video showing how they used two methods to steal data and affect other email clients. 

Ben Nassi from Cornell Tech, Stav Cohen from the Israel Institute of Technology, and Ron Bitton from Intuit created this worm. They named it 'Morris II' after the original Morris, the first computer worm that created a worldwide nuisance [online in 1988](https://www.fbi.gov/news/stories/morris-worm-30-years-since-first-major-attack-on-internet-110218). This worm targets AI apps and AI-enabled email assistants that generate text and images using models like [Gemini Pro](https://www.tomshardware.com/tech-industry/artificial-intelligence/google-launches-gemini-its-newest-and-most-capable-ai-model-and-a-full-frontal-assault-on-openais-gpt-4), [ChatGPT](https://www.tomshardware.com/news/chatgpt-nvidia-30000-gpus) 4.0, and LLaVA.

The worm uses adversarial self-replicating prompts. Here's how the authors describe the attack mechanism:

"The study demonstrates that attackers can insert such prompts into inputs that, when processed by GenAI models, prompt the model to replicate the input as output (replication) and engage in malicious activities (payload). Additionally, these inputs compel the agent to deliver them (propagate) to new agents by exploiting the connectivity within the GenAI ecosystem. We demonstrate the application of Morris II against GenAI-powered email assistants in two use cases (spamming and exfiltrating personal data), under two settings (black-box and white-box accesses), using two types of input data (text and images)."

You can see a concise demonstration in the video below. 

The researchers say this approach could allow bad actors to mine confidential information, including but not limited to credit card details and social security numbers.

## GenAI Leader's Response and Plans to Deploy Deterrents

Like all responsible researchers, the team reported their findings to Google and OpenAI. [*Wired*](https://www.wired.com/story/here-come-the-ai-worms/)reached out when Google refused to comment about the research, but an OpenAI spokesperson responded, telling *Wired* that, “They appear to have found a way to exploit prompt-injection type vulnerabilities by relying on user input that hasn’t been checked or filtered.” The OpenAI rep said the company is making its systems more resilient and added that developers should use methods that ensure they are not working with harmful input.

Get Tom's Hardware's best news and in-depth reviews, straight to your inbox.