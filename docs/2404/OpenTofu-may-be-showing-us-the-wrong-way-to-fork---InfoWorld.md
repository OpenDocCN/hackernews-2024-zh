<!--yml

category: 未分类

date: 2024-05-27 12:52:26

-->

# OpenTofu可能正在向我们展示分支的错误方式 | InfoWorld

> 来源：[https://www.infoworld.com/article/3714980/opentofu-may-be-showing-us-the-wrong-way-to-fork.html](https://www.infoworld.com/article/3714980/opentofu-may-be-showing-us-the-wrong-way-to-fork.html)

*更新：自本文发布以来，HashiCorp于2024年4月3日向OpenTofu发出了停止侵权的信函，详细阐述了本文中提出的关切。2024年4月11日，OpenTofu的维护者对HashiCorp的停止侵权信函作出了[详细分析](https://nam10.safelinks.protection.outlook.com/?url=https%3A%2F%2Fopentofu.org%2Fblog%2Four-response-to-hashicorps-cease-and-desist%2F&data=05%7C02%7Cdoug_dineley%40foundryco.com%7C55d3b6fb941246810d2708dc5a50aab5%7C6b18947b63e74323b637418f02655a69%7C0%7C0%7C638484549708243844%7CUnknown%7CTWFpbGZsb3d8eyJWIjoiMC4wLjAwMDAiLCJQIjoiV2luMzIiLCJBTiI6Ik1haWwiLCJXVCI6Mn0%3D%7C0%7C%7C%7C&sdata=MmodPmWoouqhpHiQRDa7sfyBSUKtiB7jfQNbJM%2Fi%2FPk%3D&reserved=0)，说明OpenTofu社区并未侵犯HashiCorp的知识产权。*

OpenTofu的创始人有一个使命。在2023年8月HashiCorp对其广受欢迎的Terraform基础设施即代码工具进行许可证更改后，[OpenTofu开始了](https://www.linuxfoundation.org/press/announcing-opentofu)“成为MPLv2许可证Terraform的开源继承者”的使命，并承诺“将是社区驱动的、公正的、分层和模块化的，并且向后兼容”。

极具前景，但难以实现。实际上，OpenTofu可能非法使用了HashiCorp的代码以保持竞争力。

至少，浏览OpenTofu的GitHub存储库并将其与HashiCorp的进行比较，很难避免得出这样的结论。具体来说，OpenTofu似乎抄袭了与Terraform V1.7中首次实现的新的`removed`块功能相关的代码，后者在创建OpenTofu分支几个月后以Business Software License（BUSL）发布。这个细节在哪里呢？OpenTofu采用了这些BUSL许可的HashiCorp代码，去掉了头部，并试图将其重新许可为Mozilla Public License（MPL 2.0）。

伙计们，开源不是这样的。你可以不同意版权持有者选择的许可证，但你没有权利拿别人的代码，撕掉原有许可证，然后替换成你自己的许可证。

## 青春的傲慢

OpenTofu在2023年9月隆重推出，并获得了来自140多个组织的“正式承诺”支持，其中包括Cloudflare、Harness、Oracle和GitLab。当然，[核心维护者](https://github.com/opentofu/opentofu/blob/main/MAINTAINERS)主要来自直接的HashiCorp竞争对手（如Spacelift、env0），这些竞争对手以Terraform为基础建立了业务，并因HashiCorp的许可证更改而感到不满。这也可以理解。

到了一月份，该项目正在[宣布 OpenTofu 的普遍可用性](https://www.linuxfoundation.org/press/opentofu-announces-general-availability)，同时还在提到像客户端状态加密这样 Terraform 没有的即将发布的功能。然而，尽管乐观的开端，团队很快就[开始意识到实施这一功能的困难](https://github.com/opentofu/opentofu/issues/874)。安全难度很大。（也许 HashiCorp 并不笨。）

如果这样的开发速度听起来太好以至于不真实，那么也许确实如此。毕竟，无论人们如何看待 HashiCorp 的许可变更，该公司花了十年时间打造这款产品。这样一项工作背后的工程力量不会在几个月内突然涌现，无论创始人的高飞理想多么高。

## 许可魔法

在 Terraform V1.7 中，[HashiCorp 引入](https://github.com/hashicorp/terraform/blob/v1.7/CHANGELOG.md) 了一个重大的新功能：`removed` 块自动化，使 Terraform 能更好地管理资源删除。可以把它看作是 [基于配置的方法](https://developer.hashicorp.com/terraform/language/resources/syntax#removing-resources) 来执行 `terraform state rm`。然而，这个功能本身虽然很酷，但这不是重点。重要的是，这个功能是在 2023 年 11 月底 HashiCorp 转向 BUSL 许可之后引入的。如果有人想要使用 `removed` 块功能，他们不能在 MPL 下获取它。

到了二月底，OpenTofu 推出了类似于 HashiCorp 删除块自动化的功能。不仅仅是在功能上，还包括实现它所编写的代码。看看这些存储库，告诉我你是否看到了相同的东西：

版权法律很复杂。我本人背景是律师，但并不从事实践，因此不能算是一个很好的律师。也许 OpenTofu 似乎在一些文件中删除了一些评论是有影响的。也许改变了这里或那里的一行是有影响的。也许可以很有信服力地争辩说，OpenTofu 实际上并没有创建 Terraform 的 BUSL 许可代码的衍生作品。也许。

然而，当你看看 OpenTofu 在文件头部的声明时，这样的论点就不那么有说服力了。这里是 HashiCorp 在其 `removed` 块文件上使用的标题：

// 版权所有 HashiCorp, Inc.

// SPDX-License-Identifier: BUSL-1.1

现在这里是 OpenTofu 使用的标题：

// 版权所有 2023 HashiCorp, Inc.

// SPDX-License-Identifier: MPL-2.0

看到问题了吗？OpenTofu 承认它在使用 HashiCorp 的代码，但假装相关代码是根据 MPL 许可证发布的。但实际情况并非如此。所有相关代码都是在 HashiCorp 将 Terraform 移至 BUSL 许可证之后发布的。最好的解释是，OpenTofu 社区正在进行一种一厢情愿的思维，绝望地希望能够让 BUSL 许可证下的代码神奇地成为 MPL 许可证下的代码。最糟糕的情况是，OpenTofu 的开发者们欺骗性地侵占了 HashiCorp 的知识产权，并试图使其成为他们自己的。

无论 OpenTofu 的开发者们认为什么，这种行为都与积极的“社区驱动方法”背道而驰，绝对不显示出像 Linux 基金会新闻稿中所宣称的“开源的价值”。这看起来更像是对 HashiCorp 知识产权的侵犯。OpenTofu 可以不同意 HashiCorp 的许可证更改并分叉该项目；但 OpenTofu 或其他任何人拿走 HashiCorp 的代码并随意应用他们喜欢的许可证是完全违法的。

这感觉像是治理失败，还有其他问题。云闪、甲骨文和其他负责任的公司绝对不会支持这样的社区，但似乎他们正在得到这样的结果。

版权所有 © 2024 IDG通讯公司。
