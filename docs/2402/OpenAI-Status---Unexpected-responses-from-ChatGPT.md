<!--yml

category: 未分类

date: 2024-05-29 13:18:35

-->

# OpenAI 状态 - ChatGPT 出现意外响应

> 来源：[https://status.openai.com/incidents/ssg8fh7sfyz3](https://status.openai.com/incidents/ssg8fh7sfyz3)

在2024年2月20日，为了优化用户体验而引入的一个bug影响了模型处理语言的方式。

LLMs根据概率部分随机抽样单词生成响应。它们的“语言”由映射到标记的数字组成。

在这种情况下，问题出在模型选择这些数字的步骤上。类似于迷失在翻译中，模型选择了稍微错误的数字，导致生成毫无意义的词序列。更具体地说，推断核产生了在某些GPU配置下使用时的错误结果。

在确定此事件原因后，我们推出了修复措施，并确认问题已解决。
