<!--yml

category: 未分类

日期：2024-05-27 13:00:31

-->

# 新的CSS逻辑属性！CSS进化的下一步 | 作者：Elad Shechter | Medium

> 来源：[https://elad.medium.com/new-css-logical-properties-bc6945311ce7](https://elad.medium.com/new-css-logical-properties-bc6945311ce7)

## CSS浮动

浮动非常直观，只有两个值，`**inline-start**`/`**inline-end,**`替换了`left`/`right`的值**。**

使用英语（LTR）：

`float: left` = `float: **inline-start**`

`float: right` = `float**: inline-end**`

## 文本对齐

比浮动更容易，值`left`/`right`被`**start**`/`**end**`替换。

使用英语（LTR）：

`text-align :left` = `text-align: **start**;`

`text-align :right` = `text-align: **end**;`

## 更多内容

**Resize** 属性：主要用于 `<textarea>`，其值从`horizontal`/`vertical`更新为`**inline**`/`**block**`；

使用英语（LTR）：

`resize: horizontal` = `resize: **inline**;`

`resize: vertical` = `resize: **block;**`

**background-position**：尚未在任何浏览器中实现，但如果你仔细查看，可以在Mozilla的MDN网站中找到对`**background-position-inline**`和`**background-position-block**`的引用。尚无完整的文档，但他们正在努力中！ :-)

**其他内容**：我们可以假设像`**transform-origin**`这样的属性也将被更新，就像任何与方向有关的属性一样。

## CSS Grid & CSS Flexbox

对于CSS Grid和CSS Flexbox来说，一个好消息是它们已经使用新的逻辑属性方法构建，无需更新它们。

## 了解逻辑属性的工作流程

起初看起来非常复杂，但实际上非常容易使用。在编写样式时，无需担心跨语言支持。只需使用逻辑属性，而不是旧的物理属性，并将它们匹配到你的首选语言即可。

# 根据语言应用对齐

在我们学习了新的逻辑属性的所有更新之后，这里有两个属性让我们定义**块轴对齐**（网站的流动方向），以及**行内轴对齐**（文本阅读的方向）。

## Writing-mode 属性（块轴）

定义网站的流动方向。在大多数情况下，它将是从上到下，但正如提到的，某些语言可以从右到左（日语），甚至从左到右（蒙古语）。在这两种情况下，我们将有水平滚动而不是我们习惯的垂直滚动。

注意：目前有3个主要的`**writing-mode**`值。它们的名称有点令人困惑。这是因为在它们的名称中有块轴方向，再加上在该情况下文本的对齐方式（行内轴）。这显然非常令人沮丧，因为文本对齐是多余的，只会引起混淆。

为了消除这种混淆，建议忽略值的内联轴部分，只参考块轴部分的值。

示例：

**值：**

+   `writing-mode: horizontal-**tb**;` = **从上到下**的流动，就像英语中一样（默认值）

+   `writing-mode: vertical-**rl**;` = **从右到左流动**，适用于日语。

+   `writing-mode: vertical-**lr**;` = **从左到右流动**，适用于蒙古语。

至于我的个人看法，我更喜欢值仅包括：**tb/rl/lr（块轴部分）**，以消除这种潜在的混淆。

**日语**示例定义：

```
html{
    **writing-mode**: **vertical-rl**;
}
```

## 方向属性（内联轴）

定义文本是否应从**左到右**或**右到左**开始，但仅当默认的水平`writing-mode`属性处于活动状态时。如果我们将`writing-mode`更改为其中一种垂直模式，则实际文本方向从左到右将变为从上到下。或者反之，使用`rtl`（从右到左）值，则将变为从下到上。

**阿拉伯语**示例定义：

```
html{
    **direction**: **rtl**;
}
```

真是神奇，一个从上到下的网站如何可以轻松地变成一个带有水平滚动的从右到左的网站。

这是我制作的演示，在Firefox中效果最佳（目前支持更多功能）。

