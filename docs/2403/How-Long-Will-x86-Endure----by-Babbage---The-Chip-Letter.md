<!--yml

category: 未分类

date: 2024-05-29 12:48:27

-->

# x86会持续多久？ - Babbage撰写 - The Chip Letter

> 来源：[https://thechipletter.substack.com/p/how-long-will-x86-endure](https://thechipletter.substack.com/p/how-long-will-x86-endure)

现在是1979年，英特尔面临问题。它的16位[8086](https://en.wikipedia.org/wiki/Intel_8086)微处理器正在输给竞争对手。引用来自[口述历史](http://archive.computerhistory.org/resources/access/text/2015/09/102746836-05-01-acc.pdf)：

> Dave House: … 摩托罗拉正在销售68000，而Zilog正在销售Z8000，而我们在设计胜利选择时通常排名第三，使用8086。

为什么会输？

> Dane Elliot: 因此，任何了解小型计算机架构的工程师都会欣赏摩托罗拉和Zilog所提供的内容！
> 
> Dave House: 没错。
> 
> Rich Bader: 软件界的大佬！

8086被设计为一个过渡产品。英特尔将注意力集中在[iAPX432](https://en.wikipedia.org/wiki/Intel_iAPX_432)，一个具有一系列先进功能的32位处理器。但iAPX432还需要数年才能准备好。

8086项目始于1976年，1978年首次可购买，但其设计存在各种限制。首要问题是其分段存储结构，在地址其整个存储空间时增加了显著的复杂性。

因此，Crush诞生了。这是英特尔用来对抗摩托罗拉和Zilog竞争的不太友好的反垄断名称。它专注于8086及其伴随芯片作为系统解决方案，而不是8086的架构或软件。Crush很快获得了动力。到1980年，8086已经赢得了超过2,300个设计胜利。

而8086有一个秘密武器。针对英特尔流行的8位8080 CPU的汇编代码可以自动转换为8086兼容代码。软件开发人员可以拿出他们现有的应用程序，并迅速适应在8086上运行。

然后IBM选择了8086的廉价版本8088，创建了IBM PC。

历史由此铸就。

* * *

快进到今天，临时的8086已经被增强、扩展和加速。英特尔甚至在制造架构的64位版本时得到了主要竞争对手AMD的一点帮助。在这一过程中，该架构先后完全主宰了台式机和数据中心计算。

但现在我们有了可信的竞争对手。Arm和RISC-V不仅提供了可行的设计，还提供了有吸引力的替代商业模型，即许可或开源架构。

苹果已经在Mac上转向Arm。微软为Arm制作Windows。Arm实例可以在亚马逊、谷歌、微软和甲骨文的云上使用。这些公司使用Arm是因为它为许多用户提供了真正的优势，尤其是在成本和能效方面。

但要考虑架构的持续时间。你现在仍然可以购买6502和Z80兼容的CPU。这完全是在这些架构达到顶峰四十年后。而这些设计从未像x86那样普及和高产。

并且关键是，现在已经有超过四十年的软件专为在该架构上运行而编写。一些用户将继续需要x86向后兼容性，以确保他们的业务正常运行。

我认为可以肯定的是，至少六十年后我们仍然能够购买兼容x86的机器。也许会更久。

将x86翻译成Arm到RISC-V的代码 - 实际上，象形文字和德莫提克古埃及和古希腊在罗塞塔石碑上。

但是，20世纪80年代编写8080汇编代码并将其转换为8086代码的翻译工具在2020年代有了对应的工具。苹果的[Rosetta 2](https://www.computerworld.com/article/3597949/everything-you-need-to-know-about-rosetta-2-on-apple-silicon-macs.html)对64位x86二进制代码进行了提前编译，转换为Arm代码。微软也有类似的[工具](https://learn.microsoft.com/en-us/windows/arm/arm64ec)，使64位x86 Windows应用能在Windows for Arm上运行。这些工具使用户而非开发者能够轻松地从x86转向Arm。我们肯定会看到类似的工具，将同样的功能应用于RISC-V。向后兼容的枷锁已经被打破了。

那么x86还能持续多久？作为台式机、笔记本电脑和服务器的主要架构？我敢说至少十年，但不超过二十年。作为实际的台式机架构？我猜至少三十年。而且由于翻译工具的存在，x86代码很可能在此后更长的时间内存在。

你怎么看？请在评论中告诉我。

感谢阅读《芯片通讯》。如果你喜欢这篇文章，如果能分享，我将非常感激。

[分享](https://thechipletter.substack.com/p/how-long-will-x86-endure?utm_source=substack&utm_medium=email&utm_content=share&action=share)

##### #1

更多关于Crush的内容，请观看2013年的口头历史小组讨论视频。

##### #2

为什么Arm没有制作自己的x86到64位Arm的翻译工具？它从x86转向最有利。现在可能多余了，但几年前它可能是一个鼓励用户转换的有用工具。我感到困惑。

##### #3

在起草这篇文章后不久，我发现了[Box86](https://box86.org)，支持在64位Arm系统上运行64位x86二进制文件。当然，[qemu](https://www.qemu.org)应该允许在64位Arm系统上仿真（速度较慢的）x86虚拟机。

请务必告诉我，如果你知道其他的方法。

*奔腾Pro*

https://commons.wikimedia.org/wiki/File:Ppro512K.jpg

*罗塞塔石碑*

© Hans Hillewaert 根据CC BY-SA 4.0许可
