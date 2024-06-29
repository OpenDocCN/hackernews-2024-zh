<!--yml

category: 未分类

date: 2024-05-27 13:38:22

-->

# 什么是亚马逊资源名称（ARN）？

> 来源：[https://everythingdevops.dev/what-is-amazon-resource-name-arn/](https://everythingdevops.dev/what-is-amazon-resource-name-arn/)

AWS 拥有许多[服务](https://aws.amazon.com/products/?aws-products-all.sort-by=item.additionalFields.productNameLowercase&aws-products-all.sort-order=asc&awsf.re%3AInvent=*all&awsf.Free%20Tier%20Type=*all&awsf.tech-category=*all)和资源，从 EC2 实例到[S3 存储桶](https://everythingdevops.dev/optimize-aws-storage-costs-with-amazon-s3-lifecycle-configurations/)，每种资源可以拥有多个实例。在需要识别特定资源的情况下，每个资源具有唯一的 ID 是非常合理的。这种唯一标识是称为亚马逊资源名称（ARN）。

在本文中，您将了解 ARN 及其在亚马逊网络服务（AWS）生态系统中的重要性。您将深入剖析 ARN 的结构以及如何检索它们。到本文末，识别和解析 ARN 对您来说将变得轻而易举。

## 什么是 ARN？

[ARN](https://docs.aws.amazon.com/IAM/latest/UserGuide/reference-arns.html) 是分配给 AWS 中每个资源的唯一标识符。它是一个字符串，用于标识 AWS 生态系统内的资源，如 Amazon EC2 实例、Amazon S3 存储桶、IAM（身份和访问管理）用户、角色、策略等。

ARN 用于在 AWS 的所有部分中唯一地指定资源。它们用于各种 AWS 服务和 API 中，以识别资源并授予权限。这意味着每个资源可以在不考虑其所属 AWS 区域或账户的情况下被唯一标识。

此普遍性使得 AWS 服务能够准确地定位和管理资源，而不受其位置或所有权的限制。例如，附加到一个 AWS 账户中的用户的 IAM 策略可以通过使用存储桶的 ARN 指向另一个 AWS 账户中的 S3 存储桶，以确保策略被正确应用。

## ARN 为什么重要？

除了唯一标识资源外，ARN 在 AWS 生态系统中扮演着至关重要的角色。它们可以让您以编程方式访问您的资源，这在使用基础架构即代码（IaC）工具如 CloudFormation、Terraform、[OpenTofu](https://everythingdevops.dev/kubernetes-with-opentofu-a-guide-to-being-fully-open-source/) 等时至关重要。

此外，以下是您应更多关注 ARN 的几个原因：

+   **资源级权限**：一些 AWS 服务允许您指定资源级权限，为同一服务内的各种资源授予不同级别的访问权限。ARN 在准确定义这些资源级权限中发挥着至关重要的作用。例如，您可以为同一 AWS 账户中的一个 S3 存储桶授予读取权限，为另一个 S3 存储桶授予写入权限。

+   **跨服务引用**：ARN能够实现跨服务引用，允许一个AWS服务的资源与另一个服务的资源进行交互。例如，附加到EC2实例的IAM策略可能会使用其ARN引用S3存储桶，从而使该实例能够安全地访问存储桶。

+   **API操作**：通过API或SDK与AWS进行程序交互时，会使用ARN引用资源。API和SDK提供根据资源类型及其属性构建ARN的方法，使程序化地处理它们变得更加容易。例如，在调用获取有关EC2实例信息的API时，您通常会指定该实例的ARN。

+   **IAM策略**：ARN在日常使用中的一个用例是在[IAM策略](https://docs.aws.amazon.com/IAM/latest/UserGuide/access_policies.html)中。这些策略定义谁可以访问哪些资源以及他们可以执行哪些操作。ARN用于指定策略适用的资源。使用ARN，您可以细粒度地定义IAM策略，以允许或拒绝访问AWS中的特定资源。

+   **审计和日志记录**：ARN经常包含在AWS CloudTrail日志和其他审计机制中。当你在日志中包含ARN时，可以跟踪哪些资源被访问、修改或删除，为你提供AWS账户内活动的详细审计跟踪。

+   **资源标签**：ARN的一个可选但很有用的功能是它们与资源标签的集成。标签是提供有关资源的元数据的键值对。通过结合ARN和标签，你可以创建适用于具有特定标签的资源的策略，从而更容易大规模管理资源。

## ARN的结构

AWS资源ARN具有由称为命名空间的各个组件构成的结构。每个命名空间为所识别的资源提供特定的信息。ARN由冒号（`:`）分隔，具有以下通用格式：

```
arn:partition:service:region:account-id:resource 
```

以下是ARN的每个组件的分解：

1.  **ARN前缀**：ARN始终以字符`arn:`开头。

1.  **分区**：资源所在的分区。这通常是`aws`，但也可以是中国的`aws-cn`或在AWSGovCloud（美国）中的`aws-us-gov`。

1.  **服务**：ARN中的这一部分指定资源所属的AWS服务。例如，`s3`代表亚马逊S3，`ec2`代表亚马逊EC2，`iam`代表IAM等。

1.  **区域**：资源所在的区域。此组件是可选的，仅存在于特定区域的资源中，例如EC2实例。如果资源是全球性的，则省略此部分。地区组件的示例是`us-west-2`，代表美国西部（俄勒冈州）地区。

1.  **账户 ID**：拥有资源的 AWS 账户 ID。这是每个 AWS 账户唯一的 12 位数字。如果您引用不同账户中的资源，则可以是您的账户 ID 或其他 AWS 账户的账户 ID。此 ID 看起来像 `123456789012`。

1.  **资源 ID**：账户内资源的唯一标识符。ARN 的这部分在账户内唯一标识资源。它可以是资源的名称或由 AWS 生成的唯一标识符。

ARN 的结构因资源类型而异。例如，一些 ARN 包括资源类型和资源 ID，用冒号（`:`）或斜杠（`/`）分隔。

以下是不同 ARN 变体的示例：

```
arn:partition:service:region:account-id:resource-type/resource-id
arn:partition:service:region:account-id:resource-type:resource-id 
```

+   **变体 1（斜杠分隔符）**：在此格式中，资源类型和资源 ID 由斜杠（`/`）分隔。某些 AWS 服务，如 Amazon S3 存储桶，通常使用此格式。

    **示例**：

```
arn:aws:s3:::example-bucket 
```

+   **变体 2（冒号分隔符）**：在这种格式中，资源类型和资源 ID 由冒号（`:`）分隔。大多数其他 AWS 服务使用此格式。

    **示例**：

```
arn:aws:ec2:us-west-2:123456789012:instance/i-1234567890abcdef0 
```

**注意：** 分隔符的选择不影响 ARN 的功能性；这只是 AWS 使用的约定。但对于给定服务的一致性，遵守适当的格式是至关重要的。

## ARN 路径

除了标准 ARN 结构外，某些资源支持使用 ARN 路径。ARN 路径是一种在支持层次结构资源命名的 AWS 服务中组织和结构化资源的方法。它们允许您以类似文件系统层次结构的方式组织资源。

ARN 路径附加到 ARN 的末尾，并由斜杠（`/`）分隔。路径组件表示资源的层次结构，每个元素指定层次结构中的一个级别。您可以使用路径来分组相关资源或表示资源之间的父子关系。

例如，一个 S3 存储桶可以存储对象。您可以使用 ARN 路径指定存储桶及其中的对象。路径看起来像这样：

```
arn:aws:s3:::example-bucket/example-folder/example-object 
```

在此示例中，`example-bucket` 是 S3 存储桶，`example-folder` 是存储桶中的文件夹，`example-object` 是文件夹中存储的对象。路径组件由斜杠分隔以指示层次结构。

## ARN 的通配符

ARN 的灵活性通过使用通配符进一步扩展。通配符是特殊字符，可用于 ARN 中表示多个资源或资源类型。它们提供了一种方法，可以指定一组资源或某一类型的所有资源，而无需明确列出每一个。

在处理 IAM 策略、AWS 资源策略和某些 AWS CLI 命令时，通配符尤其有用。特别是当您希望对共享公共模式的多个资源授予权限或执行操作时。

ARN中最常用的通配符是星号（`*`）字符。星号通配符在ARN组件内匹配零个或多个字符。它通常用于在策略或资源规范中匹配多个资源或值。

例如，在IAM策略中，`"Resource": "arn:aws:s3:::*"`授予对AWS帐户中所有S3存储桶的访问权限。

下面是一个使用ARN中通配符的IAM策略声明示例：

```
{
 "Effect": "Allow",
 "Action": "s3:GetObject",
 "Resource": "arn:aws:s3:::example-bucket/*"
} 
```

在这个例子中，星号（`*`）用于在存储桶名（`example-bucket`）之后匹配指定S3存储桶中的所有对象。

除了星号通配符外，在ARN中还有另一个通配符字符可以使用。这就是问号（`?`）通配符。问号通配符精确匹配ARN组件中的一个字符。它用于匹配ARN特定位置的单个字符。

在ARN中使用问号通配符的示例是：

```
{
 "Effect": "Allow",
 "Action": "s3:GetObject",
 "Resource": "arn:aws:s3:::example-bucket/prefix?/object"
} 
```

在这个例子中，问号通配符（`?`）匹配ARN路径组件中的单个字符。这允许您指定一个更具体的模式来匹配资源。

## 如何检索ARNs

有几种方法可以检索您的AWS资源的ARN。您选择的方法将取决于您的偏好和可用工具。

以下是查找您的AWS资源的ARN的一些常见方法：

1.  **AWS管理控制台**：[AWS管理控制台](https://aws.amazon.com/free/?gclid=Cj0KCQjwlZixBhCoARIsAIC745BaYzavvY4aiA_WKKzkx1MSkWM4C-8b24smig8zRqFLVO3fG6SHN4YaAkh7EALw_wcB&trk=99f831a2-d162-429a-9a77-a89f6b3bd6cd&sc_channel=ps&ef_id=Cj0KCQjwlZixBhCoARIsAIC745BaYzavvY4aiA_WKKzkx1MSkWM4C-8b24smig8zRqFLVO3fG6SHN4YaAkh7EALw_wcB:G:s&s_kwcid=AL!4422!3!645125273270!e!!g!!aws%20console!19574556890!145779847152&all-free-tier.sort-by=item.additionalFields.SortRank&all-free-tier.sort-order=asc&awsf.Free%20Tier%20Types=*all&awsf.Free%20Tier%20Categories=*all)提供了查看您的资源ARN的简便方法。通常，您会按照以下步骤进行：

+   导航到AWS管理控制台中特定服务的位置，您想要检索其ARN的资源所在位置。

+   定位您感兴趣的资源（例如S3存储桶、EC2实例、IAM角色）。

+   典型情况下，资源的ARN应该显示在资源的详细信息或设置中。

+   从控制台复制ARN并根据需要使用它。

**2\.  AWS CLI命令**：您还可以使用[AWS CLI](https://docs.aws.amazon.com/cli/latest/userguide/cli-chap-getting-started.html)命令以编程方式检索ARN。以下是用于检索不同资源ARN的AWS CLI命令示例：

+   要检索S3存储桶的ARN：

```
aws s3api get-bucket-arn --bucket your-bucket-name 
```

+   要检索EC2实例的ARN：

```
aws ec2 describe-instances --instance-ids your-instance-id --query 'Reservations[].Instances[].Arn' --output text 
```

+   用实际感兴趣的资源名称或ID替换`your-bucket-name`和`your-instance-id`。

**3\. AWS CloudFormation 输出**：如果您使用 [AWS CloudFormation](https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/Welcome.html) 来提供和管理 AWS 资源，您可以从 CloudFormation 堆栈输出中检索 ARN。此方法在您希望自动化检索 ARN 作为基础设施部署过程的一部分时非常有用。

+   在您的 CloudFormation 模板中为资源的 ARN 定义一个输出：

```
Outputs:
MyResourceArn:
Description: "ARN of my resource."
Value: !GetAtt MyResource.Arn
```

+   在部署 CloudFormation 堆栈后，您可以使用 AWS CLI 或 AWS 管理控制台检索输出值。

```
aws cloudformation describe-stacks --stack-name your-stack-name --query "Stacks[0].Outputs[?OutputKey=='MyResourceArn'].OutputValue" --output text 
```

+   将 `your-stack-name` 替换为您的 CloudFormation 堆栈的名称。

## 常见 ARN 示例

AWS 中有各种类型的资源，每种资源都有其独特的 ARN 格式。以下是不同 AWS 云资源的一些常见 ARN 示例：

```
arn:aws:s3:::example-bucket 
```

```
arn:aws:ec2:us-west-2:123456789012:instance/i-1234567890abcdef0 
```

```
arn:aws:iam::123456789012:role/MyRole 
```

```
arn:aws:lambda:us-west-2:123456789012:function:MyFunction 
```

```
arn:aws:cloudformation:us-west-2:123456789012:stack/MyStack/1a2b3c4d5e6f7g8h9i0j1k2l3m4n5o6p7q8r9s0t 
```

+   **Amazon 关系型数据库服务（RDS）实例**：

```
arn:aws:rds:us-west-2:123456789012:db:my-database 
```

这些示例展示了 AWS 中资源的多样性以及 ARN 如何用于唯一标识 AWS 生态系统内的每个资源。

## 使用 ARN 的最佳实践

ARN 是一项敏感信息，应谨慎处理，以确保 AWS 资源的安全性和完整性。这是因为它们包含了它们所标识资源的信息，如果暴露或滥用，可能会导致对资源的未经授权访问或意外操作。

下面是在 AWS 中使用 ARN 的一些最佳实践：

+   **最小化暴露度**：在配置、脚本或代码中使用 ARN 时，避免不必要地暴露它们。将 ARN 的可见性限制为仅需要知道它们以执行任务的人员。如果必须使用它们，请考虑使用变量占位符或环境变量安全地存储 ARN。

+   **在资源引用中要具体化**：尽可能明确指定资源，而不是使用通配符。这有助于减少意外访问不应包含在策略或配置中的资源的风险。

+   **使用最小权限原则**：您应该仅授予资源执行其预期任务所需的权限。避免授予比所需更广泛访问权限的过于宽松的策略。定期审查和优化 IAM 策略，以确保与最小权限原则一致。

## 结论

在本文中，我们详细介绍了 ARN 的许多内容，从它们的定义到它们的重要性。我们还讨论了如何使用不同方法检索 ARN，并提供了各种 AWS 云服务的常见 ARN 示例。

当您看到一个 ARN 时，现在您知道它不仅仅是一串字符，分解其结构可以为您提供有关其表示的资源的许多信息。

********************************************************************************************************************************喜欢这篇文章吗？请在下方订阅我们的新闻简报，成为超过1000名订阅者之一，了解DevOps领域的最新动态。立即订阅！********************************************************************************************************************************