[**在线示例**](https://codepen.io/elad2412/pen/oQJmYQ/) **（试试语言选择！）：**

# 浏览器支持

+   所有盒模型属性 `margin`/`padding`/`border` 和新的 `width`/`height`（`inline-size`，`block-size`）属性在所有主要浏览器中工作，除了Edge。

+   `text-align` 的新值在所有主要浏览器中都可以工作，除了Edge。

+   `浮动`/`定位`/`调整大小` — 值/属性仅在Firefox中有效。

# 逻辑属性的问题

在这些新的重构中，我们面临新的问题。例如，如果我们想以简写方式编写所有`margin`属性：

`margin: 10px 20px 8px 5px;` 无法预测它将如何被解释。

如果网站使用物理属性，则这些值将被解释为：

`**margin-top**`**/**`**margin-right**`**/**`**margin-bottom**`**/**`**margin-left**`**，**

但是，如果网站使用新的逻辑属性，则值应为：

`**margin-block-start**`**/**`**margin-inline-end**`**/**`**margin-block-end**`**/**`**margin-inline-start**`**。**

在英语网站中，物理属性和逻辑属性的工作方式相同。在其他语言中，当我们使用像`margin`这样的简写时，目标是根据`**direction**`属性或新属性`**writing-mode**`来工作。

这仍然是一个悬而未决的问题。我在[*github csswg-drafts*](https://github.com/w3c/csswg-drafts/issues/1282#issuecomment-443253091)上添加了一个建议，可以解决这个问题。如果您认为有更好的解决方案，请在那里评论！

目前，如果要使用逻辑属性，必须使用完整的属性名称，不能使用任何简写。

**我个人的解决方案：**

```
html{
   flow-mode:**physical**; 
       /*or*/
   flow-mode:**logical**;
}.box{
  /***will be** **interpreted** **according to the HTML flow-mode value***/
   margin:10px 5px 6px 3px;
   padding:5px 10px 2px 7px;
}
```

## 响应式设计问题

在尝试制作一个完全工作的演示时，我尝试在媒体查询中使用新的“max-width”属性`**max-inline-size**`，理解左到右/右到左时它会像`**max-width**`一样运行，而对于日语等语言，它会像`**max-height**`一样。不幸的是，目前浏览器没有解释这个属性。

```
/*Not Working*/
[@media](http://twitter.com/media) (**max-inline-size**:1000px){
  .main-content{
    background:red;
    grid-template-columns:auto;
  }
}
```

## 考虑的变化

当我写这篇文章时，在深入学习和理解逻辑属性的概念之后，仍然存在一些缺失的适应性，我认为应在将来加以考虑：

- `line-height`可以是`“line-size”`

- `border-width`可以是`“border-size”`

看起来似乎不是这种情况，至少在`border-width`方面是这样。它刚刚更新了逻辑属性，并且它的名称仍然包含“width”这个词。

示例：`border-block-start-width`。

但谁知道，也许 W3C 的合适人士会看到这篇文章 :-)

# 最后的话

就这些了，

希望您喜欢这篇文章并从我的经验中学到了东西。

如果您喜欢这篇文章，我会很感激您的掌声和分享 :-)

我创建了大量关于CSS的内容。请务必通过 [**Twitter**](https://twitter.com/eladsc)**，** [**Linkedin**](https://www.linkedin.com/in/eladshechter/)**和** [**Medium**](https://medium.com/@elad)**跟随我**。

您也可以在我的网站上访问所有内容：[**eladsc.com**](https://eladsc.com/)。

**更多关于排版的推荐文章：**

[CSS书写模式](https://24ways.org/2016/css-writing-modes/) — 非常推荐！

[文本方向](https://tympanus.net/codrops/css_reference/text-orientation/)

**我更多的CSS文章：**

[为什么 CSS HSL 颜色更好！](https://medium.com/@elad/why-css-hsl-colors-are-better-83b1e0b6eead)

[响应式设计的新演进](https://medium.com/@elad/the-new-responsive-design-evolution-2bfb9b504a4e)

[CSS粘性定位 — 真正的工作原理！](https://medium.com/@elad/css-position-sticky-how-it-really-works-54cd01dc2d46)

**我是谁？**

**我是Elad Shechter**，一名专注于CSS和HTML设计与架构的Web开发者。

（我在2019年ConFrontJS会议上的发言，波兰华沙）

# 我的CSS逻辑属性演讲 — 新内容！

这是我关于CSS逻辑属性的完整演讲，请随意分享它 [🙂](https://emojipedia.org/slightly-smiling-face/)
