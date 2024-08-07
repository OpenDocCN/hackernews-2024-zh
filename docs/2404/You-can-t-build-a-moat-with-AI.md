<!--yml

category: 未分类

date: 2024-05-27 13:06:27

-->

# 你不能仅凭AI来构建壕沟

> 来源：[https://generatingconversation.substack.com/p/you-cant-build-a-moat-with-ai](https://generatingconversation.substack.com/p/you-cant-build-a-moat-with-ai)

区分AI应用是一个热门话题，也很困难。一切都只是RAG应用吗？（不是。）如果一家公司在一个领域擅长RAG，这是否会完美地转移到另一个领域？（也许！我们不确定。）所有的AI应用最终会全部由OpenAI构建吗？（很可能不会。）

对于初创公司来说，管理技术和市场去差异化是一个永恒的问题。有些人认为一切都在技术上，而其他人认为一切都在执行上。无论情况如何，我们清楚地认识到，仅仅依靠LLMs是无法实现真正的差异化的。**真正的差异化在于你为模型提供的数据。**

起初这可能听起来令人惊讶；毕竟，正是LLMs引发了当前的AI炒作周期。如果它们不重要，那么我们在这里都在做什么呢？

LLMs are obviously incredibly powerful, but we believe the core models and their use will effectively be commoditized across competitive products in the same space. In that world, having a clever use of LLMs ([尤其是如果你依赖于单一的LLM调用](https://generatingconversation.substack.com/publish/posts/detail/142618146)!) is simply not going to cut it. Instead, you’ll need to think carefully about what data you’re using to build your application. Here’s why:

**每个人都在使用相同的模型。** 无论你认为最好的LLM是什么 — 我们大多数人认为是GPT-4 — 对于每个主要任务类别，可能有一个模型超越其竞争对手。如果你必须打赌，那个模型对于大多数用例可能仍然是GPT-4，但克劳德追赶速度很快。今天每个人都可以轻松切换模型，并尝试对他们的应用程序来说最有效的方法。这意味着，无论你正在构建什么，通过选择正确的模型获得的优势都是微小的。

偏离常规也是危险的 —— 它可能在某些情况下给你意外的好答案，但在其他情况下给你意外的坏答案。当许多AI初创公司努力建立信任时，这是一个冒险的提议，因此，大多数团队默认回到使用安全的模型。这意味着，你输入模型的内容比以往任何时候都更重要。

**而且你的提示并不是知识产权。** 也许你会觉得你应用程序的提示或提示模板是一种良好的区分形式。毕竟，你的顶级工程团队已经投入了数天来调整它们，以确保具有正确的响应特性、语调和输出风格。当然，将你的提示提供给竞争对手可能会加速他们的进展，但任何优秀的工程团队都将迅速找到正确的变更。主要原因在于，通过正确的评估数据进行实验是迅速而简单的 —— 尝试新的提示模板几乎和编写它一样简单。真正需要的只是一点耐心、一些创意和额外的Azure OpenAI信用点。

今天你的提示可能更好，但我们几乎可以保证这种优势不会持续下去。你必须在其他方面寻找。

**因此（一如既往）关键在于数据。** 如果模型和提示被商品化，那么区分你的AI应用程序的唯一剩下的就是你输入到LLMs中的数据。**幸运的是，数据确实能够产生巨大的差异**。任何你构建的应用程序 —— 尤其是如果你正在与企业合作 —— 都需要利用客户的数据来驱动你的应用程序的响应和决策。每个团队对数据的使用方式和偏好都会略有不同。

LLMs的好处在于它们能很好地处理任意、非结构化的数据。但问题在于，你的客户有大量的任意、非结构化的数据。其中一些可能是绝对宝藏；而一些则完全无用。作为应用程序构建者，你需要找出如何利用这些数据。

**当然…… 垃圾进，垃圾出。** 就像任何AI模型（或任何系统？）一样，LLMs也是垃圾进，垃圾出。这对你找到并使用正确数据的能力施加了巨大的负担。作为参考，我们RunLLM团队在上个季度大约70%的工程周期中都在进行数据工程工作 —— 从数据摄取预处理到实施混合搜索和重新排序结果。每当我们收到客户投诉或请求时，通常的第一解决方案是改进数据工程流程。

结果是一个相当复杂的数据流水线，可以考虑客户的优先事项，处理各种数据类型，并为下游的复杂推理任务生成元数据。这绝不是凭借少量实验就能复制的东西。它直接受到获取实际客户反馈的驱动。其结果使我们能够始终通过简单、扎实的响应和最少的臆想来区分自己。

* * *

我们坚信，今天AI应用程序的护城河在于数据和数据工程。在某个时候，定制LLMs的过程可能会变得如此快速和简单，以至于我们都将返回构建自己的模型。但现在情况并非如此。

这也意味着，您不需要成为人工智能天才才能成功构建应用程序。事实上，过度关注人工智能可能会在某个时刻对您的差异化产生不利影响。通过深思熟虑的软件工程和对客户数据的关注，您将逐渐建立起一道护城河。
