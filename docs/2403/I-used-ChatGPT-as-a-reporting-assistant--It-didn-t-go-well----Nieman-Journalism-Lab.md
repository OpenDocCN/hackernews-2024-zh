<!--yml

类别：未分类

日期：2024-05-29 12:31:20

-->

# 我曾将ChatGPT用作报道助手。情况不太理想。| Nieman新闻实验室

> 来源：[https://www.niemanlab.org/2024/03/i-used-chatgpt-as-a-reporting-assistant-it-didnt-go-well/](https://www.niemanlab.org/2024/03/i-used-chatgpt-as-a-reporting-assistant-it-didnt-go-well/)

当涉及到人工智能的发展时，事情进展迅速。距离像ChatGPT、Midjourney、Stable Diffusion和Meta的LLaMA这样的生成式AI工具的公开发布还不到两年时间。监管机构、立法者和企业开始逐渐理解使用生成式AI工具的影响。

这包括

[新闻组织](https://www.cjr.org/tow_center_reports/artificial-intelligence-in-the-news.php)

和已经开始的记者

[实验中](https://www.thecity.nyc/2024/02/29/chatgpt-map-stories-nyc/)

.

[Nieman实验室报道称](https://www.niemanlab.org/2024/03/five-of-this-years-pulitzer-finalists-are-ai-powered/)

今年普利策奖的45位未公布的决赛选手中，有5位使用AI进行“研究、报道或提交作品”。在调查记者和编辑的年度活动

[NICAR](https://www.ire.org/training/conferences/nicar-2024/)

上周在巴尔的摩举行的数据新闻会议，参与者达14人

[200多个会议](https://schedules.ire.org/nicar-2024/)

与AI相关的话题，讨论技术如何帮助记者优化工作流程，总结复杂文件，并调试他们的代码。

我自己也有一个，题为“[利用AI工具进行数据新闻报道](https://bit.ly/nicar24-ai-tools)”，在这场会议上，我首先回顾了许多可用工具，并强调了[关于](https://arxiv.org/abs/2403.00742) [许多](https://www.theatlantic.com/technology/archive/2024/03/ai-water-climate-microsoft/677602/) [伦理](https://www.bloomberg.com/graphics/2024-openai-gpt-hiring-racial-discrimination/) [方面](https://themarkup.org/hello-world/2023/11/11/meet-nightshade-a-tool-empowering-artists-to-fight-back-against-ai) 的担忧。

然后，我展示了我耗时的实验结果，试图使用ChatGPT作为报道故事助手。剧透警告：效果不太好！

我用于这项练习的示例故事是2023年2月在俄亥俄州东帕利斯坦发生的[列车脱轨事故](https://www.ntsb.gov/investigations/Pages/RRD23MR005.aspx)，这是一个涉及各种数据的重大事件，我请求ChatGPT帮助分析。明确一点，这并不是我报道过的故事，而是我想尝试像数据记者一样使用ChatGPT的方式。作为这个练习的一部分，我花了大量时间与ChatGPT交流，有时候感觉相当耗费精力。您可以在这里阅读我的其中一个聊天会话的[记录](https://chat.openai.com/share/c343795f-210f-4052-a506-7c662c6c78e5)。ChatGPT在提供像维基百科这样质量不高的信息或不精确的位置时，表现出来的信心有时会误导人。有时我能够让聊天代理程序给我想要的东西，但我必须非常具体并经常斥责它。

例如，当ChatGPT满足了我要求的“生成一个以事故地点为中心的简单地图”时，我立即注意到地图上的标记远离任何铁路轨道。当我询问它从哪里获得了事故地点的坐标时，它回答说这些是“根据事件位置的一般知识推断出来的”。当我要求更具体的引用时，它无法提供，并一直重复说它依赖于“一般知识”。我不得不提醒这个工具，在我们开始聊天时我已经告诉它“你引用来源至关重要，并且总是使用权威来源”。在最后它终于能够标记出正确的位置之前，我不得不提醒它，它可以从我之前在聊天中上传的联邦铁路管理局[事故报告PDF](https://safetydata.fra.dot.gov/Officeofsafety/Publicsite/FORM54/F54Report.aspx?RepType=SQL&txtf54key=NS15220720230203)获取位置坐标。

提取信息并总结长篇文档通常被认为是像ChatGPT这样工具的最大优势之一。我的结果参差不齐。经过一番交流后，我成功说服这个代理程序提取了有关事故中释放的危险化学品的详细数量和信息，并将这些信息格式化成一张表格，列出了化学品名称、释放量、典型用途以及对人类健康的影响。但这需要几次尝试。它节省时间的地方在于解释专业信息，而这些信息通常需要耗费大量时间进行谷歌搜索，比如解码铁路车辆编号。

有时，这个工具过于热情，所以我要求它稍微收敛一点：“你可以跳过闲聊和客套话。”用户可以指示机器人改变回应的语气或风格，但告诉它是个律师并不能使其更加准确。（[查看原文链接](https://themarkup.org/hello-world/2024/01/06/what-happens-when-you-roleplay-with-chatgpt)）

总体而言，这些会议很费力地尝试弄清楚代理如何获取信息，并通过精确的指导重新定向它。这花费了很长时间。

制造ChatGPT的公司OpenAI未对评论请求作出回应。

根据我的交流经验，ChatGPT最有用的能力远非其生成和调试编程代码的能力。（在East Palestine的一次演习期间，它生成了一些简单的[Python](https://en.wikipedia.org/wiki/Python_(programming_language))代码，用于创建脱轨地图。）在回应编写代码的请求时，它通常会解释其方法（即使可能不是最佳方法），展示其工作，并且如果你认为其计划不符合需求，你可以重新指导它采用不同的方法。AI代理能够在保持上下文和讨论历史的同时不断增加代码功能，这确实可以节省大量时间，避免在StackOverflow（最大的在线编码社区之一）上对类似问题进行痛苦的搜索。

NICAR的练习让我对使用生成AI工具进行数据新闻工作感到担忧。像ChatGPT这样强大的工具无法准确地提供其了解某事物的“收据”，这违反了我们作为记者接受的一切训练。此外，我担心人手不足的小型新闻编辑部过度依赖这些工具，而新闻行业却面临裁员和关闭的困境。当新闻编辑部领导层在使用这些工具的问题上缺乏指导时，可能会导致错误和不准确。

幸运的是，许多新闻编辑部已经开始通过起草AI政策来解决一些这些问题，以帮助他们的记者和读者理解他们计划如何在工作中使用AI。

[The Markup](https://themarkup.org/)跟随了其他新闻机构的步伐，上周更新了[我们的道德政策](https://themarkup.org/ethics#ai-ethics)，其中详细说明了我们在工作中使用AI的规则。总结起来，它说：

+   我们不会发布由AI创作的报道或艺术作品（除非这是关于AI的故事的一部分）。

+   我们将始终标注或披露其使用情况。

+   我们将始终严格检查我们的工作，这当然也适用于任何由AI生成的内容。

+   我们将评估任何新的AI工具的安全性、隐私和道德考虑。

谢谢阅读，并始终仔细检查聊天机器人告诉你的每件事！
