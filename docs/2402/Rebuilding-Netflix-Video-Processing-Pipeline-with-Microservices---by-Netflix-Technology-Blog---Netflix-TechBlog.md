<!--yml

类别：未分类

日期：2024-05-27 14:44:12

-->

# 用微服务重建Netflix视频处理管道 | Netflix技术博客 | Netflix TechBlog

> 来源：[https://netflixtechblog.com/rebuilding-netflix-video-processing-pipeline-with-microservices-4e5e6310e359?gi=d48b3333df75](https://netflixtechblog.com/rebuilding-netflix-video-processing-pipeline-with-microservices-4e5e6310e359?gi=d48b3333df75)

# ***用微服务重建Netflix视频处理管道***

[郭力伟](https://www.linkedin.com/in/liwei-guo-a5aa6311/)，[Anush Moorthy](https://www.linkedin.com/in/anush-moorthy-b8451142/)，[陈立衡](https://www.linkedin.com/in/li-heng-chen-a75458a2/)，[Vinicius Carvalho](https://www.linkedin.com/in/carvalhovinicius/)，[Aditya Mavlankar](https://www.linkedin.com/in/aditya-mavlankar-7139791/)，[Agata Opalach](https://www.linkedin.com/in/agataopalach/)，[Adithya Prakash](https://www.linkedin.com/in/adithyaprakash/)，Kyle Swanson，[Jessica Tweneboah](https://www.linkedin.com/in/jessicatweneboah/)，[Subbu Venkatrav](https://www.linkedin.com/in/subbu-venkatrav-126172a/)，[朱立山](https://www.linkedin.com/in/lishan-z-51302abb/)

*这是Netflix如何通过微服务重建其视频处理管道的多部分系列博客的第一篇，以便我们能够保持快速创新的步伐，并持续改进会员流媒体和工作室运营的系统。本介绍性博客重点介绍了我们的旅程概述。未来的博客将深入探讨每个服务，并分享这一过程中的见解和经验。*

Netflix 视频处理管道是在 2007 年我们的流媒体服务上线时开始运行的。自那时起，视频管道经历了重大改进和广泛扩展：

+   从标准动态范围（SDR）开始，我们扩展了编码管道到4K和高动态范围（HDR），从而支持我们的高级服务。

+   我们从集中式线性编码转向了[分布式基于块的编码](/high-quality-video-encoding-at-scale-d159db052746)。这种架构转变大大降低了处理延迟，增加了系统的弹性。

+   放弃使用数量受限的专用实例，我们利用了Netflix的[内部波动](/creating-your-own-ec2-spot-market-6dd001875f5)，这是由于自动缩放微服务而创建的，显著提升了计算弹性以及资源利用效率。

+   我们推出了编码创新，例如[按片头优化](https://medium.com/netflix-techblog/per-title-encode-optimization-7e99442b62a2)和[按镜头优化](/optimized-shot-based-encodes-now-streaming-4b9464204830)，这显著提升了Netflix会员的体验质量（QoE）。

+   通过与工作室内容系统集成，我们使管道能够利用创意方面的丰富元数据，并创建更引人入胜的会员体验，如[互动叙事](https://en.wikipedia.org/wiki/Black_Mirror:_Bandersnatch)。

+   我们扩展了管道支持，以满足我们工作室/内容开发的使用场景，这些场景与传统的流媒体使用案例相比，具有不同的延迟和可靠性要求。

在过去十五年的经验强化了我们的信念，即一个高效灵活的视频处理管道对于Netflix继续成功至关重要，允许我们创新并支持我们的流媒体服务以及工作室合作伙伴。为此，编码技术部门的视频和图像编码团队（ET）在我们的下一代基于微服务的计算平台[Cosmos](/the-netflix-cosmos-platform-35c14d9351ad)上花费了最近几年时间重建视频处理管道。

# 从重装到Cosmos

## 重装

从2014年开始，我们在我们的第三代平台[重装](https://www.youtube.com/watch?v=JouA10QJiNc)上开发和运行视频处理管道。重装架构良好，提供了良好的稳定性、可伸缩性和合理的灵活性。它作为我们团队开发的许多编码创新的基础。

当设计重装时，我们专注于一个用例：将从工作室接收到的高质量媒体文件（也称为mezzanines）转换为Netflix流媒体的压缩资产。重装被创建为一个单一的单体系统，ET各媒体团队以及我们的平台合作伙伴团队内容基础设施和解决方案（CIS）¹在同一代码库中工作，构建一个处理所有媒体资产的单一系统。多年来，该系统扩展以支持各种新的用例。这导致系统复杂性显著增加，并且重装的限制开始显现：

+   *耦合功能*：重装由多个工作模块和一个编排模块组成。设置新的重装模块并将其与编排集成需要大量的努力，这导致在开发新功能时更偏向于增强而非创造。例如，在重装中，[视频质量计算是在视频编码器模块内部实现的](/netflix-video-quality-at-scale-with-cosmos-microservices-552be631c113)。通过这种实现方式，重新计算视频质量而无需重新编码是极其困难的。

+   *单体结构*：由于重装模块通常在同一代码库中共存，容易忽视代码隔离规则，并且存在跨应该强制隔离的边界不小心重复使用代码的情况。这种重复使用导致了紧耦合，并降低了开发速度。模块之间的紧耦合进一步迫使我们将所有模块一起部署。

+   *长发布周期*：联合部署意味着增加了意外生产停机的风险，因为这样大规模的部署在调试和回滚时可能会很困难。这推动了“发布列车”的方法。每两周，所有模块的“快照”被取出，并提升为“发布候选”。然后，这个发布候选经历了详尽的测试，试图覆盖尽可能广泛的面积。这个测试阶段大约需要两周时间。因此，根据代码变更合并的时间，可能需要2到4周时间才能到达生产环境。

随着时间的推移和功能的增长，Reloaded中新功能贡献的速率下降了。由于克服架构限制需要的工作量过大，几个有前途的想法被放弃了。曾经为我们服务良好的平台现在正在成为开发的负担。

## 宇宙

作为回应，2018年CIS和ET团队开始开发下一代平台Cosmos。除了Reloaded已经享有的可伸缩性和稳定性外，Cosmos旨在显著增加系统的灵活性和功能开发速度。为了实现这一目标，Cosmos被开发为面向工作流驱动的、以媒体为中心的微服务计算平台。

微服务架构提供了服务之间的强解耦。每个微服务的工作流支持减轻了实现复杂媒体工作流逻辑的负担。最后，相关的抽象允许媒体算法开发者专注于视频和音频信号的处理，而不是基础设施问题。Cosmos提供的一系列优势可以在链接的[博客](/the-netflix-cosmos-platform-35c14d9351ad)中找到。

# 在Cosmos中构建视频处理流水线

## 服务边界

在微服务架构中，系统由许多细粒度服务组成，每个服务专注于单一功能。因此，第一步（可能也是最重要的一步）是识别边界并定义服务。

在我们的流水线中，随着媒体资产从创建到摄入再到交付的过程中，它们经历了许多处理步骤，如分析和转换。我们分析了这些处理步骤，以确定“边界”，并将它们分组到不同的领域中，这些领域又成为我们设计的微服务的构建块。

例如，在Reloaded中，视频编码模块包含了5个步骤：

1\. 将输入视频分割成小块

2\. 独立对每个块进行编码

3\. 计算每个块的质量分数（[VMAF](/vmaf-the-journey-continues-44b51ee9ed12)）

4\. 将所有编码块组装成单个编码视频

5\. 聚合所有块的质量分数

从系统角度来看，组装后的编码视频是主要关注的内容，而内部分块和单独的分块编码存在是为了满足某些延迟和弹性需求。此外，正如上文所述，视频质量计算与编码服务提供的功能完全分离。

因此，在Cosmos中，我们创建了两个独立的微服务：视频编码服务（VES）和[视频质量服务（VQS）](/netflix-video-quality-at-scale-with-cosmos-microservices-552be631c113)，每个服务都提供清晰的、解耦的功能。在实施细节上，分块编码和组装被抽象为VES中的一部分。

## 视频服务

上述方法被应用于视频处理管道的其余部分，以识别功能，并因此创建以下视频服务²。

1.  Video Inspection Service (VIS): 此服务以中间介质作为输入，并执行各种检查。它从中间介质的不同层提取元数据，以供下游服务使用。此外，检查服务在观察到无效或意外元数据时标记问题，并向上游团队提供可操作的反馈。

1.  Complexity Analysis Service (CAS): 最佳编码配方高度依赖于内容。此服务以中间介质为输入，并执行分析以了解内容复杂性。它调用视频编码服务进行[预编码](/dynamic-optimizer-a-perceptual-video-encoding-optimization-framework-e19f1e3a277f)，并调用视频质量服务进行质量评估。结果保存到数据库中以便重复使用。

1.  Ladder Generation Service (LGS): 此服务为给定的编码家族（H.264、AV1等）创建整个比特率阶梯。它从CAS获取复杂性数据，并运行优化算法以创建编码配方。CAS和LGS涵盖了我们先前在技术博客中提出的大部分创新（[按标题](http://techblog.netflix.com/2015/12/per-title-encode-optimization.html)，[移动编码](http://techblog.netflix.com/2016/12/more-efficient-mobile-encodes-for.html)，[按镜头](/optimized-shot-based-encodes-now-streaming-4b9464204830)，[优化的4K编码](/optimized-shot-based-encodes-for-4k-now-streaming-47b516b10bbb)等）。通过将阶梯生成封装到一个独立的微服务（LGS）中，我们将阶梯优化算法与复杂性分析数据的创建和管理（驻留在CAS中）解耦。我们期望这将为我们提供更大的实验自由度和更快的创新速度。

1.  Video Encoding Service (VES): 此服务以中间介质和编码配方作为输入，并创建编码视频。配方包括所需的编码格式和输出属性，如分辨率、比特率等。该服务还提供选项，允许根据使用情况进行延迟、吞吐量等的微调。

1.  视频验证服务（VVS）：该服务接收编码视频和编码预期清单作为输入。这些预期包括编码配方中指定的属性以及编解码器规范中的一致性要求。VVS 分析编码视频，并将结果与指定的预期进行比较。任何差异都会在响应中标记，以提醒调用方。

1.  [视频质量服务（VQS）](/netflix-video-quality-at-scale-with-cosmos-microservices-552be631c113)：该服务以中介格式和编码视频作为输入，计算编码视频的质量分数（VMAF）。

## 服务编排

每个视频服务都提供专门的功能，并且它们协同工作以生成所需的视频资产。目前，Netflix 视频流水线的两个主要用例是为会员流媒体和制片厂运营生成资产。针对每个用例，我们创建了专用的工作流编排器，以便可以根据对应的业务需求进行定制化的服务编排。

对于流媒体用例，生成的视频部署到我们的内容传输网络（CDN），供 Netflix 会员消费。这些视频可能会被观看数百万次。流媒体工作流编排器利用几乎所有视频服务，为会员提供卓越的观看体验。它利用 VIS 检测和拒绝不符合规范或低质量的中介格式，调用 LGS 进行编码配方优化，使用 VES 编码视频，并调用 VQS 进行质量测量，质量数据进一步用于 Netflix 的数据管道进行分析和监控。除视频服务外，流媒体工作流编排器还使用音频和定时文本服务生成音频和文本资产，以及使用打包服务将资产“容器化”供流媒体使用。

对于制片厂用例，一些示例视频资产包括营销片段和每日制作编辑代理。来自制片方的请求通常对延迟敏感。例如，制作团队的某人可能正在等待视频进行审阅，以便决定第二天的拍摄计划。因此，制片工作流编排器优化以实现快速周转，并专注于核心媒体处理服务。此时，制片工作流编排器调用 VIS 提取摄入资产的元数据，并使用预定义的配方调用 VES。与会员流媒体相比，制片运营对视频处理有不同且独特的需求。因此，制片工作流编排器独占一些编码功能，如取证水印和时间码/文本烧录。
