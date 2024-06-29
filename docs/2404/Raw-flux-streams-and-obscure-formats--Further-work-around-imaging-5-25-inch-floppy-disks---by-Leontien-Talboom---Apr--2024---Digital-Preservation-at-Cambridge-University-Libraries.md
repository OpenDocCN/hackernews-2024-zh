<!--yml

category: 未分类

date: 2024-05-27 13:23:56

-->

# 原始通量流和模糊格式：关于成像5.25英寸软盘的进一步工作 | 由莱昂提恩·塔尔布姆撰写 | 2024年4月 | 剑桥大学图书馆数字保存

> 来源：[https://digitalpreservation-blog.lib.cam.ac.uk/raw-flux-streams-and-obscure-formats-further-work-around-imaging-5-25-inch-floppy-disks-5a2cf2e5f0d1](https://digitalpreservation-blog.lib.cam.ac.uk/raw-flux-streams-and-obscure-formats-further-work-around-imaging-5-25-inch-floppy-disks-5a2cf2e5f0d1)

# **原始通量流和模糊格式：关于成像5.25英寸软盘的进一步工作**

*莱昂提恩·塔尔布姆，技术分析员，剑桥大学图书馆*

*克里斯·诺尔斯，数字档案管理员，* [*丘吉尔档案中心*](http://archives.chu.cam.ac.uk/)

本博客是我们早期合作的延续（可以在这里找到 [here](/diskovering-nostalgia-a-5-25-inch-floppy-tale-7347e5251fd0) 和 [here](https://www.chu.cam.ac.uk/news/archives-centre/all-together-now-world-digital-preservation-day-2023/)）。这些文章聚焦于我们首次尝试建立工作流程，以丘吉尔档案中心藏品中的5.25英寸软盘作为示例进行成像。

我们发现位于剑桥大学图书馆数字保存实验室（CUL）中的法庭取证设备（FRED）工作站中的当前软盘控制器（FC5025）无法正确成像丘吉尔档案中心的软盘。FC5025无法读取其中四个磁盘，并且在成像时有25个错误。只有一个磁盘 —— 那个格式为[MS-DOS](https://en.wikipedia.org/wiki/MS-DOS)的磁盘 —— 成功被成像。

我们认为无法成像的磁盘是[国际计算机有限公司（ICL）](https://en.wikipedia.org/wiki/International_Computers_Limited)和[Wang](https://en.wikipedia.org/wiki/Wang_Laboratories)磁盘。自第一次尝试以来，我们对这些磁盘类型进行了更详细的研究。ICL磁盘仍然非常难以理解（我们很愿意与任何曾使用过这些磁盘的人交流）；Wang磁盘导致了与来自数字保存领域不同机构的同事Elizabeth Kata、Richard Lehane和Tyler Thorsted的交流。Richard和Elizabeth已经建立了一个用于解释Wang磁盘图像的系统，并且我们可以确认从我们提取的内容中，它们确实符合预期的格式（尽管存在一些细微差异），但我们的图像显然是不完整或捕获不正确（或两者兼而有之），因此我们需要尝试不同的捕获方法。

这开始了我们寻找不同软盘控制器的旅程，经过剑桥大学图书馆的高级技术专家建议，剑桥大学图书馆和CAC都投资了[GreaseWeazle](https://github.com/keirf/greaseweazle)。这个控制器位于计算机和软盘驱动器之间，与[FluxEngine](https://cowlark.com/fluxengine/)和[Kryoflux](https://www.kryoflux.com/)等类似产品类似，可以读取和保存原始磁通流。GreaseWeazle和FluxEngine的软件可以在GreaseWeazle硬件上使用，还包括识别的软盘磁通流配置文件，将原始磁通流转换为二进制数据。这是访问软盘上实际文件所必需的。

**寻找软盘驱动器并使其运行起来**

获取一个可以工作的5.25英寸软盘驱动器比预期更具挑战性。剑桥大学图书馆（CUL）的数字保存实验室之前有几个驱动器，这些驱动器是早些时候慷慨捐赠给数字保存实验室的。然而，很快发现这些驱动器是为双密度软盘设计的，意味着它们无法读取和生成更现代的高密度软盘。这是一个问题，因为高密度软盘驱动器可以读取双密度软盘，反之则不行。由于CAC收藏中的大多数软盘是高密度的，我们需要找到另一种驱动器。

后来，Leontien给大学技术邮件列表发了电子邮件，询问是否有人愿意捐赠或借给大学图书馆一个高密度软盘驱动器。总共有六个驱动器被捐赠了。

然而，与大多数旧硬件一样，这些捐赠的驱动器被存放在阁楼或箱子里很长时间没有使用，因此它们都无法生成可靠的磁盘映像（使用测试盘进行测试，没有使用任何收藏资料来进行这一过程！）。更糟糕的是，在这些软盘驱动器制造的时候，没有任何标准存在，这意味着剑桥大学图书馆数字保存团队获得的所有软盘驱动器都是不同的。所有这些驱动器都需要清洁，但大多数还存在其他问题，比如零部件损坏。[计算历史中心](https://www.computinghistory.org.uk/)（CCH）的志愿者们非常友善地建议我们访问他们的收藏，查看其中的多种不同驱动器。在与志愿者们交流并观看他们如何清洁和修复驱动器之后，我们终于得到了一个能正常运行的驱动器。现在是时候进行更多的影像工作了！

*剑桥大学图书馆数字保存实验室中的六个5.25英寸软盘驱动器。可以注意到它们都略有不同。*

在CAC，克里斯已经设置了CAC的GreaseWeazle和一个3.5英寸软盘驱动器。我们假设可以轻松将从CUL带来的5.25英寸驱动器更换成3.5英寸驱动器，但情况并非如此，这是我们的第一个障碍：我们需要一个不同的电源来源。GreaseWeazle可以通过从其USB连接传递电源来为较小的驱动器供电，但对于较大的磁盘驱动器和用于驱动器的电源供应器则不够。在CUL，用于5.25英寸驱动器和GreaseWeazle设置的电源是由计算机本身的电源供应器供电的，这不能轻易地移到CAC。

最后，在丘吉尔的计算办公室的帮助下，我们想出了一个临时的但完全功能的解决方案。丘吉尔有一个[USB转SATA/IDE适配器](https://www.amazon.co.uk/FIDECO-Adapter-External-Converter-inches-Black/dp/B0919SF9CP)，来自[之前类似项目的硬盘读取](https://twitter.com/ChuArchives/status/1539947300805644288)，这个适配器有连接器可以为这些硬盘供电。这样就可以给5.25英寸驱动器供电而不用连接硬盘。

*CAC的软盘驱动器设置。*

我们开始使用CUL的测试软盘作为基准，该软盘在设置时在相同驱动器上读取良好，以确认我们已正确设置了所有内容，并在尝试了几种不同的命令表达后，确实得到了相同的结果。CUL测试软盘是一个特别好的测试例子，使用高密度可写盘上的双密度格式写入，并在高密度机器上读取，我们能够从中读取flux，并使用[Greaseweazle用户指南](https://github.com/keirf/greaseweazle/wiki/Yann-Serra-Tutorial)和内置配置正确地转换为连贯的图像。

*在高密度软盘驱动器上制作的双密度软盘的原始flux流。*

*使用Greaseweazle在软盘配置上转换的原始flux流。这个转换对于能够读取和访问软盘上的实际文件至关重要。*

我们使用GreaseWeazle软件生成了一个.raw的flux拷贝（使用‘read’命令而没有任何格式参数），然后使用[HxC软驱模拟器](https://hxc2001.com/download/floppy_drive_emulator/)来检查这个flux。这个工具可视化flux以检查捕获是否正确，并确定如何将flux处理成图像，比如计算轨道和扇区的数量以及它们的排列方式。[这个列表](https://en.wikipedia.org/wiki/List_of_floppy_disk_formats)对确定软盘配置非常有帮助。

**ICL软盘**

在设置和测试了5.25英寸驱动器之后，我们开始对磁盘进行成像。首批磁盘是来自[索姆斯男爵文件](https://archivesearch.lib.cam.ac.uk/repositories/9/archival_objects/480720)的ICL磁盘。当我们首次尝试成像这些磁盘时，我们能够使用FC5025软驱控制器创建镜像，并通过[十六进制编辑器](https://mh-nexus.de/en/hxd/)确认它们包含数据，但是镜像过程报告说每个扇区都有错误，这意味着结果不完整或格式严重错误（或可能两者兼而有之！）。成像第一张磁盘后，我们在HxC软驱仿真器上查看它，确认这些结果确实异常；我们认为它们的15扇区/35磁道设置与GreaseWeazle或FluxEngine格式列表上的任何内容都不匹配。

然而，在读取了几张磁盘后，我们注意到磁盘中间在仿真器中显示为红色，我们知道这可能是脏磁盘的迹象；这样的磁盘还会导致将污垢转移到驱动器本身的物理读头上，从而导致将来的磁盘读取效果差。由于多张磁盘的中间部分显示缺失数据，我们决定使用棉签和异丙醇清洁驱动器读头。重新成像有错误的磁盘时，读取效果明显更加干净，但我们很快又遇到了这个问题，并决定转而尝试其他收藏中的磁盘，看看它们是否表现更好。

**WANG磁盘**

我们继续进行来自[罗伯特·爱德华兹爵士文件](https://archivesearch.lib.cam.ac.uk/repositories/9/resources/1546)的WANG磁盘。这些读取良好，看起来有16扇区/40磁道，但同样不匹配GreaseWeazle列表上的任何内容（虽然这些数字与acorn.adfs.160匹配，但使用的是[FM编码](https://en.wikipedia.org/wiki/Frequency_modulation_encoding)，而不是这些磁盘上的MFM编码）。

WANG磁盘的Flux流

**4个以前无法读取的磁盘**

这些来自[尼尔·金诺克文件](https://archivesearch.lib.cam.ac.uk/repositories/9/resources/1673)的磁盘被证明是使用ISO FM编码的单密度磁盘，与迄今为止使用ISO MFM编码的所有其他磁盘不同，这解释了为什么FC5025控制器无法读取和成像它们。我们认为它们包含10扇区/80磁道，并且是[橡树DFS格式](https://beebwiki.mdfs.net/Acorn_DFS_disc_format)，正如在[FluxEngine网站上描述的](https://cowlark.com/fluxengine/doc/disk-acorndfs.html)。

这里需要指出的是，物理磁盘上的标签与其内容不符合，磁盘被标记为双面、双密度，而acorndfs格式为单密度，能够使用磁盘的两面，但作为单独的卷。这导致我们重新评估其他磁盘是否应被视为一个或两个卷，这无疑是我们将来考虑的事情。

*作为双密度磁盘标记的读取为单密度的软盘*

**下一步和教训**

我们现在已经对大多数磁盘进行了原始流通道的采集，并且已经能够在模拟器上查看它们。我们对所有非ICL磁盘的复制可靠性相对有信心，但是有5个ICL磁盘我们尚未完成图像化，我们知道其中一些复制品是在传感器脏的情况下制作的。不过，我们已经能够看到它们的格式化方式，而WANG和ICL磁盘都足够不寻常，以至于它们不会列在[GreaseWeazle](https://github.com/keirf/greaseweazle/wiki/Yann-Serra-Tutorial)或[FluxEngine](https://cowlark.com/fluxengine/#which)的配置文件列表中。幸运的是，有[一些可用的资源](https://wiki.techtangents.net/wiki/Floppy_Disk_Imaging)可以用于制作我们自己的磁盘配置文件，用于更晦涩的格式。

需要重新对脏的ICL磁盘进行图像化；考虑到我们遇到的问题，下次我们可能会在读取每个单独的磁盘之间清洁驱动器，这在实际操作中只是有可能，因为总共只有19个！

此外，我们可以测试由理查德·雷哈恩（Richard Lehane）创建的**WANG阅读工具**；我们相信我们的磁盘格式略有不同，但我们仍然希望这可能会提供一些实际的、离散的文件。

在教训方面：我们强烈建议投资于类似的GreaseWeazle用于此类工作。它价格低廉且高度可定制，正是我们所需的。Leontien成功地向周围寻找了5.25英寸软盘驱动器，并且我们不会感到惊讶，如果对其他机构也适用。在丘吉尔学院没有直接可用的东西时，Chris将调查在eBay或类似网站购买的驱动器是否能直接工作或者能够清洁至可工作状态。我们建议使用高密度磁盘驱动器，因为它可以读取所有类型的5.25英寸软盘。但请确保在插入任何收藏材料之前清洁和测试驱动器。
