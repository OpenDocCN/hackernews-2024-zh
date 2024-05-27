<!--yml
category: 未分类
date: 2024-05-27 12:57:12
-->

# MicroZed Chronicles: Linear Feedback Shift Register

> 来源：[https://www.adiuvoengineering.com/post/microzed-chronicles-linear-feedback-shift-register](https://www.adiuvoengineering.com/post/microzed-chronicles-linear-feedback-shift-register)

As a follow on from last week, one of the techniques I like to use to stimulate or capture data in our FPGA designs is a Linear Feedback Shift Register (LFSR).

In the ten-plus years of this blog, I have never talked much about LFSRs, so I am going to focus this blog on exploring them as they are helpful for our FPGA designs. These days, we are a little spoiled with the modern fabric being so capable.

LFSRs can be used for a range of applications in our FPGAs:

1.  Generating random test patterns – Leveraging the pseudo random nature of the LFSR to generate data for test patterns.

2.  Counters – using a LFSR as a counter can often provide a more efficient and higher performing implementation than alternate approaches.

3.  Data scrambling / descrambling – The output from the LFSR can be combined with data to ensure that long runs of ones or zeros are removed which could result in synchronization problems when received.

4.  Error detection and correction – By selecting the specific polynomial to generate code words, we are able to detect errors in the transmission when the data is received and applied to the same LFSR. Ethernet is a commonly used example.

The LFSR is formed around a shift register which has one or several of the register outputs fed back into the input of the first register. The elements, which are fed back to the input, are often called taps and the tap selection will determine the behavior of the LFSR. Of course, the LFSR  will eventually repeat its pattern because it has feedback. For a maximal length implementation, the shift register will repeat after 2^n-1 iterations. This is a result of the two methods of implementing a LFSR. Both have illegal values which prohibit further operation and as such, are avoided.

The two methods of implementation for a LFSR are as follows:

*   Galois – In this architecture, the XOR/XNOR gates are implemented between register stages within the shift register and the output of the last stage is wrapped around to the first register input and connected to the XOR/XNOR gate input. This means when the output is 0, the register simply shifts.

Implementing both architectures can result in maximal length LFSR being implemented. However, we do need to do a translation between the location of the taps in a Fibonacci implementation versus a Galois implementation.

For a Fibonacci implementation of a simple 8-bit LFSR, we will have taps at 4,5,6,8 when the registers are numbered 1 to 8 (with 1 being the input and 8 being the output element). This leads to a diagram as below.

For a Galois implementation, however, the taps need to be updated so the register position is updated. Be aware that it is also common to number Galois registers in the opposite direction to Fibonacci.

One advantage of using a Galois implementation is that there is never more than one logic level between registers regardless of the number of taps used. However, in modern FPGAs, this can limit the implementation to registers and not enable the use of SRL for sections of the shift register if appropriate within the CLB. The implementation of the XOR can be handled by the LUT which proves very capable for larger XOR/XNOR Implementations with six input LUT. As such, the usage of Fibonacci or Galois implementation comes down to the engineer understanding what is best suited for your application.

The use of an XNOR or XOR gate depends on the application, however, both Fibonacci and Galois LFRS implementations support both gates. When a XOR gate is used, the illegal state for the register is all zeros while the illegal state is all ones for the XNOR gate. As AMD devices generally initialize to having all registers at zero (see my blog on resets here), it makes sense for us to use the XNOR style approach, although it is possible to create a modulo 2^n counter with a little extra decoding logic.

In the remainder of this blog, we are going to create simple LFSR implementations using the Galois and Fibonacci techniques and demonstrate the result when implemented in an AMD 7 series device.

The source code for both can be seen below. For the test bench, I run through the sequence and record them to text files which ensures I can double check that the sequence is correct. As a maximal length sequence, it should repeat after 255 iterations.   

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

If we modify the design slightly, the parallel word is not being output but instead, a serial output is generated. This is a common use case. We can then synthesize the design and observe how it is implemented. Note how it uses SRL whenever it can.  

In our next blog, we will be using a LFSR as part of our miniseries that looks at how we are able to capture data using the ILA and test equipment.

If you are looking for a list of LFSR feedback taps, the great Peter Alfke wrote an interesting [app note](https://docs.amd.com/v/u/en-US/xapp052) a while back which targets now obsolete devices and provides a great introduction to LFSR, along with an extensive LFSR table of taps.

If you enjoyed the blog why not take a look at the free webinars, workshops and training courses we have created over the years. Highlights include

Do you want to know more about designing embedded systems from scratch? Check out our book on creating embedded systems. This book will walk you through all the stages of requirements, architecture, component selection, schematics, layout, and FPGA / software design. We designed and manufactured the board at the heart of the book! The schematics and layout are available in Altium [here](https://www.e3designers.com/altium-365)Learn more about the board (see previous blogs on [Bring up](https://www.adiuvoengineering.com/post/microzed-chronicles-configuring-zynq-on-a-custom-board), [DDR validation,](https://www.adiuvoengineering.com/post/microzed-chronicles-validating-your-custom-zynq-board-memory) [USB](https://www.adiuvoengineering.com/post/microzed-chronicles-smart-sensor-iot-board-getting-usb-up-and-running), [Sensors](https://www.adiuvoengineering.com/post/microzed-chronicles-petalinux-i2c-in-the-ps-and-axi-iic)) and view the schematics [here](https://www.adiuvoengineering.com/post/sensorsthink-board-schematic).