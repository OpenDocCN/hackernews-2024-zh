<!--yml

类别: 未分类

日期: 2024-05-27 14:44:22

-->

# 热力学人工智能正变得越来越热门 - Grigory Sapunov

> 来源：[https://gonzoml.substack.com/p/thermodynamic-ai-is-getting-hotter](https://gonzoml.substack.com/p/thermodynamic-ai-is-getting-hotter)

**标题**: 用于AI应用的热力学计算系统,

**作者**: 丹尼斯·梅兰松, 马哈茂德·阿布·卡特, 麦克斯韦尔·艾弗尔, 凯兰·多纳泰拉, 马克斯·亨特·戈登, 托马斯·阿勒, 加文·克鲁克斯, 安东尼奥·J·马丁内斯, 法里斯·萨巴希, 帕特里克·J·科尔斯

**论文**: [https://arxiv.org/abs/2312.04836](https://arxiv.org/abs/2312.04836)

**博客**:  [https://blog.normalcomputing.ai/posts/2023-11-09-thermodynamic-inversion/thermo-inversion.html](https://blog.normalcomputing.ai/posts/2023-11-09-thermodynamic-inversion/thermo-inversion.html)

[Normal Computing](https://normalcomputing.ai/)的工作介绍了一种新型硬件：**热力学计算机**和**随机处理单元（SPU）**。文章描述了成功实现了机器学习中广泛使用的线性代数基本操作之一，矩阵求逆的第一台设备。

这个团队早些时候还有关于**热力学人工智能**的作品，如*“热力学人工智能与波动前沿”* ([https://arxiv.org/abs/2302.06584](https://arxiv.org/abs/2302.06584))，有一篇[博客文章](https://normalcomputing.substack.com/p/thermodynamic-ai-intelligence-from)讨论了这个主题，以及*“热力学线性代数”* ([https://arxiv.org/abs/2308.05660](https://arxiv.org/abs/2308.05660))。

更早的作品，如*“热力学计算”* ([https://arxiv.org/abs/1911.01968](https://arxiv.org/abs/1911.01968))来自更广泛的作者集体，反映了关于该主题的一个研讨会的成果。

这种硬件的构建模块是随机的，最终使得软件与硬件无法分离（类似于[Hinton提出的Mortal Computers的概念](https://gonzoml.substack.com/p/mortal-computers)）。

不同于量子和模拟计算机，这里的噪声是计算所必需的资源。

大量机器学习算法基于各种物理原理，如能量模型或扩散模型。从权重初始化到神经网络构建模块（例如dropout），再到生成过程（如扩散模型、VAE和GANs），普遍使用随机性。对于这类算法，自然的随机波动可以成为重要的资源。

新的构建模块包括**随机位（s-bits）**，其状态随时间随机演变，如连续时间马尔可夫链（CTMC）。由于不是每个地方都需要位（神经网络权重或特征值更像实数），相应地，该构建模块可以表示连续变量。因此，热力学人工智能硬件的基本构建模块是**随机单元（s-unit）** - 一个正在进行布朗运动的连续变量。这可以在带有噪声电阻和电容的模拟电路上实现。

作者旨在统一现代 AI 算法，其中许多算法共享两个共同点：1）它们使用随机性，2）它们受物理启发。因此，建议基于热力学来统一这些算法。热力学算法的例子包括生成扩散模型、哈密顿蒙特卡洛和模拟退火。可以制定一个数学框架（在上述作品“*[热力学 AI 与波动前沿](https://arxiv.org/abs/2302.06584)*”中描述，并且总体而言，要了解更多详情，你应该去那里查看），在这些算法被视为特殊情况。

因此，同一热力学硬件可以加速所有这些算法。获利！

回到当前工作，它介绍了第一个**连续变量（CV）热力学计算机**。作者创建了一个适合电路板的**随机处理单元（SPU）**。

它包含 8 个单元，每个单元都是带有高斯噪声电流源的 LC 电路（在 FPGA 上实现）。所有单元都相互连接。为了定制，每个单元不是选择一个电容器，而是选择四个电容器中的一个。还可以调整噪声水平和采样频率（以 12 MHz 的基础频率读取单元电压）。

在生成的 SPU 上，可以从 8 维高斯分布中进行取样。

另一个有用的基元是矩阵反转。该过程的数学在另一部前述作品“*[热力学线性代数](https://arxiv.org/abs/2308.05660)*”中描述。为此，矩阵元素被转换为振荡器系统的连接，并且在系统达到热力学平衡后，取多次样本的电压值并计算协方差矩阵，这反过来与要找到的逆矩阵成比例。

最初，一个 4x4 矩阵在 SPU 上被反转，只需要一部分单元。取样越多，误差越低，但永远不会达到零（实验的不完美性）。

然后反转了一个 8x8 矩阵。这是在三个独立的 SPU 上完成的，显示它们产生了类似的结果，因此该过程是可重现的。

SPU 还实现了高斯过程回归。

通过谱归一化神经高斯过程进行不确定性量化，用于神经网络类别预测。SPU 的结果与在数字计算机上计算的结果非常一致。

作者预期，从大规模上看，SPU在与经典硬件相比会有优势。为了证明这一点，他们将SPU与RTX A6000 GPU在从多维高斯分布中抽样的任务上进行了比较。在大量维度下，SPU表现优于GPU上的经典方法。两条曲线的交点（所谓的“热力学优势”）在约3000个维度左右。总体而言，SPU的渐近性能估计为O(d²)，而在GPU上的Cholesky抽样为O(d³)。在能耗方面，SPU也更好。

* * *

简而言之，这是一个令人兴奋的方向。该领域正处于起步阶段，有潜力为这种新硬件的发现新的有趣算法做出贡献，就像为量子计算机的算法做出贡献一样。在这样的新硬件上训练神经网络仍然还有很长的路要走，但谁知道我们能多快地达到那里呢。
