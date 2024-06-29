<!--yml

category: 未分类

date: 2024-05-29 12:44:30

-->

# Modular: The Next Big Step in Mojo🔥 Open Source

> 来源：[https://www.modular.com/blog/the-next-big-step-in-mojo-open-source](https://www.modular.com/blog/the-next-big-step-in-mojo-open-source)

在Modular，开源已深深融入我们的DNA。我们坚信要使Mojo充分发挥其潜力，必须是开源的。我们一直在逐步开放Mojo和MAX平台的部分内容，今天我们很高兴地**宣布将Mojo标准库的核心模块发布为Apache 2许可证！**

[我们一直坚信](https://docs.modular.com/mojo/faq#open-source)，在开放中构建Mojo将会带来更好的结果，因为它允许其设计受到来自广泛社区的反馈塑造。我们非常早期地发布了Mojo，并自2023年5月以来不断推动改进（参见[变更日志](https://docs.modular.com/mojo/changelog)）。构建语言及其基础设施是一项艰苦的工作，需要时间，我们很高兴从分享我们的工作转向与全球Mojo开发者合作。

### Contribute to Mojo🔥 open source

开源代码有许多方式 - 一些项目公开源代码但不接受贡献。一些提供不透明的贡献流程，没有目标和路线图的可见性。有些开源项目然后不积极维护它。

在Modular，我们的团队或者建立过、贡献过或推动了像LLVM、Swift、TensorFlow和PyTorch等令人惊叹的开源项目，我们希望确保我们做得正确。这意味着不仅仅是“公开源代码”，而是真正培育和发展Mojo🔥周围活跃而积极参与的**开发社区**。

除了提供源代码外，我们还**公开修订历史记录**，发布Mojo编译器的**夜间构建**，提供**公共CI**，并允许通过GitHub拉取请求进行**外部贡献**。这是昂贵且不容易设置的，但根据我们的经验，允许社区扩展至关重要。

### License: Apache 2 with LLVM exceptions

我们选择使用[Apache 2.0许可证](https://opensource.org/license/apache-2-0)。它包含**专利授权条款**，为软件的用户和贡献者提供法律保护。Apache 2许可证是广泛使用且经过验证的许可证，对于许多行业内的个人和公司来说都很熟悉。

Apache 2许可证是一个很好的开始，但我们在LLVM项目中的许可证经验告诉我们有两个小问题。一些人担心Apache 2许可证可能与GPL2代码（例如Linux内核）不太兼容，而Apache 2许可证要求您在派生项目中承认使用代码。我们希望您能够在不需要承认Modular或Mojo的情况下使用Mojo（当然，欢迎您这样做！），并明确表示可以与GPL2代码混合使用。因此，我们包含了专门设计用于解决这些问题的[LLVM例外](https://spdx.org/licenses/LLVM-exception.html)。

我们相信这代表了语言和工具项目的开源许可证的最新技术水平，并建议其他Mojo开源项目采用同样的方法。我们已经在我们现有的开源代码中使用了这种方法，并将继续以同样的方法发布更多的代码。

### 我们的开源旅程的早期阶段

Mojo标准库正在快速开发和快速变化，所以我们首先公开了其核心模块。因此，我们认为这是一个重要的起点，而不是我们开源旅程的终点。我们将随着社区的学习一起演进和迭代我们的开发和贡献过程。随着时间的推移，我们将开放更多的代码，包括Mojo的更多部分和更广泛的MAX平台。

### 贡献指南

那么你如何成为Mojo的贡献者呢？标准库中有许多可以改进的地方，所以如果你计划提出拉取请求，请务必查看[Mojic的README.md](https://github.com/modularml/mojo/blob/nightly/README.md)，位于[Mojic GitHub仓库的nightly分支](https://github.com/modularml/mojo/tree/nightly)。它包含了链接到重要文档的链接，所有这些文档都值得阅读：

我们还将向对标准库做出重大贡献的Mojician赠送小礼品和其他福利！

## 夜间版本构建用于快速迭代

Mojo编译器的夜间版本构建现在也已经发布！这使您可以针对库源代码对应的最新Mojo编译器版本测试对标准库的贡献。这是向开放Mojo编译器开发迈出的重要一步，使Mojician们可以选择并测试当前树的最新开发进展。随着时间的推移，我们将扩展这个夜间版本构建，包括整个MAX平台。

请务必查看[标准库开发指南](https://github.com/modularml/mojo/blob/nightly/stdlib/docs/development.md)，了解如何获取夜间版本构建，无论是为了处于我们开发的前沿，还是为了测试您的贡献。

## 版本控制历史

我们不仅发布代码，还公开了我们的标准库提交历史。我们致力于透明，并使您能够看到功能是如何随着时间演变的，这为开发标准库功能提供了额外的背景信息。

## 让我们一起建设未来！

我们对Mojo的未来及其所有应用程序非常兴奋，人们已经在[使用它](https://github.com/mojicians/awesome-mojo)。我们认为这是在开放性和透明度方面迈出的一大步，尽管我们还有很长的路要走。如果您有兴趣做出贡献，请查看[mojo/stdlib中的README](https://github.com/modularml/mojo/blob/nightly/stdlib/README.md)，那里有您开始的所有信息，我们也欢迎在[Discord](http://modul.ar/discord)和[GitHub](https://github.com/modularml/mojo/discussions)上提供反馈。
