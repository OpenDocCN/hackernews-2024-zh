<!--yml

类别：未分类

日期：2024-05-27 12:55:03

-->

# Rickroll 梗被永久记录在定制 ASIC 中 • The Register

> 来源：[https://www.theregister.com/2024/04/01/rickroll_meme_asic/](https://www.theregister.com/2024/04/01/rickroll_meme_asic/)

一个专门用于显示臭名昭著的 Rickroll 梗的 ASIC 已经出现，还有其他 164 个各种功能。

该项目是 Matthew Venn 的 [Zero to ASIC Course](https://zerotoasiccourse.com/) 的产品，该课程为有意成为芯片工程师的人提供了“学习设计自己的 ASIC 并进行加工”的机会。自 2020 年以来，Zero to ASIC 已接受了几个设计，这些设计被合并到一个称为多项目晶片（MPW）的单一芯片中，这是一种节省成本的措施，因为为每个设计制造一个芯片将会非常昂贵。

Zero to ASIC 有两个系列的芯片：MPW 和 Tiny Tapeout。MPW 系列通常只包括少数几个设计，比如 2023 年 1 月提交的 MPW8 上的四个设计。相比之下，最初的 Tiny Tapeout 芯片包含了 [152 个设计](https://tinytapeout.com/runs/tt01/)，而 Tiny Tapeout 2（去年十月推出）则有 165 个设计，不过后来可以增加到 250 个。

在 [165 个设计](https://tinytapeout.com/runs/tt02/) 中，特别引人注目的是其中一个：设计 145，或由工程师和 YouTuber Bitluni 制作的 [Secret File](https://tinytapeout.com/runs/tt02/145/)。他的 Tiny Tapeout ASIC 的秘密文件设计用于播放 Rick Astley 的音乐视频《永不放弃你》，也被称为 Rickroll 梗。

Bitluni 是 Tiny Tapeout 2 项目的后期参与者，在提交截止日期前仅仅被邀请了三天。最初他只制作了一个视觉持续控制器，这个设计被修改了两次，总共有三个设计。

“最后，我还剩下几个小时，我想也许我应该上传一个梗项目，” Bitluni 在他记录他的 ASIC 之旅的 [视频](https://www.youtube.com/watch?v=DdF_nzMW_i8) 中说道。他选择的梗当然是 Rickroll。甚至可以说这是一个彩蛋。

然而，鉴于每个设计总共有 250 个图，既没有图形处理器也没有要呈现的文件，短小的音乐视频 GIF。最终，这必须从 217 千字节缩小到不到半个千字节，使其输出看起来类似于 1977 年的 Atari 2600 上的游戏。

访问 Rickroll 渲染处理器和其他设计并不简单。Bitluni 创建了一个定制电路板来安装 Tiny Tapeout 2 芯片，创建了一个可以插入能够选择 ASIC 上特定设计的主板的设备。不幸的是对于 Bitluni 来说，他的第一版 PCB 上有一个设计错误，他不得不进行修正，但修订版起作用了，并且能够通过 VGA 端口在硬件上显示 Rickroll GIF。

Tiny Tapeout 2中的其他设计包括一个FPGA、一个4位CPU以及一个能够估算到千位小数的π的计算器。这些芯片是由Efabless的ChipIgnite航天计划使用Skywater 130nm开源PDK制造的。

尽管这些设计没有严肃的实际功能，它们可以作为志向成为芯片设计师的人的有用概念证明。制造一片芯片的成本非常昂贵，对于普通学生或爱好者来说是不可及的。

虽然本质上属于众筹，但Zero to ASIC课程的最低参与成本是650美元，远高于能够编程执行更复杂任务的100美元FPGA。这显然是一个很大的溢价，只是为了让芯片被认为是定制的。

还值得一提的是，提交的设计本身是开源的，任何人都可以检查，这意味着对芯片和硬件语言的低级设计感兴趣的人可以深入了解和查看所有内容。

虽然Tiny Tapeout 2刚刚问世，但它的后继产品已经在紧锣密鼓地进行中。实际上，Tiny Tapeout 3到6的提交已经结束，预计全年都将推出。Tiny Tapeout 7、8和9的提交现已开放，Tiny Tapeout 7计划于12月发布，而8和9则计划在2025年中期发布。

由于[Tiny Tapeout 3](https://tinytapeout.com/runs/tt03/)的提交是公开的，我们可以看到它也不会免于梗文化的影响。第16设计，Bad Apple，在压电扬声器上播放基于东方Project的Bad Apple歌曲，而第77设计则是一种AI减速器，"保证能减慢您的AI模型训练速度"。Tiny Tapeout 2的Rickroll设计也被移植到Tiny Tapeout 3上。®
