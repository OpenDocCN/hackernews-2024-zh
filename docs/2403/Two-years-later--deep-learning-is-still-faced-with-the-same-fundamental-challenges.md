<!--yml

类别：未分类

日期：2024-05-27 14:50:57

-->

# 两年后，深度学习仍然面临着同样的基本挑战

> 来源：[https://garymarcus.substack.com/p/two-years-later-deep-learning-is](https://garymarcus.substack.com/p/two-years-later-deep-learning-is)

两年前的今天，我发表了[我最臭名昭著的文章](https://nautil.us/deep-learning-is-hitting-a-wall-238440/)：

我认为很少有人读过它（除了标题），但很多人表达了意见。在Twitter上，当时称为这个名字，数百人喜欢它。成千上万人恨它。一个月后，Dall-E出现后，萨姆·阿尔特曼嘲笑了它（请注意巧妙的艺术作品！）

文章[经得起时间的考验吗](https://nautil.us/deep-learning-is-hitting-a-wall-238440/)？

一方面，显而易见的进展已经存在，GPT-4、Sora、Claude-3，消费者采用速度之快让人瞠目。另一方面，这并不是文章讨论的重点。文章讨论的是普遍智能的障碍及为何仅仅扩展规模不足够。

让我们考虑一些我所说的，按照文章的结构：

+   2016年，辛顿预测深度学习将取代放射科医师。我写道“快进到2022年，还没有一个放射科医师被取代”。 **依然如此**

+   “至少目前，人类和机器互补彼此的优势。” **依然如此**。

+   “很少有领域像人工智能一样充满了夸大和豪言壮语。它从一个时代飞跃到另一个时代的短暂热潮中，总是承诺着月球，但只有偶尔兑现。” **依然如此**。

+   “2020年11月，辛顿在[《麻省理工技术评论》](https://www.technologyreview.com/2020/11/03/1011616/ai-godfather-geoffrey-hinton-deep-learning-will-do-everything/)中说“深度学习将能够做任何事情”。我对此持怀疑态度。 **我对此持怀疑态度至今如此，可以说至今依然未决。**

+   “我们离能真正理解人类语言的机器还有很长的路要走”。**依然如此**，尽管有人认为有些表面理解。

+   “我们远未达到科幻管家罗西机器人的日常智能水平，她不仅能解释各种各样的人类请求，还能够实时安全地执行它们。” **依然如此**

+   “埃隆·马斯克最近表示，他希望建造的新型人形机器人Optimus将来会比汽车行业还要大。”我表示怀疑。 **现在还为时过早，但可以确定的是，近期内不会有任何人形机器人成为一个大生意。**

+   “Google最新的语言贡献是一个名为Lamda的系统，如此飘忽不定，以至于其作者之一最近承认它很容易产生“[废话](https://medium.com/@blaisea/do-large-language-models-understand-us-6f881d6d8e75)。”⁵“ **依然如此。**

+   “In time we will see that deep learning was only a tiny part of what we need to build if we’re ever going to get trustworthy AI” **猜测仍未解决**，但请注意，像我建议的那样，像RAG这样的技术导入了符号技术。我们离可信的人工智能仍有很长的路要走。

+   “Deep learning, which is fundamentally a technique for recognizing patterns, is at its best when all we need are rough-ready results, where stakes are low and perfect results optional“ **仍然成立**

+   “Current deep-learning systems frequently succumb to stupid errors.” **仍然成立**。

+   “Another team briefly considered turning GPT-3 into automated suicide counselor chatbot, but found that the system was prone to [problematic] exchanges”. **很可能仍然是一个问题，尤其是在可能没有经过精心设计护栏的开源机器人中**；至少有一个与聊天机器人相关的死亡事件。

+   “GPT-3 is prone to producing toxic language, and promulgating misinformation” **仍然成立，虽然在某些方面取得了进展。 (**毒性语言可以很容易地通过越狱引发，但在普通使用中不太可能；虽然存在进展，但误信息仍然普遍。)

+   “new effort by OpenAI to solve these problems wound [still] fabricat[ing] authoritative nonsense ” **仍然成立**。

+   “In 2020, Jared Kaplan and his collaborators at OpenAI [suggested](https://arxiv.org/abs/2001.08361) that there was a set of “scaling laws” for neural network models of language; they found that the more data they fed into their neural networks, the better those networks performed.^(10) The implication was that we could do better and better AI if we gather more data and apply deep learning at increasingly large scales.” **无可否认，扩展确实有所帮助，但并没有解决我上述指出的任何问题**。

+   “There are serious holes in the scaling argument. To begin with, the measures that have scaled have not captured what we desperately need to improve: genuine comprehension…. Scaling the measures Kaplan and his OpenAI colleagues looked at—about predicting words in a sentence—is not tantamount to the kind of deep comprehension true AI would require” **仍然成立，并且越来越受到认可**。

+   “scaling laws aren’t universal *laws* like gravity but rather mere observations that might not hold forever, much like Moore’s law, a trend in computer chip production that held for decades but arguably [began to slow](https://www.nytimes.com/2015/09/27/technology/smaller-faster-cheaper-over-the-future-of-computer-chips.html) a decade ago.” **仍然成立**，最近由Altman公开承认，他指出我们在达到GPT-5之前不会真正知道它能做到什么。

+   公司富有魅力的CEO山姆·阿尔特曼写了一篇充满胜利感的[博文](https://moores.samaltman.com/)，吹嘘“一切的摩尔定律”，声称我们只需再过几年就能拥有“可以思考”的计算机，“阅读法律文件”，并（回应IBM沃森）“提供医疗建议”。或许是对的，但也或许不是。” **Pending/still true**。两年后，我们仍未拥有任何可靠版本。

+   “如果规模化不能带来安全的自动驾驶，那么数百亿美元的投资可能会变成一场空。” **Pending/still true**。结果显示超过1000亿美元，仍无商业推广，测试仍受限。多家公司失败或拒绝。

+   神经符号可能是一个有希望的替代方案。**Pending/still true**，DeepMind刚刚在神经符号系统AlphaGeometry上发表了一篇优秀的自然论文。

+   本文隐含的一点是：规模化可能不能解决上述问题。**Pending/Still true**，像[比尔·盖茨](https://indianexpress.com/article/technology/artificial-intelligence/bill-gates-feels-generative-ai-is-at-its-plateau-gpt-5-will-not-be-any-better-8998958/)、[德米斯·哈萨比斯](https://www.wired.com/story/deepmind-ceo-demis-hassabis-interview-artificial-intelligence-scale/)、[杨立昆](https://x.com/ylecun/status/1621805604900585472?s=20)和[山姆·阿尔特曼](https://www.wired.com/story/openai-ceo-sam-altman-the-age-of-giant-ai-models-is-already-over/)都意识到可能即将出现停滞期。哈萨比斯上个月在回应2022年文章的核心观点时说道：

§

这并不意味着我们最终不会有新的创新，某种程度上。或者AGI是不可能的。

但我仍认为我们需要一个范式转变。越来越多的迹象表明单靠LLM并非通向AGI的答案 — 这正是我所主张的。。

总体而言，我认为该文章说得一针见血。两年后，几乎没有什么我会改变，除了更新一些例子，稍微缓和标题，澄清某些方向的进展并不意味着所有方向的进展。我肯定会提出同样的担忧。

结尾，我引用最后一段，同样仍然属实。

*在伦理和计算中的所有挑战，以及从语言学、心理学、人类学和神经科学等领域所需的知识，而不仅仅是数学和计算机科学，将需要一个村庄来培养AI。我们永远不应忘记，人类大脑或许是已知宇宙中最复杂的系统；如果我们要构建大致相等的东西，开放而合作的态度将是关键。

[分享](https://garymarcus.substack.com/p/two-years-later-deep-learning-is?utm_source=substack&utm_medium=email&utm_content=share&action=share)

***加里·马库斯** 仍然认为该领域大多数时间在追逐登月的阶梯。*
