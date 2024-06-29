<!--yml

category: 未分类

date: 2024-05-29 12:45:48

-->

# AutoBNN: 使用组合贝叶斯神经网络进行概率时间序列预测

> 来源：[https://blog.research.google/2024/03/autobnn-probabilistic-time-series.html?m=1](https://blog.research.google/2024/03/autobnn-probabilistic-time-series.html?m=1)

AutoBNN基于过去十年来的一系列研究，通过使用具有学习内核结构的高斯过程（GP），提高了时间序列建模的预测准确性。高斯过程的内核函数编码了关于所建模函数的假设，如趋势、周期性或噪声的存在。通过学习的GP内核，内核函数被定义为组合式的：它可以是基本内核（例如`Linear`、`Quadratic`、`Periodic`、[`Matérn`](https://en.wikipedia.org/wiki/Mat%C3%A9rn_covariance_function)或`ExponentiatedQuadratic`），也可以是组合内核，使用`Addition`、`Multiplication`或[`ChangePoint`](https://icml.cc/Conferences/2010/papers/170.pdf)等运算符结合两个或多个内核函数。这种组合内核结构有两个相关目的。首先，它足够简单，使得即使对自己的数据了解充分但不一定了解GP的用户也能构建合理的时间序列先验。其次，像[Sequential Monte Carlo](https://www.stats.ox.ac.uk/~doucet/doucet_defreitas_gordon_smcbookintro.pdf)这样的技术可以用于对小结构进行离散搜索，并能输出可解释的结果。

AutoBNN在这些理念的基础上进行了改进，用贝叶斯神经网络（BNNs）取代了GP，同时保留了组合内核结构。BNN是一个具有权重概率分布而不是固定权重集的神经网络，这导致了输出的分布，捕捉到预测中的不确定性。与GP相比，BNN带来以下优势：首先，训练大型GP计算成本昂贵，传统的训练算法在时间序列数据点数量的立方级别上扩展。相比之下，对于固定宽度，训练BNN往往是近似线性的。其次，BNN比GP训练操作更适合GPU和[TPU](https://cloud.google.com/tpu?hl=en)硬件加速。第三，组合BNN可以很容易地与传统的深度BNN结合起来，后者具有进行特征发现的能力。可以想象一种“混合”架构，用户指定了一个顶层结构，例如`Add`(`Linear`、`Periodic`、`Deep`)，而深度BNN则负责学习来自潜在高维协变量信息的贡献。

如何将具有组合核的高斯过程（GP）转化为贝叶斯神经网络（BNN）呢？单层神经网络通常会在神经元数量（或“宽度”）[趋于无穷大](https://link.springer.com/chapter/10.1007/978-1-4612-0745-0_2)时收敛到高斯过程。最近，研究人员[发现](https://openreview.net/forum?id=gRwh5HkdaTm)，也存在相反方向的对应关系——许多流行的GP[核](https://www.cs.toronto.edu/~duvenaud/cookbook/)（如`Matern`、`ExponentiatedQuadratic`、`Polynomial`或`Periodic`）可以通过适当选择的激活函数和权重分布获得无限宽度的BNN。此外，即使宽度远小于无穷大，这些BNN与对应的GP仍然非常接近。例如，下面的图表显示了观测值对之间的[协方差](https://en.wikipedia.org/wiki/Covariance_matrix#:~:text=In%20probability%20theory%20and%20statistics,of%20a%20given%20random%20vector)差异，以及真实GP和它们对应的宽度为10的神经网络版本的[回归](https://en.wikipedia.org/wiki/Kriging)结果。
