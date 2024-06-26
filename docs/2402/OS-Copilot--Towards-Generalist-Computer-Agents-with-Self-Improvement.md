<!--yml

category: 未分类

date: 2024-05-27 14:56:14

-->

# OS-Copilot：朝向具有自我改进能力的通用计算机代理

> 来源：[https://os-copilot.github.io/](https://os-copilot.github.io/)

FRIDAY的设计原则旨在通过装备代理能力进行自我精炼和自主学习来最大化通用性。我们首先使用一个示例来说明FRIDAY的操作，并强调其自我精炼的能力。随后，我们深入探讨FRIDAY如何通过自主学习获得控制陌生应用程序的能力。

在上述图中，我们使用一个运行示例来展示FRIDAY在操作系统中的功能。

在收到子任务“将系统切换到暗模式”（步骤①）后，配置跟踪器使用密集检索从长期记忆中回忆相关信息以构建提示（步骤②）。该提示涵盖了相关工具、用户配置文件、操作系统版本和代理的工作目录。

在本例中，没有识别出合适的工具（相似性低于指定的阈值），促使工具生成器激活，为当前子任务设计一个应用程序定制的工具（步骤③）。正如我们从图（b）中可以看到的那样，生成的工具表现为一个Python类，利用AppleScript将系统切换到暗模式。

随后，随着工具的创建和配置提示的最终化，执行器处理提示，生成可执行动作并执行它（步骤④）。如图（b）底部所示，执行器首先将工具代码存储到Python文件中，然后在命令行终端中执行代码。

执行完毕后，评论家评估子任务是否成功完成（步骤⑤）。成功后，评论家使用LLMs对生成的工具进行评分，评分范围从0到10，评分越高表明未来重复使用的潜力越大。在当前实施中，得分高于8的工具通过更新程序内存中的工具存储库（步骤⑦）被保留。

然而，在执行失败的情况下，精炼器收集来自评论家的反馈，并启动负责的动作、工具或子任务的自我修正（步骤⑥）。FRIDAY将迭代步骤④到⑥，直到子任务被视为完成或达到最多三次尝试。

自主学习是人类获取信息和学习新技能的重要能力，在Minecraft游戏中的具体代理中已经显示出有希望的结果。

配备了预定义的学习目标，比如掌握电子表格操作技巧，**FRIDAY** 被促使提出一系列与目标相关的任务，从简单到具有挑战性不等。随后，FRIDAY 按照这一课程表，通过试验和错误解决这些任务，从而在整个过程中积累了宝贵的工具和语义知识。尽管其设计简单，我们的评估表明自主学习对于通用操作系统级代理非常关键。
