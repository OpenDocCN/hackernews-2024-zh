<!--yml

category: 未分类

date: 2024-05-29 12:37:59

-->

# 什么是绿色软件，为什么我们需要它？ - IEEE Spectrum

> 来源：[https://spectrum.ieee.org/green-software](https://spectrum.ieee.org/green-software)

**[软件可能正在]** [吞噬世界](https://a16z.com/why-software-is-eating-the-world/)，但它也在加热世界。

2023年12月，来自近200个国家的代表在迪拜参加了联合国气候变化大会COP28，讨论了迫切降低排放的需要。与此同时，[COP28的网站](https://spectrum.ieee.org/internet-carbon-emissions)根据网站可持续性评分工具Ecograder的数据，每次页面加载产生[3.69克二氧化碳](https://ecograder.com/report/bKjho1BxM1MHJRunAgpDGhSC)（CO[2]）。这看似微小，但如果该网站每个月获得1万次访问，其排放量将略高于从旧金山到多伦多的单程航班的排放量。

这并非不可避免。根据Ecograder的分析，未使用的代码、尺寸不当的图片和第三方脚本等因素影响了COP28网站的排放量。这些因素都会影响数据传输、加载和处理的能耗，在用户设备上消耗大量电力。Ecograder指出，修复和优化这些问题可以将网站每页加载的排放量削减高达93%。

尽管软件本身不会释放任何排放物，但它运行在数据中心的硬件上，并通过传输网络传送数据，这些都占据了每年全球能源相关温室气体排放量的约1%。整个信息和通信技术行业对全球温室气体排放的贡献估计为2%至4%。到2040年，这一数字可能达到[14%](https://doi.org/10.1016/j.jclepro.2017.12.239)，几乎与空中、陆地和海上运输的排放量相当。

在软件领域内，[人工智能](https://spectrum.ieee.org/topic/artificial-intelligence/)也有其自身的可持续性问题。AI公司[Hugging Face](https://huggingface.co/) [估算了其BLOOM大语言模型的碳足迹](https://www.jmlr.org/papers/volume24/23-0069/23-0069.pdf)，从设备制造到部署的整个生命周期。该公司发现，BLOOM的最终训练排放了50吨CO[2]，相当于大约从纽约到悉尼的十几次航班。

绿色软件工程是一门新兴的学科，包括建立减少碳排放的应用程序的最佳实践。绿色软件运动正在迅速获得动力。像[Salesforce](https://www.salesforce.com/)这样的公司已经推出了他们自己的[软件可持续性倡议](https://www.salesforce.com/news/stories/green-code-software/)，而[Green Software Foundation](https://greensoftware.foundation/)现在包括64家成员组织，包括科技巨头[Google](https://spectrum.ieee.org/tag/google)、[Intel](https://spectrum.ieee.org/tag/intel)和[Microsoft](https://spectrum.ieee.org/search/?q=microsoft)。但是，如果要防止软件的开发和使用导致排放加剧，该行业必须更广泛地采纳这些实践。

## 什么是绿色软件工程？

通向绿色软件的道路始于十多年前。2013年成立的[世界互联网联盟](https://www.w3.org/)（W3C）[可持续网络设计社区组](https://www.w3.org/community/sustyweb/)，而[Green Web Foundation](https://www.thegreenwebfoundation.org/)则始于2006年，作为了解驱动互联网的能源类型的一种方式。现在，Green Web Foundation正朝2030年实现无化石燃料互联网的宏伟目标迈进。

“已经存在一个关注这一领域的软件开发生态系统的大型部分，只是他们不知道该怎么做”，曾是[Intel](https://spectrum.ieee.org/tag/intel)绿色软件与生态系统总监、现任[Green Software Foundation](https://greensoftware.foundation/)主席兼执行董事[Asim Hussain](https://asim.dev/)说道。

根据Hussain的说法，如何做到这一点涉及到[三大支柱](https://greensoftware.foundation/articles/what-is-green-software)：能源效率，即使用更少的能源；硬件效率，即使用更少的物理资源；以及碳意识计算，即更智能地使用能源。Hussain补充说，碳意识计算是指在来自清洁或低碳源（例如风能和太阳能）的电力供应期间，更多地利用你的应用程序，而在非此时期则减少使用。

## 可持续软件的理由

那么程序员为什么要关心软件的可持续性呢？一方面，绿色软件是高效的软件，使程序员能够培养更快、更高质量的系统，由软件开发公司[Helmes](https://www.helmes.com/)的团队负责人和可持续软件战略师[Kaspar Kinsiveer](https://ee.linkedin.com/in/kaspar-kinsiveer)表示。

这些高效的系统还可能意味着公司的成本更低。“关于绿色软件的一个主要误解是你必须额外做些什么，然后会额外花费”，Kinsiveer说。“不会额外花费——你只需要把事情做对。”

绿色软件是高效的软件，使程序员能够培养更快、更高质量的系统。

其他激励因素，特别是软件业务的一面，是与可持续性相关的即将出台的立法和法规。例如，在欧盟，[企业可持续性报告指令](https://finance.ec.europa.eu/capital-markets-union-and-financial-markets/company-reporting-and-auditing/company-reporting/corporate-sustainability-reporting_en)要求公司报告更多有关其环境足迹、能源使用和排放，包括[与其产品使用相关的排放](https://www.ibm.com/topics/scope-3-emissions)。

然而，其他开发者可能受到[气候危机](https://spectrum.ieee.org/collections/climate-change/)的驱使，希望为未来的世代创造一个宜居的星球。软件工程师在所构建的实际目的和排放方面有着巨大的影响力。

“这不只是代码行。这些代码对人类有影响，”荷兰代尔夫特理工大学可持续软件工程的博士后研究员[June Sallou](https://jnsll.github.io/)说道。她特别强调，由于人工智能对社会的影响，开发者有责任确保他们所创建的不会对环境造成破坏。

## 建设更环保的网站和应用程序

COP28网站的制作者本可以借鉴[Lowwwcarbon](https://lowwwcarbon.com/)等目录的做法，该目录突出了现有低碳网站的例子。例如，总部位于荷兰的网络设计和品牌公司[Tijgerbrood](https://tijgerbrood.nl/en/)的公司网站，每个页面浏览仅排放少于[0.1克碳](https://lowwwcarbon.com/case-study/tijgerbrood/)。

建设像Tijgerbrood一样的可持续网站是一个团队的努力，涉及不同角色——从定义软件需求的业务分析师到设计师、架构师以及负责运营的人员——并包括可以在软件开发过程的每个阶段应用的绿色实践。

首先，分析师必须考虑他们设计的功能、应用程序或软件是否应该在第一时间开发。科技通常是关于创造下一个新事物，但使软件可持续也需要决定不建造什么，这可能需要一种思维上的转变。

设计阶段涉及选择高效的算法和架构。“在进入解决方案之前——而不是之后——考虑可持续性，”巴塞罗那的[Centre Tecnològic de Telecomunicacions de Catalunya](https://www.cttc.cat/sustainable-artificial-intelligence-sai/)的可持续人工智能研究员[Chiara Lanza](https://www.cttc.cat/people/chiara-lanza/)如是说。

在开发阶段，程序员需要专注于优化代码。“我们需要降低运行软件所消耗的总能量。其中一部分将来自于编写高效的[代码]”，可持续数字技术顾问和Green Web Foundation运营总监Hannah Smith如是说。

[Tijgerbrood](https://tijgerbrood.nl/en/)的网站通过使用低分辨率图像和现代图像格式优化了公司的代码，仅在用户滚动到视图中时加载动画，并删除了不必要的代码。这些技术有助于加快数据传输、加载和处理速度。该网站还使用了最小化的JavaScript。“当用户加载一个有很多JavaScript的网站时，它会导致他们在自己的设备上使用更多的能源，因为他们的设备必须处理所有读取和运行JavaScript的工作”，Smith解释道。

在运营方面，你可以采取的最有影响力的行动之一是选择可持续的Web托管或云计算提供商。Green Web Foundation提供了一个工具，可以[检查你的网站是否使用绿色能源](https://www.thegreenwebfoundation.org/green-web-check/)，以及一个[由可再生能源驱动的托管提供商目录](https://www.thegreenwebfoundation.org/tools/directory/)。你还可以询问你的托管提供商是否可以调整你的软件在云中运行的方式，以便峰值使用时由绿色能源提供动力，或在非高峰时段暂停或关闭某些服务。

## 绿色AI之道

程序员在开发AI时也可以应用绿色软件策略。修剪训练数据是使AI系统更环保的主要方法之一。从数据收集和预处理开始，考虑到真正需要多少数据来完成工作是值得的。清理数据集以删除不必要的数据，或仅选择数据集的子集进行训练，这可能会带来好处。

“你的数据集越大，算法处理所有数据的时间和计算量就越多，因此消耗的能源也越多”，Sallou如是说。

例如，在一项[研究](https://www.computer.org/csdl/proceedings-article/ict4s/2022/828600a035/1F8ztEmMGYM)中，Sallou及其同事对六种不同的AI算法进行了研究，这些算法用于检测短信垃圾信息，发现了随机森林算法，它将一系列决策树的输出结合起来进行预测，是最耗能的算法。但将训练数据集的大小减少到20%——即5000个数据点中的1000个——使训练的能耗减少了近75%，而精度仅损失了0.06%。

选择更环保的算法也可以节省碳排放。像[CodeCarbon](https://codecarbon.io/)和[ML CO[2] Impact](https://mlco2.github.io/impact/)这样的工具可以通过估算不同AI模型训练的能耗和碳足迹来帮助做出选择。

## 测量软件碳足迹的工具

要编写绿色代码，开发人员需要一种方法来测量整个系统生命周期内的实际碳排放。考虑到涉及的众多过程，这是一个复杂的壮举。以AI为例，其生命周期包括原材料采集、材料制造、硬件制造、模型训练、模型部署和处理，而并非所有这些阶段都有可用数据。

“目前我们并不了解生态系统的许多重要部分，而且获得可靠数据的访问很困难，” Smith说道。她补充道，最大的需求是来自大型技术数据中心运营商和云服务提供商（如[Amazon](https://spectrum.ieee.org/tag/amazon)、[Google](https://spectrum.ieee.org/tag/google)和[Microsoft](https://spectrum.ieee.org/tag/microsoft)）的“可信赖和可依赖的开放数据”。

在这些数据浮出水面之前，一个更实际的方法是测量软件消耗的能量。Sallou表示：“仅仅知道运行一段软件所消耗的能量就可以影响软件工程师改进代码的方式。”

开发人员自己正在关注对更多测量的呼吁，并且他们正在构建工具来满足这一需求。例如，W3C的可持续网页设计社区组计划提供一个测试套件，以衡量实施其网络可持续性指南的影响。类似地，绿色软件基金会编写了一个[规范](https://github.com/Green-Software-Foundation/sci)，用于计算软件系统的碳强度。为了进行准确的测量，Lanza建议将系统运行的硬件与其他操作隔离，并避免运行可能影响测量的任何其他程序。

开发人员可以使用其他工具来衡量绿色软件工程实践的影响，包括提供云工作负载估计碳排放总览的仪表板，例如[AWS客户碳足迹工具](https://aws.amazon.com/aws-cost-management/aws-customer-carbon-footprint-tool/)和[Microsoft Azure排放影响仪表板](https://www.microsoft.com/en-us/sustainability/emissions-impact-dashboard)；能源分析工具或功率监视器，如[Intel的性能计数器监视器](https://github.com/intel/pcm)；以及帮助计算网站碳足迹的工具，例如[Ecograder](https://ecograder.com/)、[Firefox Profiler](https://profiler.firefox.com/)和[Website Carbon Calculator](https://www.websitecarbon.com/)。

## 未来是绿色的。

绿色软件工程正在不断增长和发展，但我们需要更多的意识来帮助这一学科更加普及。这也是为什么，除了其[绿色软件从业者课程](https://learn.greensoftware.foundation/)外，绿色软件基金会还旨在创建更多的培训课程，其中一些甚至可能导致认证。同样，Sallou 与他人合作教授了一门[可持续软件工程研究生课程](https://luiscruz.github.io/course_sustainableSE/2023/)，其教学大纲是公开的，可作为任何希望开设类似课程的人的基础。他说，早期向学生提供这些知识可以确保他们作为未来的软件工程师将其带到工作场所。

在人工智能领域，[Navveen Balani](https://navveenbalani.dev/)，一位人工智能专家和谷歌云认证会员，同时也是绿色软件基金会的指导委员会成员，指出未来几年人工智能可能固有地包含绿色人工智能原则，就像安全考虑现在已经成为软件开发的一部分一样。“这种转变将使人工智能创新与环境可持续性保持一致，使绿色人工智能不仅成为一种专业，而是领域中的一种隐含标准，”他说。

至于网络，Smith 希望绿色网络基金会能在 2030 年前停止存在。“作为一个组织，我们的梦想是不再需要我们，我们达到了我们的目标，互联网默认就是绿色的，”她说。

Kinsiveer 观察到过去，由于当时硬件不足，软件必须优化和良好构建。“随着硬件性能和创新水平的提升，编程质量本身下降了，”他说。但现在，行业正在全面回归，重返其效率根源，并加入可持续性因素。

“未来是绿色软件，” Kinsiveer 说。“我无法想象还有其他方式。”

From Your Site Articles

Related Articles Around the Web
