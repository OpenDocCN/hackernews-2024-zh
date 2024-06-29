<!--yml

类别：未分类

日期：2024-05-27 12:57:12

-->

# MicroZed Chronicles: 线性反馈移位寄存器

> 来源：[https://www.adiuvoengineering.com/post/microzed-chronicles-linear-feedback-shift-register](https://www.adiuvoengineering.com/post/microzed-chronicles-linear-feedback-shift-register)

作为上周的延续，我喜欢在我们的FPGA设计中使用的一种刺激或捕获数据的技术是线性反馈移位寄存器（LFSR）。

在这个博客的十多年中，我从未多谈论过LFSR，所以我打算把这篇博客专注于探索它们，因为它们对我们的FPGA设计非常有帮助。如今，我们对现代结构如此强大感到有些宠坏了。

在我们的FPGA中，LFSR可以用于一系列应用：

1.  制造随机测试模式 - 利用LFSR的伪随机特性生成测试模式的数据。

1.  计数器 - 使用LFSR作为计数器通常比其他方法提供更高效和性能更高的实现。

1.  数据混淆/解混淆 - LFSR的输出可以与数据结合使用，以确保消除长时间运行的1或0，这可能导致接收时的同步问题。

1.  错误检测和校正 - 通过选择特定的多项式生成码字，我们能够在数据接收时检测到传输中的错误，并将其应用于相同的LFSR。以太网是一个常用的例子。

LFSR围绕一个移位寄存器形成，其中一个或多个寄存器的输出反馈到第一个寄存器的输入。反馈到输入的元素通常称为抽头，抽头选择将决定LFSR的行为。当然，LFSR最终会重复其模式，因为它有反馈。对于最大长度的实现，移位寄存器在2^n-1次迭代后将重复。这是实现LFSR的两种方法的结果。两种方法都有非法值，这些非法值会禁止进一步操作，因此应该避免使用。

LFSR的两种实现方法如下：

+   伽罗瓦 - 在这种架构中，XOR/XNOR门在移位寄存器的寄存器级之间实现，并且最后一个级的输出被环绕到第一个寄存器的输入，并连接到XOR/XNOR门输入。这意味着当输出为0时，寄存器简单地移位。

实施这两种架构可能会导致最大长度LFSR的实现。然而，我们确实需要在斐波那契实现和伽罗瓦实现中的反馈位置之间进行转换。

对于一个简单的8位斐波那契实现的LFSR，当寄存器从1到8编号时（其中1是输入，8是输出元素），我们将在4、5、6、8处设置抽头。这导致如下图所示的图表。

对于 Galois 实现，需要更新选项卡以更新寄存器位置。请注意，通常将 Galois 寄存器编号与 Fibonacci 相反。

使用 Galois 实现的一个优点是无论使用的选项卡数量如何，寄存器之间永远不会超过一个逻辑电平。然而，在现代 FPGA 中，这可能限制了对寄存器的实现，并且不能在 CLB 内适当使用 SRL 用于移位寄存器的部分。 XOR 的实现可以由 LUT 处理，这对于较大的 XOR/XNOR 实现非常有效。因此，Fibonacci 或 Galois 实现的使用取决于工程师理解什么最适合您的应用程序。

使用 XNOR 或 XOR 门取决于应用程序，但 Fibonacci 和 Galois LFSR 实现都支持这两种门。当使用 XOR 门时，寄存器的非法状态为全部为零，而使用 XNOR 门的非法状态为全部为一。由于 AMD 设备通常初始化为所有寄存器都为零（请参阅我关于重置的博客），因此我们使用 XNOR 风格的方法是合理的，尽管也可以使用少量额外解码逻辑创建模 2^n 计数器。

在本博客的其余部分，我们将使用 Galois 和 Fibonacci 技术创建简单的 LFSR 实现，并展示在 AMD 7 系列设备上实现时的结果。

两者的源代码如下所示。对于测试台，我会通过序列并将它们记录到文本文件中，以确保我可以双重检查序列是否正确。作为最大长度序列，它应在 255 次迭代后重复。

```
library ieee;
use ieee.std_logic_1164.all;

entity lfsr is port(

    i_clk       : in std_logic;
    o_fibonacci : out std_logic_vector(7 downto 0);
    o_galois    : out std_logic_vector(7 downto 0)
    );
end entity;

architecture rtl of lfsr is 

    signal s_galois_lfsr    : std_logic_vector(1 to 8) :=(others =>'0');
    signal s_fibonacci_lfsr : std_logic_vector(1 to 8) :=(others =>'0');

begin
    G_LFSR: process(i_clk)
    begin
        if rising_edge(i_clk) then
            s_galois_lfsr(6 to 8) <= s_galois_lfsr(5 to 7);
            s_galois_lfsr(5) <= s_galois_lfsr(4) XNOR s_galois_lfsr(8);
            s_galois_lfsr(4) <= s_galois_lfsr(3) XNOR s_galois_lfsr(8);
            s_galois_lfsr(3) <= s_galois_lfsr(2) XNOR s_galois_lfsr(8);
            s_galois_lfsr(2) <= s_galois_lfsr(1);
            s_galois_lfsr(1) <= s_galois_lfsr(8);
        end if;
    end process;

    F_LFSR: process(i_clk)
    begin
        if rising_edge(i_clk) then
            s_fibonacci_lfsr(2 to 8) <= s_fibonacci_lfsr(1 to 7);
            s_fibonacci_lfsr(1) <= s_fibonacci_lfsr(8) XNOR s_fibonacci_lfsr(6) XNOR s_fibonacci_lfsr(5) XNOR s_fibonacci_lfsr(4) ;
        end if;
    end process;

    o_galois <= s_galois_lfsr;
    o_fibonacci <= s_fibonacci_lfsr;

end architecture;
```

```
library ieee;
use ieee.std_logic_1164.all;
use ieee.numeric_std.all;
use std.textio.all;
use ieee.std_logic_textio.all;

entity lfsr_tb is
end;

architecture bench of lfsr_tb is
  -- Clock period
  constant clk_period : time := 10 ns;

  file fibonacci : text;
  file galois    : text;

  signal i_clk : std_logic :='0' ;
  signal o_fibonacci : std_logic_vector(7 downto 0);
  signal o_galois : std_logic_vector(7 downto 0);

begin

  lfsr_inst : entity work.lfsr
  port map (
    i_clk => i_clk,
    o_fibonacci => o_fibonacci,
    o_galois => o_galois
  );

process

    variable f_line : line;
    variable g_line : line;
begin 

    file_open(fibonacci, "fibonacci.txt", write_mode);
    file_open(galois, "galois.txt", write_mode);

    for i in 0 to 256 loop
        wait until rising_edge(i_clk);
        HWRITE(f_line, o_fibonacci );
        HWRITE(g_line, o_galois );
        WRITELINE(fibonacci, f_line);
        WRITELINE(galois, g_line);
    end loop;
    file_close(fibonacci);
    file_close(galois);
    report "simulation complete" severity failure;
end process;

i_clk <= not i_clk after clk_period/2;

end;
```

如果我们稍微修改设计，就不会输出并行词，而是生成串行输出。这是一个常见的用例。然后我们可以综合设计并观察其实现方式。请注意它在能使用 SRL 时如何使用 SRL。

在我们接下来的博客中，我们将在我们的迷你系列中使用 LFSR，看看我们如何使用 ILA 和测试设备捕获数据。

如果您正在寻找 LFSR 反馈选项卡列表，那么伟大的彼得·阿尔夫克曾经写过一篇有趣的[应用笔记](https://docs.amd.com/v/u/en-US/xapp052)，目标是现在已经过时的设备，并提供了有关 LFSR 的很好介绍，以及广泛的 LFSR 选项卡表。

如果您喜欢这篇博客，不妨看看多年来我们创建的免费网络研讨会、工作坊和培训课程。亮点包括

想要从零开始了解设计嵌入式系统吗？查看我们的关于创建嵌入式系统的书籍。这本书将引导您完成需求、架构、组件选择、原理图、布局和FPGA/软件设计的所有阶段。我们设计并制造了这本书核心的板！原理图和布局在Altium中[此处](https://www.e3designers.com/altium-365)可供查看。了解有关该板的更多信息（查看先前博客有关[启动](https://www.adiuvoengineering.com/post/microzed-chronicles-configuring-zynq-on-a-custom-board)、[DDR验证](https://www.adiuvoengineering.com/post/microzed-chronicles-validating-your-custom-zynq-board-memory)、[USB](https://www.adiuvoengineering.com/post/microzed-chronicles-smart-sensor-iot-board-getting-usb-up-and-running)、[传感器](https://www.adiuvoengineering.com/post/microzed-chronicles-petalinux-i2c-in-the-ps-and-axi-iic)）并查看原理图[此处](https://www.adiuvoengineering.com/post/sensorsthink-board-schematic)。
