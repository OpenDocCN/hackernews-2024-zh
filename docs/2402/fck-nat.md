<!--yml

类别：未分类

日期：2024-05-27 14:32:23

-->

# fck-nat

> 来源：[https://fck-nat.dev/v1.3.0/](https://fck-nat.dev/v1.3.0/)

# 介绍

欢迎使用fck-nat。这是一个成本可行的可配置NAT！

+   是否为AWS托管型NAT网关支付过高费用？fck-nat。

+   想要使用NAT实例并及时更新最新的安全补丁吗？fck-nat。

+   想要重用您的堡垒主机作为NAT？fck-nat。

fck-nat提供了基于Amazon Linux 2023构建的ARM和x86的AMI，可以支持t4g.nano实例上的最高5Gbps突发NAT流量。这与托管型NAT网关相比如何？

按小时计费：

+   托管型NAT网关按小时计费：$0.045

+   t4g.nano 按小时：$0.0042

每GB价格：

+   托管型NAT网关每GB价格：$0.045

+   fck-nat每GB价格：$0.00

空闲时，fck-nat的成本是托管型NAT网关的10%。实际上，节省的费用更大。

*"但是AWS的NAT实例AMI呢？"*

官方支持的AWS NAT实例AMI自2018年以来没有更新，仍在运行Amazon Linux 1，该版本已经停止更新，并且不支持ARM架构，这意味着它无法部署在EC2最经济的实例类型上。fck-nat。

*"何时应选择使用托管型NAT网关而不是fck-nat？"*

AWS限制EC2实例的出站互联网带宽为5Gbps。这意味着fck-nat能够支持的最高带宽（同时保持成本效益）为5Gbps。这足以涵盖非常广泛的使用案例，但如果您需要额外的带宽，则应使用托管型NAT网关。如果AWS解除了EC2的互联网出口带宽限制，您可以以成本效益的方式运行高达25Gbps的fck-nat，但那时您是否还需要托管型NAT网关呢？fck-nat。

[阅读更多关于EC2带宽限制的信息](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/ec2-instance-network-bandwidth.html)

另外，如果您对非托管服务有过敏反应，fck-nat可能不适合您。虽然fck-nat支持高可用模式，但并非完全免受故障影响（尽管发生的可能性非常罕见）。如果您的工作负载要求五个9的可用性时间，那么托管型NAT网关可能更适合您。

## 使用fck-nat

fck-nat的主要目标是尽可能简化、安全和可配置地部署您自己的NAT实例。虽然fck-nat力求为大多数人提供即插即用的选项和指南，但每个人的需求都不同。在fck-nat无法提供现成解决方案时，我们尽量为您提供您需要的所有开关，以尽可能少的麻烦让您快速上手。我们遵循“电池包含在内，但可替换”的理念。

### 获取fck-nat AMI

fck-nat提供了基于Amazon Linux 2023的arm64和x86_64版本的公共AMI。如果您宁愿使用不同的基础镜像或自行托管AMI，您可以构建自己的AMI。

#### 公共fck-nat AMI

fck-nat 目前在大多数地区提供公共 AMI。您可以在 [`packer/fck-nat-public-all-regions.pkrvars.hcl`](https://github.com/AndrewGuenther/fck-nat/blob/main/packer/fck-nat-public-all-regions.pkrvars.hcl) 查看完整列表。虽然 arm64 映像最具成本效益，但也提供 x86_64 映像。您可以使用以下查询查看可用的 fck-nat AMI：

```
`# Amazon Linux 2023 based AMIs
aws ec2 describe-images --owners 568608671756 --filters 'Name=name,Values=fck-nat-al2023-*'` 
```

#### 构建您自己的 fck-nat AMI

fck-nat 映像是使用 [Packer](https://www.packer.io/) 构建的。您可以在 `packer` 文件夹内找到用于构建公共映像的 Packer 文件。我们的 Packer 配置广泛使用变量，允许您自定义基础映像、构建区域、架构等。如果公开可用的 AMI 不符合您的需求，或者您希望自行托管 AMI，则可以创建自己的 `pkrvars.hcl` 文件并构建您自己的 AMI。

```
`packer  build  -var-file="your-var-file.pkrvars.hcl"  ./packer/fck-nat.pkr.hcl` 
```

### 安装 fck-nat 的 RPM 包

如果您有现有的 AMI，并且希望在其上安装 fck-nat，则可以从 [Releases](https://github.com/AndrewGuenther/fck-nat/releases) 页面获取 fck-nat 的 `.rpm` 构建版本。

### 使用您的 fck-nat AMI

AMI 并不是您启动和运行 fck-nat 所需的全部。需要配置一些选项以将流量路由到您的 NAT。主要是，您必须：

1.  为您的 fck-nat 实例配置安全组

1.  禁用源/目标检查

1.  更新您的 VPC 路由表

有些工具可以为您完成此任务，而其他工具则无法。有关使用您喜欢的基础设施即代码工具部署 fck-nat 的更多信息，请查看[部署页面](deploying/)。
