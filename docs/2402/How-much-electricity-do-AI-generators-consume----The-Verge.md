<!--yml

category: 未分类

date: 2024-05-27 14:55:20

-->

# AI发电机消耗多少电力？ - The Verge

> 来源：[https://www.theverge.com/24066646/ai-electricity-energy-watts-generative-consumption](https://www.theverge.com/24066646/ai-electricity-energy-watts-generative-consumption)

众所周知，机器学习消耗了*大量*能源。所有这些驱动电子邮件摘要、[杀害国王的聊天机器人](https://www.bbc.co.uk/news/technology-67012224)和霍默·辛普森唱新金属歌曲的AI模型正在累积一笔可观的服务器费用，以兆瓦每小时计算。但似乎没有人 — 即使是这些技术背后的公司 — 能确切地说出成本是多少。

估算是存在的，但专家们称这些数据是不完整且有条件的，只能提供AI总能耗的一瞥。这是因为机器学习模型极其多变，可以通过配置方式显著改变其能耗。此外，能够出具账单的组织 — 如Meta、Microsoft和OpenAI — 简单地没有分享相关信息。（Judy Priest，微软云运营和创新首席技术官在一封电子邮件中表示，公司目前“正在投资开发方法来量化AI的能耗和碳影响，同时致力于提高大型系统在训练和应用中的效率。”OpenAI和Meta未对评论请求做出回应。）

我们可以确定的一个重要因素是首次训练模型与将其部署给用户的差异。特别是训练过程极其耗能，消耗的电力比传统数据中心活动要多得多。例如，训练一个像GPT-3这样的大型语言模型据估计会消耗接近1300兆瓦时的电力；大约相当于130个美国家庭年度消耗的电力量。为了让人了解，观看1小时Netflix流媒体视频大约需要0.8千瓦时（0.0008兆瓦时）的电力。这意味着要观看162.5万小时才能消耗与训练GPT-3相同的电力量。

但目前最先进系统的实际应用情况如何，还很难说。能源消耗可能更大，因为多年来AI模型的规模一直在稳步增长，更大的模型需要更多能量。另一方面，公司可能正在使用一些[已验证的方法](https://arxiv.org/pdf/2104.10350.pdf)，使这些系统更加节能 — 这将抑制能耗上升的趋势。

法美人工智能公司Hugging Face的研究员Sasha Luccioni表示，更新估计的挑战在于随着人工智能变得盈利，公司变得更加保密。仅仅几年前，像OpenAI这样的公司会公布他们的训练方案的细节 — 使用什么硬件，持续多长时间。但是Luccioni表示，对于像ChatGPT和GPT-4这样的最新模型，同样的信息根本不存在。

“对于ChatGPT，我们不知道它有多大，我们不知道底层模型有多少参数，我们不知道它在哪里运行……它可能是三只穿大衣的浣熊，因为你根本不知道引擎盖下面是什么。”

“它可能是三只穿大衣的浣熊，因为你根本不知道引擎盖下面是什么。”

Luccioni，撰写了几篇关于人工智能能源使用的论文，她认为这种保密性部分是由于公司之间的竞争，但也是为了转移批评。人工智能的能源使用统计数据 — 尤其是其最不必要的用例 — 自然会引起与加密货币浪费的比较。“人们越来越意识到这一切并非免费”，她说。

训练模型只是一个方面。在系统创建后，它被推出给消费者使用以生成输出，这个过程被称为“推理”。去年十二月，Luccioni和来自Hugging Face和卡内基梅隆大学的同事们[发表了一篇论文](https://arxiv.org/pdf/2311.16863.pdf)（目前正在等待同行评审），其中包含了各种人工智能模型推理能源使用的首次估计。

Luccioni和她的同事对涵盖从回答问题到识别物体和生成图像等一系列用例的88种不同模型进行了测试。在每种情况下，他们运行了任务1,000次并估计了能源成本。他们测试的大多数任务使用的能量很少，比如对书面样本进行分类使用0.002千瓦时，生成文本使用0.047千瓦时。如果我们以我们一小时的Netflix流媒体作为比较，这相当于分别观看九秒或3.5分钟所消耗的能量。（请记住：这是执行每个任务1,000次的成本。）对于图像生成模型，这些数字显着较大，平均每1,000次推理使用2.907千瓦时。正如论文所指出的，[普通智能手机](https://www.epa.gov/energy/greenhouse-gases-equivalencies-calculator-calculations-and-references#smartphones)充电使用0.012千瓦时 — 因此使用人工智能生成一张图像几乎会消耗与充电智能手机相同的能量。

重点在于“可以”，因为这些数字并不一定适用于所有用例。Luccioni及其同事测试了十种不同的系统，从生成64 x 64像素小图片的小型模型到生成4K图片的大型模型，结果出现了巨大的差值。研究人员还标准化了使用的硬件，以更好地比较不同的AI模型。这并不一定反映了实际部署，在那里软件和硬件通常被优化为能源效率。

Luccioni说：“这绝对不代表每个人的用例，但现在至少我们有了一些数字。我想在这里立下一个标志，说'让我们从这里开始'。”

“生成AI革命带来了一个对我们完全不了解的地球成本。”

因此，这项研究提供了有用的相对数据，但并非绝对数字。例如，它显示了AI模型生成输出时需要更多的动力，而对输入进行分类时则需要更少的动力。它还表明，任何涉及图像的事物比文本更耗能。Luccioni说，尽管这些数据的偶然性可能令人沮丧，但这本身已经讲述了一个故事。她说：“生成AI革命带来了一个对我们完全不了解的地球成本，而对我来说这个差距尤其有指示意义。”她说：“总而言之，我们只是不知道。”

因此，想要确定生成一个Balenciaga教皇的能源成本是很困难的，原因在于各种变量。但如果我们想更好地了解地球成本，还有其他方法。如果不是专注于模型推理，我们可以放大视角。

这是VU Amsterdam的博士生Alex de Vries的方法，他曾在他的博客*Digiconomist*上计算比特币的能源消耗，曾经使用Nvidia GPU（AI硬件的黄金标准）来估计该行业的全球能源使用情况。正如去年de Vries在*Joule*杂志上的评论中解释的那样，Nvidia大约占AI市场销量的百分之九十五。该公司还发布了其硬件的能源规格和销售预测。

通过结合这些数据，de Vries计算到2027年，AI行业的年能耗可能在85至134太瓦时之间。这大约等同于de Vries家乡荷兰的年度能耗。

de Vries告诉《The Verge》：“到2027年，你说的AI电力消耗可能占全球电力消耗的百分之五十，我觉得这是一个相当大的数字。”

国际能源署最近的一份报告提供了[类似的估计](/2024/1/24/24049047/data-center-ai-crypto-bitcoin-mining-electricity-report-iea)，表明由于人工智能和加密货币的需求，数据中心的电力使用将在不久的将来显著增加。该机构称，目前数据中心的能源使用量为2022年约460太瓦时，到2026年可能会增加到620至1050太瓦时 —— 相当于瑞典或德国的能源需求。

但德·弗里斯表示，将这些数据放入背景中是很重要的。他指出，2010年至2018年间，数据中心的能源使用基本保持稳定，在全球能源消耗中约占1%至2%。德·弗里斯说，在此期间，需求确实上升了，但硬件效率提高了，从而抵消了增加的部分。

他担心的是，对于人工智能来说可能会有所不同，这正是因为公司倾向于简单地将更大的模型和更多的数据投入到任何任务中。“这对效率来说是一个非常致命的动态，”德·弗里斯说。“因为这创造了人们只增加更多计算资源的自然动机，一旦模型或硬件变得更高效，人们将使这些模型比以前更大。”

是否效率提高能够抵消不断增长的需求和使用量的问题无法回答。像Luccioni一样，德·弗里斯对可用数据的缺乏感到遗憾，但他说世界不能简单忽视这种情况。“弄清楚这个问题的走向有点像破解，肯定不是一个完美的数字，”他说。“但这足以提醒我们警惕。”

一些涉足人工智能的公司声称技术本身可以帮助解决这些问题。代表微软发言的普里斯表示，人工智能“将是推动可持续解决方案的强大工具”，并强调微软正在努力实现到2030年成为“碳负、水正和零废弃”的可持续发展目标。

但是一个公司的目标永远不能涵盖整个行业的需求。可能需要其他方法。

鲁奇奥尼表示，她希望看到公司为人工智能模型引入能源星级评定，让消费者可以像评估家电一样比较能源效率。对于德·弗里斯来说，我们的方法应更为基本：我们是否真的需要在特定任务中使用人工智能？“因为考虑到人工智能的所有限制，它可能不会在许多地方成为正确的解决方案，我们将浪费大量时间和资源才能弄清楚这一点，”他说。
