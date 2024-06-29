<!--yml

category: 未分类

date: 2024-05-27 13:29:51

-->

# 宣布Stainless SDK生成器

> 来源：[https://www.stainlessapi.com/blog/announcing-the-stainless-sdk-generator](https://www.stainlessapi.com/blog/announcing-the-stainless-sdk-generator)

Stainless为OpenAI、Anthropic、Cloudflare等公司生成官方客户端库。今天，我们将Stainless SDK生成器提供给每一个使用REST API的开发者。

在我们的私人测试版期间，我们已经能够帮助数百万开发者更快速、更可靠地集成一些全球最强大和最令人兴奋的API的最新功能。

Stainless SDK生成器接受OpenAPI规范，并用其生成多种编程语言的高质量SDK。随着您的API发展，我们的自动化生成器会持续推动变更，确保您的SDK始终保持最新——即使您对生成的代码进行了[任意的自定义编辑](https://app.stainlessapi.com/docs/guides/patch-custom-code)。

这里有一个快速的真实代码示例，以及你如何使用Stainless进行配置：

‍

```
 from cloudflare import Cloudflare

client = Cloudflare()
client.zones.create(account={"id": "xxxx"}, name="my zone", type="full") 

```

```
 import Cloudflare from "cloudflare";

async function main() {
  const cloudflare = new Cloudflare();
  const zone = await cloudflare.zones.create({
    account: { id: "xxx" },
    name: "example.com",
    type: "full",
  });
} 

```

```
 package main

import (
    "context"

    "github.com/cloudflare/cloudflare-go/v2"
    "github.com/cloudflare/cloudflare-go/v2/zones"
)

func main() {
    client: = cloudflare.NewClient()
    zone,
    err: = client.Zones.New(context.Background(), zones.ZoneNewParams {
        Account: cloudflare.F(zones.ZoneNewParamsAccount {
            ID: cloudflare.F("023e105f4ecef8ad9ca31a8372d0c353"),
        }),
        Name: cloudflare.F("example.com"),
        Type: cloudflare.F(zones.ZoneNewParamsTypeFull),
    })
} 

```

```
 resources:
  zones:
    methods:
      create: post /v1/zones 

```

*查看这些端点的源代码，使用* [*TypeScript*](https://github.com/cloudflare/cloudflare-typescript/blob/54faa1bd35ae5f394e37e8ac7f75c36f738baff9/src/resources/zones/zones.ts#L27-L34)*,* [*Python*](https://github.com/cloudflare/cloudflare-python/blob/77cd2adc0ea0e48fb7419dfe6d48a7a5af78f35c/src/cloudflare/resources/zones/zones.py#L129-L177)*,* 和 [*Go*](https://github.com/cloudflare/cloudflare-go/blob/21580812b4a3fb3f215ea1f539c2c14e2fa123bb/zones/zone.go#L50-L61)*.*

‍

> “决定使用Stainless让我们的注意力从构建生成引擎转移到构建高质量的架构描述服务上。
> 
> 短短几个月的时间里，我们已经从不一致的、手动维护的SDK转变为在三种语言SDK中自动发布超过1,000个端点。”
> 
> *Jacob Bednarz，Cloudflare API平台技术负责人 (*[*查看博客文章*](https://blog.cloudflare.com/lessons-from-building-an-automated-sdk-pipeline)*)*

‍

# **背景故事：在Stripe扩展SDK**

自[stripe.com第一天在互联网上存在](https://web.archive.org/web/20111213031731/https://stripe.com/)起，SDKs一直是开发者的重要组成部分。今天，超过90%的Stripe开发者使用SDK来发起超过90%的Stripe API请求。作为API的前门，SDK是开发者对Stripe的认知方式——对大多数人来说，重要的是`stripe.charges.create()`，而不是`POST /v1/charges`。

对我个人来说，直到我2017年加入Stripe，我并没有意识到这一点——但那时，Stripe API的增长范围已经超出了我们可持续手动构建SDK的能力。每当我们发布新端点时，在7种不同的编程语言中手动进行更改是繁琐且容易出错的。

不仅如此，TypeScript 已经成为了主流。现在，开发人员期望在他们的文本编辑器中直接获得自动补全和悬停式文档。构建支持各种语言全面静态类型的 SDK，如果没有代码生成，是完全不可想象的。

团队在决定没有一个能够达到 Stripe 质量标准之前，花费了一年多时间探索现有的开源代码生成工具。大多数代码生成工具要么只能用于一种语言，要么只是使用字符串模板，这会让你不断地处理问题，比如 Go 中的尾逗号和 Python 中的无效缩进。我们需要构建自己的代码生成工具，使我们能够“学习一次，到处编写”，并在所有语言中轻松生成干净、正确的代码。

在一个周末里，我突然将 JSX 与 prettier 的内部结构混合起来，以便让产品开发者能够快速生成高质量的代码模板，从而使其开箱即用地格式化。在接下来的几个月里，数十名工程师帮助将 SDK 转换为代码生成，每一字节都与我们精心制作的代码完美匹配。

那一年晚些时候，我与一位同事合作，为 [Stripe API](https://twitter.com/stripe/status/1222944951853432832) 制作了第一个官方的 TypeScript 定义。

‍

# **为所有人扩展 SDK**

离开 Stripe 后，工程师们不断询问我如何为他们的 API 构建出色的 SDK。我没有一个很好的答案——大多数公司没有几年的工程师时间来构建涵盖多种流行语言的高质量代码生成器套件。

2022 年初，我开始启动一家公司，而 Lithic 成为了我们的第一个客户，从第一天起就使不锈钢拉面有了盈利。他们的产品负责人此前曾在 Plaid 担任 SDK 的产品经理，她亲眼见证了 SDK 的价值以及通过 openapi-generator 编写出色 SDK 的难度。

尽管被一家小型自助启动公司所吸引，但我觉得将其影响力限制在我个人能处理的客户数量上很不好。更重要的是，我也被问及如何保持 OpenAPI 规范的最新和有效性，如何演变 API 版本，如何设计 RESTful 分页，如何设置 API 密钥，以及 Stripe 旧团队已解决的其他无数问题。

最终清晰地表明，世界需要一个全面的开发者平台——从文档到请求日志再到速率限制，可以让 REST 实现其潜力。

不久，Sequoia 成为了我们的第一个投资者，随后是像 Cristina Cordova、Guillermo Rauch、Calvin French-Owen 等伟大的天使投资人，以及其他数十位投资者。

显然，推动整个 REST 生态系统并帮助大量人们发布更好的 API 有着巨大的机会。

‍

# **推动 API 生态系统的进步**

API 是互联网的树突。几乎所有的互联网软件都通过 API 连接，它们构成了绝大部分的互联网流量。

今天，API 生态系统深度分裂：

1.  GraphQL。非常适合前端，但不适合服务器间交互，对公共 API 也没什么好处。

1.  gRPC。非常适合微服务，但对前端不起作用，也不受欢迎的公共 API。

1.  REST。适用于前端、微服务和公共 API。它简单、灵活，符合 web 标准，但也混乱且难以正确实现。

工程组织应能够在所有地方使用一种 API 技术——他们的前端、微服务和公共 API，并且在所有地方都有良好的体验。

在 Stainless，我们坚信“正确实现的 REST”可以实现这一愿景，而不是试图将 GraphQL 或 gRPC 强行适应它们不设计用的方形孔，或者发明一些新的第 15 个标准。

我们希望构建出色的开源标准和工具，将 GraphQL（类型、字段选择/展开、标准）和 gRPC（类型、速度、版本控制）的好处带给 REST。

当 Stainless REST 实现时，你将能够开始使用我们的 API 层构建全栈应用程序，至少会有与使用 GraphQL 相同水平的前端体验。当你添加一个 Go 微服务时，你将能够与类型化客户端、高效数据包和低延迟进行互联。然后，独特的是，当你的最大客户要求你提供外部 API 时，你只需说“是”，而不是重新编写整个东西，只需将 `internal(true)` 更改为 `internal(false)`。

今天，我们的 SDK 公告解决了 REST 的最突出问题：类型安全性。

我们的下一个项目是构建一个开发框架，使用户能够从任何 TypeScript 后端提供高质量的、类型安全的 REST API。通过即将推出的 Stainless API 框架，您可以在声明式 TypeScript 代码中定义 API 的形状和行为，并获得 OpenAPI 规范、文档和类型化的前端客户端，无需构建步骤。

我们正在围绕支持丰富分页、一致错误处理、字段包含和选择以及前端规范化缓存的 REST API 设计约定构建框架。这些约定受 Stripe 的最佳实践影响，可以帮助您的团队在不需数小时讨论的情况下实现一致且高质量的结果。

# **构建 Stainless**

> “这一切的精华在于，你可以定义你的 API 一次，然后免费获得每种语言的客户端库，无论你是否精通这些语言。
> 
> 很少有初创公司会有懂 Python、Node、Ruby、Rust、Go、Java 等等的人。但现在他们可以一次市场到所有这些开发者。”
> 
> *Calvin French-Owen，Segment 的联合创始人*

制作一个好的 SDK 比许多开发者想象的更复杂，特别是依赖代码生成时。细节很重要，不仅仅是漂亮的代码，而是在 REST API 的特征和语言习惯之间进行正确选择和平衡一些具有挑战性的折衷。

这里有一些通用的例子：

+   如何在 Java 中处理响应枚举？明显的方法在未来添加新变体时可能会导致崩溃。

+   如果你的 API 引入了联合类型，那么在 Go 中如何表达呢？考虑到语言没有标准的方法来表示联合类型。

+   如果服务器返回了意外数据——无论是由于测试功能、边缘情况还是 bug——你如何向用户公开这些数据？客户端库应该将其视为错误吗？是否有一种惯用的方法可以跨所有编程语言实现这一点？找到一个好的解决方案需要仔细权衡冲突的类型安全性和运行时安全性考虑。

+   应该自动重试 429 或 503 错误吗？多快重试？如果 API 正在经历生产中断，该怎么办？

+   在 Java 客户端库中，应该如何称呼 `/v1/invoices/{id}/void` 方法？（提示：不能是 `void`）。

注意，最后一个问题不能简单地由机器决定——它需要了解 API 其余部分的背景，并且必须由人来决定（即使是语言模型可以提供一个初步猜测）。

`void` 端点示例显然是一个边缘情况，但是每个方法和类型的命名问题都像它们一样棘手。如果 SDK 生成器直接从 OpenAPI 规范（特别是从其他来源生成的规范）中推断所有名称，用户可能会面对像 `AccountWrapperConfigurationUnionMember4` 这样的荒谬类型，这会引发对你公司整体工程质量的质疑。

如果在不解决这些问题的情况下发布 SDK，你可能会陷入设计错误中，而后果可能是你无法在后期进行不具有高度破坏性的向后兼容性破坏。

我们上面分享了 `void` 的示例，因为这样更容易理解，但在非平凡的 API 中总会出现更加微妙和深奥的潜在陷阱。我们可以构建工具来识别这些问题，但如何解决这些问题通常超出了自动化的范围——即使是使用 AI。你需要一个具有相关背景和良好判断力的人来评估选项并做出明智的决定。

从经验中，我们知道，深思熟虑的 SDK 开发比看上去更加困难——在典型的中型 API 中审查每个类型名称需要扫描数万行代码。

为了让每个开发者都能像我们对待企业客户一样慎重对待，我们创建了一个 SDK Studio，突出显示潜在问题，并使快速浏览在发布 v1 之前你可能想要审核的所有内容变得轻而易举：

Stainless SDK Studio

要开始使用 Stainless SDK 生成器，你只需要一个 OpenAPI 规范。

几分钟内，您将获得可以发布到包管理器的 alpha SDK，并经过一些打磨后，您将会为发布为 v1.0.0 而感到自豪的东西。

要开始，请查看[我们的文档](https://app.stainlessapi.com/docs/)或[连接您的 GitHub 帐户](https://app.stainlessapi.com/login)。
