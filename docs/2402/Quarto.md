<!--yml

category: 未分类

date: 2024-05-27 14:51:47

-->

# Quarto

> 来源：[https://quarto.org/](https://quarto.org/)

结合 Jupyter 笔记本的灵活选项，以生成各种格式的生产质量输出。使用传统的笔记本 UI 或者使用笔记本的纯文本 Markdown 表示来编写。

Quarto 是 Posit 推出的 R Markdown 的多语言、下一代版本，具有许多新特性和功能。与 R Markdown 类似，Quarto 使用 [knitr](https://yihui.org/knitr/) 执行 R 代码，因此能够在不修改大多数现有 Rmd 文件的情况下进行渲染。

```
---
title: "ggplot2 demo"
author: "Norah Jones"
date: "5/22/2021"
format: 
 html:
 fig-width: 8
 fig-height: 4
 code-fold: true
---

## Air Quality

@fig-airquality further explores the impact of temperature on ozone level.

```{r}

#| label: fig-airquality

#| fig-cap: "温度和臭氧水平。"

#| warning: false

library(ggplot2)

ggplot(airquality, aes(Temp, Ozone)) +

geom_point() +

geom_smooth(method = "loess")

``` 
```

*结合 Markdown 和 Julia 代码创建动态文档，完全可复制。Quarto 通过 [IJulia](https://github.com/JuliaLang/IJulia.jl) Jupyter 内核执行 Julia 代码，使您可以使用纯文本（如下所示）编写或渲染现有的 Jupyter 笔记本。

```
---
title: "Plots Demo"
author: "Norah Jones"
date: "5/22/2021"
format:
 html:
 code-fold: true
jupyter: julia-1.8
---

## Parametric Plots

Plot function pair (x(u), y(u)). 
See @fig-parametric for an example.

```{julia}

#| label: fig-parametric

#| fig-cap: "参数化绘图"

using Plots

plot(sin,

x->sin(2x),

0,

2π,

leg=false,

fill=(0,:lavender))

``` 
```

*Quarto 包含原生支持 Observable JS，这是由 D3 的作者 Mike Bostock 创造的一组 JavaScript 增强功能。Observable JS 使用响应式执行模型，特别适用于交互式数据探索和分析。

```
---
title: "observable plot"
author: "Norah Jones"
format: 
 html: 
 code-fold: true
---

## Seattle Precipitation by Day (2012 to 2016)

```{ojs}

data = FileAttachment("seattle-weather.csv")

.csv({typed: true})

Plot.plot({

width: 800, height: 500, padding: 0,

color: { scheme: "blues", type: "sqrt"},

y: { tickFormat: i => "JFMAMJJASOND"[i] },

marks: [

Plot.cell(data, Plot.group({fill: "mean"}, {

x: d => new Date(d.date).getDate(),

y: d => new Date(d.date).getMonth(),

fill: "precipitation",

inset: 0.5

}))

]

})

```
```**
