<!--yml

类别：未分类

日期：2024-05-29 12:45:21

-->

# AI机器人幻觉出软件包，开发者们下载了它们 • The Register

> 来源：[https://www.theregister.com/2024/03/28/ai_bots_hallucinate_software_packages/](https://www.theregister.com/2024/03/28/ai_bots_hallucinate_software_packages/)

深入研究几家大企业已经发布了包含先前由生成式AI幻觉的软件包的源代码。

不仅如此，还有人注意到这种频繁出现的幻觉，将这个虚构的依赖项转变为了一个真实的依赖项，结果成千上万次地被开发者下载和安装，这是由于AI的不良建议，我们已经了解到。如果这个软件包被注入了实际的恶意软件，而不是单纯的测试，后果可能是灾难性的。

据Lasso Security的安全研究员Bar Lanyado称，其中一个被AI愚弄，将这个软件包纳入项目中的企业是阿里巴巴，写作时依然在其[GraphTranslator](https://github.com/alibaba/graphtranslator)的安装说明中包含一个[`pip`命令](https://github.com/alibaba/GraphTranslator/blame/87ed496ab793180cd9d4183459b57ff6f6c3b5a0/README.md#L48)，以下载Python包`huggingface-cli`。

存在一个合法的[huggingface-cli](https://huggingface.co/docs/huggingface_hub/en/guides/cli)，可以使用`pip install -U "huggingface_hub[cli]"`进行安装。

但通过Python包索引（PyPI）分发的[`huggingface-cli`](https://pypi.org/project/huggingface-cli/)，是阿里巴巴的GraphTranslator所需的，可以使用`pip install huggingface-cli`安装，但这个包是虚构的，由AI想象出来，并由Lanyado作为实验变成了真实存在的。

他在去年12月创建了`huggingface-cli`，因为他多次看到生成式AI反复幻觉出这个包；到今年2月，阿里巴巴在GraphTranslator的README说明中提到它，而不是真正的Hugging Face CLI工具。

### 研究

Lanyado这样做是为了探索这些由生成式AI模型在项目开发期间可能发明的幻觉软件包是否会随时间推移而持续存在，并测试这些虚构软件包名是否可以被利用，并通过编写实际使用这些AI虚构代码名称的软件包，来分发恶意代码。

这里的想法是，一些邪恶的人可能向模型询问代码建议，记录AI系统反复推荐的虚构软件包的笔记，然后实现这些依赖项，这样其他程序员在使用相同的模型并获得相同的建议时，就会拉取这些可能含有恶意软件的库。

去年，通过安全公司Vulcan Cyber，Lanyado [发表了](https://vulcan.io/blog/ai-hallucinations-package-risk) 一篇详细研究，说明如何向AI模型如ChatGPT提问并得到建议，推荐使用一个不存在的软件库、包或框架。

“当攻击者运行这样的攻击活动时，他会向模型询问解决编程问题的软件包，然后会收到一些不存在的包”，Lanyado向《The Register》解释道。“他将以相同的名称上传恶意软件包到适当的注册表中，从那时起，他只需等待人们下载这些软件包。”

### 危险的假设

AI模型对于自信地[引用不存在的法院案件](https://www.theregister.com/2023/06/22/lawyers_fake_cases/)现象如今已为人熟知，并在不知情的律师中引起了相当多的尴尬。事实证明，生成式AI模型对软件包也是如此。

正如Lanyado先前指出的，恶意分子可能会使用AI虚构的名称上传到某些仓库中的恶意包。但要使此成为有效的攻击向量，AI模型需要反复推荐被篡改的名称。

这正是Lanyado试图测试的。他凭借数千个“如何”问题，向四个AI模型（GPT-3.5-Turbo、GPT-4、Gemini Pro（又名Bard）和Command [Cohere]）查询五种不同编程语言/运行时（Python、Node.js、Go、.Net和Ruby）的编程挑战，每种语言/运行时都有其自己的打包系统。

事实证明，这些聊天机器人随意生成的一部分名字是持久的，有些甚至跨越不同的模型。而持久性——即虚假名字的重复——是将AI的幻想转化为功能性攻击的关键。攻击者需要AI模型在其响应用户时重复幻想包的名称，以便在这些名称下创建的恶意软件能够被寻找并下载。

Lanyado随机选择了20个零射击幻觉问题，并对每个模型进行了100次提问。他的目标是评估幻觉包名称保持相同的频率。他的测试结果显示，这些名称通常足够持久，以便成为功能攻击向量，尽管并非始终如此，在某些包装生态系统中可能更为突出。

据Lanyado称，根据GPT-4的数据，24.2%的问题回答产生了幻觉包，其中19.6%是重复的。《The Register》提供的一张表格详细列出了GPT-4响应的更多细节。

| 21340 | 13065 | 4544 | 5141 | 3713 |
| --- | --- | --- | --- | --- |
| 5347 (25%) | 2524 (19.3%) | 1072 (23.5%) | 1476 (28.7%) 1093 exploitable (21.2%) | 1150 (30.9%) 109 exploitable (2.9%) |
| 1042 (4.8%) | 200 (1.5%) | 169 (3.7%) | 211 (4.1%) 130 exploitable (2.5%) | 225 (6%) 14 exploitable (0.3%) |
| 4532 (21%) | 2390 (18.3%) | 960 (21.1%) | 1334 (25.9%) 1006 exploitable (19.5%) | 974 (26.2%) 98 exploitable (2.6%) |
| 34.4% | 24.8% | 5.2% | 14% | – |

使用 GPT-3.5，22.2% 的问题回答引起了幻觉，其中 13.6% 是重复的。对于 Gemini，64.5% 的问题产生了虚构的名称，其中约 14% 重复。至于 Cohere，则有 29.1% 的幻觉和 24.2% 的重复。

即便如此，Go 和 .Net 中的包装生态系统都以一种限制攻击潜力的方式构建，通过拒绝攻击者访问特定路径和名称。

“在 Go 和 .Net 中，我们收到了幻觉包，但其中许多不能用于攻击（在 Go 中的数字远比在 .Net 中的多），每种语言都有其原因。”Lanyado 向 *The Register* 解释道。“在 Python 和 npm 中并非如此，因为模型推荐我们不存在的软件包，而没有任何阻止我们上传这些名称的软件包，因此在 Python 和 Node.js 上运行这种攻击显然更容易。”

### 播种 PoC 恶意软件

Lanyado 通过分发 PoC 恶意软件来表达这一点——Python 生态系统中的一组无害文件。基于 ChatGPT 的建议运行 `pip install huggingface-cli`，他上传了一个同名的空包到 PyPI（上述提到的），并创建了一个名为 `blabladsa123` 的虚假包，以帮助将软件包注册扫描与实际下载尝试分离。

他声称的结果是 `huggingface-cli` 在其上市的三个月内获得了超过 15,000 次真实下载。

“此外，我们在 GitHub 上进行了搜索，以确定该软件包是否被其他公司的代码库所使用”，Lanyado 在 [这篇报告](https://www.lasso.security/blog/ai-package-hallucinations) 中提到他的实验。

“我们的发现显示，几家大公司在其代码库中要么使用，要么推荐使用这个软件包。例如，阿里巴巴研究专用的代码库的 README 中可以找到安装这个软件包的说明。”

阿里巴巴没有回复评论请求。

Lanyado 还提到，有一个 Hugging Face 拥有的项目整合了这个假 huggingface-cli，但在他提醒公司之后，[已被移除](https://github.com/huggingface/diffusers/commit/56b68459f50f7d3af383a53b02e298a6532f3084)。

到目前为止，据 Lanyado 所知，这种技术尚未被用于实际攻击。

“除了我们的幻觉软件包（我们的软件包不是恶意的，只是一个如何利用这种技术的例子），我还未能识别出恶意行为者利用这种攻击技术的实例”，他说。“需要注意的是，要识别这种攻击是很复杂的，因为它并不留下很多踪迹。” ®
