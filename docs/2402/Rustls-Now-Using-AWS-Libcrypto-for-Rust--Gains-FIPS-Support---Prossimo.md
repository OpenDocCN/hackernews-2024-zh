<!--yml

category: 未分类

date: 2024-05-29 13:29:01

-->

# Rustls现在使用AWS Libcrypto for Rust，并获得FIPS支持 - Prossimo

> 来源：[https://www.memorysafety.org/blog/rustls-with-aws-crypto-back-end-and-fips/](https://www.memorysafety.org/blog/rustls-with-aws-crypto-back-end-and-fips/)

截至今日，[Rustls TLS库](https://github.com/rustls/rustls)默认使用AWS Libcrypto for Rust（[aws-lc-rs](https://github.com/aws/aws-lc-rs)）进行加密，并有启用FIPS支持的选项。这为许多组织中更安全的TLS消除了重大障碍。

在过去的几年中，我们清楚地认识到，为了使Rustls的最佳版本能够更广泛地受众，我们需要对提供的加密支持进行改进。第一步是引入可插拔加密，即在构建时选择加密后端的支持能力。这项工作始于2023年第三季度，现已完成。您现在可以选择使用[aws-lc-rs](https://github.com/aws/aws-lc-rs)或[*ring*](https://github.com/briansmith/ring/)构建Rustls进行加密。社区已开始为Rust Crypto、Mbed TLS和BoringSSL添加加密支持。我们希望很快能支持[SymCrypt](https://github.com/microsoft/SymCrypt)。

我们选择将aws-lc-rs作为新的默认选项，因为它是一个具有FIPS支持的高质量实现。在我们需要时，AWS加密团队已经开发了出色的Rust绑定，与他们的团队合作非常愉快。我们也欣赏他们的项目是开源的，并在GitHub上公开托管和开发。

在我们进行可插拔加密工作的同时，ISRG委托Adolfo Ochagavía为Rustls构建了[全面的基准测试](https://www.memorysafety.org/blog/rustls-performance/)。这个系统有两个目的：1）帮助防止性能退化，2）告诉我们Rustls性能如何与其他库相比。它已经阻止了一些可能导致性能退化的代码合并，关于Rustls性能优势，我们将很快有更多的话要说。

最近，我们为互联网的更安全的TLS实现迈出了[重要的一步](https://www.memorysafety.org/initiative/rustls/)，我们对我们工作的[下一阶段](https://github.com/rustls/rustls/blob/main/ROADMAP.md)充满期待，包括一个OpenSSL兼容层。OpenSSL兼容层将允许Rustls作为许多OpenSSL用户的即插即用替代品。

拥有来自世界各地资助者社区的支持，Prossimo能够承担起重写互联网关键组件的具有挑战性的工作。我们要感谢AWS、主权科技基金、Google、Fly.io和Alpha-Omega对我们在Rustls上工作的支持。
