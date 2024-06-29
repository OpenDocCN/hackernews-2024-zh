<!--yml

category: 未分类

日期：2024-05-27 13:38:59

-->

# pdf.tocgen

> 来源：[https://krasjet.com/voice/pdf.tocgen/](https://krasjet.com/voice/pdf.tocgen/)

<main>

```
 in.pdf                                    
                            │                                       
     ┌──────────────────────┼────────────────────┐                  
     │                      │                    │                  
     ▽                      ▽                    ▽                  
┌──────────┐  recipe  ┌───────────┐   ToC   ┌──────────┐            
│ pdfxmeta ├─────────▷│ pdftocgen ├────────▷│ pdftocio ├───▷ out.pdf
└──────────┘          └───────────┘         └──────────┘ 
```

[pdf.tocgen](https://sink.krj.st/pdf.tocgen/) 是一组命令行工具，用于自动提取和生成 PDF 文件的目录（ToC）。它利用嵌入式字体属性和标题位置来推断 PDF 文件的基本轮廓。

# 概述

例如，对于保罗·格雷厄姆的书籍 *On Lisp* 的 PDF 版本，可在 [他的网站](http://www.paulgraham.com/onlisptext.html) 下载，但缺少目录，我们可以使用 `pdfxmeta` 命令构建一个配方文件，

在 [1]

```
`$ pdfxmeta --auto 1 --page 14 onlisp.pdf "Extensible" >> recipe.toml` 
```

在 [2]

```
`$ pdfxmeta --auto 2 --page 14 onlisp.pdf "^Design" >> recipe.toml` 
```

在 [3]

```
`$ sed '/^#.*/d' < recipe.toml` 
```

在 [3]

```
`[[heading]]
level = 1
greedy = true
font.name = "Times-Bold"
font.size = 19.92530059814453

[[heading]]
level = 2
greedy = true
font.name = "Times-Bold"
font.size = 11.9552001953125` 
```

保存为 `recipe.toml`。然后使用 `pdftocgen` 自动为该书生成目录，使用该配方提取。

在 [4]

```
`$ pdftocgen onlisp.pdf < recipe.toml` 
```

在 [4]

```
`"Preface" 5
 "Bottom-up Design" 5
 "Plan of the Book" 7
 "Examples" 9
 "Acknowledgements" 9
"Contents" 11
"The Extensible Language" 14
 "1.1 Design by Evolution" 14
 "1.2 Programming Bottom-Up" 16
 "1.3 Extensible Software" 18
 "1.4 Extending Lisp" 19
 "1.5 Why Lisp (or When)" 21
"Functions" 22
 "2.1 Functions as Data" 22
 "2.2 Defining Functions" 23
 "2.3 Functional Arguments" 26
 "2.4 Functions as Properties" 28
 "2.5 Scope" 29
 "2.6 Closures" 30
 "2.7 Local Functions" 34
 "2.8 Tail-Recursion" 35
 "2.9 Compilation" 37
 "2.10 Functions from Lists" 40
[--snip--]` 
```

我们可以将输出保存到一个名为 `toc` 的文件中，

在 [5]

```
`$ pdftocgen onlisp.pdf < recipe.toml > toc` 
```

并将其导入到原始 PDF 文件中，使用 `pdftocio` 命令保存为 `output.pdf`。

在 [6]

```
`$ pdftocio -o output.pdf onlisp.pdf < toc` 
```

这只是基本工作流程的概述。请阅读 [section 4](#a-worked-example) 以获取详细的解释和步骤指南。

pdf.tocgen 最适合使用 `pdftex`（及其伙伴 `pdflatex`、`pdfxetex` 等）生成的 PDF 文件，但它设计用于与任何 *软件生成* 的 PDF 文件一起工作。也就是说，不应该期望它能够处理扫描的 PDF 文件。一些示例包括 `troff`/<wbr>`groff`、Adobe InDesign、Microsoft Word，可能还有其他。

pdf.tocgen 是一款免费软件。源代码可以在 [这里](https://sink.krj.st/pdf.tocgen/) 或 [GitHub](https://github.com/Krasjet/pdf.tocgen/) 找到，并且根据 GPLv3 许可发布。您可以自由地修改源代码，但任何衍生作品 *必须* 保证用户的 [自由](https://www.gnu.org/philosophy/free-sw.en.html)。如果您想为该项目做出贡献，请发送一个 [patch](https://sink.krj.st) 或在 [GitHub](https://github.com/Krasjet/pdf.tocgen/) 上打开一个拉取请求。

# 安装

pdf.tocgen 使用 Python 3 编写。已知可以在 Linux、Windows 和 macOS 上与 Python 3.7 到 3.11 兼容。在 BSD 上，您可能需要自行构建 PyMuPDF。使用

在 [7]

```
`$ pip install -U pdf.tocgen` 
```

安装最新版本系统范围内。或者，使用 [`pipx`](https://pipxproject.github.io/pipx/) 或

在 [8]

```
`$ pip install -U --user pdf.tocgen` 
```

以供当前用户安装。我建议使用后者以避免破坏您系统上的软件包管理器。

如果您使用基于 Arch 的 Linux 发行版，则该软件包也可在 [AUR](https://aur.archlinux.org/packages/pdf.tocgen/) 上找到。可以使用任何 AUR 助手进行安装，例如 [`yay`](https://github.com/Jguer/yay)：

在 [9]

```
`$ yay -S pdf.tocgen` 
```

# LaTeX 用户请注意

在我们继续之前，请注意这个工具的目标是 *读者*，而不是 *作者*。

所需的用途是为 [讲座笔记](https://see.stanford.edu/materials/lsoftaee261/book-fall-07.pdf)、[草稿书籍](https://felleisen.org/matthias/HtDC/htdc.pdf) 或者 [arXiv](https://arxiv.org/) 上的论文生成目录，以便*更容易导航*。如果你是用户，请*不要*使用此工具为你自己的手稿生成目录。

使用 [hyperref](https://ctan.org/pkg/hyperref) 包替代，

因为它更好地理解你的文档并提供更多定制选项。

# 一个示例

pdf.tocgen 的设计受到 [Unix 哲学](https://en.wikipedia.org/wiki/Unix_philosophy) 的影响。我有意将 pdf.tocgen 分离为 3 个独立的程序。它们可以协同工作，但每个程序单独使用时也很有用。

它们代表了为 PDF 文件添加目录所需的 3 个步骤：

1.  `pdfxmeta`: 提取标题的元数据（字体属性、位置）以构建一个配方文件。

1.  `pdftocgen`: 从配方生成目录。

1.  `pdftocio`: 将目录导入到 PDF 文档中。

同样，我们将使用保罗·格雷厄姆的书 *On Lisp*，在他的网站上免费提供的 [PDF](http://www.paulgraham.com/onlisptext.html)，但未嵌入目录，来演示这 3 个程序如何协同工作。

如果你想跟着做，请下载这本书并保存为 `onlisp.pdf`。

## 步骤 1：构建一个配方

`pdftocgen` 使用 PDF 文件中嵌入文本的字体属性和位置（边界框）来提取标题。我们需要为其提供一个配方，即一个 [TOML](https://toml.io) 文件，告诉 `pdftocgen` 如何识别标题、副标题或次级标题。

我们的示例配方 *On Lisp*，我之前已经提到过，看起来像这样：

```
`# filter for level 1 heading
[[heading]]
# heading level
level = 1
# attributes to search for
font.name = "Times-Bold"
font.size = 19.92530059814453

# filter for level 2 heading
[[heading]]
level = 2
font.name = "Times-Bold"
font.size = 11.9552001953125` 
```

一个配方是一个过滤器列表，每个过滤器指定 `pdftocgen` 应查找的属性。例如，在上述配方中，一级标题对应于 *On Lisp* 中的章节标题应该

1.  字体名称匹配 `"Times-Bold"`

1.  字体大小为 `19.92530059814453`

当然，字体大小不必如此精确，你可以使用 `font.size_tolerance` 属性设置容差水平：

```
`[[heading]]
level = 1
font.name = "Times-Bold"
# match anything between 19.9-20.1
font.size = 20
font.size_tolerance = 0.1` 
```

我们如何知道一个标题应该具有哪些属性？这就是为什么我们需要使用 `pdfxmeta` 提取 PDF 文件中文本的元数据。

打开你刚下载的 PDF 文件，在你喜欢的 PDF 阅读器中查看。我们将在这里使用 [zathura](https://pwmt.org/projects/zathura/)，但你可以使用任何你喜欢的软件。

在 [10]

```
`$ zathura onlisp.pdf &` 
```

滚动到第 14 页，从封面开始计数。第一个章节标题“可扩展语言”在这里。要查看与该标题相关的元数据，请使用 `pdfxmeta` 命令：

在 [11]

```
`$ pdfxmeta -p 14 onlisp.pdf "The Extensible"` 
```

在 [11]

```
`The Extensible Language:
 font.name = "Times-Bold"
 font.size = 19.92530059814453
 font.color = 0x000000
 font.superscript = false
 font.italic = false
 font.serif = true
 font.monospace = false
 font.bold = true
 bbox.left = 138.60000610351562
 bbox.top = 306.947998046875
 bbox.right = 354.4866638183594
 bbox.bottom = 334.5445251464844` 
```

`-p` 标志告诉 `pdfxmeta` 它应仅在第 14 页上搜索。这不是必需的，但我强烈建议指定它以减少搜索的歧义性。

现在我们已经提取了章节标题的元数据，我们可以通过复制粘贴它，或者将输出重定向到名为`recipe.toml`的食谱文件。

在 [12]

```
`$ pdfxmeta -p 14 onlisp.pdf "The Extensible" >> recipe.toml` 
```

使用你喜欢的编辑器打开食谱文件并删除缩进在Vim中，你可以在正常模式下按`<<`来取消行缩进，并添加`[[heading]]`头和`level`属性来指定标题级别。

```
`[[heading]]
level = 1
font.name = "Times-Bold"
font.size = 19.92530059814453
font.color = 0x000000
font.superscript = false
font.italic = false
font.serif = true
font.monospace = false
font.bold = true
bbox.left = 138.60000610351562
bbox.top = 306.947998046875
bbox.right = 354.4866638183594
bbox.bottom = 334.5445251464844` 
```

这已经是一个有效的食谱文件，但太具体了。其他章节标题几乎不可能与所有四个边界框（`bbox`）的值匹配，这意味着它们的位置和宽度与本章的标题完全相同。

如果要忽略一个属性，只需从过滤器中删除它。根据我的经验，通常只需要`font.name`和`font.size`。

```
`[[heading]]
level = 1
font.name = "Times-Bold"
font.size = 19.92530059814453` 
```

如果你懒得话，也可以使用`--auto`或`-a`标志将输出格式化为带有默认设置的标题过滤器。但输出会稍微难以阅读：

在 [13]

```
`$ pdfxmeta -a 1 -p 14 onlisp.pdf "The Extensible"` 
```

Out [13]

```
`[[heading]]
# The Extensible Language
level = 1
greedy = true
font.name = "Times-Bold"
font.size = 19.92530059814453
# font.size_tolerance = 1e-5
# font.color = 0x000000
# font.superscript = false
# font.italic = false
# font.serif = true
# font.monospace = false
# font.bold = true
# bbox.left = 138.60000610351562
# bbox.top = 306.947998046875
# bbox.right = 354.4866638183594
# bbox.bottom = 334.5445251464844
# bbox.tolerance = 1e-5` 
```

`-a`标志后的参数是输出标题过滤器的标题级别，在本例中为`1`。现在不要担心`greedy`选项，对于本书来说并不必要。

接下来，我们需要提取二级标题的元数据：

在 [14]

```
`$ pdfxmeta -p 14 -i onlisp.pdf "^design"` 
```

Out [14]

```
`Design by Evolution:
 font.name = "Times-Bold"
 font.size = 11.9552001953125
 font.color = 0x000000
 font.superscript = false
 font.italic = false
 font.serif = true
 font.monospace = false
 font.bold = true
 bbox.left = 165.42388916015625
 bbox.top = 624.02880859375
 bbox.right = 268.2960205078125
 bbox.bottom = 640.5867309570312` 
```

注意我们可以使用正则表达式（Python风格）作为模式。 `-i`选项可以用于启用大小写不敏感搜索。

使用`-a`标志将其转储到`recipe.toml`

在 [15]

```
`$ pdfxmeta -a 2 -p 14 -i onlisp.pdf "^design" >> recipe.toml` 
```

并选择我们需要的属性：

```
`[[heading]]
level = 2
font.name = "Times-Bold"
font.size = 11.9552001953125` 
```

如果找不到你想要的内容，可以省略查询字符串来转储整个页面，

在 [16]

```
`$ pdfxmeta -p 14 onlisp.pdf` 
```

Out [16]

```
`1:
 font.name = "Times-Bold"
 font.size = 24.906600952148438
 font.color = 0x000000
 font.superscript = false
 font.italic = false
 font.serif = true
 font.monospace = false
 font.bold = true
 bbox.left = 138.60000610351562
 bbox.top = 239.2274932861328
 bbox.right = 151.05331420898438
 bbox.bottom = 273.72314453125
The Extensible Language:
 font.name = "Times-Bold"
 font.size = 19.92530059814453
 font.color = 0x000000
 font.superscript = false
 font.italic = false
 font.serif = true
 font.monospace = false
 font.bold = true
 bbox.left = 138.60000610351562
 bbox.top = 306.947998046875
 bbox.right = 354.4866638183594
 bbox.bottom = 334.5445251464844
[--snip--]` 
```

但很可能`pdftocgen`无法使用`pdfxmeta`找到标题来为您生成有意义的目录。对此感到抱歉。

顺便说一句，有时你会在`font.name`中看到`+`符号，例如在Matthias Felleisen等人撰写的[*如何设计类*](https://felleisen.org/matthias/HtDC/htdc.pdf)的PDF草稿中。在最新版本中，子集将自动被去除，所以你可能看不到这个符号了。

在 [17]

```
`$ pdfxmeta -p 19 htdc.pdf "The Varieties"` 
```

Out [17]

```
`The Varieties of Data:
 font.name = "UFRVLO+Palatino-Bold"
 font.size = 17.21540069580078
 font.color = 0x221f1f
 font.superscript = false
 font.italic = false
 font.serif = true
 font.monospace = false
 font.bold = true
 bbox.left = 293.6400146484375
 bbox.top = 129.75328063964844
 bbox.right = 459.6872253417969
 bbox.bottom = 146.77931213378906` 
```

`font.size`中的`UFRVLO+`代表`Palatino-Bold`完整字体系列的*子集*。不同标题中的字体可能分布在多个子集中，因此通过删除子集前缀来匹配整个系列几乎总是一个好主意：

```
`[[heading]]
level = 1
font.name = "Palatino-Bold"
font.size = 17.21540069580078
font.color = 0x221f1f` 
```

你也可以像以前一样使用`-a`标志来自动处理这个问题。

在 [18]

```
`$ pdfxmeta -a 1 -p 19 htdc.pdf "The Varieties"` 
```

Out [18]

```
`[[heading]]
# The Varieties of Data
level = 1
greedy = true
font.name = "Palatino-Bold"
font.size = 17.21540069580078
# font.size_tolerance = 1e-5
# font.color = 0x221f1f
# font.superscript = false
# font.italic = false
# font.serif = true
# font.monospace = false
# font.bold = true
# bbox.left = 293.6400146484375
# bbox.top = 129.75328063964844
# bbox.right = 459.6872253417969
# bbox.bottom = 146.77931213378906
# bbox.tolerance = 1e-5` 
```

还要注意`font.name`可以接受正则表达式：

```
`[[heading]]
level = 3
font.name = "(Palatino-Bold|Palatino-Italic)"
font.size = 11.9552001953125
font.color = 0x221f1f` 
```

你可能需要一些实验来确定PDF的最佳食谱。你可以在这里找到一些预制的食谱[here](https://github.com/Krasjet/pdf.tocgen/tree/master/recipes)，但在大多数情况下，你需要设计自己的食谱。欢迎通过发送补丁或拉取请求贡献更多的食谱。

## 第2步：生成目录

现在我们已经为*On Lisp*制定了一个名为`recipe.toml`的食谱，

```
`[[heading]]
level = 1
font.name = "Times-Bold"
font.size = 19.92530059814453

[[heading]]
level = 2
font.name = "Times-Bold"
font.size = 11.9552001953125` 
```

我们可以使用`pdftocgen`命令为我们的PDF生成目录。

在 [19]

```
`$ pdftocgen onlisp.pdf < recipe.toml` 
```

出 [19]

```
`"Preface" 5
 "Bottom-up Design" 5
 "Plan of the Book" 7
 "Examples" 9
 "Acknowledgements" 9
"Contents" 11
"The Extensible Language" 14
 "1.1 Design by Evolution" 14
 "1.2 Programming Bottom-Up" 16
 "1.3 Extensible Software" 18
 "1.4 Extending Lisp" 19
 "1.5 Why Lisp (or When)" 21
"Functions" 22
 "2.1 Functions as Data" 22
 "2.2 Defining Functions" 23
 "2.3 Functional Arguments" 26
 "2.4 Functions as Properties" 28
 "2.5 Scope" 29
 "2.6 Closures" 30
 "2.7 Local Functions" 34
 "2.8 Tail-Recursion" 35
 "2.9 Compilation" 37
 "2.10 Functions from Lists" 40
"Functional Programming" 41
 "3.1 Functional Design" 41
 "3.2 Imperative Outside-In" 46
 "3.3 Functional Interfaces" 48
 "3.4 Interactive Programming" 50
[--snip--]` 
```

`pdftocgen` 的输出是 CSV 的一种方言。主要区别在于

1.  每 *4 spaces* 的缩进表示一个新的标题级别，因此三级标题是 *8 spaces* 的缩进

1.  分隔符是一个空格，而不是逗号

1.  标题需要使用双引号引起来。

这种格式故意设计成在 Vim 中易于 *编辑*，因为 `pdftocgen` 的输出在许多情况下被期望是 *不准确的*，您可能需要在将目录导入到原始 PDF 文件之前调整它。

如果标题包含实际的双引号字符 (`"`)，请使用两个双引号 (`""`) 进行转义，

```
"a ""quoted"" heading" 2
```

但是大多数 PDF 中的双引号通常是智能引号（`“` 和 `”`），因此很少需要转义。

如果您不想将目录导入到 PDF，可以使用 `-H` 标志以更可读的格式打印目录，

在 [20]

```
`$ pdftocgen -H onlisp.pdf < recipe.toml` 
```

出 [20]

```
`Preface ··· 5
 Bottom-up Design ··· 5
 Plan of the Book ··· 7
 Examples ··· 9
 Acknowledgements ··· 9
Contents ··· 11
The Extensible Language ··· 14
 1.1 Design by Evolution ··· 14
 1.2 Programming Bottom-Up ··· 16
 1.3 Extensible Software ··· 18
 1.4 Extending Lisp ··· 19
 1.5 Why Lisp (or When) ··· 21
Functions ··· 22
 2.1 Functions as Data ··· 22
 2.2 Defining Functions ··· 23
 2.3 Functional Arguments ··· 26
 2.4 Functions as Properties ··· 28
 2.5 Scope ··· 29
 2.6 Closures ··· 30
 2.7 Local Functions ··· 34
 2.8 Tail-Recursion ··· 35
 2.9 Compilation ··· 37
 2.10 Functions from Lists ··· 40
Functional Programming ··· 41
 3.1 Functional Design ··· 41
 3.2 Imperative Outside-In ··· 46
 3.3 Functional Interfaces ··· 48
 3.4 Interactive Programming ··· 50
[--snip--]` 
```

但是 `pdftocio` 无法读取这种格式。

现在，我们将目录保存到名为 `toc` 的文件中：

在 [21]

```
`$ pdftocgen onlisp.pdf < recipe.toml > toc` 
```

## 第三步：将 ToC 导入到 PDF

要将生成的目录导入到原始的 PDF 文件中，只需将 `toc` 文件重定向到 `pdftocio`。

在 [22]

```
`$ pdftocio onlisp.pdf < toc` 
```

输出的 PDF 文档名为 `onlisp_out.pdf`，如果需要不同的文件名，请使用 `-o` 标志。当您在 zathura 中打开它时

在 [23]

```
`$ zathura onlisp_out.pdf` 
```

并按 `TAB` 键，您应该看到目录已成功导入到 PDF 文件中。

实际上，如果您不想编辑目录，`pdftocgen` 的输出可以直接导入到 `pdftocio` 中

在 [24]

```
`$ pdftocgen onlisp.pdf < recipe.toml | pdftocio onlisp.pdf` 
```

我将这个程序称为 `pdftocio`，因为它还可以用于在没有提供任何外部输入的情况下 *输出* PDF 文档的现有目录

在 [25]

```
`$ pdftocio onlisp_out.pdf` 
```

出 [25]

```
`"Preface" 5
 "Bottom-up Design" 5
 "Plan of the Book" 7
 "Examples" 9
 "Acknowledgements" 9
"Contents" 11
"The Extensible Language" 14
 "1.1 Design by Evolution" 14
 "1.2 Programming Bottom-Up" 16
 "1.3 Extensible Software" 18
 "1.4 Extending Lisp" 19
 "1.5 Why Lisp (or When)" 21
"Functions" 22
 "2.1 Functions as Data" 22
 "2.2 Defining Functions" 23
 "2.3 Functional Arguments" 26
 "2.4 Functions as Properties" 28
 "2.5 Scope" 29
 "2.6 Closures" 30
 "2.7 Local Functions" 34
 "2.8 Tail-Recursion" 35
 "2.9 Compilation" 37
 "2.10 Functions from Lists" 40
"Functional Programming" 41
 "3.1 Functional Design" 41
 "3.2 Imperative Outside-In" 46
 "3.3 Functional Interfaces" 48
 "3.4 Interactive Programming" 50
[--snip--]` 
```

或者使用 `-H` 标志以更可读的格式显示它。

在 [26]

```
`$ pdftocio -H onlisp_out.pdf` 
```

出 [26]

```
`Preface ··· 5
 Bottom-up Design ··· 5
 Plan of the Book ··· 7
 Examples ··· 9
 Acknowledgements ··· 9
Contents ··· 11
The Extensible Language ··· 14
 1.1 Design by Evolution ··· 14
 1.2 Programming Bottom-Up ··· 16
 1.3 Extensible Software ··· 18
 1.4 Extending Lisp ··· 19
 1.5 Why Lisp (or When) ··· 21
Functions ··· 22
 2.1 Functions as Data ··· 22
 2.2 Defining Functions ··· 23
 2.3 Functional Arguments ··· 26
 2.4 Functions as Properties ··· 28
 2.5 Scope ··· 29
 2.6 Closures ··· 30
 2.7 Local Functions ··· 34
 2.8 Tail-Recursion ··· 35
 2.9 Compilation ··· 37
 2.10 Functions from Lists ··· 40
Functional Programming ··· 41
 3.1 Functional Design ··· 41
 3.2 Imperative Outside-In ··· 46
 3.3 Functional Interfaces ··· 48
 3.4 Interactive Programming ··· 50
[--snip--]` 
```

## 一个总结

尽管上面的例子变得很长，但关键步骤非常简单。

首先，使用 `pdfxmeta` 搜索标题元数据

在 [27]

```
`$ pdfxmeta -a 1 -p page in.pdf "Section" >> recipe.toml
$ pdfxmeta -a 2 -p page in.pdf "Subsection" >> recipe.toml` 
```

编辑 `recipe.toml` 文件以选择您需要的属性，或者如果不需要任何自定义，则保持默认设置：

在 [28]

```
`$ vim recipe.toml # edit` 
```

然后使用配方生成目录，并将其导入到 PDF 文件中

在 [29]

```
`$ pdftocgen in.pdf < recipe.toml | pdftocio -o out.pdf in.pdf` 
```

或者，如果您想在导入之前编辑目录，

在 [30]

```
`$ pdftocgen in.pdf < recipe.toml > toc
$ vim toc # edit
$ pdftocio in.pdf < toc` 
```

这三个程序各自都有一些额外的功能。使用 `-h` 选项查看您可以传递的所有选项。

# 另一个例子：数学

pdf.tocgen 的配方格式提供了一些方便的选项来处理标题中的数学符号。它们对于处理数学讲义、教科书或论文特别有用。

让我们看另一个例子，[斯坦福的EE261讲义](https://see.stanford.edu/materials/lsoftaee261/book-fall-07.pdf)，由 Brad Osgood 教授编写。这份文档也没有目录，并且标题中包含许多数学符号，如，，等。

这些数学符号与周围文本的字体不同，如果我们使用`font.name`指定标题，则可能会出现问题：

```
`[[heading]]
level = 1
font.name = "CMBX12"
font.size = 24.78696060180664

[[heading]]
level = 2
font.name = "CMBX12"
font.size = 14.346190452575684

[[heading]]
level = 3
font.name = "CMBX12"
font.size = 11.955169677734375` 
```

上述配方是使用生成的

在 [31]

```
`$ pdfxmeta -a 1 -p 7 ft.pdf "Fourier Series" >> ft.toml` 
```

在 [32]

```
`$ pdfxmeta -a 2 -p 7 ft.pdf "Introduction" >> ft.toml` 
```

在 [33]

```
`$ pdfxmeta -a 3 -p 28 ft.pdf "punchline" >> ft.toml` 
```

并清理掉注释。

如果我们传递配方文件给`pdftocgen`，您会注意到标题中缺少一些数学符号。

在 [34]

```
`$ pdftocgen -H ft.pdf < ft.toml` 
```

出 [34]

```
`EE 261 ··· 1
Contents ··· 3
Fourier Series ··· 7
 1.1 Introduction and Choices to Make ··· 7
 1.2 Periodic Phenomena ··· 8
 1.2.1 Time and space ··· 8
 1.2.2 More on spatial periodicity ··· 9
 1.3 Periodicity: Definitions, Examples, and Things to Come ··· 10
 1.3.1 The view from above ··· 12
 1.3.2 The building blocks: a few more examples ··· 13
 1.3.3 Musical pitch and tuning ··· 14
 1.4 It All Adds Up ··· 15
 1.5 Lost at ··· 16
 1.6 Period, Frequencies, and Spectrum ··· 19
 1.6.1 What if the period isn’t 1? ··· 20
 1.7 Two Examples and a Warning ··· 22
 1.8 The Math, the Majesty, the End ··· 27
 1.8.1 Square integrable functions ··· 27
 1.8.2 The punchline revealed ··· 28
 1.9 Orthogonality ··· 32
 1.10 Appendix: The Cauchy-Schwarz Inequality and its Consequences ··· 39
 1.11 Appendix: More on the Complex Inner Product ··· 42
 1.12 Appendix: Best Approximation by Finite Fourier Series ··· 44
 1.13 Fourier Series in Action ··· 45
 1.13.1 Hot enough for ya? ··· 45
 1.13.2 A nonclassical example: What’s the buzz? ··· 53
 1.14 Notes on Convergence of Fourier Series ··· 56
 1.14.1 How big are the Fourier coefficients? ··· 56
 1.14.2 Rates of convergence and smoothness ··· 59
 1.14.3 Convergence if it’s not continuous? ··· 60
 1.15 Appendix: Pointwise Convergence vs. Uniform Convergence ··· 64
 1.16 Appendix: Studying Partial Sums via the Dirichlet Kernel: The Buzz Is Back ··· 65
 1.17 Appendix: The Complex Exponentials Are a Basis for ··· 67
 1.18 Appendix: More on the Gibbs Phenomenon ··· 68
[--snip--]` 
```

注意第1.5、1.12和1.17节的标题。将其与PDF第3页进行比较，您应该注意到标题中缺少数学符号。

有几种解决此问题的方法。

## 选项1：贪婪过滤器

正如你可能已经注意到的，这是推荐的方法。它是`pdfxmeta`的`-a`选项的默认设置，因为它简单且在所有选项中效果最好。

贪婪过滤器将提取标题的*封闭*或*周围*区域的任何文本，即使某些部分与当前过滤器不匹配也可以。

这恰好是我们当前的情况，因为标题中的数学符号通常由匹配过滤器的普通文本包围，但它们本身采用不同的字体。

如果您想使一个标题过滤器贪婪，只需添加

```
greedy = true
```

到配方文件中相应的标题过滤器。例如，对于上面的配方，我们有

```
`[[heading]]
level = 1
greedy = true
font.name = "CMBX12"
font.size = 24.78696060180664

[[heading]]
level = 2
greedy = true
font.name = "CMBX12"
font.size = 14.346190452575684

[[heading]]
level = 3
greedy = true
font.name = "CMBX12"
font.size = 11.955169677734375` 
```

现在配方应该能够自动提取标题中的数学符号。

在 [35]

```
`$ pdftocgen -H ft.pdf < ft.toml` 
```

出 [35]

```
`EE 261 ··· 1
Contents ··· 3
Fourier Series ··· 7
 1.1 Introduction and Choices to Make ··· 7
 1.2 Periodic Phenomena ··· 8
 1.2.1 Time and space ··· 8
 1.2.2 More on spatial periodicity ··· 9
 1.3 Periodicity: Definitions, Examples, and Things to Come ··· 10
 1.3.1 The view from above ··· 12
 1.3.2 The building blocks: a few more examples ··· 13
 1.3.3 Musical pitch and tuning ··· 14
 1.4 It All Adds Up ··· 15
 1.5 Lost at c ··· 16
 1.6 Period, Frequencies, and Spectrum ··· 19
 1.6.1 What if the period isn’t 1? ··· 20
 1.7 Two Examples and a Warning ··· 22
 1.8 The Math, the Majesty, the End ··· 27
 1.8.1 Square integrable functions ··· 27
 1.8.2 The punchline revealed ··· 28
 1.9 Orthogonality ··· 32
 1.10 Appendix: The Cauchy-Schwarz Inequality and its Consequences ··· 39
 1.11 Appendix: More on the Complex Inner Product ··· 42
 1.12 Appendix: Best L 2  Approximation by Finite Fourier Series ··· 44
 1.13 Fourier Series in Action ··· 45
 1.13.1 Hot enough for ya? ··· 45
 1.13.2 A nonclassical example: What’s the buzz? ··· 53
 1.14 Notes on Convergence of Fourier Series ··· 56
 1.14.1 How big are the Fourier coefficients? ··· 56
 1.14.2 Rates of convergence and smoothness ··· 59
 1.14.3 Convergence if it’s not continuous? ··· 60
 1.15 Appendix: Pointwise Convergence vs. Uniform Convergence ··· 64
 1.16 Appendix: Studying Partial Sums via the Dirichlet Kernel: The Buzz Is Back ··· 65
 1.17 Appendix: The Complex Exponentials Are a Basis for L 2 ([0 , 1]) ··· 67
 1.18 Appendix: More on the Gibbs Phenomenon ··· 68
[--snip--]` 
```

当然，它仍然需要进一步清理，但使用一个体面的文本编辑器应该很容易。

## 选项2：正则表达式

或者，由于`font.name`接受正则表达式，我们可以使用`(a|b)`语法提供多个字体名称。例如，我们可以先使用GNU或BSD的`grep`来查找数学符号的元数据：

在 [36]

```
`$ pdfxmeta -p 16 ft.pdf | grep -A 25 "Lost at"` 
```

出 [36]

```
`Lost at:
 font.name = "HYPGHW+CMBX12"
 font.size = 14.346190452575684
 font.color = 0x000000
 font.superscript = false
 font.italic = false
 font.serif = true
 font.monospace = false
 font.bold = true
 bbox.left = 87.12625122070312
 bbox.top = 510.04034423828125
 bbox.right = 137.03665161132812
 bbox.bottom = 524.40087890625
 c:
 font.name = "SYTZSX+CMMIB10"
 font.size = 14.346190452575684
 font.color = 0x000000
 font.superscript = false
 font.italic = true
 font.serif = true
 font.monospace = false
 font.bold = true
 bbox.left = 137.03665161132812
 bbox.top = 510.04034423828125
 bbox.right = 150.005615234375
 bbox.bottom = 524.3865356445312` 
```

现在我们知道`c`的字体名称是`CMMIB10`，我们可以修改过滤器以包含它。

```
`[[heading]]
level = 2
font.name = "(CMBX12|CMMIB10)"
font.size = 14.346190452575684` 
```

然而，由于上标通常具有与周围文本不同的字体大小，这种方法在这里不适用。

## 选项3：多个过滤器

另一种选择是为同一级别的标题使用多个过滤器。在内部，如果它们在同一块中，则相同级别的匹配文本将合并成一个。

对于前一节的示例，写法等同于

```
`[[heading]]
level = 2
font.name = "CMBX12"
font.size = 14.346190452575684

[[heading]]
level = 2
font.name = "CMMIB10"
font.size = 14.346190452575684` 
```

但这使我们能够指定不同的字体大小。

使用与前一节相同的方法，我们可以找到文本中所有数学符号的元数据，并为它们创建单独的标题过滤器，从而得到以下配方：

```
`[[heading]]
level = 1
font.name = "CMBX12"
font.size = 24.78696060180664

[[heading]]
# text
level = 2
font.name = "CMBX12"
font.size = 14.346190452575684

[[heading]]
# math
level = 2
font.name = "(CMMIB10|CMMI12|CMR12)"
font.size = 14.346190452575684

[[heading]]
# superscript
level = 2
font.name = "CMR10"
font.size = 9.962639808654785
font.superscript = true

[[heading]]
level = 3
font.name = "CMBX12"
font.size = 11.955169677734375` 
```

此过滤器能够提取所有数学符号，

在 [37]

```
`$ pdftocgen -H ft.pdf < ft.toml` 
```

出 [37]

```
`EE 261 ··· 1
 Prof. Brad Osgood Electrical Engineering Department Stanford University ··· 1
Contents ··· 3
Fourier Series ··· 7
 1.1 Introduction and Choices to Make ··· 7
 1.2 Periodic Phenomena ··· 8
 1.2.1 Time and space ··· 8
 1.2.2 More on spatial periodicity ··· 9
 1.3 Periodicity: Definitions, Examples, and Things to Come ··· 10
 1.3.1 The view from above ··· 12
 1.3.2 The building blocks: a few more examples ··· 13
 1 ··· 13
 1 ··· 13
 1.3.3 Musical pitch and tuning ··· 14
 1.4 It All Adds Up ··· 15
 1.5 Lost at c ··· 16
 1.6 Period, Frequencies, and Spectrum ··· 19
 1.6.1 What if the period isn’t 1? ··· 20
 1.7 Two Examples and a Warning ··· 22
 1.8 The Math, the Majesty, the End ··· 27
 1.8.1 Square integrable functions ··· 27
 1.8.2 The punchline revealed ··· 28
 1.9 Orthogonality ··· 32
 1.10 Appendix: The Cauchy-Schwarz Inequality and its Consequences ··· 39
 1.11 Appendix: More on the Complex Inner Product ··· 42
 1.12 Appendix: Best L 2 Approximation by Finite Fourier Series ··· 44
 1.13 Fourier Series in Action ··· 45
 1.13.1 Hot enough for ya? ··· 45
 1.13.2 A nonclassical example: What’s the buzz? ··· 53
 1.14 Notes on Convergence of Fourier Series ··· 56
 1.14.1 How big are the Fourier coefficients? ··· 56
 1.14.2 Rates of convergence and smoothness ··· 59
 1.14.3 Convergence if it’s not continuous? ··· 60
 1.15 Appendix: Pointwise Convergence vs. Uniform Convergence ··· 64
 1.16 Appendix: Studying Partial Sums via the Dirichlet Kernel: The Buzz Is Back ··· 65
 1.17 Appendix: The Complex Exponentials Are a Basis for L 2 ([0 , 1]) ··· 67
 1.18 Appendix: More on the Gibbs Phenomenon ··· 68
[--snip--]` 
```

但这种方法不够精确，引入了许多不必要的符号，但在后处理中应该相对容易清理。

如果有疑问，只需使用选项1并使过滤器贪婪。选项2和选项3只是最后的手段。

# 配方文件

除了由`pdfxmeta`生成的所有元数据之外，在配方文件中还有几个其他选项可以指定。以下是每个标题过滤器的所有有效键的列表。

```
`# filter for level 1 heading
[[heading]]
# heading level
level = 1
# makes this filter a *greedy* filter, useful for PDFs
# containing many math symbols
greedy = true
font.name = "CMBX12"
# matches font.size ± font.size_tolerance
font.size = 14.346199989318848
font.size_tolerance = 1e-5
font.color = 0x000000
font.superscript = false
font.italic = false
font.serif = true
font.monospace = false
font.bold = true
# matches bbox.left ± bbox.tolerance
bbox.left = 157.98439025878906
# matches bbox.top ± bbox.tolerance
bbox.top = 335.569580078125
# matches bbox.right ± bbox.tolerance
bbox.right = 477.66058349609375
# matches bbox.bottom ± bbox.tolerance
bbox.bottom = 349.93011474609375
bbox.tolerance = 1e-5

# filter for level 2 heading
[[heading]]
level = 2
greedy = false
font.name = "CMBX10"
font.size = 9.962599754333496
font.size_tolerance = 1e-5
font.color = 0x000000
font.superscript = false
font.italic = false
font.serif = true
font.monospace = false
font.bold = true
bbox.left = 168.76663208007812
bbox.top = 127.2930679321289
bbox.right = 280.66656494140625
bbox.bottom = 137.2556610107422
bbox.tolerance = 1e-5

# ...` 
```

再次，不要过于具体地关注边界框，因为标题通常具有不同的宽度。如果不确定要选择哪些属性，请使用 `pdfxmeta` 的 `-a` 选项。

# 命令示例

由于设计的模块化，每个程序都可以单独使用，尽管是管道的一部分。本节将提供一些更多的示例，说明您可以如何使用它们。随意想出更多。

## `pdftocio`

`pdftocio` 应该最好展示这一点，这个程序可以独立完成很多工作。

要显示 PDF 中现有的目录到 `stdout`：

在 [38]

```
`$ pdftocio doc.pdf` 
```

出 [38]

```
`"Level 1 heading 1" 1
 "Level 2 heading 1" 1
 "Level 3 heading 1" 2
 "Level 3 heading 2" 3
 "Level 2 heading 2" 4
"Level 1 heading 2" 5` 
```

要将 PDF 中现有的目录写入名为 `toc` 的文件：

在 [39]

```
`$ pdftocio doc.pdf > toc` 
```

要将 `toc` 文件写回到 `doc.pdf`：

在 [40]

```
`$ pdftocio doc.pdf < toc` 
```

要指定输出 PDF 的名称：

在 [41]

```
`$ pdftocio -o out.pdf doc.pdf < toc` 
```

要将目录从 `doc1.pdf` 复制到 `doc2.pdf`：

在 [42]

```
`$ pdftocio  -v  doc1.pdf  |  pdftocio  doc2.pdf` 
```

注意 `-v` 标志有助于在复制过程中保留标题的垂直位置。

打印目录以供阅读：

在 [43]

```
`$ pdftocio -H doc.pdf` 
```

出 [43]

```
`Level 1 heading 1 ··· 1
 Level 2 heading 1 ··· 1
 Level 3 heading 1 ··· 2
 Level 3 heading 2 ··· 3
 Level 2 heading 2 ··· 4
Level 1 heading 2 ··· 5` 
```

## `pdftocgen`

如果您已经获得了用于 `doc.pdf` 的现有配方 `rcp.toml`，您可以应用它并通过以下方式打印大纲到 `stdout`：

在 [44]

```
`$ pdftocgen  doc.pdf  <  rcp.toml` 
```

出 [44]

```
`"Level 1 heading 1" 1
 "Level 2 heading 1" 1
 "Level 3 heading 1" 2
 "Level 3 heading 2" 3
 "Level 2 heading 2" 4
"Level 1 heading 2" 5` 
```

要将目录输出到名为 `toc` 的文件中：

在 [45]

```
`$ pdftocgen doc.pdf < rcp.toml > toc` 
```

要导入生成的目录到 PDF 文件，并输出到 `doc_out.pdf`：

在 [46]

```
`$ pdftocgen doc.pdf < rcp.toml | pdftocio -o doc_out.pdf doc.pdf` 
```

要打印生成的目录以供阅读：

在 [47]

```
`$ pdftocgen -H doc.pdf < rcp.toml` 
```

出 [47]

```
`Level 1 heading 1 ··· 1
 Level 2 heading 1 ··· 1
 Level 3 heading 1 ··· 2
 Level 3 heading 2 ··· 3
 Level 2 heading 2 ··· 4
Level 1 heading 2 ··· 5` 
```

如果您想要在每个标题的页面中包括垂直位置，请使用 `-v` 标志

在 [48]

```
`$ pdftocgen -v doc.pdf < rcp.toml` 
```

出 [48]

```
`"Level 1 heading 1" 1 306.947998046875
 "Level 2 heading 1" 1 586.3488159179688
 "Level 3 heading 1" 2 586.5888061523438
 "Level 3 heading 2" 3 155.66879272460938
 "Level 2 heading 2" 4 435.8687744140625
"Level 1 heading 2" 5 380.78875732421875` 
```

`pdftocio` 可以理解输出中的垂直位置，以生成链接到标题确切位置而不是页面顶部的目录条目。

在 [49]

```
`$ pdftocgen -v doc.pdf < rcp.toml | pdftocio doc.pdf` 
```

注意 `pdftocio` 的默认输出在这里是 `doc_out.pdf`。

要在整个 PDF 中搜索 `Anaphoric`：

在 [50]

```
`$ pdfxmeta onlisp.pdf "Anaphoric"` 
```

出 [50]

```
`14\. Anaphoric Macros:
 font.name = "Times-Bold"
 font.size = 9.962599754333496
 font.color = 0x000000
 font.superscript = false
 font.italic = false
 font.serif = true
 font.monospace = false
 font.bold = true
 bbox.left = 308.6400146484375
 bbox.top = 307.1490478515625
 bbox.right = 404.33282470703125
 bbox.bottom = 320.9472351074219
[--snip--]` 
```

要将结果输出为具有自动设置的标题过滤器，

在 [51]

```
`$ pdfxmeta -a 1 onlisp.pdf "Anaphoric"` 
```

出 [51]

```
`[[heading]]
# 14\. Anaphoric Macros
level = 1
greedy = true
font.name = "Times-Bold"
font.size = 9.962599754333496
# font.size_tolerance = 1e-5
# font.color = 0x000000
# font.superscript = false
# font.italic = false
# font.serif = true
# font.monospace = false
# font.bold = true
# bbox.left = 308.6400146484375
# bbox.top = 307.1490478515625
# bbox.right = 404.33282470703125
# bbox.bottom = 320.9472351074219
# bbox.tolerance = 1e-5
# [--snip--]` 
```

可以直接写入配方文件的内容：

在 [52]

```
`$ pdfxmeta -a 1 onlisp.pdf "Anaphoric" >> recipe.toml` 
```

在整个 PDF 中进行不区分大小写的搜索 `Anaphoric`：

在 [53]

```
`$ pdfxmeta -i onlisp.pdf "Anaphoric"` 
```

出 [53]

```
`to compile-time. Chapter 14 introduces anaphoric macros, which allow you to:
 font.name = "Times-Roman"
 font.size = 9.962599754333496
 font.color = 0x000000
 font.superscript = false
 font.italic = false
 font.serif = true
 font.monospace = false
 font.bold = false
 bbox.left = 138.60000610351562
 bbox.top = 295.6583557128906
 bbox.right = 459.0260009765625
 bbox.bottom = 308.948486328125
[--snip--]` 
```

使用正则表达式在整个 PDF 中进行不区分大小写的搜索，搜索 `Anaphoric`：

在 [54]

```
`$ pdfxmeta onlisp.pdf "[Aa]naphoric"` 
```

出 [54]

```
`to compile-time. Chapter 14 introduces anaphoric macros, which allow you to:
 font.name = "Times-Roman"
 font.size = 9.962599754333496
 font.color = 0x000000
 font.superscript = false
 font.italic = false
 font.serif = true
 font.monospace = false
 font.bold = false
 bbox.left = 138.60000610351562
 bbox.top = 295.6583557128906
 bbox.right = 459.0260009765625
 bbox.bottom = 308.948486328125
[--snip--]` 
```

仅在第 203 页上搜索：

在 [55]

```
`$ pdfxmeta -p 203 onlisp.pdf "anaphoric"` 
```

出 [55]

```
`anaphoric if, called:
 font.name = "Times-Roman"
 font.size = 9.962599754333496
 font.color = 0x000000
 font.superscript = false
 font.italic = false
 font.serif = true
 font.monospace = false
 font.bold = false
 bbox.left = 138.60000610351562
 bbox.top = 283.17822265625
 bbox.right = 214.81094360351562
 bbox.bottom = 296.4683532714844
[--snip--]` 
```

要转储第 203 页的整个页面：

在 [56]

```
`$ pdfxmeta -p 203 onlisp.pdf` 
```

出 [56]

```
`190:
 font.name = "Times-Roman"
 font.size = 9.962599754333496
 font.color = 0x000000
 font.superscript = false
 font.italic = false
 font.serif = true
 font.monospace = false
 font.bold = false
 bbox.left = 138.60000610351562
 bbox.top = 126.09941101074219
 bbox.right = 153.54388427734375
 bbox.bottom = 139.38951110839844
[--snip--]` 
```

要转储整个 PDF 文档：

在 [57]

```
`$ pdfxmeta onlisp.pdf` 
```

出 [57]

```
`i:
 font.name = "Times-Roman"
 font.size = 9.962599754333496
 font.color = 0x000000
 font.superscript = false
 font.italic = false
 font.serif = true
 font.monospace = false
 font.bold = false
 bbox.left = 458.0400085449219
 bbox.top = 126.09941101074219
 bbox.right = 460.8096008300781
 bbox.bottom = 139.38951110839844
[--snip--]` 
```

# 开发

如果您想修改源代码或贡献任何内容，请首先安装 [`poetry`](https://python-poetry.org/)，这是 pdf.tocgen 使用的 Python 依赖项和包管理器。然后运行

在存储库的根目录中设置开发依赖项。

如果您想测试 pdf.tocgen 的开发版本，请添加

```
poetry run
```

每个命令的前缀，例如

在 [59]

```
`$ poetry run pdfxmeta in.pdf "pattern"` 
```

或者，您也可以使用

命令以打开虚拟环境并直接运行开发版本：

在 [61]

```
`(pdf.tocgen) $ pdfxmeta in.pdf "pattern"` 
```

在发送补丁或拉取请求之前，请确保单元测试通过运行：

# GUI 前端

如果您使用Emacs，可以安装Daniel Nicolai的[toc-mode](https://github.com/dalanicolai/toc-mode)包作为pdf.tocgen的GUI前端。toc-mode还提供许多其他功能，如从PDF文件中提取（打印的）目录。请注意，它在内部使用pdf.tocgen，因此在使用toc-mode作为pdf.tocgen的前端之前，仍然需要安装pdf.tocgen。

# 许可证

pdf.tocgen本身是自由软件。pdf.tocgen的源代码根据GNU GPLv3许可证授权。然而，`recipes`目录中的配方单独授权为[CC BY-NC-SA 4.0许可证](https://creativecommons.org/licenses/by-nc-sa/4.0/)，以防止任何商业用途，因此未包含在分发中。

pdf.tocgen基于[PyMuPDF](https://github.com/pymupdf/PyMuPDF)开发，其许可证为GNU GPLv3，而PyMuPDF又基于MuPDF开发，MuPDF的许可证为GNU AGPLv3。AGPLv3许可证的副本已包含在代码库中。

如果您想基于此项目制作任何衍生作品，请遵循GNU GPLv3许可证的条款。

</main>
