<!--yml

category: 未分类

date: 2024-05-27 14:56:45

-->

# 目录 - Casey Muratori - Computer, Enhance!

> 来源：[https://www.computerenhance.com/p/table-of-contents](https://www.computerenhance.com/p/table-of-contents)

此系列适用于懂得如何编写程序但不了解硬件如何运行这些程序的程序员。它旨在让您快速了解现代CPU的工作原理，如何估计性能关键代码的预期速度，以及每个程序员都应了解的基本优化技术。

该课程分为几个部分，第一部分（“序幕”）纯粹是演示，没有相关的作业。后续部分包括每周作业。

*每周一发布问答视频。 **如果您有问题需要解答，请将其放在最新问答视频的评论中**。作业列表可从[github](https://github.com/cmuratori/computer_enhance)获取。*

**序幕：五个乘法器** (3 1/2 小时，无作业)

*本课程的这一部分展示了如何通过看似不起眼的代码更改来产生截然不同的软件性能，即使对于非常简单的操作也是如此。*

1.  [欢迎来到性能感知编程系列！](https://www.computerenhance.com/p/welcome-to-the-performance-aware) (22:05)

1.  [废物](https://www.computerenhance.com/p/waste) (32:56)

1.  [每时钟周期指令](https://www.computerenhance.com/p/instructions-per-clock) (25:05)

1.  [单指令多数据](https://www.computerenhance.com/p/single-instruction-multiple-data) (35:31)

1.  [缓存](https://www.computerenhance.com/p/caching) (22:55)

1.  [多线程](https://www.computerenhance.com/p/multithreading) (32:11)

1.  [Python 重访](https://www.computerenhance.com/p/python-revisited) (36:22)

**插曲** (1 小时，无作业)

1.  [哈文赛因距离问题](https://www.computerenhance.com/p/the-haversine-distance-problem) (30:28)

1.  [“干净”的代码，可怕的性能](https://www.computerenhance.com/p/clean-code-horrible-performance) (22:40)

**第一部分：阅读汇编语言** (*7 小时，加作业*)

*本课程的这一部分旨在确保所有参加本课程的人都对CPU在汇编语言级别上的工作原理有扎实的理解。*

1.  [8086 上的指令解码](https://www.computerenhance.com/p/instruction-decoding-on-the-8086) (28:28)

1.  [解码多条指令和后缀](https://www.computerenhance.com/p/decoding-multiple-instructions-and) (43:51)

1.  [8086 算术中的操作码模式](https://www.computerenhance.com/p/opcode-patterns-in-8086-arithmetic) (20:01)

1.  [8086 解码器代码审查](https://www.computerenhance.com/p/8086-decoder-code-review) (1:17:49)

1.  [使用参考解码器作为共享库](https://www.computerenhance.com/p/using-the-reference-decoder-as-a) (8:48)

1.  [模拟非存储MOV](https://www.computerenhance.com/p/simulating-non-memory-movs) (18:00)

1.  [模拟ADD、SUB和CMP](https://www.computerenhance.com/p/simulating-add-jmp-and-cmp) (25:56)

1.  [模拟条件跳转](https://www.computerenhance.com/p/simulating-conditional-jumps) (19:41)

1.  [模拟内存](https://www.computerenhance.com/p/simulating-memory) (26:32)

1.  [模拟真实程序](https://www.computerenhance.com/p/simulating-real-programs) (16:02)

1.  [其他常见指令](https://www.computerenhance.com/p/other-common-instructions) (19:43)

1.  [堆栈](https://www.computerenhance.com/p/the-stack) (26:58)

1.  [估算周期数](https://www.computerenhance.com/p/estimating-cycles) (23:56)

1.  [从8086到x64](https://www.computerenhance.com/p/from-8086-to-x64) (26:21)

1.  [8086模拟代码审查](https://www.computerenhance.com/p/8086-simulation-code-review) (33:05)

**第2部分：基础性能分析** (*4小时，加上作业*)

-   在本课程的这一部分，我们学习如何测量时间，并为程序进行仪器化，自动确定时间消耗的位置。

1.  [生成Haversine输入JSON](https://www.computerenhance.com/p/generating-haversine-input-json) (15:40)

1.  [编写简单的Haversine距离处理器](https://www.computerenhance.com/p/writing-a-simple-haversine-distance) (12:09)

1.  [初始Haversine处理器代码审查](https://www.computerenhance.com/p/initial-haversine-processor-code) (29:22)

1.  [介绍RDTSC](https://www.computerenhance.com/p/introduction-to-rdtsc) (48:05)

1.  [QueryPerformanceCounter如何测量时间？](https://www.computerenhance.com/p/how-does-queryperformancecounter) (31:43)

1.  [基于仪器的性能分析](https://www.computerenhance.com/p/instrumentation-based-profiling) (18:01)

1.  [分析嵌套块](https://www.computerenhance.com/p/profiling-nested-blocks) (26:12)

1.  [分析递归块](https://www.computerenhance.com/p/profiling-recursive-blocks) (30:44)

1.  [首次查看性能分析开销](https://www.computerenhance.com/p/a-first-look-at-profiling-overhead) (18:37)

1.  [比较RDTSC和QueryPerformanceCounter的开销](https://www.computerenhance.com/p/comparing-the-overhead-of-rdtsc-and) (13:00)

**第3部分：数据移动** (*目前正在进行中*)

-   使用我们从第1部分和第2部分的知识，第3部分我们研究数据如何进入CPU，以及如何估计软件的性能上限，受数据移动需求的影响。

1.  [测量数据吞吐量](https://www.computerenhance.com/p/measuring-data-throughput) (21:54)

1.  [重复测试](https://www.computerenhance.com/p/repetition-testing) (27:57)

1.  [监视操作系统性能计数器](https://www.computerenhance.com/p/monitoring-os-performance-counters) (20:25)

1.  [页面错误](https://www.computerenhance.com/p/page-faults) (38:52)

    1.  [探索操作系统页面错误行为](https://www.computerenhance.com/p/probing-os-page-fault-behavior)* (33:05)

    1.  [四级分页](https://www.computerenhance.com/p/four-level-paging)* (31:23)

    1.  [分析页面错误异常](https://www.computerenhance.com/p/analyzing-page-fault-anomalies)* (31:44)

    1.  [强大的页面映射技术](https://www.computerenhance.com/p/powerful-page-mapping-techniques)* (39:20)

    1.  [使用大页面分配进行更快读取](https://www.computerenhance.com/p/faster-reads-with-large-page-allocations) (25:52)

    1.  [内存映射文件](https://www.computerenhance.com/p/memory-mapped-files)* (20:46)

1.  [检查循环组件](https://www.computerenhance.com/p/inspecting-loop-assembly) (32:31)

1.  [直觉延迟和吞吐量](https://www.computerenhance.com/p/intuiting-latency-and-throughput) (22:57)

1.  [分析依赖链](https://www.computerenhance.com/p/analyzing-dependency-chains) (29:06)

1.  [直接链接到ASM进行实验](https://www.computerenhance.com/p/linking-directly-to-asm-for-experimentation) (48:07)

1.  [CPU前端基础](https://www.computerenhance.com/p/cpu-front-end-basics) (31:09)

1.  [分支预测](https://www.computerenhance.com/p/branch-prediction) (42:03)

1.  [代码对齐](https://www.computerenhance.com/p/code-alignment) (32:03)

1.  [RAT与寄存器文件](https://www.computerenhance.com/p/the-rat-and-the-register-file) (45:21)

1.  [执行端口和调度器](https://www.computerenhance.com/p/execution-ports-and-the-scheduler) (34:51)

1.  [使用SIMD指令提高读取带宽](https://www.computerenhance.com/p/increasing-read-bandwidth-with-simd) (37:52)

1.  [缓存大小和带宽测试](https://www.computerenhance.com/p/cache-size-and-bandwidth-testing) (34:00)

1.  [非二次幂缓存大小测试](https://www.computerenhance.com/p/non-power-of-two-cache-size-testing) (35:15)

1.  [未对齐负载惩罚](https://www.computerenhance.com/p/unaligned-load-penalties) (27:25)

**带星号的条目是“奖励”条目，可以跳过。**

*第3部分仍在进行中 - 更多视频将随着计划添加到此处。第3部分完成后将继续添加其他部分。*

1.  [1994年我在微软实习面试中的四个编程问题](https://www.computerenhance.com/p/the-four-programming-questions-from) (19:02)

1.  [问题＃1：矩形复制](https://www.computerenhance.com/p/microsoft-intern-interview-question) (24:50)

1.  [问题＃2：字符串复制](https://www.computerenhance.com/p/microsoft-intern-interview-question-ab7) (14:50)

1.  [问题＃3：洪水填充检测](https://www.computerenhance.com/p/microsoft-intern-interview-question-a3f) (23:58)

1.  [问题＃4：概述圆形](https://www.computerenhance.com/p/efficient-dda-circle-outlines) (1:09:01)
