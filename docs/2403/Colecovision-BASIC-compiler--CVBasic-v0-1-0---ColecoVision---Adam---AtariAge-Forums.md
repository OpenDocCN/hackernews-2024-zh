<!--yml

category: 未分类

date: 2024-05-29 11:59:27

-->

# Colecovision BASIC 编译器：CVBasic v0.1.0 - ColecoVision / Adam - AtariAge 论坛

> 来源：[https://forums.atariage.com/topic/362182-colecovision-basic-compiler-cvbasic-v010/](https://forums.atariage.com/topic/362182-colecovision-basic-compiler-cvbasic-v010/)

这是个好消息！但我确实有几个问题：

1) 我知道这款新的编译器在开发初期，但你们是否计划提供一个瓦片/精灵编辑器或声音/音乐编辑器？如果你们目前还没有考虑过这些，也许可以与那些已经制作过这类编辑器的AtariAge社区成员合作（主要是为了数据导入/导出功能）？

2) 虽然编程语言是项目的核心部分，但另一个非常重要的部分是调试器。你们是否计划在CoolCV中添加一个合适的调试器（包括断点、RAM使用监视器、中断监视器和下一个VBLANK时刻监视器）？在为ColecoVision编程时的一个困难之处在于你无法真正控制VBLANK NMI发生的时间，如果你的代码在VBLANK之间执行了太多操作，会导致图形损坏，甚至完全的运行时崩溃。

3) 你的样本Basic代码似乎没有为指定编译设置提供任何条款（即游戏是否使用分段切换（32K、MegaCart、Activision PCB），是否需要超级游戏模块，是否应该支持（或忽略）Roller Controller的中断等）。

4) 你们是否会在语言中提供内置函数？例如，一个用于检测SGM的标准函数，或者在启动/复位时清除VRAM的函数，或者将声音芯片初始化为“静音”模式的函数，这样程序员就不需要编写自己的例程了？

这些只是我脑海中的问题，我确信还能想出更多。 ![:P](img/d030feb833ae20437add57e38619cf1c.png ":P")

**Edited February 28 by Pixelboy**
