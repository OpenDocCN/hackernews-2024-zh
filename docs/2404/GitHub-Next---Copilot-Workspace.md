<!--yml

category: 未分类

date: 2024-05-27 13:37:57

-->

# GitHub Next | Copilot Workspace

> 来源：[https://githubnext.com/projects/copilot-workspace](https://githubnext.com/projects/copilot-workspace)

Copilot Workspace 提供了开发者两个重要时刻，可以通过自然语言引导系统：通过修改规范和修改计划。

当您提示 Copilot Workspace 进行更改时，它首先会阅读您的代码库并生成一个规范。该规范由两个项目符号列表组成：一个用于代码库当前状态，另一个用于期望状态。您可以编辑这两个列表，以更正系统对当前代码库的理解，或者完善对期望状态的要求。

一旦您对规范满意，Copilot Workspace 将生成一个具体的执行更改计划。在此步骤中，它将列出它打算创建、修改或删除的每个文件，以及每个文件中要执行的操作的项目符号列表。再次强调，您可以编辑计划的任何部分 —— 修改具体的指令，甚至更改将受影响的文件列表。

最终生成的差异本身是可以直接编辑的，因此在创建拉取请求或将代码推送到分支之前，您可以微调代码。

如果您对建议的更改不满意，您不必从头开始。只需在任何时候编辑规范或计划，Copilot Workspace 将使用您的新输入重新生成所有下游步骤。
