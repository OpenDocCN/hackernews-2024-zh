<!--yml
category: 未分类
date: 2024-05-27 14:34:54
-->

# First new vax in ...30 years? :-)

> 来源：[https://mail-index.netbsd.org/port-vax/2021/07/03/msg003899.html](https://mail-index.netbsd.org/port-vax/2021/07/03/msg003899.html)

Port-vax archive

* * *

[[Date Prev](/port-vax/2021/05/03/msg003898.html)][[Date Next](/port-vax/2021/07/03/msg003900.html)][[Thread Prev](/port-vax/2021/04/17/msg003867.html)][[Thread Next](/port-vax/2021/07/03/msg003906.html)][[Date Index](../../../2021/07/date1.html#003899)][[Thread Index](../../../2021/07/thread1.html#003899)][[Old Index](../oindex.html)]

# First new vax in ...30 years? :-)

* * *

* * *

```
Hi,

```

`some time ago I ended up in an architectural discussion (risc vs cisc` `etc...) and started to think about vax.` `Even though the vax is considered the "ultimate cisc" I wondered if its` `cleanliness and nice instruction set still could be implemented` `efficient enough.` `Well, the only way to know would be to try to implement it :-)  I had an` `15-year-old demo board with a small low-end FPGA (Xilinx XC3S400), so I` `just had to learn Verilog and try to implement something.  And it just` `passed EVKAA.EXE:`

```

>FR00000000 200
>G
EVKAA V10.4 Hardcore Instruction Test

Hit any key to continue
EVKAA   V10.4 pass # 1(X) done!
EVKAA   V10.4 pass # 19(X) done!
EVKAA   V10.4 pass # 32(X) done!
EVKAA   V10.4 pass # 4B(X) done!
EVKAA   V10.4 pass # 64(X) done!
EVKAA   V10.4 pass # 7D(X) done!
EVKAA   V10.4 pass # 96(X) done!
EVKAA   V10.4 pass # AF(X) done!
EVKAA   V10.4 pass # C8(X) done!
EVKAA   V10.4 pass # E1(X) done!^C
Input breakpoint: 000000001
>

```

`(the microcode console works like the Nova4 microcode console - simpler` `to implement than the VAX style...)` `It runs at 50MHz, but could easily be increased to about 80, just to` `program DCM.`

```

Photo of the "vax": https://www.ludd.ltu.se/~ragge/pics/IMG_0837.jpg

```

`I had to get a new FPGA board, since I started to get bit errors on the` `old one, so I bought a chinese board with essentially the same FPGA` `(XC3S500E):``I have implemented all addressing modes, the interrupt hierarchy,` `timers, and 151 instructions.`

```
No memory management yet though, but that should be quite straight-forward.

```

`Currently it uses about 70% of available FPGA resources, which is around` `6000 LUTs (which is quite inefficient implemented, since I have learned` `how to do verilog programming while writing the code...)``I'll follow up this mail with two more, one about the implementation and` `one about how to go forward with the vax architecture :-)`

```

-- R

```

* * *

* * *

**[Home](/index.html) | [Main Index](../../../index.html) | [Thread Index](../../../tindex.html) | [Old Index](../../../oindex.html)**