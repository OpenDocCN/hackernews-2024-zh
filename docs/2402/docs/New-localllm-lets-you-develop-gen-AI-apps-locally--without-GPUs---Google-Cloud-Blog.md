<!--yml
category: 未分类
date: 2024-05-27 14:39:40
-->

# New localllm lets you develop gen AI apps locally, without GPUs | Google Cloud Blog

> 来源：[https://cloud.google.com/blog/products/application-development/new-localllm-lets-you-develop-gen-ai-apps-locally-without-gpus](https://cloud.google.com/blog/products/application-development/new-localllm-lets-you-develop-gen-ai-apps-locally-without-gpus)

 In today's fast-paced AI landscape, developers face numerous challenges when it comes to building applications that use large language models (LLMs). In particular, the scarcity of GPUs, which are traditionally required for running LLMs, poses a significant hurdle. In this post, we introduce you to a novel solution that allows developers to harness the power of LLMs locally on CPU and memory, right within [Cloud Workstations](https://cloud.google.com/workstations), Google Cloud’s fully managed development environment. The models we use in this walkthrough are located on [Hugging Face](https://huggingface.co/) and are specifically in a repo from “The Bloke” and are compatible with the quantization method used to allow them to run on CPUs or low power GPUs. This innovative approach not only eliminates the need for GPUs but also opens up a world of possibilities for seamless and efficient application development. By using a combination of “quantized models,” Cloud Workstations, a new open-source tool named [localllm](https://github.com/googlecloudplatform/localllm), and generally available resources, you can develop AI-based applications on a well-equipped development workstation, leveraging existing processes and workflows.

### **Quantized models + Cloud Workstations == Productivity**

Quantized models are AI models that have been optimized to run on local devices with limited computational resources. These models are designed to be more efficient in terms of memory usage and processing power, allowing them to run smoothly on devices such as smartphones, laptops, and other edge devices. In this case, we are running them on Cloud Workstations with ample available resources. Here are some great examples of why leveraging quantized models in your development loop may unblock your efforts:

*   **Improved performance:** Quantized models are optimized to perform computations using lower-precision data types such as 8-bit integers, instead of standard 32-bit floating-point numbers. This reduction in precision allows for faster computations and improved performance on devices with limited resources.

*   **Reduced memory footprint:** Quantization techniques help reduce the memory requirements of AI models. By representing weights and activations with fewer bits, the overall size of the model is reduced, making it easier to fit on devices with limited storage capacity. 

*   **Faster inference:** Quantized models can perform computations more quickly due to their reduced precision and smaller model size. This enables faster inference times, allowing AI applications to run more smoothly and responsively on local devices.

Combining quantized models with Cloud Workstations allows you to take advantage of the flexibility, scalability and cost effectiveness of Cloud Workstations. Moreover, the traditional approach of relying on remote servers or cloud-based GPU instances for LLM-based application development can introduce latency, security concerns, and dependency on third-party services. A solution that lets you leverage LLMs locally, within your Cloud Workstations, without compromising performance, security, or control over your data, can have a lot of benefits.

## **Introducing** **localllm**

Today, we’re introducing `localllm`, a set of tools and libraries that provides easy access to quantized models from HuggingFace through a command-line utility. `localllm` can be a game-changer for developers seeking to leverage LLMs without the constraints of GPU availability. This repository provides a comprehensive framework and tools to run LLMs locally on CPU and memory, right within the Google Cloud Workstation, using this method (though you can also run LLM models on your local machine or anywhere with sufficient CPU). By eliminating the dependency on GPUs, you can unlock the full potential of LLMs for your application development needs.

### **Key features and benefits**

**GPU-free LLM execution**: `localllm` lets you execute LLMs on CPU and memory, removing the need for scarce GPU resources, so you can integrate LLMs into your application development workflows, without compromising performance or productivity.

**Enhanced productivity**: With `localllm`, you use LLMs directly within the Google Cloud ecosystem. This integration streamlines the development process, reducing the complexities associated with remote server setups or reliance on external services. Now, you can focus on building innovative applications without managing GPUs.

**Cost efficiency**: By leveraging `localllm`, you can significantly reduce infrastructure costs associated with GPU provisioning. The ability to run LLMs on CPU and memory within the Google Cloud environment lets you optimize resource utilization, resulting in cost savings and improved return on investment.

**Improved data security**: Running LLMs locally on CPU and memory helps keep sensitive data within your control. With `localllm`, you can mitigate the risks associated with data transfer and third-party access, enhancing data security and privacy.

**Seamless integration with Google Cloud services**: `localllm` integrates with various Google Cloud services, including data storage, machine learning APIs, or other Google Cloud services, so you can leverage the full potential of the Google Cloud ecosystem.

### **Getting started with** **localllm**

To get started with the `localllm`, visit the GitHub repository at [https://github.com/googlecloudplatform/localllm](https://github.com/googlecloudplatform/localllm). The repository provides detailed documentation, code samples, and step-by-step instructions to set up and utilize LLMs locally on CPU and memory within the Google Cloud environment. You can explore the repository, contribute to its development, and leverage its capabilities to enhance your application development workflows. 

Once you’ve cloned the repo locally, the following simple steps will run localllm with a quantized model of your choice from the HuggingFace repo “The Bloke,” then execute an initial sample prompt query. For example we are using Llama.