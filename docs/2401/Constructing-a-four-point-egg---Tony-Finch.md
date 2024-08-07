<!--yml

类别：未分类

日期：2024 年 05 月 27 日 15 时 23 分 18 秒

-->

# 构造一个四点蛋 – Tony Finch

> 来源：[`dotat.at/@/2024-01-29-four-point-egg.html`](https://dotat.at/@/2024-01-29-four-point-egg.html)

出于本条目的范围之外的原因，我一直在研究椭圆和卵形的形状。 [Moss’s egg 的 Wikipedia 文章](https://en.wikipedia.org/wiki/Moss%27s_egg)有一个链接到[Freyja Hreinsdóttir 的欧几里德蛋教程](https://web.archive.org/web/20200618202007/https://www.dynamat.oriw.eu/upload_pdf/20121022_154322__0.pdf)，其中（除其他外）描述了如何构造“四点蛋”。 我认为它比 Moss 的蛋更漂亮。

Freyja 的构造使用了古典风格的直尺和圆规，因此它缺乏尺寸。 下面是我的版本，带有在计算机上绘制所需的数字。

起初，这个构造似乎相当刚性，但其中一些选择比它们看起来更任意。 我做了[一个交互式的四点蛋](https://dotat.at/@/2024-01-eggsperiment.html)，所以你可以拖动点并观察其形状如何变化。

在接下来的过程中，我将以一圈的分数来测量角度`𝜏`，顺时针从正午开始，因为蛋的尖端朝上。

1.  从坐标轴和单位圆开始。

1.  想象两条对角支架线：一条通过单位圆的南点和东点；另一条通过其南点和西点。

1.  在单位圆的南点为中心，并与其北点等距离，从一条支架线到另一条支架线画出一个弧。 这形成了蛋的大端。

```
x: 0 y: -1 radius: 2 from: 𝜏*3/8 to: 𝜏*5/8

```

1.  从单位圆的西点为中心，画出蛋的右下弧，从底部弧的右端到正 X 轴。

```
x: -1 y: 0 radius: 2+√2 from: +𝜏*2/8 to: +𝜏*3/8

```

1.  画出与右下弧相映射的蛋的左下弧。

```
x: +1 y: 0 radius: 2+√2 from: -𝜏*3/8 to: -𝜏*2/8

```

1.  右下和左下的弧在 X 轴上遇到蛋的东点和西点。 想象一个以原点为中心的支架圆连接这些点。

1.  蛋的北部中心是这个支架圆与 Y 轴相交处，在`+1+√2`处。

    想象两条新的对角支架线：一条通过蛋的西点和北部中心；另一条通过其东点和北部中心。

1.  从蛋的西点为中心，画出蛋的右上弧，从蛋的东点到一个支架线。

```
x: -1-√2 y: 0 radius: 2+2√2 from: +𝜏*1/8 to: +𝜏*2/8

```

1.  画出与右上弧相映射的蛋的左上弧。

```
x: +1+√2 y: 0 radius: 2+2√2 from: -𝜏*2/8 to: -𝜏*1/8

```

1.  在蛋的北部中心画出一个弧，连接右上弧和左上弧的末端。 这个弧形成了蛋的小端。

```
x: 0 y: 1+√2 radius: √2 from: -𝜏*1/8 to: +𝜏*1/8

```

在上图中标记的六个彩色点是构成蛋的六个弧的中心。 这包括蛋以及它们的镜像被命名的四个点。

Moss 的蛋是一个三点蛋。 它的顶半部分与这个四点蛋相同，但底半部分是一个简单的半圆。
