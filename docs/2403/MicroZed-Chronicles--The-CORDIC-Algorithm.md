<!--yml

category: 未分类

date: 2024-05-27 14:55:07

-->

# MicroZed编年史：CORDIC算法

> 来源：[https://www.adiuvoengineering.com/post/microzed-chronicles-the-cordic-algorithm](https://www.adiuvoengineering.com/post/microzed-chronicles-the-cordic-algorithm)

在上周的博客中，我们讨论了有史以来最重要的算法之一：FFT。在本周的博客中，我们将讨论CORDIC算法，这个算法的重要性与FFT相仿。

CORDIC（COordinate Rotation DIgital Computer）算法简称，由Jack Volder于1959年为B58项目发明，是FPGA工程师工具箱中最重要的算法之一，但很少有人知晓。工程师们几乎肯定曾使用过类似HP35的科学计算器计算的结果，而现代计算器继续使用该算法进行三角函数和指数函数的计算。然而，理解CORDIC算法同样至关重要。CORDIC算法的真正优势在于它可以用非常小的FPGA占用空间实现，并且只需要一个小查找表以及执行移位和加法的逻辑。同样重要的是，该算法不需要专用的乘法器或除法器来实现。

这个算法是DSP和工业控制应用中最有用的之一，还可以根据其模式和配置实现一些非常有用的数学函数。CORDIC算法可以在三种配置中的一种下运行：线性、圆形或双曲。在每种配置中，算法以旋转或矢量模式中的一种模式运行。在旋转模式中，输入向量按指定角度旋转。在矢量模式中，算法将输入向量旋转到x轴，并记录所需的旋转角度。

统一的CORDIC算法如下所示，涵盖了所有三种配置。它有三个输入（X、Y和Z），它们在启动时的初始化取决于操作模式（矢量化或旋转）。

在这个算法中，m定义了双曲（m = -1）、线性（m = 0）或圆形（m = 1）的配置，ei的值（表示旋转角度）根据配置而变化。ei的值通常作为FPGA内的一个小查找表实现。

di是旋转方向，取决于旋转模式下的操作模式，如果Zi < 0，则di = -1，否则di = +1；而在矢量化模式下，如果Yi < 0，则di = +1，否则di = -1。

当配置为圆形或双曲，并使用旋转模式时，输出结果将具有增益，可以使用以下方程预先计算旋转数来定义。

这个增益通常反馈到算法的初始设置中，以消除后续结果的后缩放需求。

尽管上述算法对设计工程师非常重要，但需要注意的是，CORDIC算法仅在严格的收敛区内运行，这可能需要工程师执行一些预缩放，以确保算法表现如预期。值得注意的是，算法将随着工程师决定实现的每次迭代（串行）或阶段（并行）而变得更加精确。一个经验法则是，对于n位精度，需要n次迭代或阶段。

CORDIC算法仅在有限的输入值范围内收敛（工作）。对于CORDIC算法的圆形配置，仅在查找表中角度之和以下的角度（即-99.7至99.7度）内收敛是有保证的。对于超出此范围的角度，工程师必须使用三角恒等式将其转化为内部角度。这对于线性配置的收敛同样适用。然而，在双曲模式下，必须重复某些迭代（4, 13, 40, K… 3K+1）以获得收敛性。在这种情况下，Ɵ的最大输入约为1.118弧度。

正如之前所述，CORDIC广泛应用于从DSP和图像处理到工业控制系统的各种应用中。使用CORDIC的最基本方法是与相位累加器配合生成正弦和余弦波形。如果操作正确，该算法生成的波形将具有较高的无杂散动态范围（SFDR）。值得注意的是，大多数信号处理应用都要求良好的SFDR性能。在机器人领域，CORDIC在运动学中的使用使得通过圆形CORDIC在矢量化模式下添加坐标值与新坐标值成为可能。在图像处理领域，像光照和向量旋转这样的三维操作则是算法实现的理想候选。然而，该算法最常见的用途或许是在实现传统数学函数中，如表一所示。在这里，我们可以看到，在没有专用乘法器或DSP块的设备中，需要乘法器、除法器或更复杂的数学函数。这意味着，在许多小型工业控制器中，使用CORDIC实现数学转换函数是非常常见的。真有效值（True RMS）测量就是一个这样的例子。

**FPGA内的实现**

关于在我们的AMD FPGA中实现CORDIC，我们可以实现Vivado提供的CORDIC IP块。该模块提供了一系列配置选项，用于实现矢量化、旋转以及正弦和余弦生成，同时支持双曲线和平方根操作。

结论：我们将配置模块以生成正弦和余弦向量，并对其进行模拟，以进一步了解其操作。在未来的博客中，我们将扩展测试平台以查看其他功能。

在正弦和余弦模式下工作时，如果我们希望能够覆盖整个圆周，需要确保选择了粗旋转选项。如果没有选中，我们只能覆盖-Pi/4到Pi/4。选中后，我们能够完成整个圆周。

方框图如下所示。

利用创建的简单设计，我制作了一个简单的测试台，该测试台使用AXIS输入值。这个向量的输入格式为3个有符号整数位和剩余的分数部分。这允许输入值在+/- Pi之间。

输出向量格式为1位有符号整数位和剩余分数部分，允许输出在-1到1之间的值，并且正弦和余弦输出共享在输出向量上。

测试台非常直接，使用计数器来不断计数在-pi和pi之间，并作为相位输入应用到CORDIC IP核心中。

```
use IEEE.STD_LOGIC_1164.ALL;
```

```
use IEEE.NUMERIC_STD.ALL;
```

```
use UNISIM.VComponents.all;
```

```
architecture Behavioral of cordic_tb is
```

```
constant clk_period : time := 10 ns;
```

```
constant increment : real := 0.01256637;
```

```
signal M_AXIS_DOUT_0_tdata    :  STD_LOGIC_VECTOR ( 31 downto 0 );
```

```
signal M_AXIS_DOUT_0_tvalid   :  STD_LOGIC; 
```

```
signal S_AXIS_PHASE_0_tdata   :  STD_LOGIC_VECTOR ( 15 downto 0 ); 
```

```
signal S_AXIS_PHASE_0_tvalid  :  STD_LOGIC := '0'; 
```

```
signal clk                    :  STD_LOGIC := '0'; 
```

```
signal phase_wheel : real range -MATH_PI  to MATH_PI ; 
```

```
signal direction :  std_logic := '0'; 
```

```
clk <= not clk after (clk_period/2);
```

```
 M_AXIS_DOUT_0_tdata     =>  M_AXIS_DOUT_0_tdata, 
```

```
 M_AXIS_DOUT_0_tvalid    =>  M_AXIS_DOUT_0_tvalid, 
```

```
 S_AXIS_PHASE_0_tdata    =>  S_AXIS_PHASE_0_tdata, 
```

```
 S_AXIS_PHASE_0_tvalid   =>  S_AXIS_PHASE_0_tvalid, 
```

```
 S_AXIS_PHASE_0_tvalid <= '1';
```

```
 S_AXIS_PHASE_0_tdata  <= std_logic_vector(to_sfixed(phase_wheel, 2,-13 )) ; 
```

```
 if phase_wheel >= MATH_PI and direction = '0' then
```

```
 elsif phase_wheel <= -MATH_PI and direction = '1' then 
```

```
 phase_wheel <= phase_wheel - increment;
```

```
 phase_wheel <= phase_wheel + increment;
```

在Vivado模拟器中运行测试台将提供以下输出。

在未来的博客中，我们将更深入地探讨如何利用这一点进行各种应用，但在此之前，我们现在更了解CORDIC算法！

如果你喜欢这篇博客，不妨看看多年来我们创建的免费网络研讨会、工作坊和培训课程。亮点包括

想要了解如何从零开始设计嵌入式系统吗？查看我们的关于创建嵌入式系统的书籍。这本书将引导你完成需求、架构、组件选择、原理图、布局以及FPGA / 软件设计的所有阶段。书中的核心板我们设计并制造了！原理图和布局在Altium [这里](https://www.e3designers.com/altium-365)可见。了解更多关于板子的信息（参见之前关于[上电](https://www.adiuvoengineering.com/post/microzed-chronicles-configuring-zynq-on-a-custom-board)、[DDR验证](https://www.adiuvoengineering.com/post/microzed-chronicles-validating-your-custom-zynq-board-memory)、[USB](https://www.adiuvoengineering.com/post/microzed-chronicles-smart-sensor-iot-board-getting-usb-up-and-running)、[传感器](https://www.adiuvoengineering.com/post/microzed-chronicles-petalinux-i2c-in-the-ps-and-axi-iic)的先前博客），并查看原理图[这里](https://www.adiuvoengineering.com/post/sensorsthink-board-schematic)。
