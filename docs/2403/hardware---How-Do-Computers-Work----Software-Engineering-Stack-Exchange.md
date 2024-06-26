<!--yml

category: 未分类

date: 2024-05-29 12:49:09

-->

# 硬件 - 计算机是如何工作的？ - 软件工程交流平台

> 来源：[https://softwareengineering.stackexchange.com/questions/81624/how-do-computers-work](https://softwareengineering.stackexchange.com/questions/81624/how-do-computers-work)

我将从可能相关的最低级别开始（我可以从更低级别开始，但它们可能太不相关了），从原子开始，到电力，到晶体管，到逻辑门，到集成电路（芯片/CPU），最后到组装（我认为你对更高级别比较熟悉）。

# 起初

## 原子

[原子](http://en.wikipedia.org/wiki/Atom)是由电子、质子和中子（它们本身由[基本粒子](http://en.wikipedia.org/wiki/Elementary_particle)组成）构成的结构。对计算机和电子学最有趣的部分是电子，因为电子是移动的（即它可以相对容易地移动），它可以自由漂浮而不被困在原子内部。

通常，每个原子具有相等数量的质子和电子，我们称之为"中性"状态。事实上，原子可能会失去或获得额外的电子。处于这种不平衡状态的原子分别被称为"带正电"的原子（质子比电子多）和"带负电"的原子（电子比质子多）。

电子是无法被构造或破坏的（在量子力学中并非如此，但这与我们的目的无关）；因此，如果一个原子失去一个电子，附近的另一个原子必须接收额外的电子，或者电子必须释放到一个自由漂浮的电子中。反过来，由于电子是无法被构造的，要获得额外的电子，原子必须从附近的原子或自由漂浮的电子中吸取它。电子的运动机制是这样的：如果附近有一个带负电的原子和一个带正电的原子，那么一些电子将迁移，直到两个原子具有相同的电荷。

## 电力

[电力](http://en.wikipedia.org/wiki/Electricity)只是电子从电子数量非常多的区域流向电子数量非常多的区域的流动。某些化学反应可以产生这样一种情况：我们有一个节点有大量带负电的原子（称为"阳极"），另一个节点有大量带正电的原子（称为"阴极"）。如果我们用导线连接两个带相反电荷的节点，电子将从阳极流向阴极，这种流动就是我们所说的"电流"。

并非所有导线都能同等容易地传输电子，电子在"导电"材料中的流动比在"电阻"材料中更容易。"导电"材料具有较低的电阻（例如电缆中的铜导线），而"电阻"材料具有较高的电阻（例如橡胶电缆绝缘）。一些有趣的材料被称为半导体（例如硅），因为它们可以在特定条件下轻易改变电阻，半导体在某些条件下可能表现为导体，在其他条件下可能表现为电阻体。

电流总是倾向于通过电阻最小的材料流动，因此如果一个阴极和阳极通过两根导线连接，其中一根电阻非常高，另一根电阻非常低，大部分电子将通过电阻较低的电缆流动，几乎没有电子通过高电阻材料流动。

# 中世纪

## 开关与晶体管

开关/触发器就像普通的电灯开关一样，可以放置在两根导线之间，用于切断和/或恢复电流。晶体管的工作原理与电灯开关完全相同，只是它不是通过物理连接和断开导线来连接/断开电流，而是通过改变其电阻来连接/断开电流，这取决于基极节点是否有电流。正如你可能已经猜到/知道的那样，晶体管由半导体制成，因为我们可以改变半导体的性质，使其成为电阻体或导体，以连接或切断电流。

一种常见的晶体管类型，[NPN 双极晶体管](http://en.wikipedia.org/wiki/Bipolar_junction_transistor)（BJT），有三个节点："基极"、"集电极"和"发射极"。在 NPN BJT 中，只有在基极充电时电流才能从发射极流向集电极。当基极未充电时，几乎没有电子能够流过；而当基极充电时，电子就能在发射极和集电极之间流动。

## 晶体管的行为

（我强烈建议你在继续之前阅读[这篇文章](http://www.spsu.edu/cs/faculty/bbrown/web_lectures/transistors/)，因为它可以通过交互式图形更好地解释。）

假设我们有一个连接到电源的晶体管，其基极和集电极连接，然后我们在其集电极附近接线一个输出电缆（见图3在[http://www.spsu.edu/cs/faculty/bbrown/web_lectures/transistors/](http://www.spsu.edu/cs/faculty/bbrown/web_lectures/transistors/)）。

当我们既不对基极也不对集电极施加电流时，因为没有电流可供谈论，根本就不会有电流流动：

```
B   C  |  E   O
0   0  |  0   0 
```

当我们对集电极施加电流但不对基极施加电流时，电流无法流向发射极，因为基极成为高电阻材料，所以电流逃向输出电线：

```
B   C  |  E   O
0   1  |  0   1 
```

当我们向基极施加电流但不向集电极施加电流时，也不能流动电流，因为集电极和发射极之间没有电荷差：

```
B   C  |  E   O
1   0  |  0   0 
```

当我们将电流应用于基极和集电极时，电流通过晶体管流动，但由于晶体管现在的电阻比输出导线低，几乎没有电流通过输出导线：

```
B   C  |  E   O
1   1  |  1   O 
```

## 逻辑门

当我们将一个晶体管的发射极（E1）连接到另一个晶体管的集电极（C2），然后我们在第一个晶体管的基极附近连接一个输出（O）（参见图4中的[http://www.spsu.edu/cs/faculty/bbrown/web_lectures/transistors/](http://www.spsu.edu/cs/faculty/bbrown/web_lectures/transistors/)），然后会发生一些有趣的事情。同时假设我们总是向第一个晶体管的集电极（C1）施加电流，因此我们只是在晶体管的基节点（B1、B2）上玩耍：

```
B1   B2   C1   E1/C2  |  E2   O
----------------------+----------
0    0    1    0      |  0    1
0    1    1    0      |  0    1
1    0    1    0      |  0    1
1    1    1    1      |  1    0 
```

让我们总结表格，所以我们只看到B1、B2和O：

```
B1   B2  |  O
---------+-----
0    0   |  1
0    1   |  1
1    0   |  1
1    1   |  0 
```

*瞧，如果*你熟悉布尔逻辑和/或逻辑门，你应该注意到这正是NAND门。如果你熟悉布尔逻辑和/或逻辑门，你可能也知道NAND（以及NOR）是[功能完备](http://en.wikipedia.org/wiki/Functional_completeness)，即仅使用NAND，你可以构建所有其他逻辑门和其余的真值表。换句话说，你可以仅使用NAND门设计一个完整的计算机芯片。

实际上，大多数CPU是（还是曾经是？）仅使用NAND设计的，因为这比使用NAND、NOR、AND、OR等的组合更便宜制造。

## 从NAND推导出其他布尔运算符

我不会描述如何制作所有布尔运算符，只有NOT和AND门，你可以在其他地方找到其余的内容。

假设有一个NAND运算符，那么我们可以构建一个NOT门：

```
Given one input B
O = NAND(B, B)
Output O 
```

假设有一个NAND和NOT运算符，那么我们可以构建一个AND门：

```
Given two inputs B1, B2
C = NAND(B1, B2)
O = NOT(C) // or NAND(C,C)
Output O 
```

我们可以以类似的方式构建其他逻辑门。由于NAND门是*功能完备*的，因此也可以构建具有多于2个输入和多于1个输出的逻辑门，我不打算在这里讨论如何构建这样的逻辑门。

# 启蒙时代

## 从布尔门构建图灵机

CPU只是图灵机的更复杂版本。 CPU寄存器是图灵机的内部状态，RAM是图灵机的磁带。

图灵机（CPU）可以做三件事：

+   从磁带（从RAM的内存单元读取0或1）

+   改变其内部状态（改变其寄存器）

+   左移或右移（从RAM读取多个位置）

+   向磁带写入0或1（将数据写入RAM的内存单元）

对于我们的目的，我们正在使用组合逻辑构建Wolfram的[2状态3符号图灵机](http://en.wikipedia.org/wiki/Wolfram%27s_2-state_3-symbol_Turing_machine)（现代CPU将使用微码，但它们比我们的目的更复杂）。

Wolfram 的 (2,3) 图灵机状态表如下：

```
 A       B
0   P1,R,B  P2,L,A
1   P2,L,A  P2,R,B
2   P1,L,A  P0,R,A 
```

我们希望将上面的状态表重新编码为一个真值表：

```
Let I1,I2 be the input from the tape reader (0 = (0,0), 1 = (0,1), 2 = (1,0))
Let O1,O2 be the tape writer (symbol encoding same as I1,I2)
Let M be connected to the machine's motor (0 = move left, 1 = move right)
Let R be the machine's internal state (A = 0, B = 1)
(R(t) is the machine's internal state at timestep t, R(t+1) at timestep t+1)
(Note that we used two input and two outputs since this is a 3-symbol Turing machine.)

      R  0          1
I1,I2
(0,0)    (0,1),1,1  (1,0),0,0
(0,1)    (1,0),0,0  (1,0),1,1
(1,0)    (0,1),0,0  (0,0),1,0

The truth table for the state table above:

I1  I2  R(t) | O1  O2  M   R(t+1)
-------------+--------------------
0   0   0    | 0   1   1   1
0   0   1    | 1   0   0   0
0   1   0    | 1   0   0   0
0   1   1    | 1   0   1   1
1   0   0    | 0   1   0   0
1   0   1    | 0   0   1   0 
```

我不会真的去构建这样一个逻辑门（我不确定如何在 SE 中绘制它，而且它可能会相当庞大），但由于我们知道 NAND 门是*功能完备*的，那么我们有一种方法找到一系列 NAND 门来实现这个真值表。

图灵机的一个重要特性是可以使用仅有固定状态表的图灵机来模拟[存储程序计算机](http://en.wikipedia.org/wiki/Stored-program_computer)。因此，任何通用图灵机都可以从磁带（RAM）中读取其程序，而不必将其指令硬编码到内部状态表中。换句话说，我们的 (2,3) 图灵机可以从 I1、I2 引脚（作为软件）读取其指令，而不是被硬编码到逻辑门实现中（作为硬件）。

## 微代码

由于现代 CPU 的复杂性不断增加，仅仅使用组合逻辑来设计整个 CPU 已变得难以实现。现代 CPU 通常被设计为微代码指令的解释器；微代码是嵌入在 CPU 中的一个小程序，用于 CPU 解释实际的机器码。这个微代码解释器本身通常是使用组合逻辑设计的。

## 寄存器、高速缓存和 RAM

我们上面忘了一些东西。我们如何记住一些东西？我们如何实现磁带和 RAM？答案在一个叫做电容器的电子元件中。电容器就像是可充电电池，如果一个电容器充电了，它会保留额外的电子，它也可以将电子返回到电路中。

要向电容器写入，我们将电子填充到电容器中（写入 1）或者将电容器中的所有电子耗尽直到它为空（写入 0）。要读取电容器的值，我们尝试放电。如果我们尝试放电时没有电流流过，那么电容器是空的（读取 0），但是如果检测到电流，那么电容器必须是充电的（读取 1）。你可能注意到读取电容器会耗尽其电子存储，现代 RAM 具有电路以周期性地重新充电电容器，使其在有电的情况下保持其存储。

在 CPU 中使用多种类型的电容器，CPU 寄存器和更高级别的 CPU 高速缓存是用非常高速的"电容器"（实际上由晶体管构建，因此几乎没有从中读/写的"滞后"），这些被称为静态 RAM (SRAM)；而主存储器 RAM 是使用功耗较低、速度较慢和成本更低的电容器制成的，这些被称为动态 RAM (DRAM)。

## 时钟

CPU 的一个非常重要的组成部分是时钟。时钟是一个定期“滴答”的组件，用于同步处理。时钟通常包含石英或其他具有已知且相对恒定振荡周期的材料，时钟电路通过维护和测量这种振荡来保持时间感。

CPU 操作是在时钟之间完成的，读/写是在时钟中完成的，以确保所有组件在中间状态时不会相互干扰。在我们的 (2,3) 图灵机中，在时钟之间，电力通过逻辑门来计算输入（I1, I2, R(t)）的输出；而在时钟中，打字机将 O1, O2 写入磁带，电机将根据 M 的值移动，内部寄存器从 R(t+1) 的值中写入，然后磁带读取器将读取当前磁带并将电荷放入 I1, I2，内部寄存器重新读取到 R(t)。

## 与外围设备交流

注意 (2,3) 图灵机如何与其电机交互。这是 CPU 可能与任意硬件接口的极其简化的视图。任意硬件可以监听或写入特定的电线来进行输入/输出。对于 (2,3) 图灵机来说，它与电机的接口只是一根单独的电线，指示电机顺时针或逆时针旋转。

在这台机器中未说的一点是，电机必须有另一个与机器内部“时钟”同步运行的“时钟”，以便知道何时启动和停止运行，所以这是一个同步数据传输的例子。

# 数字时代

## 机器码和汇编语言

汇编语言是机器码的一种人类可读的助记符。在最简单的情况下，汇编语言到机器码之间存在一对一的映射；尽管在现代汇编语言中，一些指令可能映射到多个操作码。

## 编程语言

我们都对此非常熟悉，不是吗？

* * *

哎呀，终于完成了，我仅用了 4 小时打完这些字，所以我确信某处肯定有错（我主要是程序员，不是电工或物理学家，所以可能有很多错误）。如果你发现错误，请@我或自行修正（如果你有权限），或者提供一个补充的答案。
