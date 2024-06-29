<!--yml

category: 未分类

date: 2024-05-29 13:28:11

-->

# Modular：宣布 MAX 开发者版预览

> 来源：[https://www.modular.com/blog/announcing-max-developer-edition-preview](https://www.modular.com/blog/announcing-max-developer-edition-preview)

Modular 是基于[愿景](https://www.modular.com/vision)成立的，旨在使任何人任何地方都能使用AI。我们始终认为，要实现这一愿景，我们首先必须修复今天构建AI基础设施的分散和不连贯性。正如我们[两年前所说](https://www.modular.com/blog/the-case-for-a-next-generation-ai-developer-platform)，我们想象一个不同的AI软件未来，现在比以往任何时候更为真实：

> 我们的成功意味着全球开发者将通过使用真正可用、便携和可扩展的AI软件而得到赋能。一个世界，其中没有无限预算或接触顶级人才的开发者可以和世界上最大的科技巨头一样高效；一个AI硬件效率和总体拥有成本被优化的世界，组织可以轻松地为他们的用例插入定制ASICs；一个将部署到边缘设备与部署到服务器一样简单的世界，组织可以使用最适合他们需求的任何AI框架，以及AI程序可以在各种硬件上无缝扩展，从而将最新的AI研究部署到生产环境中“毫无障碍”。

**今天，我们非常激动地迈向更加光明的未来，宣布 Modula Accelerated Xecution (MAX) 平台** [**现在全球可用**](https://www.modular.com/max)，首先在 Linux 系统上提供预览。MAX 是一组统一的工具和库，解锁了您的AI推理管道的性能、可编程性和可移植性。MAX 的每个组件都旨在简化将模型部署到任何硬件的过程，为您的工作负载提供最佳的成本性能比。

### **MAX 包含什么？**

**MAX 被设计用来满足AI部署的严苛需求，为您提供部署低延迟、高吞吐量、实时推理管道所需的一切**。MAX 的首个发布版本包含三个关键组件：

+   [**MAX Engine**](https://www.modular.com/max/engine)：一款支持PyTorch、TensorFlow和ONNX模型的最先进AI编译器和运行时系统，如[Mistral](https://performance.modular.com/?task=language&model=mistral-7b-v0.1&instance=c7g.16xlarge&framework=stock-pytorch&view=chart)，[Stable Diffusion](https://performance.modular.com/?task=computer_vision&model=stable-diffusion-v1.5-unet&instance=c5.9xlarge&framework=stock-pytorch&view=chart)，[Llama2](https://performance.modular.com/?task=language&model=llama2-7B-MS-context-encoding&instance=c6a.16xlarge&framework=stock-pytorch&view=chart)，[WavLM](https://performance.modular.com/?task=language&model=wavlm-large&instance=c6i.4xlarge&framework=stock-pytorch&view=chart)，[DLMR](https://performance.modular.com/?task=recommenders&model=dlrm-rm2&instance=c5a.8xlarge&framework=stock-pytorch&view=chart)，[ClipVit](https://performance.modular.com/?task=computer_vision&model=clip-vit-large-patch14&instance=c6i.4xlarge&framework=stock-pytorch&view=chart)等，提供跨多种硬件平台的无与伦比的推理速度，我们仅仅是刚刚开始。

+   [**MAX Serving**](https://www.modular.com/max/serving)**：** MAX引擎的高效服务包装器，确保与当前的AI服务系统（如NVIDIA Triton）无缝互操作，并平稳部署到容器生态系统（如Kubernetes）。

+   [**Mojo🔥**](https://www.modular.com/max/mojo)**：** 世界上第一个从头开始为AI开发构建的编程语言，具有前沿的编译器技术，在任何硬件上都能提供无与伦比的性能和可编程性。

MAX 给开发者带来超凡的力量，为他们提供了一系列工具和库，用于构建高性能的AI应用程序，可以高效地部署在多个硬件平台上。

### **如何使用MAX？**

MAX 是为了满足当前AI开发者的需求而构建的。它为你现有的模型提供即时的性能提升，并与你现有的工具链和服务基础设施无缝集成，通过最小的代码更改捕获MAX的即时价值和性能改进。然后，当你准备好时，你可以扩展MAX以支持AI流水线的其他部分，实现更多的性能、可编程性和可移植性。

#### **快速性能和可移植性优势**

**MAX 编译并在各种硬件上运行您现有的AI模型，提供即时的性能增益**。使用我们的Python或C API来开始使用MAX引擎推理调用，替换您当前的PyTorch、TensorFlow或ONNX推理调用就这么简单。

仅改变几行代码，[您的模型执行速度可提高到5倍](https://performance.modular.com/?task=recommenders&model=dlrm-rm1-multihot&instance=c7g.4xlarge&framework=stock-pytorch&view=chart)，可以极大地减少计算成本，从而使您能够生产更大的模型，同时提高效率，显著降低与标准PyTorch、TensorFlow或ONNX Runtime相比的计算成本。并且MAX引擎在广泛的CPU架构（英特尔、AMD、ARM）之间提供了全面的可移植性，GPU支持即将推出。这意味着您可以无缝地将您的模型移动到最适合您使用情况的硬件上，而无需重写你的模型。

此外，你可以将 MAX 引擎与MAX Serving一起作为 NVIDIA Triton推理服务器的后端进行生产级部署。Triton提供gRPC和HTTP端点，高效处理传入的输入请求。

#### **扩展和优化您的管道**

**除了使用MAX引擎执行现有模型外，你还可以借助 MAX 独特的可扩展性和可编程能力进一步优化性能**。MAX引擎是使用Mojo构建的，并且在Mojo中完全可扩展。如今，MAX图形API使你能够在Mojo中构建整个推理模型， consagenbg流行的“点解决方案”推理框架（如llama.cpp）提供的复杂性 - 提供更好的灵活性和性能。 [即将推出](https://docs.modular.com/max/roadmap)，您甚至将能够编写自定义操作，MAX引擎编译器将这些操作原生融入到您现有的模型图中。

除了使用MAX引擎进行模型优化外，你还可以通过将你的数据预处理/后处理代码和应用程序代码迁移到Mojo，并使用MAX引擎Mojo API的原生Mojo，进一步加速您的AI管道的其余部分。我们的北极星是为您提供大量的杠杆作用，让您能够更快地将AI创新引入您的产品中。

#### **MAX 是如何工作的？**

MAX的基础是[Mojo](https://www.modular.com/max/mojo)，这是一种统一所有AI硬件的通用编程模型，旨在统一整个AI堆栈，从图形内核到应用程序层，并提供一种可移植的替代Python、C和CUDA。

[MAX引擎](https://www.modular.com/max/engine)透明地使用Mojo加速您的AI模型在各种AI硬件上运行，通过先进的编译器和运行时堆栈。用户可以使用MAX引擎的Python和C绑定加载和优化来自PyTorch、TensorFlow和ONNX的现有模型，或使用Mojo图形API构建自己的推理图。此外，我们已经让使用[MAX Serving](https://www.modular.com/max/serving)能够快速集成和支持您现有的推理服务管道变得容易。

由于MAX的内部是在Mojo中构建的，因此您始终可以编写原生Mojo进一步优化您的余下AI管道，包括大大加速您的前处理和后处理逻辑，并最终将Python从您的生产AI服务系统中移除。

### **更好的开发者体验**

除了MAX平台的预览版本，我们还很高兴宣布对MAX开发者体验的许多增强。这些包括:

+   [**MAX示例存储库**](https://github.com/modularml/max/tree/main/examples): MAX GitHub存储库提供了丰富的示例，展示了如何在实践中使用MAX。这包括使用MAX引擎Python和C API的PyTorch和TensorFlow模型推断示例，以及使用Graph API开发的llama2.**🔥**模型。

+   [**新文档站点**](https://docs.modular.com/): 从头重建，我们的新文档站点具有极快的搜索速度，更好的分类和结构，更丰富的代码示例，以及增强的开发者可用性。更不用说为所有新的MAX功能提供的新教程了！

+   [**Mojo的新“Playground”体验**](https://docs.modular.com/mojo/playground): 我们的新Mojo编码沙盒非常适合早期学习和快速实验。您可以在几秒钟内开始执行Mojo代码，并通过Gist轻松与朋友分享。

+   [**新开发者仪表板**](https://developer.modular.com/dashboard): 我们已经进行了全面改进和更新我们的开发者仪表板，使您能够很快访问更新内容、性能仪表板，并很快获得我们的MAX企业功能。

+   **… 以及更多内容**: 几乎每个方面都进行了产品改进和修复，我们将继续快速迭代，为全球开发者提供世界级的AI基础设施。

## **更多精彩内容即将推出！**

今天在[**MAX开发者版预览**](https://www.modular.com/max)中全部内容均已提供。我们很高兴推出此版本，让MAX全球化，并期待您用它构建的项目。虽然这只是我们MAX的第一个预览版本，但我们在不久的将来有许多后续版本的计划：包括Mac支持、企业功能和GPU支持。请查看我们的[路线图](https://docs.modular.com/max/roadmap)获取更多信息。

要了解更多关于MAX的信息，并参与Modular社区：

我们知道这只是一个开始，我们很高兴推出此版本，并期待听取您的反馈，以便我们快速迭代改进MAX，为世界提供帮助推动AI前进，并使AI能够被任何人在任何地方使用 - 加入我们！

前进！

Modular团队
