<!--yml

类别：未分类

日期：2024-05-27 12:51:21

-->

# 视觉Transformer被高估了 | 弗兰克的随笔

> 来源：[https://frankzliu.com/blog/vision-transformers-are-overrated](https://frankzliu.com/blog/vision-transformers-are-overrated)

[视觉Transformer](https://arxiv.org/abs/2010.11929)（ViTs）在过去四年中见证了令人难以置信的上升。它们显然有优势：在视觉识别设置中，一个纯粹的ViT的感受野实际上是整个图像的大小。特别是，基本的ViT保持了与语言模型相同的二次时间复杂度（关于输入补丁数量的）。

另一方面，卷积网络中的卷积核具有对应于其应用的输入像素/体素不变的特性，这通常称为*平移等变性*。这是可取的，因为它允许模型有效地识别空间中任何位置的模式和对象。卷积层中的权重共享也使得卷积神经网络具有高参数效率，并且不太容易过拟合 - 这是ViTs所不具备的特性。

你可能会期望在利用视觉模型的生产环境中，ViTs 和卷积神经网络（convnets）的使用是平等的 - ViTs 用于“全局”任务，如场景识别，而卷积神经网络用于更“局部”的任务，如物体识别。即便如此，我们在使用ViTs的工作中不断涌现，带有大胆的高层次声明（主要是由媒体发布的），声称[卷积神经网络已经成为过去](https://syncedreview.com/2023/07/17/deepmind-proposes-novel-vision-transformer-for-arbitrary-size-resolution/)。

我好奇是否可以帮助揭穿这种说法，因此我着手研究一个基本的ResNet是否能够达到甚至超过ViT和ConvNeXt的性能。特别是与ConvNeXt的比较尤为重要，因为它是一种完全卷积网络，试图弥合变压器和卷积神经网络之间的差距。

在Imagenet-1k上进行一些实验后，我们可以使用176x176的训练图像尺寸而无需额外数据，达到**82.0%**的准确率，与ConvNeXt-T（v1，不使用MAE预训练）的表现相当，并超过了ViT-S（具体来说，是DeiT-III中的ViT变体）。

#### 训练方法

我们从采用Pytorch 2021年末博客中设置的训练方法开始，他们在Imagenet-1k上使用了一个标准的ResNet50模型，达到了[**80.8%**的准确率](https://pytorch.org/blog/how-to-train-state-of-the-art-models-using-torchvision-latest-primitives/)。以下是一些关键要点：

+   我们坚持使用SGD作为优化器，而不是选择RMSProp或Adam（或它们的任何变体）。

+   调度器使用余弦衰减，包括五个预热epochs和600个总epochs。这可能看起来epochs数量过多，但我们会在之后进行减少。

+   我们利用现代文献中找到的大量数据增强方法，包括但不限于：标签平滑、mixup、cutmix 和模型 EMA。

+   为了防止在验证数据集上过拟合，我们将跳过超参数调整和网格搜索，并坚持在博客文章中列出的原始训练方法。

几乎所有这些训练优化措施已经被用来提升现代视觉识别模型的性能，但采用这些变化并没有使我们达到期望的神奇的 82% 准确率。

#### 架构修改

基准的 ResNet 结构强大但不是最优的，因此我们采用了几种架构修改来提升性能：

**ResNet-d**

首要任务是改进 ResNet 的一些“现代化”。为了全面，这里列出了所做的更改：

1.  初始的 7x7 卷积改为一系列连续的三个 3x3 卷积，分别具有 32、64 和 128 个输出通道。步幅仍然在第一个卷积层上。通过这一变化，我们现在在整个网络中完全使用 3x3 和 1x1 卷积，同时保持网络头部的原始感受野大小。

1.  在下采样残差块中，步幅从第一个 1x1 卷积层移至随后的 3x3 卷积层。这样做的效果是捕捉下采样块中的所有输入像素，因为步幅为 1x1 的卷积实际上会跳过每隔一个像素。

1.  在初始部分的最大池化已被移除。第一个残差块的第一个 3x3 卷积现在步幅为二，与其余的残差块保持一致。虽然最大池化在理论上对保留边缘、角点和其他低级特征有用，但在实践中我并未发现它特别有用。

1.  在下采样块的快捷连接中，步幅为 1x1 的卷积被替换为 2x2 平均池化，随后是标准的 1x1 卷积层。同样，这种做法的效果是捕捉所有输入激活而不仅仅是每四个输入通道中的一个。

这些微观优化的结果导致的架构与 [ResNet-d](https://arxiv.org/abs/1812.01187v2) 非常接近，仅有一些非常小的差异。

**ReLU -> SiLU**

与其他激活函数相比，ReLU有两个弱点：1）它不平滑（ReLU严格来说在0处不可微分），2）“死亡ReLU”问题，在正向传递期间，预激活值几乎普遍为负，导致梯度始终为零且神经元不传递信息。因此，多年来提出了许多新型激活函数 - 漏值ReLU、参数ReLU、ELU和Softplus是三个著名但较老的例子。所有这些方法的背后都是为了解决上述一个或两个问题；例如，参数化ReLU通过引入可学习参数 $\alpha$ 来定义负预激活值的斜率，试图解决死亡ReLU问题。对于这个模型，我选择了SiLU（也被称为[Swish](https://arxiv.org/abs/1710.05941v2)），其定义为 $SiLU(x) = \frac{x}{1+e^{-x}}$，在许多视觉识别模型中已经取得了成功。由于这种转换带来了更快的训练速度，我将epoch数从600降低到了450。

虽然我本可以使用[GELU](https://arxiv.org/pdf/1606.08415.pdf)，但我决定使用SiLU，因为它有一个`inplace`参数，并且可以作为原始参考实现中ReLU的可替代品。像GELU或GLU变体（如SwiGLU、GeGLU）可能会表现得更好，因为它们在语言模型中被广泛使用。尽管GELU和SiLU高度相关 ^(，使用GELU训练的网络*并不*等同于使用SiLU训练的网络，因为它们在权重衰减和初始化方面存在差异。)

最后，我假设使用SiLU网络可能会在[随机深度](https://arxiv.org/abs/1603.09382)的情况下表现更好，因为ReLU可能像一个弱的隐式正则化器，通过增加网络激活的稀疏性。这对于超参数化模型可能是有益的，但对于参数高效模型则不然。另一方面，SiLU在所有值 $x$（除了 $x \approx -1.278$）处都有非零梯度。因此，从ReLU转换为SiLU后，可能需要加入一些正则化。我将在未来几周内继续进行更多实验。

*更新（03/23/2024）*：经过一些实验，我发现使用概率为`0.1`的随机深度会对网络性能产生负面影响（大约`0.2%`左右），但将其减少到`0.05`则会导致几乎相同的准确性。我需要再多尝试一下。

**分割归一化**

香草ResNet使用大量的批量归一化（BN）；每个卷积层精确地使用一个BN层。原始的BN论文认为BN改善了*内部协变量转移*（ICS）- 作者定义为任何中间层在上游网络权重变化时看到的变化 - 但这后来被证明是不正确的（稍后我会详细说明）。我想回到最初的ICS论文中，即BN中的归一化旨在重新集中激活，而紧随其后的可学习仿射变换旨在保持每个层的表示能力。对我来说，这两者必须依次应用根本没有道理。此外，由于反向传播有效地将每个单独的神经元层视为独立的学习者，最明智的做法是归一化层*输入*而不是*输出*。

长话短说，我发现将BN分割为两个独立的层 - 卷积前归一化和卷积后仿射变换 - 可以将网络的性能提高超过0.4%。尽管这在训练过程中会对速度和内存消耗产生负面影响，但对推断性能几乎没有影响，因为归一化和仿射变换可以在网络完全训练后表示为对角矩阵，并与卷积层的权重融合。

[分割归一化，可视化。]

我想更好地理解“分割”归一化背后的理论，但在机器学习文献中找不到它^(。因此，我首先转向了BN理论；我眼中最具说服力的研究来自[Santurkar等人2018年的论文](https://arxiv.org/abs/1805.11604)。在这篇文章中，他们表明BN通常*增加*ICS。相反，他们认为批量归一化之所以有效，是因为它改善了损失景观的一阶和二阶性质。）

通过一个快速的练习，我们可以证明分割归一化（SN）具有相同的效果。让我们考虑两个网络 - 一个没有由损失函数$L$定义的SN，另一个有由损失函数$\hat{L}$定义的SN。对于有SN的网络，每个层的梯度如下：

\[\begin{aligned} \frac{\partial\hat{L}}{\partial y_i} &= \frac{\partial\hat{L}}{\partial \hat{y}_i}\gamma \\ \frac{\partial\hat{L}}{\partial\hat{x}_i} &= \mathbf{W}^T\frac{\partial\hat{L}}{\partial\hat{y}_i} \\ \frac{\partial\hat{L}}{\partial x_i} &= \frac{1}{\sigma}\frac{\partial\hat{L}}{\partial\hat{x}_i} - \frac{1}{m\sigma}\sum_{k=1}^{m}\frac{\partial\hat{L}}{\partial\hat{x}_k} - \frac{1}{m\sigma}\hat{x}_i\sum_{k=1}^{m}\frac{\partial\hat{L}}{\partial\hat{x}_k}\hat{x}_k \end{aligned}\]

其中 $m$ 是每个小批量的大小，$y_i$, $\hat{y}_i$, $\hat{x}_i$, $x_i$ 表示我们批次中第 $i$ 个样本的激活。在实践中，激活张量的维度可以是任意大或小（例如，大多数卷积网络为3维）。有了这些，我们可以通过点积表示完整的损失梯度：

\[\begin{aligned} \frac{\partial\hat{L}}{\partial\mathbf{x}} &= \frac{\gamma}{m\sigma}\mathbf{W}^T\left(m\frac{\partial\hat{L}}{\partial\mathbf{y}} - \mathbf{1} \cdot \frac{\partial\hat{L}}{\partial\mathbf{y}} - \mathbf{x} \left(\frac{\partial\hat{L}}{\partial\mathbf{y}} \cdot \mathbf{x}\right)\right) \end{aligned}\]

对于函数 $f(a)$，其梯度的 L2 范数 $\left\Vert\frac{df}{da}\right\Vert$ 是 Lipschitz 程度的良好代理。同样地，我们的损失函数也满足这一点，即我们希望展示 $\left\Vert\frac{\partial\hat{L}}{\partial\mathbf{x}}\right\Vert \leq \left\Vert\frac{\partial L}{\partial\mathbf{x}}\right\Vert$。给定矩阵 $\mathbf{A}$ 和向量 $\mathbf{b}$，它们的乘积的范数由 $\mathbf{A}$ 的最大奇异值上界给出，即 $\Vert\mathbf{A}\cdot\mathbf{b}\Vert \leq s_{max}(\mathbf{A})\Vert\mathbf{b}\Vert = \sqrt{\lambda_{max}(\mathbf{W}^T\mathbf{W})}\Vert\mathbf{b}\Vert$。有了这一点，我们有：

\[\begin{aligned} \left\Vert\frac{\partial\hat{L}}{\partial\mathbf{x}}\right\Vert^2 &\leq \left(\frac{\gamma}{m\sigma} \right)^2 s_{max}(\mathbf{W})^2\left\Vert m\frac{\partial\hat{L}}{\partial\mathbf{y}} - \mathbf{1} \cdot \frac{\partial\hat{L}}{\partial\mathbf{y}} - \mathbf{x} \left(\frac{\partial\hat{L}}{\partial\mathbf{y}} \cdot \mathbf{x}\right)\right\Vert^2 \end{aligned}\]

应用 Santurkar 等人在 C.2 中的简化，我们得到：

\[\begin{aligned} \left\Vert\frac{\partial\hat{L}}{\partial\mathbf{x}}\right\Vert^2 &\leq \frac{\gamma^2s_{max}^2}{\sigma^2} \left( \left\Vert \frac{\partial L}{\partial\mathbf{y}}\right\Vert^2 - \frac{1}{m}\left\Vert\mathbf{1} \cdot \frac{\partial L}{\partial\mathbf{y}}\right\Vert^2 - \frac{1}{m}\left\Vert\frac{\partial L}{\partial\mathbf{y}} \cdot \mathbf{x}\right\Vert^2 \right) \end{aligned}\]

在我看来，我们应该将乘法项（即$\frac{\gamma^2s_{max}^2}{\sigma^2}$）与加法项（即$- \frac{1}{m}\left\Vert\mathbf{1} \cdot \frac{\partial L}{\partial\mathbf{y}}\right\Vert^2 - \frac{1}{m}\left\Vert\frac{\partial L}{\partial\mathbf{y}} \cdot \mathbf{x}\right\Vert^2$）分开，因为a）乘法效应可以通过增加或减少学习率来抵消，b）$\mathbf{W}$相对于等式中的其他项变化较慢。特别是，加法项严格为负，这意味着整体损失平面更加平滑，而潜在的大乘法上限意味着在某些情况下SN可能会*增加损失的Lipschitz常数*。同时，每一层的输入ICS严格减少，因为可学习的仿射变换现在位于权重之后而不是之前。

#### 结果如下

最终的2600万参数模型在Imagenet-1k上成功达到了82.0%的准确率，没有任何外部数据的辅助！顺应现代机器学习研究的精神，让我们给这个网络取个花哨的名字：GResNet（好/伟大/帮派/神级ResNet）。

| 模型 | 准确率 | 参数 | 吞吐量 |
| --- | --- | --- | --- |
| GResNet | 82.0%* | 25.7M | 2057 im/s |
| ConvNeXt | 82.1% | 28.6M | 853 im/s |
| ViT (DeiT) | 81.4% | 22.0M | 1723 im/s |

[不同模型的比较。在未经网络优化的单个Nvidia A100上，批量大小为256，*当我们使用ConvNeXt的训练图像大小为224x224而不是176x176时，准确率提高到82.2%，吞吐量下降到1250 im/s。]

GResNet模型定义可以在[这里](https://gist.github.com/fzliu/006d2043dc1e90d68ae562c5bde8066c)找到，而权重可以在[这里](https://drive.google.com/file/d/1WKPkLybhrE389sierVj2-uZ_ZYAJI1Gf/view?usp=drive_link)找到。

[训练过程中的准确率曲线。]

#### 结语

这里到底展示了什么？通过对ResNet进行一些简单修改，我们可以在小型数据集上达到出色的性能——与ViT和ViT启发的卷积网络（ConvNeXt）相媲美甚至更好。

[ResNet再次发力...]

你可能会问：为什么选择Imagenet-1k？难道没有更大的标记视觉数据集，比如YFCC、LAION等吗？其次，由于现代LLM完全基于变压器，使用变压器来进行视觉处理是否有利于利用交叉注意力或通过线性投影将补丁传送到解码器？答案是肯定的：对于受文本约束的大型多模型，自注意力至关重要。但是小型模型（例如大多数嵌入模型）可能更为重要，因为它们具有可移植性和适应性，这些模型极大地受益于本文中概述的类型实验：通过强大的数据增强在有限数据上进行跨多个时期的训练。这正是Imagenet-1k所代表的数据类型。

至于ViTs在大型数据集上优于卷积网络的话题：来自Google DeepMind的2023年论文[*Convnets match vision transformers at scale*](https://arxiv.org/pdf/2310.16764.pdf)值得一读。结论部分包含一个鲜明的结果：“虽然ViTs在计算机视觉中的成功非常令人印象深刻，但在我们看来，并没有强有力的证据表明预训练的ViTs在公平评估时优于预训练的卷积网络。”这简单地强化了一个应该重复的教训：模型架构的优化应始终放在以下几点之后：1）大型、高质量的数据集，2）稳固的、高度可并行化的训练策略，以及3）大量的H100。我认为transformer的成功主要来自它们能够*高效且有效地*扩展到数百亿参数的能力 - 这种扩展理论上也可以用RNNs完成，如果研究科学家们有数十年时间来训练它们的话（剧透：他们没有）。

#### 附录 - 比较嵌入向量质量

我觉得比较一下从GResNet、ConvNeXt和ViT提取的嵌入向量可能会很有趣，通过在[Milvus](https://github.com/milvus-io/milvus)中存储和索引每个模型的嵌入向量进行。

```
>>> from milvus import default_server
>>> from pymilvus import MilvusClient
>>> default_server.start()
>>> client = MilvusClient(uri="http://127.0.0.1:19530")
>>> # initialize model, transform, and in1k val paths
...
>>> with torch.no_grad():
...     for n, path in enumerate(paths):
...         img = Image.open(path).convert("RGB")
...         feat = gresnet(transform(img).unsqueeze(0))
...         client.insert(collection_name="gresnet", data=[feat])
...
>>> 
```

我删除了模型初始化和数据加载的片段以简洁起见，并使用欧几里得/L2作为距离度量，没有进行索引（即FLAT）。完成这一步后，我们可以查询集合以获取类似以下结果的结果：

有人可能会认为，GResNet倾向于挑选风格与查询图像更接近的图像，而不仅仅是相同类别的图像，但除此之外，这三个模型的结果非常相似。

* * *

* * *
