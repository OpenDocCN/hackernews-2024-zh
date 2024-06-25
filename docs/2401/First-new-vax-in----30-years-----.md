<!--yml

类别：未分类

日期：2024-05-27 14:34:54

-->

# 30 年来的第一台新的 vax？ :-)

> 来源：[`mail-index.netbsd.org/port-vax/2021/07/03/msg003899.html`](https://mail-index.netbsd.org/port-vax/2021/07/03/msg003899.html)

Port-vax 存档

* * *

[上一日期][下一日期][上一线程][下一线程][日期索引][线程索引][旧索引]

# 30 年来的第一台新的 vax？ :-)

* * *

* * *

```
Hi,

```

`一段时间以前，我陷入了一个关于架构的讨论（risc vs cisc 等...）并开始思考 vax。` `即使 vax 被认为是“终极 cisc”，我想知道它的清晰度和良好的指令集是否仍然可以实现足够的高效性。` `好吧，唯一的办法就是尝试去实现它 :-) 我有一个带有小型低端 FPGA（Xilinx XC3S400）的 15 岁演示板，所以我只需要学习 Verilog 并尝试实现一些东西。` `而且它刚刚通过了 EVKAA.EXE：`

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

`(微码控制台的工作方式类似于 Nova4 微码控制台 - 比 VAX 样式更简单实现...)` `它运行在 50MHz，但可以轻松增加到约 80MHz，只是为了编程 DCM。`

```

Photo of the "vax": https://www.ludd.ltu.se/~ragge/pics/IMG_0837.jpg

```

`我不得不买一个新的 FPGA 板，因为旧的开始出现位错误，所以我买了一个带有基本相同 FPGA 的中国板（XC3S500E）：``我已经实现了所有寻址模式、中断层次结构、定时器和 151 条指令。`

```
No memory management yet though, but that should be quite straight-forward.

```

`目前它使用了大约可用 FPGA 资源的 70%，大约是 6000 个 LUTs（这种实现相当低效，因为我在编写代码时学会了如何进行 verilog 编程...）``我会在这封邮件后跟进两封，一封关于实现，一封关于如何继续进行 vax 架构 :-)`

```

-- R

```

* * *

* * *

**主页 | 主索引 | 线程索引 | 旧索引**
