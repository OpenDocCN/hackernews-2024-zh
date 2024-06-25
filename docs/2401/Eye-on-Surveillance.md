<!--yml

类别：未分类

日期：2024-05-27 14:28:09

-->

# Eye on Surveillance

> 来源：[`eyeonsurveillance.org/blog/rag-for-new-orleans-city-council-transparency`](https://eyeonsurveillance.org/blog/rag-for-new-orleans-city-council-transparency)

[Sawt](https://sawt.us) 是一个由新奥尔良市议会会议录制训练的 [开源](https://github.com/eye-on-surveillance/sawt) AI 助手。如今，任何居民都可以问 Sawt 有关议会正在进行的事情的问题。我们的长期目标是开发一个道德的、社区控制的大型语言模型（LLM）。

如果你住在新奥尔良，考虑参加我们的年度会议。你可以在[这里](https://bit.ly/2024EOS) RSVP。如果你不住在新奥尔良，我们接受加密货币[捐赠](https://omo.so/sawt)。我们的 OpenAI 账单开始增长，任何帮助都将不胜感激。

‍

图 1：Sawt 的[响应](https://www.sawt.us/s/8a03906d-d257-45d2-935a-efdfea3ace7c)截图

‍

## 至今为止的结果

尽管我们刚刚公开发布了 Sawt 的 beta 版本，但一些当地居民的精选群体自去年夏天以来一直在测试该版本。迄今为止，新奥尔良人已经提出了 319 个问题。他们还就系统生成的 201 个回答提供了反馈。

### 保留

尽管进行了有针对性的推广，但我们注意到社区成员在首次互动之后没有再次使用该工具。除了在 11 月中旬的一个焦点小组会议期间出现的高峰之外，几乎没有什么参与。这表明 Sawt 目前还不够有用，不能促使人们想要回来使用。

‍

图 2：按周在 Sawt 上提出的问题

‍

我们以谨慎的态度对待 Sawt 的开发，深知技术本身并不是变革的催化剂。真正的草根参与至关重要。尽管我们对该项目持批评态度，但我们也对其潜力感到兴奋。旨在增加 Sawt 有用性的即将推出的功能包括新议会会议的即时数据上传、注册关键字在会议期间讨论时的提醒、更大的数据集和更精确的引用。

### 准确性与偏见

提高准确性和减少偏见对于使 Sawt 成为一个有价值的工具至关重要。通过与组织者、非营利工作者和来自不同地区的社区成员进行焦点小组讨论，我们确认了 Sawt 的回答并不总是准确的，有时会存在偏见。当被要求根据 1 - 5 的评分标准提供反馈时，人们目前将 Sawt 的评分定为约 3 分。

‍

图 3：对 Sawt 的反馈跨两个不同版本的工具进行比较（v0.0.1 和 v0.0.2）

‍

这些反馈会话也揭示了关于真实世界使用的一些反直觉的模式。例如，人们认为当源文档数量（k）最少时，生成的回答是最准确的。换句话说，当考虑的信息较少时，人们发现回答最准确。

‍

图 4：Sawt 的详细反馈。图 4 包含与图 3 相同的结果，但是对每个受访者进行了归一化处理。图 4 显示人们如何根据准确性、有用性、平衡和整体评分回答。k 参数对应于用于通知生成响应的文档数量。k=5 意味着使用了五个视频片段、会议记录和/或文章来通知响应。k=15 意味着使用了十五个这样的文档。

‍

最终，样本量较小，因此这些结果的统计显著性有限。如果您是新奥尔良市的居民，您可以通过提交一些反馈来[帮助](https://www.sawt.us/feedback)。

‍

## 制作过程

最初的 Sawt [原型](https://colab.research.google.com/drive/1DpEuim10ZxngSQ_m3hAWjnFETqO9_Zaf?usp=sharing) 是由[@ayyubibrahimi](https://github.com/ayyubibrahimi) 在一个周末开发的。核心逻辑从那时起保持一致。我们首先摄取原始数据，创建嵌入，并设置 FAISS。当有人向 Sawt 提问时，我们使用 FAISS 识别最相关的文档，将它们与查询组合，并将它们发送到 OpenAI 以获得回应。

‍

图 5：Sawt 的工作原理

‍

在预处理/FAISS 阶段，我们实现了 '[无相关标签的精确零-shot 稠密检索](https://arxiv.org/abs/2212.10496)’ 中的 Hypothetical Document Embeddings（HyDE）方法，以为我们的检索增强生成创建嵌入空间。RAG 是一种技术，它将大型语言模型的优势与外部数据检索相结合，旨在增强 AI 模型生成的响应的相关性和准确性。用于生成这些嵌入的假设文档是使用零-shot 提示生成的。这个提示 [如下所示：](https://github.com/eye-on-surveillance/sawt/blob/ec64a5f92e2ec978131df55832bdaab4707df5d9/packages/backend/src/preprocessor.py#L29C4-L29C134)

‍

> 作为人工智能助手，利用你的训练数据中的新奥尔良市议会记录，以提供详细而公正的回答以下问题：“{user_query}”。

‍

这种方法确保模型使用其训练时的特定数据集，旨在对问题提供全面和公正的答案。

在最后阶段发送给 OpenAI 的提示更加复杂和详细。它包含了几条强调清晰格式化回答的指示。它旨在平衡回答的广度与简洁性。提示的一个重要部分专门用于明确调查和解决回答中潜在偏见的问题。此外，提示指示模型定义不常见的词语，使得对于可能不熟悉与市议会活动相关的特定术语的人们来说，回答更易于理解和信息丰富。

‍

图 5：[用于查询的提示](https://github.com/eye-on-surveillance/sawt/blob/ec64a5f92e2ec978131df55832bdaab4707df5d9/packages/googlecloud/functions/getanswer/inquirer.py#L214)

‍

尽管 Sawt 是 Eye on Surveillance 项目，但我们与[卡洛塔博士](https://sse.tulane.edu/aron-culotta)和三名 Tulane 大学的本科生合作。他们的工作重点是解决偏见问题，并建立了当前的反馈机制。卡洛塔博士还生成了图 4 中使用的分析。

‍

### 理解我们的背景

在 2020 年 12 月，新奥尔良警察局（NOPD）在被发现多年来一直在使用面部识别技术后，[被抓个正着](https://thelensnola.org/2020/11/12/new-orleans-police-department-using-facial-recognition-despite-years-of-denial/)，一个由当地组织联合组成的监控关注组织，决定从监控撤资并投资社区，[推动新奥尔良市议会通过数据隐私条例](https://www.wwno.org/latest-news/2020-12-18/city-council-bans-police-from-using-facial-recognition-technology)，其中包括禁止使用面部识别和其他监控技术。然而，仅仅一年后，市府取消了对面部识别和基站模拟器的禁令。而且，发起该条例的议会成员现在担任地区检察官，正在[试点预测性警务技术项目](https://thelensnola.org/2023/11/15/the-das-office-wants-to-use-predictive-analytics-software-to-direct-city-resources-to-places-that-drive-crime-will-it-work/)，这在他通过的条例下是被禁止的。

Eye on Surveillance 的大多数成员是有白天工作的志愿者。在他们的会议期间，要跟上议会正在采取的所有倒退步骤是很困难的。虽然官方议会会议是公开的，理论上是可访问的，但它们经常在工作时间内举行，持续一整天，议程会在最后一刻改变。这使我们很难跟上。尤其是当市政府策略性地安排有争议的决定/会议在假期前一天宣布时。

全年来，有许多情况，我们只是在市议会提出几个月后才获悉重要的监控计划。我们希望 Sawt 能成为我们和其他新奥尔良人未来的有用工具。

‍

图 6：新奥尔良在 2023 年进行了没有透明度的监控

‍

+   [ShotSpotter](https://www.wwltv.com/article/news/crime/new-orleans-city-council-police-shotspotter-technology-crime-reduction-shootings/289-966251f7-4097-4c61-bbba-f4c1138e81de)：市议会在一月份考虑使用它，我们在二月份发现了这一消息。

+   [面部识别](https://youtu.be/PwiJYkLNzZA?t=9953)：报告已于 2 月份可用，但市府拒绝了几次公开记录请求，直到 7 月份。Sawt 本身在提供信息方面非常有帮助，导致我们的公开记录请求最终被接受。

+   [预测性警务](https://thelensnola.org/2023/11/15/the-das-office-wants-to-use-predictive-analytics-software-to-direct-city-resources-to-places-that-drive-crime-will-it-work/): 以前支持禁止预测性警务的委员会成员现在是地方检察官，并正在试点这种非法技术。我们是从报纸上得知的。

+   [无人机](https://www.fox8live.com/2023/11/29/nopd-solicits-public-feedback-proposed-drone-program/): NOPD 征求对他们已购买的无人机的反馈意见。

‍

## 下一步

随着我们迈入 2024 年，我们正在思考 Sawt 的更广泛影响。我们在质疑我们可能正在持续 AI 偏见，并考虑到可能阻碍我们努力的监控眼成员的特定偏见。尽管如此，我们对持续的进展和在 2024 年夏季推出官方版本的前景感到兴奋。我们致力于创建社区拥有和运营的 AI 模型。

如果你想参与：

+   [确认出席](https://bit.ly/2024EOS)我们的年会

+   [注册](https://actionnetwork.org/forms/join-eye-on-surveillances-mailing-list/)我们的邮件列表

+   [贡献](https://github.com/eyeonsurveillance/sawt)一些代码

+   [捐赠](https://omo.so/sawt)

+   比特币: bc1qkseneu5cv9g6u4gpmnlen3q3at59r6sj6kn07q

+   SOL: CawuhzDxyytazxywF942VsLwi4RKEWqryLYDsv4hndNa

+   以太坊: 0xAA37b8a54e49e6c61De9904985e2887dfEABBA20
