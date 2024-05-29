<!--yml
category: 未分类
date: 2024-05-27 15:01:03
-->

# New from Google Cloud: Assured Open Source Software service | Google Cloud Blog

> 来源：[https://cloud.google.com/blog/products/identity-security/introducing-assured-open-source-software-service](https://cloud.google.com/blog/products/identity-security/introducing-assured-open-source-software-service)

 The figure above details the many stages of the software supply chain for an open source dependency. Enterprises have widely different entry points to this lifecycle - some organizations may build packages from source themselves, while others pull packages from repos that they trust. 

A few organizations including Google centralize control and actively secure each step of the end-to-end process. In our case, we start by maintaining separate secured copies of the source code for our dependencies and perform our own vulnerability scanning. We continuously fuzz [550 of the most commonly-used open source projects](https://github.com/google/oss-fuzz), and as of January 2022 have found [more than 36,000 vulnerabilities](https://github.com/google/oss-fuzz). This makes us one of the largest contributors to the OSV.

We then manage an end-to-end build, deploy, and distribution process that includes integrated integrity, provenance, and security checks. Based on our internal security practices, we have created the [SLSA framework](https://slsa.dev/) to enable organizations to assess the maturity of their software supply chain security and understand key steps to progress to the next level. 

We recognize that most organizations do not have the resources or experience to construct and operate such a comprehensive program. Instead, their development teams might individually decide where they get third-party source code and packages, how they are built, and how to redistribute them within their own organizations according to their goals, threat and risk model, and resources. However, the lack of an end-to-end process creates risk exposure [each step of the way](https://security.googleblog.com/2021/08/updates-on-our-continued-collaboration.html).

Assured OSS allows enterprise customers to directly benefit from the in-depth, end-to-end security capabilities and practices we apply to our own OSS portfolio by providing access to the same OSS packages that Google depends on. Users will also be able to submit packages from their own OSS portfolio to be secured and managed through the Google Cloud managed service.

### Take the next step

We are excited to partner with you to help manage your use of OSS in your enterprise development workflows. Our collaboration with Snyk will help further simplify and secure cloud migrations and application modernization for developers, especially on Google Cloud. 

To learn more about Assured OSS and get started, please fill out this interest [form](https://go.chronicle.security/assured-oss).