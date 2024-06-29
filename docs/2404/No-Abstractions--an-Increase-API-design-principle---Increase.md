<!--yml

category: 未分类

date: 2024-05-27 13:31:19

-->

# 无抽象：Increase API 设计原则 — Increase

> 来源：[https://increase.com/articles/no-abstractions](https://increase.com/articles/no-abstractions)

API 资源

是您的 API 的名词。决定如何命名和建模这些名词可以说是设计 API 中最困难和最重要的部分。您公开的资源组织了用户对产品工作方式和功能的心理模型。在 Increase，我们的团队使用了一个叫做“无抽象”的原则来帮助。这是什么意思？

我们团队的大部分成员来自 Stripe，在设计我们的 API 时，我们考虑了在那里取得成功的相同价值观。Stripe 擅长设计

抽象

在他们的 API 中 — 将复杂领域的基本特征提取为他们的用户可以轻松理解和处理的内容。在他们的案例中，这主要意味着

[建模](https://stripe.com/blog/payment-api-design)

通过许多不同网络的支付合并到一个名为 API 资源的

[PaymentIntent](https://docs.stripe.com/api/payment_intents)

。例如，Visa 和 Mastercard 对于为何可以发起退款有微妙不同的原因代码，但 Stripe 将这些代码合并为单个枚举，使他们的用户无需分别考虑这两个网络。

这是有道理的，因为 Stripe 的许多用户是从事与支付无关的产品的初创企业。他们不一定知道或需要了解信用卡的微妙之处。他们想要快速集成 Stripe，回到构建他们的产品，不再思考支付问题。

“对 Increase 用户来说，试图隐藏这些网络的基本复杂性会激怒他们，而不是简化他们的生活。”

Increase 的用户并非如此。他们经常对支付网络有深入的现有知识，始终考虑金融技术，并因我们的直接网络连接和允许他们构建的深度集成而来找我们。他们想知道

[确切](https://status.increase.com/#ach-submission-timeline)

当联邦 ACH 窗口关闭和转账何时到账。他们明白在 ACH 转账上设置不同的标准输入类别代码可能导致不同的返回时间。试图隐藏这些网络的基本复杂性（例如，通过一个 API 资源对 ACH 转账和电汇进行建模）会激怒他们，而不是简化他们的生活。

与这些用户的早期对话帮助我们在构建 API 的第一个版本时表达了我们所谓的“无抽象”原则。一些例子说明了这种思维方式后来如何影响了其设计：

现实世界的命名

我们不会为 API 资源及其属性发明自己的名称，而是倾向于使用底层网络的词汇。例如，通过我们的 API 进行 ACH 转账时，我们公开的参数是根据

[Nacha 规范](https://achdevguide.nacha.org/ach-file-details)

。

不可变性

与我们使用网络命名方式类似，我们尝试模仿真实世界中的事件，如已执行的操作或发送的消息来建模我们的资源。这导致我们的 API 资源中的更多资源是不可变的。对我们的 API 起作用的一种方法是将这些不可变资源聚合在一起（例如，可以作为 ACH 转账生命周期的一部分发送的所有网络消息）并在状态机“生命周期对象”下进行分组。例如，我们的 API 中的 `ach_transfer` 对象具有一个随时间变化的 `status` 字段，并且作为转账通过其生命周期创建的几个不可变子对象。一个新铸造的 `ach_transfer` 对象如下所示：

```
{
  "id": "ach_transfer_abc123",
  "created_at": "2024-04-24T00:00:00+00:00",
  "amount": 1000,
  "status": "pending_approval",
  "approval": null,
  "submission": null,
  "acknowledgement": null

}
```

在同一转账经过我们的管道并且我们已将其提交到 FedACH 之后，它看起来像：

```
{
  "id": "ach_transfer_abc123",
  "created_at": "2024-04-24T00:00:00+00:00",
  "amount": 1000,
  "status": "submitted",

  "approval": {
    "approved_by": "administrator@yourcompany.com",
    "approved_at": "2024-04-24T01:00:00+00:00"
  },

  "submission": {
    "trace_number": "058349238292834",
    "submitted_at": "2024-04-24T02:00:00+00:00"
  },

  "acknowledgement": {
    "acknowledged_at": "2024-04-24T03:00:00+00:00"
  }

}
```

通过用例分离资源

如果对于给定的 API 资源，用户可以对不同实例执行的操作集合差异很大，我们倾向于将其拆分为多个资源。例如，您可以在发起的 ACH 转账上执行的操作集合与您可以在接收的 ACH 转账上执行的操作截然不同，因此我们将其分开成 `ach_transfer` 和 `inbound_ach_transfer` 资源。

* * *

这种方法可能会使我们的 API 在第一眼看起来更冗长和令人生畏 - 我们

[文档](https://increase.com/documentation/api)

页面！我们认为这会使事情在长期内更加可预测。

重要的是，我们的工程团队已经承诺采取这种方法。在多年设计复杂 API 时，你会一直做出小的渐进式决策。最初承诺基础原则已减少了这些决策的认知负荷。例如，当向联邦储备发送电汇时，有一个必填字段叫做

[输入消息责任数据](https://increase.com/documentation/fedwire#technical-implementation)

作为该转账的全局唯一标识符。在构建支持电汇的时候，一个在抽象层面很重的 API 中的工程师可能需要仔细考虑如何以“用户友好”的方式命名这个字段 -

`trace_number`

？

`reference_number`

？

`id`

？在 Increase 中，这位假设的工程师给这个字段命名

`input_message_accountability_data`

并继续前进。当 Increase 用户第一次遇到这个字段时，虽然它可能不是最容易识别的名称，但它立即帮助他们理解这如何映射到底层系统。

不是每个 API 都适合没有抽象化，但考虑对接 API 的开发者适合的抽象层次是一个有价值的练习。这将取决于他们在处理您的产品领域方面的经验水平以及他们将投入到集成中的精力，等等。如果您正在构建一个抽象层次很高的 API，请在添加新功能之前做好深思熟虑。如果您正在构建一个抽象层次较低的 API，请坚定地维持，并抵制在遇到诱惑时添加抽象化的冲动。
