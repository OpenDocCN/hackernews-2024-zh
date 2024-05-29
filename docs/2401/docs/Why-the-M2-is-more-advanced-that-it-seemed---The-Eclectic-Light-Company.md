<!--yml
category: 未分类
date: 2024-05-27 14:48:45
-->

# Why the M2 is more advanced that it seemed – The Eclectic Light Company

> 来源：[https://eclecticlight.co/2024/01/15/why-the-m2-is-more-advanced-that-it-seemed/](https://eclecticlight.co/2024/01/15/why-the-m2-is-more-advanced-that-it-seemed/)

When Apple launched its M2 chip at WWDC 18 months ago, in June 2022, pretty well everyone saw it as evolutionary, significantly faster than its predecessor the M1, but offering little real change in capability. In this article, which follows on from last week’s about [changed support for AI](https://eclecticlight.co/2024/01/13/how-m1-macs-may-lag-behind/), I take a deeper dive inside Apple silicon to discover whether the M2 is more than we thought at the time. The clues come in the instruction set supported by CPU cores inside the chips.

When comparing chips, emphasis is laid on performance, particularly benchmark tests that are the equivalent of sprint performance of an athlete in that they tell you how quickly the chip can run today’s tasks. If you intend keeping your Mac for longer than a year, you should also be interested in how its chip will perform when running the tasks of the future, or for our track athlete whether they’re also good enough at field events to be a good decathlete, their capability.

#### Changes in instructions since the M1

For CPUs, capability is primarily determined by the instructions they run. Today you may not be interested in whether your Mac’s chip can perform ray-tracing very quickly, but in a couple of year’s time the hardware-accelerated ray-tracing in the GPU of an M3 could make all the difference. While Apple adds plenty of its own hardware, including GPU, a neural engine, and its legendary matrix co-processor the AMX, the capability of CPU cores remains central to many of the tasks performed by its chips. Those are defined by Arm, and licensed to Apple, in its Instruction Set Architecture (ISA), documented in a manual currently well over 5,000 pages long.

Mercifully, Arm defines its ISA in versions. CPU cores in M1 chips use ARMv8.5-A, while those in M2 and M3 chips use ARMv8.6-A, and Arm helpfully explains their main differences in [this list of changes](https://community.arm.com/arm-community-blogs/b/architectures-and-processors-blog/posts/arm-architecture-developments-armv8-6-a) in ARMv8.6-A of 2019:

*   General Matrix Multiply (AI and others)
*   bfloat16 data type and arithmetic instructions (AI and others)
*   Finer-grained traps for virtualisation (virtualisation)
*   Wait-for-event traps for virtualisation (virtualisation)
*   High precision time (1 GHz, general)
*   Extended Pointer Authentication (security).

Of those, support for bfloat16 and General Matrix Multiply are likely to have the most impact on the user.

Although Macs based on the M1 chip weren’t released until November 2020, a year after the introduction of ARMv8.6-A, the lead time in design and development is such that there would have been no time to incorporate changes from 2019, the year after bfloat16 first appeared.

#### bfloat16

As I [explained before](https://eclecticlight.co/2024/01/13/how-m1-macs-may-lag-behind/), we’re dealing here with three different floating-point number formats, each expressed using a *sign* (+ or -), a *fraction* whose length determines its precision, and an *exponent* that determines the overall range of numbers that can be represented in that format. Before the introduction of bfloat16, the choice for AI and some other computation came down to two formats:

*   *float32* (single-precision), with a range of about +/- 1.2 x 10^-38 to 3.4 x 10^38, occupying 32 bits
*   *float16* (half-precision), with a range of about +/- 6.1 x 10^-5 to 65,504, occupying 16 bits.

float32 has been almost universally used in AI and other applications where double-precision (float64) isn’t required.

bfloat16 adds to those an intermediate, with the [same range as float32](https://en.wikipedia.org/wiki/Bfloat16_floating-point_format), of about +/- 1.2 x 10^-38 to 3.4 x 10^38, but only occupying half the space, at lower precision. It’s designed for easy conversion with float32, as its sign and exponent remain unchanged, only the fraction (significand, or mantissa) has to be extended or truncated, depending on which direction you’re going in. Converting between float32 and float16 is more involved, and most importantly, as the range allowed for float16 is far smaller, numbers outside its range would lose their numeric value. That means any floating-point number larger than 65,504, which is a severe limitation for many applications.

A number format that is half the length of float32 numbers isn’t only important when storing large amounts of data, but has substantial effects on the performance of operations. Those are usually accelerated using ‘single instruction, multiple data’ (SIMD) techniques, where [a register is packed](https://eclecticlight.co/2021/08/23/code-in-arm-assembly-lanes-and-loads-in-neon/) with two or more values, and the core then executes instructions on them in parallel. In the CPU cores of M-series chips, that’s normally done using 128-bit registers, which can hold four float32 values, or eight bfloat16 values. For tasks involving thousands of arithmetic operations, packing registers with twice the number of values can almost double the throughput, as [reported in tests](https://community.arm.com/arm-community-blogs/b/ai-and-ml-blog/posts/bfloat16-processing-for-neural-networks-on-armv8_2d00_a) on Arm processors.

In applications where its reduced precision can be accepted, the bfloat16 format thus offers the same range as float32, simple and quick conversion with float32, occupies half the storage, and delivers up to double the performance in SIMD execution.

#### But my Mac isn’t training AI models

When Google’s AI researchers [first made the claim](https://cloud.google.com/blog/products/ai-machine-learning/bfloat16-the-secret-to-high-performance-on-cloud-tpus) that bfloat16 is “the secret to high performance”, they were of course referring to those developing AI models, rather than ordinary users. That article was published in August 2019, less than a year before Apple announced the M1, explaining why none of the hardware in the M1 could have supported bfloat16, and that support wasn’t added by Arm until ARMv8.6-A.

Both Arm and Apple recognise the importance of performing as much AI training as possible in-device rather than in the cloud. The case for this has been [argued eloquently by Hellen Norman](https://community.arm.com/arm-community-blogs/b/architectures-and-processors-blog/posts/ai-vs-ml-whats-the-difference) of Arm, independently of Apple.

It doesn’t take much imagination to come up with local training tasks that could improve commonplace features in macOS. Many of us disable spell-checking because of its seeming inability to recognise when we should be using similar words like *their, there* and *they’re.* Wouldn’t it be so much better if suggested corrections were based on grammar, usage and context? While that’s already starting to improve, and Sonoma’s auto-completion is getting smarter, there’s ample room for improvement. That depends in part at least on your Mac learning your writing style using in-device training.

At the start of this article, I explained how this isn’t about the performance of current tasks, but the capability to accomplish the tasks our apps and macOS will be doing in the future, when some of those will involve the sort of training that’s currently left to specialised or dedicated systems.

CPU cores aren’t the only hardware in Apple silicon to support AI: depending on the task, macOS may use their GPU, Apple’s specialised neural engine, or its legendary AMX. Given the fact that those in the M1 were designed and developed over the same timescale as the CPU cores, it seems improbable that they would have bfloat16 support, and Apple has only just [added that number type](https://developer.apple.com/documentation/metalperformanceshaders/mpsdatatype/bfloat16?changes=l_8_6) to Metal Performance Shaders in Sonoma.

#### Does my Mac support it?

If you’re unconvinced whether your Apple silicon Mac supports bfloat16 in its CPU cores, then there’s an easy way to check. In Terminal, run the command
`sysctl -A > ~/Documents/sysctloutput.text`
where the last path is a new text file to take the output from the command.

If that file contains the line
`hw.optional.arm.FEAT_BF16: 1`
then its cores have hardware support for bfloat16\. If the number given is 0, then I’m afraid they don’t. Apple provides information to decode most of the `hw.optional.arm` features [here](https://developer.apple.com/documentation/kernel/1387446-sysctlbyname/determining_instruction_set_characteristics).

Not having bfloat16 support in hardware isn’t the end of the world, nor does it mean your M1 Mac is already obsolete. What it does mean, though, is that as more and heavier AI is rolled out in the coming years, some of those features will run noticeably more slowly on it. Spare a thought, though, for Intel Macs, for no matter how fast their CPUs might be, or how many cores they have, they will never have any equivalent hardware support for AI.