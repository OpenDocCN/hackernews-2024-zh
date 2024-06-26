<!--yml

category: 未分类

date: 2024-05-27 14:34:27

-->

# 微控制器的早期历史：来自德克萨斯州的小芯片（第一部分）

> 来源：[`thechipletter.substack.com/p/tiny-computers-from-texas`](https://thechipletter.substack.com/p/tiny-computers-from-texas)

TMS 1000C - 作者：Pauli Rautakorpi - 自己的作品，CC BY 3.0，https://commons.wikimedia.org/w/index.php?curid=52406652

在圣诞节期间，我们的儿子得到了很多新玩具。其中他最喜欢的是一把塑料吉他，吉他的指板上有按钮，还有许多灯光，甚至还有一个 whammy bar。非常有趣。他一岁多的时候还太小玩计算机，但他的许多玩具，包括吉他在内，都有内置的计算机。但是，当然，我们通常不称它们为计算机。相反，它们被称为“微控制器”。

一想就知道，微控制器无处不在：在我们的家中，洗衣机、冰箱和微波炉中，在我们的汽车中，以及许多其他地方。这些微控制器中的一些是基于几十年前的设计的。我不敢冒险打开玩具吉他 - 他会非常不高兴 - 但如果吉他内部的微控制器使用的是 20 世纪 70 年代首次开发的 8 位架构，我也不会感到惊讶。在儿子愉快地玩着他的新玩具吉他的推动下，现在似乎是深入了解无处不在但通常看不见的微控制器历史的好时机。

在我们深入了解微控制器的历史之前，我们需要定义什么是微控制器。维基百科将微控制器定义为单个集成电路上的“小型计算机”。这将其与微处理器区分开来，后者仅是芯片上的中央处理单元，并需要额外的集成电路来创建功能性计算机，“SoC”或“系统芯片”是指当计算机不被认为是“小型”时所使用的术语。

或许让人惊讶的是，最早的微控制器出现的时间与第一批微处理器大致相同。人们可能会认为，在单一集成电路上构建完整计算机比仅仅构建一个 CPU 要困难得多。然而，最早的微处理器通常用于需要非常少量 ROM 或 RAM，并且高度价格敏感的设计中。工程师们很快就能够克服在处理器旁边添加少量内存和输入/输出电路的技术挑战。而系统设计者由于使用微控制器而带来的成本降低的潜在商业利益是相当可观的。因此，在 20 世纪 70 年代和 80 年代，微控制器有着庞大的市场，许多领先的半导体制造商都有自己的微控制器产品线，与其微处理器设计一同销售。

微控制器通常比微处理器低调，并吸引的注意力较少，原因有两个。首先，需要保持集成电路芯片尺寸小，同时也要容纳内存和输入/输出，这意味着它们的 CPU 比当代微处理器不太复杂。其次，它们通常用于用户无法阅读或更改其代码的系统中。因此，虽然不同的时代长大了有‘家庭’然后是‘个人’电脑或工作站，熟悉驱动这些机器的微处理器的体系结构，但是微控制器一直被锁在计算器、玩具、汽车和家用电器内部。

随着半导体节点尺寸多年来的缩小，使用现有的微处理器设计作为微控制器的 CPU 部分成为可能。这些 CPU 通常具有更高的性能、更丰富的软件生态系统，并且比之前为早期微控制器专门设计的设计更容易编程，它们取代了早期设计。今天，大多数微控制器设计都包含来自 8 位时代的处理器（例如 Z80、6502）、16 位时代的处理器（例如 68000）或现代 32 位设计（如 ARM Cortex 或 RISC-V）。

这是关于最早期微控制器的系列帖子中的第一篇。我们将看看当时的一些主要设计，它们的历史以及它们的用途。尽管这些设计比早期的微处理器低调得多，但它们在使计算成为无处不在的过程中发挥了关键作用。

在我们讨论真正的微控制器之前，值得讨论一下微控制器的“概念”。在 1960 年代，随着越来越多的组件被塞入集成电路中，很明显很快就可以创建一个‘芯片上的计算机’。

1970 年，吉尔伯特·海特（Gilbert Hyatt）申请了一项“微型计算机体系结构……促进了单一集成电路芯片上的完全集成电路计算机”的专利。换句话说，就是“微控制器”。这项专利花了二十年才被授予。1990 年，《洛杉矶时报》报道道：

> 7 月份，在一场持续 20 年的法律争斗中，美国专利商标局授予 Hyatt 第 4,942,516 号专利，内容是“单片集成电路计算机体系结构”。这项专利震惊了电子界。

洛杉矶时报报道了它与 Hyatt 的对话：

> “我没发明计算机，但我提出了一个非常好的改进，” 52 岁的工作狂 Hyatt 在拉帕尔玛（La Palma）一条死胡同的一栋简朴房子里独自工作时说道。“那个时代我的工作导致了今天的个人电脑。”

海特具有技术专业知识，毕业于加州大学伯克利分校，并在电子公司 Teledyne 工作过，曾尝试开发自己的想法，筹集资金，但最终与投资者发生了矛盾。然而，根据一位前同事的说法：

> 海特是一个固执、不切实际的研究者，没有一项可行的技术，却痴迷于“纸上权利”。

在 1990 年获得专利批准后，海特能够从索尼和其他主要公司那里获得数千万美元的版税。然而，其他公司继续对专利批准提出质疑，直到 1996 年，海特关于微控制器‘发明’的一个核心主张被推翻。

> “在对数万页文件进行了五年的审查后，专利局裁定海特在 1977 年之前并未描述微控制器，这是众多修订案之一。”

海特后来又卷入了另一系列法律案件，这次涉及数百万美元的潜在税务责任和相关损害赔偿，这些案件曾经上诉至美国最高法院，并直到 2019 年才最终解决。

在 2014 年的一次采访中，海特被引用说：

> “我觉得这个行业拿走了我的技术，不仅没有给予我回报，而且直到我的专利开始发放之前都没有给予我认可，”海特说。

然而，事实是他没有获取创建可工作微控制器所需的技术，如果他有可能也不会成功创建。

领导挑战海特专利的是德州仪器（TI）。TI 于 1951 年成立，之前是达拉斯一家为石油行业制造地震设备的公司‘地球物理服务公司’（Geophysical Service Incorporated）经过重组后成立，该公司扩展到了电子领域。1952 年，TI 从 AT&T 的制造部门 Western Electric 购买了生产锗晶体管的许可证，AT&T 的贝尔实验室研究部门发明了该设备。TI 迅速成为半导体和相关技术开发和商业化的先驱。1954 年的第一台晶体管收音机后，1958 年的第一台集成电路以及 1960 年代初期极受欢迎的‘双极型’晶体管—晶体管逻辑或 TTL 集成电路的 7400 系列接连推出。到 70 年代初期，TI 已经成为一个年收入约十亿美元的半导体巨头。

在 20 世纪 60 年代末，德州仪器的注意力转向使用集成电路来缩小‘台式’计算器。早期的电子计算器使用离散晶体管，因此重量沉重且需要交流电源。随着集成电路日益强大，它们成为了显而易见的商业应用。1967 年，在为期两年的项目结束时，由杰克·基尔比领导的德州仪器团队开发了并获得了专利的第一台‘袖珍’计算器，被称为‘Cal Tech’（或许令人惊讶地以加利福尼亚理工学院命名）。‘Cal Tech’将早期计算器中的数百个离散晶体管缩减为仅仅四个集成电路。美国国家历史博物馆有第一台带有日期（1967 年 3 月 29 日）的 Cal Tech 计算器，当时它被赠予德州仪器的总裁帕特·哈格蒂。

TI 将与佳能合作，将 Cal Tech 的技术转化为商业设计，即 1970 年出现的“Pocketronic”（尽管其名称很难放入大多数口袋）。

佳能 Pocketronic - **CC BY-SA 2.0 DEED - https://www.flickr.com/photos/44337451@N00/27065545266**

从 2024 年的角度来看，很难想象 20 世纪 70 年代初期小型电子计算器的出现有多么重要。算术对于许多工作都是基本的，从小型企业的基本会计到复杂的工程。在价格实惠、便携式电子计算器问世之前，这些都需要手工完成（准确性不同）或使用昂贵且笨重的基于晶体管的电子或电机电子桌面计算器。

便宜和更便携的计算器有着巨大的潜在市场。他们的开发很快成为美国和日本主要半导体和电子公司之间的竞争。这反过来又是美国和日本公司之间更广泛竞争斗争的核心部分。有关计算器“竞赛”的更多信息，请参阅本文末尾的进一步阅读中引用的文章《发展便携式电子计算器竞赛的故事》。

集成度的提高意味着很快就能将“Pocketronic”中的四个芯片合并成一个。但是在创造出第一个单芯片计算器的竞赛中，TI 在 1970 年 11 月被德克萨斯的竞争对手 Mostek 打败了，后者生产了 MK6010。《电子》杂志在 1971 年 2 月报道：

> 在生产芯片计算器的竞赛中，明显的赢家已经出炉。德克萨斯州卡罗顿的 Mostek 公司现在为日本的 Busicom 公司生产这样的芯片... 这款面积为 180 毫米的芯片（4.6 毫米）包含了一个四功能 12 位数计算器的逻辑电路——360 个门加 160 个触发器中的 2100 多个晶体管。

MK 6010，像第一颗微处理器 Intel 4004 一样，是在“...与东京的日本计算机公司签订的销售协议下生产的，用于日本 Busicom 计算器系列”。

TI 花了一年的时间做出回应。1971 年 11 月，他们宣布推出自己的“芯片计算器”，即 TMS 1802NC（不要与 RCA 1802 微处理器混淆，后者是一个后来的、无关的设计）。 TMS 1802NC 的特色在于它是可编程的。根据 TI 的营销：

> 该设备是完全可编程的，“程序”只读存储器、定时部分、控制部分和输入/输出译码器可以编程以实现不同的计算特性。这种方法在非常低的成本下提供了最大的设计灵活性。

TI 的计算器芯片的快速集成展示

当 TI 说“完全可编程”时，他们指的是“在工厂可编程”，因为程序在制造过程中固定在“掩模可编程”只读存储器中。根据 1802 的专利：

> 可编程只读存储器和可编程逻辑阵列仅通过在制造过程中更改金属-绝缘体-半导体集成系统实施中的栅极绝缘体掩模来修改。

换句话说，计算器设计师可以支付 TI 来修改此程序以适应他们自己的产品，但对于最终用户来说，程序是固定的。

当我们将其与同样在 1971 年 11 月推出的英特尔 4004 微处理器进行比较时，微控制器方法的成本优势很快就显现出来了。4004 的价格为 60 美元，需要多个支持芯片才能创建计算器。相比之下，1802 以 10,000 个以上的数量售价为 20 美元（尽管单个零件的价格高达 150 美元）。4004 更为复杂，但价格差异使 1802 成为一个引人注目的设计。

TI 在 1802 的 20 美元价格点上采取了积极的态度。正如 Ken Shirriff 所指出的，1802 的晶片尺寸比英特尔 4004 要大得多。由于晶片更大，1802 能够使用 7000 个晶体管（尽管请参见有关晶体管数量的脚注），而 4004 仅使用 2300 个，MK6010 仅使用 2100 个。

根据领导 1802 早期开发的 TI 工程师 Gary Boone 的说法，该项目的起源并不是为了降低成本或与 Mostek 竞争，而是为了充分利用 TI 可用的工程资源，并且已经专注于为多个 TI 计算器客户开发定制集成电路：

> 具体要求在细节上有所不同，但原则上和整体功能上几乎是相同的。所以，你会想，“我厌倦了这样做。我工作时间很长。我的家人不开心。我必须找到更好的方法来做这件事。”
> 
> … 这就是特别是 TMS 100 微控制器芯片的起源：它出于厌倦、高需求和对由部署大量芯片的庞大团队不断增加的共性的愿景。

从他们在大约同一时间建造的 TMX1795 微处理器的布局可以看出 TI 团队的问题。引用 Ken Shiriff 的话：

> … 德州仪器似乎并没有花太多精力在布局上，[英特尔工程师 Stan] Mazor 称之为“相当杂乱的技术”和“把一些模块放在一起”。虽然 4004 和尤其是 8008 被密集地布置，但 TMX 1795 芯片却有大量未使用和浪费的空间。

很可能 TI 团队根本没有时间为任何一个客户完善设计。他们需要做完这件事，然后继续下一个。

与 TI 的 MOS 市场经理丹尼尔·博登（Daniel Baudouin）合作，布恩编制了一个客户需求矩阵，然后着手创建一个尽可能满足这些需求的设计。1802 是结果，由布恩和约六名工程师小组设计。布恩还参与了 TI 1795X 的工作，该设计也为英特尔的第一款 8 位微处理器设计，即英特尔 8008 的架构提供了基础。布恩关于他在 1802 上的工作的证词将成为 TI 推翻海雅特专利的努力的核心。

对于 1802 设计是否符合“微控制器”而不是“固定功能”计算器芯片的问题存在一些争论。正如我们所见，1802 中的程序 ROM 可以被更改，TI 也通过更改该程序提供了变种。TI 为使用 1802 的工程师创建的文档（当时已经更名为 TMS 0102，属于 TMS 0100 系列）显示，它主要设计用于计算器。最重要的是，1802 的输入/输出引脚设计用于驱动计算器显示屏，并支持从简单计算器键盘读取。

然而，1802 的关键专利，美国专利号 4,074,351，着重于其在计算器中的使用，但简要讨论了它在更广泛的设备范围内的使用：

> 例如，计算器系统可以编程执行诸如数字电压表、事件计数、仪表平滑、出租车费计费器、里程表、用于测量重量的称量表等仪表功能。

或许解决争论的一种方法是说 1802 是一个微控制器，但不是一个“通用目的”的微控制器。换句话说，它是一个可编程的芯片上的计算机，但它具有相当狭窄定义的应用程序集。

1802 的设计具有创新性。根据布恩的说法：

> 我们依靠自己的模拟和自己的软件。今天它可能被称为寄存器传输模型模拟。我们模拟了这个芯片上的[指令]处理、程序和数据以及输入和输出资源。基本上允许我们在我们的硅存在之前以高层次测试设计。那是一个飞跃，设计方法上的创新。

TMS 1802NC 的架构 - 源自 TI 的专利申请

《无线世界》报道了 1802 的架构和功能：

> 该 i.c. 包含一个八位 b.c.d. 算术逻辑单元；一个三寄存器 182 位随机存储器；一个 3520 位只读存储器，用于存储程序；以及定时、输出和控制译码器。可以执行浮点或定点运算，并且有数字的自动四舍五入和前导零抑制。算术和控制操作基于 4μs 单相时钟系统。

因此，这是 182 位的 RAM（以 3 x 13 4 位 BCD（二进制编码十进制）寄存器和 2 x 13 位标志寄存器组织），以及 3520 位的 ROM（以 320 条指令 x 11 位指令长度排列）。将这与基于 Intel 4004 的 Busicom 141-PF 计算器中使用的内存进行比较是很有趣的，后者具有 640 位的 RAM 和 8196 位的 ROM。Busicom 提供了更多功能，但即便如此，在 1802 的小内存占用中挤入计算器也是一项令人印象深刻的成就。当时，整个程序，无论是二进制还是汇编语言形式，都被认为足够引人注目，因此在 1802 的关键专利中列出（请参阅本帖子的注释以获取此专利的注释）。

TI 的首款计算器 TI 2500 Datamath - 由 TMS 1802NC 驱动 - 作者 Mister rf - 自己的作品，CC BY-SA 4.0，https://commons.wikimedia.org/w/index.php?curid=46565536

如果模仿是最高形式的恭维，那么 TI 应该感到受宠若惊。首个单片计算器的成就先于 TI 而得，而 Mostek 在此之后推出了 MK 5020，这是 1802 的克隆。

1802 很快被用于来自美国和日本的计算器中。TI 自己在 1972 年 7 月使用它在其自己的第一款计算器设计中，即 TI-2500 Datamath。然而，1802 最有创意的用途之一来自英国。从低成本硬件中挤出额外功能是 Clive Sinclair 的专长。受到 1802 低廉价格的吸引，他委托 Chris Curry 开发一个新的计算器。Curry 讲述了在 1972 年元旦还未醒来的情况下，从德克萨斯州的 TI 工厂收集了三个 1802 样品的故事。

纤薄而低成本 - 由 TMS 1802NC 驱动的 Sinclair Executive - 作者 MaltaGC，CC BY-SA 3.0，https://commons.wikimedia.org/w/index.php?curid=2747195

Curry 在英国返回后，围绕 1802 构建了“Sinclair Executive”计算器。1802 是使用 PMOS 制造工艺制造的，因此比后来基于 CMOS 的设计更耗电。Sinclair 团队通过以 1.7 微秒的脉冲为 1802 提供电源，使其功耗大幅降低，芯片的电容在无电源时保持数据。这将电池寿命延长到约 20 小时，同时使用小型“助听器”电池，这进一步使 Executive 支持了“纤薄”的外形，厚度仅为 9 毫米。

1802 的低廉价格使得 Sinclair Executive 的售价为 £79.95（当时便宜，但相当于 2024 年的超过 £1,000 或 $1,600）。其创新设计和更低的成本使其取得了巨大成功，并使 Sinclair 得以建立一个计算器业务，在其鼎盛时期，每月生产超过 100,000 台计算器。

辛克莱尔（Sinclair）并没有停止使用 TI 计算器芯片进行创新。 1974 年，他推出了 Sinclair Scientific，使用了其中一个 1802 的后继产品 TMS 0805，这个产品通过将科学和三角函数压缩到小小的空间中展示了什么是可能的，仅使用了 320 条指令。肯·希里夫（Ken Shiriff）已经对该程序进行了逆向工程，以展示如何完成，并构建了计算器的模拟器，模拟了肯称之为 TI 的“疯狂”11 位操作码。

在历史的转折中，克里斯·库里（Chris Curry）后来成立了英国橡树电脑（Acorn Computers），这家公司开发了最初的 ARM 架构。后来的 ARM 架构版本成为 32 位微控制器市场上最受欢迎的处理器设计之一。TI 采用 ARM 架构并在诺基亚等公司的早期 3G 手机中使用基于 ARM 的 TI SoC 设计，是 ARM 成功走向市场的关键一步。TI 在 1994 年关闭其位于贝德福德的英国制造工厂时，还向 ARM 提供了未来的 CEO，沃伦·伊斯特（Warren East）。

[TMS 1000 微控制器](https://commons.wikimedia.org/w/index.php?curid=61345074) - 由安东尼奥·马蒂·坎普伊（Antonio Martí Campoy）创作，CC BY-SA 4.0 许可，https://commons.wikimedia.org/w/index.php?curid=61345074

TMS 1802 和 TMS 0100 系列的其他成员在计算器市场取得了成功，但 TI 有机会将“芯片上的计算机”扩展到其低成本将是一个引人注目的建议的其他应用程序中。因此，TMS 0100 系列后面是 TMS 1000 系列，旨在用于计算器和更广泛的控制应用程序。 TMS 1000 于 1974 年推出，采用 4 位处理器架构，成本仅为 2 美元。

让我们来看看 TMS 1000 系列的架构。看到为了使其适合于如此便宜的设备而必须进行的简化是很有趣的。

需要强调的是，与微处理器不同，TMS 0100 和 TMS 1000 系列都无法访问外部 RAM 或 ROM。基本的 TMS 1000 提供了 1,024 x 8 位的 ROM 和 64 x 4 位的 RAM（对于该系列的某些成员，RAM 和 ROM 的数量可加倍），这就是全部。如果您的设计需要比这更多的内存，那么您就需要一个基于微处理器的系统，这会增加额外的成本和复杂性。由于无法访问外部存储器，TMS 1000 封装的大多数引脚可用于读取和写入来自外部硬件的信号，如键盘或显示器。

由于内存非常少，将程序加载到 RAM 中是没有意义的，因此 TMS 1000 系列，就像 TMS 0100 系列一样，都是哈佛架构设计：指令和数据存储器完全分开。

TMS 1000 系列的指令集与更为常见的微处理器设计相比引入了一个有趣的特性。它是“可编程的”。它带有一个“标准”的指令集，我们马上会描述。然而，指令部分是由所谓的“可编程逻辑阵列”实现的，这是另一种形式的“可掩码可编程”ROM，可以为单个客户更改。来自 TMS 1000 系列数据手册：

> 可编程指令解码由指令 PLA 定义。三十个可编程输入 NAND 门解码指令字的八位。每个 NAND 门输出选择 16 个微指令的组合。这 16 个微指令控制算术单元、状态逻辑、状态锁存器和对 RAM 的写入输入。
> 
> 例如，“将 8 加到累加器，结果为累加器”指令可以修改为执行“将 8 加到 Y 寄存器，结果为 Y”的指令。如果修改的指令通过增加指令的效率来节省 ROM 字数，则带走不经常使用的指令是可取的。

当我们看一下 TMS 1000 的架构时，我们可以看到采取了一些措施来减少设计所需的晶体管数量，以降低制造成本。只举几个例子：

+   **分页 RAM**：TMS 1000 有一个 4 位累加器（A）、一个 4 位内存指针寄存器（Y）和一个 2 位寄存器（X），提供任何 RAM 地址的上两位。作为实际内存访问工作的示例，有一条指令“TAM”，它将 A 传输到内存地址 X * 16 + Y。因此，TMS 1000 仅使用 64 个“半字节”或 RAM 实现了分段或分页内存寻址。

+   **分页 ROM**：指令地址也与地址段分割，地址由 6 位程序计数器加上其高 4 位的 4 位“页寄存器”派生。程序保留在一个 64 字节的“页”中，除非它们分支到另一个页面。

最后一个例子可能看起来特别奇怪：

> 程序计数器是作为反馈移位寄存器实现的，而不是作为二进制计数器实现的。这意味着逻辑上连续的程序存储器位置在程序存储器中并不物理上连续。

与二进制计数器相比，反馈移位寄存器所需的门数量较少。逻辑位置到物理位置的映射如下表所示（来自亚当·奥斯本的 4 位和 8 位微处理器手册）。

因此，例如，在位置 #37 执行指令后，要执行的下一个指令在位置 #2F。

我相信对许多读者来说，TMS 1000 的指令集看起来令人恼火！编程的便利性败给了保持成本低廉！尽管如此，我真的很钦佩 TI 团队在寻找保持 TMS 1000 体积小的方法方面的创造力。

TMS 1000 系列很快就进入了各种设备中，TI 经常在自己的产品中使用这些芯片。TI 提供了一个关于如何在简单的“终端”设备中使用 TMS 1000 的原理图。

众所周知，TI 在自己的“Speak and Spell”教育玩具中使用了它。Speak and Spell 会播放一套合成的 200 个单词，并邀请孩子们在键盘上拼写它们。一个 TMS 1000 微控制器控制了玩具的操作，读取键盘并处理显示。然而，语音合成的工作是由另一款 TI 芯片 TMS 5100 完成的，连接着两片 128 kbit ROM 芯片。

原作：Original ‘Speak and Spell’ - 由 FozzTexx 在英文维基百科上，CC BY-SA 4.0，https://commons.wikimedia.org/w/index.php?curid=79574006

TI 微控制器的流行很快吸引了竞争对手。随着在单个低成本集成电路上实现更复杂设计的可能性的增加，TMS 1000 的 4 位架构成为一个不必要的限制。一些竞争对手会构建 4 位设计，而其他人则会转向更强大的 8 位架构。我们将在“微控制器早期历史”的第二部分中看看这些竞争对手和接下来发生了什么。

尽管 TMS 1000 已不再生产，但在我们完全离开它之前，有一个开放源代码的设计实现可在 OpenCores 网站上获取。正如经常发生的那样，流行的技术永远不会完全消失。

* * *

如果你想知道这个玩具吉他看起来和听起来是什么样子，这里是一个短视频。

所有的按钮和“颤音杆”都能用，但它也会播放一个预先编程的曲调。

仔细听的话，你可以听到背景中我一岁的孩子非常兴奋！

* * *

休息一下，参考文献，更多关于早期 TI 微控制器的“进一步阅读”的链接，包括原始文档和关键专利。
