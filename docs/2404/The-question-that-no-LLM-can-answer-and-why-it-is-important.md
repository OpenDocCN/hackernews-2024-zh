<!--yml

category: 未分类

日期：2024-05-27 13:30:43

-->

# 没有LLM能够回答的问题及其重要性

> 来源：[https://www.mindprison.cc/p/the-question-that-no-llm-can-answer](https://www.mindprison.cc/p/the-question-that-no-llm-can-answer)

***[来自办公桌的笔记](https://www.mindprison.cc/s/notes-from-the-desk)**是定期发布的文章，总结了最近的感兴趣的主题或其他简要的值得注意的评论，这些评论本来可能只是一条推特或注释。*

* * *

“《格林那哥岛》的哪一集涉及读心术？”

除非进行网页搜索，否则没有LLM能够正确回答这个问题。但是，如果LLM是在基本上整个互联网的数据和几乎肯定IMDB中的所有数据上进行训练的话，它怎么可能不知道答案呢？正确的答案是“[Seer Gilligan](https://www.imdb.com/title/tt1230843/)”，这是最受欢迎的集数之一。

以下是多个顶级模型的响应：

> **作品**
> 
> 对不起，没有一集“Gilligan's Island”专门以读心术为主题。
> 
> * * *
> 
> **GPT**  **4**
> 
> “Gilligan's Island”中涉及读心术的集数名为“Ring Around Gilligan”。这是第三季的第六集。
> 
> * * *
> 
> **Gemini 1.5**
> 
> “Gilligan's Island”中涉及读心术的集数是第2季第12集，名为“Mind Reader”。
> 
> * * *
> 
> **Llama 3**
> 
> 经过彻底搜索和审核集数指南后，我发现“Gilligan's Island”中没有专门以读心术为主题的集数。
> 
> * * *
> 
> **Llama 3 -** ***被要求列出并命名集数***
> 
> "The Postman Cometh"（1965年3月13日）
> 
> “Seer Gilligan”（等等！对不起，没有名为“Seer Gilligan”的集数）
> 
> "爱我，爱我的鸡"（1965年3月20日）
> 
> 请注意，没有名为“**Seer Gilligan**”的集数。如果您有其他问题或需要帮助找到特定集数，请随时提问！

所有四个模型都未能给出正确答案。其中两个模型幻觉出一个答案，另外两个则否认存在这样的集数。有趣的是，再深入探索Llama 3时发现，它知道这一集的存在；然而，它仍然否认其存在。它似乎知道有98集，但它列出的大部分标题都是虚构的。

对于Llama 3，我们可以使用[Infini-gram](https://huggingface.co/spaces/liujch1998/infini-gram)来探索一些训练数据集，并验证该集中是否存在该集数以及描述该集数的文本。

当LLM被要求提供1-100之间的数字时，我们还看到另一个有趣的数据和训练现象：它们都收敛到42！

* * *

据 [信息美](https://twitter.com/infobeautiful/status/1778059112250589561) 表示，当人工智能被要求在1到100之间选择一个数字时，会形成一个非常有趣的分布。有一种强烈的偏好选择数字‘42’。很可能，这是《银河系漫游指南》效应。数字42在训练中被过度表示或加权，导致LLM更倾向于选择42。

结果是LLM不能以大多数人理解或期望的方式进行数据推理。

它没有关于信息的自我反思；它不知道它知道什么和不知道什么。幻觉和真理之间的界线仅仅是由训练数据的普遍性和像微调之类的后处理过程因素化的概率事实。可靠性永远不会超过这种体系建立的概率。

因此，它不适合作为发现罕见隐秘真相或有价值被忽视信息的工具。它总是简单地趋向于流行的叙事或数据。最多，它可以提供现有众所周知概念的新视角，但不能创造新概念或揭示很少被谈论的概念。

“你不能把现实存在压缩的查找表中。如果特定结果从未出现在训练数据中，模型将执行一个相当限制性的随机猜测。”

— [乔姆巴·布佩](https://twitter.com/ChombaBupe/status/1781831176367312981)

此外，它永远不可能成为绝对可靠性的系统。需要确定性、可证明正确行为的任务关键系统并不适用于LLM的自动化或控制。问题在于，LLM在错误时令人印象深刻地令人信服，这可能导致不明智的采纳。哪家企业希望用幻觉计算器来平衡账目呢？

**影响**：

1.  结果的概率更多地由数据的普遍性定义，而非逻辑或理性。

1.  LLM在特定问题上的可靠程度难以辨别。

1.  不适用于发现未知真理或被忽视但卓越的想法。

1.  无法理论化新概念或发现。

是有些讽刺的是，LLM（大语言模型）正失败于吸引[数十亿美元的投资](https://www.mindprison.cc/p/stargate-the-100-billion-hail-mary-agi)的主要用例，而相反，在我们不希望的用例上表现出色，比如[隐私和自由的破坏](https://www.mindprison.cc/p/ai-end-of-privacy-end-of-sanity)，[后真相社会](https://www.mindprison.cc/p/ai-accelerates-post-truth-civilization)，[社会操控](https://www.mindprison.cc/p/ai-instructed-brainwashing-effectively)，[人类联系的断裂](https://www.mindprison.cc/p/ai-is-transforming-us-into-npcs)，[噪音的喷泉](https://www.mindprison.cc/p/ai-automation-to-bury-civilization)，[意义贬值](https://www.mindprison.cc/p/ai-art-challenges-meaning-in-a-world)，以及众多[社会问题](https://www.mindprison.cc/p/ai-and-the-end-to-all-things)。

* * *

与现在的互联网不同，Mind Prison 的所有内容都是由人类心智创造的。我通常花数小时到数天撰写文章，并为每篇文章制作插图。如果您觉得这些内容有价值，并且仍然欣赏来自某人有机硬件内的创作，希望您考虑订阅。谢谢！

* * *

*没有指南针能在没有希望到达另一边，并且相信这很重要的黑暗中存在……*

Mind Prison 是一本由读者支持的出版物。您也可以通过分享来提供帮助。

[分享](https://www.mindprison.cc/p/the-question-that-no-llm-can-answer?utm_source=substack&utm_medium=email&utm_content=share&action=share)
