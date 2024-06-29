<!--yml

类别：未分类

日期：2024-05-29 13:21:43

-->

# Rust Bytes：未来是锈蚀的：2023年调查揭示广泛采用情况

> 来源：[https://weeklyrust.substack.com/p/rust-bytes-the-future-is-rusty-2023](https://weeklyrust.substack.com/p/rust-bytes-the-future-is-rusty-2023)

*图片：学习rust螃蟹吉祥物*

你好，Rustacean！欢迎来到另一期Rust Bytes通讯。在本期中，我们将聚焦一款令人惊叹的Rust项目，提出我们的发现错误挑战，并分享本周一些不可思议的链接。

欢迎阅读第11期！

**你如何称呼一个热爱并发的Rust程序员？**

多线程威胁。

**2023年度Rust调查结果**已出炉。以下是主要见解：

+   93%的受访者自认为是Rust用户。在2023年使用Rust的人中，49%几乎每天都在使用。

+   31%的非Rust用户认为其难度是不使用它的主要原因。

+   在2023年调查中参与的前Rust用户中，31%因更喜欢其他语言而停止使用Rust，24%因难度而放弃。

+   在用于Rust开发的IDE中，Visual Studio Code仍然似乎是最受欢迎的选择，而去年发布的RustRover也开始受到一些关注。

+   79%的人报告说Rust帮助他们的公司实现了目标。

+   9,374人分享了他们对Rust未来的主要担忧，其中43%担心Rust变得过于复杂。

查看2023年度[Rust调查结果](https://blog.rust-lang.org/2024/02/19/2023-Rust-Annual-Survey-2023-results.html)以获取更多详细信息。

生命周期技巧：

辨析代码编译失败的原因，并提出修复方法。

想要分享您的解决方案并在我们即将发布的通讯中亮相吗？请在下面的评论中分享您的解决方案，以便包含在即将发布的通讯中。

**clearcheck**旨在使断言语句尽可能清晰简洁。它允许多个断言链在一起，形成流畅直观的语法，从而使测试用例更具自描述性。

特点：

+   流畅API：链式断言，实现自然和可读的体验。

+   广泛的断言：涵盖常见验证需求的多种断言。

+   可定制：根据特定领域要求扩展您自己的断言。

+   类型安全：使用Rust的类型系统构建可靠和表达丰富的断言。

+   自定义断言：为您的确切需求制定断言，确保各种数据结构的全面验证。

该项目由Sarthak Makhija维护，并在[GitHub](https://github.com/SarthakMakhija/clearcheck)上开源。

1.  **Michael Lohr**撰写了关于[“嵌入式Rust在生产中..？使用Rust在嵌入式中超过一年后的评估”](https://blog.lohr.dev/embedded-rust)

1.  在之前的努力基础上，Rust Foundation 发布了其 [“第二个安全倡议报告详细说明 Rust 安全进展”](https://foundation.rust-lang.org/news/second-security-initiative-report-details-rust-security-advancements/) 报告，突出了语言安全性方面的进展。

1.  Sebastian Martinez 在他的文章 ["为什么应该考虑用 Rust 重写您的传统系统"](https://www.wyeworks.com/blog/2024/02/20/why-you-should-consider-rewriting-your-legacy-system-in-rust/) 中探讨了通过利用 Rust 的强大功能为老旧软件注入新生命的令人信服的理由。

1.  Jordan Andrews 的文章 ["Prodzilla: 从零到 Prod 使用 Rust 和 Shuttle"](https://codingupastorm.dev/about/) 详细描述了 Prodzilla 的开发过程，这是一个用 Rust 构建的合成监控工具。它突出了 Prodzilla 的独特功能，例如使用 YAML 测试复杂用户流程等。

1.  **[用 Rust 在 Linux 中编写用户空间调度器](https://arighi.blogspot.com/2024/02/writing-scheduler-for-linux-in-rust.html)**，arighi 的博客详细介绍了 scx_rustland 的开发过程，这是一个用 Rust 为用户空间编写的 Linux 调度器。

1.  这篇博客文章，由 Jakub Beránek、Jack Huey 和 Paul Lenz 撰写，宣布了 [Rust 项目参与 Google 夏季编程活动 (GSoC) 2024](https://blog.rust-lang.org/2024/02/21/Rust-participates-in-GSoC-2024.html)。

1.  Kasper Hermansen 在他的文章 ["使用 Rust 构建服务"](https://blog.kasperhermansen.com/posts/building-business-services-in-rust/) 中认为，Rust 已经准备好用于构建业务服务。

1.  作者 Joshua Mo 探索了 [8 个提升 Rust 开发效率的工具，包括 cargo 插件、测试框架、调试器和性能分析器。](https://www.shuttle.rs/blog/2024/02/15/best-rust-tooling)

1.  RustConf 2024，是 Rust 编程语言社区的年度聚会，将于 9 月 10 至 13 日在加拿大蒙特利尔举行，[请参见此处](https://foundation.rust-lang.org/news/save-the-date-rustconf-2024-september-10-13/)。

1.  [Rust 在 TechEmpower Web 框架基准测试中如何如此快速？](https://kerkour.com/rust-fast-techempower-web-framework-benchmarks)

```
Explanation:

The function greater_of_two takes two string slices (&str) and returns the one with greater length. &str is an immutable reference to a sequence of UTF-8 text owned by someone else. This means &str (/string slice) has a definite lifetime: the stretch of the code for which a reference is valid. 

The return type of the function is also a reference, which means Rust needs a way to define an association between the lifetimes of the input parameters and the lifetime of the return.

Solution:
To fix the compilation error, we need to use the lifetime annotation in the function greater_of_two.

fn greater_of_two<'a>(one: &'a str, other: &'a str) -> &'a str {
    if one.len() > other.len() {
        return one;
    }
    return other;
}

The function signature means that for some lifetime 'a, the input parameters live atleast as long as 'a and the returned reference will also live at least as long as the lifetime 'a.
```

为了支持新闻简报：

暂时就这些，Rustaceans！下周再见，祝你们有一个高效的一周。
