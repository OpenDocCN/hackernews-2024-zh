<!--yml

类别：未分类

日期：2024-05-27 14:35:58

-->

# AI蠕虫通过AI增强的电子邮件客户端感染用户 — Morris II生成式AI蠕虫在传播过程中窃取机密数据 | Tom's Hardware

> 来源：[https://www.tomshardware.com/tech-industry/artificial-intelligence/ai-worm-infects-users-via-ai-enabled-email-clients-morris-ii-generative-ai-worm-steals-confidential-data-as-it-spreads](https://www.tomshardware.com/tech-industry/artificial-intelligence/ai-worm-infects-users-via-ai-enabled-email-clients-morris-ii-generative-ai-worm-steals-confidential-data-as-it-spreads)

一组研究人员创建了第一代AI蠕虫，它能够窃取数据、传播恶意软件、通过电子邮件客户端发送垃圾邮件，并在多个系统中传播。该蠕虫是使用流行的LLMs在测试环境中开发并成功运行的。团队分享了[研究论文](https://sites.google.com/view/compromptmized)，并发布了一段视频，展示他们如何使用两种方法窃取数据并影响其他电子邮件客户端。

本·纳西来自康奈尔科技，斯塔夫·科恩来自以色列理工学院，罗恩·比顿来自Intuit共同创建了这种蠕虫。他们将其命名为“Morris II”，以纪念最初的莫里斯蠕虫，这是1988年在网络上制造全球性麻烦的第一种计算机蠕虫。该蠕虫针对使用模型如[Gemini Pro](https://www.tomshardware.com/tech-industry/artificial-intelligence/google-launches-gemini-its-newest-and-most-capable-ai-model-and-a-full-frontal-assault-on-openais-gpt-4)、[ChatGPT](https://www.tomshardware.com/news/chatgpt-nvidia-30000-gpus) 4.0和LLaVA的AI应用程序和AI增强电子邮件助手，这些助手生成文本和图像。

蠕虫使用对抗性自我复制提示。以下是作者描述的攻击机制：

“该研究表明，攻击者可以将这些提示插入到输入中，当由GenAI模型处理时，提示模型将输入复制为输出（复制）并参与恶意活动（有效负载）。此外，这些输入迫使代理通过利用GenAI生态系统内的连接性将它们（传播）传递给新代理。我们展示了Morris II针对GenAI驱动的电子邮件助手在两个使用案例（垃圾邮件和外泄个人数据）下的应用，采用两种设置（黑盒和白盒访问），使用两种类型的输入数据（文本和图像）。"

您可以在下面的视频中看到简明的演示。

研究人员表示，这种方法可能允许不良行为者挖掘包括但不限于信用卡详细信息和社会安全号码在内的机密信息。

## GenAI领导人的回应和部署威慑计划

与所有负责任的研究人员一样，团队将他们的发现报告给了Google和OpenAI。[*Wired*](https://www.wired.com/story/here-come-the-ai-worms/)在Google拒绝评论该研究后联系了他们，但OpenAI的一位发言人回应说，"他们似乎已经找到了一种利用提示注入类型漏洞的方法，依赖于未经检查或过滤的用户输入。" OpenAI的代表表示公司正在使其系统更具韧性，并补充说开发者应该采用确保他们不使用有害输入的方法。

获取[Tom's Hardware](https://www.tomshardware.com/)最新的新闻和深度评测，直接送到您的收件箱。
