<!--yml

类别：未分类

日期：2024-05-27 15:08:31

-->

# 技术债务的年度成本：$1.52万亿 | 作者：Itza Reyes | Medium

> 来源：[https://itzareyesmx.medium.com/the-annual-cost-of-technical-debt-1-52-trillion-65afaa1e0005](https://itzareyesmx.medium.com/the-annual-cost-of-technical-debt-1-52-trillion-65afaa1e0005)

# 技术债务的年度成本：$1.52万亿

*Para leer en Español clic* [*aquí*](/1ead73736a46)*.*

如果您需要更多理由来优先考虑软件质量，请让我与您分享一下，美国低质量软件的成本已经每年上升至至少$2.41万亿，而软件的累积技术债务已经增长到大约$1.52万亿。

这是由[信息与软件质量联盟 (CISQ)](https://www.it-cisq.org/)于2022年进行的一项研究所证明的，并计划于2024年更新。

虽然该报告着重于低质量软件的成本（CPSQ），提供了关于领域的分析、数据和解决方案，例如网络犯罪和软件供应链，以及开源软件（OSS），本文将专门探讨技术债务的主题。

CPSQ 2022

# 📈 技术债务为何产生？

当今存在的大部分技术债务是通过“快速而肮脏”的开发技术（例如，缺乏软件工程纪律的敏捷开发）创建的。

当决策者选择短期解决方案而不是更全面的长期解决方案来解决软件开发问题时，技术债务就会累积。这最初会隐藏组织必须后来支付的巨大成本。

技术债务增加

技术债务有各种类型，包括需求、架构、代码、测试和运行过程。技术债务可以在软件开发的任何阶段注入，并扩散到系统的其他阶段和部分，引发各种问题。

技术债务类型

# 💰 成本

技术债务成本有两个关键组成部分。

+   主要是指对软件构件进行重构/修改以实现所需的可维护性和可演变性的成本。

+   利息是开发人员为进行这些更改而额外投入的努力，这是由于技术债务的存在而积累的，随着软件变得越来越脆弱，这种利息会随时间而增加。在不理想的代码上花费的每一分钟都会为债务增加利息。

# 📏 软件测量

处理技术债务的主要挑战之一是缺乏衡量方式。为了帮助克服这个问题，CISQ/OMG主导了自动化技术债务（[ATD](https://www.omg.org/spec/ATDM/)）测量标准的开发，该标准目前正在更新，预计将于2023年推出新版本。

ATD 标准估计了解决 ISO/IEC 5055:2021 标准中包含的所有软件弱点实例所需的工作量，这些弱点在软件应用发布时仍然存在于代码中。

这一估算可用于预测未来的纠正性维护成本，是使用静态分析工具计算的。

该措施通过估算纠正代码结构缺陷的未来维护成本来表达软件质量的成本。

# 👀 应对建议

在报告中提出了几项建议，并在最后提供了一个更具体的建议清单，以避免软件开发中的质量低下，包括技术债务。

+   使用软件质量标准、相关指标和新兴工具。

+   分析和评估将包含在任何系统中的所有 OSS/第三方组件的质量。在运行过程中密切监控它们。及时应用补丁。

+   避免不包含持续质量工程的最佳实践和工具的 DevOps 和 CI/CD 模型。

+   将持续技术债务补救融入软件开发生命周期。

+   投资于您的软件工程师的专业知识和工具。

+   考虑开发人员在 ISO/IEC 5055 中认证其对关键代码和架构缺陷的了解的可能性（仍在进行中）。

# 🌟 结论

+   技术债务的不断增长已成为更改现有代码库的最大障碍。

+   有许多方法可以实现更好的软件质量，但它们都始于一个由高层管理完全支持的精心构思的测量方案。

+   管理技术债务的潜力在 [Stepsize](https://www.stepsize.com/) 的研究中显而易见，该研究发现，积极管理技术债务的组织将至少释放 50% 更快。

**正如我们在 CPSQ 中所看到的那样，防止或早期消除技术债务是最具成本效益的长期策略。**

# 🔗 资源

[https://www.it-cisq.org/the-cost-of-poor-quality-software-in-the-us-a-2022-report/](https://www.it-cisq.org/the-cost-of-poor-quality-software-in-the-us-a-2022-report/)

[https://www.theee.ai/2021/01/06/6838-poor-software-quality-cost-the-usd-2-08-tn-in-2020/](https://www.theee.ai/2021/01/06/6838-poor-software-quality-cost-the-usd-2-08-tn-in-2020/)