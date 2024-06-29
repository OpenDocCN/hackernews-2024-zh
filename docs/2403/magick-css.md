<!--yml

category: 未分类

date: 2024-05-29 12:34:17

-->

# magick.css

> 来源：[https://css.winterveil.net](https://css.winterveil.net)

<main>

> 在宇宙知识的宏伟总集中，普遍认为巫师的涂鸦不仅超越了世俗，而且毫无疑问是人类所知最令人振奋的记号形式。因为谁能抗拒边距上巧妙施放的魔法呢？
> 
> Albus Greybeard，《神秘边注的美德》，1132年

## 基础知识

magick.css 是一个极简（大部分）无类名的 CSS 框架，旨在易于使用和理解。它只包含一个文件，并且每一个选择都有注释。其目标是在最大化可读性和传达信息能力的同时，达到优雅但魔法般好玩的外观；*有些类似于巫师的笔记*。

该框架在 **所有设备** 和屏幕尺寸上均保持其美观和功能性，并且完全 **不需要 JavaScript**。它受到 [LaTeX](https://www.latex-project.org/)、老派 TTRPG 规则书、CSS 框架如 [concrete.css](https://concrete.style/) 和 [Tufte CSS](https://edwardtufte.github.io/tufte-css/) 以及“可用性即设计”伦理的启发。

## 如何使用 magick.css

检查 magick.css 是否适合你自己的项目的最简单方法是从 CDN 添加它。只需将以下两行添加到你的 html `<head>` 中：

```
<link rel="stylesheet" href="https://unpkg.com/normalize.css">
<link rel="stylesheet" href="https://unpkg.com/magick.css">
```

你也可以从这个仓库下载 magick.css 文件，并像往常一样将其包含到你的 html `<head>` 中：

```
<link rel="stylesheet" href="path/to/magick.css">
```

你也可以像这样将 magick.css 添加到你的 JS 项目中：

```
npm install normalize.css magick.css
```

然后在你的代码中导入它们：

```
import 'normalize.css'
import 'magick.css'
```

对于任何更多的需求：magick.css 应该可以与任何 HTML5 文档一起工作，因为它几乎没有类。虽然有一些你可以使用的酷炫功能；只需向下滚动以发现它们！

## 布局

要使你的页面结构化为响应式、可读的列，请将所有内容包装在 `<main>` 标签中，然后使用 `<section>` 标签将长篇内容划分为各个部分。本页是如何操作的一个很好的例子。

（当然，你也可以忽略这些建议，只使用 magick.css 进行排版和样式组件。）

这里是通常结构的简单示例：

```
<html>
  <head>
  </head>
  <body>
    <main>
      <header> </header>
      <section> </section>
      <section> </section>
      <section> </section>
      <footer> </footer>
    </main>
  </body>
</html>
```

要添加相关信息，magick.css 使用 `<aside>` 标签。你会在本页不时看到它的使用。如果你想详细阐述某一点或提供额外的背景信息，可以使用 magick.css 的边注。边注会自动编号！你可以在下面的 [Extras](#extras) 部分了解更多。

## 排版

这里展示了 magick.css 提供的排版示例：

# Heading 1: 一个粗体而宏大的标题，故事的门户。

## Heading 2: 副标题在下方，章节等待着。

### Heading 3: 在这里，秘密低语，情节变得更加复杂。

#### Heading 4: Fine details shared, the message stands clear.

段落如流水般流淌，

[像门户一样的链接，我们接下来要去哪里？](#)

**强如磐石，我们的话语坚定不移，**

*强调的低语，轻声降临*。

下划线的秘密，轻微地暗示，

虽小却强大，我们的话语被铸造。

`代码以逻辑，简洁而整洁，`

排版的咒语，现在完成。

`<h4>` 以下的标题不受 magick.css 的样式影响；它们通常不需要用于层次结构化，而是用于非常特定的用例。因此，如果您需要它们，可以根据需要自定义样式。

> [值得注意的是] 费曼讲义[...] 用 1800 页的篇幅写出所有物理内容，仅使用了两个层次的标题结构：章节和文本中的 A 级标题。

## 结构化内容

### 列表

列表保持简单，以不分散内容为原则：

+   这是...

+   ... 一个无序列表。

1.  这是...

1.  ... 一个有序列表。

这个怪人在这里...

... 是一个描述列表。

### 表格

与列表类似，表格清晰而简洁，引导读者关注手头的数据：

| 药剂 | 主要成分 | 催化剂 |
| --- | --- | --- |
| 发光露水 | 生物发光藻类（81mg） | 🜼 |
| 浮叶 | 蒲公英花粉（61mg） | 🝆 |
| 强壮酿酒 | 人参根（78mg） | 🝀 |
| 消失草稿 | 触须草（26mg） | 🜞 |
| 记忆茶 | 银杏叶（39mg） | 🜇 |

## 代码、引用和预格式化

### 代码

显示代码可以内联或在一个独立的块中完成。要内联代码，请使用 `<code>` 标签。对于较大的代码片段，请使用 `<pre>` 包装 `<code>` 标签。顺便说一下，您也可以使用图像来显示代码片段！

```
123456789void fib(int x) {
    unsigned long a = 0, b = 1, c;
    for(int i = 0; i < x; i++) {
        printf("%lu ", a);
        c = a + b;
        a = b;
        b = c;
    }
}
```

此 C 函数生成斐波那契数列的数字。

（代码的行编号需要对内容进行额外预处理，但即使手动也很简单。在 [附加](#extras) 部分了解更多！）

### 引用

对于引用，请使用 `<blockquote>` 标签。根据内容，您可能需要使用预格式化文本或让浏览器处理格式：

> 四月的一天阳光明媚而寒冷，钟声敲响了十三下。

在引用块内部使用 `<footer>` 添加引用。查看此页面的源代码以了解如何操作！

现在让我们考虑一首诗，其中格式对浏览器无法动态确定的重要性很大。在这种情况下，我们将诗歌内容包裹在 `<pre>` 标签中：

> ```
> The angels, not half so happy in Heaven,
>    Went envying her and me— 
> Yes!— that was the reason (as all men know,
>    In this kingdom by the sea)
> That the wind came out of a cloud, chilling
>    And killing my Annabel Lee.
> ```
> 
> 来自爱伦·坡的《安娜贝尔·李》摘录

## 就这样！

希望我能引起您对使用 magick.css 的兴趣。如果您喜欢所见的内容，请考虑在 [GitHub](https://github.com/wintermute-cell/magick.css) 上给它一个星！这不会花费您任何代价，但会让我知道**有人在乎**！

如果您对我建造的其他东西感兴趣，请加入我的 [Discord](https://discord.gg/f8nyyK4ngZ)！

</main>
