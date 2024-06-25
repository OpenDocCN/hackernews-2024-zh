<!--yml

category: 未分类

date: 2024-05-27 15:23:01

-->

# IPv4 surcharge - Your AWS bill is going up this February | Border0

> 来源：[`www.border0.com/blogs/ipv4-surcharge---your-aws-bill-is-going-up-this-february`](https://www.border0.com/blogs/ipv4-surcharge---your-aws-bill-is-going-up-this-february)

从明天开始，您的 AWS 账单将会上涨！自 2024 年 2 月 1 日起，每个公共 IPv4 地址每小时将收取 0.005 美元的费用，无论是否连接到服务。这总共是每年 43.80 美元，一个相当可观的数字！其原因在 [AWS 公告](https://aws.amazon.com/blogs/aws/new-aws-public-ipv4-address-charge-public-ip-insights/) 中已经概述：

‍

> 如您所知，IPv4 地址是一种日益稀缺的资源，过去 5 年里，获得单个公共 IPv4 地址的成本已经上涨了 300% 以上。这种变化反映了我们自身的成本，也旨在鼓励您更节俭地使用公共 IPv4 地址。

‍

在本博客中，我将介绍如何通过使用 Border0 来消除不必要的公共 IPv4 地址来节省 AWS 账单。但在我们继续之前，让我们看看亚马逊有多少个 IPv4 地址，价值多少，以及 AWS 将通过对您的月度账单收取新费用赚取多少钱。

‍

## AWS 有多少个 IPv4 地址？

‍

经营亚马逊基础设施并跟上 AWS 的惊人增长需要大量的 IP 地址。因此，不足为奇的是，多年来，亚马逊已经花费了大量资金来收购大量的 IPv4 地址。所有这些都是为了我们能够继续启动 ec2 实例、负载均衡器和 NAT 网关而不用担心 IPv4 地址。

‍

要准确确定亚马逊有多少个 IPv4 地址，我们可以查看各种公开可用的数据集。我使用的数据是 [AWS IP json](https://docs.aws.amazon.com/vpc/latest/userguide/aws-ip-ranges.html) 以及各种 whois（ARIN、RIPE 等）数据条目。

‍

*对所有这些数据进行处理，我们可以确定亚马逊至少拥有 1,319,327,520 个 IPv4 地址。

让我们将其四舍五入，说是* ***1.32 亿个 IPv4 地址****！这相当于近八个 /8 网段 😮* 想知道数据和包含的 IPv4 地址吗？请参阅[此链接获取原始数据。](https://gist.github.com/atoonk/0ee3f5bebcea874f6032215f16c3c30a)

‍

## 亚马逊 IPv4 地产价值多少？

‍

IPv4 地址就像数字房地产一样。这些 32 位整数具有实际货币价值，可以买卖。事实上，IPv4 地址的价格在过去十年里显著上涨，如果您早早介入，它们将是一项很好的投资！

[`auctions.ipv4.global/prior-sales`](https://auctions.ipv4.global/prior-sales)

‍

那么，下一个合乎逻辑的问题是，亚马逊的 IPv4 地产价值是多少？根据 ipv4.global 的数据，目前一个 IPv4 地址的平均价格约为 35 美元。有了这些数据，我们可以做一些简单的估算：

‍

‍

*因此，亚马逊的 IPv4 地产今天的大约估值是：

‍****46 亿美元！*** 也不算太糟糕！

‍

## AWS 将通过新的 IPv4 收费赚取多少钱？

‍

谈到美元，让我们看看我们是否可以对 AWS 从新的 IPv4 费用中赚取多少有一个合理的猜测。为此，我们需要每个 IP 的价格以及 AWS 客户使用的 IPv4 地址数量。

‍

我们知道第一个变量，每个 IP 每小时 0.005 美元，或每年每个 IPv4 地址 43.80 美元。第二个变量，AWS 客户使用的 IPv4 地址数量，很难确定。但是，为了好玩，我们可以做一些有根据的猜测！

‍

正如前面所述，这里的一个重要变量是 AWS 客户在任何给定时间使用多少个 IP 地址。让我们探讨几种情况，从一个非常保守的估计开始，比如说，假设在一年内使用了 [AWS JSON](https://docs.aws.amazon.com/vpc/latest/userguide/aws-ip-ranges.html) 中发布的 IPv4 地址的 10%（7900 万个 IPv4 地址）。那就是 790 万个 IPv4 地址 x 43.80 美元，几乎是 3.46 亿美元每年。在 25% 使用率下，几乎是每年 8.65 亿美元。而在 30% 使用率下，那就是十亿美元了！

‍

这给了我们一个相当好的规模指标。另一个方法是尝试进行测量。AWS 方便地发布了所有地址，所以我们可以向它们发送 ICMP 回显请求（ping），并测量多少个地址发送了回显响应。

‍

听起来像一个有趣的项目！所以我写了一个快速的 [程序](https://github.com/atoonk/ping-aws-ips)，它下载了所有 AWS IP 地址的 JSON，并过滤出了类别 "AMAZON"、"EC2" 和 "GLOBAL ACCELERATOR"。我们将假设这些都是所有客户使用（收费）的 IP 地址。也就是说，我们不会 ping 诸如 Route53 健康检查或 Cloudfront 之类的服务，因为这些不会在您的账单上显示为 IPv4 费用。

‍

该程序向所有 IP 地址发送单个 ICMP 包，并收集所有回复。通过这个，我们有了一些实际的测量数据，并观察到我们从大约 600 万个 IPv4 地址收到了回复。

六百万个地址 x 43.80 美元是 2.63 亿美元的年收入！

‍

这是另一个很好的数据点。但是，请记住，许多 ec2 实例和其他服务将具有严格的安全组，并且默认情况下不会响应 ping 包。因此，可以说六百万个活动 IP 是绝对的最低限度。由于各种默认安全组阻止 ICMP，实际的活动 IPv4 地址数量可能很容易是这个数字的两倍。

‍

根据这些数据，我认为可以说 ***AWS 有可能每年通过这项新的 IPv4 费用赚取 4 亿到 10 亿美元！*** 这对 AWS 来说是一个不错的增长，尤其是考虑到直到今天这项服务都是免费提供的。

‍

## 使用 Border0 降低您的 AWS 账单

您的 AWS 服务，比如 ec2 实例，可能有多个公共 IPv4 地址，原因有很多。一个常见的原因是为了管理服务器。例如，使用 SSH 或 RDP。或者访问在您的机器上运行的应用程序，比如数据库或 HTTP 应用程序。

‍

你的一些应用可能只应该让授权用户访问，并且最好不直接连接到互联网。例如，[最近的 Border0 研究](https://www.border0.com/blogs/help-my-postgres-database-was-compromised) 表明僵尸网络正在积极攻击公开可访问的 Mysql 和 Postgres 服务器！你不希望这些服务器在互联网上没有受保护，让所有人都能随意访问。

‍

在可能的情况下，我们建议在私有子网中运行您的 AWS 基础设施，只使用 NAT 网关进行出站互联网连接。这样，它们就不会暴露在互联网上，极大地降低了被攻击的风险。作为额外的好处，你还能省下 AWS 的 IPv4 费用！（注意：这个费用仅适用于公共 IP 地址）。

[`www.youtube.com/embed/NHR3Do4WhVU`](https://www.youtube.com/embed/NHR3Do4WhVU)

视频

在你的私有网络中部署一个 Border0 连接器，你和你的团队就可以只使用现有的单点登录凭证来访问所有服务，而不需要 VPN。

部署 Border0 比你想的要 [简单得多](https://youtu.be/e0KeS5x-GZ0)！想试试吗？看看我们的 [terraform 示例](https://www.border0.com/blogs/border0-terraform-provider) 或者这篇关于 Border0 for AWS 的 [博客](https://www.border0.com/blogs/elevating-security-and-simplifying-aws-access-with-border0) 吧。

‍

Border0 提供 [慷慨的免费套餐](https://portal.border0.com/register)，入门简单！

使用 Border0，访问更容易、更安全；你的工程师团队和安全团队会喜欢它。而且，由于节省了公共 IP 地址，你的 FinOps 人员也会很高兴！

‍
