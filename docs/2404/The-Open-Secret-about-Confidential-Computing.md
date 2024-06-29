<!--yml

类别：未分类

日期：2024-05-27 13:03:21

-->

# 关于保密计算的开放性秘密

> 来源：[https://stiankri.substack.com/p/the-open-secret-about-confidential](https://stiankri.substack.com/p/the-open-secret-about-confidential)

[保密计算](https://zh.wikipedia.org/wiki/%E4%BF%9D%E5%AF%86%E8%AE%A1%E7%AE%97)是一个新兴领域，旨在保护正在运行的工作负载（“使用中的数据”）免受其环境的影响，从而减少可信计算基础（TCB）。对于虚拟机（VMs），这意味着更新了威胁模型，不再信任虚拟化管理程序。主要推动力来自公共云供应商，以便运行更敏感的工作负载。简而言之，CPU 可信，并创建一个干净的 VM/enclave，可以进行测量和证明。认证可以发送到可信环境，该环境可以用其交换所需的秘密来执行工作。

如果你阅读来自 [Azure](https://azure.microsoft.com/en-us/solutions/confidential-compute) 和 [GCP](https://cloud.google.com/security/products/confidential-computing) 的保密计算营销，很容易认为这是一个解决了的问题。如果你阅读《[Trust in Computer Systems and the Cloud](https://www.goodreads.com/book/show/58150353)》 - 这是该主题的优秀介绍 - 它承认这是一个需要做很多正确的事情的难题，并且有很多被视为超出范围的事情（第11章）。如果你阅读《[Security Engineering: A Guide to Building Dependable Distributed Systems](https://www.goodreads.com/book/show/3268675-security-engineering)》 - 总体上是一本强烈推荐的阅读 - 它更加直接；指出生态系统可能会被一个提取的认证密钥破坏，这已经被证明过了（第20.6节，第三版）。

Azure和GCP使用的三种主要技术是[AMD SEV-SNP](https://www.amd.com/en/developer/sev.html)（虚拟机）、[Intel SGX](https://www.intel.com/content/www/us/en/products/docs/accelerator-engines/software-guard-extensions.html)（飞地）和[Intel TDX](https://www.intel.com/content/www/us/en/developer/tools/trust-domain-extensions/overview.html)（虚拟机）。Intel TDX使用Intel SGX进行证明。[Intel SGX](https://www.usenix.org/system/files/conference/usenixsecurity18/sec18-van_bulck.pdf)和[AMD SEV](https://arxiv.org/pdf/1908.11680.pdf)的证明密钥已被提取。在Intel SGX的情况下，这被用来[破解UHD蓝光DRM](https://sgx.fail/files/sgx.fail.pdf)。Intel SGX中有很多侧信道，他们使用表格来追踪哪些CPU容易受到哪些攻击的影响。当Google审查[AMD SEV-SNP](https://googleprojectzero.blogspot.com/2022/05/release-of-technical-report-into-amd.html)和[Intel TDX](https://cloud.google.com/blog/products/identity-security/rsa-google-intel-confidential-computing-more-secure)时，他们在两者中发现了一些问题。本周，瑞士苏黎世联邦理工学院的研究人员发布了更多的问题，涉及这两种解决方案。

随着时间的推移，随着漏洞的被发现和修复，Intel和AMD的解决方案应该变得更难被攻击。然而，一些架构决策可能会继续产生漏洞；Intel和AMD的方法基于微码在与其他工作负载相同的核心上运行，而不是专用硬件。这使它们更容易受到侧信道攻击的影响（这是许多已知漏洞的来源）。好处是某些问题可以在不必报废硬件的情况下修补。尽管如此，这也使得更新过程成为潜在的攻击向量。

[AWS使用他们专有的Nitro系统](https://aws.amazon.com/blogs/security/confidential-computing-an-aws-perspective/)，表面上看起来很有趣，但缺乏公开细节和外部审查，因此很难与Intel和AMD的解决方案进行比较。在概念上，Azure和GCP将一些信任转移到CPU供应商（已经信任的AMD或Intel），而AWS则通过他们自己的定制硬件扮演同样的角色。

不要误会：我认为减少TBC（即机密计算的目标）是一个伟大且值得追求的目标。它应该增加攻击的成本并有助于保护敏感数据。但让我们清醒地看待当前的现状；适度的热情是适宜的。
