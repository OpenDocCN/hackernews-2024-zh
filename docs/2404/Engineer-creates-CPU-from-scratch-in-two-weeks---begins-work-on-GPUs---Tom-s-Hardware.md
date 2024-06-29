<!--yml

category: 未分类

date: 2024-05-27 13:12:57

-->

# 工程师在两周内从零开始设计CPU —— 开始工作于GPU | 汤姆硬件

> 来源：[https://www.tomshardware.com/pc-components/cpus/engineer-creates-cpu-from-scratch-in-two-weeks-begins-work-on-gpus](https://www.tomshardware.com/pc-components/cpus/engineer-creates-cpu-from-scratch-in-two-weeks-begins-work-on-gpus)

一位工程师分享了他从零开始设计CPU的经验，仅用了两周时间，“没有任何先前经验”。在这短暂的期间内，[亚当·马吉穆达尔](https://twitter.com/MajmudarAdam/status/1778235769150423121) 声称他学会了[芯片架构](https://www.tomshardware.com/tech-industry/artificial-intelligence/jim-keller-responds-to-sam-altmans-plan-to-raise-dollar7-billion-to-make-ai-chips)的基础知识，掌握了[芯片制造](https://www.tomshardware.com/news/new-us-fabs-everything-we-know)的精要，并使用[EDA工具](https://www.tomshardware.com/news/ai-tools-take-chip-design-industry-by-storm-200-chips-tape-out)准备了他的第一个完整芯片布局。他“极速运行芯片堆栈”的下一个步骤是从零开始设计GPU。完成后，该项目将通过[马修·文](https://twitter.com/matthewvenn)的TinyTapeout 6进行生产。

我们此前已经报道了一些爱好者的[DIY CPU设计](https://www.tomshardware.com/news/man-builds-own-silicon-chip-at-home)，以及[DIY GPU项目](https://www.tomshardware.com/pc-components/gpus/new-open-source-gpu-is-free-to-all-supports-modern-windows-software-stack-runs-on-an-fpga-with-custom-pcb)。然而，这些壮举中的一些耗费了参与者多年的空余时间。马吉穆达尔必须是在度假并把所有的空余时间花在这个“极速运行”项目上，才能达到他从零开始的成就。

这位新兴的芯片设计师自称是一家Web3开发公司的创始工程师之一，概述了他迄今为止在这一探索中取得的进展。您可以通过上述嵌入的推文点击阅读所有导致当前GPU关注的步骤。我们还列出了截至目前完成的极速运行步骤。

+   学习芯片架构的基础知识——深入理解是关键基础

+   学习芯片制造的基础知识——材料、晶片准备、图案制作和封装

+   通过逐层制作CMOS晶体管开始电子设计自动化

+   在Verilog中创建我的第一个完整电路——“我第一次使用软件编程硬件的经历。”

+   在我的电路中实施仿真和形式验证

+   设计我的第一个完整芯片布局——使用开源EDA工具OpenLane进行设计和优化

图片 1/5

（图片来源：亚当·马吉穆达尔）

（图片来源：亚当·马吉穆达尔）

（图片来源：亚当·马吉穆达尔）

（图片来源：亚当·马吉穆达尔）

（图片来源：亚当·马吉穆达尔）

如我们在简介中提到的那样，Majmudar现在面临的主要挑战是自研GPU。他知道这将是一项艰难的任务，并承认初步调查后发现这比预期的更为困难([来自Twitter的记载](https://twitter.com/MajmudarAdam/status/1778235788880420870))。这位起步中的芯片设计师解释道，网上没有足够的在线学习资源可以用于构建GPU。这位工程师发现：“因为GPU公司都在努力保守各自的秘密，大部分GPU架构数据都是专有的和封闭源的。”

尽管存在这一挑战，Majmudar表示，大型GPU制造商的保密行为，让此项目的这一部分“对我来说更加有趣”。值得注意的是，Anthropic的Claude Opus AI工具在这一GPU设计阶段非常有用。“我向Claude提出了我如何让各个组件工作的想法，然后有些如何正确实施的思路就被引导了出来，随后我就可以通过开源库进行验证。”工程师解释道。然而，他观察到“如果我查找一些公开信息，结果一片空白，这表明了如何实施的细节被隐藏得多深。”

在成功完成三段中的其中三段、五段挑战中的三段之后，所表达的对于GPU的担忧可能会让读者感到，Majmudar 可能遇到了障碍、阻碍甚至是一堵无法逾越的墙。但这似乎并非真实情况，因为他乐观地预测，他的GPU设计将在“接下来的几天内”出货，并寄送一个简化版进行原型制作。

留意这位工程师的后续更新可能非常值得。然而，我们知道从提交工作到项目如TinyTapeout的生产运行之间，中间可能会经过相当长的时间。例如，制作了[里克滚动ASIC](https://www.tomshardware.com/maker-stem/rickroll-asic-heralded-as-a-world-first-this-chip-is-never-gonna-let-you-down)的制作者表示，他的设计提交后到收到硅片这段时间共有九个月。请注意，[TT06](https://tinytapeout.com/runs/tt02/145/)将在如今后仅剩八天结束。

获取Tom's Hardware的最新新闻和深入评测内容，直接送达您的邮箱。
