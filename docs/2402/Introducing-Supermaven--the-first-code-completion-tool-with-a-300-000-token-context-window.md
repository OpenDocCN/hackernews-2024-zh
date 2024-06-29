<!--yml

category: 未分类

date: 2024-05-27 15:04:34

-->

# 介绍Supermaven，第一个具有300,000个标记上下文窗口的代码完成工具

> 来源：[https://supermaven.com/blog/introducing-supermaven](https://supermaven.com/blog/introducing-supermaven)

| 工具 | 帧 | 延迟（毫秒） |
| --- | --- | --- |
| Supermaven | 15 | 250 |
| Copilot | 47 | 783 |
| Codeium | 53 | 883 |
| Tabnine | 50 | 833 |
| Cursor | 113 | 1,883 |

*详细信息见下文。*

代码完成已经走过了一段很长的路程。当我在2018年创建了TabNine，这是第一个使用深度学习进行代码补全的工具时，几乎没有开发人员工具使用AI。那时的模型很小，功能有限，人们怀疑这些工具是否足够有用，值得付费。

此后，AI编码工具已被广泛采用：GitHub最近[宣布](https://www.theinformation.com/briefings/microsoft-github-copilot-revenue-100-million-ARR-ai)，其工具Copilot年收入达到1亿美元。现在，数百万开发人员每天都在使用AI工具。这种采用是由于部署了具有数十亿参数的大模型，这些模型比2018年的模型更加实用。

鉴于代码完成已经“普及化”，最大语言模型的规模不断增长，并且训练最大模型的巨大成本，这引发了一个问题：小型初创公司还有空间吗？代码完成是否成为微软或谷歌等大公司的专属领域？

在Supermaven，我们认为这个问题还没有最终解决。原因在于大型模型的高成本服务。虽然像GPT-4这样的模型提供了无与伦比的建议质量，但不可能在每次按键时运行它（除非向用户收取每月1000美元）。如今，AI编码工具的部署受到运行工具所需的GPU成本的限制，同样也受到开发工具本身成本的限制。这就是为什么Copilot通过[限制提供建议的上下文](https://thakkarparth007.github.io/copilot-explorer/posts/copilot-internals.html#preventing-poor-requests-via-contextual-filter)来减少成本的原因。

服务代码完成工具的高成本使竞争变得公平：这意味着每个人，无论是小型初创公司还是微软，都必须使用小型模型来保持盈利。最佳产品是能够最有效地使用小型模型的产品。

Supermaven 有何不同？

### 300,000-token 上下文窗口

代码完成模型的上下文窗口确定了它在提出建议时能够考虑的代码量。长期的上下文窗口对提供良好的补全至关重要——这就是为什么Copilot最近将其上下文窗口扩展到8,192个标记的原因。

在Supermaven，我们从头开始开发和训练了一种新的神经网络架构，比Transformer（当前标准架构）在整合长上下文窗口中的信息方面更高效。这使我们能够在保持成本和延迟与具有4,000令牌上下文窗口的Transformer相同的情况下，提供一个30万令牌的上下文窗口。

作为用户，这意味着在Supermaven花费10-20秒处理您的代码库后，它将熟悉您的API和代码库的独特约定。这解决了Copilot最常见的抱怨之一：尽管它在处理覆盖率高的知名API和库时表现出色，但在处理实际工作中使用时所遇到的大型、特异的代码库时则显得力不从心。

这里有一个简单的例子，说明了一个30万令牌上下文窗口的差异。在这个例子中，我在同一段代码上运行了Supermaven和Copilot，并要求它们生成一个名为`doubleIndentation`的函数，其参数类型为`Indentation`。由于文件中未定义或使用`Indentation`结构体，Copilot在上下文中没有任何关于它的信息，因此生成的代码不能编译通过。而Supermaven生成了正确的代码，因为它的上下文窗口足够大，可以包含整个5万令牌的代码库。

<https://cdn.supermaven.com/video/longdef.mp4>

### 速度

Supermaven速度很快：我们建立了定制的基础设施，使其在处理大型代码库的长提示时保持响应性。

我设计了一个简单的测试来比较Supermaven和其他竞争对手。为了确保每个工具都在缓存中有提示信息，我从一个上下文开始，其中每个工具都同意最可能的完成是`for (let i`。然后我键入了`for (let j`，强制每个工具生成一个新的完成，并测量了在屏幕上出现'j'和完成出现之间的帧数。以下是结果：

| Tool | Frames | Latency (ms) |
| --- | --- | --- |
| Supermaven | 15 | 250 |
| Copilot | 47 | 783 |
| Codeium | 53 | 883 |
| Tabnine | 50 | 833 |
| Cursor | 113 | 1,883 |

<https://cdn.supermaven.com/video/speed_comparison.mp4>

### 培训基于编辑序列，而不仅仅是文件

与大多数代码助手不同，Supermaven不将您的代码视为一系列文件。相反，Supermaven看到您对代码库进行的编辑序列 - 类似于运行`git diff`后看到的内容。这有助于Supermaven快速理解您试图完成的任务，并非常适合重构。以下是一些示例：

<https://cdn.supermaven.com/video/edit_sequence.mp4>

### 现在就试试吧

[下载 Supermaven](/download) 体验一下。我们很期待您的反馈。

如果您不使用VS Code，请[关注我们的Twitter](https://twitter.com/SupermavenAI)，了解Supermaven何时适用于您喜欢的编辑器。
