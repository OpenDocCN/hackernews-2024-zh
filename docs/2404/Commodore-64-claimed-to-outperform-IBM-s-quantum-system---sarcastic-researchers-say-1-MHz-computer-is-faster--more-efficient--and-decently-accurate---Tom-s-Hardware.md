<!--yml

类别：未分类

日期：2024年05月27日 13:15:03

-->

# Commodore 64声称超越IBM的量子系统 - 讽刺性研究人员称1 MHz的计算机更快、更高效，而且表现“相当准确” | Tom's Hardware

> 来源：[https://www.tomshardware.com/tech-industry/quantum-computing/commodore-64-outperforms-ibms-quantum-systems-1-mhz-computer-said-to-be-faster-more-efficient-and-decently-accurate](https://www.tomshardware.com/tech-industry/quantum-computing/commodore-64-outperforms-ibms-quantum-systems-1-mhz-computer-said-to-be-faster-more-efficient-and-decently-accurate)

在[2024年SIGBOVIK会议](http://sigbovik.org/)期间发布的一篇论文详细描述了试图在Commodore 64上模拟[IBM的‘量子实用性’实验](https://www.tomshardware.com/news/ibm-unlocks-quantum-utility-127-qubit-eagle)的尝试。这个想法似乎很荒谬 - 将一台40年历史的家用电脑与由[127量子比特‘Eagle’](https://www.tomshardware.com/news/ibm-127-qubit-eagle-quantum-processor)量子处理单元（QPU）驱动的设备进行比较。然而，匿名研究人员得出结论，‘Qommodore 64’的表现更快，更有效，比[IBM](https://www.tomshardware.com/tag/ibm)的自豪产品“在这个问题上表现相当准确”。

在论文开头，研究人员承认他们的‘Qommodore 64’项目是“一个笑话”，但令IBM感到遗憾的是，其量子实用性的证明也建立在摇摇欲坠的基础上，而‘Qommodore 64’团队提出了一些看似令人信服的[基准](https://www.tomshardware.com/tag/benchmark)。当时，IBM的声明引起了一些争议，我们被提醒，这个量子实验在一台普通的[MacBook M1 Pro](https://www.tomshardware.com/reviews/macbook-pro-m1-13-inch-2020)笔记本电脑上模拟仅用了五天。笑话性的量子劣势论文（[PDF链接](https://www.sigbovik.org/2024/proceedings.pdf)，主要部分从第199页开始）将这个实验移植到一台装有更为谦卑的MOS Technology 6510处理器的机器上。

图片1/3

（图片来源：SIGBOVIK 2024）

（图片来源：SIGBOVIK 2024）

（图片来源：SIGBOVIK 2024）

要深入研究量子实用性实验背后的量子理论和数学，请参阅上述PDF链接。然而，简要总结，基于Commodore 64的实验使用了Beguŝić、Hejazi和Chan开发的稀疏Pauli动力学技术，以近似铁磁材料的行为。文章回顾道，IBM曾声称这些计算“使用领先的近似技术在传统计算机上执行起来太困难，难以达到可接受的精度”。然而，并不尽然，正如上文所述，普通笔记本电脑可以获得类似的结果。

匿名的C64用户提供了他们量子击败壮举的一些有趣细节。他们采用了激进的截断和浅层深度优先搜索模型，仅使用了64kB宽敞内存中的15kB，存储在[标志性的Commodore机器](https://www.tomshardware.com/reviews/history-of-computers,4518-27.html)上。与此同时，最终的代码由约2500行6502汇编组成，存储在适配C64扩展端口的插卡中。这段代码由强大的1 MHz 8位[MOS 6510 CPU](https://www.tomshardware.com/news/raspberry-pi-pico-powers-c64-cartridge)处理。C64每个数据点大约需要4分钟时间。（在现代笔记本上测试相同代码，每个数据点大约需要800μs。）

总结来说，研究者们声称‘Qommodore 64’“在每个数据点上比量子设备更快……它在能源效率上更高……并且在这个问题上相当准确。”关于这项研究对其他量子问题的适用性，他们挖苦地暗示“它可能对几乎任何其他问题都不起作用（但话又说回来，现在量子计算机也是如此）。”总体而言，很难确定结果是否完全真实，尽管论文中提供了大量细节并且所链接的研究引用似乎是真实的。

我们知道许多读者是复古计算爱好者，同时也是DIY和创客。因此，知道这篇论文的作者们说他们将提供源代码，以允许其他人复制他们的结果是件好事。然而，他们说源代码只会以三种格式之一提供：“手写在莎草纸上的副本，录制在VHS磁带上的模糊截图幻灯片，或者我亲自通过电话口述给你。”因此，请对这个故事多加一点怀疑。

获取Tom's Hardware最佳新闻和深度评测，直接发送到您的收件箱。
