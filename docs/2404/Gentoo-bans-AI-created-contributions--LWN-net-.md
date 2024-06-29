<!--yml

分类：未分类

日期：2024-05-27 13:18:06

-->

# Gentoo禁止AI创建的贡献 [LWN.net]

> 来源：[https://lwn.net/SubscriberLink/970072/93a5696aa497d415/](https://lwn.net/SubscriberLink/970072/93a5696aa497d415/)

| **LWN.net需要您的支持！**没有订阅者，LWN将无法存在。请考虑[订阅支持](/subscribe/)，帮助LWN继续发布 |
| --- |

由**Joe Brockmeier**提供

2024年4月18日

[Gentoo理事会](https://wiki.gentoo.org/wiki/Project:Council)成员Michał Górny在二月底向gentoo-dev邮件列表发表了一篇RFC，提议禁止在Gentoo Linux项目中使用"<q>'AI'-backed (LLM/GPT/whatever) contributions</q>"。Górny写道，"<q>AI泡沫</q>"的蔓延表明Gentoo有必要正式对AI工具采取立场。经过长时间讨论，Gentoo理事会本周一致投票通过了他的提议，并禁止使用AI/ML工具生成的贡献。

#### 反方观点

在他的RFC中，他列出了版权、质量和伦理三个广泛关注的问题。在版权问题上，他认为LLM是在受版权保护的材料上进行训练的，而且这些背后的公司对版权侵犯毫不在意。"<q>特别是，这些工具很可能产生我们无法合法使用的内容。</q>"

他对LLM输出的质量提出了质疑，尽管他承认LLM可能会"<q>在你足够小心的情况下提供良好的帮助</q>"。但他说，不能保证贡献者意识到风险。他对使用AI的伦理观点毫不妥协。Górny对由AI引发的能源消耗、劳工问题以及"<q>各种垃圾和骗局</q>"表示反对。他认为唯一合理的做法是完全禁止在Gentoo中使用这些工具创作作品：

> 换句话说，明确禁止人们使用ChatGPT、Bard、GitHub Copilot等工具，为Gentoo创建ebuilds、代码、文档、消息、Bug报告等。

他补充说，这仅限于为Gentoo项目专门创建的作品，并不涵盖使用类似ChatGPT的上游项目。Andreas K. Hüttel [询问](https://lwn.net/ml/gentoo-dev/11768224.nUPlyArG6x@kona/)是否有反对在Gentoo上打包AI软件的意见。这没有在列表上引起赞成或反对的回应，但[AI政策页面](https://wiki.gentoo.org/wiki/Project:Council/AI_policy)明确提到该政策不禁止打包与AI相关的软件。

#### 这是必要的吗？

Rich Freeman [写道](https://lwn.net/ml/gentoo-dev/CAGfcS_k-8MmEivL8_MUwGGM9t+Zdd8iPczc=dBNFF1QBbcGdqw@mail.gmail.com/)，他认为考虑使用AI是有道理的，但建议[Gentoo开发者原产地证书](https://www.gentoo.org/glep/glep-0076.html#certificate-of-origin)（DCO）已经有必要的语言来禁止AI生成的贡献。"<q>也许我们应该重新宣传已经存在的政策？</q>" 他还评论了Górny提出的伦理案例，并表示即使项目的大多数成员支持，也会使一些贡献者疏远。Freeman认为重申Gentoo不希望只是将GPT应用的产出导入论坛、bug报告、提交等是个不错的主意，但并不认为这需要任何新政策。

Ulrich Mueller [回复说](https://lwn.net/ml/gentoo-dev/ua5nllr13@gentoo.org/)，现有政策有重叠之处，但并不觉得多余，并支持明确如何处理AI生成的代码的想法。Sam James [赞同提议](https://lwn.net/ml/gentoo-dev/87le757phq.fsf@gentoo.org/)，但担心这是"<q>略显表演性质 [...] 因为我们实际上无法强制执行</q>。" Górny [写道](https://lwn.net/ml/gentoo-dev/baa3755ec638bac5abb35fedf17cf1958b3615d0.camel@gentoo.org/)，项目很难检测到这些贡献，或者说并不想积极寻找它们。他说，重点在于声明它们是不受欢迎的。

Oskari Pirhonen [想知道](https://lwn.net/ml/gentoo-dev/Zd6i7GMVrNXwoicc@dj3ntoo/)，如果贡献者使用ChatGPT帮助撰写文档或提交消息（但不包括代码），因为他们"<q>对英语没有出色的掌握</q>"，那么这些贡献明确注明是AI生成内容，是否可接受？Górny [表示](https://lwn.net/ml/gentoo-dev/076b8ede8f3e2d6d49571d516132a96a08b4d591.camel@gentoo.org/)，这并没有太大帮助，并否定了ChatGPT生成内容的质量。Mueller想知道界限在哪里：<q>像DeepL这样的翻译工具允许吗？对于这些，我看不到版权问题。</q>

Matt Jolly在罕见的异议中[回应](https://lwn.net/ml/gentoo-dev/cbb02a15-54b8-4804-b062-4731ed3d7f73@gentoo.org/)，认为Gentoo总是会有质量低劣的贡献，可以简单地运用常识来过滤出质量低下的LLM材料。他说 "<q>我们已经有方法来清除质量低下的贡献和恶意贡献者——让我们信任这些方法，并看看我们能做些什么来加强这些工具和流程。</q>" 他支持使用LLMs进行代码文档化，并问为什么他必须手动解释代码功能，如果LLM可以生成只需进行少量编辑的内容。他认为这项提议是一个坏主意，禁止LLMs "<q>就像是拿着孩子和洗澡水一起扔了</q>"。他认为可以制定指南，甚至禁止完全由人工智能生成的作品，但他反对 "<q>预先禁止有用工具</q>"。

James [回复](https://lwn.net/ml/gentoo-dev/875xy6sa6x.fsf@gentoo.org/)，表明使用基于Gentoo当前仓库训练的工具以及LLMs辅助提交消息是可以接受的。但他表示，许多自由开源软件项目已经因为太多人工智能垃圾信息而失去了对 "<q>可能有好处</q>" 部分的兴趣。

David Seifert [回复](https://lwn.net/ml/gentoo-dev/b45203b73decd68ea0abba34ce8b25370c5306c5.camel@gentoo.org/) 支持这项RFC，并询问是否可以将其添加到下次Gentoo理事会会议的议程上。Górny表示，他已被要求提供具体的动议，并提供了以下语言：

> 禁止将任何通过自然语言处理人工智能工具创建的内容贡献给Gentoo是明确规定的。如果有关于某个工具不构成版权、伦理和质量方面问题的案例，可以重新审议此决议。

#### 已批准

考虑到支持禁止AI生成贡献的评论与反对该禁令的评论的比例，Gentoo理事会投票接受了Górny的提议。现在的问题是Gentoo如何实施这一禁令。在回答问题的电子邮件中，Górny表示，Gentoo依赖于对贡献者的信任，让他们遵守这一政策，而不是试图监管贡献是否是由人工智能/机器学习工具生成的：

> 在这两种情况下，我们的主要目标是明确可接受和不可接受的内容，并礼貌地要求我们的贡献者尊重这一点。如果我们收到的贡献包含真正 "奇怪" 的错误，那种错误似乎不太可能是人为错误造成的，我们将开始提出问题，但我认为这是我们能做到的最好的。

随着人工智能/机器学习继续主导科技行业的议程，Gentoo采取与其他人不同的立场，试图排斥它而不是试图加入这场盛宴。这项政策的实施效果如何，以及何时进行测试，将会非常有趣。

* * *

(

[登录](https://lwn.net/Login/?target=/Articles/970072/)

发表评论）
