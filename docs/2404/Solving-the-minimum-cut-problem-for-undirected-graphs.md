<!--yml

category: 未分类

date: 2024-05-27 13:16:40

-->

# 解决无向图的最小割问题

> 来源：[https://research.google/blog/solving-the-minimum-cut-problem-for-undirected-graphs/](https://research.google/blog/solving-the-minimum-cut-problem-for-undirected-graphs/)

[图（discrete_mathematics）](https://zh.wikipedia.org/wiki/%E5%9B%BE_(%E6%95%B0%E5%AD%A6))是计算机科学中广泛使用的一种数据结构，由节点（或顶点）和连接这些节点对的边组成，用于捕捉对象及其关系。[最小割问题](https://zh.wikipedia.org/wiki/%E6%9C%80%E5%B0%8F%E5%89%B2)（通常称为“min-cut”）是关于图的连接性的基本结构问题，询问：断开网络的最经济方式是什么？更正式地说，给定一个边没有方向（即图是*无向*的）并且与正权重相关联的输入图（例如道路的容量、关系的强度、端点之间的相似度水平等），一个切割是将节点分成两侧的一个分割。切割的大小是连接切割两侧节点的边的总权重，而最小割问题就是找到一个最小大小的切割。

在算法图论中，高效解决这一问题一直是最基本的问题之一。此外，最小割在实践中有着多样的应用，如图像恢复、计算机视觉中的立体和分割，以及网络韧性分析（如道路或电网）。当底层图数据过大需要分割成较小组件以便进行分治处理时，最小割也通常非常有用。

在算法设计理论中，对于任何需要读取整个输入的问题（例如最小割问题），其渐近复杂度至少是与输入大小线性相关的（因为这是读取输入所需的时间）。一个[接近线性时间](https://web.eecs.umich.edu/~gurevich/Opera/82.pdf)的算法基本上实现了这个下界，因此被认为是可以达到的最佳结果。对于最小割问题，现有的接近线性时间算法要么是随机的（可能以一定概率输出错误答案），要么仅适用于图是[简单的](https://mathworld.wolfram.com/SimpleGraph.html)情况（无法模拟许多现实世界的应用），因此其最优复杂性仍然是一个未解决的问题。

在《[加权图中确定性近线性时间最小割](https://epubs.siam.org/doi/10.1137/1.9781611977912.111)》一文中，该论文荣获 ACM-SIAM 离散算法研讨会（[SODA2024](https://www.siam.org/conferences/cm/conference/soda24)）的最佳论文奖。我们设计了第一个几乎线性的确定性算法，用于解决最小割问题，能够在所有情况下找到正确答案，同时适用于一般的图结构，从而确定了最小割问题的最优复杂度。
