<!--yml

category: 未分类

date: 2024-05-27 13:34:49

-->

# Terraform的终结之始？ | 作者：Alistair Grew | Netpremacy全球服务 | 2024年4月 | Medium

> 来源：[https://medium.com/netpremacy-global-services/the-beginning-of-the-end-for-terraform-cfffcd2c5420](https://medium.com/netpremacy-global-services/the-beginning-of-the-end-for-terraform-cfffcd2c5420)

# Terraform的终结之始？

进入IBM软件墓地的好软件…

来源：imgflip.com

当我在4月25日写下这篇文章时，我仍然被[IBM收购Hashicorp的消息](https://www.hashicorp.com/blog/hashicorp-joins-ibm)震惊。昨天我第一次听到这些传言时，我对可能是我最喜欢的基础设施即代码（IaC）工具的未来感到担忧。长期以来，显而易见的是Hashicorp一直在努力赚钱，导致了在2023年的[2.74亿美元亏损](https://ir.hashicorp.com/static-files/1d62b124-e329-4694-844d-aa6e028f2f92)。这无疑导致了2023年8月高度争议的[切换到BSL许可证](https://www.hashicorp.com/blog/hashicorp-adopts-business-source-license)以及[OpenTofu的社区分叉](https://opentofu.org/)。Hashicorp在其困境中并不孤单，例如[Redis采用了类似的方法](https://redis.io/blog/redis-adopts-dual-source-available-licensing/)，导致了[Valkey的分叉](https://valkey.io/)或[Elasticsearch的许可证变更](https://www.elastic.co/blog/elastic-license-update)。

# 开源商业化的挑战

从根本上说，这些公司中的许多都在努力将他们开发的开源工具充分商业化，这正是我想要打破一些人似乎存在的误解的地方：

> 没有所谓的‘免费’软件

当然，有很多软件在使用时是免费的，并且可以根据宽松许可证等进行修改，但从根本上讲，开发者正在花时间编写代码，而时间通常需要一些报酬。

来源：[https://accidentalastro.com/2022/12/will-code-for-food/](https://accidentalastro.com/2022/12/will-code-for-food/)

我欣赏人们可能会在他们的***空闲***时间为几个爱好项目做贡献，但是大型开源项目通常由大型科技公司支持，这些公司为开发人员提供工程师或资金，使他们能够全职或兼职地从事项目工作；例如，看看那些支付成为[Linux基金会](https://www.linuxfoundation.org/about/members)成员的公司。再举一个例子，Google仍然是Kubernetes项目的最大单一贡献者。Google之所以能够做到这一点，是因为它通过运行依赖于该项目的服务赚钱，并在[Google Kubernetes Engine](https://cloud.google.com/kubernetes-engine?hl=en)中销售托管服务。

来源：[https://www.cncf.io/reports/kubernetes-project-journey-report/](https://www.cncf.io/reports/kubernetes-project-journey-report/)

Hashicorp 与此有所斗争，长期以来，人们一直使用他们的免费工具来编排公共和私有云环境，但即使 Terraform Cloud 产品有一些优势，他们也没有太大理由付费使用它。

# IBM 收购 Hashicorp 的问题

来源：[https://cdn.thenewstack.io/media/2024/04/967925a7-hashicorp-ibm.jpg](https://cdn.thenewstack.io/media/2024/04/967925a7-hashicorp-ibm.jpg)

虽然 [Hashicorp](https://www.hashicorp.com/products/terraform) 产品不错，但是它们在财务上却是一场灾难，我个人曾与 [Terraform](https://www.hashicorp.com/products/terraform) 和 [Packer](https://www.hashicorp.com/products/packer) 合作过，并见过很多 [Vault](https://www.hashicorp.com/products/vault) 的实施。许多公司喜欢 Hashicorp 的工具集，因为它天然支持多云平台；对于多云组织来说，在 Terraform 中使用共同的配置语言或在 Vault 中使用秘密管理解决方案，比起在 AWS、Azure、Google Cloud 或其他多个提供商中学习更为可取。

这让我关注到我的第一个问题是，IBM 存在利益冲突。 [IBM 有一个云](https://www.ibm.com/cloud) 提供，尽管市场份额仅为 1.8%。他们为什么要继续开发工具，明显这些工具更有利于他们的竞争对手而非他们自己？

我的第二个担忧，也许是最大的，是 IBM 有一堆失败的软件公司收购案例。最近，他们收购 Red Hat 导致 CentOS 的变化，摧毁了这个曾经流行的发行版的市场份额，传闻中很多人转而使用 Ubuntu LTS。这远非唯一的例子，2000 年代的 [Lotus Software](https://en.wikipedia.org/wiki/Lotus_Software) 也是如此。我对 IBM 看管 Hashicorp 公司毫无信任可言。

# 那么，接下来我们该怎么办？

来源：[https://media.licdn.com/dms/image/C4E12AQG7bJnXPlbrsw/article-cover_image-shrink_600_2000/0/1520066260072?e=2147483647&v=beta&t=LZ84Bom6KUx4nsjuPkvZ_Pzb_W9znd8nih_k_s-XjfI](https://media.licdn.com/dms/image/C4E12AQG7bJnXPlbrsw/article-cover_image-shrink_600_2000/0/1520066260072?e=2147483647&v=beta&t=LZ84Bom6KUx4nsjuPkvZ_Pzb_W9znd8nih_k_s-XjfI)

所以，关键问题是，就像最近 VMware 提高价格一样，我认为我们现在面临抉择。要么我们继续坚持使用 Terraform，因为 IBM 毫无疑问会试图通过重新商业化来回报他们的投资，要么我们更换我们的 IaC 工具。对于后者，我们有哪些选择？

那么，让我们从明显的地方开始：[OpenTofu](https://opentofu.org/)。对于那些没有运行最新版 Terraform 的用户来说，这可能是最简单的短期“修复”，他们可以使用基于早期版本的兼容二进制文件。

[Pulumi](https://www.pulumi.com/) 是另一个潜在的选择，尽管我对他们的商业模式有财务可持续性的担忧，这与 Hashicorp 旧模式类似。

[Crossplane](http://crossplane.io) 是另一个选择，它以 Kubernetes 为中心的视角引起了相当大的关注。我之前在 Google 的 Config Connector 的背景下有过一些关于这种方法的博客文章，显示了一些优点。Crossplane 使用类似 Terraform 的“提供程序中心”方法，允许更容易地扩展到不同的平台。虽然该项目得到 CNCF 的支持（并且正在撰写本文时进行采用），但主要贡献者似乎是 [Upbound](https://www.upbound.io/)，其财务可持续性同样让我担忧。

另一个选择是“走本地化”路线，三大超级扩展器分别提供其专用选项：AWS 的 Cloud Formation，Azure 的 Bicep 和 Google Cloud 的 Config Controller。我认为这种方法的主要缺点是碎片化。Terraform 的吸引力之一在于其一致的配置语言，提供商抽象了特定供应商的 API 调用。

毫无疑问，我没有提到的许多其他工具，请在您有任何额外建议时评论。

# 我的想法是什么？

来源：[https://images.newscientist.com/wp-content/uploads/2018/05/15173534/gettyimages-611975304.jpg](https://images.newscientist.com/wp-content/uploads/2018/05/15173534/gettyimages-611975304.jpg)

我相信由从中获利的人们支持开放式的 Terraform 将是解决方案。这有点类似于 OpenTofu，其支持者包括 [Gruntwork](https://gruntwork.io/)。我希望看到这种支持基础扩展到包括所有主要的超级扩展器，他们实际上是从 Terraform 中获得最多收益的人。许多超级扩展器已经深度参与开发提供程序，这些提供程序使 Terraform 能够与其平台进行接口。将这种支持扩展到工具本身似乎并不是一个很大的飞跃；不过，他们是否决定这样做，尤其是在新闻如此新鲜的情况下，我们还需要观察……
