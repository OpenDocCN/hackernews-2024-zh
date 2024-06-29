<!--yml

分类：未分类

日期：2024-05-27 15:04:07

-->

# C++ 语言的创始人回击白宫的警告 | InfoWorld

> 来源：[https://www.infoworld.com/article/3714401/c-plus-plus-creator-rebuts-white-house-warning.amp.html](https://www.infoworld.com/article/3714401/c-plus-plus-creator-rebuts-white-house-warning.amp.html)

C++ 语言的创始人 Bjarne Stroustrup 在回应 [拜登政府的一份报告](https://www.infoworld.com/article/3713203/white-house-urges-developers-to-dump-c-and-c.html) 时捍卫了这种广泛使用的编程语言，报告呼吁开发者使用内存安全的语言，并避免使用易受攻击的语言，比如 [C++](https://www.infoworld.com/article/3688923/c-23-language-standard-declared-feature-complete.html) 和 [C](https://www.infoworld.com/article/3402023/why-the-c-programming-language-still-rules.html)。

在 InfoWorld 对其进行的调查后，Stroustrup 在 3 月 15 日的回应中指出了 C++ 的优势，该语言于 1979 年设计。他说：“我发现那些政府文件的作者似乎忽略了当代 C++ 结构的优势以及提供强有力的安全性保证的努力。与此同时，他们似乎意识到，编程语言只是工具链的一部分，因此改进的工具和开发过程至关重要。”

安全改进一直是 C++ 开发工作的一个目标，Stroustrup 强调说：“从一开始的 C++ 到其演变过程中都一直致力于提高安全性。只需比较 K&R C 语言与最初的 C++，以及早期的 C++ 与当代的 C++ 就可知晓。我的 [CppCon 2023 主题演讲](https://youtu.be/I8UvQKvOSSw?si=HA4s8pXHg1V9J9Xy) 概述了这一演变过程。”他说：“很多优质的 C++ 代码都是采用基于 RAII（资源获取即初始化）、容器和资源管理指针的技术编写的，而不是传统的 C 风格指针混乱。”

白宫在一份 [于2月26日发布的报告中提到](https://www.whitehouse.gov/wp-content/uploads/2024/02/Final-ONCD-Technical-Report.pdf)，呼吁开发者通过使用没有内存安全漏洞的编程语言来降低网络攻击风险。 其中，C++ 和 C 被列举为存在内存安全漏洞的两种语言。美国国家安全局（NSA）在 2022 年 11 月发布的一份 [网络安全信息简报](https://media.defense.gov/2022/Nov/10/2003112742/-1/-1/0/CSI_SOFTWARE_MEMORY_SAFETY.PDF) 中，将 C#、Go、Java、Python 和 Rust 等语言列为被认为是内存安全的语言。

Stroustrup 引用了多项改进 C++ 安全性的努力。"关于安全性有两个问题。在数十亿行的 C++ 代码中，几乎没有一行完全遵循现代指南，而人们对安全性的重视程度各不相同。我和 C++ 标准委员会正在努力解决这个问题，" 他说。"[Profiles](https://www.open-std.org/jtc1/sc22/wg21/docs/papers/2023/p3038r0.pdf) 是一个指定代码所需保证并使实现能够验证它们的框架。委员会的网站上有相关文件——查找 [WG21](https://www.open-std.org/jtc1/sc22/wg21/) ——而且还会有更多文件发布。不过，有些人对委员会的进展速度不太满意，不愿等待。"

Profiles，Stroustrup 表示，"是一个允许我们逐步改进保证的框架——例如，通过局部静态分析和最小运行时检查，可以相对快速消除大部分范围错误，并逐步在大型代码库中引入保证。我对 C++ 的长期目标一直是在需要的地方提供类型和资源安全性。也许当前对内存安全性的推动——这是我想要的保证的一个子集——会对我的努力有所帮助，这些努力得到了 C++ 标准委员会的支持。"

Stroustrup [此前为了捍卫 C++ 的安全性](https://www.infoworld.com/article/3686517/c-plus-plus-creator-bjarne-stroustrup-defends-its-safety.html) 曾对抗 NSA，后者在 [2022 年 11 月的公告](https://media.defense.gov/2022/Nov/10/2003112742/-1/-1/1/CSI_SOFTWARE_MEMORY_SAFETY.PDF) 中建议使用内存安全语言而非 C++ 和 C。
