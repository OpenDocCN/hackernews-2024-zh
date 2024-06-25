<!--yml

分类：未分类

日期：2024年5月27日14:52:15

-->

# 尼古劳斯·维尔特的一些回忆 | 语言与代码

> 来源：[https://odersky.github.io/blog/2024-01-04-post.html](https://odersky.github.io/blog/2024-01-04-post.html)

# 尼古劳斯·维尔特的一些回忆

2024年1月4日

2024年1月

尼古劳斯·维尔特在今年年初的第一天去世了。他是计算机科学最重要的先驱之一，可能是在保持严格的科学和工程原则的同时产生了最大实际影响的人。尼古劳斯自称为工程师，他也是如此。作为一个工程师，他将科学发现和原则应用于有用产品的开发中，其中包括编程语言、编译器和其他系统软件。但由于当时这部分科学还不存在，他不得不自己创造大部分科学。

我很荣幸能够作为他的博士生与他合作，并从中学到了很多。在这篇笔记中，我想写写尼古劳斯影响了我的工作和编程方法的一些方式。 

实际上，如果不是因为尼古劳斯，我很怀疑我是否会一直从事计算机科学。我的第一个学位是数学。我对计算机科学感兴趣是因为我对编译器和编程语言很着迷。我最深入欣赏的第一种语言是 Pascal。它简单而清晰，从设计和实现的角度都很容易理解。然后，我和同学彼得·索利希（Peter Sollich）一起偶然发现了 Pascal 到 P-Code 编译器的源代码清单。一个完整的 Pascal 编译器可以用大约5000行易于理解的代码编写，这让人惊讶。这启动了我们为一款新的微型计算机 [Osborne 1](https://en.wikipedia.org/wiki/Osborne_1) 编写自己的 Pascal 编译器的项目。这是在 Turbo Pascal 发布前大约一年左右。

在项目进行到一半时，彼得在苏黎世联邦理工学院发现了另一对细长的黄色研究报告，出自尼古劳斯·维尔特（Niklaus Wirth）之手。其中一份描述了 Modula-2 语言，另一份描述了实现该语言的 Lilith 计算机的指令集。我们立即被这两种语言和指令集所吸引，因为它们既简单又非常紧凑。当时，由于我们的计算机只有54千字节的可用内存，节省内存至关重要。因此，我们将编译后的源语言切换为 Modula-2，中间代码切换为 Lilith 代码的变体。这个编译器最终成为了[ Turbo Modula-2](https://github.com/Oric4ever/Turbo-Modula-2-Reloaded)，适用于8位计算机。我们将其卖给了Borland，但该公司没有用自己的名字发布它。

在体验并与维尔特的美妙发明共事后，我清楚地意识到我想继续攻读编程语言的博士学位，并向他的研究组申请。我非常高兴我的申请被接受了，于是我在1986年开始在 ETH 工作。

维尔特小组的工作非常特别。我只有在离开后才意识到工作场所是多么不同。你知道，我们所使用的一切都是“自制”的东西，但同时又远远超越了当时的标准。从计算机开始：一个屏幕能够在位图显示器上显示完整的A4页面，还有一个用于与之交互的鼠标。这至少比具备这些特征的商用计算机出现要早5年。接着是字体。因为没有位图显示器，所以也没有适用于这种显示器的字体。因此，小组中的一名成员是专业的字体设计师，另一名成员是字体设计工具的设计师！然后是操作系统和编程工具。我们有一个简单但功能强大的文本编辑器，可配置为非常高效的集成开发环境。我们还有一个简单而漂亮的调试器，显示了一组平铺窗口中的线程转储和完整的内存探索。我忘了提到，当然窗口系统也是自家开发的，一切都是。

现在你可能会认为这些事情本来应该有一支开发团队来完成。但事实并非如此，这是尼古拉斯·维尔特（Niklaus Wirth）和6名博士生以及一些研究员在于尔格·古特克内希特（Jürg Gutknecht）和彼得·默森伯克（Peter Mössenböck）的相关团队中进行的。最好的一点是，如果你有问题或建议，你只需走进隔壁的办公室，与写软件的人交谈即可。

所有这一切都是因为尼古拉斯以他的无情简单主义方式引领着。简单是强制性的。每个特性都必须被证明是既至关重要又非常紧凑易实现的。众所周知，只有当编译器启动速度提高时才会添加编译器优化。尼古拉斯在他自己的编码和教导我们方面引领着道路。

我认为尼古劳斯所从事的语言序列是冯·诺伊曼语言的一种顶点。在他的论文项目EULER之后，他致力于ALGOL-W，这是Algol 60的一种拟议的继任者，得到了计算机科学的其他领先人物的支持，比如托尼·霍尔或艾德斯格·迪科斯特拉，但最终没有被接受为Algol的官方下一个版本。接下来是Pascal，在教学和个人电脑上取得了巨大成功。在Pascal之后是Modula，然后是Modula-2，这是我个人最喜欢的语言。它增加了一些Pascal的模块化概念，允许开发人员团队在一个共同的代码库上工作，以及比15年后的Java更干净和更强大的并发概念。维尔斯语言的最后一种是奥伯龙，它增加了一种极简的构造，支持面向对象的可扩展程序的领域，并从Modula-2中删除了相当多的特性。因此，它甚至比Modula-2更容易实现，因此可以用一个小的代码库开发完整的带有GUI和编译器的操作系统，并在一本书中描述它。

我自己的论文是关于开发一种新类型的属性文法，并在其中编写奥伯龙的规范。在我在ETH的时候，我发现并爱上了函数式编程，这与我们在ETH所做的事情非常不同。但我从尼古劳斯那里学到的风格和价值观至今仍然伴随着我，我非常非常感激我能亲身经历它们。

我记得几件具体的事情：

+   尼古劳斯不喜欢委员会，可能是因为他在Algol 68委员会上的经历。我被邀请成为Modula-2的ISO标准化委员会的一部分。维尔斯告诉我我可以去，但明确表示他不认为这是一个值得的努力。他自己也不会考虑出席。我参加了委员会的几次会议，然后就退出了。

+   尼古劳斯将自己的角色大多视为先驱和研究员，而不是语言社区的领导者。在我到达苏黎世之前不久，Modula-2曾经有过一小段辉煌时刻，当时当时领先的个人电脑杂志*Byte*专门介绍了它。如果得到更多支持，Modula-2可能已经成为一种广泛应用的系统语言，它绝对值得这样做。还有一个后继语言提案叫做Modula-3，由卢卡·卡尔德利、格雷格·尼尔森和其他DEC SRC的人开发。这也是一种非常干净而优雅的语言设计，比Modula-2更雄心勃勃、更强大。如果尼古劳斯加入了这个努力，谁知道，它可能已经成为C++的流行继承者和竞争对手。但他已经在着手他的下一个项目，奥伯龙。

Niklaus比我认识的任何其他人都更深入地结合了理论和实践结果。他是结构化编程的奠基人之一，开创了程序细化的先河，这一点在他的书“数据结构+算法=程序”中得到了美妙的阐述。然而，他所做的一切都是为了获得更好的实用抽象，帮助他设计干净的操作系统、编译器和其他工具，所有这些只需要几千字节的内存。与C等其他系统语言不同，Wirth的语言从未在拥有紧密抽象方面妥协。我相信这是他的永恒遗产：理论和抽象如何在日常编程中*有所帮助*，但前提是必须做得对，专注于简单而毫不留情。