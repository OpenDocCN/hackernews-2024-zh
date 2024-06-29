<!--yml

类别：未分类

日期：2024-05-29 12:42:46

-->

# 研究人员为AI提供了一种“内心独白”，显著提高了其性能 | Live Science

> 来源：[https://www.livescience.com/technology/artificial-intelligence/researchers-gave-ai-an-inner-monologue-and-it-massively-improved-its-performance](https://www.livescience.com/technology/artificial-intelligence/researchers-gave-ai-an-inner-monologue-and-it-massively-improved-its-performance)

给予人工智能（AI）系统一个“内心独白”使它们在推理方面显著提升，新研究显示。

该方法训练AI系统在回答提示之前先思考，就像许多人在讲话之前考虑接下来应该说什么一样。这与科学家们训练的像ChatGPT这样的主流AI聊天机器人的方式不同，它们不“思考”自己写的内容或预测对话下一步可能出现的不同可能性。

新方法名为"Quiet-STaR"，指导AI系统在回答对话提示之前并行生成多个内部推理。当AI回答提示时，它生成这些预测的混合，有些带有推理，有些没有，输出最佳答案 —— 可以由人类参与者根据问题的性质进行验证。

最后，它通过丢弃证明不正确的推理来学习。实际上，该训练方法赋予了AI代理能力，使其能够预测未来的对话并从进行中的对话中学习。

**相关文章：**[**AI的奇点可能会在2027年到来，人工“超级智能”比我们想象的更早，顶级科学家称**](https://www.livescience.com/technology/artificial-intelligence/ai-agi-singularity-in-2027-artificial-super-intelligence-sooner-than-we-think-ben-goertzel)

研究人员将Quiet-STaR算法应用于Mistral 7B，一个开源大型语言模型（LLM），并于3月14日将结果发布到预印本数据库[arXiv](https://arxiv.org/pdf/2403.09629.pdf)。 （该论文尚未经同行评审。）

Quiet-STaR训练版本的Mistral 7B在推理测试中得分为47.2%，而之前未经任何训练时仅为36.3%。它仍然在学校数学测试中不及格，得分为10.9%。但这几乎是香草版本起始得分5.9%的两倍。

将世界上最令人着迷的发现直接发送到您的收件箱。

像ChatGPT和Gemini这样的模型是由神经网络构建的 —— 这些机器学习算法的集合排列方式模仿了[人脑](https://www.livescience.com/29365-human-brain.html)的结构和学习模式。然而，使用此架构构建的系统在常识推理或语境化方面表现不佳 —— AI聊天机器人没有真正的"理解"。

以往改善LLM推理能力的尝试高度特定于领域，并且无法应用于不同类型的AI模型。

研究人员使用的自学推理算法（STaR），作为他们工作的基础之一，就是这种训练算法的一个例子 — 但受到这些限制的制约。

开发Quiet-STaR的科学家们将其命名为这样是因为STaR的原则可以在背景中悄悄应用，并且通常可以在多种不同类型的LLM上独立于原始训练数据。现在，他们希望调查像他们这样的技术如何缩小基于神经网络的人工智能系统与类人推理能力之间的差距。
