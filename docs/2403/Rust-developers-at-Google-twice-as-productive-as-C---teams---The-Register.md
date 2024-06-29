<!--yml

category: 未分类

date: 2024-05-29 12:49:32

-->

# Rust 开发人员在谷歌的生产力是 C++ 团队的两倍 • The Register

> 来源：[https://www.theregister.com/2024/03/31/rust_google_c/](https://www.theregister.com/2024/03/31/rust_google_c/)

在过去两年的 Rust 宣传和对 C/C++ 的厌倦中，Google 报告称 Rust 在生产中表现出色，其开发人员使用该语言的生产力是使用 C++ 的两倍。

在本周伦敦举行的 [Rust Nation UK Conference](https://www.youtube.com/live/6mZRWFQRvmw?feature=shared&t=26575) 上发言的 Lars Bergstrom 是谷歌工程主管，负责 Android 平台工具与库，他描述了这家网络巨头将项目从 Go 或 C++ 迁移到 Rust 编程语言的经验。

Bergstrom 提到，尽管 [2016 年的 Dropbox](https://blog.rust-lang.org/2016/05/16/rust-at-one-year.html) 和 [2018 年的 Figma](https://www.figma.com/blog/rust-in-production-at-figma/) 提供了早期关于在内存安全的 Rust 中重写代码的案例，对生产力和语言的疑虑已经消失，但对其可靠性和安全性仍然存在疑虑。

“就在六个月前，这真是一次非常艰难的对话，”他说。“我会去和人们交流，他们会说，‘等等，你有一个 `unsafe` 关键字。这意味着我们应该一直写 C++ 直到宇宙热死。’”

但是，Bergstrom 认为，在软件开发生态系统中，对于使用非内存安全语言的挑战的认识发生了变化。现在，来自美国和其他国家的政府机构也开始意识到软件在关键基础设施中的作用。

[绝大多数](https://alexgaynor.net/2020/may/27/science-on-memory-unsafety-and-security/)大型代码库中的安全漏洞可以追溯到内存安全 bug。而如果正确实现，Rust 代码可以在很大程度上，如果不是完全地避免这些问题，内存安全问题现在看起来很像是国家安全问题。

2022 年 9 月，Microsoft Azure CTO Mark Russinovich 认为，可能已经在 C/C++ 中开始的软件项目 [应该使用](https://www.theregister.com/2022/09/20/rust_microsoft_c/) Rust 替代。现在这一推荐不仅适用于新项目，还包括重写以非内存安全语言编写的旧代码的呼声。

今年早些时候，Microsoft 呼吁开发人员帮助 [将](https://www.theregister.com/2024/01/31/microsoft_seeks_rust_developers/) 其自己的 C# 代码移植到 Rust。而互联网安全研究组织 (ISRG) 的 Prossimo 项目一直在用 Rust 重写关键库的核心开源元素（例如 NTP、DNS、TLS）以保证 [内存安全](https://www.memorysafety.org/)。

C++ 的创始人比雅尼·斯特劳斯特鲁普（Bjarne Stroustrup）及其他人对此提出了异议。针对2022年11月NSA的[备忘录](https://media.defense.gov/2022/Nov/10/2003112742/-1/-1/1/CSI_SOFTWARE_MEMORY_SAFETY.PDF) [PDF] 呼吁内存安全，斯特劳斯特[辩称](https://www.open-std.org/jtc1/sc22/wg21/docs/papers/2023/p2739r0.pdf) [PDF] ，通过适当的工具，C++ 可以在成本仅为各种新型“安全”语言的一小部分的情况下匹配 Rust 的内存安全保证。

在二月份，当美国国家网络主任办公室发布了[一份报告](https://www.whitehouse.gov/wp-content/uploads/2024/02/Final-ONCD-Technical-Report.pdf) [PDF] 关于安全软件时，一些[公开评论](https://www.regulations.gov/document/ONCD-2023-0002-0001/comment)指出，内存安全是更广泛的软件安全挑战的一部分，不应被视为对所有问题的回答。

例如，卡内基梅隆大学软件工程研究所强调，所有编程语言都有权衡，选择编程语言应该取决于它是否适合特定目的。

“大多数内存安全的语言不重视时序性能，因此不适合对有严格性能和时序要求的用例。” 软件组[声称](https://www.regulations.gov/comment/ONCD-2023-0002-0068)。

“同样，与任何编程语言一样，开发人员必须学习语言的正确机制，如语法、语义、结构、习语和工具。否则，结果可能是以牺牲少量与内存相关的漏洞来换取其他类型的漏洞或缺陷。”

尽管软件安全不仅仅局限于内存安全，但斯特劳斯特声称的成本优势现在面临来自像谷歌这样的 Rust 采用者的反例。

### 不会因此而降低生产力 - 恰恰相反。

在巧克力工厂，将被认为是[内存安全的](https://www.memorysafety.org/docs/memory-safety/)但性能不如优秀的 Go 代码转换为 Rust，显示出显著的好处。

“当我们将系统从 Go 重写为 Rust 时，发现用相同规模的团队、相同的时间来构建。” [Bergstrom 说道](https://www.youtube.com/live/6mZRWFQRvmw?feature=shared&t=27048)。“也就是说，从 Go 到 Rust 的转换不会导致生产力下降。有趣的是我们确实看到了一些好处。”

“因此，我们看到我们已经从 Go 迁移到 Rust 的服务中减少了内存使用……我们看到这些服务在 Rust 中被重写后随着时间的推移缺陷率降低 - 因此提高了正确性。”

更重要的是，Bergstrom 表示，比较将 C++ 代码重写为 Rust 的情况。

"在我们看到的每一个案例中，使用Rust构建服务以及维护和更新这些服务的工作量都减少了超过2倍，" 他 [说](https://www.youtube.com/live/6mZRWFQRvmw?feature=shared&t=27094)。

"因此，对我们来说这是一件非常重要的事情，因为C++代码非常昂贵。这些是庞大的团队。这是很多工作。存在很大的风险。"

Bergstrom说，Google正在进行类似的迁移，将开发者从Java迁移到Kotlin，而在这两种情况下重新培训开发者所需的时间——从Java到Kotlin以及从C++到Rust——是相似的。也就是说，大约两个月后，三分之一的开发者感觉在他们的新语言上与旧语言一样高效。大约四个月后，半数开发者表示如此，基于匿名内部调查。

大约一半以上的开发者说Rust更容易审查，据Bergstrom称。

"当我们试图探究其原因时，" 他说道，"我们发现了调查中最令人难以置信的问题，这个问题让我们所有人都震惊，那就是人们对他们所看到的Rust代码正确性的自信程度——与其他语言的代码相比，你觉得你的团队的Rust代码有多正确？"

答案，Bergstrom 说，是85%。

"这是一个巨大的数字，" 他说道。"我无法让这个房间里的85%的人同意我们喜欢M&M'S巧克力豆。85%的人相信，他们的Rust代码比系统中的其他代码更可能是正确的。……我在生活中经历过多次语言调查，但我从未见过这种数字。" ®
