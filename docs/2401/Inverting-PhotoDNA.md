<!--yml

category: 未分类

date: 2024-05-27 14:32:35

-->

# 反转 PhotoDNA

> 来源：[`anishathalye.com/inverting-photodna/`](https://anishathalye.com/inverting-photodna/)

微软的[PhotoDNA](https://www.microsoft.com/en-us/photodna)为图像创建了一个“独特的数字签名”，可以与包含先前识别的非法图像（如 CSAM）签名的数据库进行匹配。该技术被包括谷歌、Facebook 和 Twitter 在内的公司[使用](https://www.makeuseof.com/what-is-photodna-how-does-it-work/)。微软表示：

> PhotoDNA 哈希是不可逆的，因此无法用于重新创建图像。

[Ribosome](https://github.com/anishathalye/ribosome)使用机器学习反转 PhotoDNA 哈希。

这个演示使用挑衅性的图像来说明一个观点：可以从 PhotoDNA 哈希中恢复粗糙的身体形状和面部特征。顶部行中的图像来自《体育画报》，底部行中的图像是[虚构的人物](https://thispersondoesnotexist.com/)。第一列显示了 144 字节的 PhotoDNA 哈希的一部分，中间列显示了使用 Ribosome 可以从此哈希中重建的图像。

像任何其他有损函数一样，PhotoDNA 哈希并非完全可逆，但哈希泄漏了大量关于原始输入的信息，正如这些图像重建所证明的那样。

PhotoDNA 算法的详细信息和实现都没有正式向公众公开，但该算法已经根据公开文件进行了[逆向工程](https://hackerfactor.com/blog/index.php?/archives/931-PhotoDNA-and-Limitations.html)，并且在发现苹果的 NeuralHash 算法存在[碰撞](https://github.com/anishathalye/neural-hash-collider)时，一种用于计算 PhotoDNA 哈希的[编译库](https://github.com/jankais3r/pyPhotoDNA)在此前已经泄露。

由于 PhotoDNA 的闭源性质，对哈希函数的研究工作并不多。2019 年，[Nadeem 等人](https://kar.kent.ac.uk/77165/)与微软合作，通过对其进行 ML 分类测试，调查了 PhotoDNA 的隐私保护能力。该论文声称“PhotoDNA 对基于机器学习的分类攻击具有抵抗力”。最近，在 2021 年 11 月，[Prokos 等人](https://eprint.iacr.org/2021/1531)对 PhotoDNA 执行了有针对性的第二预影像攻击。

Ribosome 是对 PhotoDNA 哈希函数的反向攻击，并调查了 PhotoDNA 哈希“无法用于重新创建图像”的说法。

Ribosome 将 PhotoDNA 视为黑盒，并使用机器学习攻击哈希函数。由于 PhotoDNA 的[实现被泄露](https://github.com/jankais3r/pyPhotoDNA)，因此可以生成图像/哈希对的数据集。Ribosome 在这样的数据集上训练神经网络，以学习如何根据其哈希合成图像。

PhotoDNA 哈希是 144 个字节向量。Ribosome 使用类似于[DCGAN](https://arxiv.org/pdf/1511.06434/)生成器和[快速风格转换](https://cs.stanford.edu/people/jcjohns/papers/fast-style/fast-style-supp.pdf)网络的神经网络，使用残差块后跟分数步长的卷积进行学习，以将其转换为 100x100 的图像。

选择用于训练模型的数据集会影响结果。理想情况下，数据集中的图像应该来自于与要反转哈希的图像相同的分布。例如，当反转预期包含人物的图像的哈希时，训练模型在[Places](http://places.csail.mit.edu/)数据集上可能不会产生最佳结果。

即使不知道图像所绘制的确切分布，Ribosome 也可以产生良好的结果。下图说明了由在不同测试集上进行训练的模型计算得出的哈希反转：

在上图中，来自每个数据集的保留测试图像以及来自互联网的额外图像经过哈希处理，然后使用在不同数据集上进行训练的 Ribosome 模型进行复制。训练数据集包括[CelebA](https://mmlab.ie.cuhk.edu.hk/projects/CelebA.html)、[COCO](https://cocodataset.org/)和从 SFW 和 NSFW 子版块抓取的 10 万张图像数据集。第四个数据集是通过将三个数据集的图像组合而成的（从 CelebA 中仅取 4 万张图像）。

图中展示了一些有趣的现象：

+   使用大型且多样化的数据集允许模型从 PhotoDNA 哈希中生成良好的复制品（第四列）。

+   先验条件越好，结果就越好，例如利用 CelebA 训练的模型重现面部（第一列，第二行）。

+   Ribosome 可以处理一些分布变化，例如在 COCO 上进行训练时重现面部（第二列，第二行）。

+   毫不奇怪，将模型训练在一个范围较窄的数据集上会导致其对于生成类似该数据集中的图像具有较强的偏好，例如在 CelebA 上进行训练会导致模型生成包含面部的图像（第一列）。

+   模型学会重新着色图像（PhotoDNA 在处理之前将图像转换为灰度）。

+   模型有时会有有趣的失败行为，例如 Reddit 数据集未经精心筛选，其中有许多“图片不可用”的图像，这些现象在一些结果中可见（第三列）。

上面的图像是手动选择的示例，根据结果的质量、图像的多样性（例如不同的姿势）和符合工作场所的性质进行选择。重建并不总是完美的。为了让你了解模型的平均工作方式，这里展示了 5 个随机选择的 COCO 测试数据点及其原始图像：

Ribosome 表明 PhotoDNA 并未完全隐藏有关用于计算签名的源图像的信息，事实上，可以使用 PhotoDNA 哈希来生成原始图像的缩略图质量的复制品。

代码和预训练模型可在 [GitHub](https://github.com/anishathalye/ribosome) 上找到。
