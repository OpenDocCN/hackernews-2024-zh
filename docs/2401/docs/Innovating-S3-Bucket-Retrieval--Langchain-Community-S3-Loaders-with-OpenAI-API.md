<!--yml
category: 未分类
date: 2024-05-27 15:22:40
-->

# Innovating S3 Bucket Retrieval: Langchain Community S3 Loaders with OpenAI API

> 来源：[https://blog.min.io/langchain-openai-s3-loader/](https://blog.min.io/langchain-openai-s3-loader/)

In the rapidly evolving world of data storage and processing, combining efficient cloud storage solutions with advanced AI capabilities presents a transformative approach to handling vast volumes of data. This article demonstrates a practical implementation using MinIO, Langchain and OpenAI’s GPT-3.5 model, focusing on summarizing documents stored in MinIO buckets.

## The Power of MinIO

MinIO is open-source, high-performance object storage that is fully compatible with the Amazon S3 API. Known for its scalability, MinIO is ideal for storing unstructured data such as photos, videos, log files, backups and container images. It’s not just about storage; MinIO also offers features like data replication, lifecycle management and high availability, making it a top choice for modern cloud-native applications.

## Integrating Langchain and OpenAI

[Langchain](https://www.langchain.com/?ref=blog.min.io), a Python-based tool, facilitates the interaction between document loaders and AI models. In our use case, we combine Langchain with OpenAI’s [gpt-3.5-turbo-1106](https://platform.openai.com/docs/models/gpt-3-5?ref=blog.min.io) model to summarize documents from MinIO buckets. This setup exemplifies how AI can extract essential information from extensive data, simplifying data analysis and interpretation. For additional information and supporting materials related to this article such as notebooks and loaded documents, please visit the [MinIO Github repository](https://github.com/minio/blog-assets?ref=blog.min.io) in the [langchain-s3-minio](https://github.com/minio/blog-assets/tree/main/langchain-s3-minio?ref=blog.min.io) directory.

## Installing Langchain

Before diving into the implementation, ensure you have Langchain installed. Install it via pip:

```
pip install --upgrade langchain 
```

This will encapsulate all the required libraries we will be using for our S3 loaders and OpenAI model.

## **Step 1:** Langchain S3 Directory and File Loaders

Initially, we focus on loading documents using Langchain's `S3DirectoryLoader` and `S3FileLoader`. These loaders are responsible for fetching multiple and single documents from specified directories and files in MinIO buckets.

### MinIO Configurations and Langchain S3 File Loader

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

Python Langchain Example - S3 File Loader

### Langchain S3 Directory Loader:

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

Python Langchain Example - S3 Directory Loader

## **Step 2:** Summarizing with OpenAI

After loading the documents, we use OpenAI's GPT-3.5 model (which are included in the Langchain library via `ChatOpenAI`) to generate summaries. This step illustrates the model's capability to understand and condense the content, providing quick insights from large documents. 

To access the OpenAI API, you can acquire an API key by visiting the [OpenAI platform](https://platform.openai.com/api-keys?ref=blog.min.io). Once you have the key, integrate it into the code below, to harness the power of GPT-3.5 for document summarization.

### Code Example for Document Summarization:

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

Python Langchain Example - Summarizing Documents with OpenAI API

*Below is the output from running this demo, and is a result of integrating LangChain with OpenAI’s GPT-3.5 and MinIO S3 storage; the output has been shortened for demonstrative purposes:*

```
Summary: The document is a quickstart guide for MinIO, a high-performance object storage system that is compatible with Amazon S3\. It explains how to run MinIO on bare metal hardware or in containers. For Kubernetes environments, it recommends using the MinIO Kubernetes Operator. The key points are:

- MinIO is a high-performance object storage system.
- It is released under the GNU Affero General Public License v3.0.
- MinIO is API compatible with Amazon S3.
- It can be used to build high-performance infrastructure for machine learning, analytics, and application data workloads.
- The guide provides quickstart instructions for running MinIO on bare metal hardware or in containers.
- For Kubernetes environments, the MinIO Kubernetes Operator is recommended.
```

Response from OpenAI API

This method highlights an interesting way to load documents from S3 storage into an LLM using the Langchain framework to process them, while OpenAI’s GPT-3.5 model generates a concise summary and key points of the `MinIO_Quickstart.md` which is fetched from the `play.min.io` server. The use of AI to analyze and condense extensive documentation, provides users with a quick and thorough understanding of essential aspects like installation, server configuration, SDKs and other MinIO features. It showcases the capability of AI in extracting and presenting critical information from comprehensive data sources.

## Loading Documents from MinIO Buckets with Langchain

The integration of MinIO, Langchain and OpenAI offers a compelling toolset for managing large data volumes. While Langchain's S3 loaders, S3DirectoryLoader and S3FileLoader, play an important role in retrieving documents from MinIO buckets, they are solely for loading data into Langchain. These loaders do not perform actions related to uploading data into buckets. For tasks like uploading, modifying or managing bucket policies, the *MinIO Python SDK* is the appropriate tool. This SDK provides a comprehensive set of functionalities for interacting with MinIO storage, including file uploads, bucket management and more. For additional information, please see [Quickstart Guide — MinIO Object Storage for Linux](https://min.io/docs/minio/linux/developers/python/minio-py.html?ref=blog.min.io), [Python Client API Reference — MinIO Object Storage for Linux](https://min.io/docs/minio/linux/developers/python/API.html?ref=blog.min.io).

While Langchain streamlines the process of fetching and processing data using AI models, the heavy lifting of data management within the MinIO buckets is dependent on the MinIO Python SDK. This is an important distinction that must be understood by developers and data engineers building efficient, AI-integrated storage solutions. For a thorough understanding of MinIO's capabilities and how to utilize its Python SDK for various storage operations, refer to MinIO's official [documentation](https://docs.min.io/docs/python-client-api-reference.html?ref=blog.min.io).

By using MinIO object storage as the primary data repository for AI and ML processes, you can simplify your data management pipeline. MinIO excels as a one-stop solution for [storing, managing, and retrieving large datasets](https://blog.min.io/the-architects-guide-to-the-modern-data-stack/), which is crucial for effective AI and ML operations. This streamlined approach reduces complexity and overhead, potentially accelerating insights by ensuring swift access to data.

For those interested in delving deeper into the integration of MinIO with Langchain to enhance LLM tool-use, the article “[Developing Langchain Agents with MinIO SDK for LLM Tool-Use](https://blog.min.io/minio-langchain-tool)” offers a comprehensive exploration of the subject.

Good luck in your development endeavors! We hope [MinIO](https://min.io/download?ref=blog.min.io) continues to play a key role in your AI/ML journey. Reach out to us on [Slack](https://slack.min.io/?ref=blog.min.io) and share your insights and discoveries!