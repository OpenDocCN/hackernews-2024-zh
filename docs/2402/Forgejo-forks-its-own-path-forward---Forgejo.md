<!--yml

category: 未分类

date: 2024-05-27 14:55:37

-->

# Forgejo在前进中铸造自己的道路 — Forgejo

> 来源：[https://forgejo.org/2024-02-forking-forward/](https://forgejo.org/2024-02-forking-forward/)

自 [创立](../2022-12-15-hello-forgejo/) 以来，Forgejo（一个类似于 GitHub 的自托管 Git Forge）一直是 Gitea 的软分叉。将其升级到此版本 - 至少在当前阶段仍然如此 - 就像 [更改下载发行版的 URL](../download/) 一样简单。随着时间的推移，Forgejo 的治理和开发模式得到了发展。为了能够提供稳定、安全、可靠的发布版本，Forgejo 要求对每一次代码更改都进行 [合理的测试工作](https://codeberg.org/forgejo/governance/src/branch/main/PullRequestsAgreement.md)。这一策略非常成功，它不仅捕捉到了导入代码中的回归问题，还发现了提议更改中的错误。此外，Forgejo 还接受了一些在 Gitea 中不可用的功能和其他更改，并且在其他方面已经 [分叉](#the-hard-forking-process) 了。

今天，Forgejo有许多人参与其主要使命：

> 1.  社区拥有控制权，并确保我们开发以解决社区需求。
> 1.  
> 1.  我们将帮助解放软件开发，摆脱专有工具的枷锁。

为了继续遵循这一声明，于 2024 年初 [做出了决定](https://codeberg.org/forgejo/governance/issues/58) 成为硬分叉。通过这样做，Forgejo 不再受制于 Gitea，并且可以自行前进，允许维护者和贡献者以更高的速度减少技术债务，并实施新功能或错误修复等更改，这些更改否则可能会与 Gitea 中的更改冲突。简而言之，Gitea 和 Forgejo 的治理和开发模式随着时间的推移而分歧，它们的目标也有所不同。成为硬分叉是这种分歧的顶点。

## 硬分叉过程[](#the-hard-forking-process)

Forgejo 自 2022 年末成立以来，一直是 Gitea 的软分叉，这意味着它包含了整个 Gitea 的代码，包括好的和坏的，而 Forgejo 对构建的内容几乎没有控制权。然而，在此之前，Gitea 的某些部分已经被“硬分叉”了：

大部分这些步骤是为了解放代码库的部分内容，从专有解决方案转而使用自由软件来管理，并在这一过程中，使得管理Forgejo特定变更更加简单。同时，尽量减少对软件本身的影响。

## 成为硬分叉的后果[](#consequences-of-becoming-a-hard-fork)

自 Forgejo v1.21 版本开始，Forgejo 包含了所有的 Gitea 功能，并且这样做有利于使 Forgejo 成为一个即插即用的替代品。但是，随着决定进行硬分叉，这一点将不再被保证。在硬分叉时仍然可以从当时发布的最新[Gitea 版本](https://github.com/go-gitea/gitea/releases/tag/v1.21.5)升级，但在此之后的版本将不再有此保证。

因此，如果您考虑升级到 Forgejo，我们建议您尽早行动，因为随着项目自然分歧的进一步加深，升级将变得越来越困难。这不会一夜之间发生，甚至可能不会很快发生，但最终，Forgejo 将停止作为一个即插即用的替代品。

在硬分叉后，Forgejo API 将努力保持与 Gitea API 的兼容性。分叉时存在的 API 是公开的，对它们进行更改将是一个破坏性的变更，必须非常谨慎地评估，不能轻率地进行。未来的 API 同样需要进行评估，Forgejo 将尝试与 Gitea 保持兼容。然而，Forgejo 的贡献者也将根据自己的判断决定是否以及如何实现一个 API，同时考虑先前的目标。
