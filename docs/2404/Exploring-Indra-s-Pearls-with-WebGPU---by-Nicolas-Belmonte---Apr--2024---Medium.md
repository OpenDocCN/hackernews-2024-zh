<!--yml

类别：未分类

日期：2024-05-27 13:23:42

-->

# 使用WebGPU探索Indra's Pearls | Nicolas Belmonte | 2024年4月 | Medium

> 来源：[https://medium.com/@philogb/exploring-indras-pearls-with-webgpu-e0f4a745c2f6](https://medium.com/@philogb/exploring-indras-pearls-with-webgpu-e0f4a745c2f6)

## 推广极限集：超越圆

我们可以通过直接从Mobius生成器计算极限集，而不是仔细选择圆和将一个圆外部映射到另一个圆内部的Mobius变换，来推广工作。这样可以得到不一定由圆构建的极限集。

调整参数以获得不同的极限集

能够给出这些“尖刺”极限集的Mobius变换参数化被称为Maskit参数化。Maskit切片是使群体变为非离散（我们渲染无意义的区域）的区域下方。

µ 值（x=Re(µ)，y=Im(µ)）和Maskit切片。

如果我们停留在确切的边界上，我们将得到处于非离散极限的群体。可视化允许探索这些尖刺群体。该书详细介绍了这些群体的特性。

穿越Maskit切片的边界

# 实施

我用这个项目来深入研究WebGPU和WGSL用于着色编程。

最初的可视化是通过 `KleinGroup` 类创建的。该类将每个迭代渲染到一个纹理中，然后将该纹理反馈到类中进行下一次迭代。从编程角度看，这很好地模拟了Mobius变换的操作。最终的纹理被馈送到 `Quad` 类，后者将四边形渲染到屏幕上。

我们随后包括一个 `RSphere` 类，它将渲染黎曼球。`KleinGroup` 向 `Quad` 和 `RSphere` 同时提供纹理。然后，`quad` 在球体上方移动，描述共轭作用的方式。

动画展示了RSphere和Quad类的运行过程。

## 极限集

为了可视化极限集，我们实现了Jos Ley的：[*Kleinian群的极限集快速算法（Maskit参数化）*](https://www.josleys.com/articles/Kleinian%20escape-time_3.pdf)。该算法通过片段着色器直接渲染极限集。片段着色器的输入是Mobius变换和一种能描述集合每一边界的纹理。对于屏幕上的每个点，如果它落在曲线的上方或下方的区域内，我们应用Mobius变换 `a` 或其逆 `A`。

极限集与底层纹理。

对于远离Maskit边界的集合，可以将线实现为平滑曲线。

## 尖点

对于处于Maskit切片边界上的极限集，这更加复杂。我们需要找到Mobius乘积的组合，使得群是抛物线的（我们称之为抛物线“词”）。对于每两个Mobius变换及其逆变换，`a`、`A`、`b`、`B`，使其成为抛物线的词可能是这样的：`aaaaBaaaaB`。找到这个矩阵乘积的不动点给出了一个点，它与集合中的两个圆相切。

然后，我们需要循环遍历这些词以获得与集合中每个圆相切的所有其他不动点。我实现了获取词及其循环排列的方法可以在[此笔记本](https://observablehq.com/d/cde177e22e1a9a6e#getWord)找到。

为了Maskit切片边界处的尖角群计算纹理。

上述纹理使用绿色和蓝色表示算法，在评估和应用Mobius变换之前进行水平移位（这让我头疼不已）。

## 计算Maskit切片

最后，为了追踪Maskit边界，我们需要为抛物生成器（其中`Trace=2`）实现一个牛顿求解器。我们通过获取分数的Farey序列来沿着x轴行走，对于每个分数，我们生成一个Trace多边形，并使用牛顿法求解它，这将找到我们需要的µ值。然后，给定Farey序列的分子`p`、分母`q`和复数值µ，将尖角极限集提供给它。
