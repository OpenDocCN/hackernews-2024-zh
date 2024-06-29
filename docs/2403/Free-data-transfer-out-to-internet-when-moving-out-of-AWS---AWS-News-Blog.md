<!--yml

category: 未分类

date: 2024-05-27 14:40:41

-->

# 免费数据传输出网时从AWS迁移出 | AWS 新闻博客

> 来源：[https://aws.amazon.com/blogs/aws/free-data-transfer-out-to-internet-when-moving-out-of-aws/](https://aws.amazon.com/blogs/aws/free-data-transfer-out-to-internet-when-moving-out-of-aws/)

您告诉我们采用亚马逊网络服务（AWS）的主要原因之一是我们提供的广泛服务选择，使您能够创新、构建、部署和监控您的工作负载。AWS已经不断扩展其服务以支持几乎任何云工作负载。现在，它提供超过200种功能齐全的服务，用于计算、存储、数据库、网络、分析、机器学习（ML）和人工智能（AI），以及更多。例如，[Amazon Elastic Compute Cloud（Amazon EC2）](https://aws.amazon.com/ec2/)提供超过750个通用实例——比任何其他主要云提供商都多——您可以从[多个关系型、分析、键值、文档或图数据库](https://aws.amazon.com/getting-started/decision-guides/databases-on-aws-how-to-choose/)中进行选择。

我们认为这个选择必须包括将数据迁移到其他云提供商或本地设备的选择。因此，从今天开始，当您希望从AWS迁移到其他云提供商或本地设备时，我们将免除向互联网的数据传输（DTO）费用。

我们超过90%的客户已经不需要支付从AWS传输数据的费用，因为[我们每月从AWS区域到互联网提供100GB免费](https://aws.amazon.com/blogs/aws/aws-free-tier-data-transfer-expansion-100-gb-from-regions-and-1-tb-from-amazon-cloudfront-per-month/)。这包括来自Amazon EC2、[Amazon Simple Storage Service（Amazon S3）](https://aws.amazon.com/s3/)、[Application Load Balancer](https://docs.aws.amazon.com/elasticloadbalancing/latest/application/introduction.html)等的流量。此外，我们还每月免费提供从[Amazon CloudFront](https://aws.amazon.com/cloudfront/)传输出的1TB数据。

如果在过渡期间需要超过100GB的每月数据传输出网，您可以联系[AWS支持](https://aws.amazon.com/contact-us/)，请求额外数据的免费DTO费率。这是必须通过支持来完成的，因为您每天进行数亿次数据传输，通常我们无法确定传输到互联网的数据是您业务的正常部分，还是切换到其他云提供商或本地设备的一次性转移的一部分。

我们将在AWS账户级别审查请求。一旦批准，我们将为正在迁移的数据提供信用。我们不要求您关闭账户或以任何方式改变与AWS的关系。您随时可以回来。当然，如果同一AWS账户多次申请免费DTO，我们将进行额外审查。

我们信仰客户的选择权，包括选择将您的数据移出 AWS。对于向互联网传输数据的费用豁免，也符合[欧洲数据法](https://digital-strategy.ec.europa.eu/en/news/european-data-act-enters-force-putting-place-new-rules-fair-and-innovative-data-economy)的方向，并向全球所有 AWS 客户及所有 AWS 区域开放。

自由选择不仅限于数据传输速率。AWS 还支持[公平软件许可原则](https://www.fairsoftware.cloud/principles/)，这使得您可以轻松地与其他IT服务提供商共同使用软件。[您可以阅读这篇博客文章获取更多细节。](https://aws.amazon.com/blogs/networking-and-content-delivery/promoting-customer-choice-aws-takes-another-step-to-lower-costs-for-customers-changing-it-providers/)

您可以查看[常见问题解答获取更多信息](https://aws.amazon.com/ec2/faqs/#Data_transfer_fees_when_moving_all_data_off_AWS)，或者[联系 AWS 客户支持](https://aws.amazon.com/contact-us/)申请在切换时的 DTO 抵扣。

但我真诚地希望您不要这样做。

[-- seb](https://twitter.com/sebsto)
