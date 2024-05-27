<!--yml
category: 未分类
date: 2024-05-27 13:38:22
-->

# What Is Amazon Resource Name (ARN)?

> 来源：[https://everythingdevops.dev/what-is-amazon-resource-name-arn/](https://everythingdevops.dev/what-is-amazon-resource-name-arn/)

AWS has many [services](https://aws.amazon.com/products/?aws-products-all.sort-by=item.additionalFields.productNameLowercase&aws-products-all.sort-order=asc&awsf.re%3AInvent=*all&awsf.Free%20Tier%20Type=*all&awsf.tech-category=*all) and resources ranging from EC2 instances to [S3 buckets](https://everythingdevops.dev/optimize-aws-storage-costs-with-amazon-s3-lifecycle-configurations/), and you can have multiple instances for each resource. In a scenario where you need to identify a specific resource, it is only logical to have a unique ID for each resource. Well, there is, and it's called Amazon Resource Name (ARN).

In this article, you will understand ARNs and their significance within the Amazon Web Services (AWS) ecosystem. You'll get to dissect the structure of an ARN and how you can retrieve them. By the end of this article, identifying and deciphering ARNs will be a breeze for you.

## What Is an ARN?

An [ARN](https://docs.aws.amazon.com/IAM/latest/UserGuide/reference-arns.html) is a unique identifier assigned to each resource in AWS. It is a string that's used to identify resources within the AWS ecosystem. ARNs identify resources such as Amazon EC2 instances, Amazon S3 buckets, IAM (Identity and Access Management) users, roles, policies, and others within AWS.

ARNs are required to specify a resource unambiguously across all of AWS. They are used in various AWS services and APIs to identify resources and grant permissions. This means that each resource can be identified uniquely regardless of the AWS region or AWS account it belongs to.

This universality allows AWS services to accurately locate and manage resources irrespective of their location or ownership. For example, an IAM policy attached to a user in one AWS account can refer to an S3 bucket in another AWS account by using the ARN of the bucket, ensuring that the policy is applied correctly.

## Why are ARNs Important?

Aside from uniquely identifying resources, ARNs play a crucial role in the AWS ecosystem. They can allow you to access your resources programmatically, which is essential when using Infrastructure as Code (IaC) tools like CloudFormation, Terraform, [OpenTofu](https://everythingdevops.dev/kubernetes-with-opentofu-a-guide-to-being-fully-open-source/) etc.

Alongside this, below are a few reasons why you should pay more attention to ARNs:

*   **Resource-Level Permissions**: Some AWS services allow you to specify resource-level permissions, granting different levels of access to various resources within the same service. ARNs play a vital role in defining these resource-level permissions accurately. For instance, you can grant read access to one S3 bucket and write access to another S3 bucket within the same AWS account.
*   **Cross-Service References**: ARNs enable cross-service references, allowing resources from one AWS service to interact with resources from another service. For example, an IAM policy attached to an EC2 instance might reference an S3 bucket using its ARN, enabling the instance to access the bucket securely.
*   **API Operations**: When interacting with AWS programmatically through APIs or SDKs, ARNs are used to reference resources. APIs and SDKs provide methods for constructing ARNs based on the type of resource and its attributes, making it easier to work with them programmatically. For example, when making an API call to retrieve information about an EC2 instance, you would typically specify the ARN of the instance.
*   **IAM Policies**: An everyday use case for ARNs is in [IAM policies](https://docs.aws.amazon.com/IAM/latest/UserGuide/access_policies.html). These policies define who can access which resources and what actions they can perform. ARNs are used to specify the resources to which the policy applies. With ARNs, you can granularly define IAM policies to allow or deny access to specific resources within AWS.
*   **Auditing and Logging**: ARNs are often included in AWS CloudTrail logs and other auditing mechanisms. When you include ARNs in your logs, you can track which resources were accessed, modified, or deleted, providing a detailed audit trail of activities within your AWS account.
*   **Resource Tagging**: An optional but useful ability of ARNs is their integration with resource tagging. Tags are key-value pairs that provide metadata about resources. By combining ARNs with tags, you can create policies that apply to resources with specific tags, making it easier to manage resources at scale.

## Structure of an ARN

AWS resource ARNs have a specific structure consisting of various components called namespaces. Each namespace provides a particular piece of information about the resource being identified. ARNs are delimited by colons (`:`) and have the following general format:

```
arn:partition:service:region:account-id:resource 
```

Below is a breakdown of each component of an ARN:

1.  **ARN Prefix**: The ARN always starts with the characters `arn:`.
2.  **Partition**: The partition that the resource is in. This is usually `aws` but can also be `aws-cn` for resources in China or `aws-us-gov` in AWS GovCloud (US).
3.  **Service**: This part of the ARN specifies the AWS service that the resource belongs to. For example, `s3` for Amazon S3, `ec2` for Amazon EC2, `iam` for IAM, etc.
4.  **Region**: The region where the resource is located. This component is optional and is only present for region-specific resources, such as EC2 instances. If the resource is global, this part is omitted. An example of a region component is `us-west-2` for the US West (Oregon) region.
5.  **Account ID**: The AWS account ID that owns the resource. This is a 12-digit number unique to each AWS account. It can be your account ID or the account ID of another AWS account if you are referencing a resource in a different account. This ID looks like `123456789012`.
6.  **Resource ID**: The unique identifier for the resource within the account. This part of the ARN uniquely identifies the resource within the account. It can be the resource's name or a unique identifier generated by AWS.

There are variations in the structure of ARNs depending on the resource type. For example, some ARNs include a resource type in addition to the resource ID, separated by a colon (`:`) or a slash (`/`).

The following are examples of different ARN variations:

```
arn:partition:service:region:account-id:resource-type/resource-id
arn:partition:service:region:account-id:resource-type:resource-id 
```

*   **Variation 1 (Slash Separator)**: In this format, the resource type and the resource ID are separated by a forward slash (`/`). This format is commonly used for certain AWS services, such as Amazon S3 buckets.
    **Example**:

```
arn:aws:s3:::example-bucket 
```

*   **Variation 2 (Colon Separator)**: In this format, the resource type and the resource ID are separated by a colon (`:`). This format is used for most other AWS services.
    **Example**:

```
arn:aws:ec2:us-west-2:123456789012:instance/i-1234567890abcdef0 
```

**NOTE:** The choice of separator doesn't affect the functionality of the ARN; it's merely a convention used by AWS. However, adhering to the appropriate format for a given service is essential for consistency.

## ARN Paths

In addition to the standard ARN structure, some resources support using ARN paths. ARN paths are a way to organize and structure resources within AWS services that support hierarchical resource naming. They allow you to organize resources in a tree-like structure, similar to a file system hierarchy.

ARN paths are appended to the end of the ARN and are separated by a forward slash (`/`). The path components represent the hierarchy of the resources, with each element specifying a level in the hierarchy. You can use paths to group related resources or represent parent-child relationships between resources.

For example, an S3 bucket can have objects stored within it. You can use an ARN path to specify the bucket and object within it. The path would look like this:

```
arn:aws:s3:::example-bucket/example-folder/example-object 
```

In this example, `example-bucket` is the S3 bucket, `example-folder` is a folder within the bucket, and `example-object` is an object stored within the folder. The path components are separated by forward slashes to indicate the hierarchy.

## ARN Wildcards

The flexibility of ARNs extends further with the use of wildcards. Wildcards are special characters that can be used in ARNs to represent multiple resources or resource types. They provide a way to specify a group of resources or all resources of a certain type without explicitly listing each one.

Wildcards can be particularly useful when working with IAM policies, AWS resource policies, and certain AWS CLI commands. Especially when you want to grant permissions or perform actions on multiple resources that share a common pattern.

The most common wildcard used in ARNs is the asterisk (`*`) character. The asterisk wildcard matches zero or more characters within the ARN component. It's often used to match multiple resources or values in policy or resource specification.

For example, in an IAM policy, `"Resource": "arn:aws:s3:::*"` grants access to all S3 buckets in the AWS account.

Below is an example IAM policy statement using a wildcard in the ARN:

```
{
 "Effect": "Allow",
 "Action": "s3:GetObject",
 "Resource": "arn:aws:s3:::example-bucket/*"
} 
```

In this example, the wildcard (`*`) is used after the bucket name (`example-bucket`) to match all objects within the specified S3 bucket.

Aside from the asterisk wildcard, there is another wildcard character that you can use in ARNs. This is the question mark (`?`) wildcard. The question mark wildcard matches exactly one character within the ARN component. It's used to match a single character in a specific position within the ARN.

An example of using the question mark wildcard in an ARN is:

```
{
 "Effect": "Allow",
 "Action": "s3:GetObject",
 "Resource": "arn:aws:s3:::example-bucket/prefix?/object"
} 
```

In this example, the question mark wildcard (`?`) matches a single character in the path component of the ARN. This allows you to specify a more specific pattern for matching resources.

## How to Retrieve ARNs

There are several ways to retrieve the ARNs of your AWS resources. The method you choose will depend on your preference and the tools you have available.

Below are some common methods for finding the ARNs of your AWS resources:

1.  **AWS Management Console**: The [AWS Management Console](https://aws.amazon.com/free/?gclid=Cj0KCQjwlZixBhCoARIsAIC745BaYzavvY4aiA_WKKzkx1MSkWM4C-8b24smig8zRqFLVO3fG6SHN4YaAkh7EALw_wcB&trk=99f831a2-d162-429a-9a77-a89f6b3bd6cd&sc_channel=ps&ef_id=Cj0KCQjwlZixBhCoARIsAIC745BaYzavvY4aiA_WKKzkx1MSkWM4C-8b24smig8zRqFLVO3fG6SHN4YaAkh7EALw_wcB:G:s&s_kwcid=AL!4422!3!645125273270!e!!g!!aws%20console!19574556890!145779847152&all-free-tier.sort-by=item.additionalFields.SortRank&all-free-tier.sort-order=asc&awsf.Free%20Tier%20Types=*all&awsf.Free%20Tier%20Categories=*all) provides an easy way to view the ARNs of your resources. Typically, you'd follow these steps:

*   Navigate to the specific service in the AWS Management Console where the resource whose ARN you want to retrieve is located.
*   Locate the resource you're interested in (e.g., an S3 bucket, an EC2 instance, an IAM role).
*   Typically, the ARN of the resource should be displayed somewhere in the resource's details or settings.
*   Copy the ARN from the console and use it as needed.

**2\.  AWS CLI Commands**: You can also use [AWS CLI](https://docs.aws.amazon.com/cli/latest/userguide/cli-chap-getting-started.html) commands to retrieve ARNs programmatically. Below are examples of AWS CLI commands to retrieve the ARNs of different resources:

*   To retrieve the ARN of an S3 bucket:

```
aws s3api get-bucket-arn --bucket your-bucket-name 
```

*   To retrieve the ARN of an EC2 instance:

```
aws ec2 describe-instances --instance-ids your-instance-id --query 'Reservations[].Instances[].Arn' --output text 
```

*   Replace `your-bucket-name` and `your-instance-id` with the actual names or IDs of the resources you're interested in.

**3\. AWS CloudFormation Outputs**: If you're using [AWS CloudFormation](https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/Welcome.html) to provision and manage your AWS resources, you can retrieve ARNs from CloudFormation stack outputs. This method is useful when you want to automate the retrieval of ARNs as part of your infrastructure deployment process.

*   Define an output for the ARN of the resource in your CloudFormation template:

```
Outputs:
MyResourceArn:
Description: "ARN of my resource."
Value: !GetAtt MyResource.Arn
```

*   After deploying the CloudFormation stack, you can retrieve the output value using the AWS CLI or AWS Management Console.

```
aws cloudformation describe-stacks --stack-name your-stack-name --query "Stacks[0].Outputs[?OutputKey=='MyResourceArn'].OutputValue" --output text 
```

*   Replace `your-stack-name` with the name of your CloudFormation stack.

## Common ARN Examples

There are various types of resources in AWS, each with its unique ARN format. Below are some common examples of ARNs for different AWS cloud resources:

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

*   **Amazon Relational Database Service (RDS) Instance**:

```
arn:aws:rds:us-west-2:123456789012:db:my-database 
```

These examples showcase the diversity of resources in AWS and how ARNs are used to identify each resource within the AWS ecosystem uniquely.

## Best Practices for Working with ARNs

An ARN is a sensitive piece of information that should be handled carefully to ensure the security and integrity of your AWS resources. This is because they contain information about the resources they identify, and if exposed or misused, they can lead to unauthorized access or unintended actions on your resources.

Below are some best practices for working with ARNs in AWS:

*   **Minimal Exposure**: When working with ARNs in configurations, scripts, or code, avoid exposing them unnecessarily. Limit the visibility of ARNs to only those who need to know them to perform their tasks. If you're to use them, consider using variable placeholders or environment variables to store ARNs securely.
*   **Be Specific in Resource References**: Whenever possible, specify resources explicitly rather than using wildcards. This helps to reduce the risk of unintended access to resources that should not be included in the policy or configuration.
*   **Use Least Privilege Principle**: You should grant only the permissions necessary for a resource to perform its intended tasks. Avoid overly permissive policies that grant broader access than required. Regularly review and refine IAM policies to ensure they align with the principle of least privilege.

## Conclusion

In this article, we covered so much about ARNs, from what they are to why they are essential. We also discussed how you can retrieve ARNs using different methods and provided examples of common ARNs for various AWS cloud services.

When you see an ARN, you now know that it's more than just a string of characters, and breaking down its structure can give you much information about the resource it represents.

********************************************************************************************************************************Like this article? Sign up for our newsletter below and become one of over 1000 subscribers who stay informed on the latest developments in the world of DevOps. Subscribe now!********************************************************************************************************************************