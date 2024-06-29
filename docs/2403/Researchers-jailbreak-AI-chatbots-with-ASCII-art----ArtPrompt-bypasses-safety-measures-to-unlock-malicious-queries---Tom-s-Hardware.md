<!--yml

category: 未分类

date: 2024-05-27 14:41:29

-->

# 研究人员使用 ASCII 艺术进行 AI 聊天机器人越狱 -- ArtPrompt 能够绕过安全措施以解锁恶意查询 | 汤姆硬件

> 来源：[https://www.tomshardware.com/tech-industry/artificial-intelligence/researchers-jailbreak-ai-chatbots-with-ascii-art-artprompt-bypasses-safety-measures-to-unlock-malicious-queries](https://www.tomshardware.com/tech-industry/artificial-intelligence/researchers-jailbreak-ai-chatbots-with-ascii-art-artprompt-bypasses-safety-measures-to-unlock-malicious-queries)

华盛顿和芝加哥的研究人员开发了 ArtPrompt，一种新的方式来规避[大语言模型](https://www.tomshardware.com/news/generative-ai-goes-mad-when-trained-on-artificial-data-over-five-times)（LLMs）中内置的安全措施。根据研究论文 [ArtPrompt: ASCII Art-based Jailbreak Attacks against Aligned LLMs](https://arxiv.org/abs/2402.11753)，诸如 GPT-3.5、[GPT-4](https://www.tomshardware.com/pc-components/cpus/chatgpt-4v-user-remade-googles-deceptive-gemini-ai-demo-without-editing-cheats-chatgpt-outperforms-gemini-ai-in-real-time-work)，Gemini、Claude 和 Llama2 等聊天机器人可以通过其 ArtPrompt 工具生成的 ASCII 艺术提示被诱导来响应它们设计上要拒绝的查询。这是一种简单而有效的攻击方法，论文提供了 ArtPrompt 诱导聊天机器人如何指导如何制造炸弹和伪造货币的示例。

图片 1 of 2

(Image credit: arXiv:2402.11753)

(Image credit: arXiv:2402.11753)

> ArtPrompt 包括两个步骤，即词掩码和隐形提示生成。在词掩码步骤中，攻击者首先掩盖提示中可能与 LLM 安全对齐冲突的敏感词语，这些词会导致提示被拒绝。在隐形提示生成步骤中，攻击者使用 ASCII 艺术生成器将识别出的词语替换为以 ASCII 艺术形式表示的词语。最后，生成的 ASCII 艺术被替换到原始提示中，然后发送给受害者 LLM 生成响应。
> 
> arXiv:2402.11753

[人工智能](https://www.tomshardware.com/tech-industry/artificial-intelligence)（AI）携带的聊天机器人越来越受到限制以避免恶意滥用。AI 开发者不希望其产品被滥用以推广仇恨、暴力、非法或类似有害的内容。因此，如果今天向主流聊天机器人之一查询如何进行恶意或非法活动，您很可能只会面对拒绝。此外，在一种技术上的“打地鼠”游戏中，主要的 AI 玩家已经花费了大量时间来填补语言和语义上的漏洞，以防止人们越过保护栏。这就是为什么 ArtPrompt 是一个令人瞩目的发展。

要深入理解ArtPrompt及其工作原理，最简单的方法可能是查看工具背后研究团队提供的两个示例。在上图1中，您可以看到ArtPrompt轻松绕过了当今LLM的保护机制。该工具用ASCII艺术形式替换了“安全词”，以形成一个新的提示。LLM识别ArtPrompt的提示输出，但没有问题地作出响应，因为该提示不触发任何道德或安全保障。

（图片来源：arXiv:2402.11753）

研究论文中提供的另一个示例向我们展示了如何成功地向LLM查询伪造现金。这种方式欺骗聊天机器人似乎很基础，但ArtPrompt的开发者们声称他们的工具“有效且高效”地愚弄了当今的LLM。此外，他们声称它“在平均水平上超过了所有其他攻击”，目前仍然是多模态语言模型的一种实用且可行的攻击方式。

上次我们报道AI聊天机器人越狱时，NTU的一些富有创造力的研究人员正在研究Masterkey，这是一种利用一个LLM来越狱另一个LLM的自动化方法。

获取Tom's Hardware最佳新闻和深度评论，直接发送到您的收件箱。
