<!--yml

category: 未分类

日期：2024-05-27 14:49:26

-->

# John Graham-Cumming的博客：原始WWW提案是来自1990年的Word for Macintosh 4.0文件，我们能打开它吗？

> 来源：[https://blog.jgc.org/2024/02/the-original-www-proposal-is-word-for.html](https://blog.jgc.org/2024/02/the-original-www-proposal-is-word-for.html)

W3C有一个页面提供了[原始WWW提案](https://www.w3.org/History/1989/proposal.html)，由Tim Berners-Lee提供下载。

“我无法测试它”让我感到悲伤。还有另外两个文件（RTF版本和1998年从原始文件生成的HTML版本）。但我们能打开原始文档吗？

原始文档是68,608字节，我的Mac上的文件称它是Microsoft Word for Macintosh 4.0文件。这与TBL在W3C页面上的注释相符：“对原始MacWord（或Word for Mac？）文档的手工转换，于1989年3月编写，并在1990年5月添加日期后未改变。”

Microsoft Office for Mac在1989年推出，配备了System 6.0。这是Microsoft Word 4.0，因此我们需要兼容Microsoft Word for Macintosh 4.0。让我们看看现代软件如何打开它。我真正希望能够做的是打开它并将其转换为高保真度的PDF。

## Microsoft Word

让我们从Microsoft Word本身开始。我将文件上传到Microsoft OneDrive，并使用.doc扩展名点击打开Microsoft Word。

## Apple Pages

我换到Mac上，并希望Apple Pages能理解一个旧的Microsoft Word for Macintosh文件。没那么幸运。

## Apache OpenOffice

接下来让我们希望开源软件能挽救它。我下载了最新的Apache OpenOffice，它确实打开了文件，但格式已经消失，并且缺少图表。

## LibreOffice

好吧，也许我需要不同的开源软件，所以我转向了最新的

[LibreOffice](https://www.libreoffice.org/)

而且它打开了。图表很清晰！尽管边距有些奇怪，并且还有其他格式问题。

## CERN PDF

CERN提供

[PDF版本](https://cds.cern.ch/record/369245/files/dd-89-001.pdf)

这份提案显然是1998年使用Acrobat Distiller Daemon 2.1 for SunOS/Solaris（SPARC）创建的。它有20页。LibreOffice导入版本有24页。

为了获得不同之处的概述，我从LibreOffice版本创建了一个PDF，然后在Apple Preview的联系表版本中查看了它和CERN PDF。

这是CERN的PDF：

这是由LibreOffice生成的PDF：

不同之处：

1\. LibreOffice版本中缺少右边距。

2\. LibreOffice版本使用14 pt而不是大多数文本的12 pt。

3\. LibreOffice版本将带有TBL缩写的页眉变成了页脚。

4\. 页码看起来放置得很合适（看看图像是如何正确放置在末尾的）；因此最大的问题可能是字体大小。

5\. CERN PDF文档在标题下有一个空白，而LibreOffice版本没有。

## 模拟

为了确保我知道实际原始文档的外观，我决定使用

[Infinite Mac](https://infinitemac.org/1990/System%206.0.5)

启动1990年代的Macintosh并在原始文档上运行实际的Word for Macintosh 4.0。

这样我就可以看到实际的字体、字号和布局，确认文档应该看起来的样子。这时很明显，原始Mac上的原始文档和CERN PDF文档有很大不同。CERN PDF有20页，在运行Word for Macintosh 4.0并使用A4纸的Mac上有22页。所以我决定努力使我们接近Mac上原始文档的状态。

所以... 设置为A4纸，并将右页边距设置为与左页边距相同。由于第一页没有相同的装订线、页脚或页眉，更改第一页格式是不同的。手动将正文文本从14pt（和其他大小）更改为12pt。手动处理文本在页面上错误地跨页和其他对齐问题。修复应该是页眉的页脚。

最终，我得到了与Mac上可见内容非常接近的结果。

将这份文档从原始格式转换为开源软件取得了一点胜利。这是关于文档保护有多难的一课。为了帮助稍微保存它，并且使用开放格式，我将我的.odt版本上传到了GitHub

[这里](https://github.com/jgrahamc/www-proposal)

。看到这份34年前的文档难以打开，即使打开了，输出的结果也不完全与原始一致，这令人感兴趣，也有点令人沮丧。

PS 如果你想知道我为什么开始这个项目。我只是想要原始提案中图表的高质量版本，花了比我预想的时间长。

PPS A

[评论](https://news.ycombinator.com/item?id=39358074)

指出，我可能可以通过模拟的Mac机器创建一个PostScript文件或PDF文件。我能够启动另一台装有Word 1992和Print2PDF（创建PDF文件的打印驱动程序）的Mac（System 7），并直接从Macintosh 5.1a的Word打印。

我已将生成的PDF文件添加到GitHub。这个版本有20页，字体不同，但满足了我最初对PDF的要求。

PPPS Hacker News上的

[评论](https://news.ycombinator.com/item?id=39363611)

链接到另一个转换，使用不同版本的Word和一些摆弄，以便在现代格式中得到文档的真正好版本。
