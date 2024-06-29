<!--yml

category: 未分类

date: 2024-05-27 12:54:23

-->

# 改进的基础设施定价 – Vercel

> 来源：[https://vercel.com/blog/improved-infrastructure-pricing](https://vercel.com/blog/improved-infrastructure-pricing)

基于您的反馈，我们正在更新如何测量和计费我们基础设施产品的使用情况。

+   我们正在**降低**Vercel基础设施的价格，例如**带宽和功能**

+   对于大多数客户，每月账单将**保持不变或减少**

+   我们引入了更简单优化的**新原语**

+   您现在可以精确按使用量付费，以更细粒度的增量计费

+   我们的**Hobby层继续免费**

[这些变化](https://vercel.com/pricing/coming-soon)将显示在您2024年6月25日至7月24日期间的第一张账单上。接下来会给账户所有者发送关于您账户的下一步操作的电子邮件。

不再使用两个大的组合指标（带宽和功能），现在您可以分别优化每个指标的粒度定价。

*¹ 对于大部分流量的定价，费用会因[*地区*](https://vercel.com/docs/pricing#regional-pricing)而异

我们更新的定价更符合行业标准，并更好地反映了Vercel为您所做的工作，而不是将带宽和功能指标捆绑在一起。

此外，在过去一年中，我们还为平台添加了功能，帮助自动防止支出失控，包括[硬支出限制](https://vercel.com/changelog/improved-hard-caps-for-spend-management)，[递归保护](https://vercel.com/changelog/automatic-recursion-protection-for-vercel-serverless-functions)，[改进的函数默认设置](https://vercel.com/changelog/serverless-functions-can-now-run-up-to-5-minutes)，[攻击挑战模式](https://vercel.com/changelog/prevent-malicious-traffic-with-attack-challenge-mode-for-vercel-firewall)，等等。

对于我们的企业客户，现有合同不受影响。如果您有任何疑问或想讨论这些变化，请联系您的账户团队。

可视化我们更精细的基础设施指标如何累积。

可视化我们更精细的基础设施指标如何累积。

对于大多数专业版客户，每月账单将**保持不变或降低**。

邮件将会发送给账户所有者，详细说明**Hobby层仍然免费**并且将以先前包含的带宽和功能的比例进行免费使用。

如果Hobby客户在接下来的6个月内超出这些新指标，他们的应用将不受影响，以便有时间进行调整。

这次基础设施定价更新不会改变我们的任何其他定价内容。

这些变化将在2024年5月至6月的您的计费周期内生效。

在此日期之前，Vercel仪表板将显示更新的使用和计费信息，以帮助您了解这些变化。此外，更新的文档将在线上，详细解释每个定价指标及其优化方法。

根据我们当前的定价，Pro 计划的开发者在首个 $20/月 计划中有一些包含的使用量，之后可以按需支付额外的使用量。例如，带宽。前1TB包含在每月 $20 中，之后是每100GB $40。

这不是理想的情况，有两个原因。首先，“带宽”价格中包含多项服务，包括快速数据传输、快速源传输和边缘请求。通过我们改进的基础设施定价，现在这些指标是单独计费的，让您能够单独优化每个指标。我们仍然在基础的 $20/月 中包含使用量。您的前 1TB 快速数据传输、100GB 快速源传输和1千万边缘请求已包含在内。当您超出这些值时，您可以分别累积每个指标的使用量。

第二，之前的使用量是打包为100GB增量。如果您在计划包含的量之后使用了1GB的按需使用量，您将会被收取 $40。通过我们改进的基础设施定价，您只需支付 $0.15 而不是 $15。

由于这一变化，许多客户将看到诸如带宽和功能等基础服务价格的降低。

我们更细化的基础设施定价将带宽和功能拆分为快速数据传输、快速源传输、边缘请求等指标。

您可以在[我们的文档](https://vercel.com/docs/pricing)中了解每个指标的更多信息。此外，要查看所有价格，您可以预览我们[更新的定价页面](https://vercel.com/pricing/coming-soon)。

为了支持开源生态系统，我们提供免费的 Vercel Pro 账户给各种前端框架、JavaScript 库、非营利组织和慈善事业。

这些赞助计划**不会有所更改**。

我们致力于为 Vercel 用户提供免费计划。提醒一下，免费计划不需要添加付款方式，并包含基础设施使用量，可用于构建您的前几个项目。

在使我们的基础设施指标更细化的同时，我们保留了以前免费使用层所包含的相当数量。例如，100GB 带宽现在是100GB 快速数据传输、10GB 快速源传输和100万边缘请求。

如果您超出包含的免费使用量，您的项目将会暂停。然后，您可以升级到付费计划以继续与 Vercel 一同成长。

缓存响应可以减少 Vercel 函数的使用量。

如果函数响应已经被[缓存](https://vercel.com/docs/edge-network/caching)，它将不会运行，也不会产生函数调用或任何 GB/小时的持续时间。向 Vercel 发送的任何请求都会产生边缘请求。对静态资产的请求，如果响应被缓存或未缓存，都会产生相同的快速数据传输使用量。

此外，Vercel 自动为所有响应添加 etag，以防止最终用户多次下载相同的资源。我们的边缘网络自动[压缩](https://vercel.com/docs/edge-network/compression)响应，以减少您的快速数据传输。

我们为客户提供工具，观察、控制和警告他们的基础设施支出，使用[Spend Management](https://vercel.com/blog/introducing-spend-management-realtime-usage-alerts-sms-notifications)。

您可以定义一个支出金额（例如 $40），并在达到该金额时收到电子邮件、网络和短信通知。当达到 100% 时，您可以选择自动暂停所有项目以进行硬限制。

过去一年来，我们根据您的反馈向平台添加了一些功能，帮助自动防止超支，包括[递归保护](https://vercel.com/changelog/automatic-recursion-protection-for-vercel-serverless-functions)，[改进的函数默认设置](https://vercel.com/changelog/serverless-functions-can-now-run-up-to-5-minutes)，[硬额度限制](https://vercel.com/changelog/improved-hard-caps-for-spend-management)，[攻击挑战模式](https://vercel.com/changelog/prevent-malicious-traffic-with-attack-challenge-mode-for-vercel-firewall)，等等。

我们为快速数据传输、快速源传输、边缘请求和其他指标引入了[区域定价](https://vercel.com/docs/pricing#regional-pricing)。由于基础设施底层结构的不同，这些资源的定价因用户访问位置而异。
