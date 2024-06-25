<!--yml

类别：未分类

日期：2024-05-27 15:19:23

-->

# 12 现代 CSS 一行升级 | 现代 CSS 解决方案

> 来源：[https://moderncss.dev/12-modern-css-one-line-upgrades/](https://moderncss.dev/12-modern-css-one-line-upgrades/)

有时，改进应用的 CSS 只需要一行升级或增强！了解 12 种属性，开始将它们融入到您的项目中，享受减少技术债务、删除 JavaScript 并为用户体验赢得轻松胜利。

属性被探索以下类别：

+   **[稳定的升级](#stable-upgrades)**：通过替换较旧的技术修复 hack 或问题

+   **[稳定的增强](#stable-enhancements)**：通过良好支持的现代属性提供改进的体验

+   **[渐进式增强](#progressive-enhancements)**：在支持这些属性时提供升级的体验，而不会在不支持的浏览器中造成损害

以下受到良好支持的属性可以通过替换较旧的技术来修复 hack 或长期存在的问题。

你曾经使用过 “[padding hack](https://css-tricks.com/aspect-ratio-boxes/)” 来强制视频嵌入的宽高比，比如 16:9 吗？截至 2021 年 9 月，`aspect-ratio` 属性在常青浏览器中是稳定的，是唯一需要定义宽高比的属性。

对于高清视频，您只需使用 `aspect-ratio: 16/9`。对于完美的正方形，只需要 `aspect-ratio: 1`，因为隐含的第二个值也是 `1`。

<details open=""><summary>用于“基本使用 aspect-ratio”的 CSS</summary>

```
.aspect-ratio-hd {
  aspect-ratio: 16/9;
}

.aspect-ratio-square {
  aspect-ratio: 1;
} 
```</details>

值得注意的是，应用了 `aspect-ratio` 是宽容的，并且将允许内容优先。这意味着当内容会导致元素在至少一个维度上超过比例时，元素仍然会增长或改变形状以容纳内容。为了防止或控制此行为，您可以添加额外的尺寸属性，例如 `max-width`，这可能是必要的，以避免在 flex 或 grid 容器中扩展。

<details open=""><summary>用于“宽容的 aspect-ratio”的 CSS</summary>

```
 .aspect-ratio-square {
  aspect-ratio: 1;
} 
```</details>

Lorem ipsum dolor sit amet consectetur adipisicing elit. Iste molestias maiores velit quaerat debitis incidunt delectus nulla, quibusdam fugit eos? Fugiat asperiores assumenda nulla corrupti, sit repellendus ducimus necessitatibus voluptates.

Lorem ipsum dolor sit amet consectetur adipisicing elit.

Iste molestias maiores velit quaerat debitis incidunt delectus nulla, quibusdam fugit eos?

如果您不愿完全替换填充 hack 并仍想提供一些尺寸限制，可以查看 [Smol Aspect Ratio Gallery](https://smolcss.dev/#smol-aspect-ratio-gallery) 了解 `aspect-ratio` 的渐进增强解决方案。

这实际上是列表中最古老的属性，但它解决了一个重要问题，并且绝对符合一行升级的情感。

使用 `object-fit` 会导致 `img` 或其他[替换元素](https://developer.mozilla.org/en-US/docs/Web/CSS/Replaced_element)作为其内容的容器，并使这些内容采用类似于 `background-size` 的调整大小行为。

虽然 `object-fit` 有几个可用的值，但以下是你最有可能使用的：

+   `cover` - 图像调整大小以*覆盖*元素，并保持其纵横比，以使内容不变形

+   `scale-down` - 图像在元素内（如有需要）调整大小，以便完全可见而不被裁剪，并保持其纵横比，这可能会导致图像周围出现额外的空间（"letterboxing"）如果元素具有不同的渲染纵横比

无论哪种情况，`object-fit` 都是与 `aspect-ratio` 配对的绝佳属性，以确保在应用自定义纵横比时图像不会变形。

<details open=""><summary>"使用 object-fit 和 aspect-ratio" 的 CSS</summary>

```
.image {
  object-fit: cover;
  aspect-ratio: 1;

  max-block-size: 250px;
} 
```</details>

> 在这个免费的 egghead 视频课程中，查看我的[对 object-fit 的解释](https://egghead.io/lessons/css-apply-aspect-ratio-sizing-to-images-with-css-object-fit?af=2s65ms)。

作为众多逻辑属性之一，`margin-inline` 作为设置内联（在水平写入模式中为左右）边距的简写。

这里的替换很简单：

```
 margin-left: auto;
margin-right: auto;

margin-inline: auto;
```

逻辑属性已经可用了几年，现在支持率已经达到[98%以上](https://caniuse.com/css-logical-props)（偶尔需要前缀）。阅读 Ahmad Shadeed 的这篇文章，了解更多关于[使用逻辑属性](https://ishadeed.com/article/css-logical-properties/)及其在面向国际观众的网站中的重要性。

这些得到很好支持的现代 CSS 属性可以提供更好的体验，并且还可以替换旧的方法甚至是JavaScript辅助的解决方案。不太可能需要回退解决方案，尽管这取决于你特定的应用考虑因素，始终鼓励进行测试。

使用 `text-underline-offset` 属性允许你控制文本基线和下划线之间的距离。这个属性已经成为我的标准重置的一部分，应用如下：

```
a:not([class]) {
	text-underline-offset: 0.25em;
}
```

你可以使用这个偏移量来清除下降部分，并（主观地）提高可读性，特别是当链接紧密组合在一起时，比如链接的项目列表。

这个升级可能会取代旧的黑客方法，比如边框或伪元素，甚至是渐变背景，特别是在与其朋友一起使用时：

你是否一直在使用 `box-shadow` 或者一个伪元素来在焦点时提供自定义轮廓与元素之间的距离？

好消息！长期以来可用的 `outline-offset` 属性（[早在2006年](https://caniuse.com/?search=outline-offset)！）可能是你错过的一个，它允许用正值将轮廓推离元素或用负值将轮廓拉入元素内。

在演示中，灰色实线是元素边框，蓝色虚线是通过`outline-offset`定位的轮廓。

<details open=""><summary>"正偏移和负偏移的 CSS"</summary>

```
.outline-offset {
  outline: 2px dashed blue;
  outline-offset: var(--outline-offset, .5em);
} 
```</details>

正偏移 负偏移

**提醒**：轮廓*不*计算为元素的框大小的一部分，因此增加距离不会增加元素占用的空间量。这类似于`box-shadow`也不会影响元素大小的渲染方式。

> 了解更多关于[将 outline-offset 用作可访问性改进](https://moderncss.dev/modern-css-upgrades-to-improve-accessibility/#focus-visibility)的信息。

`scroll-margin`属性集（及相应的`scroll-padding`）允许在滚动位置的上下文中添加元素的偏移量。换句话说，添加`scroll-padding-top`可以增加元素上方的滚动偏移量，但不会影响其在文档中的布局位置。

为什么这很有用？嗯，当粘性导航元素覆盖内容时激活锚链接时，可以减轻问题。使用`scroll-margin-top`，我们可以增加元素上方的空间，以便通过导航滚动到元素时，考虑到粘性导航占用的空间。

我喜欢在我的重置中为任何带有`[id]`属性的元素包含一个通用的起始规则，因为它有可能成为锚链接。

```
[id] {
	scroll-margin-top: 2rem;
}
```

在[基于组件的架构](https://moderncss.dev/modern-css-for-dynamic-component-based-architecture/#css-reset-additions)的现代 CSS 文章中探讨了另一种选择器，并且也在本站使用，在文章目录侧边栏中的链接可以进行测试。

当考虑到粘性、固定或绝对定位元素的重叠时，你可能想使用带有回退值的自定义属性来获得更可靠的解决方案。然后，借助 JavaScript 的帮助，测量真实需要的距离并[更新自定义属性值](https://12daysofweb.dev/2021/css-custom-properties/#accessing-and-setting-custom-properties-with-javascript)。

```
[id] {

	scroll-margin-top: var(--scroll-margin, 2rem);
}
```

我鼓励你进一步更新此解决方案，并使用逻辑属性的等效项：`scroll-padding-block-start`和`-block-end`。

你可能熟悉`prefers-color-scheme`媒体查询以自定义深色和浅色主题。CSS 属性`color-scheme`是一种选择性适应浏览器 UI 元素，包括表单控件、滚动条和 CSS 系统颜色。适应请求浏览器以`light`或`dark`方案呈现这些项目，该属性允许定义首选顺序。

如果你正在启用整个应用的自适应功能，请在`:root`上设置以下内容，这表示优先选择`dark`主题（或者反转顺序以优先选择`light`主题）。

```
:root {
	color-scheme: dark light;
}
```

你也可以在单个元素上定义`color-scheme`，例如调整带有暗色背景的元素中的表单控件，以获得更好的对比度。

```
.dark-background {
	color-scheme: dark;
}
```

从萨拉·乔伊的演示中了解如何[使用 color-scheme 实现简单的深色模式](https://www.youtube.com/watch?v=Lye56NHGtLA)，以及更多关于集成此功能的信息。

如果您曾经想要更改复选框或单选按钮的颜色，您一定在寻找 `accent-color`。使用此属性，您可以修改单选按钮和复选框的 `:checked` 外观，以及进度元素和范围输入的填充状态。如果您没有其他覆盖，则还可以调整浏览器的默认焦点“光环”。

<details open=""><summary>“使用 accent-color 的效果” CSS</summary>

```
:root {
  accent-color: mediumvioletred;
} 
```</details>

考虑将 `accent-color` 和 `color-scheme` 都添加到您的基线应用程序样式中，以快速实现自定义主题管理的胜利。

> 如果您需要更全面的自定义样式来控制表单控件，请查看以[单选按钮](https://moderncss.dev/pure-css-custom-styled-radio-buttons/)开头的现代 CSS 系列。

我最喜欢的 CSS 隐藏宝石之一是使用 `fit-content` 将元素“包裹”到其内容中。

尽管您可能已经使用了内联显示值，如 `display: inline-block` 来将元素的宽度减小到内容尺寸，但升级到 `width: fit-content` 将实现相同的效果。`width: fit-content` 的优势在于它保留了 `display` 值，因此除非您也调整了位置，否则不会更改布局中元素的位置。计算的盒子尺寸将根据 `fit-content` 创建的尺寸调整。

<details open=""><summary>“基本使用 fit-content”的 CSS</summary>

```
.fit-content {
  width: fit-content;
} 
```</details>

使用 fit-content

没有使用 fit-content

`fit-content` 值是[启用内在尺寸的关键字之一](https://moderncss.dev/contextual-spacing-for-intrinsic-web-design/)。

考虑将此技术升级为逻辑属性等效的 `inline-size: fit-content`。

## **渐进增强**

[#](#progressive-enhancements)

当这些属性得到支持并且可以在不支持的浏览器中使用时，最后一组属性提供了升级的体验。这意味着即使它们是现代 CSS 的最新添加，也不需要回退方法。

包含滚动区域的默认行为——即具有有限尺寸的区域，允许溢出内容滚动——是，当滚动超出元素范围时，滚动交互会传递到背景页面。对于您的用户来说，这可能是最好的情况，最糟糕的情况则可能令人沮丧。

使用 `overscroll-behavior: contain` 将滚动隔离到包含的区域，防止在滚动边界达到时将滚动移动到父页面。这在诸如导航链接的边栏之类的情况下非常有用，该边栏可能与主页面内容具有独立的滚动，该内容可能是一篇长文章或文档页面。

<details open=""><summary>CSS for "基本使用方式的 overscroll-behavior"</summary>

```
.sidebar, .article {
  overscroll-behavior: contain;
} 
```</details>

Lorem ipsum dolor sit amet consectetur adipisicing elit. Maxime nobis consectetur earum!

Aliquid a praesentium quis in consequuntur mollitia laboriosam illum nemo commodi aut?

Doloremque sapiente quos dignissimos sequi cupiditate commodi nemo non perspiciatis placeat totam.

Reiciendis, at nulla! Hic nemo eius atque laborum consequuntur iusto exercitationem quasi.

Est, assumenda amet culpa veritatis maxime debitis? Suscipit error amet quas sed?

Magnam exercitationem neque error deleniti consequuntur, dolor repellat quo perferendis dicta sunt.

Ipsum id repellat velit laudantium vel autem eos non aperiam qui nobis?

Ratione maxime neque numquam minima, omnis dolorem temporibus laboriosam dolor atque suscipit?

Dolores assumenda dolore similique eaque, odio voluptatibus. Aliquid iusto nostrum iste? Exercitationem.

Quos adipisci ea ullam amet blanditiis voluptatibus, fuga laborum sed facere quasi!

Odio rerum labore veritatis esse eaque quae debitis possimus ea omnis vitae.

Quasi eum obcaecati laborum eaque nostrum numquam tempore reprehenderit qui beatae debitis?

Optio atque fugit quia reprehenderit dolor rem et delectus praesentium ratione provident.

Sed accusamus at architecto dolore minima error, assumenda amet nam perferendis odit?

Et eius enim est hic doloribus reiciendis qui cupiditate? Autem, iure cupiditate?

最新的属性之一（截至 2023 年）是 `text-wrap`，它有两个值可解决不平衡行的排版问题。这包括防止“孤立词”，这描述了最后一行中孤立的单词。

第一个可用的值是 `balance`，其目标是平衡每行文字的字符数。

文本包装最多只能有六行，因此该技术最适用于标题或其他较短的文本段落。限制应用范围还有助于限制对页面呈现速度的影响。

<details open=""><summary>CSS for "应用 text-wrap: balance"</summary>

```
:is(h1, h2, h3, h4, .text-balance) {
  text-wrap: balance;

  max-inline-size: 25ch;
} 
```</details>

此文本已通过 text-wrap 进行平衡

此文本未经 text-wrap 平衡处理

`pretty` 的另一个值专门解决了孤立词汇的问题，并且可以更广泛地应用。`pretty` 背后的算法将评估文本块中的最后四行，以确保必要时进行调整，以确保最后一行有两个或更多单词。

<details open=""><summary>CSS for "应用 text-wrap: balance"</summary>

```
p {
  text-wrap: pretty;

  max-inline-size: 35ch;
} 
```</details>

**使用 text-wrap: pretty**

Lorem ipsum dolor sit amet consectetur adipisicing elit. Dolore cupiditate aliquid, facere explicabo voluptatibus iure! Saepe nostrum quasi corporis totam accusamus beatae obcaecati rerum fugiat minima perferendis voluptatibus.

**Without**

Lorem ipsum dolor sit amet consectetur adipisicing elit. Dolore cupiditate aliquid, facere explicabo voluptatibus iure! Saepe nostrum quasi corporis totam accusamus beatae obcaecati rerum fugiat minima perferendis voluptatibus.

使用`text-wrap`是一个很好的渐进增强方法。但是，任何调整都不会改变元素的计算宽度，因此在某些布局中的副作用可能是在文本旁边增加不需要的空间。

在某些情况下，滚动条的出现或消失可能导致不希望的布局移位。例如，当显示对话框叠加层时，背景页面添加`overflow: hidden`以防止滚动，导致移除不再需要的滚动条时发生移位。

现代 CSS 属性`scrollbar-gutter`使布局中保留了滚动条的空间，从而防止了不希望的移位。当不需要滚动条时，浏览器仍会绘制一个额外空间作为滚动容器上的填充之外的额外空间。

> **重要提示**：仅当用户系统设置为*不*使用“叠加”滚动条时，此属性才会生效，如默认情况下的 macOS，滚动条只会在正在主动滚动的内容上显示为叠加层。不要为了使用`scrollbar-gutter`而放弃填充，因为在使用叠加滚动条时，您将失去所有预期的空间。

由于这是明显的额外空间，您可以选择使用两个关键字来分配此属性：`scrollbar-gutter: stable both-edges`。仅使用`stable`会在滚动条位置添加间隙，而添加`both-edges`则会将空间添加到滚动容器的另一侧。这样可以确保在布局尚未需要滚动条可见时保持视觉平衡。请参阅 [使用 scrollbar-gutter 的可视化](https://defensivecss.dev/tip/scrollbar-gutter/)来自 Ahmad Shadeed。

* * *

一定要查看这里的更多文章，以进一步提升您的 CSS 知识！一个很好的起点是[主题列表。](https://moderncss.dev/topics/)
