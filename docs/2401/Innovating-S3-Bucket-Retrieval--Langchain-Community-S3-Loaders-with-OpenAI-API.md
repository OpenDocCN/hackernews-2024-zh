<!--yml

类别：未分类

日期：2024-05-27 15:22:40

-->

# 创新的 S3 存储桶检索：Langchain 社区 S3 加载器与 OpenAI API

> 来源：[`blog.min.io/langchain-openai-s3-loader/`](https://blog.min.io/langchain-openai-s3-loader/)

在不断发展的数据存储和处理领域，将高效的云存储解决方案与先进的人工智能能力相结合，提供了一种处理大量数据的变革性方法。本文演示了使用 MinIO、Langchain 和 OpenAI 的 GPT-3.5 模型的实际实现，重点是摘要存储在 MinIO 存储桶中的文档。

## MinIO 的威力

MinIO 是开源的、高性能的对象存储，完全兼容 Amazon S3 API。以其可扩展性而闻名，MinIO 非常适合存储非结构化数据，如照片、视频、日志文件、备份和容器映像。这不仅仅是存储；MinIO 还提供数据复制、生命周期管理和高可用性等功能，使其成为现代云原生应用的首选。

## 集成 Langchain 和 OpenAI

[Langchain](https://www.langchain.com/?ref=blog.min.io)，一个基于 Python 的工具，促进了文档加载器与 AI 模型之间的交互。在我们的用例中，我们将 Langchain 与 OpenAI 的 [gpt-3.5-turbo-1106](https://platform.openai.com/docs/models/gpt-3-5?ref=blog.min.io) 模型结合起来，从 MinIO 存储桶中摘要文档。这个设置展示了人工智能如何从大量数据中提取重要信息，简化数据分析和解释。有关本文的附加信息和支持材料，如笔记本和加载的文档，请访问 [MinIO Github 存储库](https://github.com/minio/blog-assets?ref=blog.min.io) 中的 [langchain-s3-minio](https://github.com/minio/blog-assets/tree/main/langchain-s3-minio?ref=blog.min.io) 目录。

## 安装 Langchain

在深入实现之前，请确保已安装 Langchain。通过 pip 安装：

```
pip install --upgrade langchain 
```

这将封装我们将用于 S3 加载器和 OpenAI 模型的所有必需的库。

## **第一步：** Langchain S3 目录和文件加载器

最初，我们专注于使用 Langchain 的 `S3DirectoryLoader` 和 `S3FileLoader` 加载文档。这些加载器负责从 MinIO 存储桶中的指定目录和文件获取多个和单个文档。

### MinIO 配置和 Langchain S3 文件加载器

```
from langchain_community.document_loaders.s3_file import S3FileLoader

# MinIO Configuration for the public testing server
endpoint = 'play.min.io:9000'
access_key = 'minioadmin'
secret_key = 'minioadmin'
use_ssl = True

# Initialize and load a single document
file_loader = S3FileLoader(
    bucket='web-documentation',
    key='MinIO_Quickstart.md',
    endpoint_url=f'http{"s" if use_ssl else ""}://{endpoint}',
    aws_access_key_id=access_key,
    aws_secret_access_key=secret_key,
    use_ssl=use_ssl
)

document = file_loader.load()
```

Python Langchain 示例 - S3 文件加载器

### Langchain S3 目录加载器：

```
from langchain_community.document_loaders.s3_directory import S3DirectoryLoader

# Initialize and load documents
directory_loader = S3DirectoryLoader(
    bucket='web-documentation',
    prefix='',
    endpoint_url=f'http{"s" if use_ssl else ""}://{endpoint}',
    aws_access_key_id=access_key, 
    aws_secret_access_key=secret_key, 
    use_ssl=use_ssl
)

documents = directory_loader.load()
```

Python Langchain 示例 - S3 目录加载器

## **第二步：** 使用 OpenAI 进行摘要

在加载文档之后，我们使用 OpenAI 的 GPT-3.5 模型（通过 `ChatOpenAI` 包含在 Langchain 库中）生成摘要。这一步展示了模型理解和压缩内容的能力，从大型文档中提供快速见解。

要访问 OpenAI API，您可以通过访问[OpenAI 平台](https://platform.openai.com/api-keys?ref=blog.min.io)获取 API 密钥。一旦您获得了密钥，将其集成到以下代码中，以利用 GPT-3.5 的强大功能进行文档摘要。

### 文档摘要的代码示例：

```
from langchain_openai import ChatOpenAI
from langchain_core.output_parsers import StrOutputParser
from langchain_core.prompts import ChatPromptTemplate
from langchain_core.runnables import RunnableLambda
import os

# Set your OpenAI API key
os.environ['OPENAI_API_KEY'] = 'your-openai-api-key'
model = ChatOpenAI(temperature=0, model="gpt-3.5-turbo-1106")

prompt = ChatPromptTemplate.from_template(
    "Summarize the following document '{document_name}':{context}Please provide the summary and key points."
)

loaded_documents = [documents, document]  # From S3 Loaders
flattened_documents = [doc for sublist in loaded_documents for doc in sublist] 

for loaded_document in flattened_documents:
    document_text = loaded_document.page_content
    document_name = getattr(loaded_document, 'name', 'Unknown Document')  # Assuming each document has a 'name' attribute
    chain = (
        RunnableLambda(lambda x: {"context": document_text, "document_name": document_name})
        | prompt
        | model
        | StrOutputParser()
    )
    summary = chain.invoke(None)
    print("Summary:", summary) 
```

Python Langchain 示例 - 使用 OpenAI API 对文档进行摘要

*下面是运行此演示的输出，是将 LangChain 与 OpenAI 的 GPT-3.5 和 MinIO S3 存储集成的结果；为了演示目的，输出已经被缩短：*

```
Summary: The document is a quickstart guide for MinIO, a high-performance object storage system that is compatible with Amazon S3\. It explains how to run MinIO on bare metal hardware or in containers. For Kubernetes environments, it recommends using the MinIO Kubernetes Operator. The key points are:

- MinIO is a high-performance object storage system.
- It is released under the GNU Affero General Public License v3.0.
- MinIO is API compatible with Amazon S3.
- It can be used to build high-performance infrastructure for machine learning, analytics, and application data workloads.
- The guide provides quickstart instructions for running MinIO on bare metal hardware or in containers.
- For Kubernetes environments, the MinIO Kubernetes Operator is recommended.
```

OpenAI API 的响应

该方法突显了一种有趣的方式，使用 Langchain 框架从 S3 存储中加载文档并处理它们，而 OpenAI 的 GPT-3.5 模型则生成了`play.min.io`服务器中获取的`MinIO_Quickstart.md`的简洁摘要和要点。利用 AI 来分析和概括广泛的文档，为用户提供了快速而全面的理解，涵盖了安装、服务器配置、SDK 和其他 MinIO 功能等重要方面。它展示了 AI 从综合数据源中提取和呈现关键信息的能力。

## 使用 Langchain 从 MinIO 存储桶加载文档

MinIO、Langchain 和 OpenAI 的集成为管理大数据量提供了强大的工具集。虽然 Langchain 的 S3 加载器 S3DirectoryLoader 和 S3FileLoader 在从 MinIO 存储桶检索文档方面发挥了重要作用，但它们仅用于将数据加载到 Langchain 中。这些加载器不执行与上传数据到存储桶相关的操作。对于上传、修改或管理存储桶策略等任务，*MinIO Python SDK*是适当的工具。该 SDK 提供了一套全面的功能，用于与 MinIO 存储进行交互，包括文件上传、存储桶管理等。有关更多信息，请参阅[快速入门指南 —— 用于 Linux 的 MinIO 对象存储](https://min.io/docs/minio/linux/developers/python/minio-py.html?ref=blog.min.io)、[Python 客户端 API 参考 —— 用于 Linux 的 MinIO 对象存储](https://min.io/docs/minio/linux/developers/python/API.html?ref=blog.min.io)。

虽然 Langchain 通过使用 AI 模型简化了获取和处理数据的过程，但在 MinIO 存储桶中进行数据管理的重任依赖于 MinIO Python SDK。这是开发人员和数据工程师必须了解的重要区别，他们构建高效的、与 AI 集成的存储解决方案。要全面了解 MinIO 的功能以及如何利用其 Python SDK 进行各种存储操作，请参阅 MinIO 的官方[文档](https://docs.min.io/docs/python-client-api-reference.html?ref=blog.min.io)。

将 MinIO 对象存储用作 AI 和 ML 过程的主要数据存储库，可以简化数据管理流程。 MinIO 凭借其出色的一站式解决方案，[存储、管理和检索大型数据集](https://blog.min.io/the-architects-guide-to-the-modern-data-stack/)，这对于有效的 AI 和 ML 操作至关重要。 这种简化的方法减少了复杂性和开销，可能通过确保对数据的快速访问来加速洞察力。

对于那些对将 MinIO 与 Langchain 集成以增强 LLM 工具使用感兴趣的人士，《[使用 MinIO SDK 开发 Langchain 代理以增强 LLM 工具使用](https://blog.min.io/minio-langchain-tool)》一文全面探讨了这个主题。

在您的发展努力中祝您好运！我们希望 [MinIO](https://min.io/download?ref=blog.min.io) 继续在您的 AI/ML 旅程中发挥关键作用。 在 [Slack](https://slack.min.io/?ref=blog.min.io) 上与我们联系，分享您的见解和发现！
