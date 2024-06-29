<!--yml

category: 未分类

date: 2024-05-27 15:01:03

-->

# 新的Google Cloud：保证的开源软件服务 | Google Cloud Blog

> 来源：[https://cloud.google.com/blog/products/identity-security/introducing-assured-open-source-software-service](https://cloud.google.com/blog/products/identity-security/introducing-assured-open-source-software-service)

上图详细说明了开源依赖软件的软件供应链的许多阶段。企业在这个生命周期中有着广泛不同的入口点 - 有些组织可能自行从源代码构建软件包，而其他则从他们信任的仓库中拉取软件包。

包括Google在内的少数组织集中控制并积极保护端到端流程的每一步。在我们的情况下，我们首先维护我们依赖的源代码的单独安全副本，并进行自己的漏洞扫描。我们持续对[550个最常用的开源项目](https://github.com/google/oss-fuzz)进行模糊测试，截至2022年1月，已发现超过36,000个漏洞。这使我们成为OSV的最大贡献者之一。

然后，我们管理端到端的构建、部署和分发过程，包括集成的完整性、来源和安全性检查。基于我们的内部安全实践，我们创建了[SLSA框架](https://slsa.dev/)，帮助组织评估其软件供应链安全的成熟度，并了解向下一级进展的关键步骤。

我们认识到大多数组织没有资源或经验来构建和运行这样全面的程序。相反，他们的开发团队可能会个别决定获取第三方源代码和软件包的位置、构建方式以及根据他们的目标、威胁和风险模型以及资源在其组织内部重新分发它们的方式。然而，缺乏端到端的流程会在[每一步中](https://security.googleblog.com/2021/08/updates-on-our-continued-collaboration.html)暴露出风险。

保证的开源软件（Assured OSS）允许企业客户直接从我们自己的开源软件组合中受益，通过提供访问Google依赖的同一开源软件包的深度、端到端的安全能力和实践。用户还可以提交他们自己的开源软件组合中的软件包，以通过Google Cloud管理服务进行安全保障和管理。

### 迈出下一步

我们很高兴与您合作，帮助您管理企业开发工作流中对开源软件的使用。我们与Snyk的合作将进一步简化和保障开发者在Google Cloud上进行云迁移和应用现代化的过程。

要了解更多关于保证的开源软件（Assured OSS）并开始的信息，请填写此兴趣[表单](https://go.chronicle.security/assured-oss)。
