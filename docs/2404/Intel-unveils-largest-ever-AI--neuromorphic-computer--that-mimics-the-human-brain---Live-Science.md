<!--yml

类别：未分类

date: 2024-05-27 13:20:20

-->

# 英特尔推出最大规模的AI类脑计算机，模仿人类大脑 | Live Science

> 来源：[https://www.livescience.com/technology/computing/intel-unveils-largest-ever-ai-neuromorphic-computer-that-mimics-the-human-brain](https://www.livescience.com/technology/computing/intel-unveils-largest-ever-ai-neuromorphic-computer-that-mimics-the-human-brain)

英特尔的科学家们建造了世界上最大的类脑计算机，或一种旨在模仿**人类大脑**的设备（参见 <https://www.livescience.com/29365-human-brain.html>）。该公司希望这款设备能支持未来的AI研究。

称为 "Hala Point" 的这台机器相比于使用中央处理器（CPU）和图形处理器（GPU）的传统计算系统，其执行AI工作负载的速度提高50倍，能源使用效率提高100倍（英特尔代表在 <https://www.intel.com/content/www/us/en/newsroom/news/intel-builds-worlds-largest-neuromorphic-system.html#gs.84tuhb> 的声明中提及）。这些数据显示，基于3月18日上传至预印论文服务器IEEE Explore（<https://ieeexplore.ieee.org/document/10448003>）上还未经过同行评审研究得出的数据。

Hala Point 初始部署于美国新墨西哥州的桑迪亚国家实验室，科学家在那里使用它来解决设备物理学、计算架构和计算机科学领域的问题。

**相关链接**：[中国开发新光基芯片组件，有可能推动人工智能——这里的人工智能比人类更聪明](https://www.livescience.com/technology/electronics/china-develops-light-based-chiplet-power-agi-artificial-general-intelligence)

这款大型系统由 1,152 枚英特尔新的 Loihi 2 处理器（类脑研究芯片）提供动力，包含 115 亿个人工神经元和 1,280 亿个人工突触，分布在 140,544 个处理核心上。

这台机器每秒能够执行 20 万亿次操作，或者说 20 petaops。类脑电脑以不同于超级计算机的方式处理数据，因此很难将它们相互对比。不过，全球第38强大的超级计算机“Trinity”拥有大约20 petaFLOPS的计算能力（每秒浮点运算量）。全球最强大的超级计算机“前线（Frontier）”则拥有1.2 exaFLOPS的性能，相当于1,194 petaFLOPS。

## 如何理解类脑计算的工作原理

神经形态计算与传统计算不同，因为其体系结构由[Prasanna Date](https://www.researchgate.net/profile/Prasanna-Date)，美国奥克里奇国家实验室（ORNL）的计算机科学家，在[ResearchGate](https://www.researchgate.net/figure/Comparison-of-the-von-Neumann-architecture-with-the-neuromorphic-architecture-These_fig1_358255092)上写道。这些类型的计算机使用神经网络构建机器。

获取世界上最引人入胜的发现，直接送到您的收件箱。

在经典计算中，二进制的1和0位流入硬件，如CPU、GPU或内存，在处理计算前按顺序进行处理，并输出二进制输出。

（图片来源：瓦尔登·基尔什/英特尔公司）

然而，在神经形态计算中，“尖峰输入”——一组[离散电信号](https://www.ncbi.nlm.nih.gov/pmc/articles/PMC9313413/#:~:text=In%20SNNs%2C%20such%20as%20in,and%20work%20in%20continuous%20time.)——被输入到尖峰神经网络（SNNs）中，这些网络由处理器表示。在软件基础的神经网络中，是一组排列成仿真人脑的机器学习算法，而SNNs是这些信息传递方式的物理体现。它允许并行处理，并根据计算后的尖峰输出进行测量。

类似于大脑，Hala Point和Loihi 2处理器使用这些SNNs，其中不同的节点相连，信息在不同层次进行处理，类似于大脑中的神经元。芯片还将内存和计算能力集成在一个地方。在传统计算机中，处理能力和内存是分离的；这会导致数据必须在这些组件之间物理传输，从而形成瓶颈。这两者都实现了并行处理并减少了功耗。

## 为什么神经形态计算可能是人工智能的游戏改变者

早期结果还显示，Hala Point在AI工作负载的能效读数达到了每瓦15万亿次操作（TOPS/W）。大多数传统的神经处理单元（NPU）和其他AI系统的能效远低于[10 TOPS/W](https://basicmi.github.io/AI-Chip/)。

神经形态计算仍然是一个发展中的领域，如果有的话，像Hala Point这样的机器很少部署。然而，澳大利亚西悉尼大学国际神经形态系统中心（ICNS）的研究人员[宣布计划在2023年12月部署类似的机器](https://www.westernsydney.edu.au/newscentre/news_centre/more_news_stories/world_first_supercomputer_capable_of_brain-scale_simulation_being_built_at_western_sydney_university)。

他们的计算机“DeepSouth”以每秒228万亿次突触操作模拟大规模的尖峰神经网络，ICNS研究人员在声明中说道，这相当于人类大脑操作的速率。

据英特尔代表称，Hala Point与此同时是一个“起点”，一个研究原型，最终将被纳入未来可以商业化部署的系统。

这些未来的神经形态计算机甚至可能导致大型语言模型（LLMs），例如ChatGPT，持续从新数据中学习，这将减少当前人工智能部署中固有的巨大训练负担。
