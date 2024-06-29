<!--yml

类别：未分类

日期：2024-05-27 14:44:39

-->

# 谷歌在线安全博客：安全设计：谷歌对内存安全的看法

> 来源：[https://security.googleblog.com/2024/03/secure-by-design-googles-perspective-on.html](https://security.googleblog.com/2024/03/secure-by-design-googles-perspective-on.html)

Google的Project Zero [报告](https://googleprojectzero.blogspot.com/2022/04/the-more-you-know-more-you-know-you.html)指出，由于与程序访问内存相关的微妙编码错误引起的内存安全漏洞，已经成为“过去几十年攻击软件的标准，并且攻击者仍然通过这种方式取得成功”。他们的分析显示，在野外检测到的0-day漏洞利用中，三分之二使用了内存破坏漏洞。尽管进行了大量投资以改善内存不安全语言，但这些漏洞仍然居于[最常被利用的漏洞类别之首](https://cwe.mitre.org/top25/archive/2023/2023_kev_list.html)。

在这篇文章中，我们分享了我们对内存安全的观点，详述了数据、解决内存不安全问题的挑战，并讨论了实现内存安全及其权衡的可能方法。我们还将重点介绍我们承诺实施的几种解决方案，最近包括向[Rust基金会提供100万美元的资助](https://security.googleblog.com/2024/02/improving-interoperability-between-rust-and-c.html)，从而推动健全的内存安全生态系统的发展。

# 为什么我们现在发布这篇文章

2022年标志着内存安全漏洞50周年。自那时以来，内存安全风险变得更加明显。与其他公司一样，谷歌的内部漏洞数据和研究显示，内存安全错误在内存不安全代码库中广泛存在，是漏洞的主要原因之一。这些漏洞危及最终用户、我们的行业和更广泛的社会。我们看到政府也认真对待这个问题，例如美国国家网络主任办公室最近发布了一篇关于这一主题的[论文](https://www.whitehouse.gov/wp-content/uploads/2024/02/Final-ONCD-Technical-Report.pdf)。

通过分享我们的见解和经验，我们希望激励更广泛的社区和行业采纳内存安全的实践和技术，从而最终使技术更加安全。

# 我们的观点

在Google，我们有数十年的经验，规模化地解决了一类类似于内存安全问题的漏洞。我们的方法被称为“[安全编码](https://research.google/pubs/pub53116/)”，将易受漏洞影响的编码结构本身视为危险因素（即独立于可能引起的漏洞之外），并且致力于确保开发人员在日常编码实践中不会遇到这些危险因素。

基于这一经验，我们预计只有通过以[安全设计方法](https://blog.google/technology/safety-security/tackling-cybersecurity-vulnerabilities-through-secure-by-design/)为中心，全面采用具有严格内存安全保证的语言，才能实现高可靠性的内存安全。因此，我们正在考虑逐步过渡到像Java、Go和Rust这样的内存安全语言。

在过去的几十年里，除了大规模的Java和Go内存安全代码库外，Google还开发和积累了数亿行正在积极使用和持续开发的C++代码。这个非常庞大的现有代码库对于过渡到内存安全存在重大挑战：

+   我们认为没有现实的路径可以将C++演变为具有包括时间安全在内的严格内存安全保证的语言。

+   将所有现有C++代码大规模重写为不同的、内存安全的语言看起来非常困难，可能仍然不切实际。

我们认为对于新代码以及特别是风险组件的过渡到内存安全语言非常重要，同时还要改进现有C++代码的安全性，尽可能地。我们相信通过逐步过渡到部分内存安全的C++语言子集，并在可用时增加硬件安全功能，可以实现显著的改进。例如，请参阅我们在GCP网络堆栈中改进[空间安全性的工作](https://bughunters.google.com/blog/6368559657254912/llvm-s-rfc-c-buffer-hardening-at-google)。

# 我们对内存安全语言的投资

我们正在积极投资于我们白皮书中概述的多种解决方案，并在我们对美国联邦政府关于开源软件安全的[响应](https://www.regulations.gov/comment/ONCD-2023-0002-0074)中进行投资。

+   Android在过去几年中在Rust中编写了几个组件，导致了[引人注目的安全改进](https://security.googleblog.com/2022/12/memory-safe-languages-in-android-13.html)。在Android的超宽带（UWB）模块中，这不仅提高了模块的安全性，还减少了内存使用和程序间调用。

+   Chrome已经[开始在Rust中实现一些功能](https://groups.google.com/a/chromium.org/g/chromium-dev/c/UhwVDk4HZFA/m/UAA2D96QBAAJ)；其中一个案例中，Chrome通过采用用Rust编写的新的内存安全库，将其QR码生成器移出沙盒，从而既提升了安全性，又提升了性能。

+   谷歌最近宣布向[Rust基金会提供100万美元的资助](https://security.googleblog.com/2024/02/improving-interoperability-between-rust-and-c.html)以增强与C++代码的互操作性。这将促进Rust在现有的内存不安全代码库中的渐进采用，这对于在内存安全语言中进行更多新开发至关重要。与此相关的是，我们还在解决[Rust和C++混合在同一二进制中可能导致的跨语言攻击](https://bughunters.google.com/blog/4805571163848704/llvm-cfi-and-cross-language-llvm-cfi-support-for-rust)。

我们知道，内存安全语言并不能解决所有安全漏洞，但就像我们通过工具化消除[跨站脚本攻击](https://bughunters.google.com/blog/5896512897417216/a-recipe-for-scaling-security)一样，消除大量的漏洞类别不仅直接造福软件消费者，还使我们能够将重点转移到解决更多类别的安全漏洞上。

要获取完整的白皮书，并了解更多关于谷歌对内存安全的看法，请访问[https://research.google/pubs/secure-by-design-googles-perspective-on-memory-safety/](https://research.google/pubs/secure-by-design-googles-perspective-on-memory-safety/)
