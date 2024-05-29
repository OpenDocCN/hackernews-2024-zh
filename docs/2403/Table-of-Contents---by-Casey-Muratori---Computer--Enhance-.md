<!--yml
category: 未分类
date: 2024-05-27 14:56:45
-->

# Table of Contents - by Casey Muratori - Computer, Enhance!

> 来源：[https://www.computerenhance.com/p/table-of-contents](https://www.computerenhance.com/p/table-of-contents)

This series is designed for programmers who know how to write programs, but don’t know how hardware runs those programs. It’s designed to bring you up to speed on how modern CPUs work, how to estimate the expected speed of performance-critical code, and the basic optimization techniques every programmer should know.

The course is broken into parts, with the first part (the “prologue”) being strictly a demonstration with no associated homework. Later parts feature weekly homework.

*Q&A session videos are posted every Monday. **If you have a question you’d like answered, please put it in the comments of the most recent Q&A video**. Homework listings are available [from github](https://github.com/cmuratori/computer_enhance).*

**Prologue: The Five Multipliers** (3 1/2 hours, no homework)

*This part of the course gives simple demonstrations of how seemingly minor code changes can produce dramatically different software performance, even for very simple operations.*

1.  [Welcome to the Performance-Aware Programming Series!](https://www.computerenhance.com/p/welcome-to-the-performance-aware) (22:05)

2.  [Waste](https://www.computerenhance.com/p/waste) (32:56)

3.  [Instructions Per Clock](https://www.computerenhance.com/p/instructions-per-clock) (25:05)

4.  [Single Instruction, Multiple Data](https://www.computerenhance.com/p/single-instruction-multiple-data) (35:31)

5.  [Caching](https://www.computerenhance.com/p/caching) (22:55)

6.  [Multithreading](https://www.computerenhance.com/p/multithreading) (32:11)

7.  [Python Revisited](https://www.computerenhance.com/p/python-revisited) (36:22)

**Interlude** (1 hour, no homework)

1.  [The Haversine Distance Problem](https://www.computerenhance.com/p/the-haversine-distance-problem) (30:28)

2.  [“Clean” Code, Horrible Performance](https://www.computerenhance.com/p/clean-code-horrible-performance) (22:40)

**Part 1: Reading ASM** (*7 hours, plus homework*)

*This part of the course is designed to ensure that everyone taking the course has a solid understanding of how a CPU works at the assembly-language level.*

1.  [Instruction Decoding on the 8086](https://www.computerenhance.com/p/instruction-decoding-on-the-8086) (28:28)

2.  [Decoding Multiple Instructions and Suffixes](https://www.computerenhance.com/p/decoding-multiple-instructions-and) (43:51)

3.  [Opcode Patterns in 8086 Arithmetic](https://www.computerenhance.com/p/opcode-patterns-in-8086-arithmetic) (20:01)

4.  [8086 Decoder Code Review](https://www.computerenhance.com/p/8086-decoder-code-review) (1:17:49)

5.  [Using the Reference Decoder as a Shared Library](https://www.computerenhance.com/p/using-the-reference-decoder-as-a) (8:48)

6.  [Simulating Non-memory MOVs](https://www.computerenhance.com/p/simulating-non-memory-movs) (18:00)

7.  [Simulating ADD, SUB, and CMP](https://www.computerenhance.com/p/simulating-add-jmp-and-cmp) (25:56)

8.  [Simulating Conditional Jumps](https://www.computerenhance.com/p/simulating-conditional-jumps) (19:41)

9.  [Simulating Memory](https://www.computerenhance.com/p/simulating-memory) (26:32)

10.  [Simulating Real Programs](https://www.computerenhance.com/p/simulating-real-programs) (16:02)

11.  [Other Common Instructions](https://www.computerenhance.com/p/other-common-instructions) (19:43)

12.  [The Stack](https://www.computerenhance.com/p/the-stack) (26:58)

13.  [Estimating Cycles](https://www.computerenhance.com/p/estimating-cycles) (23:56)

14.  [From 8086 to x64](https://www.computerenhance.com/p/from-8086-to-x64) (26:21)

15.  [8086 Simulation Code Review](https://www.computerenhance.com/p/8086-simulation-code-review) (33:05)

**Part 2: Basic Profiling** (*4 hours, plus homework*)

*In this part of the course, we learn about how to measure time, and instrument programs to automatically determine where time is being spent.*

1.  [Generating Haversine Input JSON](https://www.computerenhance.com/p/generating-haversine-input-json) (15:40)

2.  [Writing a Simple Haversine Distance Processor](https://www.computerenhance.com/p/writing-a-simple-haversine-distance) (12:09)

3.  [Initial Haversine Processor Code Review](https://www.computerenhance.com/p/initial-haversine-processor-code) (29:22)

4.  [Introduction to RDTSC](https://www.computerenhance.com/p/introduction-to-rdtsc) (48:05)

5.  [How does QueryPerformanceCounter measure time?](https://www.computerenhance.com/p/how-does-queryperformancecounter) (31:43)

6.  [Instrumentation-Based Profiling](https://www.computerenhance.com/p/instrumentation-based-profiling) (18:01)

7.  [Profiling Nested Blocks](https://www.computerenhance.com/p/profiling-nested-blocks) (26:12)

8.  [Profiling Recursive Blocks](https://www.computerenhance.com/p/profiling-recursive-blocks) (30:44)

9.  [A First Look at Profiling Overhead](https://www.computerenhance.com/p/a-first-look-at-profiling-overhead) (18:37)

10.  [Comparing the Overhead of RDTSC and QueryPerformanceCounter](https://www.computerenhance.com/p/comparing-the-overhead-of-rdtsc-and) (13:00)

**Part 3: Moving Data** (*currently in progress*)

*Using our knowledge from parts 1 and 2, in Part 3 we look at how data moves into the CPU, and how to estimate the upper performance limits of our software imposed by the need to move data.*

1.  [Measuring Data Throughput](https://www.computerenhance.com/p/measuring-data-throughput) (21:54)

2.  [Repetition Testing](https://www.computerenhance.com/p/repetition-testing) (27:57)

3.  [Monitoring OS Performance Counters](https://www.computerenhance.com/p/monitoring-os-performance-counters) (20:25)

4.  [Page Faults](https://www.computerenhance.com/p/page-faults) (38:52)

    1.  [Probing OS Page Fault Behavior](https://www.computerenhance.com/p/probing-os-page-fault-behavior)* (33:05)

    2.  [Four-Level Paging](https://www.computerenhance.com/p/four-level-paging)* (31:23)

    3.  [Analyzing Page Fault Anomalies](https://www.computerenhance.com/p/analyzing-page-fault-anomalies)* (31:44)

    4.  [Powerful Page Mapping Techniques](https://www.computerenhance.com/p/powerful-page-mapping-techniques)* (39:20)

    5.  [Faster Reads with Large Page Allocations](https://www.computerenhance.com/p/faster-reads-with-large-page-allocations) (25:52)

    6.  [Memory-Mapped Files](https://www.computerenhance.com/p/memory-mapped-files)* (20:46)

5.  [Inspecting Loop Assembly](https://www.computerenhance.com/p/inspecting-loop-assembly) (32:31)

6.  [Intuiting Latency and Throughput](https://www.computerenhance.com/p/intuiting-latency-and-throughput) (22:57)

7.  [Analyzing Dependency Chains](https://www.computerenhance.com/p/analyzing-dependency-chains) (29:06)

8.  [Linking Directly to ASM for Experimentation](https://www.computerenhance.com/p/linking-directly-to-asm-for-experimentation) (48:07)

9.  [CPU Front End Basics](https://www.computerenhance.com/p/cpu-front-end-basics) (31:09)

10.  [Branch Prediction](https://www.computerenhance.com/p/branch-prediction) (42:03)

11.  [Code Alignment](https://www.computerenhance.com/p/code-alignment) (32:03)

12.  [The RAT and the Register File](https://www.computerenhance.com/p/the-rat-and-the-register-file) (45:21)

13.  [Execution Ports and the Scheduler](https://www.computerenhance.com/p/execution-ports-and-the-scheduler) (34:51)

14.  [Increasing Read Bandwidth with SIMD Instructions](https://www.computerenhance.com/p/increasing-read-bandwidth-with-simd) (37:52)

15.  [Cache Size and Bandwidth Testing](https://www.computerenhance.com/p/cache-size-and-bandwidth-testing) (34:00)

16.  [Non-Power-of-Two Cache Size Testing](https://www.computerenhance.com/p/non-power-of-two-cache-size-testing) (35:15)

17.  [Unaligned Load Penalties](https://www.computerenhance.com/p/unaligned-load-penalties) (27:25)

** Entries with an asterisk were “bonus” entries that can be skipped.*

*Part 3 is still in progress - more videos will be added here as they are scheduled. Additional parts will follow after Part 3 is complete.*

1.  [The Four Programming Questions from My 1994 Microsoft Internship Interview](https://www.computerenhance.com/p/the-four-programming-questions-from) (19:02)

2.  [Question #1: Rectangle Copy](https://www.computerenhance.com/p/microsoft-intern-interview-question) (24:50)

3.  [Question #2: String Copy](https://www.computerenhance.com/p/microsoft-intern-interview-question-ab7) (14:50)

4.  [Question #3: Flood Fill Detection](https://www.computerenhance.com/p/microsoft-intern-interview-question-a3f) (23:58)

5.  [Question #4: Outline a Circle](https://www.computerenhance.com/p/efficient-dda-circle-outlines) (1:09:01)