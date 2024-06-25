<!--yml

类别：未分类

日期：2024-05-27 14:58:45

-->

# Datomic - Datomic Local 已发布

> 来源：[`blog.datomic.com/2023/08/datomic-local-is-released.html`](https://blog.datomic.com/2023/08/datomic-local-is-released.html)

*2023 年 8 月 16 日*

Datomic 团队很高兴地宣布[Datomic Local](https://docs.datomic.com/cloud/datomic-local.html)的发布，它在[Apache 许可证 2.0](https://www.apache.org/licenses/LICENSE-2.0.html)下提供。Datomic Local 是一个既可嵌入又可重新分发的库。它是 Datomic 的轻量级版本，旨在轻松嵌入应用程序中并免费重新分发。Datomic Local 支持客户端 API，使其成为小型单进程应用程序和基于 Datomic 客户端 API 的系统测试的优秀选择。

### 有什么新的？

Datomic Local 是由本地存储支持的 Datomic 的轻量级版本，支持[客户端 API](https://docs.datomic.com/cloud/client/client-api.html)。Datomic Local 和支持的二进制文件遵循[Apache 许可证 2.0](https://www.apache.org/licenses/LICENSE-2.0.html)许可证发布。

### 使用案例

Datomic 的 Datomic Local 版本适用于：

+   想要编写依赖于 Datomic 的库的人们

+   想要嵌入式数据库的人们

+   想要在本地开发的云用户

+   已经使用 dev-local 的人们

# 常见问题

### 什么是 Datomic Local？

Datomic Local 是由本地存储支持的 Datomic 的轻量级版本，支持客户端 API。

### 发生了什么变化？

Datomic Local 是一个库，可嵌入和可重新分发。Datomic Local 取代了 dev-local。

### 如何获取 Datomic Local？

### 许可证方面有什么变化？

Datomic Local 二进制文件可在[Apache 2.0 许可证](https://www.apache.org/licenses/LICENSE-2.0.html)下获得，这意味着可以重新分发它。

### Datomic Local 推荐给谁？

+   想要编写依赖于 Datomic 的库的人们

+   想要嵌入式数据库的人们

+   想要在本地开发的云用户

+   已经使用 dev-local 的人们

### Datomic Free 发生了什么？

如果你习惯使用 Datomic Free，请使用[Datomic Pro](https://www.datomic.com/)。如果你想要一个可重新分发、可嵌入的 Datomic，请使用 Datomic Local。

### Datomic Cloud、Datomic Local 和 Datomic Pro 之间有什么区别？

[**Datomic Cloud**](https://docs.datomic.com/cloud/index.html)与 AWS 紧密集成。它仅支持客户端访问。Datomic Cloud 具有 ions，这允许用户在 Datomic 上运行整个应用程序，通过 AWS Lambda 事件和 AWS API 网关进行可重现部署、弹性自动缩放和集成。

[**Datomic Pro**](https://docs.datomic.com/pro/overview/introduction.html)是一个分布式数据库，旨在支持在本地或云中运行的可扩展、灵活的应用程序。它专为大规模、多用户系统中针对多种存储进行设计。

[**Datomic Local**](https://docs.datomic.com/cloud/datomic-local.html) 是 Datomic 的轻量级版本，可以嵌入、重新分发，并支持客户端 API。它非常适用于单进程应用程序和测试基于客户端 API 的 Datomic 系统。

### Datomic Local 是开源的吗？

不。*二进制文件*遵循 Apache 2.0 协议，与其他版本的 Datomic 相同。

* * *
