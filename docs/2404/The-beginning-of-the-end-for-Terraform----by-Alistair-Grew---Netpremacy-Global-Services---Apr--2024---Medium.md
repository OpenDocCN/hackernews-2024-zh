<!--yml
category: 未分类
date: 2024-05-27 13:34:49
-->

# The beginning of the end for Terraform? | by Alistair Grew | Netpremacy Global Services | Apr, 2024 | Medium

> 来源：[https://medium.com/netpremacy-global-services/the-beginning-of-the-end-for-terraform-cfffcd2c5420](https://medium.com/netpremacy-global-services/the-beginning-of-the-end-for-terraform-cfffcd2c5420)

# The beginning of the end for Terraform?

Entering the graveyard of good software that is IBM…

Source:imgflip.com

As I write this on the 25th of April, I am still reeling from the announcement of [IBM’s acquisition of Hashicorp](https://www.hashicorp.com/blog/hashicorp-joins-ibm). When I first heard the rumours yesterday, I was concerned about the future of possibly my favourite Infrastructure-as-code (IaC) tool. It has long been obvious that Hashicorp has been struggling to make money, making a [$274 million loss in 2023](https://ir.hashicorp.com/static-files/1d62b124-e329-4694-844d-aa6e028f2f92). This undoubtedly led to the highly controversial [switch to the BSL license in August 2023](https://www.hashicorp.com/blog/hashicorp-adopts-business-source-license) and the community fork of [OpenTofu](https://opentofu.org/). Hashicorp is hardly alone in its struggles, with [Redis adopting a similar approach](https://redis.io/blog/redis-adopts-dual-source-available-licensing/), resulting in the [Valkey fork](https://valkey.io/) or [Elasticsearch with its license changes](https://www.elastic.co/blog/elastic-license-update).

# The challenge of open-source monetisation

Fundamentally, many of these companies have struggled to sufficiently monetise the open-source tools they have developed, and this is where I want to bust a myth some people seem to have:

> There is no such thing as ‘free’ software

Sure, there is plenty of software that is free at the point of use and can be modified under permissive licenses and the like, but fundamentally, developers are spending time writing code, and time typically requires some payment.

Source: [https://accidentalastro.com/2022/12/will-code-for-food/](https://accidentalastro.com/2022/12/will-code-for-food/)

I appreciate there are several hobby projects that people may contribute to in their ***spare*** time, but large open-source projects are typically supported by big tech firms contributing engineers or money to enable developers to work part or full-time on them; for example, look at the companies who pay to be members of the [Linux foundation](https://www.linuxfoundation.org/about/members). As another example, Google is still the largest single contributor to the Kubernetes project. Google can do this because it makes money running services that rely on it, and sells a managed offering in [Google Kubernetes Engine](https://cloud.google.com/kubernetes-engine?hl=en).

Source: [https://www.cncf.io/reports/kubernetes-project-journey-report/](https://www.cncf.io/reports/kubernetes-project-journey-report/)

Hashicorp struggles with this, people have long used their free tool to orchestrate public and private cloud environments but had little reason to pay for the Terraform Cloud product even if it had some advantages.

# The problem with IBM buying Hashicorp

Source: [https://cdn.thenewstack.io/media/2024/04/967925a7-hashicorp-ibm.jpg](https://cdn.thenewstack.io/media/2024/04/967925a7-hashicorp-ibm.jpg)

So Hashicorp was a financial disaster even though they had good products, I have personally worked with [Terraform](https://www.hashicorp.com/products/terraform) & [Packer](https://www.hashicorp.com/products/packer) and seen plenty of [Vault](https://www.hashicorp.com/products/vault) implementations. A lot of companies liked Hashicorp tooling because it was inherently cloud-agnostic; for multi-cloud organisations, having a common configuration language in Terraform or a secret management solution in Vault was preferable to learning each in AWS, Azure, Google Cloud, or the multitude of other providers.

This leads me to my first concern, IBM has a conflict of interest. [IBM has a cloud](https://www.ibm.com/cloud) offering, [admittedly with a 1.8% market share](https://kinsta.com/blog/cloud-market-share/). Why would they want to keep developing tools that frankly benefit their competitors more than them?

My second concern, perhaps my biggest, is that IBM has a graveyard of failed acquisitions of good software companies. In recent history, their acquisition of Red Hat resulted in changes to CentOS that destroyed the market share of this once-popular distribution, anecdotally often in favour of Ubuntu LTS. This is far from the only example with [Lotus Software](https://en.wikipedia.org/wiki/Lotus_Software) being another from the 2000s. I simply have no trust in IBM to look after Hashicorp as a company.

# So, where do we go from here?

Source: [https://media.licdn.com/dms/image/C4E12AQG7bJnXPlbrsw/article-cover_image-shrink_600_2000/0/1520066260072?e=2147483647&v=beta&t=LZ84Bom6KUx4nsjuPkvZ_Pzb_W9znd8nih_k_s-XjfI](https://media.licdn.com/dms/image/C4E12AQG7bJnXPlbrsw/article-cover_image-shrink_600_2000/0/1520066260072?e=2147483647&v=beta&t=LZ84Bom6KUx4nsjuPkvZ_Pzb_W9znd8nih_k_s-XjfI)

So, the big question, much like the recent post-VMware price rises, I think there is a fork in the road. Either we continue to stick with Terraform, as IBM no doubt attempts some renewed monetisation effort to make good on their investment, or we change our IaC tool. With the latter, what are our alternatives?

Well, let’s start with the obvious: [OpenTofu](https://opentofu.org/). This is perhaps the easiest short-term ‘fix’ for those not running cutting-edge Terraform, where they can use a compatible binary based on an earlier version.

[Pulumi](https://www.pulumi.com/) is another potential option, though I would have financial sustainability concerns for their business model which is similar to Hashicorp’s old model.

[Crossplane](http://crossplane.io) is another option that generates quite a lot of noise with a Kubernetes-centric view of the world. I [previously blogged a little about](/cts-technologies/are-terraforms-days-numbered-a9a15ec0435a) this approach in the context of Google’s Config Connector, showing some merit. Crossplane uses a similar ‘provider-centric’ approach like Terraform allowing easier extensibility to different platforms. Whilst the project is CNCF-backed (and going through adoption at the time of writing) the main contributor appears to be [Upbound](https://www.upbound.io/) whose financial sustainability I am equally unsure of.

Another option is to ‘go native,’ with each of the big three hyperscalers offering its dedicated options: Cloud Formation with AWS, Bicep with Azure, and Config Controller in Google Cloud. The main downside I see with this approach is fragmentation. Part of the appeal of Terraform was its consistency in configuration language, with providers abstracting the vendor-specific API calls.

There are no doubt many other tools I have failed to mention, and please comment on any additional suggestions you have.

# What do I think?

Source: [https://images.newscientist.com/wp-content/uploads/2018/05/15173534/gettyimages-611975304.jpg](https://images.newscientist.com/wp-content/uploads/2018/05/15173534/gettyimages-611975304.jpg)

I believe an ‘Open Terraform’ supported in development by those who profit from it is the answer. This is sort of what’s happened with OpenTofu, which has backers such as [Gruntwork](https://gruntwork.io/). I would love to see an expansion of this support base to include all the major hyperscalers, who, in practice, are the ones making the most money from Terraform. Many of these hyperscalers are already deeply involved in developing providers that interface Terraform with their platform. Expanding this toward the tool itself doesn’t seem like a big leap; if they decide to do this, though, is anyone’s guess, especially with the news so fresh, we will have to wait and see…