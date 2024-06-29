<!--yml

category: 未分类

date: 2024-05-27 13:35:26

-->

# 3D空间中的平面

> 来源：[https://alexharri.com/blog/planes](https://alexharri.com/blog/planes)

<main class="css-fkkl8v">

# 3D空间中的平面

2024年4月27日

3D空间中的平面可以看作是一个平坦的表面，无限延伸，将空间分为两半。

在处理3D几何的应用程序中，平面有很多用途。我主要在[建筑建模师](https://www.arkio.is/)的背景下使用它们，其中几何图形是通过平面及其交集来定义的。

学习平面感觉抽象且不直观。*“当然，那是一个平面方程，但我该怎么办？平面看起来什么样？”* 我花了一些时间来建立关于如何理解和处理它们的直觉。

在撰写本文时，我希望为您提供一个专注于建立对平面实用和直观理解的介绍。我希望通过视觉（和互动！）解释来实现这一目标，这些解释将随着我们逐步解决更复杂的问题而进行。

说完这些，让我们开始吧！

## 描述平面

描述平面的方法有很多，比如通过

1.  3D空间中的一个点和一个法线，

1.  3D空间中的三个点形成一个三角形，或者

1.  一个法线和一个来自原点的距离。

在本文中，“*正常*”一词将指代一个*标准化方向向量*（单位向量），其大小（长度）等于1，通常表示为<mjx-container classname="MathJax" jax="SVG"></mjx-container>其中<mjx-container classname="MathJax" jax="SVG"></mjx-container>。

从点和法线情况开始，这是一个由3D空间中一个点<mjx-container classname="MathJax" jax="SVG"></mjx-container>和一个法线<mjx-container classname="MathJax" jax="SVG"></mjx-container>描述的平面的示例：

法线<mjx-container classname="MathJax" jax="SVG"></mjx-container>描述了平面的方向，其中平面的表面垂直于<mjx-container classname="MathJax" jax="SVG"></mjx-container>，而点<mjx-container classname="MathJax" jax="SVG"></mjx-container>描述了平面上的*一个*点。

我们用一个点<mjx-container classname="MathJax" jax="SVG"></mjx-container>描述了这个平面，但请记住这个平面——我们称之为<mjx-container classname="MathJax" jax="SVG"></mjx-container>——包含无限多个点。

如果<mjx-container classname="MathJax" jax="SVG"></mjx-container>由其中一个其他点描述，我们将描述相同的平面。这是平面无限性质的结果。

这种描述平面的方式——通过一个点和一个法线——是平面的[点法式](https://en.wikipedia.org/wiki/Euclidean_planes_in_three-dimensional_space#Point%E2%80%93normal_form_and_general_form_of_the_equation_of_a_plane)。

我们还可以使用三维空间中的三个点 <mjx-container classname="MathJax" jax="SVG"></mjx-container>, <mjx-container classname="MathJax" jax="SVG"></mjx-container>, <mjx-container classname="MathJax" jax="SVG"></mjx-container> 来描述一个平面形成的三角形：

这个三角形形成一个隐式平面，但为了能够对平面进行有用的操作，我们需要计算它的法向量 <mjx-container classname="MathJax" jax="SVG"></mjx-container>。一旦我们计算出平面的法向量，我们就可以使用这个法向量以及三角形的一个顶点来描述平面的点法式。

正如前面提到的，描述平面的法向量 <mjx-container classname="MathJax" jax="SVG"></mjx-container> 是一个垂直于平面的单位向量 (<mjx-container classname="MathJax" jax="SVG"></mjx-container>)。

我们可以使用 <mjx-container classname="MathJax" jax="SVG"></mjx-container> 和 <mjx-container classname="MathJax" jax="SVG"></mjx-container> 作为平面表面平行的两个边向量。

由于与平面表面平行，向量 <mjx-container classname="MathJax" jax="SVG"></mjx-container> 和 <mjx-container classname="MathJax" jax="SVG"></mjx-container> 都垂直于平面的法向量。这就是叉乘对我们有用的地方。

[叉乘](https://en.wikipedia.org/wiki/Cross_product) 接受两个向量 <mjx-container classname="MathJax" jax="SVG"></mjx-container> 和 <mjx-container classname="MathJax" jax="SVG"></mjx-container>，返回一个垂直于它们的向量 <mjx-container classname="MathJax" jax="SVG"></mjx-container>。

<mjx-container classname="MathJax" jax="SVG"></mjx-container>

例如，给定向量 <mjx-container classname="MathJax" jax="SVG"></mjx-container> 和 <mjx-container classname="MathJax" jax="SVG"></mjx-container>，它们的叉乘是向量 <mjx-container classname="MathJax" jax="SVG"></mjx-container>，我们将其标记为 <mjx-container classname="MathJax" jax="SVG"></mjx-container>：

这个解释故意简单。稍后我们将更详细地讨论叉乘。

因为三角形的边向量 <mjx-container classname="MathJax" jax="SVG"></mjx-container> 和 <mjx-container classname="MathJax" jax="SVG"></mjx-container> 都与三角形的表面平行，它们的叉乘将垂直于三角形的表面。让我们将两个边向量的叉乘命名为 <mjx-container classname="MathJax" jax="SVG"></mjx-container>：

<mjx-container classname="MathJax" jax="SVG"></mjx-container>

<mjx-container classname="MathJax" jax="SVG"></mjx-container> 已经被缩小了，仅用于说明目的。

<mjx-container classname="MathJax" jax="SVG"></mjx-container>指向正确的方向，但它不是一个法线。要成为法线，其大小必须等于1。我们可以通过将<mjx-container classname="MathJax" jax="SVG"></mjx-container>除以其大小来对其进行归一化，将结果分配给<mjx-container classname="MathJax" jax="SVG"></mjx-container>：

<mjx-container classname="MathJax" jax="SVG"></mjx-container>

这给出了一个常态<mjx-container classname="MathJax" jax="SVG"></mjx-container>，其中<mjx-container classname="MathJax" jax="SVG"></mjx-container>：

找到三角形的法线<mjx-container classname="MathJax" jax="SVG"></mjx-container>后，我们可以使用它和任意的点<mjx-container classname="MathJax" jax="SVG"></mjx-container>，<mjx-container classname="MathJax" jax="SVG"></mjx-container>，<mjx-container classname="MathJax" jax="SVG"></mjx-container>来描述包含这三个点的平面的点法线形式。

使用点法线形式时，无论我们使用哪个点作为<mjx-container classname="MathJax" jax="SVG"></mjx-container>，我们始终得到同一个平面。

### 常数法线形式

描述平面的另一种方法是通过一个法线<mjx-container classname="MathJax" jax="SVG"></mjx-container>和一个距离<mjx-container classname="MathJax" jax="SVG"></mjx-container>。

这是平面的*常数法线形式*，它使得使用平面进行许多计算变得更简单。

在常数法线形式中，距离<mjx-container classname="MathJax" jax="SVG"></mjx-container>表示平面靠近原点的程度。另一种想法：将法线<mjx-container classname="MathJax" jax="SVG"></mjx-container>乘以<mjx-container classname="MathJax" jax="SVG"></mjx-container>，得到离原点最近的平面上的点。

这是一个简化。更正式地说，给定平面上一点<mjx-container classname="MathJax" jax="SVG"></mjx-container>其法线为<mjx-container classname="MathJax" jax="SVG"></mjx-container>，我们可以用两种形式描述平面上的所有点：点法线形式<mjx-container classname="MathJax" jax="SVG"></mjx-container>和常数法线形式<mjx-container classname="MathJax" jax="SVG"></mjx-container>，其中<mjx-container classname="MathJax" jax="SVG"></mjx-container>。详见[进一步阅读](/blog/planes#further-reading)。

为了感受点法线形式和常数法线形式之间的差异，可以看这个例子，它在两种形式中描述了同一个平面：

绿色箭头代表常法式中的<mjx-container classname="MathJax" jax="SVG"></mjx-container>，而蓝色点和箭头代表点法式中的点<mjx-container classname="MathJax" jax="SVG"></mjx-container>和法向量<mjx-container classname="MathJax" jax="SVG"></mjx-container>。

从点法式到常法式的转换非常简单：距离<mjx-container classname="MathJax" jax="SVG"></mjx-container>是[mjx-container classname="MathJax" jax="SVG"></mjx-container]的点积。

<mjx-container classname="MathJax" jax="SVG"></mjx-container>

如果你对点积不熟悉，别担心，我们稍后会介绍。

<mjx-container classname="MathJax" jax="SVG"></mjx-container>和<mjx-container classname="MathJax" jax="SVG"></mjx-container>的符号表示可能表明它们属于不同类型，但它们都是向量。我通过仅使用箭头符号区分空间中的点（例如<mjx-container classname="MathJax" jax="SVG"></mjx-container>和<mjx-container classname="MathJax" jax="SVG"></mjx-container>）和方向向量（例如<mjx-container classname="MathJax" jax="SVG"></mjx-container>和<mjx-container classname="MathJax" jax="SVG"></mjx-container>）。

法向量<mjx-container classname="MathJax" jax="SVG"></mjx-container>在两种形式中保持不变。

## 平面距离

给定任意点<mjx-container classname="MathJax" jax="SVG"></mjx-container>和常法式的平面<mjx-container classname="MathJax" jax="SVG"></mjx-container>，我们可能想知道点到平面的距离有多远。换句话说，<mjx-container classname="MathJax" jax="SVG"></mjx-container>需要移动多少才能在平面上。

如果我们构造一个包含<mjx-container classname="MathJax" jax="SVG"></mjx-container>的平面<mjx-container classname="MathJax" jax="SVG"></mjx-container>并且平行于<mjx-container classname="MathJax" jax="SVG"></mjx-container>，我们可以使用点法式，其中<mjx-container classname="MathJax" jax="SVG"></mjx-container>作为点，<mjx-container classname="MathJax" jax="SVG"></mjx-container>的法向量作为法向量：

有了两个平行平面，我们可以将问题构造为找到两个平面之间的距离。使用它们的常法式使这变得微不足道，因为它允许我们取其距离分量<mjx-container classname="MathJax" jax="SVG"></mjx-container>和<mjx-container classname="MathJax" jax="SVG"></mjx-container>的差异。

所以让我们使用我们学到的方程式来找到<mjx-container classname="MathJax" jax="SVG"></mjx-container>的距离：

通过平面的两个距离<mjx-container classname="MathJax" jax="SVG"></mjx-container>和<mjx-container classname="MathJax" jax="SVG"></mjx-container>，解决方案简化为：

<mjx-container classname="MathJax" jax="SVG"></mjx-container>

因此，简而言之，给定一个法线为<mjx-container classname="MathJax" jax="SVG"></mjx-container>距离为<mjx-container classname="MathJax" jax="SVG"></mjx-container>的平面，我们可以这样计算点<mjx-container classname="MathJax" jax="SVG"></mjx-container>到<mjx-container classname="MathJax" jax="SVG"></mjx-container>的距离：

<mjx-container classname="MathJax" jax="SVG"></mjx-container>

距离可能是正数或负数，这取决于点位于平面的哪一侧。

### 投影点到平面上

计算点到平面的距离变得有用的一个情况是，例如，如果你想将一个点投影到平面上。

给定一个点<mjx-container classname="MathJax" jax="SVG"></mjx-container>我们希望投影到法线为<mjx-container classname="MathJax" jax="SVG"></mjx-container>距离为<mjx-container classname="MathJax" jax="SVG"></mjx-container>的平面上，我们可以相对容易地做到这一点。首先，让我们定义<mjx-container classname="MathJax" jax="SVG"></mjx-container>作为点到平面的距离：

<mjx-container classname="MathJax" jax="SVG"></mjx-container>

将平面的法线<mjx-container classname="MathJax" jax="SVG"></mjx-container>乘以<mjx-container classname="MathJax" jax="SVG"></mjx-container>得到一个向量，将其加到<mjx-container classname="MathJax" jax="SVG"></mjx-container>可以将其投影到平面上。我们将投影后的点称为<mjx-container classname="MathJax" jax="SVG"></mjx-container>：

<mjx-container classname="MathJax" jax="SVG"></mjx-container>

投影沿着平面的法线进行，这有时是有用的。然而，能够沿着*任意*方向将点投影到平面上则更加有用。这涉及到找到直线和平面的交点。

## 线平面交点

我们可以用一个点<mjx-container classname="MathJax" jax="SVG"></mjx-container>和法线<mjx-container classname="MathJax" jax="SVG"></mjx-container>描述三维空间中的直线。法线<mjx-container classname="MathJax" jax="SVG"></mjx-container>描述了线的方向，而点<mjx-container classname="MathJax" jax="SVG"></mjx-container>描述了线通过的点。

在本章中，线由点<mjx-container classname="MathJax" jax="SVG"></mjx-container>和法线<mjx-container classname="MathJax" jax="SVG"></mjx-container>组成，而平面——以常法形式给出——具有法线<mjx-container classname="MathJax" jax="SVG"></mjx-container>和距离<mjx-container classname="MathJax" jax="SVG"></mjx-container>。

我们的目标是找到一个距离<mjx-container classname="MathJax" jax="SVG"></mjx-container>，使得<mjx-container classname="MathJax" jax="SVG"></mjx-container>沿着<mjx-container classname="MathJax" jax="SVG"></mjx-container>移动，使其位于平面上。

我们可以计算出<mjx-container classname="MathJax" jax="SVG"></mjx-container>需要移动的距离，如果<mjx-container classname="MathJax" jax="SVG"></mjx-container>和<mjx-container classname="MathJax" jax="SVG"></mjx-container>是平行的，这就是我们在投影平面法线时所做的。

<mjx-container classname="MathJax" jax="SVG"></mjx-container>

让我们尝试使用<mjx-container classname="MathJax" jax="SVG"></mjx-container>作为标量来沿着<mjx-container classname="MathJax" jax="SVG"></mjx-container>投影<mjx-container classname="MathJax" jax="SVG"></mjx-container>：

<mjx-container classname="MathJax" jax="SVG"></mjx-container>

我们将<mjx-container classname="MathJax" jax="SVG"></mjx-container>视为一个红点：

当<mjx-container classname="MathJax" jax="SVG"></mjx-container>和<mjx-container classname="MathJax" jax="SVG"></mjx-container>变得平行时，<mjx-container classname="MathJax" jax="SVG"></mjx-container>使我们越来越接近正确的解。然而，随着<mjx-container classname="MathJax" jax="SVG"></mjx-container>和<mjx-container classname="MathJax" jax="SVG"></mjx-container>之间的角度增加，<mjx-container classname="MathJax" jax="SVG"></mjx-container>变得越来越小。

在这里，点积非常有用。对于两个向量<mjx-container classname="MathJax" jax="SVG"></mjx-container>和<mjx-container classname="MathJax" jax="SVG"></mjx-container>，点积定义为

<mjx-container classname="MathJax" jax="SVG"></mjx-container>

其中<mjx-container classname="MathJax" jax="SVG"></mjx-container>是<mjx-container classname="MathJax" jax="SVG"></mjx-container>和<mjx-container classname="MathJax" jax="SVG"></mjx-container>之间的角度。

考虑<mjx-container classname="MathJax" jax="SVG"></mjx-container>和<mjx-container classname="MathJax" jax="SVG"></mjx-container>的点积。由于两个法线都是单位向量，其大小为1

<mjx-container classname="MathJax" jax="SVG"></mjx-container>

我们可以从方程中消去它们的大小，

<mjx-container classname="MathJax" jax="SVG"></mjx-container>

使 <mjx-container classname="MathJax" jax="SVG"></mjx-container> 和 <mjx-container classname="MathJax" jax="SVG"></mjx-container> 的点积成为它们之间角度的余弦。

对于两个向量，它们的夹角余弦随着向量变得越来越平行而接近 1，随着向量变得越来越垂直而接近 0。

由于在两个向量变得越来越垂直时，<mjx-container classname="MathJax" jax="SVG"></mjx-container> 变得越来越小，我们可以用它作为 <mjx-container classname="MathJax" jax="SVG"></mjx-container> 的分母。我们将这个放大版本的 <mjx-container classname="MathJax" jax="SVG"></mjx-container> 赋给 <mjx-container classname="MathJax" jax="SVG"></mjx-container>：

<mjx-container classname="MathJax" jax="SVG"></mjx-container>

作为我们放大的距离，我们通过以下方法找到交点：

<mjx-container classname="MathJax" jax="SVG"></mjx-container>

现在我们可以摆脱被定义为 <mjx-container classname="MathJax" jax="SVG"></mjx-container> 的 <mjx-container classname="MathJax" jax="SVG"></mjx-container>，从而得到 <mjx-container classname="MathJax" jax="SVG"></mjx-container> 的完整方程：

<mjx-container classname="MathJax" jax="SVG"></mjx-container>

将这些转化为代码，我们得到：

```
Vector3  LinePlaneIntersection(Line line,  Plane plane)  {  float denom = Vector3.Dot(line.normal, plane.normal);  float dist = Vector3.Dot(plane.normal, line.point);  float D =  (plane.distance - dist)  / denom;  return line.point + line.normal * D;}
```

然而，我们的代码还没有完全完成。在直线与平面表面平行的情况下，直线和平面不相交。

当 <mjx-container classname="MathJax" jax="SVG"></mjx-container> 和 <mjx-container classname="MathJax" jax="SVG"></mjx-container> 垂直时，它们的点积为零。因此，如果 <mjx-container classname="MathJax" jax="SVG"></mjx-container>，则直线和平面不相交。这给了我们一个可以添加到代码中的简单测试，以产生“无交点”的结果。

然而，对于许多应用场景，我们希望将*几乎*平行视为实际平行。为了做到这一点，我们可以检查点积是否小于某个非常小的数值——通常称为 epsilon。

```
float denom = Vector3.Dot(line.normal, plane.normal);if  (Mathf.Abs(denom)  < EPSILON)  {  return  null;  }
```

看看你能否弄清楚为什么在这里使用 Mathf.Abs。我们稍后会详细介绍，所以你会知道自己是否正确。

我们将在后面的一章中讨论如何选择 epsilon 的值，用于两个平面的交点。

有了这个，我们的线平面交点实现变成了：

```
Vector3  LinePlaneIntersection(Line line,  Plane plane)  {  float denom = Vector3.Dot(line.normal, plane.normal);  if  (Mathf.Abs(denom)  < EPSILON)  {  return  null;    }  float dist = Vector3.Dot(plane.normal, line.point);  float D =  (plane.distance - dist)  / denom;  return line.point + line.normal * D;}
```

### 射线和直线

我们一直在谈论线与平面的交点，但为了视觉上的清晰度，我有点撒谎，其实是在想射线与平面的交点。

射线和直线非常相似；它们都通过一个法向量 <mjx-container classname="MathJax" jax="SVG"></mjx-container> 和一个点 <mjx-container classname="MathJax" jax="SVG"></mjx-container> 表示。

区别在于射线（红色）朝 远离扩展，而线段（绿色）朝另一个方向扩展：

这意味着射线沿其法向后退时不会与平面相交：

我们的射线-平面交点实现与现有的线-平面交点实现不同，只是在射线的法向量 指向平面的法向量 为钝角时应得到 "无交点" 的结果。

由于 表示沿法向量前进到达交点的距离，所以当 变为负数时我们可以得到 "无交点" 的结果：

```
if  (D <  0)  {  return  null;}
```

但然后我们必须首先计算 。由于 和 之间的角度为90°至180°之间，所以这并不必要。

如果这看起来不明显，记住点乘编码其两个分量向量之间的余弦值，这就是为什么在钝角时点乘的结果变为负数。

知道了这一点，我们可以将初始的 "法向量平行" 测试改为：

```
Vector3  LinePlaneIntersection(Line line,  Plane plane)  {  float denom = Vector3.Dot(line.normal, plane.normal);  if  (Mathf.Abs(denom)  < EPSILON)  {  return  null;    }}
```

对于这一点：

```
Vector3  RayPlaneIntersection(Line line,  Plane plane)  {  float denom = Vector3.Dot(line.normal, plane.normal);  if  (denom < EPSILON)  {  return  null;  }}
```

检查同时涵盖了 *"平行于平面的直线"* 情况 *以及* 两个法向量成钝角的情况。

注意： 是表示epsilon的符号。

## 平面-平面交点

两个平面的交点形成一个无限长的直线。

快速回顾一下：在三维空间中，线段用一个点 和法向量 来表示，其中法向量 描述了线段的方向，而点 是线段经过的一个点。

让我们取两个平面的法向量分别为 和 。

寻找 <mjx-container classname="MathJax" jax="SVG"></mjx-container> 和 <mjx-container classname="MathJax" jax="SVG"></mjx-container> 的方向向量看似简单。因为两个平面的交线位于两个平面的表面上，所以交线必须垂直于两个平面的法线，这意味着交线的方向是两个平面法线的叉积。我们将其赋值给 <mjx-container classname="MathJax" jax="SVG"></mjx-container>。

<mjx-container classname="MathJax" jax="SVG"></mjx-container>

叉积的大小等于两个分量向量形成的平行四边形的[面积](https://en.wikipedia.org/wiki/Cross_product#/media/File:Cross_product_parallelogram.svg)。这意味着我们不能期望叉积是单位向量，因此我们将对 <mjx-container classname="MathJax" jax="SVG"></mjx-container> 进行归一化，并将归一化后的方向向量赋给 <mjx-container classname="MathJax" jax="SVG"></mjx-container>。

<mjx-container classname="MathJax" jax="SVG"></mjx-container>

这给了我们交点的法线 <mjx-container classname="MathJax" jax="SVG"></mjx-container>。让我们放大看一看这个过程。

但这只是问题的一半！我们还需要找到一个空间点来表示交线（即线通过的点）。我们将在讨论无交点情况之后详细讨论如何做到这一点。

### 处理平行平面

两个法线平行的平面永远不会相交，这是一个我们必须处理的情况。

两个平行法线的叉积是 <mjx-container classname="MathJax" jax="SVG"></mjx-container>。所以如果 <mjx-container classname="MathJax" jax="SVG"></mjx-container>，那么这两个平面不相交。

正如前面提到的，对于许多应用，我们希望将几乎平行的平面视为平行。这意味着我们的平面相交过程应在 <mjx-container classname="MathJax" jax="SVG"></mjx-container> 的大小小于称为 epsilon 的非常小的数时产生“无交点”的结果。

```
Line  PlanePlaneIntersection(Plane P1,  Plane P2)  {  Vector3 direction = Vector3.cross(P1.normal, P2.normal);  if  (direction.magnitude < EPSILON)  {  return  null;    }}
```

但 epsilon 的值应该是多少？

给定两个法线 <mjx-container classname="MathJax" jax="SVG"></mjx-container> 和 <mjx-container classname="MathJax" jax="SVG"></mjx-container>，其中 <mjx-container classname="MathJax" jax="SVG"></mjx-container> 和 <mjx-container classname="MathJax" jax="SVG"></mjx-container> 之间的角度是 <mjx-container classname="MathJax" jax="SVG"></mjx-container>，我们可以通过绘制 <mjx-container classname="MathJax" jax="SVG"></mjx-container> 的不同值来找到一个合理的 epsilon：

两个轴都是[对数](https://en.wikipedia.org/wiki/Logarithmic_scale)的。

关系是线性的：随着平面之间夹角减半，它们法向量的叉积大小也减半。<mjx-container classname="MathJax" jax="SVG"></mjx-container>的大小为<mjx-container classname="MathJax" jax="SVG"></mjx-container>，<mjx-container classname="MathJax" jax="SVG"></mjx-container>的大小为它的一半。

因此，要确定 epsilon，我们可以问：角度需要多小（以度为单位）才能认为两个平面是平行的？给定角度<mjx-container classname="MathJax" jax="SVG"></mjx-container>，我们可以通过以下公式找到 epsilon：<mjx-container classname="MathJax" jax="SVG"></mjx-container>。

<mjx-container classname="MathJax" jax="SVG"></mjx-container>

如果那个角度是1/256°，我们得到：

<mjx-container classname="MathJax" jax="SVG"></mjx-container>

通过这样做，您可以根据平面之间的角度多小来确定适当的 epsilon。这将取决于您的使用情况。

### 寻找交点

在计算法向量并处理平行平面后，我们可以继续找到沿交线的点<mjx-container classname="MathJax" jax="SVG"></mjx-container>。

由于描述平面-平面交点的线是无限的，我们可以选择无限多个点作为<mjx-container classname="MathJax" jax="SVG"></mjx-container>。

我们可以通过取与两个平面法线平行的平面<mjx-container classname="MathJax" jax="SVG"></mjx-container>来缩小问题范围，并观察它在一点上的交点。

由于该点位于与两个平面法线平行的平面上，我们可以通过沿着这些法线独占行进来找到它。

最简单的情况是<mjx-container classname="MathJax" jax="SVG"></mjx-container>和<mjx-container classname="MathJax" jax="SVG"></mjx-container>垂直的情况。在这种情况下，解决方案就是<mjx-container classname="MathJax" jax="SVG"></mjx-container>。以下是其可视化效果：

当拖动滑块时，请注意随着平面变得更加平行，平行四边形顶点离交点的距离增加。

我们还可以观察到，随着我们远离交点，较长的两个向量（红色）将我们远离交点，而较短的（蓝色）向量则不会。如果我们从原点到交点画一条线，这更容易观察：

让我们定义<mjx-container classname="MathJax" jax="SVG"></mjx-container>和<mjx-container classname="MathJax" jax="SVG"></mjx-container>为我们应用于<mjx-container classname="MathJax" jax="SVG"></mjx-container>和<mjx-container classname="MathJax" jax="SVG"></mjx-container>的缩放因子（其结果是红色和蓝色向量）。现在我们正在使用平面的距离分量<mjx-container classname="MathJax" jax="SVG"></mjx-container>和<mjx-container classname="MathJax" jax="SVG"></mjx-container>作为缩放因子：

`<mjx-container classname="MathJax" jax="SVG"></mjx-container>`

`<mjx-container classname="MathJax" jax="SVG"></mjx-container>`

要解决这种不对称的推力效应，我们需要沿着较长向量的方向行进较少。我们需要某种调整向量的“拉动因子”，使其尖端在平行平面上保持在直线上。

这里我们的老朋友点积再次派上了用场。当平面垂直时，<mjx-container classname="MathJax" jax="SVG"></mjx-container>和<mjx-container classname="MathJax" jax="SVG"></mjx-container>的点积等于0，但随着平面逐渐平行，它接近于1。我们可以利用这一点逐渐增加我们尚未定义的拉动因子。

`<mjx-container classname="MathJax" jax="SVG"></mjx-container>`

`<mjx-container classname="MathJax" jax="SVG"></mjx-container>`

让我们给点积<mjx-container classname="MathJax" jax="SVG"></mjx-container>一个名字<mjx-container classname="MathJax" jax="SVG"></mjx-container>，以减少一些噪音：

`<mjx-container classname="MathJax" jax="SVG"></mjx-container>`

`<mjx-container classname="MathJax" jax="SVG"></mjx-container>`

完美的拉动因子恰好是距离分量<mjx-container classname="MathJax" jax="SVG"></mjx-container>和<mjx-container classname="MathJax" jax="SVG"></mjx-container>作为彼此的抵消！

`<mjx-container classname="MathJax" jax="SVG"></mjx-container>`

`<mjx-container classname="MathJax" jax="SVG"></mjx-container>`

考虑为什么会这样。当<mjx-container classname="MathJax" jax="SVG"></mjx-container>和<mjx-container classname="MathJax" jax="SVG"></mjx-container>垂直时，它们的点积等于0，这导致

`<mjx-container classname="MathJax" jax="SVG"></mjx-container>`

`<mjx-container classname="MathJax" jax="SVG"></mjx-container>`

我们知道这将产生正确的解。

在<mjx-container classname="MathJax" jax="SVG"></mjx-container>和<mjx-container classname="MathJax" jax="SVG"></mjx-container>平行的情况下，它们的点积等于1，导致：

`<mjx-container classname="MathJax" jax="SVG"></mjx-container>`

`<mjx-container classname="MathJax" jax="SVG"></mjx-container>`

因为<mjx-container classname="MathJax" jax="SVG"></mjx-container>和<mjx-container classname="MathJax" jax="SVG"></mjx-container>的绝对值相等，这意味着两个向量的大小—定义为<mjx-container classname="MathJax" jax="SVG"></mjx-container>和<mjx-container classname="MathJax" jax="SVG"></mjx-container>—是相等的：

<mjx-container classname="MathJax" jax="SVG"></mjx-container>

这意味着随着平面变得平行，我们的向量大小将变得*更加*相等，这正是我们想要的！

让我们看看这一过程：

向量保持在同一条直线上，但当<mjx-container classname="MathJax" jax="SVG"></mjx-container>和<mjx-container classname="MathJax" jax="SVG"></mjx-container>变得平行时，它们的长度逐渐变短。

再次，我们可以使用点积。因为我们希望向量的长度随着平面变得平行而增加，我们可以将我们的标量<mjx-container classname="MathJax" jax="SVG"></mjx-container>和<mjx-container classname="MathJax" jax="SVG"></mjx-container>除以<mjx-container classname="MathJax" jax="SVG"></mjx-container>，其中<mjx-container classname="MathJax" jax="SVG"></mjx-container>是<mjx-container classname="MathJax" jax="SVG"></mjx-container>和<mjx-container classname="MathJax" jax="SVG"></mjx-container>的点积，而<mjx-container classname="MathJax" jax="SVG"></mjx-container>是<mjx-container classname="MathJax" jax="SVG"></mjx-container>的绝对值。

<mjx-container classname="MathJax" jax="SVG"></mjx-container>

<mjx-container classname="MathJax" jax="SVG"></mjx-container>

这个结果看起来是这样的：

使用<mjx-container classname="MathJax" jax="SVG"></mjx-container>作为分母确实会增加平行四边形的大小，但增幅过大。

然而，请注意当我们可视化平行四边形的象限时发生了什么：

随着平面变得更加平行，交点逼近平行四边形的中心。

要理解为什么是这样，请考虑我们的分母<mjx-container classname="MathJax" jax="SVG"></mjx-container>对平行四边形面积的影响。当<mjx-container classname="MathJax" jax="SVG"></mjx-container>时，形成平行四边形的两个向量长度加倍，这导致平行四边形的面积增加四倍。

这意味着当我们按照平行四边形的分量向量进行缩放时，

<mjx-container classname="MathJax" jax="SVG"></mjx-container>

它具有通过缩放平行四边形的面积的效果：

<mjx-container classname="MathJax" jax="SVG"></mjx-container>

要通过<mjx-container classname="MathJax" jax="SVG"></mjx-container>来缩放平行四边形的*面积*，我们需要将<mjx-container classname="MathJax" jax="SVG"></mjx-container>在分母中平方：

<mjx-container classname="MathJax" jax="SVG"></mjx-container>

平方使我们可以去除 <mjx-container classname="MathJax" jax="SVG"></mjx-container>，因为负数的平方是正数。

有了这个，我们的标量 <mjx-container classname="MathJax" jax="SVG"></mjx-container> 和 <mjx-container classname="MathJax" jax="SVG"></mjx-container> 变为：

<mjx-container classname="MathJax" jax="SVG"></mjx-container>

<mjx-container classname="MathJax" jax="SVG"></mjx-container>

这会缩放平行四边形，使其顶点位于交点处：

把所有这些放进代码，我们得到：

```
float dot = Vector3.Dot(P1.normal, P2.normal);float denom =  1  - dot * dot;float k1 =  (P1.distance - P2.distance * dot)  / denom;float k2 =  (P2.distance - P1.distance * dot)  / denom;Vector3 point = P1.normal * k1 + P2.normal * k2;
```

基于来自 [Christer Ericson 的《实时碰撞检测》](/blog/planes#further-reading) 的代码。

通过一些数学魔法，可以将其优化为：

```
Vector3 direction = Vector3.cross(P1.normal, P2.normal);float denom = Vector3.Dot(direction, direction);Vector3 a = P1.distance * P2.normal;Vector3 b = P2.distance * P1.normal;Vector3 point = Vector3.Cross(a - b, direction)  / denom;
```

如何进行这种优化可以在 [Christer Ericson 的《实时碰撞检测》](/blog/planes#further-reading) 的第5.4.4章找到。

这完成了我们的平面-平面相交实现：

```
Line  PlanePlaneIntersection(Plane P1,  Plane P2)  {  Vector3 direction = Vector3.cross(P1.normal, P2.normal);  if  (direction.magnitude < EPSILON)  {  return  null;    }  float denom = Vector3.Dot(direction, direction);  Vector3 a = P1.distance * P2.normal;  Vector3 b = P2.distance * P1.normal;  Vector3 point = Vector3.Cross(a - b, direction)  / denom;  Vector3 normal = direction.normalized;  return  new  Line(point, normal);}
```

顺便说一句，沿着平面法线旅行的一个有趣特性是，它给出了与原点最近的交线上的点。很酷的东西！

## 三平面相交

给定三个平面 <mjx-container classname="MathJax" jax="SVG"></mjx-container>, <mjx-container classname="MathJax" jax="SVG"></mjx-container>, <mjx-container classname="MathJax" jax="SVG"></mjx-container>，它们有五种可能的配置，可以相交或不相交：

1.  所有三个平面都平行，彼此不相交。

1.  两个平面平行，第三个平面与其他两个相交。

1.  所有三个平面沿一条直线相交。

1.  三个平面两两相交，形成三条平行的相交线。

1.  所有三个平面在一个点相交。

在找到交点时，我们首先需要确定所有三个平面是否在一个点相交——对于配置1到4，它们不会相交。

给定平面法线 <mjx-container classname="MathJax" jax="SVG"></mjx-container>, <mjx-container classname="MathJax" jax="SVG"></mjx-container>, <mjx-container classname="MathJax" jax="SVG"></mjx-container>，我们可以使用以下公式确定平面是否在一个点相交：

<mjx-container classname="MathJax" jax="SVG"></mjx-container>

当我第一次看到这个时，我觉得难以相信这对所有情况都有效。但它确实有效！让我们深入了解发生了什么。

### 两个或更多平面平行

我们将从两个或更多平面平行的配置开始：

如果 <mjx-container classname="MathJax" jax="SVG"></mjx-container> 和 <mjx-container classname="MathJax" jax="SVG"></mjx-container> 平行，则 <mjx-container classname="MathJax" jax="SVG"></mjx-container> 是一个大小为零的向量。

<mjx-container classname="MathJax" jax="SVG"></mjx-container>

由于点乘是其组分向量的大小的倍数：

<mjx-container classname="MathJax" jax="SVG"></mjx-container>

最终结果是，当 <mjx-container classname="MathJax" jax="SVG"></mjx-container> 和 <mjx-container classname="MathJax" jax="SVG"></mjx-container> 平行时，结果为零。

<mjx-container classname="MathJax" jax="SVG"></mjx-container>

这解决了“所有平面平行”配置，以及 <mjx-container classname="MathJax" jax="SVG"></mjx-container> 和 <mjx-container classname="MathJax" jax="SVG"></mjx-container> 平行的配置。

因此，让我们考虑 <mjx-container classname="MathJax" jax="SVG"></mjx-container> 平行于 <mjx-container classname="MathJax" jax="SVG"></mjx-container> 或 <mjx-container classname="MathJax" jax="SVG"></mjx-container> 的情况，但 <mjx-container classname="MathJax" jax="SVG"></mjx-container> 和 <mjx-container classname="MathJax" jax="SVG"></mjx-container> 之间不平行。

让我们考虑 <mjx-container classname="MathJax" jax="SVG"></mjx-container> 平行于 <mjx-container classname="MathJax" jax="SVG"></mjx-container>，但 <mjx-container classname="MathJax" jax="SVG"></mjx-container> 既不平行于这个也不平行于那个的特定情况。

在这里，叉乘 <mjx-container classname="MathJax" jax="SVG"></mjx-container> 是一个向量（红色），它垂直于 <mjx-container classname="MathJax" jax="SVG"></mjx-container> 和 <mjx-container classname="MathJax" jax="SVG"></mjx-container>。

由于 <mjx-container classname="MathJax" jax="SVG"></mjx-container> 平行于 <mjx-container classname="MathJax" jax="SVG"></mjx-container>，这意味着 <mjx-container classname="MathJax" jax="SVG"></mjx-container> 也垂直于 <mjx-container classname="MathJax" jax="SVG"></mjx-container>。正如我们所学，两个垂直向量的点积为零，这意味着：

<mjx-container classname="MathJax" jax="SVG"></mjx-container>

这也适用于 <mjx-container classname="MathJax" jax="SVG"></mjx-container> 平行于 <mjx-container classname="MathJax" jax="SVG"></mjx-container> 而非 <mjx-container classname="MathJax" jax="SVG"></mjx-container> 的情况。

### 平行的交线

我们已经证明，其中三个法线中有两个平行会导致 <mjx-container classname="MathJax" jax="SVG"></mjx-container>。但是对于三个平面沿平行线交叉的配置，这些配置没有平行的法线。

当我们观察平面平行交线时，两个平面法线的叉乘给出了平面交线的方向向量。

当所有交线都平行时，定义这些线的平面法线都垂直于它们。

再次因为垂直向量的点积为 0，我们可以得出这些配置下 <mjx-container classname="MathJax" jax="SVG"></mjx-container>。

现在我们可以开始我们的实现。像往常一样，我们将使用一个 epsilon 来处理 *"大致平行"* 的情况：

```
Vector3  ThreePlaneIntersection(Plane P1,  Plane P2,  Plane P3)  {  Vector3 cross = Vector3.Cross(P2.normal, P3.normal);  float dot = Vector3.Dot(P1.normal, cross);  if  (Mathf.Abs(dot)  < EPSILON)  {  return  null;    }}
```

## 计算交点

我们想找到三个平面 <mjx-container classname="MathJax" jax="SVG"></mjx-container>, <mjx-container classname="MathJax" jax="SVG"></mjx-container>, <mjx-container classname="MathJax" jax="SVG"></mjx-container> 的交点：

我们在两平面交点部分学到的一些知识将在这里发挥作用。让我们从取得 <mjx-container classname="MathJax" jax="SVG"></mjx-container> 和 <mjx-container classname="MathJax" jax="SVG"></mjx-container> 的交线开始，并改变 <mjx-container classname="MathJax" jax="SVG"></mjx-container> 的位置。你会注意到交点就是 <mjx-container classname="MathJax" jax="SVG"></mjx-container> 与这条线相交的点。

当 <mjx-container classname="MathJax" jax="SVG"></mjx-container> 距离原点为 0 时，从原点指向交点的向量与 <mjx-container classname="MathJax" jax="SVG"></mjx-container> 平行（且垂直于 <mjx-container classname="MathJax" jax="SVG"></mjx-container> 的法线）。

这个向量，我们称之为 <mjx-container classname="MathJax" jax="SVG"></mjx-container>，在计算交点时起到了重要作用。

通过两个其他向量的叉乘我们可以找到 <mjx-container classname="MathJax" jax="SVG"></mjx-container>，这两个向量分别是 <mjx-container classname="MathJax" jax="SVG"></mjx-container>，第一个只是 <mjx-container classname="MathJax" jax="SVG"></mjx-container> 的法线。

<mjx-container classname="MathJax" jax="SVG"></mjx-container>

后一个向量可以通过方程找到：

<mjx-container classname="MathJax" jax="SVG"></mjx-container>

其中 <mjx-container classname="MathJax" jax="SVG"></mjx-container> 和 <mjx-container classname="MathJax" jax="SVG"></mjx-container> 是平面 <mjx-container classname="MathJax" jax="SVG"></mjx-container> 和 <mjx-container classname="MathJax" jax="SVG"></mjx-container> 的常法形式中的距离。

有了 <mjx-container classname="MathJax" jax="SVG"></mjx-container> 和 <mjx-container classname="MathJax" jax="SVG"></mjx-container> 的定义，我们将它们的叉乘赋给 <mjx-container classname="MathJax" jax="SVG"></mjx-container>：

<mjx-container classname="MathJax" jax="SVG"></mjx-container>

让我们看看它是什么样子：

嗯，还不够长。 <mjx-container classname="MathJax" jax="SVG"></mjx-container>确实指向了正确的方向，但要使 <mjx-container classname="MathJax" jax="SVG"></mjx-container>的顶端位于交线上，我们需要计算一些缩放因子 <mjx-container classname="MathJax" jax="SVG"></mjx-container>。

结果表明，我们已经计算出了这个缩放因子：

<mjx-container classname="MathJax" jax="SVG"></mjx-container>

<mjx-container classname="MathJax" jax="SVG"></mjx-container>的乘积——我们称之为 <mjx-container classname="MathJax" jax="SVG"></mjx-container>——可以被认为表示 <mjx-container classname="MathJax" jax="SVG"></mjx-container>的法向量与 <mjx-container classname="MathJax" jax="SVG"></mjx-container>交线的平行程度。

当 <mjx-container classname="MathJax" jax="SVG"></mjx-container>的法向量与交线平行时，<mjx-container classname="MathJax" jax="SVG"></mjx-container>趋近于 <mjx-container classname="MathJax" jax="SVG"></mjx-container>，而它们垂直时趋近于 0。

我们希望 <mjx-container classname="MathJax" jax="SVG"></mjx-container>的大小随着 <mjx-container classname="MathJax" jax="SVG"></mjx-container>的减小而增加，因此我们将 <mjx-container classname="MathJax" jax="SVG"></mjx-container>作为 <mjx-container classname="MathJax" jax="SVG"></mjx-container>的缩放因子。

<mjx-container classname="MathJax" jax="SVG"></mjx-container>

完全展开，<mjx-container classname="MathJax" jax="SVG"></mjx-container>的方程为：

<mjx-container classname="MathJax" jax="SVG"></mjx-container>

啪！现在问题已经简化为沿着交线方向行进，直到与 <mjx-container classname="MathJax" jax="SVG"></mjx-container>相交。

我们可以利用我们对直线平面交点的知识来解决这个问题，但我想展示一种更有效的方法。

它涉及寻找一个方向向量 <mjx-container classname="MathJax" jax="SVG"></mjx-container>的缩放因子，使其顶端位于 <mjx-container classname="MathJax" jax="SVG"></mjx-container>。让我们称这个方向向量为 <mjx-container classname="MathJax" jax="SVG"></mjx-container>。

我们可以得出一个简化的观察结论。由于<mjx-container classname="MathJax" jax="SVG"></mjx-container>垂直于<mjx-container classname="MathJax" jax="SVG"></mjx-container>的法向量，从<mjx-container classname="MathJax" jax="SVG"></mjx-container>的尖端到<mjx-container classname="MathJax" jax="SVG"></mjx-container>沿着方向向量<mjx-container classname="MathJax" jax="SVG"></mjx-container>的距离与从原点到<mjx-container classname="MathJax" jax="SVG"></mjx-container>沿着相同方向的距离相同。

考虑到向量<mjx-container classname="MathJax" jax="SVG"></mjx-container>，其中<mjx-container classname="MathJax" jax="SVG"></mjx-container>和<mjx-container classname="MathJax" jax="SVG"></mjx-container>是<mjx-container classname="MathJax" jax="SVG"></mjx-container>的法向量和距离。

如果<mjx-container classname="MathJax" jax="SVG"></mjx-container>与<mjx-container classname="MathJax" jax="SVG"></mjx-container>平行，则<mjx-container classname="MathJax" jax="SVG"></mjx-container>将是我们所需的缩放因子，但让我们看看<mjx-container classname="MathJax" jax="SVG"></mjx-container>会发生什么：

随着<mjx-container classname="MathJax" jax="SVG"></mjx-container>和<mjx-container classname="MathJax" jax="SVG"></mjx-container>变得不那么平行，<mjx-container classname="MathJax" jax="SVG"></mjx-container>变得越来越短。

还要注意的一点是，即使<mjx-container classname="MathJax" jax="SVG"></mjx-container>和<mjx-container classname="MathJax" jax="SVG"></mjx-container>完全平行，由于<mjx-container classname="MathJax" jax="SVG"></mjx-container>不是单位向量，<mjx-container classname="MathJax" jax="SVG"></mjx-container>仍然太短了。如果我们在乘以<mjx-container classname="MathJax" jax="SVG"></mjx-container>之前将其归一化，这个问题就解决了。

但我们在这里过早地——我们不需要对<mjx-container classname="MathJax" jax="SVG"></mjx-container>进行归一化。让我们重新审视一下<mjx-container classname="MathJax" jax="SVG"></mjx-container>的定义：

<mjx-container classname="MathJax" jax="SVG"></mjx-container>

将<mjx-container classname="MathJax" jax="SVG"></mjx-container>定义为<mjx-container classname="MathJax" jax="SVG"></mjx-container>之后，我们可以简化为

<mjx-container classname="MathJax" jax="SVG"></mjx-container>

我之前提到过，我们可以将<mjx-container classname="MathJax" jax="SVG"></mjx-container>看作是<mjx-container classname="MathJax" jax="SVG"></mjx-container>与<mjx-container classname="MathJax" jax="SVG"></mjx-container>的正交正面的测量标准。这是正确的，但不是全部的真相！

由于点积是其组成向量的大小的倍数，<mjx-container classname="MathJax" jax="SVG"></mjx-container>还编码了<mjx-container classname="MathJax" jax="SVG"></mjx-container>的大小。因此，通过<mjx-container classname="MathJax" jax="SVG"></mjx-container>乘以<mjx-container classname="MathJax" jax="SVG"></mjx-container>做两件事：

1.  它将<mjx-container classname="MathJax" jax="SVG"></mjx-container>归一化，并且

1.  它增加了<mjx-container classname="MathJax" jax="SVG"></mjx-container>的长度，因为它与<mjx-container classname="MathJax" jax="SVG"></mjx-container>变得不那么平行。

因此，<mjx-container classname="MathJax" jax="SVG"></mjx-container>既是我们所需的<mjx-container classname="MathJax" jax="SVG"></mjx-container>的缩放因子，也是<mjx-container classname="MathJax" jax="SVG"></mjx-container>：

我们找到了解决方案！让我们快速回顾一下。

我们将<mjx-container classname="MathJax" jax="SVG"></mjx-container>定义为：

<mjx-container classname="MathJax" jax="SVG"></mjx-container>

我们将重新定义<mjx-container classname="MathJax" jax="SVG"></mjx-container>，以包含<mjx-container classname="MathJax" jax="SVG"></mjx-container>：

<mjx-container classname="MathJax" jax="SVG"></mjx-container>

我们的分母<mjx-container classname="MathJax" jax="SVG"></mjx-container>仍然定义为：

<mjx-container classname="MathJax" jax="SVG"></mjx-container>

有了这个，我们通过将<mjx-container classname="MathJax" jax="SVG"></mjx-container>和<mjx-container classname="MathJax" jax="SVG"></mjx-container>相加并按<mjx-container classname="MathJax" jax="SVG"></mjx-container>缩放，找到了我们的交点：

<mjx-container classname="MathJax" jax="SVG"></mjx-container>

完全展开后变成：

<mjx-container classname="MathJax" jax="SVG"></mjx-container>

将此放入代码中，我们得到：

```
Vector3  ThreePlaneIntersection(Plane P1,  Plane P2,  Plane P3)  {  Vector3 dir = Vector3.Cross(P2.normal, P3.normal);  float denom = Vector3.Dot(u);  if  (Mathf.Abs(denom)  < EPSILON)  {  return  null;    }  Vector3 a = P2.normal * P3.distance;  Vector3 b = P3.normal * P2.distance;  Vector3 V = Vector3.Cross(P1.normal, a - b);  Vector3 U = dir * P1.distance;  return  (V + U)  / denom;}
```

## 分手的话

谢谢阅读！

在编写和构建本文可视化过程中，花费了大量的时间，希望它达到了帮助您建立平面直观心理模型的目标。

非常感谢[Gunnlaugur Þór Briem](https://www.linkedin.com/in/gunnlaugur-briem/)和[Eiríkur Fannar Torfason](https://eirikur.dev/)在这篇文章中提供的宝贵反馈。我在[GRID](https://grid.is/)与他们合作过；他们是非常棒的人，与他们一起工作和相处是一种享受。

— Alex Harri

PS：如果你有兴趣了解本文中可视化的构建方式，请查看这个网站在[Github上的开源](https://github.com/alexharri/website)。

## 进一步阅读

我强烈推荐阅读[克里斯特·埃里克森的《实时碰撞检测》](https://www.amazon.com/Real-Time-Collision-Detection-Interactive-Technology/dp/1558607323)。如果你在构建使用3D几何的应用程序，它将证明是一个非常有用的资源。如果没有这本书——特别是关于平面相交的两章——这篇文章不会存在。

我最近分析了[Arkio](https://www.arkio.is/)中的编辑性能，并注意到在重新计算几何时，解决三平面相交的方法占用了大约一半的总计算时间。通过实施书中描述的更有效的三平面相交方法，我们使这个方法**快了约500%**，将Arkio的编辑性能提高了**超过1.6倍**。疯狂的事情！

我开始写这篇文章是为了理解三平面相交方法的工作原理。然而，我觉得读者需要对平面有更好的基础和理解，才能使这篇文章有价值。在建立这个基础过程中，这篇文章比我预期的长了很多。

无论如何，这是一本很棒的书。[去看看吧](https://www.amazon.com/Real-Time-Collision-Detection-Interactive-Technology/dp/1558607323)!

</main>
