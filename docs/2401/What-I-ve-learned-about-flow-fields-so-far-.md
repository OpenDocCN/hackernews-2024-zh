<!--yml

分类：未分类

日期：2024 年 05 月 27 日 14:26:56

-->

# 到目前为止，我对流场了解到的内容。

> 来源：[`damoonrashidi.me/articles/flow-field-methods`](https://damoonrashidi.me/articles/flow-field-methods)

# 到目前为止，我对流场了解到的内容。

2021 年 6 月 15 日更新于 2023 年 12 月 27 日。

> 如果你觉得这篇文章太啰嗦，可以跳过所有文字，玩玩插图，随着文章的发散，它们会变得越来越有趣！
> 
> -- 我

## 简要介绍。

本文将描述我用来创建下面所示系列生成艺术品的方法和概念。我尝试可视化算法并提供一些代码示例。代码示例是用 Typescript 编写的，并做了一些简化，以使代码在移动设备上可读，并使其更简洁易懂。尽管如此，这些通用算法可以轻松移植到任何语言中，例如，我已经使用 Rust 做了一些[实现](https://github.com/damoonrashidi/generative-art)。

顺便提一句，比我更有才华的人已经写了[关于这个主题的文章](https://tylerxhobbs.com/essays/2020/flow-fields)，显然我的工作是基于那篇文章的。我也推荐阅读那篇！

## 噪声函数

这些流场图像背后的主要驱动力通常是一个噪声函数。不重复引述[维基百科文章](https://en.wikipedia.org/wiki/Perlin_noise)，噪声函数是一个接受二维空间坐标（高维度噪声函数也存在，但对本文的目的无关）并返回范围在`-1..=1`之间的值的函数，例如，靠近的点返回相似但略有不同的值。

下面的交互式插图展示了这是如何工作的，通过在网格中均匀采样点并调用该点的噪声函数。请注意，为了便于阅读，这些值已经四舍五入到小数点后一位，实际值具有更高的精度。点击“重新生成”以使用新的种子再次运行噪声函数以获取新的噪声值。

这是生成上述图片的代码，为了清晰起见做了简化。

```
[object Object]
```

为了让它更容易理解，我们可以将这些噪声值转化为角度并以更有效的方式进行可视化，从起始位置开始，并根据噪声值的角度绘制几个像素的线条。

```
[object Object]
```

目前看起来不怎么样。似乎线条相当零散，并朝着看似随机的方向延伸。一点也不是我们要追求的平滑效果。

结果表明，噪声函数非常敏感。你可能会认为点`(1.0, 1.0)`和`(1.0, 2.0)`会产生相似的噪声值，因为附近的点通常会产生相似的噪声值，但它们彼此之间并不够接近。

为了对抗这一点，我们可以通过仅将我们的 x 和 y 坐标除以一些平滑度常数来使我们的点彼此更接近。例如，我们两个示例点之间的距离先前为 y 轴上的 `1.0`，x 轴上的 `0.0`，总距离为 `1.0`。

如果我们将所有的 x 和 y 值都除以常数 `smoothness = 100`，我们得到的结果是：`p1 = (0.01, 0.01), p2 = (0.01, 0.02)`，使距离仅为 `0.01`，但保持点之间的关系不变。

我们代码中唯一需要做出的改变是以下内容：

```
[object Object]
```

您可以将其视为缩小我们的域以更好地适应噪声值。或者如果有助于理解，可以将其视为对噪声函数进行放大。

正如我们通过提高平滑度值所看到的，靠近 100，在这种情况下，线条开始变得平滑，并且开始出现模式。

## 绘制线条。

现在我们知道如何浏览流场。选择任意点 `P`，读取该点的噪声值 `n`，并将 `P.x` 增加 `cos(n)`，`P.y` 增加 `sin(n)`，以及表示我们想要在场的方向上行进的距离的一些额外像素。

在前面的示例中，我们通过在网格中采样点并设置固定的线长度来完成此操作。为了更好地近似本文开头图片中所示的效果，需要另一种实现方式。首先，我们需要在画布上选择一个随机点，然后沿着流场骑行直到超出边界。

```
[object Object]
```

从上述示例转换到绘制实际线条现在应该很简单。从画布上的任意给定点开始并实例化您的线条，沿着场域进行移动，就像以前一样，但不是在给定位置绘制一个新点，而是将其添加到线条的点中。当线条到达画布的末端时停止循环，最后绘制线条。

```
[object Object]
```

将鼠标悬停在下图上，可以根据鼠标位置创建新的行。

## 试验线条。

到目前为止，并没有实现很大的变化。无论噪声函数的种子是什么，所有结果都会产生相似的图片。有几种方法可以解决这个问题，一种方法是稍微扭曲噪声值，导致线条夸大其转向，这只需在应用 `cos()` 和 `sin()` 函数之前简单地将噪声值乘以一些常数即可实现。

```
[object Object]
```

另一种方法是在线条在场中前进时调整每步线条的步长长度。较短的步长将产生更加平滑的曲线，而较长的步长将使线条更加锯齿状。

```
[object Object]
```

## 替代噪声函数

到目前为止，我们一直在使用一个叫做 OpenSimplex 的噪声函数。

> OpenSimplex 噪声是一个 n 维梯度噪声函数，旨在克服 Simplex 噪声相关的专利问题，同时也避免了 Perlin 噪声的视觉上显著的方向性伪影。
> 
> - Kurt Spencer

如引文所述，有几种不同的噪声函数。一个适当的噪声函数有点复杂，超出了本文的范围，但没有什么可以阻止我们编写一个函数，为给定的坐标返回类似的值，这部分可以很容易地完成。

这些自制函数产生的结果看起来不像噪声函数那样随机，但它们让我们能够更好地控制最终的输出。它们还能让我们在尝试新事物时更有创造性，因为现在我们可以绘制可靠地遵循路径的线条。

我们可以做的一个简单的第一次测试就是简单地取一个坐标并返回它到另一点（我们称之为`focalPoint`）的距离，只是为了看看会发生什么。这个函数将满足点彼此靠近时产生类似值的规则，使我们的线条看起来漂亮而平滑。

```
[object Object]
```

将鼠标悬停在下面的插图上以设置一个新的焦点

如果我们不是返回到`focalPoint`的距离，而是返回线条从我们的点到的角度，并沿半径偏移我们的点（对 y 轴略微扭曲），我们可以得到一个漂亮的涡旋效果。

```
[object Object]
```

```
[object Object]
```

## 碰撞检测

在我看来，真正有趣的事情实际上并没有在我们开始让线条相互作用之前开始。与其在到达画布的尽头或达到最大线条长度时停止，我们可以在该线条与另一条线条碰撞时停止。

将鼠标悬停在插图上（或者在其上滑动手指）可以添加不同宽度的线条。

现在，关于碰撞检测以及如何使其性能良好有很多可以说的话。我只会展示一种方法和一个小的优化来保持解决方案的性能。

检查[两条直线的交点](https://en.wikipedia.org/wiki/Line%E2%80%93line_intersection)并不太困难，可以在 O(1)时间内完成。然而，我们的线条并不是直的，所以我们需要聪明一点。如果我们回到绘制线条的开始，它是通过遍历噪声场并为每一步添加一个点开始的。如果我们将该点变成一个圆，给它与线条相同的半径，并跟踪每个圆，不仅在线条中，而且在所有线条中，那么检查我们要绘制的圆是否与任何其他圆重叠就容易得多了。

检查两个圆是否重叠只是检查它们的半径之和是否小于它们之间原点的距离。

```
[object Object]
```

这里是另一个插图，使用这种方法突出显示了当两条非线性线条相遇时的情况。与插图互动以移动相撞的线条。

<canvas class="h-[500px] w-full touch-none"></canvas>

```
[object Object]
```

### 稍微优化一下。

这些示例图案相当小，因此当检查我们的线条是否与任何其他线条相撞时，我们尚未遇到任何性能问题……至少目前还没有。然而，当尝试制作更大的图像时，例如适合打印的尺寸，我们将得到许多具有许多点的线条，这些线条可能与许多我们可能相撞的点相撞，这意味着每添加一个新点时，我们必须检查与所有其他点的碰撞。这样很快就会堆积起来，并使您的渲染时间比预期的要长得多。缓解这种情况的方法是仅针对足够接近我们可能碰撞的点进行检查。

这样做的最简单方法是将我们的画布分成一个方格网格，每当我们要添加一个新点时，检查它应该放在哪个方格中，然后只检查该方格中的点是否与其他点相撞。

是时候再举个例子了。这次，移动你的手指或鼠标光标，看看哪些点属于同一个方格。

现在，有了 100 个方格（横向 10 个，纵向 10 个），如果点均匀分布在画布上，我们最终只需进行之前的 1/100^(th) 检查数量的检查，将渲染性能提高了很多！但需要注意的是，如果我们的方格太小而无法可靠地容纳线条的半径，则在边缘会开始出现重叠的线条。如果点的原点恰好位于方格的边缘，导致其主体溢出方格区域，也会发生这种情况。我们可以通过检查周围的方格来修复这个问题，但这意味着除了当前方格之外，我们还将检查另外八个方格，增加了搜索空间，但结果将更精确。

我们可以使用这种技术的最后优化是，如果我们的步长（每条线中每个点之间的距离）足够小，我们可以跳过检查方格中的几个点，因为如果我们的圆与其中一个圆重叠，则与其他一些圆重叠的可能性很高。现在，这可能会产生一个不太准确的结果，但准确性并不一定是最终目标。一些线条的小重叠可能会引入一些视觉上令人愉悦的艺术效果。通常，这些细节都是*令人愉快的小意外*。

```
[object Object]
```

在这里，我们检查每个方格中的第 7 个圆，希望如果有的话会碰到。常数`7`在某些情况下可能太高，或者甚至可以增加更多，这完全取决于线条的步长，可以进行调整以在渲染时间和正确性之间取得良好的平衡。

## 最后，颜色。

本文的主题是收敛于*"通过一些调整你可以实现很多变化"*。这对于给这些图像着色也是正确的。到目前为止，本文的内容都非常单调，重点是着重于如何实现总体外观的基本技术。

最简单的方式是创建一个包含几种不同颜色的调色板，在创建新行时随机选择颜色。

```
[object Object]
```

另一种着色方法是通过根据线条起始点的噪声函数角度来对每条线条着色。这将在较大的图像上产生类似渐变的着色效果。

```
[object Object]
```

最后，我最喜欢的方法是将画布细分为一组区域，可以采用一些[Piet Mondrian Composition style](https://en.wikipedia.org/wiki/Composition_with_Red_Blue_and_Yellow)的方式，或者通过递归地将画布分割成越来越精细的多边形。画布被划分后，我会为每个区域分配一种颜色，并在线条产生时为其分配该区域的颜色。

这种方法产生了一个很好的效果，使得事物看起来不那么不连贯，更像是油漆流入其他油漆桶的流动。

## 结论

尽管这篇文章已经相当长了，但它只是描述了使用基本技术所能实现的各种变体的冰山一角。我强烈建议尝试各种方法和实验，尝试在某处用`cos()`换成`sin()`，或者如果你很疯狂的话，甚至用`tan()`。

尝试将画布细分为各自具有自己规则的子区域，或者更深入地研究噪声函数。或者在画布周围留有小边框，让一些线条的一小部分逃离。为什么要使用线条？为什么不使用圆形、正方形或斑点？

感谢你一直坚持到这里，希望这篇文章有所帮助。
