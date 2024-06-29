<!--yml

类别：未分类

日期：2024-05-27 15:03:59

-->

# 使用 Vim/Neovim、VimTeX 和代码片段提升 LaTeX 功能 | ejmastnak

> 来源：[https://www.ejmastnak.com/tutorials/vim-latex/intro/](https://www.ejmastnak.com/tutorials/vim-latex/intro/)

# 超级数学排版指南

这是一个教程系列，旨在帮助您配置 Vim 或 Neovim 文本编辑器，高效地用 LaTeX 写数学公式。这里是我设想的一个例子：

蓝色背景白色文字显示我输入的按键，底部显示生成的 LaTeX 源代码，顶部是编译后的输出。关于这如何工作的更多信息，请看下文。

<details class="my-8 py-2.5 px-4 border border-gray-500 dark:border-gray-500 rounded-lg overflow-hidden bg-blue-50 dark:bg-blue-900"><summary class="text-sky-600 dark:text-neutral-300 font-medium cursor-pointer">等等，你在说什么，LaTeX 是什么？</summary>[LaTeX](https://www.latex-project.org/)

是撰写数学、物理、计算机科学以及其他定量科学领域（但在这些领域外几乎未知，所以从未听说过也很正常）的行业标准排版软件。LaTeX 以生成高质量文档而闻名，但打字过程相对笨拙——本系列旨在提供一个消除这种笨拙的框架。</details>

**本指南的目标：** 使得用 LaTeX 写作像手写数学一样简单（快速、高效、愉快……）。技术栈：使用 UltiSnips 或 LuaSnip 代码片段插件的 Vim 文本编辑器，以及 VimTeX 插件的 LaTeX 编辑功能。如果您……

+   对使用 LaTeX 实时做笔记感兴趣，像 [Gilles Castel](https://castel.dev/) 一样，

+   想要一种明显比您可能最初学到的更愉快、更高效的 LaTeX 经验，无论您的动机是实时大学讲座速度还是其他，

+   希望从其他 LaTeX 编辑器切换到 Vim，但不确定如何继续，或者

+   只是出于好奇想要浏览他人的工作流程和配置。

**成本是什么：** 指南中的所有内容都是免费的，但这需要您的时间和精力。您可以在大约 15-30 分钟内浏览完指南；仔细阅读可能需要几个小时；而实际上，如果您对 Vim 不熟悉，您可能需要几个周末（或者可能几周）的专注和努力才能完全掌握。从那时起，要达到本页 GIF 图中的速度可能需要几个月的练习。

## 目录

1.  [**先决条件**](/tutorials/vim-latex/prerequisites/)

    包含获取系列最大利益的先决条件，以及如果需要的话，应该使您快速上手的参考资料。

1.  [**UltiSnips**](/tutorials/vim-latex/ultisnips/) 或 [**LuaSnip**](/tutorials/vim-latex/luasnip/)

    解释片段，实时 LaTeX 的关键。两篇文章涵盖了相同的内容——一次使用 UltiSnips 插件，一次使用 LuaSnip 插件。

1.  [**Vim 的 ftplugin 系统**](/tutorials/vim-latex/ftplugin/)

    介绍了Vim的文件类型插件系统，这将帮助你理解VimTeX插件。

1.  [**VimTeX插件**](/tutorials/vim-latex/vimtex/)

    出色的VimTeX插件是使用Vim而不是其他LaTeX编辑器的*原因*。

1.  [**编译**](/tutorials/vim-latex/compilation/)

    如何从Vim内部编译LaTeX文档。

1.  [**PDF阅读器**](/tutorials/vim-latex/pdf-reader/)

    如何集成Vim和PDF阅读器以查看LaTeX文档。

1.  [**Vim配置**](/tutorials/vim-latex/vimscript/)

    一个Vim配置指南，解释了本教程中使用的键映射和Vimscript函数。

## 系列更多内容

### 闭嘴，给我看结果

作为这个教程中的技术技巧有效性的具体证据，这里是来自我本科学习期间的[1500+页排版的物理笔记](/notes/fmf/fmf/)，其中大部分是实时在大学课堂上书写的（尽管语法和风格稍后进行了改进）。这些笔记的一些示例如下：

这里有更多GIF演示，展示LaTeX可以以手写速度书写：

这实际上比我手写要*快*一些 — 尝试拿出一支铅笔和纸张，看看你能否跟上！（是的，我知道通过投入一堆难以手写的积分来作弊。）如果你愿意，你可以在[**YouTube上看更多示例**](https://www.youtube.com/watch?v=P7iMX1lqGnU)。

**功劳归于**: 上述GIF图像的灵感来自Gilles Castel的视频 [使用Vim和UltiSnips进行快速LaTeX编辑](https://www.youtube.com/watch?v=a7gpx0h-BuU) — 非常出色，我鼓励你观看。

### 最初的Vim-LaTeX文章

顺便说一句：关于Vim和LaTeX的这一主题的重要工作，以及我尝试并最终成功地使用Vim实时编写LaTeX的灵感，来自Gilles Castel的[*我如何在数学讲座中使用LaTeX和Vim做笔记*](https://castel.dev/post/lecture-notes-1/)。如果你涉足Vim或LaTeX圈子，你可能已经在互联网上看到了，如果还没有的话，你绝对应该阅读一下。

本系列文章在Castel的文章基础上，通过更详细地介绍技术实现细节（例如设置带前向和逆向搜索的PDF阅读器的细节，如何使用VimTeX插件，如何编写Vimscript函数和键映射，Vim的`ftplugin`系统的工作原理，如何编译LaTeX文档等）。

### 配置

这里是本系列使用的设置概述：

### 反馈、建议等等

如果你有改进系列的想法，我很可能会实现它们，感谢你的意见，并因你的贡献而赞扬。欢迎并感谢您的反馈。

特别感谢之前的读者：非常感谢[@rltyty](https://github.com/rltyty)，[Carl von Randow](https://github.com/carlvr)，Ehud Gordon，Andrey Rukhin，[Merlin Büge](https://github.com/camoz)，[Albert Gu](https://github.com/albertfgu)，Pano Otis，Jason Yao，[@Glirastes](https://github.com/Glirastes)，[Daniele Avitabile](https://www.danieleavitabile.com/)，Kai Breucker，Maxwell Jiang，[@lodisy](https://github.com/lodisy)，以及[@subnut](https://github.com/subnut)，因为发现错误并提供了改进这个系列的好主意。

你可以通过电子邮件联系我，邮箱是[[email protected]](/cdn-cgi/l/email-protection#a0c5ccc9cac1cee0c5cacdc1d3d4cec1cb8ec3cfcd)，或者在[github.com/ejmastnak/ejmastnak.com](https://github.com/ejmastnak/ejmastnak.com)上提出问题或请求合并。

### 想要表达感谢？

你可以：

+   [给我发送电子邮件！](/contact/)真的，如果这篇材料对你有帮助，我会很高兴知道的。我喜欢听读者的反馈，你几乎肯定会收到我的回复。

+   [资助我买咖啡。](https://www.buymeacoffee.com/ejmastnak)根据读者的反馈，确实有人有兴趣通过金钱来补偿我写这篇指南。这真是太棒了，谢谢！你可以在这里[给我买杯咖啡](https://www.buymeacoffee.com/ejmastnak)。
