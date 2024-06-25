<!--yml

category: 未分类

日期：2024-05-27 14:43:49

-->

# Fortran是“一门死语言”吗？- 作者：Jim Cownie - CPU fun

> 来源：[https://cpufun.substack.com/p/is-fortran-a-dead-language](https://cpufun.substack.com/p/is-fortran-a-dead-language)

最近一条推文中含有“Fortran等死语言”的短语，尽管这可能是玩笑，但值得驳斥，因为在做关于投资编译器的决策时，不应将Fortran视为“遗留语言”，甚至不应像上面所说的那样将其视为“死语言”，因为在它的主要领域中，Fortran仍然至关重要。

这种误解有几个原因：-

+   大多数程序员从未学过Fortran，它也不是计算机科学或Web语言。

+   Fortran在2022年9月的[Tiobe指数](https://www.tiobe.com/tiobe-index/)中仅排名第15位，而且在[IEEE的语言流行度估计](https://spectrum.ieee.org/top-programming-languages-2022)中也排名靠后。

+   它于1957年发布（与我一样），因此，由于老年人什么都不懂，而且上世纪的任何东西在技术上都值得怀疑，它显然必须在消失。

这里的问题实际上是封闭性的问题。人们使用的网站和教学资源取决于他们的工作和他们试图解决的任务。因此，我们不会指望会计师是结构力学方面的专家，或者知道如何驾驶叉车。我们被教授使用我们需要的工具来完成我们预期要做的事情，并对此感兴趣。

因此，学习物理和化学等硬科学的人被教授使用Fortran进行编程，而更一般的程序员（如果语言流行指数可信的话，他们可能主要开发Web应用程序）则没有，因为Fortran在那个领域不被使用。

这种情况影响较大的一个群体是编写编译器的人员。由于大多数编译器现在是用C或C++编写的（例如[Gnu Compiler Collection](https://github.com/gcc-mirror/gcc)（GCC）：62.6% C++或C，[LLVM](https://llvm.org/)：84% C++或C），这些是编译器编写者每天使用的语言，因此往往被视为“真正”的语言。特别是，如果他们改进这些编译器，他们会亲身体验到好处。

另外，由于他们对Fortran不感兴趣，许多人不追踪当前Fortran的状态（Fortran持续通过Fortran 2018不断发展，预计明年推出Fortran 2023），但他们却误以为自Fortran 77（或最迟Fortran 90）以来什么都没发生。虽然仍然可以以固定的打孔卡格式编译代码，但现代Fortran是一种更友好的语言，具有高性能计算[HPC]代码的有用内置功能（例如在不需要编写循环的情况下对整个数组进行操作，显式并行循环[`do concurrent`]以及通过[协数组](https://www.nag.com/nagware/np/r62_doc/nag_f2008.html#AUTOTOC_3_3:~:text=3.3-,Coarrays,-Coarrays%20are%20variables)支持分区全局地址空间[PGAS]并行性）。

的确，Fortran在编程流行度指数中排名较低，但它们并未测量任何给我们有关语言重要性的有用信息，而是告诉我们有多少人在询问它（TIOBE），或者有多少人在使用它([IEEE](https://spectrum.ieee.org/top-programming-languages-2022#:~:text=we%20look%20at%20nine%20metrics%20that%20we%20think) : “我们查看了九项我们认为是衡量人们正在使用哪些语言的良好代理指标。”）

然而，这些流行度指标忽略了编写某种语言的人数与代码的重要性以及它可能消耗的计算资源之间的完全不同的问题。

以维也纳第一性原理模拟软件包[[VASP](https://www.vasp.at/)]为例，这是英国最大的HPC机器（[Archer2](https://www.archer2.ac.uk/)）上消耗最多资源的代码，它是用Fortran编写的。在VASP网站上，我们可以看到[VASP团队的照片](https://www.vasp.at/info/team/)，该团队由15人组成。显然，15个人编写Fortran对流行指数的统计没有任何影响，然而，正如我们将在下面看到的，他们的代码非常重要。

由于我无法理解的原因，“传统”的成为了一种侮辱，尽管我们大部分的知识都是来自前几代人的传承。（我没有看到物理学家因为能量守恒不是这一代人理论化而将其撕毁，演员抱怨要学习莎士比亚剧本的角色，或者音乐家抱怨演奏莫扎特、巴赫、贝多芬或[希尔德加德·冯·宾根](https://en.wikipedia.org/wiki/Hildegard_of_Bingen)，她于1169年去世！）。

有许多经过充分测试的Fortran代码正在被使用和维护，而不是被扔掉以被代价高昂、新的、未经测试的代码所取代，这是一件好事，而不是坏事。

尽管你可能会认为你得到的天气预报很糟糕，但这些预报能够挽救生命，而且许多都是用 Fortran 编写的（例如美国的 [WRF](https://github.com/wrf-model/WRF)，英国气象局的统一模型 [UM]，以及欧洲中期天气预报中心的综合预报系统 [IFS]）。

尽管Fortran可能对于硬核计算机科学家来说没有用处，但对于寻找一个在实现其真正目标——推动其领域知识进步——时需要的具有良好理解性、高性能、可移植性的语言的科学家来说，它仍然是最重要的语言之一。

众所周知，硬件仍在不断进步（尽管[丹纳德缩放](https://en.wikipedia.org/wiki/Dennard_scaling)的结束产生了影响），但拥有一个我们可以继续使用的工具是重要的。如果我们买了一个十字螺丝刀，我们不会扔掉我们已经拥有的直槽单槽螺丝刀，尽管两者都是螺丝刀，而是我们意识到我们需要两者都放在我们的工具箱里。同样地，扔掉Fortran代码是没有意义的。Fortran也在[不断发展](https://wg5-fortran.org/)，只是对于那些大声抱怨C++标准状态的人来说看不到而已！

通过添加[OpenMP](https://openmp.org)®，Fortran代码可以以供应商中立的方式扩展，以在既有CPU又有GPU或其他加速器的机器上运行。

许多人（包括一些自己编写代码的人，如果他们是理性的，就不会期望得到报酬）认为软件，尤其是编译器，应该是免费的。然而，在现实世界中，必须有人支付开发人员的费用（[glassdoor](https://www.glassdoor.co.uk/Salaries/us-c-developer-salary-SRCH_IL.0,2_IN1_KO3,14.htm?countryRedirect=true) 在2022年9月估计美国平均C++开发人员工资为$94,466/年，这意味着雇佣一名C++开发人员的成本可能在考虑到办公空间、设备、差旅费等因素后约为每年$150k）。当然，这意味着支付开发人员的人必须能够看到他们提供的代码的重要价值，即使它是免费提供的，因为我们生活在一个资本主义经济体系中，他们希望获得利润。

用于高性能计算 [HPC] 的编译器大多由硬件供应商编写，通常意味着 CPU（或 GPU）芯片的设计者。

有很多好理由：-

+   在新芯片发布前准备好编译器工作必须在规格公开之前开始，因此一个无关的软件公司不能做到这一点。

+   如果他们是明智的，硬件架构师应该担心他们美妙的新功能是否可以被编译器使用，因此他们需要与编译器编写者进行交流，并在新指令的规范最终确定之前接受他们的反馈。

+   由于CPU或GPU的售价与其性能相关，因此具有改善任何用于此评估的基准代码性能的编译器是有价值的，因为它使您能够为您销售的每台机器收取更高的价格。事实上，这可能是实现性能的更简单、更便宜的方法，而不是使用改进的工艺来制造芯片，或者使其以更高的时钟频率运行（从而消耗更多的功率）。

+   拥有内部编译器团队可以在运行特定采购所需的基准测试时快速修复问题。

+   要在特定市场上竞争，那里的客户必须能够在您试图销售的机器上运行他们的代码。在高性能计算市场上，这意味着必须具备良好的Fortran支持。

最后一个原因应该很明显，即程序员使用一种语言的数量并不是资助编译器开发人员使用的正确指标。由于他们并没有出售编译器并向每个用户收取许可证费用，因此使用编译器的人数是无关紧要的。重要的是运行需要支持编译器支持的语言的代码所需的硬件量。

幸运的是，爱丁堡并行计算中心（[EPCC](https://www.epcc.ed.ac.uk/)）运行着[Archer2](https://www.archer2.ac.uk/)（这是英国最大的中央资助科学研究机器，也是2022年6月[Top500](https://www.top500.org/system/180036/)超级计算机榜单上的第25名）发布了在该机器上运行的应用程序的[信息](https://www.archer2.ac.uk/support-access/status.html#:~:text=0.0-,Historical%20usage%20data,-Period)。

尽管他们目前不公布每个应用程序所使用的语言的信息，但通过上午在谷歌上搜索和给一些代码作者发几封电子邮件，我已经为大多数代码填写了这些信息。EPCC的统计数据中有很大一部分是“未识别”的代码；我们将讨论如何将这些代码有用地分配给一种编程语言。

查看过去六个月（2022年3月至8月）的统计数据，有43个应用程序使用了超过机器0.1%的资源；那些使用了超过机器1%的资源的应用程序（根据统计数据中的程序名称确定）是

即使乍一看，Fortran似乎很重要，但还有更多讨论。从图形上看，数据如下：-

查看所有代码的结果，而不仅仅是“超过1%”的代码，并按语言聚合，我们得到了以下视图：-

首先，Fortran明显没有退出舞台！

但是，我们应该多做一些分析，对“一个代码是用特定语言编写”的含义进行更多思考。尤其是，尽管在分析中没有提到，但我们可以合理地假设，这台基于 Linux 的计算机上的所有代码都链接到了`libc`，这是用 C 和汇编代码混合编写的，并且大多数将使用动态链接器，该链接器也是用 C 编写的，因此对 C 和汇编支持是基础性的。然而，由于 C **是** 基础性的，可以假定它是存在的，而在供应商工作时，他们需要计算出需要资助哪些项目才能出售 HPC 机器，不必资助 C 编译器。（嘿，如果没有 C 编译器，Linux 就不会存在于这台机器上！）

对于这个问题，他们需要考虑他们提出的机器有多有用，因此，如果他们无法运行包含 Fortran 代码的代码，它能支持多少所需的负载。从这个角度来看，我们可以将用 Fortran + 其他语言编写的代码的使用分配给 Fortran。如果我们这样做，并且（缺乏更好的信息）假设未知代码使用与我们了解的代码一样多的 Fortran，我们会得到以下结果

因此，超过 80% 的机器使用需要支持 Fortran。

我们还可以查看每种语言编写的代码数量（而不是编写它的代码所消耗的计算资源），这向我们表明，并非只有一个重要的 Fortran 代码在占用所有时间，而大多数代码都是用其他语言编写的。相反，每种语言的代码比例与所消耗的资源量相似。

嗯，据信 Archer2 的成本为 7,900 万英镑（约合当时的 1.02 亿美元），其中 81.1% 用于运行 Fortran 代码，使得拥有这样的编译器成为能够支出约 8270 万美元的必要条件。或者，对于 CPU 供应商来说，需要高端、64 核处理器的 9,450 个插槽（总插槽数的 81.1%）来运行基于 Fortran 的代码。[The Next Platform 估计](https://www.nextplatform.com/2019/09/18/amd-revs-up-hpc-variant-as-rome-chips-ramp/#:~:text=Our%20guess%20is%20that%20list%20price%20is%20around%20%249%2C000%20each%20for%20the%20Epyc%207H12)，这些插槽中的 CPU 的标价约为 9000 美元，因此即使打了 50% 的折扣，这也是一笔价值 4250 万美元的单笔销售，完全依赖于运行 Fortran。当然，这只是一次采购。

看起来值得资助一个编译器！

+   从个人经验泛化可能会产生误导。仅因为你不认识任何 Fortran 程序员，也不自己编写这种语言，这并不意味着它不重要。（我不认识任何长途卡车司机或火车司机，但他们确实存在，并且运送我生存所需的食物和商品！）

+   “免费”软件必须在某个地方付费，值得稍微思考一下是如何实现的。

+   如果你想提及一门已经不再使用的语言，也许可以使用[B](https://en.wikipedia.org/wiki/B_(programming_language))，[BCPL](https://en.wikipedia.org/wiki/BCPL)，或[Algol](https://en.wikipedia.org/wiki/ALGOL)，而不是Fortran。这将表明你对编程语言历史有更多了解，而且不会说出被证明是错误的话。

+   在侮辱编程语言时要小心你的愿望；微软Azure的CTO Mark Russinovich最近建议[应该淘汰C和C++，转而使用Rust](https://twitter.com/markrussinovich/status/1571995117233504257)，所以也许Fortran将比C和C++更长寿。

+   而且，最重要的是：**Fortran 现在仍然活跃。**

感谢EPCC Archer2团队将他们的使用统计数据公开，并感谢那些迅速回复我邮件的人告诉我他们的代码是用哪种语言编写的。

我编写的脚本用于聚合数据（其中包括应用到语言的映射）和我使用的数据文件都在GitHub [这里](https://github.com/JimCownie/CpuFun/tree/main/Archer2Stats)。为了生成图表，我从[summarise.py](https://github.com/JimCownie/CpuFun/blob/main/Archer2Stats/summarise.py)的输出中剪切/粘贴数据到电子表格中。（抱歉，我想尽快完成这个任务，而且不指望经常运行它，所以没必要完全自动化这项任务）。当然，任何想要的人都可以拿取那段代码并加以改进，或者只提取他们需要的部分（比如应用程序名称→语言字典）。

EPCC用于生成他们统计数据的代码（“SLURM Code Usage Analysis” [SCUA]）都在GitHub [这里](https://github.com/ARCHER2-HPC/usage-analysis)可用，并包括他们的应用程序→语言映射数据（我使用了一个附加数据）。

如果有人对跟踪资金流向和作为软件工程师的成长有更广泛的讨论感兴趣，[我在2020年远程发表的演讲](https://www.ucl.ac.uk/research-it-services/sites/research_it_services/files/socials-20200715-cownie.pdf)可能会引起兴趣。