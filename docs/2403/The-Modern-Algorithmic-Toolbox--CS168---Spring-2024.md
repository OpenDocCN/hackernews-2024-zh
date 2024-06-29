<!--yml

类别：未分类

日期：2024年5月29日12:28:57

-->

# 现代算法工具箱（CS168），2024年春季学期

> 来源：[https://web.stanford.edu/class/cs168/index.html](https://web.stanford.edu/class/cs168/index.html)

# CS 168：现代算法工具箱，2024年春季学期

## **公告：**

+   5/28：[迷你项目 #9](p9.pdf) 已发布。这里是[close.csv](close.csv)和[tickers.csv](tickers.csv)数据文件。项目截止日期为6月6日（星期四），上午11点。

+   5/23：这里是一个[期末考试练习](practice_final.pdf)。

+   5/22：[迷你项目 #8](p8.pdf) 已发布。这里是[laurel_yanny.wav](laurel_yanny.wav)音频剪辑，用于第三部分，以及一个压缩的文件夹[worldsYikes.zip](worldsYikes.zip)，包含我们创建的例子，证明了10分的研究导向奖励部分并非不可能：）也可以查看这篇关于这个奖励部分的研究论文[这里](https://theory.stanford.edu/~valiant/polyperceivable/index.html)，该论文源于两位前CS68学生的工作。

+   5/14：[迷你项目 #7](p7.pdf) 已发布。截止日期为5月23日（星期四），上午11点。[这里](parks.csv)是国家公园列表，以及第二部分的[plot_route.py](plot_route.py)。对于第三部分，MATLAB的QWOP脚本可以在[这里](qwop.m)找到，Python的可以在[这里](qwop.py)找到。

+   5/8：[迷你项目 #6](p6.pdf) 已发布。截止日期为5月16日（星期四），上午11点。[这里](cs168mp6.csv)是第二部分的数据文件。

+   5/1：[迷你项目 #5](p5.pdf) 已发布（这是我的其中一个最爱！）。截止日期为5月9日，上午11点。你需要的数据文件有：[dictionary.txt](dictionary.txt)，[co_occur.csv](co_occur.csv)，[analogy_task.txt](analogy_task.txt)，以及[p5_image.gif](p5_image.gif)。

+   4/23：[迷你项目 #4](p4.pdf) 已发布。基因组数据集可以在[这里](p4dataset2024.txt)找到，解码人口统计标签的简短文件在[这里](p4dataset2024_decoding.txt)。截止日期为5月2日（上午11点）。

+   4/16：[迷你项目 #3](p3.pdf) 已发布。截止日期为4月25日（上午11点）。

+   4/9：[迷你项目 #2](p2.pdf) 已发布。数据集可以[在这里](p2_data.zip)找到。截止日期为4月18日（星期四），上午11点。

+   大多数助教的办公时间现已发布，尽管在接下来的几天中可能会略有变动。在明确哪些办公时间最受欢迎之后，我们可能会在几周后调整时间表。

+   这里是[LaTex解决方案模板迷你项目](sol_template.tex)。你也可以使用自己的解决方案模板，尽管通常助教更希望你在解答中不包括问题提示，因为这会增加很多长度/混乱。

+   这里是[迷你项目 #1](p1.pdf)。截止日期为4月11日（星期四），上午11点。请通过Gradescope提交，输入代码为2PN4J7。[这里](histogram.py)提供了绘制Python直方图的一些起始代码，可能会有帮助。

+   第一次讲座于4月2日星期二，下午1:30至2:50在NVIDIA Auditorium举行。到时见！

+   讲座视频将通过Canvas提供，尽管我强烈建议您如有可能亲自参加讲座——对我们所有人来说更有趣。

+   [**这是我们Ed在线讨论论坛的链接。**](https://edstem.org/us/join/ZYbyeA)

**## 讲师：**

+   [Gregory Valiant](http://theory.stanford.edu/~valiant/)（办公时间：周二下午3-4点，Gates 162）。联系方式：通过我的姓氏在stanford.edu上的电子邮件或通过我们的课程Ed页面联系我。

## **课程助教：**

+   Sidhant Bansal（办公时间：周一下午4:15-6:15，Huang Basement）

+   Luci Bresette（办公时间：周三上午10:30-11:30，周五上午10:30-11:30，Huang Basement）

+   Isaac Gorelik（办公时间：周一上午11-1，Huang Basement）

+   Meenal Gupta（办公时间：周三晚上8-10点，[Zoom链接，入口密码856878](https://stanford.zoom.us/j/91570040226?pwd=b3hESUVKdVB6N01kNCtZTXYyTlFZQT09)）

+   Benson Kung（办公时间：周三11-1，[Zoom链接](https://stanford.zoom.us/j/9649412299?pwd=eERaSGZnVU5NeVp4ZmVOSmZXZ3VHZz09)）

+   Ali Malik（办公时间：周三晚上6-8点，Huang Basement）

+   Ligia Melo（办公时间：周一上午9-11点，Huang Basement）

+   Joey Rivkin（办公时间：周二上午10:45-11:45和周三上午10-11点，Huang Basement）

+   Luna Yang（办公时间：周三下午2-4点，Huang Basement）

## **讲座时间/地点：**

周二/周四，下午1:30-2:50在NVIDIA Auditorium

[**Ed网站用于在线讨论/问题。**](https://edstem.org/us/join/ZYbyeA) 这个链接包括您第一次注册时需要的访问代码。

## **先决条件：**

CS107和CS161，或者得到教师许可。

## 课程描述

本课程将提供对现代算法工具包核心思想和算法的深入且动手的介绍。重点将放在理解我们讨论的算法背后的高层次理论直觉和原则上，以及在何时以及如何实现和应用这些算法的具体理解上。课程将被组织成一系列为期一周的研究；每周将介绍一个算法思想，并讨论该算法思想的动机、理论基础和实际应用。每个主题都将伴随一个迷你项目，指导学生如何实际应用这周的思想。主题包括现代哈希技术、维度缩减、线性和凸优化、梯度下降和回归、抽样和估计、压缩感知、线性代数技术（主成分分析、奇异值分解、频谱技术）以及差分隐私简介。

## 建议的讲座时间表

+   ### 第1周：现代哈希

    +   **讲座1（周二4/2）：** 课程介绍。``一致''哈希。补充材料：

    +   **讲座2（周四4/4）：** 保持属性的有损压缩。从大多数元素到近似重要元素。从布隆过滤器到计数最小草图。补充材料：

+   ### 第2周：具有距离的数据（相似性搜索、最近邻、维度缩减、LSH）

    +   **第3讲（周二4/9）：** 相似性搜索。相似（不相似）度量：Jaccard，欧几里得，Lp。使用k-d树在小/中维度（即<20）中找到相似元素的高效算法。补充材料：

    +   **第4讲（周四4/11）：** 维数诅咒，接吻数。保持距离的压缩。使用MinHash估计Jaccard相似性。保持欧几里得距离的维度约简（即约翰逊-林登斯特劳斯变换）。补充材料：

        +   关于高维空间中的“接吻数”和其他一些奇怪现象的良好调查：[接吻数、球填充和一些意想不到的证明](http://www.ams.org/notices/200408/fea-pfender.pdf)（2000年出版）。

        +   MinHash在Alta Vista的起源：Broder，[识别和过滤近似重复文档](http://cs.brown.edu/courses/cs253/papers/nearduplicate.pdf)（2000年出版）。

        +   Ailon/Chazelle，[更快的维度约简](https://www.cs.princeton.edu/~chazelle/pubs/fasterdim-ac10.pdf)，CACM '10。

        +   Andoni/Indyk，[高维近似最近邻的近似最优哈希算法](http://mags.acm.org/communications/200801/#pg119)，CACM '08。

        +   想要了解更多关于局部敏感哈希的信息，请参阅CS246教材的[本章](http://infolab.stanford.edu/~ullman/mmds/ch3.pdf)（Leskovec、Rajaraman和Ullman著）。

+   ### 第3周：泛化和正则化

    +   **第5讲（周二4/16）：** 泛化（或者说需要多少数据才够？）。从未知分布的样本学习未知函数。训练误差与测试误差。线性分类器的PAC保证。经验风险最小化。

    +   **第6讲（周四4/18）：** 正则化。多项式嵌入和随机投影，L2正则化，以及作为L0正则化的计算可行替代品的L1正则化，正则化的频率主义和贝叶斯视角。补充材料：

        +   一篇2016年的论文认为，要理解深度学习为什么有效，我们需要重新思考泛化理论。这篇论文颇具争议，一方认为其结论显而易见，另一方则认为其揭示了一个极其深奥的谜团。你可以自己决定！论文链接在[这里](https://arxiv.org/pdf/1611.03530.pdf)。

+   ### 第4周：线性代数技术：理解主成分分析

    +   **第7讲（周二4/23）：** 理解主成分分析（PCA）。最小化平方距离等于最大化方差。数据可视化和数据压缩的用例。PCA的故障模式。补充材料：

        +   23andMe关于欧洲人基因SNP数据前两个主成分基本恢复了欧洲地理结构的阐述：[详细说明及图表](http://blog.23andme.com/news/a-different-kind-of-gene-mapping-comparing-genetic-and-geographic-structure-in-europe-the-return/)。原始的Nature论文：[基因反映欧洲地理](http://www.nature.com/nature/journal/v456/n7218/abs/nature07331.html)，Nature，2008年8月。

        +   [特征脸](http://www.cs.ucsb.edu/~mturk/Papers/jcn.pdf)（还请参阅这篇[博客文章](http://jeremykun.com/2011/07/27/eigenfaces/)）

        +   有大量关于PCA的教程在网络上流传（有些好，有些不那么好），您也可以参考。

    +   **讲座 8 (周四 4/25):** PCA的工作原理。最大化方差等同于寻找协方差矩阵的“最大伸展方向”。“伪装成对角线”的简单几何性质。幂迭代算法。

+   ### 第5周：线性代数技术：理解奇异值分解和张量方法

    +   **讲座 9 (周二 4/30):** 低秩矩阵逼近。奇异值分解（SVD），应用于矩阵压缩，去噪和矩阵补全（即恢复丢失的条目）。补充材料：

        +   对于低秩逼近的一个引人注目的最新应用，请查看[LoRA paper](https://arxiv.org/abs/2106.09685)，于2021年发表。要点是在许多情况下，当调整大型语言模型时，更新接近低秩。因此，可以在低秩因子化参数空间中明确地训练这些调整更新到原始模型，从而减少需要训练的参数数量1000倍或10000倍！！

    +   **讲座 10 (周四 5/2):** 张量方法。矩阵与张量的区别，低秩张量因子分解的唯一性，以及Jenrich算法。补充材料：

        +   讨论Spearman的原始实验和张量方法动机的博客文章：[这里](http://www.offconvex.org/2015/12/17/tensor-decompositions/)。

        +   参阅Ankur Moitra的课程笔记的第3章[这里](http://people.csail.mit.edu/moitra/docs/bookex.pdf)，获取更多关于张量方法和Jenrich算法的技术深度讨论。

+   ### 第6周：光谱图论

    +   **讲座 11 (周二 5/7):** 图作为矩阵及图的拉普拉斯矩阵。对拉普拉斯矩阵最大和最小特征向量/特征值的解释。谱嵌入，以及应用概述（例如图着色，谱聚类）。补充材料：

        +   Dan Spielman关于光谱图论的[优秀讲座笔记](http://www.cs.yale.edu/homes/spielman/561/)。这些笔记包含许多有用的图表。

        +   几年前，阿明·萨贝里曾经开设过一个研究前沿的光谱图论研讨会。希望他不久将再次提供此课程...

    +   **第12讲（周四 5/9）：** 光谱技术，第二部分。第二特征值的解释（通过导电和等周数），以及与随机行走/扩散收敛速度的关系。

+   ### 第7周：抽样与估计

    +   **第13讲（周二 5/14）：** 水库抽样（如何从数据流中选择随机样本）。基本概率工具：马尔可夫不等式和切比雪夫不等式。重要抽样（如何基于来自不同分布的样本对一个分布进行推断）。Good-Turing 估计缺失/未见质量。补充材料：

        +   我的一篇论文展示了可以准确估计分布的未观察部分的结构---不仅仅是总概率质量。[论文链接在这里](https://dl.acm.org/citation.cfm?id=3125643)。

    +   **第14讲（周四 5/16）：** 马尔可夫链，稳态分布。马尔可夫链蒙特卡罗（MCMC）作为从精心制作的分布中采样以解决难题的方法。补充材料：

        +   MCMC 的基本描述，[这里](http://jeremykun.com/2015/04/06/markov-chain-monte-carlo-without-all-the-bullshit/)。

        +   Persi Diaconis 关于 MCMC 的讲座笔记，包括解码替代密码的 MCMC 方法描述，[这里](https://www.researchgate.net/publication/215446278_The_Markov_Chain_Monte_Carlo_Revolution)。

        +   用于拟合极其复杂生物模型的 MCMC 示例：[人类剪接密码...](http://www.sciencemag.org/content/347/6218/1254806.full) Science，2015 年 1 月。

        +   对于对计算机围棋感兴趣的人：[这里](http://www.nature.com/nature/journal/v529/n7587/full/nature16961.html) 是 Google DeepMind 小组 2016 年 1 月的自然期刊论文。

+   ### 第8周：傅里叶视角（及其他基础）。

    +   **第15讲（周二 5/21）：** Fourier 方法，第一部分。补充材料：

        +   傅里叶变换的非常基本介绍，带有一些漂亮的可视化：[这里](http://betterexplained.com/articles/an-interactive-guide-to-the-fourier-transform/)。

        +   斯坦福 Fourier 变换课程的书籍版本，具有许多非常好的应用：[pdf 连接](http://see.stanford.edu/materials/lsoftaee261/book-fall-07.pdf)。

    +   **第16讲（周四 5/23）：** Fourier 方法，第二部分。重点是卷积及其许多应用（快速乘法，物理模拟等）。

        +   第15讲的[笔记](l/l15.pdf)，结合了第15讲。

+   ### 第9周：使用乘法权重的在线学习，以及凸优化。

    +   **第17讲（周二 5/28）：** 在线学习和乘法权重算法，及其应用。

    +   **第18讲（周四 5/30）：** 线性和凸优化，以及通过乘法权重算法解决 LP 问题。

+   ### 第10周：隐私保护计算

    +   **第19讲（周二 6/4）：** 数据分析与机器学习中的差分隐私。

## 课程作业

+   **作业（成绩的70%）**：每周将有9个小型项目，围绕当周涵盖的主题展开。每个小项目包含书面和编程部分。强烈建议您组成最多四名学生的小组共同完成。如果您选择组队，**只需一个成员**提交所有相关文件。

    对于书面部分，建议您使用LaTeX排版作业；我们为您提供了一个[模板](homework.tex)以便您使用。我们将使用[GradeScope](https://gradescope.com/)在线提交系统。请使用您的斯坦福ID创建GradeScope账户，并使用入门代码2PN4J7加入CS168课程。

    对于编程部分，建议您使用Python中的Numpy和Pyplot（[Python教程](https://docs.python.org/3/tutorial/index.html)，[Numpy教程](http://www.numpy.org/)，[Matplotlib教程](https://matplotlib.org/stable/)），matlab（[教程](http://www.cyclismo.org/tutorial/matlab/)），或其他科学计算工具（带有绘图功能）。[这里](https://colab.research.google.com/github/cs231n/cs231n.github.io/blob/master/python-colab.ipynb)是一个完整的Python教程，使用Google的Colab为CS231n课程编写，其中包括使用matplotlib进行绘图的示例。

    作业将于每周二发布，截止时间为隔周星期四上午11点。**不接受任何迟交的作业**，但在计算最终成绩时，我们会放弃您的最低作业成绩。

+   **期末考试（成绩的30%）**：将有一次课堂期末考试，覆盖所有材料，重点是每堂课的高层次要点。在本学期结束前几周，我们将发布一份练习试卷。

## 合作政策

您可以与其他小组高层次地讨论问题，并通过Ed或办公时间联系课程工作人员寻求额外帮助。当然，您也鼓励帮助回答Ed的问题。

您可以参考课程网页链接的课程笔记和研究论文，并可以使用您在网上找到的其他参考资料。您**不得**查阅本课程过去提供的解答或代码。当然，您应该理解并熟悉您名下提交的任何作业内容；如果您参考了其他资料，请确保适当引用，并确保所有文字均为您自己原创。

您也可以使用通用资源来支持您选择的任何编程语言。所有作业都不需要您编写大量代码，*您不得*从朋友或互联网源代码中复制任何代码。如果您使用了未经您编写的辅助函数或代码片段（除了标准Python包等），请在报告中清楚标明来源。

请遵守[诚信守则](https://communitystandards.stanford.edu/policies-and-guidance/honor-code)。
