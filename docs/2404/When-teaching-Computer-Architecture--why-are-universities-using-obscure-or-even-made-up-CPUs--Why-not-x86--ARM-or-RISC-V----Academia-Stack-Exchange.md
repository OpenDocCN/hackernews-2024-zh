<!--yml

category: 未分类

date: 2024-05-27 13:08:02

-->

# 在教授计算机架构时，为什么大学使用晦涩甚至是虚构的CPU？为什么不使用x86、ARM或RISC-V？- Academia Stack Exchange

> 来源：[https://academia.stackexchange.com/questions/209300/when-teaching-computer-architecture-why-are-universities-using-obscure-or-even](https://academia.stackexchange.com/questions/209300/when-teaching-computer-architecture-why-are-universities-using-obscure-or-even)

为了辩护教授们，如终身教授和访问/兼职教授，以及助教（T.A.s），创建/更新演示文稿集合、讲座笔记、实验室/实验分配和课程项目，用于计算机架构课程甚至相关的计算机系统课程，需要付出大量工作。

除了准备他们所教授的课程（或者你喜欢的讲座）、管理的实验室会话和他们领导/教授的辅导/教程会话外，还需要创建/更新和评分任务、课程项目和考试。

有必要为计算机架构课程选择一个有效的指令集架构（I.S.A.）的工具链。商业/专有的工具链，如商业电子设计自动化（EDA）软件，需要助教的技术支持。开源EDA软件可以缓解问题，但仍需要更多行业支持。如果许多招聘经理创建包含开源EDA工具和最近创建的硬件描述/构建语言（HDLs/HCLs）的工作描述，如Chisel HDL、PyMTL和PyRTL，许多助教将支持使用这些开源EDA工具、HDLs和HCLs。

使用x86-64，包括英特尔64（英特尔的x86-64版本），作为教授计算机架构的I.S.A.是一场噩梦。基于复杂指令集计算机（CISC）的I.S.A.比基于精简指令集计算机（RISC）的I.S.A.更难教授。因此，使用x86 I.S.A.来教授32位处理器架构也不会更好。

ARM I.S.A.是一种RISC I.S.A.，但仍包含足够复杂的指令，使得教学变得烦人。

这导致许多教授在ARM I.S.A.和MIPS I.S.A.之间进行选择。ARM I.S.A.越来越多地用于嵌入式系统中，而MIPS I.S.A.则没有被采用到新产品的嵌入式系统和其他计算机系统中。

PicoBlaze I.S.A.来自Xilinx，Inc.，并配备了一个支持的工具链，帮助人们学习在其FPGA板上设计处理器，并用汇编语言和编译器开发软件。

这使得RISC-V I.S.A.成为一个不错的选择，可以让学生在在线共享他们的项目的同时展示他们的技能水平。RISC-V国际组织（之前称为RISC-V基金会）有一个网页分享了针对RISC-V的大学教学资源，以及用于模拟/仿真RISC-V处理器和汇编/编译计算机程序的相关工具链。该网页还包括用于验证RISC-V处理器实现的RTL级别的基准程序。

但是，教师和助教们仍然需要花时间来熟悉新的I.S.A.、工具链或者EDA工具，并且更新新I.S.A.的课程资料。如果他们能够得到学术部门的资助来做这些工作，这个问题就可以得到解决。如果你想加快这个过程，你、一些你认识的富人，或者通过众筹来捐款给你的学术部门，都可以加快这个过程。
