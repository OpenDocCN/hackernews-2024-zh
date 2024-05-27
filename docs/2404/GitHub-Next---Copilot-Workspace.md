<!--yml
category: 未分类
date: 2024-05-27 13:37:57
-->

# GitHub Next | Copilot Workspace

> 来源：[https://githubnext.com/projects/copilot-workspace](https://githubnext.com/projects/copilot-workspace)

Copilot Workspace offers developers two important moments where they can steer the system via natural language: by altering the specification, and by altering the plan.

When you prompt Copilot Workspace to make a change, it start by reading your codebase and generating a specification. The spec is comprised of two bullet-point lists: one for the current state of the codebase, and one for the desired state. You can edit both lists, either to correct the system’s understanding of the current codebase, or to refine the requirements for the desired state.

Once you’re happy with the spec, Copilot Workspace will generate a concrete plan for executing the changes. In this step, it will list out every file it intends to create, modify, or delete, as well as a bullet-point list of the actions to be taken in each file. Once again, you can edit any part of this plan — altering the specific instructions, or even changing the list of files that will be affected.

Finally, the generated diff is itself directly editable, so you can tweak the code before creating a pull request or pushing the code to a branch.

If you aren’t satisfied with the proposed changes, you don’t have to start over from scratch. Simply edit the spec or plan at any point, and Copilot Workspace will regenerate all the downstream steps with your new input.