<!--yml

类别：未分类

日期：2024-05-27 13:31:01

-->

# llamafile 的进展，四个月后 - Mozilla Hacks - Web 开发者博客

> 来源：[https://hacks.mozilla.org/2024/04/llamafiles-progress-four-months-in/](https://hacks.mozilla.org/2024/04/llamafiles-progress-four-months-in/)

当 Mozilla 的创新团队 [首次推出](https://hacks.mozilla.org/2023/11/introducing-llamafile/) 去年年底推出 [llamafile 项目](https://github.com/Mozilla-Ocho/llamafile) 时，我们对来自开源 AI 开发者的立即积极响应感到非常兴奋。它已成为 Mozilla 在 GitHub 上排名前三的最受喜爱的仓库之一，吸引了许多贡献者、一些出色的 PR，以及我们 [Discord 服务器](https://discord.gg/YTgM42NZEr) 上不断增长的社区。

在整个过程中，首席开发人员和项目愿景师 [Justine Tunney](https://github.com/jart) 一直在项目的各种基本改进上努力工作。就在昨晚，Justine [发布了 llamafile 的 v0.8 版本](https://github.com/Mozilla-Ocho/llamafile/releases/tag/0.8)，其中不仅包括对最新开放模型的支持，还有一些针对 CPU 推理的重大性能改进。

多亏 Justine 的工作，今天 llamafile 成为在您自己的硬件上运行各种开放大语言模型最简单且*最快速*的方法。亲自体验：使用 llamafile，您可以在日常 Macbook 上运行 Meta 刚发布的 [LLaMA 3 模型](https://huggingface.co/jartine/Meta-Llama-3-8B-Instruct-llamafile)——这款模型与其同类中最好的模型不相上下。

我们是如何做到的？为了解释这一点，让我们退后一步，告诉您自 v0.1 以来发生的一切变化。

**tinyBLAS：为 NVIDIA GPU 支持民主化** ***和*** **AMD**

llamafile 建立在如今已成传奇的 [llama.cpp](https://github.com/ggerganov/llama.cpp) 项目之上。llama.cpp 通过 cuBLAS 线性代数库支持 NVIDIA 处理器的 GPU 加速推理，但这要求用户安装 NVIDIA 的 CUDA SDK。我们对此感到不安，因为这与我们构建一个完全开源和透明的 AI 栈的项目目标相冲突，任何人都可以在商品硬件上运行。此外，在某些系统上正确设置 CUDA 可能会很麻烦。必须有更好的方法。

借助社区的帮助（看这里，[@ahgamut](https://ahgamut.github.io/) 和 [@mrdomino](https://github.com/mrdomino)!），我们创造了自己的解决方案：它叫做 tinyBLAS，是 llamafile 全新且高效的线性代数库。tinyBLAS 让 llamafile 用户轻松使用 NVIDIA 加速。在 Windows 上，您甚至不需要安装 CUDA；您只需安装您可能已经安装的显示驱动程序。

但tinyBLAS不仅仅是支持NVIDIA：它也支持AMD的GPU。这绝非小事。虽然AMD在今天的GPU市场中占有可观的20%，但由于软件和驱动程序的支持较差，它们在机器学习领域一直是次要的参与者。这是一种遗憾，因为AMD的GPU提供高性能、具有价格竞争力且广泛可用。

llamafile的一个目标是使开源AI技术普及化，这意味着为AMD争取一个位置。这正是我们所做的：有了llamafile的tinyBLAS，你现在可以轻松地充分利用你的AMD GPU来加速本地推理。而且，就像CUDA一样，如果你是Windows用户，甚至不需要安装AMD的ROCm SDK。

所有这些意味着对许多用户来说，llamafile将自动使用你的GPU，无需或仅需很少的努力。

**更快本地AI的CPU性能提升**

在Mozilla，我们对“本地AI”的前景非常感兴趣，即AI模型和应用直接在最终用户硬件上运行，而不是在云中。本地AI令人兴奋，因为它使得用户更有可能控制这些系统，并为用户提供更大的隐私和安全性。

但许多消费设备缺乏通常需要用到的高端GPU。在这方面，llama.cpp是一个改变游戏规则的因素，因为它使得在CPU上进行本地推理变得可能且性能可用，而不仅限于GPU。

Justine在llamafile上的最新工作进一步推动了技术前沿。正如她在[关于此主题的详细博客文章](http://justine.lol/matmul/)中所记录的，通过编写84个新的矩阵乘法核心，她成功将llamafile的快速评估性能提升了惊人的10倍，与我们之前的版本相比。这是在推动使消费者硬件上的本地AI变得可行方面的重要而有影响力的一步。

这项工作也是我们致力于开源AI社区的一个典范。完成这项工作后，我们立即向llama.cpp的上游[提交了一个PR](https://github.com/ggerganov/llama.cpp/pull/6414)，以将这些性能改进反馈到llama.cpp。这只是我们为llama.cpp贡献的众多增强措施中的最新一项，而我们计划继续这种实践。

**树莓派性能提升**

谈到消费者硬件，没有比备受喜爱的树莓派更有趣且更谦卑的例子了。以超值价格，你可以获得一台运行Linux的功能齐全计算机，具备足够的计算能力以满足典型桌面应用需求。这是一份令人印象深刻的套餐，但从历史上看，它并不被认为是AI应用的可行平台。

不再是这样了。llamafile现在已经针对最新模型（例如Raspberry Pi 5）进行了优化，结果是一些小型LLM（如Rocket-3B ([下载](https://huggingface.co/jartine/rocket-3B-llamafile/resolve/main/rocket-3b.Q5_K_M.llamafile?download=true))、TinyLLaMA-1.5B ([下载](https://huggingface.co/jartine/TinyLlama-1.1B-Chat-v1.0-GGUF/resolve/main/TinyLlama-1.1B-Chat-v1.0.Q5_K_M.llamafile?download=true))和Phi-2 ([下载](https://huggingface.co/jartine/phi-2-llamafile/resolve/main/phi-2.Q5_K_M.llamafile?download=true))）在今天最便宜的计算机上运行速度可用。我们在某些情况下看到了高达[每秒80个token](https://twitter.com/JustineTunney/status/1776440470152867930)的快速评估速度！

**跟上最新模型**

在开放模型领域，进展的速度 [惊人地快](https://twitter.com/maximelabonne/status/1779123021480865807)。在过去几个月里，通过微调已发布或更新了数百个模型。沿途，明显有一个明显的趋势，即模型性能不断提升，模型大小也在不断减小。

llama.cpp项目一直在积极跟进所有这些新模型，经常在发布几天内就支持新架构和模型功能。

就我们而言，我们一直在紧密同步llamafile和llama.cpp，以便支持所有相同的模型。鉴于这两个项目的复杂性，这绝非小事，所以我们很幸运有Justine负责。

今天，由于她的辛勤工作，你可以使用llamafile使用最新最强大的开放模型。例如，我们能够在Meta最新的LLaMA 3模型- [8B-Instruct](https://huggingface.co/jartine/Meta-Llama-3-8B-Instruct-llamafile) 和 [70B-Instruct](https://huggingface.co/jartine/Meta-Llama-3-70B-Instruct-llamafile)发布当天就推出llamafile。昨天的0.8版本中，llamafile还可以运行Grok、Mixtral 8x22B和Command-R。

**创建你自己的llamafiles**

自从llamafile发布以来，人们就一直想要创建自己的llamafiles。以前，这需要几个步骤，但今天你可以用一个命令完成，例如：

`llamafile-convert [model.gguf]`

就在这一刻，这将生成一个名为“model.llamafile”的文件，可以立即使用。感谢社区成员[@chan1012](https://github.com/chand1012)为改进做出的贡献。

在相关发展中，Hugging Face最近在其模型中心正式支持了llamafile。这意味着你现在可以通过特定于llamafile的搜索和过滤，搜索其他开源社区成员创建和分发的llamafile。

**兼容OpenAI API服务器**

由于它建立在llama.cpp之上，llamafile继承了该项目的服务器组件，提供了兼容OpenAI API的端点。这使得那些在OpenAI基础上构建的开发者可以转而使用开源模型。在Mozilla，我们非常希望支持这种未来：一个开源AI可以作为集中式、封闭、商业化解决方案的可行替代品。

虽然开源模型目前还不能完全与封闭模型的能力相媲美，但它们正在快速进步。我们相信，使现有代码更容易转向使用开源模型将增加需求，并进一步推动这一进展。

在过去几个月中，我们努力扩展这些端点，旨在增强功能和提升兼容性。如今，llamafile可以在各种用例中作为OpenAI的即插即用替代品。

我们希望进一步扩展API服务器的功能，并且渴望听到开发者的需求和反馈。您因何未采用开源模型？您需要哪些功能、能力或工具？[请告诉我们](https://discord.gg/YTgM42NZEr)！

**与其他开源AI项目的集成**

最后，看到独立开发者采纳llamafile并将其整合到领先的开源AI项目中（例如[Open Interpreter](https://github.com/OpenInterpreter/open-interpreter)），实在是令人欣喜。特别要表扬我们自己的[Kate Silverstein](https://github.com/k8si)，她提交的PR为[LangChain](https://github.com/langchain-ai/langchain)和[LlamaIndex](https://github.com/run-llama/llama_index)增加了llamafile支持（[AutoGPT](https://github.com/Significant-Gravitas/AutoGPT)即将推出）。

如果您是维护者或开源AI项目的贡献者，认为llamafile集成会对项目有所裨益，[请告诉我们如何提供帮助](https://discord.gg/YTgM42NZEr)。

**加入我们吧！**

llamafile项目刚刚起步，这也是Mozilla致力于贡献和参与开源AI社区的重大新举措的第一步。我们将很快分享更多相关内容，但现在：我邀请您加入我们的llamafile项目！

与Mozilla的llamafile团队和整个llamafile社区交流的最佳场所是我们的Discord服务器，其中有[专门的llamafile频道](https://discord.gg/YTgM42NZEr)。当然，您在我们的[GitHub仓库](https://github.com/Mozilla-Ocho/llamafile)上的增强请求、问题和PR，我们也随时欢迎。

希望您能加入我们。接下来的几个月对于llamafile和开源AI本身将会比过去更加有趣和意外。

Stephen在Mozilla的创新团队中领导开源AI项目（包括llamafile）。他以前管理过社交书签先驱del.icio.us；共同创立了Storium、Blockboard和FairSpin；并参与了Yahoo搜索和BEA WebLogic的工作。

[更多由斯蒂芬·胡德（Stephen Hood）撰写的文章…](https://hacks.mozilla.org/author/slangtonhoodmozilla-com/)
