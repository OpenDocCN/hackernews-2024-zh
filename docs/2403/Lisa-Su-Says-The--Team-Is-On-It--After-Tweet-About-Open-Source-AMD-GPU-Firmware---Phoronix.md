<!--yml

category: 未分类

date: 2024-05-28 18:15:08

-->

# Lisa Su在关于开源AMD GPU固件的推文后表示"团队正在处理中" - Phoronix

> 来源：[https://www.phoronix.com/news/Lisa-Su-Tweet-OSS-Firmware](https://www.phoronix.com/news/Lisa-Su-Tweet-OSS-Firmware)

Tiny Corp的George Hotz正在致力于Tinygrad和TinyBox，开源AI领域的有趣发展之前

[指责AMD在ROCm问题上](https://www.phoronix.com/news/Lisa-Su-ROCm-Commitment)

. 昨天，由"小型公司"发布了关于AI训练运行崩溃和MES错误的新推文，并呼吁AMD开源固件，对此AMD首席执行官Lisa Su已经做出了回应。

Tiny Corp

[推文](https://twitter.com/__tinygrad__/status/1765085827946942923)

关于这些问题：

> "按目前的情况看，我不同意发布7900XTX平台。我们应该怎么办？
> 
> ...
> 
> 我们不是AMD的QA团队，与他们没有关系。去年我看到了一些让我充满希望的东西，但这个平台已经出现了14个月，仍然存在严重问题。
> 
> MES不开源让我感到不安。虽然比NVIDIA开源更多的东西，但如果有二进制代码，我们就无法拥有硬件，并且我不愿意在这上面投入时间。
> 
> 今天的编译器bug是锦上添花。起初我以为是`launch_bounds`功能的问题，但看起来没有这个功能也能触发。无法信任编译器削弱了整个平台的信任。
> 
> 这将使我们退步，但也许我们应该转向3090或@intel。
> 
> 无论如何，除非解决了这个问题，否则我们不会发布Tinybox（或批量订购7900XTX）。
> 
> ...
> 
> 我确信@AMD也不想要这些bug，但他们的关注点放错了地方。
> 
> 他们应该立即停止开发高端ML库，并解决他们的基础问题。他们的编译器和驱动程序都有bug，为什么要在这些问题解决之前花费一分钟去构建任何东西。
> 
> 这看起来像是测试方法的问题。模糊器将捕获这些问题。在基础良好之前投资更高级的东西是多么浪费。

To which was then

[另一条推文](https://twitter.com/LisaSu/status/1765209899418423751?s=09)

：

> "如果AMD开源他们的固件，我将修复他们的LLVM溢出错误并为HSA编写模糊器。否则，投入大量精力修复一个你不拥有的平台上的bug是不值得的。"

具体来说，他们目前寻求的固件至少是GPU的Micro Engine Scheduler "MES"的固件。

Lisa

[回应了](https://twitter.com/LisaSu/status/1765209899418423751)

：

> "感谢合作和反馈。我们全力以赴为您找到一个好解决方案。团队正在处理中。"

我们将看看这件事会有什么结果，显然今天 Tiny Corp 和 AMD 之间预定了一次电话会议。AMD 最终是否会开源他们的 MES 固件或者有一些临时解决方案将会很有趣。由于法律/代码审查，这不太可能是一个快速的过程，更不太可能他们会开放大片区域的固件。但由于客户的需求，他们一直在通常开源固件方面进行工作，比如支持声音开放固件的支持，有关 CPU 方面的有趣的开放 SIL 努力，以及去年曾经

[将他们的 SEV 固件作为开源发布](https://www.phoronix.com/news/AMD-SEV-Firmware-Open-Source)

. 敬请关注。

**更新：** [Tiny Corp 对 AMD 开源某些相关 GPU 固件有 "70%" 的信心](https://www.phoronix.com/news/Tiny-Corp-70p-MES-Firmware)
