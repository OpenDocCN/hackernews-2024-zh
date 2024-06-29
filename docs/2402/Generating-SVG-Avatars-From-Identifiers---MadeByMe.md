<!--yml

类别：未分类

日期：2024-05-29 13:23:02

-->

# 从标识符生成SVG头像 - MadeByMe

> 来源：[https://madebyme.today/projects/generating-svg-avatars-from-identifiers/](https://madebyme.today/projects/generating-svg-avatars-from-identifiers/)

<main class="main">

当您在互联网上创建一个新的社交服务时，您需要考虑其中一件事，即如何使用户的空间感觉像自己的。像[GitHub](https://github.blog/2013-08-14-identicons/)、Roll20或[Reddit](https://www.reddit.com/r/reddit/comments/vtkmni/introducing_collectible_avatars/)这样的服务会生成 —— 或允许您生成 —— 自定义头像（即[Identicons](https://en.wikipedia.org/wiki/Identicon)）供您的帐户使用。自动生成头像尤其不错，因为它不需要用户的任何参与。让我们试着自己解决这个问题。

我认为这篇博文是那种先看最终结果会更容易理解的事物之一。因此，对于有兴趣的人，这里有个剧透。

这是最终结果：

标识符“Foo”的头像。

头像有24个部分，由24个[`<path>`](https://www.w3.org/TR/SVG2/paths.html#PathElement)标签组成。描述路径的坐标是静态的，但它们的颜色是根据标识符的[SHA-256](https://en.wikipedia.org/wiki/SHA-2)生成的。

```
[1](#hl-0-1)SHA256("Foo") == [ 44,  38, 180, 107, 104, 255, 198, 143, [2](#hl-0-2) 249, 155,  69,  60,  29,  48,  65,  52, [3](#hl-0-3) 19,  66,  45, 112, 100, 131, 191, 160, [4](#hl-0-4) 249, 138,  94, 136,  98, 102, 231, 174] [5](#hl-0-5) [6](#hl-0-6)Base64(SHA256("Foo")) == LCa0a2j/xo/5m0U8HTBBNBNCLXBkg7+g+YpeiGJm564= 
```

本博文的其余部分侧重于将字节数组翻译成一个看起来*不错*的头像。

## [#](#svg-scaffolding) SVG结构

让我们考虑如何准备SVG的结构。我们知道每个部分必须是自己的[`<path>`](https://www.w3.org/TR/SVG2/paths.html#PathElement)元素，因此我们可以用[fill](https://www.w3.org/TR/SVG2/painting.html#SpecifyingFillPaint)属性分别对它们上色。生成环形扇区可能具有挑战性 —— 这需要使用两个(!)[arcs](https://www.w3.org/TR/SVG2/paths.html#PathDataEllipticalArcCommands)，这实在太多了 —— 相反，我们可以稍微作弊，创建重叠的圆形扇区。这将使路径更简单，只需要一个(!)arc。

从`<svg>`标签开始：

```
[1](#hl-0-1)<svg viewBox="-1 -1 2 2" xmlns="http://www.w3.org/2000/svg"> [2](#hl-0-2) <!-- ... --> [3](#hl-0-3)</svg> 
```

如果“圆圈”的中心位于$(0, 0)$，则路径生成会变得更加容易。我们可以通过设置`viewBox`属性来实现：左上角为$(-1, -1)$，宽度为$2$，高度为$2$，如上所示。

好了。现在我们需要创建圆形扇区。SVG有一些用于构建[基本形状](https://developer.mozilla.org/en-US/docs/Web/SVG/Tutorial/Basic_Shapes)（如圆圈、矩形或椭圆）的标签。可惜的是，披萨片并不被认为是“基本”的，所以这里没有`<pizza-slice>`来帮助我们。相反，我们需要使用[`<path>`](https://www.w3.org/TR/SVG2/paths.html#PathElement)标签。

### [#](#making-pizza-slices-pizza) 制作披萨片 🍕

`<path>` 标签很好，因为它允许您构建任何复杂的形状。要使其工作，我们需要使用其特殊的 [路径数据](https://www.w3.org/TR/SVG2/paths.html#PathData) 语法来描述我们希望形状看起来的方式。它非常复杂，但我们只需要掌握其 4 个命令：

+   [绝对 **“moveto”**](https://www.w3.org/TR/SVG2/paths.html#PathDataMovetoCommands) 指令 — 用于设定新的起始点，

+   [绝对 **“lineto”**](https://www.w3.org/TR/SVG2/paths.html#PathDataLinetoCommands) 指令 — 用于从当前点画一条到给定 $(x, y)$ 的直线，

+   [绝对椭圆弧曲线](https://www.w3.org/TR/SVG2/paths.html#PathDataEllipticalArcCommands) 指令 — 用于从当前点画一条到给定 $(x, y)$ 的椭圆弧，

+   [**“closepath”**](https://www.w3.org/TR/SVG2/paths.html#PathDataClosePathCommand) 指令 — 用于通过将当前路径连接回其起始点来结束当前路径。

为了简单起见，让我们考虑一个有 4 个披萨块的例子：

```
[1](#hl-1-1)<svg overflow="visible" viewBox="-1 -1 2 2" xmlns="http://www.w3.org/2000/svg"> [2](#hl-1-2) <path d="M 0,0 L 0,-1 A 1,1,0,0,1,1,0 z" fill="IndianRed" stroke="black" stroke-width="0.01" /> [3](#hl-1-3) <path d="M 0,0 L 1,0 A 1,1,0,0,1,0,1 z" fill="LightCoral" stroke="black" stroke-width="0.01" /> [4](#hl-1-4) <path d="M 0,0 L 0,1 A 1,1,0,0,1,-1,0 z" fill="Salmon" stroke="black" stroke-width="0.01" /> [5](#hl-1-5) <path d="M 0,0 L -1,0 A 1,1,0,0,1,0,-1 z" fill="DarkSalmon" stroke="black" stroke-width="0.01" /> [6](#hl-1-6)</svg> 
```

它呈现为：

$ r_1 = 1 $

###### *注意*

*我在 SVG 标签中添加了 `overflow="visible"`，因为带有非零描边的图像*略大*于其 `viewBox`，并且描边在 $(-1, 0)$，$(1, 0)$，$(0, 1)$ 和 $(-1, 0)$ 处被截断。*

*我们可以看到这里有一个有趣的模式：[**“lineto”**](https://www.w3.org/TR/SVG2/paths.html#PathDataLinetoCommands) 指令的坐标与*前一个*元素的 [椭圆弧曲线](https://www.w3.org/TR/SVG2/paths.html#PathDataEllipticalArcCommands) 指令的坐标相同。*

**好的，但是如何将这些块划分成两半呢？* — 你可能会问。*

*为了做到这一点，我们需要找到弧线中点的点。*

*<svg xmlns="http://www.w3.org/2000/svg" id="slices-with-points" overflow="visible" viewBox="-1 -1 2 2"><text x=".70710678118" y="-.70710678118" text-anchor="start" dominant-baseline="text-top">(ax, ay)</text><text x=".70710678118" y=".70710678118" text-anchor="start" dominant-baseline="hanging">(bx, by)</text><text x="-.70710678118" y="-.70710678118" text-anchor="end" dominant-baseline="text-top">(cx, cy)</text><text x="-.70710678118" y=".70710678118" text-anchor="end" dominant-baseline="hanging">(dx, dy)</text></svg>*

*$ r_1 = 1 $*

*幸运的是 — 由于圆的中心点是 $(0, 0)$，弧线点位于其弧线中点 — 我们知道它们的坐标的绝对值是相同的。因此，我们的图表简化为：*

*<svg xmlns="http://www.w3.org/2000/svg" id="slices-with-same-points" overflow="visible" viewBox="-1 -1 2 2"><text x=".70710678118" y="-.70710678118" text-anchor="start" dominant-baseline="text-top">(a, -a)</text><text x=".70710678118" y=".70710678118" text-anchor="start" dominant-baseline="hanging">(a, a)</text><text x="-.70710678118" y="-.70710678118" text-anchor="end" dominant-baseline="text-top">(-a, -a)</text><text x="-.70710678118" y=".70710678118" text-anchor="end" dominant-baseline="hanging">(-a, a)</text></svg>*

*$ r_1 = 1 $*

*太好了。 👌*

*现在我们可以使用圆的方程找到中间点。*

*$ \begin{aligned} (x - x_0)^2 + (y - y_0)^2 &= r^2 \\[0.25em] (a - 0)^2 + (a - 0)^2 &= 1^2 \\[0.25em] 2a^2 &= 1 \\[0.25em] a^2 &= \frac{1}{2} \\[0.25em] a &= \sqrt{\frac{1}{2}} \end{aligned} $*

*再加四片披萨我们得到：*

*$ r_1 = 1 $*

### *[#](#layers-of-pizza) 披萨的层次*

*下一步是创建额外（更小）的圆扇形层。假设我们想要三个圆；选择分割点有两种明显的方法：等半径或等面积。等半径比较简单，所以我们先试试这个。*

*如果最外面的圆半径为$1$，那么中间的圆将为$0.\overline{6}$，最小的将为$0.\overline{3}$。*

*$ r_1 = 1 \quad\text{和}\quad r_2 = 0.\overline{6} \quad\text{和}\quad r_3 = 0.\overline{3} $*

*虽然这并不重要，但我不太喜欢最小的圆太小了。但在决定之前，我们需要先看看等面积的圆。*

*$ \begin{aligned} \Pi r_1^2 - \Pi r_2^2 = \Pi r_2^2 - \Pi r_3^2 \quad&\text{和}\quad \Pi r_2^2 - \Pi r_3^2 = \Pi r_3^2 \\[0.25em] 1 - r_2^2 = r_2^2 - r_3^2 \quad&\text{和}\quad r_2^2 - r_3^2 = r_3^2 \\[0.25em] 1 = 2r_2^2 - r_3^2 \quad&\text{和}\quad r_2^2 = 2r_3^2 \\[0.25em] 1 = 3r_3^2 \quad&\text{和}\quad r_2^2 = 2r_3^2 \\[0.25em] r_3 = \sqrt{\frac{1}{3}} \quad&\text{和}\quad r_2 = \sqrt{\frac{2}{3}} \end{aligned} $*

*这给了我们这个头像：*

*$ r_1 = 1 \quad\text{和}\quad r_2 = \sqrt{\frac{2}{3}} \quad\text{和}\quad r_3 = \sqrt{\frac{1}{3}} $*

*这也不理想，但反过来说&mldr; 我稍微调整了半径，我觉得我找到了一个[折衷方案](https://en.wikipedia.org/wiki/Arithmetic_mean)，我感到很满意。*

*$ r_1 = 1 \quad\text{和}\quad r_2 = \frac{0.\overline{6} + \sqrt{\frac{2}{3}}}{2} \quad\text{和}\quad r_3 = \frac{0.\overline{3} + \sqrt{\frac{1}{3}}}{2} $*

*以上圆中的半径是等半径变体和等面积变体的算术平均值。*

## *[#](#working-with-colors) 使用颜色*

*完成SVG结构后，我们可以专注于根据标识符的哈希选择准随机颜色来填充圆形段。[SHA-256](https://zh.wikipedia.org/wiki/SHA-2)哈希有32字节；这比我们需要的多 —— 假设我们需要每个圆形扇区一个字节 —— 这给我们带来另一个好处：如果需要，我们可以插入第4个圆。*

*不过，现在让我们来谈谈颜色。如果我们使用[HSL](https://zh.wikipedia.org/wiki/HSL%E5%92%8CHSV)模型，我们可以将每个字节分成4个部分：4位用于色调（因为它是最显著的），2位用于饱和度，2位用于亮度。路径的[fill](https://www.w3.org/TR/SVG2/painting.html#SpecifyingFillPaint)属性接受任何[paint](https://www.w3.org/TR/SVG2/painting.html#SpecifyingPaint)值，这基本上意味着我们需要将我们的颜色格式化为标准CSS的色调。*

```
 *`[1](#hl-2-1)fn to_color(byte: u8)  -> String { [2](#hl-2-2) let  h  =  byte  >>  4; [3](#hl-2-3) let  s  =  (byte  >>  2)  &  0x03; [4](#hl-2-4) let  l  =  byte  &  0x03; [5](#hl-2-5) [6](#hl-2-6) let  normalized_h  =  (h  as  f32)  /  16.0; [7](#hl-2-7) let  normalized_s  =  (s  as  f32)  /  4.0; [8](#hl-2-8) let  normalized_l  =  (l  as  f32)  /  4.0; [9](#hl-2-9) [10](#hl-2-10) let  h  =  360.0  *  normalized_h; [11](#hl-2-11) let  s  =  20.0  +  80.0  *  normalized_s; [12](#hl-2-12) let  l  =  40.0  +  50.0  *  normalized_l; [13](#hl-2-13) [14](#hl-2-14) format!("hsl({h}, {s}%, {l}%)") [15](#hl-2-15)}`* 
```

*好了，让我们为`"foo"`生成一个头像，看看它是什么样子。*

*"foo"的头像。*

*嗯，它看起来并不可怕，但也不太吸引人。说实话，我们应该预料到这样的结果；[SHA-256](https://zh.wikipedia.org/wiki/SHA-2)返回（以及其他所有哈希函数）准随机值。如果我们将这些*原始*字节转换为填充颜色，我们将得到准随机的颜色。为了使它更好看，我们需要稍微修改我们的算法；我们需要计算一个全局主题，如果可以的话，为头像的部分圆环颜色。我们可以通过将所有哈希字节折叠成一个单一字节来实现这一点。*

```
 *`[1](#hl-3-1)fn to_color(normalized_theme: f32,  byte: u8)  -> String { [2](#hl-3-2) let  h  =  byte  >>  4; [3](#hl-3-3) let  s  =  (byte  >>  2)  &  0x03; [4](#hl-3-4) let  l  =  byte  &  0x03; [5](#hl-3-5) [6](#hl-3-6) let  normalized_h  =  (h  as  f32)  /  16.0; [7](#hl-3-7) let  normalized_s  =  (s  as  f32)  /  4.0; [8](#hl-3-8) let  normalized_l  =  (l  as  f32)  /  4.0; [9](#hl-3-9) [10](#hl-3-10) let  h  =  360.0  *  normalized_theme  +  120.0  *  normalized_h; [11](#hl-3-11) let  s  =  20.0  +  80.0  *  normalized_s; [12](#hl-3-12) let  l  =  40.0  +  50.0  *  normalized_l; [13](#hl-3-13) [14](#hl-3-14) format!("hsl({h}, {s}%, {l}%)") [15](#hl-3-15)} [16](#hl-3-16) [17](#hl-3-17)fn calculate_theme(bytes: &[u8])  -> f32 { [18](#hl-3-18) let  theme  =  bytes.iter() [19](#hl-3-19) .fold(0u8,  |acc,  byte|  acc  ^  byte); [20](#hl-3-20) (theme  as  f32)  /  (u8::MAX  as  f32) [21](#hl-3-21)} [22](#hl-3-22) [23](#hl-3-23)fn generate_paths(hash: [u8;  32])  { [24](#hl-3-24) let  theme  =  calculate_theme(&hash); [25](#hl-3-25) [26](#hl-3-26) let  colors  =  hash.iter() [27](#hl-3-27) .cloned() [28](#hl-3-28) .map(|byte|  to_color(theme,  byte)) [29](#hl-3-29) .collect::<Vec<_>>(); [30](#hl-3-30) [31](#hl-3-31) unimplemented!("Generate SVG paths."); [32](#hl-3-32)}`* 
```

*这段代码渲染如下：*

*"foo"的带有主题的头像。*

*我觉得它看起来相当不错。这个解决方案按照预期运行：它*很可能*为不同的标识符生成不同的头像（[哈希碰撞](https://zh.wikipedia.org/wiki/%E5%93%88%E5%B8%8C%E7%A2%B0%E6%92%9E)确实会发生），它们看起来是可以接受的。然而，这些环似乎*相位*进入彼此 —— 它们的颜色太相似了。我们可以尝试另一个主题：一个环主题 —— 它将是代表单个环的所有字节的XOR折叠 —— 并检查结果。*

```
 *`[1](#hl-4-1)fn to_color(normalized_theme: f32,  normalized_ring_theme: f32,  byte: u8)  -> String { [2](#hl-4-2) let  h  =  byte  >>  4; [3](#hl-4-3) let  s  =  (byte  >>  2)  &  0x03; [4](#hl-4-4) let  l  =  byte  &  0x03; [5](#hl-4-5) [6](#hl-4-6) let  normalized_h  =  (h  as  f32)  /  16.0; [7](#hl-4-7) let  normalized_s  =  (s  as  f32)  /  4.0; [8](#hl-4-8) let  normalized_l  =  (l  as  f32)  /  4.0; [9](#hl-4-9) [10](#hl-4-10) let  h  =  360.0  *  normalized_theme [11](#hl-4-11) +  120.0  *  normalized_ring_theme [12](#hl-4-12) +  30.0  *  normalized_h; [13](#hl-4-13) let  s  =  20.0  +  80.0  *  normalized_s; [14](#hl-4-14) let  l  =  40.0  +  50.0  *  normalized_l; [15](#hl-4-15) [16](#hl-4-16) format!("hsl({h}, {s}%, {l}%)") [17](#hl-4-17)} [18](#hl-4-18) [19](#hl-4-19)fn calculate_theme(bytes: &[u8])  -> f32 { [20](#hl-4-20) let  theme  =  bytes.iter().fold(0u8,  |acc,  byte|  acc  ^  byte); [21](#hl-4-21) (theme  as  f32)  /  (u8::MAX  as  f32) [22](#hl-4-22)} [23](#hl-4-23) [24](#hl-4-24)fn generate_paths(hash: [u8;  32])  { [25](#hl-4-25) let  theme  =  calculate_theme(&hash); [26](#hl-4-26) let  ring_themes  =  hash [27](#hl-4-27) .windows(8) [28](#hl-4-28) .map(calculate_theme) [29](#hl-4-29) .collect::<Vec<_>>(); [30](#hl-4-30) [31](#hl-4-31) let  colors  =  hash [32](#hl-4-32) .iter() [33](#hl-4-33) .cloned() [34](#hl-4-34) .enumerate() [35](#hl-4-35) .map(|(idx,  byte)|  to_color(theme,  ring_themes[idx  %  8],  byte)) [36](#hl-4-36) .collect::<Vec<_>>(); [37](#hl-4-37) [38](#hl-4-38) unimplemented!("Generate SVG paths."); [39](#hl-4-39)}`* 
```

*加上最后的修饰，我们得到：*

*"foo"的全局主题和环主题头像。*

*这正是隐藏在顶部剧透块中的东西。如果你克制住了自己没有去查看：恭喜你。现在可以随意查看了。 😃*

## *[#](#conclusions) 结论*

*撰写这篇博文是我努力想要更好地理解SVG内部工作原理的努力，同时也是一个从[Representing SHA-256 Hashes As Avatars](https://francoisbest.com/posts/2021/hashvatars)中逆向工程思想的挑战。*

*我目前没有为任何受益于这些头像的服务工作。如果将来我做了一个这样的服务，我肯定会试试它们。为了让这个解决方案更容易在其他地方重用，我做了一个名为[`svg_avatars`](https://crates.io/crates/svg_avatars)的 Rust 包，它实现了这个确切的解决方案。这个包非常简洁，如果有人有任何提升它的想法，我会很乐意听取您的意见或查看您的 PR。*

*作为结束思考，[绝对椭圆弧曲线](https://www.w3.org/TR/SVG2/paths.html#PathDataEllipticalArcCommands)命令的一个参数是 `sweep-flag` 参数。它允许您确定弧是笑脸还是哭脸。如果参数是 `1` — 就像我们的情况一样 — 那么弧是哭脸。然而，如果您将标志翻转为 `0`，那么所有的弧都会成为笑脸&mldr;*

**Smiley-face* 头像用于 "foo"。*

*&mldr;并且头像[出现](https://en.wikipedia.org/wiki/Emergence)为一个蜘蛛网。谁不喜欢 Spooktober 的单行更改功能。*

</main>
