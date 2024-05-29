<!--yml
category: 未分类
date: 2024-05-27 14:34:17
-->

# JetBrains' unremovable AI assistant prompts customer outcry • The Register

> 来源：[https://www.theregister.com/2024/02/01/jetbrains_unremovable_ai_assistant/](https://www.theregister.com/2024/02/01/jetbrains_unremovable_ai_assistant/)

JetBrains introduced an AI assistant in December to help programmers write code. Now the biz is trying to figure out how to allow its customers to get rid of it.

"JetBrains AI Assistant is similar to GitHub Copilot, but it’s deeply integrated into JetBrains development environments (IDEs), code editors, and other products," the software development tool maker, based in the Czech Republic, said at the time.

While neural-network-powered services have been widely hyped, they come with baggage – concerns about the security, legal risk, privacy, and ethics of large language models remain largely unresolved.

Google, Microsoft, and OpenAI have opted to offer indemnify customers under certain conditions to reassure those worried about potential legal claims arising from generative AI.

But not every outfit offering AI services can make this bet and in some cases [there's a cost](https://www.theregister.com/2023/12/21/artificial_intelligence_is_a_liability/). For example, Zoom last August had to clarify that it would not use people's chats and calls for AI training despite [a change](https://www.theregister.com/2023/08/08/zoom_ai_legalese_update/) to its terms of service that allowed that.

Some JetBrains customers feel strongly about AI Assistant and really don't want the plugin to be present in their JetBrains applications at all, whether that's due to corporate policies that are incompatible with AI Assistant or other concerns. But because the plugin code has been "deeply integrated," removal has proven complicated.

More than dozen threads on JetBrains' YouTrack issue board have been posted seeking a way to delete, uninstall, or otherwise excise the AI Assistant plugin since it debuted.

A [thread](https://youtrack.jetbrains.com/issue/LLM-1760/Can-not-remove-Jetbrains-AI-Assistant-plugin-completely) titled "Provide the possibility to remove a plugin completely from the system" makes the case for why one might not wish to have this plugin baked into the company's developer tools, such as the [PyCharm](https://www.jetbrains.com/help/pycharm/ai-assistant.html), [IntelliJ IDEA](https://www.jetbrains.com/help/idea/ai-assistant.html), and [other](https://www.jetbrains.com/help/rider/AI_Assistant.html) applications.

Software developers posting to the support forum raise a number of concerns, calling the plugin "[bloatware](https://youtrack.jetbrains.com/issue/LLM-1760/Can-not-remove-Jetbrains-AI-Assistant-plugin-completely#focus=Comments-27-8775047.0-0)," [a risk to corporate intellectual property](https://youtrack.jetbrains.com/issue/LLM-1760/Can-not-remove-Jetbrains-AI-Assistant-plugin-completely#focus=Comments-27-8710484.0-0), a security issue, an annoyance, and a breach of trust.

"I just want to make it clear that I cannot use this product at the company I work for because security will not allow for a by-default AI implementation to be a part of the product," [wrote](https://youtrack.jetbrains.com/issue/LLM-1973/Provide-the-possibility-to-remove-a-plugin-completely-from-the-system#focus=Comments-27-8951378.0-0) an individual posting under the name Rusty Deaton.

Another developer posting under the name Marcus Grenängen [wrote](https://youtrack.jetbrains.com/issue/LLM-1760/Can-not-remove-Jetbrains-AI-Assistant-plugin-completely#focus=Comments-27-8766615.0-0), "This is unacceptable. It should not bundle something by default that can send off the entire code base to God knows where. It's a real bad move JetBrains and I will not renew any subscriptions after this. It's a breach of trust that can't be undone."

### 'Dubious'

Yet another developer posting under the name Alan Burlison [opined](https://youtrack.jetbrains.com/issue/LLM-1760/Can-not-remove-Jetbrains-AI-Assistant-plugin-completely#focus=Comments-27-8753176.0-0), "The fact that this service is AI based and that you don't know the provenance of the code it provides to you is a huge issue on its own, due to the dubious licensing/IP/legal situation it creates. But the biggest issue is not because it's AI, it's because it is leaking the user's company's IP to who knows where, potentially to direct competitors."

In a statement to *The Register*, Matt Ellis, developer advocate at JetBrains, tried to address misapprehensions and to reassure to those alarmed by AI Assistant's persistence.

"Although the AI Assistant plugin is bundled, and the plugin itself is enabled, there is no AI functionality enabled by default, and no data is sent off-machine without your consent," said Ellis, echoing similar replies posted to some of the discussion threads. "You have to log in, accept the data policies and either purchase a subscription or start a trial."

> There is no AI functionality enabled by default, and no data is sent off-machine without your consent

"Any data sent to the AI service is not used for training. We also have ML support for ranking and completion, and this is provided by on-device models and no data is sent off device."

Ellis said JetBrains has a lot of customers who are happy with the bundled plugin and so the biz is looking into how its AI helper can be both bundled and fully removable.

"The problem with removing a bundled plugin is that it can break application signatures and cause problems with updates," explained Ellis.

Asked what portion of JetBrains' customers have objected to AI Assistant, a spokesperson said, "We can't say for sure how well these opinions represent the position of all of our customers. We understand the strength of feeling, and we're listening to feedback from all parts of our community, including the many happy individuals and enterprises that are using the AI Assistant." ®