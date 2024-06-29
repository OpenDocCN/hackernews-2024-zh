<!--yml

category: 未分类

date: 2024-05-27 14:39:40

-->

# 新的localllm让您在没有GPU的情况下本地开发通用AI应用 | 谷歌云博客

> 来源：[https://cloud.google.com/blog/products/application-development/new-localllm-lets-you-develop-gen-ai-apps-locally-without-gpus](https://cloud.google.com/blog/products/application-development/new-localllm-lets-you-develop-gen-ai-apps-locally-without-gpus)

在当今快节奏的人工智能领域，开发者在构建使用大语言模型（LLMs）的应用程序时面临诸多挑战。特别是，传统上用于运行LLMs的GPU稀缺性构成了重要障碍。在本篇文章中，我们为您介绍一种新颖的解决方案，允许开发者在本地CPU和内存上利用LLMs的强大功能，正好适用于[Cloud Workstations](https://cloud.google.com/workstations)，谷歌云的完全托管开发环境。我们在本文中使用的模型存储在[Hugging Face](https://huggingface.co/)上，特别来自“The Bloke”的一个repo，并且与用于在CPU或低功耗GPU上运行它们的量化方法兼容。这种创新方法不仅消除了对GPU的需求，还为无缝高效的应用程序开发打开了无限可能。通过使用“量化模型”、Cloud Workstations、一个名为[localllm](https://github.com/googlecloudplatform/localllm)的新开源工具和普遍可用的资源的组合，您可以在配备良好的开发工作站上开发基于人工智能的应用程序，利用现有的流程和工作流程。

### **量化模型 + 云工作站 == 生产力**

量化模型是经过优化以在具有有限计算资源的本地设备上运行的AI模型。这些模型在内存使用和处理能力方面设计更高效，使其能够在智能手机、笔记本电脑和其他边缘设备等设备上平稳运行。在本例中，我们在具有充足资源的Cloud Workstations上运行它们。以下是一些优秀的例子，说明为什么在开发过程中利用量化模型可能会解除您的障碍：

+   **性能提升:** 量化模型经过优化，使用低精度数据类型（如8位整数）进行计算，而不是标准的32位浮点数。这种精度降低可以实现更快的计算速度和在资源有限设备上的改进性能。

+   **减少内存占用:** 量化技术有助于减少AI模型的内存需求。通过使用更少的位数表示权重和激活，模型的整体大小减小，更容易适配存储容量有限的设备。

+   **更快的推断速度**：量化模型由于其较低的精度和更小的模型尺寸，可以更快地执行计算。这使得推断时间更短，使得AI应用能够在本地设备上更加流畅和响应迅速。

将量化模型与Cloud Workstations结合使用，使您可以充分利用Cloud Workstations的灵活性、可扩展性和成本效益。此外，依赖远程服务器或基于云的GPU实例进行基于LLM的应用程序开发的传统方法可能会引入延迟、安全问题和对第三方服务的依赖。在您的Cloud Workstations内部本地使用LLMs的解决方案，不会牺牲性能、安全性或数据控制权，具有很多好处。

## **介绍** **localllm**

今天，我们推出了`localllm`，这是一套工具和库，通过命令行实用程序提供了对HuggingFace的量化模型的便捷访问。对于希望利用LLMs而无需依赖GPU可用性的开发者来说，`localllm`可以是一个改变游戏规则的工具。该代码库提供了在CPU和内存上本地运行LLMs的全面框架和工具，可以在Google Cloud Workstation内部使用此方法（尽管您也可以在本地机器或任何具有足够CPU的地方运行LLM模型）。通过消除对GPU的依赖，您可以释放LLMs在应用开发需求中的全部潜力。

### **主要功能和好处**

**无需GPU的LLM执行**：`localllm`允许您在CPU和内存上执行LLMs，无需稀缺的GPU资源，因此可以将LLMs集成到应用程序开发工作流中，而无需牺牲性能或生产力。

**增强生产力**：通过`localllm`，您可以直接在Google Cloud生态系统内使用LLMs。这种集成简化了开发过程，减少了与远程服务器设置或依赖外部服务相关的复杂性。现在，您可以专注于构建创新应用，而不必管理GPU。  

**成本效益**：通过利用`localllm`，您可以显著降低与GPU配置相关的基础设施成本。在Google Cloud环境中在CPU和内存上运行LLMs的能力，使您可以优化资源利用率，实现成本节约和投资回报率的提升。

**提升数据安全性**：在CPU和内存上运行LLMs有助于保护敏感数据。通过`localllm`，您可以减少数据传输和第三方访问的风险，增强数据安全性和隐私保护。

**与Google Cloud服务的无缝集成**：`localllm`与各种Google Cloud服务集成，包括数据存储、机器学习API或其他Google Cloud服务，因此您可以充分利用Google Cloud生态系统的全部潜力。

### **开始使用** **localllm**

要开始使用`localllm`，请访问GitHub仓库[https://github.com/googlecloudplatform/localllm](https://github.com/googlecloudplatform/localllm)。该仓库提供了详细的文档、代码示例和逐步说明，以便在Google Cloud环境中在CPU和内存上设置和利用LLMs。您可以探索该仓库，为其开发贡献力量，并利用其功能来增强您的应用程序开发工作流程。

一旦您在本地克隆了该仓库，以下简单步骤将运行localllm，使用HuggingFace仓库中您选择的量化模型“The Bloke”，然后执行初始样本提示查询。例如，我们正在使用Llama。
