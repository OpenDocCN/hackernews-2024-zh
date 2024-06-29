<!--yml

category: 未分类

date: 2024-05-29 12:41:14

-->

# Podman 5.0 已经发布！

> 来源：[https://blog.podman.io/2024/03/podman-5-0-has-been-released/](https://blog.podman.io/2024/03/podman-5-0-has-been-released/)

Podman 版本 5.0.0 已发布！这是我们两年来的首个重大版本，包括了几个新功能和重大变更。Podman 版本 5.0.0 对于 Windows 和 Mac 上的 Podman 来说非常重要，其中包括了对这些平台的完全代码重写以及在两个平台上的主要改进的虚拟化支持。还有许多其他功能，包括在清单中支持 OCI artifact，将 Pasta 设为无根网络的默认选项，改进了我们的 `containers.conf` 配置文件等等。继续阅读以了解详情！

Podman 5 的主要特性，也是我们决定发布新的主要版本的原因，是完全重写了 `podman machine` 命令。Podman 机器用于启动 Linux 虚拟机（VM），允许 Windows 和 Mac 系统运行 Linux 容器。通过重写，我们提高了性能和稳定性，并显著增加了不同 VM 提供者之间的代码共享，使未来的维护和修复变得更加容易。我们还增加了对 Mac 上 Apple hypervisor 的支持，极大地提高了稳定性、启动时间和文件共享性能。由 `podman machine` 管理的 VM 也可以通过新的 `podman machine reset` 命令轻松移除。机器的重写将需要现有用户将他们的 VM 迁移到新的后端；您可以在 [这里](https://blog.podman.io/2024/03/migration-of-podman-4-to-podman-5-machines/) 阅读更多细节。

Podman 5 还包括了一些已弃用功能、默认更改和改进。[Pasta](https://passt.top/passt/about/) 现在是默认的无根网络后端，为无根 Podman 提供了显著改进的性能。我们从 Podman 版本 4.4 开始支持 Pasta，并且现在认为其性能已经足够好，可以作为默认选择。我们还弃用了 BoltDB 数据库后端，并移除了对创建新 Bolt 数据库的支持（现有数据库仍可正常使用）。SQLite 在 Podman 版本 4.9 中成为了新安装的默认数据库，大大提高了稳定性。

Podman 5 也已移除了大多数平台上的 CNI 网络支持。我们在 Podman 4.0 中引入了 Netavark，我们自己的网络堆栈，它在所有 Podman 使用情况中已经达到或超过了 CNI。我们移除 CNI 支持是因为它给团队带来了持续的支持负担，以及 CNI 未来计划改变其架构以便专注于 Kubernetes，这将阻止 Podman 使用它。某些仍需要 CNI 支持的发行版（例如 FreeBSD 和 RHEL 9）将保持启用。

`containers.conf` 配置文件的处理方式已更改，确保我们永远不会重写用户修改过的配置文件 – 更多详情请见[这里](https://blog.podman.io/2024/03/podman-5-0-containers-conf-changes/)。我们还进行了几处改动以提升与 Docker 的兼容性，包括微调 `podman inspect` 的输出以更好地匹配 Docker 的输出。最后，我们已经弃用了对 cgroups v1 的支持，并将在未来的版本中移除在没有 cgroups v2 的系统上运行的能力。你可以在[这篇博文](https://blog.podman.io/2024/03/podman-5-0-breaking-changes-in-detail/)中详细了解所有重大变更的细节。

Podman 版本 5.0.0 还包括几项其他改进。现在可以通过 `--retry` 和 `--retry-delay` 选项控制镜像拉取和推送的重试。Quadlet 已经增加了几个新功能，支持模板单元、pod（通过 .pod 文件）以及 .container 文件中的几个额外键。最后，Podman 版本 5.0.0 包含数十个 bug 修复。你可以在[发布说明](https://github.com/containers/podman/releases/tag/v5.0.0)中找到更多信息。

.
