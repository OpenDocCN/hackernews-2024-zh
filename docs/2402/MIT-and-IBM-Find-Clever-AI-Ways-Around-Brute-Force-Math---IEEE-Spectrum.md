<!--yml

category: 未分类

date: 2024-05-27 14:37:18

-->

# MIT 和 IBM 发现了巧妙的 AI 方法来避开 brute-force 数学 - IEEE Spectrum

> 来源：[https://spectrum.ieee.org/mathematical-model-ai](https://spectrum.ieee.org/mathematical-model-ai)

自艾萨克·牛顿时代以来，自然的基本法则最终都可以归结为一组重要而广泛的方程。现在，研究人员找到了一种新方法，利用类似大脑的神经网络比以往更高效地解决这些方程，适用于科学和工程的众多潜在应用。

在现代科学和工程中，[偏微分方程](https://people.math.harvard.edu/~knill/pedagogy/pde/index.html)有助于建模[涉及多种变化速率的复杂物理系统，例如在空间和时间上都发生变化的系统](https://uwaterloo.ca/applied-mathematics/future-undergraduates/what-you-can-learn-applied-mathematics/differential-equations/partial-differential-equations-pdes)。它们可以帮助[建模](https://www.ias.edu/ideas/curiosities-partial-differential-equations)各种事物，如飞机机翼上空气流动，空气中污染物的扩散，或星体坍缩成黑洞。

为了解决这些困难的方程，科学家传统上使用高精度数值方法。然而，这些方法可能非常耗时，且需要大量计算资源来运行。

目前，存在较简单的替代方案，称为[数据驱动的替代模型](https://en.wikipedia.org/wiki/Surrogate_model)。这些模型，包括[神经网络](https://spectrum.ieee.org/deep-neural-network)，通过从数值求解器中训练数据来预测它们可能产生的答案。然而，这些模型仍然需要大量来自数值求解器的数据进行训练。随着这些模型规模的增大，所需的数据量呈指数增长，这使得这种策略难以扩展，乔治亚理工学院亚特兰大分校的计算科学家、研究首席作者[Raphaël Pestourie](https://cse.gatech.edu/people/raphael-pestourie)说道。

在一项新研究中，研究人员开发了一种建立替代模型的方法。这种策略使用物理模拟器来帮助训练神经网络，使其能够与高精度数值系统的输出相匹配。其目的是通过领域内的专家知识（在这种情况下是物理学）生成准确的结果，而不仅仅是简单地投入大量计算资源来 brute-force（暴力求解）解决这些问题。

研究人员发现，数值替代模型（这里用詹姆斯·克勒克·麦克斯韦的卡通形象来象征）可以解决之前需要高精度 brute-force 数学（用麦克斯韦的达盖尔类型来象征）才能解决的艰难数学问题。MIT

科学家们测试了他们称之为物理增强深度代理（PEDS）模型在三种物理系统上的应用。包括扩散，比如染料随时间在液体中的扩散；反应-扩散，比如化学反应后可能发生的扩散；以及电磁散射。

研究人员发现，这些新模型在处理偏微分方程时可以比其他神经网络精度高出三倍。同时，这些模型只需约1000个训练点即可。这减少了至少100倍的训练数据，以达到5%目标误差。

“这个想法非常直观——让神经网络进行学习，让科学模型进行科学探索”，Pestourie 说道。“PEDS 表明，结合两者远比各自单独使用效果好得多。”

PEDS 模型的潜在应用包括加速“工程中到处都能见到的复杂系统的模拟——比如天气预报、碳捕集和核反应堆”，Pestourie 说。

科学家们在《自然·机器智能》期刊中详细描述了[他们的发现](https://www.nature.com/articles/s42256-023-00761-y)。

*2024年2月5日更新，删除了错误陈述的自然法则。《光谱》对此表示遗憾。*

来自您网站的文章

相关文章参见网络资源
