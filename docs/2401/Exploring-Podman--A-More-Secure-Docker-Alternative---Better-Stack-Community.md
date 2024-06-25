<!--yml

分类：未分类

日期：2024-05-27 14:44:49

-->

# 探索Podman：更安全的Docker替代方案 | 更好的堆栈社区

> 来源：[https://betterstack.com/community/guides/scaling-docker/podman-vs-docker/](https://betterstack.com/community/guides/scaling-docker/podman-vs-docker/)

容器化已经成为开发人员和系统运维人员在各种系统和平台上高效打包和部署应用程序的重要工具。今天存在许多容器化解决方案，但毫无疑问，Docker已经成为事实上的标准。这在很大程度上归功于其出色的工具、强大的社区以及庞大的预构建映像生态系统，这些映像可以轻松共享和在不同环境中使用。

Docker在许多年里一直占据着这个位置，它真正改变了应用程序的发布方式。同时，它的广泛采用也激发了许多其他容器化解决方案的发展，这些解决方案提供了更多的功能和能力。其中一种解决方案是[Podman](https://podman.io/)。

Podman是一个开源容器引擎，旨在提供一个更安全和轻量的Docker替代方案。它允许用户在不需要守护程序的情况下运行容器，从而更容易在各种系统上管理和部署容器。此外，Podman通过诸如无根容器（即通过非根用户运行容器）、用户命名空间和更谨慎地利用内核功能等功能提供了更好的安全默认设置，所有这些都可以保护主机系统免受潜在的漏洞和安全威胁的影响。

随着其不断增长的社区和与Docker映像和命令的密切兼容性，Podman在寻找替代容器化解决方案的开发人员和系统管理员中获得了显著的关注。

在本文中，我们将探讨使用Podman作为容器化工具的一些关键功能和优势。我们还将讨论它与Docker的比较，并看看为什么它已经成为行业中流行的替代选择。

让我们立即开始。

## 先决条件

在继续本文之前，请确保您满足以下先决条件：

+   优秀的Linux命令行技能。

+   具有Docker的先前经验。

+   （可选）需要一个Docker Hub账户来跟随私有注册表设置示例。

## Podman与Docker比较

虽然Podman和Docker都允许用户以高效且可伸缩的方式运行、管理和部署容器，但两者之间存在一些关键差异。在本节中，我们将探讨其中几个差异：

### 1\. 架构差异

Podman 和 Docker 之间的主要区别之一在于它们的架构。虽然 Docker 依赖于客户端 - 服务器模型，但 Podman 使用无守护进程的架构。使用 Podman 的方法，用户直接管理容器，消除了后台持续守护进程的需要。这种直接管理通常导致 Podman 容器的启动速度明显更快，有时甚至比 Docker 快 50%，这取决于使用的镜像。

此架构还增强了安全性。在 Docker 中，启动容器意味着通过 Docker 客户端向 Docker 守护进程发送请求，后者随后启动容器，这意味着容器进程是 Docker 守护进程的子进程，而不是用户会话的子进程：

因此，任何由容器进程产生的重要事件，由 Linux 审计系统（`auditd`）捕获，都会将其审计用户 ID 指定为 `unset` 而不是最初启动相应容器的用户的实际 ID。这使得将恶意活动链接到特定用户变得极为困难，并且玷污了系统的安全性。

使用 Podman，由于每个容器直接通过用户登录会话实例化，容器进程数据保留了这些信息，`auditd` 可以准确地检测和列出启动特定容器进程的每个用户的 ID，维护着清晰的审计追踪。

### 2\. 容器生命周期管理

在 Podman 中缺少守护进程导致了与 Docker 相比管理容器生命周期的独特方法。在 Linux 上，Podman 主要依赖于[systemd](https://systemd.io/)来实现这一目的。例如，为了正确地强制执行使用 `--restart always` 标志的容器的重启策略，Podman 依赖于一个名为 `podman-restart` 的 `systemd` 服务。该服务在每次系统重新启动后自动重新启动所有指定的容器。

此外，Podman 提供了一个方便的命令，用于从运行中的容器生成 Systemd 服务文件。这允许您将容器纳入 `systemd` 管理，以更轻松地启动、停止和检查其中运行的各种服务。相比之下，Docker 通过守护进程内部处理所有这些任务。

### 3\. 容器编排

在本地开发时，Docker 用户通常会使用 [Docker Compose](https://docs.docker.com/compose/) 来更轻松地定义和管理多容器应用。虽然 Podman 不直接支持 Compose 文件，但提供了一个兼容的替代方案，称为 [Podman Compose](https://github.com/containers/podman-compose)，它通常与现有的 `docker-compose.yml` 文件无缝配合。为了获得原生体验，您还可以使用 pod，这是 Podman 从 [Kubernetes](https://kubernetes.io/) 借鉴的概念，允许用户将一组容器作为一个统一单元进行管理。

在生产部署方面，Podman 缺乏类似 [Docker Swarm](https://docs.docker.com/engine/swarm/) 的工具，用于编排多容器应用工作负载。在这种情况下，最好的替代方案是使用外部编排系统，例如 Kubernetes，它提供类似的功能并且与 Podman 集成良好，尽管可能需要一些额外的配置和设置来确保一切正常运行。

### 4\. 安全性考虑

容器旨在安全地将应用程序与主机系统隔离开来，从而最大程度地减少兼容性问题并增强安全性。主要的安全性问题是容器突破的风险，攻击者可能会危害主机系统。为了减轻此类风险，以最低权限运行容器至关重要。

Podman 的设计目标是通过提供比 Docker 更强的默认安全设置来帮助实现这一目标。诸如 [无根容器](https://docs.docker.com/engine/security/rootless/)、[用户命名空间](https://docs.docker.com/engine/security/userns-remap/) 和 [seccomp 配置文件](https://docs.docker.com/engine/security/seccomp/) 等功能，虽然在 Docker 中可用，但默认情况下并未启用，并且通常需要额外的设置。

Podman 的默认设置包括在隔离的用户命名空间中运行的无根容器，限制了任何潜在突破的影响。相比之下，Docker 的默认设置将容器进程作为 `root` 运行，这在突破的情况下存在更高的风险。

此外，与 Docker 不同，Podman 容器与用户会话绑定，允许审计系统追溯恶意活动至特定用户，这是 Docker 难以做到的，因为追溯到用户在系统范围内的守护进程上是具有挑战性的。

Podman 和 Docker 使用 Linux 内核功能和 seccomp 配置文件来控制进程权限。默认情况下，Podman 使用比 Docker 更狭窄的 11 个能力集来启动容器，而 Docker 使用更宽松的默认设置，具有 14 个能力。

一般来说，虽然 Podman 和 Docker 都可以配置为具有强大的安全性，但 Podman 通常需要更少的工作量才能达到安全的配置。

**了解更多**: [Docker 安全性：你应该知道的 14 个最佳实践](/community/guides/scaling-docker/docker-security-best-practices/)

* * *

总的来说，无论是 Docker 还是 Podman，都是功能强大的工具，了解两者之间的主要区别可以帮助您选择适合您特定需求的工具。请参考下表，了解两者之间的一些主要区别，您可以假设大多数其他功能完全相同。

|  | Podman | Docker |
| --- | --- | --- |
| 无守护进程架构 | ✔ | ✘ |
| Systemd 集成 | ✔ | ✘ |
| 将容器分组到 Pod 中 | ✔ | ✘ |
| 支持 Docker Swarm | ✘ | ✔ |
| 支持 Kubernetes YAML | ✔ | ✘ |

有了这一澄清，让我们继续在本地安装 Podman。

## 安装 Podman

像 Docker 一样，Podman 可以在所有流行的操作系统上运行良好。这包括 macOS 和 Windows，以及所有主要的 Linux 发行版。唯一需要注意的重大区别是，虽然 Podman 可以在 Linux 上本地运行，但在 Windows 和 macOS 上需要虚拟机才能工作。这在这些系统上增加了一些额外的安装步骤，但除此之外，核心功能保持不变。

本教程假设你使用的是基于 Debian 的 Linux 发行版，如 Ubuntu、Mint 或 Debian 本身。其他操作系统的安装过程非常类似，可以通过参考官方[Podman 安装说明](https://podman.io/docs/installation)来应用。

确保你使用的是相对较新版本的你偏好的发行版，因为旧版本的 PPA 仓库可能没有最新可用的 Podman 可供安装。例如，在撰写本文时，Podman 的最新主要版本是`4.x`，然而最新的 Ubuntu LTS 版本（[Ubuntu 22.04 LTS](https://discourse.ubuntu.com/t/jammy-jellyfish-release-notes/24668)）将你锁定到 Podman `3.x`。因此，本教程基于较新的非 LTS 版本（[Ubuntu 23.10](https://discourse.ubuntu.com/t/mantic-minotaur-release-notes/35534)）。

首先从所有配置的上游软件包源下载最新信息：

然后运行以下命令来安装 Podman：

你会看到类似的输出：

```
Reading package lists... Done
Building dependency tree... Done
Reading state information... Done
The following additional packages will be installed:
  aardvark-dns buildah catatonit conmon containernetworking-plugins crun fuse-overlayfs golang-github-containers-common golang-github-containers-image libslirp0 libsubid4 libyajl2 netavark slirp4netns uidmap
Suggested packages:
  containers-storage libwasmedge0 docker-compose
The following NEW packages will be installed:
  aardvark-dns buildah catatonit conmon containernetworking-plugins crun fuse-overlayfs golang-github-containers-common golang-github-containers-image libslirp0 libsubid4 libyajl2 netavark podman slirp4netns uidmap
0 upgraded, 16 newly installed, 0 to remove and 37 not upgraded.
Need to get 28.4 MB of archives.
After this operation, 116 MB of additional disk space will be used.
Do you want to continue? [Y/n] Y 
```

在提示时键入`Y`，然后按`Enter`继续。

安装完成后，你可以通过输入以下命令来验证 Podman 是否可用：

输出表明版本`4.3.1`已成功本地安装，并且你可以运行`podman`命令。现在，你已准备好启动你的第一个容器。

## 运行你的第一个容器

让我们通过运行来验证安装是否正常工作著名的["Hello World!"镜像](https://hub.docker.com/_/hello-world)从[Docker Hub](https://hub.docker.com/)：

```
podman run --rm hello-world 
```

你会看到类似的输出：

```
Resolved "hello-world" as an alias (/etc/containers/registries.conf.d/shortnames.conf)
Trying to pull docker.io/library/hello-world:latest...
Getting image source signatures
Copying blob 719385e32844 done
Copying config 9c7a54a9a4 done
Writing manifest to image destination
Storing signatures

Hello from Docker!
This message shows that your installation appears to be working correctly.

To generate this message, Docker took the following steps:
 1\. The Docker client contacted the Docker daemon.
 2\. The Docker daemon pulled the "hello-world" image from the Docker Hub.
    (amd64)
 3\. The Docker daemon created a new container from that image which runs the
    executable that produces the output you are currently reading.
 4\. The Docker daemon streamed that output to the Docker client, which sent it
    to your terminal.

To try something more ambitious, you can run an Ubuntu container with:
 $ docker run -it ubuntu bash

Share images, automate workflows, and more with a free Docker ID:
 https://hub.docker.com/

For more examples and ideas, visit:
 https://docs.docker.com/get-started/ 
```

此输出显示了一些重要的内容。即使"Hello World!"镜像是使用 Docker 生态系统中的工具构建的，Podman 仍然可以运行容器。

这是因为 Docker 和 Podman 都遵循 OCI（[Open Container Initiative](https://opencontainers.org/)）标准。OCI 定义了一个[镜像格式规范](https://github.com/opencontainers/image-spec/blob/main/spec.md)和一个[运行时规范](https://github.com/opencontainers/runtime-spec)，使不同的容器运行时（如 Podman 和 Docker）能够进行互操作。

因此，Podman 可以无缝地与大多数 Docker 镜像和容器一起工作。这种兼容性使您可以轻松地将现有的工作负载迁移到 Podman，而无需进行任何修改，还可以利用 Docker Hub 上可用的大量 Docker 镜像库。

让我们进一步解析上述命令的输出。第一行说：

```
Resolved "hello-world" as an alias (/etc/containers/registries.conf.d/shortnames.conf) 
```

此消息表示 Podman 引用了配置文件以获取`hello-world`镜像的完全限定名称。与 Docker 不同，Podman 建议不要使用简称来引用镜像，并且不会默认使用特定的注册表，除非通过其配置文件明确指示这样做。另一方面，Docker 总是使用 Docker Hub (`docker.io`) 作为其默认注册表，并尝试在没有明确提供完全限定名称时在那里定位每个镜像。

您可以进一步检查`shortnames.conf`文件，并确认`hello-world`别名映射到图像`docker.io/library/hello-world`：

```
grep hello-world /etc/containers/registries.conf.d/shortnames.conf 
```

```
"hello-world" = "docker.io/library/hello-world" 
```

接下来的几行说：

```
Trying to pull docker.io/library/hello-world:latest...
Getting image source signatures
Copying blob 719385e32844 done
Copying config 9c7a54a9a4 done
Writing manifest to image destination
Storing signatures 
```

这些消息表明最新版本的`hello-world`镜像已成功下载到您的本地计算机。

您可以通过发出以下命令进行验证：

```
REPOSITORY                     TAG         IMAGE ID      CREATED       SIZE
docker.io/library/hello-world  latest      9c7a54a9a43c  6 months ago  19.9 kB 
```

剩下的输出（“Hello from Docker!”消息和所有后续行）显示了嵌入在`hello-world`镜像中附带的[hello](https://github.com/docker-library/hello-world/blob/master/hello.c)二进制文件中的消息。此消息可能有点误导，因为它暗示 Docker 引擎执行了容器，而实际上是 Podman 执行了。重要的是要理解，Docker 客户端和 Docker 守护程序在任何情况下都没有参与该过程。

在您进一步操作之前，删除`hello-world`镜像，因为它已不再需要：

输出表明已删除镜像，并显示其标签和 ID 供参考：

```
Untagged: docker.io/library/hello-world:latest
Deleted: 9c7a54a9a43cca047013b82af109fe963fde787f63f9e016fdc3384500c2823d 
```

接下来，让我们看看当您仅使用其简称运行任意镜像时会发生什么。

## 使用简称

正如我们之前讨论的，Podman 建议使用完全限定名称来避免歧义，并确保始终从特定注册表引用正确的镜像。

让我们尝试使用[Docker Hub](https://hub.docker.com/_/caddy)的官方 Caddy 镜像运行容器。如果您对[Caddy](/community/guides/web-servers/caddy/)不熟悉，它是一个轻量级的 Web 服务器和反向代理，以易用性和出色的性能而闻名。通过将 Caddy 作为容器运行，您可以快速启动一个用于测试目的的本地 Web 服务器。

从 Docker 背景出发，您通常会使用以下命令启动容器：

```
docker run --rm -p 8080:80 caddy 
```

这在 Podman 中对应以下命令：

```
podman run --rm -p 8080:80 caddy 
```

大多数情况下，从 Docker 切换到 Podman 只是在命令中将`docker`改为`podman`的问题。事实上，许多用户将`alias docker=podman`添加到其 shell 配置文件中，并使用 Podman 作为`docker`命令的别名。

标志也是相同的：

+   `--rm` 指定容器在退出后应自动删除，以免在系统中留下垃圾。

+   `-p 8080:80` 指定主机上的端口 `8080` 应映射到容器中的端口 `80`，这样您就可以通过在浏览器中键入 `localhost:8080` 访问容器内运行的 web 服务器。

尝试运行该命令。如果一切顺利，一个容器应该启动，允许你在浏览器中打开 `localhost:8080` 来查看内置的 Caddy 测试页面。

与你可能预期的相反，该命令失败并返回以下错误：

```
Error: short-name "caddy" did not resolve to an alias and no unqualified-search registries are defined in "/etc/containers/registries.conf" 
```

该命令未生效，因为 Podman 无法理解从何处拉取 `caddy` 镜像。

它首先在 `shortnames.conf` 中查找名为 `caddy` 的别名，但找不到。尝试对 `shortnames.conf` 进行 `grep`，你会看到它确实没有返回任何匹配项：

```
grep caddy /etc/containers/registries.conf.d/shortnames.conf 
```

然后它在 `registries.conf` 文件中查找一组所谓的未限定搜索注册表。未限定搜索注册表是指每当 `run` 命令提供了非完全限定的镜像名称时，Podman 尝试联系的注册表。

针对你遇到的问题，有三种可能的解决方案。让我们探索所有这些：

### 1\. 指定完全限定名称

你可以指定完全限定名称：

```
podman run --rm -p 8080:80 docker.io/library/caddy 
```

这有效！但是，如果你来自 Docker 背景，你可能会发现使用短名称更符合人体工程学，因为这是你习惯的工作流程。这在 Podman 中绝对是可能的，使用别名或未限定的搜索注册表，我们将在下面探讨这一点。

### 2\. 定义别名

你可能决定为 `caddy` 定义一个别名。在这样做时，请记住 `/etc/containers/registries.conf.d/shortnames.conf` 文件不应直接修改，因为它作为 [shortnames 项目](https://github.com/containers/shortnames) 的一部分进行了打包。定义新别名的正确方式是向你的 `registries.conf` 文件添加一个 `[aliases]` 部分，就像这样：

/etc/containers/registries.conf

复制！

```
[aliases]
"caddy"="docker.io/caddy" 
```

Podman 的默认注册表配置位于 `/etc/containers/registries.conf`，但修改此文件需要 root 权限。这在某种程度上违背了 Podman 旨在支持的无根访问的理念。然而，Podman 提供了一种克服此限制的机制。你可以将你的配置放入 `$HOME/.config/containers/registries.conf`，它将优先于 `/etc/containers/registries.conf`。这不需要 root 权限。

继续并在你的系统上创建 `$HOME/.config/containers/registries.conf` 文件：

```
mkdir -p $HOME/.config/containers && touch $HOME/.config/containers/registries.conf 
```

然后将以下行添加到你的文件中：

~/.config/containers/registries.conf

复制！

```
[aliases]
"caddy"="docker.io/caddy" 
```

现在重新运行：

```
podman run --rm -p 8080:80 caddy 
```

你将看到以下输出，指示解决方案有效：

```
Resolved "caddy" as an alias (/home/<your_user_name>/.config/containers/registries.conf)
Trying to pull docker.io/library/caddy:latest...
... 
```

虽然这个解决方案有效，但为你打算使用的每个镜像添加别名将在长期内变得乏味和耗时。这引导我们进入第三种可能的解决方案——配置未限定的搜索注册表。

在这之前，撤销你到目前为止所做的更改，这样你就可以从一个干净的状态开始。

按下 `Ctrl+C` 终止 Caddy 容器，并返回到你的终端。然后通过截断整个文件，从 `registries.conf` 文件中删除 `[aliases]` 配置：

```
echo > $HOME/.config/containers/registries.conf 
```

删除 Podman 刚下载的 `caddy` 镜像：

正如预期的那样，输出显示已删除的图像的标签和 ID：

```
Untagged: docker.io/library/caddy:latest
Deleted: 70c3913f54e294079c54cc8b651576b08dd4add42a0f4c0bc93d539913ae335d 
```

您现在已准备好定义一个未命名搜索注册表。

### 3\. 定义未命名搜索注册表

打开您的 `$HOME/.config/containers/registries.conf` 文件并粘贴以下内容：

~/.config/containers/registries.conf

复制！

```
unqualified-search-registries=["docker.io"] 
```

现在重新运行：

```
podman run --rm -p 8080:80 caddy 
```

您将看到以下输出：

```
Resolving "caddy" using unqualified-search registries (/home/<your_user_name>/.config/containers/registries.conf)
Trying to pull docker.io/library/caddy:latest
... 
```

`caddy` 镜像已成功下载，一个容器已启动，Caddy 现在已准备好提供 Web 请求服务。

为了确认一切正常工作，您可以导航到 `localhost:8080`。 在那里，您应该看到 Caddy 的测试页面：

使用未命名搜索注册表无疑是一个比使用别名更好的选择，特别是如果您打算将 Podman 作为 Docker 的替代品使用，因为您可以继续使用您习惯的方式使用短名称，并且它们将默认从 Docker Hub 解析。

在继续之前，请返回到您的终端并按下 `Ctrl+C` 停止容器，然后删除 Caddy 镜像：

```
Untagged: docker.io/library/caddy:latest
Deleted: 70c3913f54e294079c54cc8b651576b08dd4add42a0f4c0bc93d539913ae335d 
```

## 使用私有镜像注册表

您可能习惯于使用托管您组织专有图像的私有注册表。 Docker 无疑可以促成这一点，Podman 也可以。对于这两个工具，该过程非常相似。

本示例假设您拥有一个可用的 Docker Hub 账户。 如果您没有账户，可以[免费注册一个](https://hub.docker.com/signup)。 免费套餐允许您免费维护一个私有仓库。

登录您的 Docker Hub 账户并转到 [Account Settings](https://hub.docker.com/settings/general)：

转到 [Security](https://hub.docker.com/settings/security) 并点击 **New Access Token**：

将 **Podman 教程** 指定为描述，**读写** 为所需权限，然后点击 **生成**：

复制生成的访问令牌并将其存储在安全的地方。我们将此令牌称为 `<your_access_token>`：

令牌现在应列在您的 Docker Hub 账户的可用访问令牌下：

返回到 [Repositories](https://hub.docker.com/repositories) 并按以下步骤创建一个新的私有仓库：

如果一切顺利，您应该在您的 Docker Hub 账户中看到仓库列表：

现在让我们配置 Podman 以使用您的私有仓库运行。

输入以下命令：

如果您在 `registries.conf` 文件中将 `docker.io` 列为第一个 `unqualified-search-registries` 条目，则可以省略此处的 `docker.io`。尽管如此，明确指定注册表仍然被认为是一个好的做法。

否则，`podman login` 将失败，并显示以下错误：

```
Error: no registries found in registries.conf, a registry must be provided 
```

当提示时输入您的用户名和访问令牌，您应该收到以下输出：

```
Username: <your_user_name>
Password: <your_access_token>
Login Succeeded! 
```

继续通过在本地下载官方的 `hello-world` 镜像：

```
podman pull docker.io/hello-world 
```

成功的输出将类似于这样：

```
Trying to pull docker.io/library/hello-world:latest...
Getting image source signatures
Copying blob 719385e32844 done
Copying config 9c7a54a9a4 done
Writing manifest to image destination
Storing signatures
9c7a54a9a43cca047013b82af109fe963fde787f63f9e016fdc3384500c2823d 
```

现在列出您本地的 Podman 图像：

您应该看到以下输出：

```
REPOSITORY                     TAG         IMAGE ID      CREATED       SIZE
docker.io/library/hello-world  latest      9c7a54a9a43c  6 months ago  19.9 kB 
```

继续并将 `hello-world` 镜像的副本上传到您的私有仓库。

```
podman push docker.io/library/hello-world docker.io/<your_user_name>/hello-private 
```

您将收到以下输出：

```
Getting image source signatures
Copying blob 01bb4fce3eb1 skipped: already exists
Copying config 9c7a54a9a4 done
Writing manifest to image destination
Storing signatures 
```

转到 Docker Hub 上的私有存储库，以验证镜像是否成功上传：

您现在可以从您的本地计算机中删除公共 `hello-world` 镜像了：

```
Untagged: docker.io/library/hello-world:latest
Deleted: 9c7a54a9a43cca047013b82af109fe963fde787f63f9e016fdc3384500c2823d 
```

现在，尝试使用来自您私有存储库的镜像运行容器：

```
podman run --rm docker.io/<your_user_name>/hello-private 
```

Podman 继续进行，成功拉取私有镜像，并使用该镜像启动容器：

```
Trying to pull docker.io/<your_user_name>/hello-private:latest...
Getting image source signatures
Copying blob 719385e32844 done
Copying config 9c7a54a9a4 done
Writing manifest to image destination
Storing signatures

Hello from Docker!
... 
```

如果没有有效的登录凭据，您将收到以下错误：

```
Trying to pull docker.io/<your_user_name>/hello-private:latest...
Error: initializing source docker://<your_user_name>/hello-private:latest: reading manifest latest in docker.io/<your_user_name>/hello-private: errors:
denied: requested access to the resource is denied
unauthorized: authentication required 
```

正如您所见，使用 Podman 连接私有注册表几乎与使用 Docker 相同。唯一的区别是您的命令前缀为 `podman` 而不是 `docker`。与 Docker 一样，您可以使用 Podman 连接所有流行的私有注册表。

在继续之前，请确保注销：

```
Removed login credentials for docker.io 
```

同时，从本地计算机中删除私有镜像：

```
Untagged: docker.io/<your_user_name>/hello-private:latest
Deleted: 9c7a54a9a43cca047013b82af109fe963fde787f63f9e016fdc3384500c2823d 
```

有了这个，您可以继续下一节，并学习如何使用 Podman 管理多个容器。

## 管理多个容器

到目前为止，您只启动了一个容器来探索 Podman 的工作方式。在某些时候，您肯定会发现有必要运行多个容器一起作为一个单位工作。在本节中，您将探索使用 [Podman Compose](https://github.com/containers/podman-compose) 的可能方式之一。

Podman 提供了几种用于管理多个容器的选项，但 Podman Compose 是最接近 Docker 世界使用的选项。其他选项包括 pods 和 Kubernetes 清单，但这两者都需要更深入地理解 Podman（和 Kubernetes），所以我们暂时不讨论它们。

在以下示例中，您将使用 Docker Hub 提供的 [official WordPress image](https://hub.docker.com/_/wordpress) 上的 Docker Compose 指令来启动一个简单的由 MySQL 数据库服务器支持的 [WordPress](https://github.com/WordPress/WordPress) 安装。如果您不熟悉 WordPress，它是一个流行的用 PHP 编写的博客和内容管理平台。

在 Docker 生态系统中，Compose 允许您通过存储在 `docker-compose.yml` 文件中的定义来定义和管理多个容器。习惯于使用 Docker Compose，您可以借助 Podman Compose 继续使用现有的 `docker-compose.yml` 文件。

Podman Compose 是一个由社区驱动的工具，它实现了 [Compose specification](https://compose-spec.io/) 并与 Podman 无缝集成。它依赖于 Python 3 运行，并且通过 [pipx](https://pipx.pypa.io/stable/) 是开始使用它的最简单的方法之一。

如果您的系统尚未安装 Python 3 或 `pipx`，您可以通过运行以下命令来安装它们：

您可以通过输入以下命令来验证 `pipx` 是否正常工作：

这将输出一个类似于版本标识符：

输出确认您的系统已安装了版本为 `1.2.0` 的 `pipx`，并且您已准备好开始使用它。

您可以运行以下命令来安装 Podman Compose：

```
pipx install podman-compose 
```

如果一切顺利，您应该会看到：

```
installed package podman-compose 1.0.6, installed using Python 3.11.6
These apps are now globally available
  - podman-compose
⚠️ Note: '/home/<your_user_name>/.local/bin' is not on your PATH environment variable. These apps will not be globally accessible until your PATH is updated. Run `pipx ensurepath` to automatically add it, or manually modify your PATH in your shell's config file (i.e. ~/.bashrc).
done! ✨ 🌟 ✨ 
```

输出指示`pipx`已放置在你的`$HOME/.local/bin`文件夹中。 但是，那个文件夹可能没有包含在你的`$PATH`变量中，这意味着如果你现在键入`podman-compose`，你会得到类似的错误：

```
Command 'podman-compose' not found, but can be installed with:
sudo apt install podman-compose 
```

不要被这个错误搞混了。 Podman Compose 已成功安装，但是你需要将其安装文件夹添加到你的`$PATH`中。

你可以通过运行来解决这个问题：

这个命令将通过你的一个 shell 配置文件（`.profile`、`.bash_profile`、`.bashrc`等，取决于你的具体设置）确保将`$HOME/.local/bin`追加到你的`$PATH`中。

运行该命令后，你将看到以下输出：

```
/home/<your_user_name>/.local/bin has been added to PATH, but you need to open a new terminal or re-login for this PATH change to take effect.

You will need to open a new terminal or re-login for the PATH changes to take effect.

Otherwise pipx is ready to go! ✨ 🌟 ✨ 
```

你可以按照说明重新打开你的终端会话。 或者，如果你确切地知道哪个 shell 配置文件被修改了，你可以为了使更改立即生效而源文件。

例如：

要确认`$PATH`是否设置正确，你可以运行：

```
echo $PATH | tr ':' '\n' | grep -F .local 
```

你应该在输出中看到`/home/<your_user_name>/.local/bin`：

```
/home/<your_user_name>/.local/bin 
```

创建一个新文件夹并`cd`进入：

```
mkdir podman-tutorial && cd podman-tutorial 
```

创建一个`.env`文件并定义以下变量：

```
DB_USER=<your_example_username>
DB_PASS=<your_example_password>
DB_NAME=<your_example_database> 
```

然后，创建一个新的`docker-compose.yml`文件并粘贴以下内容：

docker-compose.yml

复制！

```
version: '3'

services:

  wordpress:
    image: wordpress
    container_name: wordpress
    restart: always
    ports:
      - '8080:80'
    environment:
      WORDPRESS_DB_HOST: db
      WORDPRESS_DB_USER: ${DB_USER}
      WORDPRESS_DB_PASSWORD: ${DB_PASS}
      WORDPRESS_DB_NAME: ${DB_NAME}
    volumes:
      - wordpress:/var/www/html

  db:
    image: mysql:5.7
    container_name: db
    restart: always
    ports:
      - '3306:3306'
    environment:
      MYSQL_DATABASE: ${DB_NAME}
      MYSQL_USER: ${DB_USER}
      MYSQL_PASSWORD: ${DB_PASS}
      MYSQL_RANDOM_ROOT_PASSWORD: '1'
    volumes:
      - db:/var/lib/mysql

volumes:
  wordpress:
  db: 
```

保存文件，然后运行：

让我们逐段检查输出。

首先，`podman-compose`启动并开始分析`docker-compose.yml`文件：

```
podman-compose version: 1.0.6
['podman', '--version', '']
using podman version: 4.3.1
** excluding:  set()
['podman', 'ps', '--filter', 'label=io.podman.compose.project=podman-tutorial', '-a', '--format', '{{ index .Labels "io.podman.compose.config-hash"}}'] 
```

它检测到两个名为`wordpress`和`db`的服务，并按顺序开始处理它们。

首先，它发现`wordpress`服务需要一个外部卷。 它尝试通过发出`podman volume inspect <volume_name>`来定位卷，但由于不存在，它继续通过发出`podman volume create <volume_name>`来创建一个卷：

```
podman volume inspect podman-tutorial_wordpress || podman volume create podman-tutorial_wordpress
['podman', 'volume', 'inspect', 'podman-tutorial_wordpress']
Error: inspecting object: no such volume podman-tutorial_wordpress
['podman', 'volume', 'create', '--label', 'io.podman.compose.project=podman-tutorial', '--label', 'com.docker.compose.project=podman-tutorial', 'podman-tutorial_wordpress']
['podman', 'volume', 'inspect', 'podman-tutorial_wordpress'] 
```

接下来，它检查是否有适用于部署`wordpress`服务的合适网络。 它未找到，因此创建并执行另一个检查以确认网络是否存在：

```
['podman', 'network', 'exists', 'podman-tutorial_default']
['podman', 'network', 'create', '--label', 'io.podman.compose.project=podman-tutorial', '--label', 'com.docker.compose.project=podman-tutorial', 'podman-tutorial_default']
['podman', 'network', 'exists', 'podman-tutorial_default'] 
```

最后，使用合适的网络和已创建的外部卷，Podman Compose 启动`wordpress`容器：

```
podman run --name=wordpress -d --label io.podman.compose.config-hash=c3e51c9a26e54848a06f92083413698bee90acd721b4aa3054280e110819a68c --label io.podman.compose.project=podman-tutorial --label io.podman.compose.version=1.0.6 --label PODMAN_SYSTEMD_UNIT=podman-compose@podman-tutorial.service --label com.docker.compose.project=podman-tutorial --label com.docker.compose.project.working_dir=/home/<your_user_name>/podman-tutorial --label com.docker.compose.project.config_files=docker-compose.yml --label com.docker.compose.container-number=1 --label com.docker.compose.service=wordpress -e WORDPRESS_DB_HOST=db -e WORDPRESS_DB_USER=testuser -e WORDPRESS_DB_PASSWORD=testpass -e WORDPRESS_DB_NAME=testdb -v podman-tutorial_wordpress:/var/www/html --net podman-tutorial_default --network-alias wordpress -p 8080:80 --restart always wordpress 
```

由于 Podman 无法在本地找到可用的`wordpress`镜像，因此它继续查找你之前配置的未经授权的搜索注册表（`docker.io`），然后下载它：

```
Resolving "wordpress" using unqualified-search registries (/home/<your_user_name>/.config/containers/registries.conf)
Trying to pull docker.io/library/wordpress:latest...
Getting image source signatures
Copying blob c5d33b602102 done
...
Copying config bc823df9ea done
Writing manifest to image destination
Storing signatures
980f5792fb9c34ea9357ffa290d5d79ac0b48b70701cd4693f5d7e2b4f2312f1
exit code: 0 
```

接下来，Podman Compose 继续处理`db`服务的指令。

首先，它创建其外部卷：

```
podman volume inspect podman-tutorial_db || podman volume create podman-tutorial_db
['podman', 'volume', 'inspect', 'podman-tutorial_db']
Error: inspecting object: no such volume podman-tutorial_db
['podman', 'volume', 'create', '--label', 'io.podman.compose.project=podman-tutorial', '--label', 'com.docker.compose.project=podman-tutorial', 'podman-tutorial_db']
['podman', 'volume', 'inspect', 'podman-tutorial_db'] 
```

然后，它确保有适用于部署的合适网络：

```
['podman', 'network', 'exists', 'podman-tutorial_default'] 
```

最后，它启动`db`容器：

```
podman run --name=db -d --label io.podman.compose.config-hash=c3e51c9a26e54848a06f92083413698bee90acd721b4aa3054280e110819a68c --label io.podman.compose.project=podman-tutorial --label io.podman.compose.version=1.0.6 --label PODMAN_SYSTEMD_UNIT=podman-compose@podman-tutorial.service --label com.docker.compose.project=podman-tutorial --label com.docker.compose.project.working_dir=/home/<your_user_name>/podman-tutorial --label com.docker.compose.project.config_files=docker-compose.yml --label com.docker.compose.container-number=1 --label com.docker.compose.service=db -e MYSQL_DATABASE=testdb -e MYSQL_USER=testuser -e MYSQL_PASSWORD=testpass -e MYSQL_RANDOM_ROOT_PASSWORD=1 -v podman-tutorial_db:/var/lib/mysql --net podman-tutorial_default --network-alias db --restart always mysql:5.7 
```

就像`wordpress`镜像一样，Podman 无法在本地找到`mysql:5.7`镜像，因此继续从 Docker Hub 获取它：

```
Resolving "mysql" using unqualified-search registries (/home/<your_user_name>/.config/containers/registries.conf)
Trying to pull docker.io/library/mysql:5.7...
Getting image source signatures
Copying blob 62aca7179a54 done
...
Copying config bdba757bc9 done
Writing manifest to image destination
Storing signatures
21615c3d36236f181b94831b5c1eb699153f7cf912fdefcb342732069f99c09d
exit code: 0 
```

此时，Podman Compose 成功退出，并且一切似乎都启动正确。 让我们继续验证这一点。

首先，尝试在浏览器中打开`localhost:8080`。 你应该看到WordPress安装页面：

现在，回到你的终端并键入：

实际上，`wordpress`容器和`db`容器都已经启动并运行：

```
CONTAINER ID  IMAGE                               COMMAND               CREATED        STATUS            PORTS                 NAMES
980f5792fb9c  docker.io/library/wordpress:latest  apache2-foregroun...  3 minutes ago  Up 3 minutes ago  0.0.0.0:8080->80/tcp  wordpress
21615c3d3623  docker.io/library/mysql:5.7         mysqld                2 minutes ago  Up 2 minutes ago                        db 
```

你也可以继续探索可用镜像的列表：

列表包含了运行MySQL和WordPress所需的所有必要镜像：

```
REPOSITORY                   TAG         IMAGE ID      CREATED       SIZE
docker.io/library/wordpress  latest      bc823df9ead2  28 hours ago  683 MB
docker.io/library/mysql      5.7         bdba757bc933  4 weeks ago   520 MB 
```

你可以进一步探索可用网络的列表：

有两个网络显示出来：

```
NETWORK ID    NAME                     DRIVER
2f259bab93aa  podman                   bridge
3f5fa386b31c  podman-tutorial_default  bridge 
```

当你首次安装Podman时，默认创建`podman`网络。当没有显式指定其他网络时，它用于启动容器。另一方面，`podman-tutorial_default`网络是由Podman Compose创建的，用于隔离你的`docker-compose.yml`中定义的容器与可能在同一系统上运行的任何其他容器。

最后，让我们检查可用的卷：

不出所料，出现了两个外部卷：

```
DRIVER      VOLUME NAME
local       podman-tutorial_db
local       podman-tutorial_wordpress 
```

你可以继续停止容器：

Podman Compose 停止并移除了你系统中的容器，但保留了网络和卷：

```
podman-compose version: 1.0.6
['podman', '--version', '']
using podman version: 4.3.1
** excluding:  set()
podman stop -t 10 db
db
exit code: 0
podman stop -t 10 wordpress
wordpress
exit code: 0
podman rm db
db
exit code: 0
podman rm wordpress
wordpress
exit code: 0 
```

你可以通过输入来验证没有容器在运行：

结果显示一个空输出，这证实了所有容器都已停止和移除。

```
CONTAINER ID  IMAGE       COMMAND     CREATED     STATUS      PORTS       NAMES 
```

如果你想连同卷一起移除，可以输入：

```
podman volume rm podman-tutorial_db podman-tutorial_wordpress 
```

要移除网络，请输入：

```
podman network rm podman-tutorial_default 
```

正如你所看到的，命令与通常使用Docker和Docker Compose时使用的命令相同。唯一显著的区别是，你要输入`podman`和`podman-compose`，而不是`docker`和`docker-compose`。

## 最终思考

Podman是一种功能强大的容器化技术，为运行容器工作负载提供了与Docker相媲美的可行选择。你选择Podman还是Docker完全取决于你的具体需求和偏好。

Podman可以做的大多数事情都与Docker一样，而且不需要在后台运行守护进程的额外好处。除此之外，Podman还提供了一些Docker没有的功能，比如与Kubernetes清单文件一起工作，以及将单个容器组织成Pod。

最终的决定由你做出。如果你需要一个更轻量级和安全的容器管理解决方案，Podman可能是一个更好的选择。然而，如果你优先考虑具有广泛社区支持的强大生态系统，那么Docker可能是更好的选择。最终，这两个工具都提供了强大的容器化能力，以满足你的需求。

要进一步探索Podman，请考虑访问[官方Podman网站](https://podman.io/)，探索其[文档](https://podman.io/docs)，并加入其不断增长的[社区](https://podman.io/community)。

谢谢阅读！
