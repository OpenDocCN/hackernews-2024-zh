<!--yml

类别：未分类

日期：2024-05-27 14:42:29

-->

# Nimbus的创始工程师 | Y Combinator

> 来源：[https://www.ycombinator.com/companies/nimbus-3/jobs/TgQFIkz-founding-engineer](https://www.ycombinator.com/companies/nimbus-3/jobs/TgQFIkz-founding-engineer)

Nimbus是现代可观察性的智能控制平台。我们的使命是为全球每个组织实现类似FAANG的可观察性。我们首先从优化引擎开始，可以在不需要开发人员手动干预的情况下将Datadog成本降低一个数量级。

工作原理：更新Datadog代理配置中的三行，即可将数据发送到Nimbus。Nimbus分析所有传入的日志以查找常见模式，并在上游代理数据的同时进行优化。根据分析结果，Nimbus提出可供人类一键审查和部署的优化方案。我们采用新颖的压缩技术将计费量降低一个数量级，而不会丢失数据。你可以在[nimbus.dev](https://nimbus.dev/?utm_source=hm)上观看演示。

我们正在寻找我们的第一位创始工程师，帮助我们扩展日志以外的功能。今天的Nimbus可以在不丢失信号的情况下将日志量减少超过90% - 我们的即时优先事项是对指标和跟踪也做同样处理。

你将从基础层开始，并直接与我们的创始人在整个技术栈的各个层级一起工作。我们正在快速成长，有多年的发展时间，预计在今年中期实现盈利。

## What You'll Do

你将拥有自主权，可以解决一系列具有挑战性的问题。以下是你可能会涉及的一些工作：

+   派生新算法，优化可观察性数据而不丢失信号

+   构建高性能、始终可用的多云多租户可观察性管道

+   在Rust中进行低级别调整以提高管道性能

+   设计和扩展我们的[自定义查询语言](https://docs.nimbus.dev/overview/ntl)

+   与开源工作组合作，制定可观察性标准

+   创建负载测试，以评估不同网络拓扑和计算节点上管道性能

+   帮助建立一个由世界级、高度合作的软件工程师组成的团队

## Who You Are

+   你是一位杰出的资深工程师（具有6年以上的行业经验），沟通清晰，渴望改进

+   你习惯于在技术栈的各个层次上工作 - 从使用Wireshark捕获网络流量到部署CRUD应用程序

+   你深知如何与AWS、Docker和Kubernetes合作

+   至少精通以下一种语言：Typescript、Python、Rust

## 奖金

+   在可观察性领域有工作经验

+   熟悉Datadog的基础设施、APM和日志服务

+   对可观察性管道的深入理解（理想情况下是向量）

+   深入理解Prometheus和Grafana

+   与CDK合作过

## 我们的技术栈

### 代码

+   后端使用Typescript（Express）和前端（React、Next和Material）

+   用于管道代理的Rust（具体来说是[Vector](https://vector.dev/)）

+   机器学习和分析引擎Python

### 基础设施

+   亚马逊网络服务AWS

+   Kubernetes（EKS）

+   监控使用的Grafana & Prometheus

+   用于基础设施即代码的CDK和Kubectl
