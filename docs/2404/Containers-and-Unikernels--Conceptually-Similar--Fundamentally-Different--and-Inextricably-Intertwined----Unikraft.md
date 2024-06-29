<!--yml

category: 未分类

date: 2024-05-27 12:54:12

-->

# Containers and Unikernels: 概念上相似，但根本上不同，紧密交织在一起。 — Unikraft

> 来源：[https://unikraft.io/blog/containers-and-unikernels/](https://unikraft.io/blog/containers-and-unikernels/)

在阅读比较unikernels和容器的文章时，我认为Unikraft，我们将容器视为我们的一部分。我们当然是unikernel公司，这似乎毫无意义。为什么一个开发工具来构建和运行轻量级VM作为容器替代品的unikernel公司会支持使用容器？这是疯狂的，这肯定是亵渎！

容器和unikernel的景观自它们第一次被放在同一个句子里一起提及以来，已经发生了很大变化。事实上，我认为现在不再正确地说它们是两种不同的计算单元的意识形态。这是因为在过去的5-10年中，“容器”和“unikernel”都有了显著的发展。这两个术语现在比以前更加有实际意义，并且这两种技术领域有非常好的特性可以很好地结合在一起。让我们来探讨一下。

## 从混乱到物质崇拜

无论是Docker本身还是它鞋子中的另一个工具，容器在2010年代初期的到来都源于开发人员在容器之前时期面临的几个挑战：

+   **环境不一致**：应用程序在不同的操作系统及其配置下表现不同，这使得开发、测试和部署过程变得令人沮丧。

+   **虚拟机（VMs）笨重**：当时主流的应用程序隔离技术VMs，资源密集且启动缓慢。这阻碍了开发工作流程和部署的灵活性。

+   **依赖管理的头痛**：在不同环境中管理应用程序依赖关系是一个复杂且容易出错的任务。开发人员需要一种方法来确保他们的应用程序能够正确运行所需的一切。

Docker的旅程始于2008年，事实上，在解决这些问题时它并不孤单。当时，它已经依赖于许多现有的基础技术，包括：当然，大家最喜欢的`chroot`，较少人知的[Capsicum](https://www.usenix.org/legacy/event/sec10/tech/full_papers/Watson.pdf)，以及LXC（Linux Containers）的兴起，等等。LXC提供了在Linux系统上隔离进程的方法，并且是迈向今天基于容器的解决方案的第一步之一，但它缺乏一个用户友好的生态系统，并且，正如其名称所示，仅限于Linux。

2013年Docker登场。

基于LXC构建，Docker引入了一个完整的容器管理系统，可以说也是最重要的贡献之一：*容器镜像*。

## `FROM` 爱意满满

最终，Docker的最初使命是在日益多样化的执行环境中实现一致性，摒弃“在我的机器上可以运行”的方式，并使处理这个问题比当时更加简单直接。因此，Docker因其工具化而受到赞扬，我们和许多其他人在我们自己的项目中也秉持这一信条：通过工具化使复杂问题更易解决，尤其是那些文档充分、稳定且易用的工具。这正是UNIX哲学的回响。

此外，除了其即时的命令行界面之外，还有三个关键的创新，这些创新在容器社区中由Docker以及许多其他人显著发展，我们在Unikraft中也采用了这些创新：文件集合的定义、存储和传输：

1.  `Dockerfile`是解决不一致环境问题的一个非常简单且极其优雅的方法。通过使用强制的基本语法，并允许在现有文件系统之间进行交叉引用（`FROM`和`COPY`/`ADD`），定义文件系统的蓝图变得不仅容易，而且是可扩展的。

1.  通过为结果文件系统本身引入丰富的元数据，“镜像”（描述诸如其他文件或块等内容的一系列JSON文件）以及对文件系统每次变更（覆盖）的每个事务，“层”，最终文件系统的整体构成变得更加实用。这使得引入了有趣的分发和缓存机制，以及更为知情的查找。

虽然unikernels源于安全领域，而容器则源于可重现性（通过隔离实现），但如今，容器和unikernels仍然共享一个核心原则：软件封装。两者都为应用程序创建了隔离的环境。容器通过共享主机内核但隔离应用程序的进程、库和文件来实现。而unikernels则更进一步。它们将应用程序代码与最小化预配置的内核捆绑在一起，完全消除了对完整主机内核的需求。因此，unikernels从更强的保护域（硬件）中受益，并且可以消除内部的保护/权限检查功能，从而提高整体应用程序的性能。

由于引入了[Open-Container Initiative’s image specification (OCI image spec)](https://github.com/opencontainers/image-spec)格式，定义和分发文件系统变得轻而易举。（实际上，OCI镜像和[distribution spec](https://github.com/opencontainers/distribution-spec)不仅仅使存储和传输应用程序及其文件系统变得更加容易！首先是通过DockerHub，现在是通过开源CNCF项目如Harbor和产品如GHCR和Artifact Registry，OCI格式已经成为该行业中分发应用程序的普遍标准。）

## 解开互操作性之谜

在这种标准化背景下，开发和使用一个新格式最终旨在服务相同目的是毫无意义的：定义、存储和传输代表应用程序的文件。在Unikraft，我们正是出于这个目的，拥抱了OCI镜像规范格式：既能够分发unikernels，也能帮助定义它们。

要理解我们如何使用OCI镜像格式，我们首先应考虑unikernel镜像可以通过两种方式构建。实际上可以通过两种方式完成：

1.  通过将应用程序原生编译为一个库操作系统，最终生成一个能够独立引导和执行的单一二进制镜像；或者，

1.  作为文件系统的一部分读取预编译的[ELF二进制文件](https://en.wikipedia.org/wiki/Executable_and_Linkable_Format)并在unikernel之上执行它们。

Unikraft支持这两种方法，并更普遍地使用第二种方法，因为它允许与OCI镜像规范实现互操作性。通过将生成的文件系统展平为单一树，并提取`ENTRYPOINT`程序并在顶部执行它，Unikraft Unikernels简单地扩展了规范，提供了表示unikernel的附加文件。

为了实现这一目标，可以通过在`Kraftfile`的`rootfs`元素中提供`Dockerfile`生成的结果文件，并在您的仓库中使用它来构建unikernel：

```
 cmd: ["/usr/bin/my-program"] 
```

（[*更多示例请参见此处*](https://github.com/kraftcloud/examples).)

## 容器（镜像）与Unikernels一起

在使用这两种技术时，Unikraft的重点是使互操作性成为双向街。尽可能利用现有基础设施，以使unikernels更易接近。

当然，我们并未止步于仅读取镜像。OCI镜像规范允许以层级方式声明工件。最顶层的元素是一个“索引”，其中包含一个“清单”列表。每个清单表示一个“镜像”，其中包括应用文件系统……和unikernel！

由于“index”，可以将具有不同特性的应用程序打包在一起。这在传统OCI容器镜像中用于区分不同的硬件架构更为常见。然而，由于Unikraft可以针对不同的架构以及不同的虚拟机监视器（VMMs）和hypervisors，我们欢迎地在OCI镜像规范中采用了这种能力，并允许将unikernels打包到这种格式中。

为了证明这一点，可以使用[Unikraft和KraftCloud的开源命令行客户端，`kraft`](https://github.com/unikraft/kraftkit)，您可以查看同一图像在不同平台上的所有变体：

```
 kraft pkg info -u unikraft.org/helloworld:latest 
```

```
 TYPE  NAME                     VERSION  FORMAT  MANIFEST  INDEX    PLAT               SIZE

app   unikraft.org/helloworld  latest   oci     6ccbe82   76f2074  linuxu/arm64       138 kB

app   unikraft.org/helloworld  latest   oci     cc9e88c   76f2074  qemu/arm64         191 kB

app   unikraft.org/helloworld  latest   oci     fc7587a   76f2074  linuxu/x86_64      93 kB

app   unikraft.org/helloworld  latest   oci     4aacacc   76f2074  xen/x86_64         143 kB

app   unikraft.org/helloworld  latest   oci     28218e9   76f2074  qemu/x86_64        225 kB

app   unikraft.org/helloworld  latest   oci     4eb44f4   76f2074  fc/x86_64          225 kB

app   unikraft.org/helloworld  latest   oci     38ac2b6   76f2074  kraftcloud/x86_64  229 kB 
```

（*这些“Hello, world!”图像是我们的[开源社区目录](http://github.com/unikraft/catalog)的一部分。）

通过采用OCI格式，Unikraft开发的技术可以根据架构、支持的VMM和hypervisor，以及特定内核功能的选择进行适当分发和选择。这是因为我们也将KConfig选项嵌入到[`os.features`](https://github.com/opencontainers/image-spec/blob/main/config.md#properties)列表中。

这最棒的部分？能够无缝使用您现有的基础设施。

## 容器只是开始

在Unikraft，我们建立了KraftCloud，这是一种基于毫秒级语义的新一代云平台，我们使得可以在Linux上实际运行容器的开销的情况下使用现有容器镜像成为可能。由于Unikraft技术的性能和安全性，容器革命的下一阶段已经到来。

了解KraftCloud如何支持各种基于`Dockerfile`的编程语言，框架和应用程序，可以查看我们平台的[应用指南](https://docs.kraft.cloud/guides/)或[示例存储库](https://github.com/kraftcloud/examples)。

我们不断改进这一点，并且有兴趣了解您的工作方式以及您正在解决的问题。如果您没有看到您喜欢的应用程序，框架，语言或用例，请与我们联系，我们会考虑添加它。
