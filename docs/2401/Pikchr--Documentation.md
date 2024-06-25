<!--yml

分类：未分类

日期：2024-05-27 14:30:52

-->

# Pikchr：文档

> 来源：[https://pikchr.org/home/doc/trunk/homepage.md](https://pikchr.org/home/doc/trunk/homepage.md)

Pikchr（发音为"picture"）是一种用于技术文档中的图表的[PIC](https://en.wikipedia.org/wiki/Pic_language)类似的标记语言。 Pikchr 被设计为嵌入在Markdown的[fenced code blocks](https://spec.commonmark.org/0.29/#fenced-code-blocks)或其他文档标记语言的类似机制中。

例如，下图：

<svg xmlns="http://www.w3.org/2000/svg" class="pikchr" viewBox="0 0 423.821 195.84"><text x="74.16" y="25.74" text-anchor="middle" fill="rgb(0,0,0)" dominant-baseline="central">Markdown</text> <text x="74.16" y="49.14" text-anchor="middle" fill="rgb(0,0,0)" dominant-baseline="central">源代码</text> <text x="209.75" y="17.28" text-anchor="middle" fill="rgb(0,0,0)" dominant-baseline="central">Markdown</text> <text x="209.75" y="37.44" text-anchor="middle" fill="rgb(0,0,0)" dominant-baseline="central">格式化器</text> <text x="209.75" y="57.6" text-anchor="middle" fill="rgb(0,0,0)" dominant-baseline="central">(markdown.c)</text> <text x="345.341" y="25.74" text-anchor="middle" fill="rgb(0,0,0)" dominant-baseline="central">HTML+SVG</text> <text x="345.341" y="49.14" text-anchor="middle" fill="rgb(0,0,0)" dominant-baseline="central">输出</text> <text x="209.75" y="138.24" text-anchor="middle" fill="rgb(0,0,0)" dominant-baseline="central">Pikchr</text> <text x="209.75" y="158.4" text-anchor="middle" fill="rgb(0,0,0)" dominant-baseline="central">格式化器</text> <text x="209.75" y="178.56" text-anchor="middle" fill="rgb(0,0,0)" dominant-baseline="central">(pikchr.c)</text></svg>

```
arrow right 200% "Markdown" "Source"
box rad 10px "Markdown" "Formatter" "(markdown.c)" fit
arrow right 200% "HTML+SVG" "Output"
arrow <-> down 70% from last box.s
box same "Pikchr" "Formatter" "(pikchr.c)" fit

```

由 7 行 Markdown 生成：

```
 ``` pikchr

arrow right 200% "Markdown" "源代码"

box rad 10px "Markdown" "格式化器" "(markdown.c)" fit

arrow right 200% "HTML+SVG" "输出"

arrow <-> down 70% from last box.s

box same "Pikchr" "格式化器" "(pikchr.c)" fit

``` 
```

Pikchr 图表可以出现在：

+   文档

+   Wiki 页面

+   票务和错误报告

+   论坛帖子

+   检入注释

+   在Markdown或类似的标记语言中使用的任何其他地方

Pikchr 图表易于生成。 语言简单。 有许多在线文档和示例（下面的链接）。 任何熟悉使用Markdown的人都应该能够轻松掌握Pikchr，几乎不需要额外的努力。

Pikchr 在面向互联网的应用中是安全的。 有害的Pikchr脚本不会造成任何伤害（除了生成丑陋的图表）。

## 演示

## Pikchr 文档

## 历史 PIC 文档副本

## 源代码

## 外部链接

## 源代码许可证：0-条款 BSD

Pikchr源代码是一个独立的原创作品。除了标准C库之外，它没有外部依赖，也不使用从互联网或其他外部来源获取的代码。所有的Pikchr源代码都发布在[零条款BSD许可证](https://spdx.org/licenses/0BSD.html)下。在使用[Lemon](https://www.sqlite.org/lemon.html)处理后，Pikchr源代码是一个名为"`pikchr.c`"的C89的单一文件。这些特性旨在使Pikchr[易于集成到其他系统中](https://pikchr.org/home/doc/trunk/doc/integrate.md)。
