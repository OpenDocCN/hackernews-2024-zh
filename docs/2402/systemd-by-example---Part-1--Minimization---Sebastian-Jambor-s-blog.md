<!--yml

category: 未分类

date: 2024-05-27 14:54:21

-->

# systemd by example - Part 1: Minimization - Sebastian Jambor's blog

> 来源：[https://seb.jambor.dev/posts/systemd-by-example-part-1-minimization/](https://seb.jambor.dev/posts/systemd-by-example-part-1-minimization/)

November 23, 2021

# 通过示例理解 systemd

## Part 1: Minimization

### 系列概述

本文是系列 *systemd by example* 的一部分。以下文章也可供参考。

### 引言

这是一个系列文章的第一篇，我将通过创建小型容器化示例来理解 systemd。

systemd 对我来说一直是个谜。我知道它用于系统初始化和服务管理，但我并不真正理解它的工作原理。每次我试图深入了解，比如查看我的机器设置或阅读文档，我很快就感到不知所措。我的系统上有超过 300 个活跃的 systemd 单元，要知道哪些是重要的，以及它们的用途并不容易。手册内容非常详尽，但很容易在细节中迷失。在线资源也是如此：虽然有很多，但没有一篇真正让我豁然开朗。尽管 systemd 的原始公告 [重新思考 PID 1](http://0pointer.de/blog/projects/systemd.html) 解答了不少疑问。

在像这样的情况下，通常能帮助我解决问题的方法是从一个只包含基本要素的最小示例开始，试图理解其工作原理；然后逐步扩展它：添加新功能，探索文档中描述的内容，尝试不同的设置；最后进行迭代。对于 systemd 来说，这一开始看起来很难做到。毕竟，如果我不知道自己在做什么，我确实不想在系统配置中胡乱折腾。此外，实验必然意味着可能会出问题，这绝对是我不愿在实际系统上尝试的事情。

我后来找到了这篇关于 [如何在容器中运行 systemd](https://developers.redhat.com/blog/2019/04/24/how-to-run-systemd-in-a-container) 的文章。这让我能够完全按自己的意愿进行操作！它为示例提供了一个测试环境，并允许快速迭代实验。因为限定在容器中，所以即使出了问题也没关系。而且通过使用不同的目录和版本控制可以轻松跟踪不同的示例。

那么，让我们从创建一个最小的 systemd 示例开始吧！

*注*: 显然我不是 systemd 的专家，所以请对我写的一切持保留态度。我希望这篇文章不是权威信息，而是邀请你进行自己的 systemd 实验。

### systemd 单元

systemd的基本构建块是一个单元。它们大多数在单元文件中定义，这些文件位于`/lib/systemd/system`和`/etc/systemd/system`（前者应包含发行版安装的单元文件，后者包含系统管理员创建的单元文件）。每个单元都有一个类型，编码在文件扩展名中。目前有十一个不同的单元类型，但在本文中我们只考虑其中的三种：目标、服务和套接字。

*目标*是它们中最简单的。systemd根据系统状态激活特定的目标。例如，有`bluetooth.target`，当蓝牙控制器可用时激活；有`sleep.target`，当系统进入睡眠状态时激活；还有`default.target`，这是systemd在启动时激活的单元。

一个目标本身是相当无用的。只有通过它的*依赖*才变得有用，它可以是任何其他单元。这就是*服务*发挥作用的地方，这些服务是由systemd控制的进程。它们可以是通常意义上的服务，比如一个http服务器或者ssh守护进程，它们在系统启动时启动（通过将它们添加为`default.target`的依赖项）。但是一个systemd服务也可以是任何其他程序；例如，你可以通过将一个服务作为`sleep.target`的依赖项，在系统进入睡眠时启动一个屏幕锁定。然后当系统再次唤醒时，屏幕锁定仍然在运行并需要输入密码才能使用系统。因此，通过将服务单元作为目标的依赖项，当目标变为活动时程序将被执行。粗略地说，通过定义一个服务，我们告诉systemd*要做什么*，通过将其作为目标的依赖项，我们告诉systemd*什么时候*去做。

我们最小示例中最终需要的单元类型是套接字；稍后我们会回到这一点。

### 从一个干净的板开始

我们想要创建一个最小的systemd示例，这意味着尽可能少的单元。在一个真实的系统上，有很多单元文件，例如，在我的机器上`/lib/systemd/system`中有355个条目。我们将摆脱所有这些，并从零开始。

我们将使用Ubuntu基础镜像，在其上安装systemd，然后删除所有单元文件。

```
FROM ubuntu:20.04   RUN apt-get update && \  apt-get install --assume-yes systemd   RUN rm -rf /lib/systemd/system/* /etc/systemd/system/*   CMD ["/lib/systemd/systemd"] 
```

Dockerfile

让我们用以下命令构建镜像

```
podman build --tag systemd . 
```

然后用以下命令运行容器

```
podman run --tty --rm --name systemd systemd 
```

我们使用`--name systemd`，这样我们稍后可以更轻松地访问容器；`--rm`，这样当我们停止容器时它会自动删除，这使我们更轻松地进行示例迭代；以及`--tty`以查看容器的输出（感谢Даниил Леонтьев指出早期版本中缺少的最后一个参数）。请注意，我们使用`podman`而不是`docker`作为容器运行时，因为它内置支持运行systemd；更多详情请参阅我上面链接的文章。

运行容器失败，并显示以下输出。

```
[...]
Unit default.target not found.
Falling back to rescue.target.
Unit rescue.target not found.
[!!!!!!] Failed to load rescue.target.
Exiting PID 1... 
```

因此，我们有点过分了，移除了太多东西。但是 systemd 也告诉了我们缺少什么。如我之前所述，`default.target` 是系统启动时激活的目标。所以让我们将它添加到我们的系统中。

### 使系统启动

在真实的系统中，`default.target` 通常是一个符号链接到另一个目标单元。例如，在我的系统上，它指向 `graphical.target`。该目标依赖于一个启动 Gnome 显示管理器的服务，因此当我的系统启动时，我会看到图形化登录界面。它还有其他依赖项。非常多。我们可以使用 `systemctl` 查看它们，命令如下

```
systemctl list-dependencies graphical.target 
```

在我的机器上，这列出了 180 个依赖项。它们并非全部是直接依赖项；大多数是依赖项的依赖项，或者那些依赖项的依赖项，依此类推。正如我之前提到的，真实的系统可能会令人不知所措。

对于我们的示例，让我们从零依赖项开始。我们还将使用一个简单的文件来作为 `default.target`，而不是一个符号链接。这是我能想到的最简单的目标。

```
[Unit] Description=A minimal default target 
```

`default.target`

单元文件都是按照这种结构的。它们包含节（例如 `[Unit]`）和键值对，称为*指令*。您可以在手册页中了解每种单元类型的存在哪些指令（例如，`man systemd.service` 查看服务的指令，或者 `man systemd.unit` 查看所有单元都可用的指令；如果您有指令并想知道它在哪个手册页中定义，请使用 `man systemd.directives`）。

通过添加一行将目标添加到镜像中

```
COPY default.target /lib/systemd/system/ 
```

到 Dockerfile 中，构建并启动容器，完成！一个成功运行 systemd 的容器。

```
Welcome to Ubuntu 20.04.2 LTS!
Set hostname to <7f02fa832078>.
[  OK  ] Reached target A minimal default target.
Startup finished in 45ms. 
```

### 使系统停止

现在 systemd 启动起来了。但是停止…… 情况就不一样了。运行

在容器中引发错误：

```
Failed to enqueue halt.target job: Unit halt.target not found. 
```

它也不会退出进程。它在十秒钟后什么都不做，然后 `podman` 向进程发送一个 `SIGKILL`。

让我们试着解决这个问题。首先，我们遵循 systemd 给出的提示，再次添加一个 `halt.target` 文件，保持尽可能简单。

```
[Unit] Description=A minimal halt target 
```

`halt.target`

（记得将它添加到 Dockerfile 中，这样它就会被复制到镜像中）。如果我们运行这个容器并再次停止它，关于缺少 `halt.target` 的错误消息就消失了（进展！），但是进程仍然需要十秒钟才能终止。这里发生了什么？如我之前所说，目标本身几乎没用，我们需要将一个服务作为依赖项，该服务执行实际的工作。

所以让我们创建我们的第一个服务。关闭 systemd 的命令是 `systemctl --force halt`，我们可以将它编码到我们的服务单元中，如下所示。

```
[Unit] Description=Halt systemd DefaultDependencies=no   [Service] ExecStart=systemctl --force halt 
```

`halt.service`

这个单元包含一个服务节，其中 `ExecStart=` 指令告诉 systemd 当服务启动时要执行的命令（还有 `ExecStop=` 用于当服务停止时执行命令，以及 `ExecReload=` 用于重新加载服务时执行的命令）。`DefaultDependencies=` 指令也是新的，我们稍后会详细讨论。

现在我们需要告诉 systemd 在达到 halt.target 时执行 `halt.service`。我们通过在目标单元文件中添加 `Requires=` 指令将 `halt.service` 添加为 `halt.target` 的依赖项来完成这一点。（我们将在后续文章中详细介绍定义依赖关系的更多细节。）

```
[Unit] Description=A minimal halt target Requires=halt.service 
```

halt.target

而且，当我们运行此容器并尝试停止它时，这次进程立即退出。在退出之前还会发出警告：

```
halt.service: Failed to connect stdout to the journal socket, ignoring: No such file or directory 
```

我们将在下一节中讨论这个问题。但首先，让我们回到 `DefaultDependencies=no` 行。我们希望一个最小的示例，那么当我们移除它时会发生什么？如果我们这样做，并且通过构建、启动和停止容器的流程，我们会得到消息

```
Failed to enqueue halt.target job: Unit sysinit.target not found. 
```

并且进程继续运行。原因是服务也可以有依赖关系（实际上，每个 systemd 单元都可以依赖于任何其他单元），并且 systemd 默认为每个服务添加一些依赖关系。其中一个默认依赖关系是对 `sysinit.target` 的要求。在真实系统上，此目标确保系统已正确设置（通过依赖于执行实际设置的单元）。在我们的系统中，此目标不存在，因此服务失败，因为无法满足要求。添加 `DefaultDependencies=no` 告诉 systemd 不要添加这些默认依赖关系。

### 添加一个日志系统

systemd 在上面抱怨缺少日志套接字。这个套接字是 journald 的一部分，是 systemd 的日志框架。它接收来自不同来源的日志消息，如内核日志和系统日志。它还接收系统服务的标准输出(stdout)和标准错误(stderr)的所有内容：systemd 默认将这两者连接到日志中。一旦日志消息存储在 journald 中，我们可以使用 `journalctl` 查询它们：我们可以显示存储在日志中的所有日志消息，或者我们可以细化我们的查询，仅显示某个时间段内的日志或属于某个服务的日志。这非常有用，特别是当出现问题并且我们想找出原因时。

要启动 journald，我们需要一个服务。这类似于上面的 halt 服务：我们提供应该执行的命令，然后将服务作为目标的依赖项添加进去；这次是默认的目标，因为我们希望 journald 在容器“启动”后启动。但除了服务之外，我们还需要其他东西：一个套接字单元。

在“经典”的服务中，例如 PostgreSQL 服务器，套接字是在服务本身中设置的。可以通过命令行设置或配置文件进行配置，但实际设置是由服务完成的。例如，我可以执行

```
postgres -p 2345 -D test.db 
```

并且它将创建一个监听 TCP 端口 2345 的 PostgreSQL 服务器。

systemd将套接字与服务解绑。它将套接字作为一个可以独立存在的概念，这使得例如在没有运行服务的情况下打开套接字成为可能，只有在套接字上有流量时才启动服务（我们将在后续文章中看到这一点）。在套接字单元文件中，我们可以指定不同的套接字类型进行监听，如文件系统套接字或IPv4或IPv6套接字。对于journald，我们创建一个带有两个文件套接字的套接字单元，一个流套接字和一个数据报套接字。这些的文件名由systemd定义；我们无法更改它们，否则尝试向journald记录日志的服务将失败。

```
[Unit] Description=Journal Socket DefaultDependencies=no   [Socket] ListenStream=/run/systemd/journal/stdout ListenDatagram=/run/systemd/journal/socket 
```

`systemd-journald.socket`

然后我们创建一个服务单元，指定应传递的套接字单元。

```
[Unit] Description=Journal Service DefaultDependencies=no   [Service] ExecStart=/lib/systemd/systemd-journald Sockets=systemd-journald.socket 
```

`systemd-journald.service`

最后，我们将服务作为默认目标的依赖项添加。

```
[Unit] Description=A minimal default target Requires=systemd-journald.service 
```

`default.target`

有了这个，当我们运行容器时，journald会自动启动。

```
Welcome to Ubuntu 20.04.2 LTS!

Set hostname to <311635618eb2>.
[  OK  ] Reached target A minimal default target.
[  OK  ] Listening on Journal Socket.
[  OK  ] Started Journal Service. 
```

我们可以在容器上执行`journalctl`以查看已记录的内容：

```
podman exec systemd journalctl 
```

```
-- Logs begin at Tue 2021-11-16 19:23:40 UTC, end at Tue 2021-11-16 19:23:40 UTC. --
Nov 16 19:23:40 311635618eb2 systemd-journald[15]: Journal started
Nov 16 19:23:40 311635618eb2 systemd-journald[15]: Runtime Journal (/run/log/journal/93a63408ad78310aa88bd408619404da) is 8.0M, max 788.1M, 780.1M free.
Nov 16 19:23:40 311635618eb2 systemd: Startup finished in 43ms. 
```

这不多，只是来自journald本身的两个日志声明和一个来自systemd的日志声明。但如果我们有其他服务，它们的输出也会显示在这里。

最后，在关闭容器时，它会立即关闭而没有警告。

### 未来的保障

最后，我们将添加一个`sysinit`目标。这并非严格必需，但如果我们想稍后添加更多服务，它会很有用。所有服务都默认依赖于此目标，因此如果缺少它，服务将无法启动；我们在`halt.service`中已经看到了这一点。

在真实系统中，`sysinit.target`在引导过程中被激活，并且有很多直接和间接的依赖关系负责设置系统。对我们来说，不需要任何设置，所以我们保持尽可能简单。

```
[Unit] Description=Empty sysinit target 
```

`sysinit.target`

然后我们将其添加为`default.target`的依赖项。（在实际系统中，它不是直接的依赖项，而是依赖于依赖于依赖等等的依赖项。）

```
[Unit] Description=A minimal default target Requires=systemd-journald.service sysinit.target 
```

`default.target`

就是这样！一个功能完备的、最小化的systemd设置。

### 结论

让我们回顾一下我们所创建的内容。我们有一个`default.target`，在容器启动时被激活。这个目标引入了`systemd-journald.service`，设置了journald（借助于`systemd-journald.socket`），以便服务有东西可以记录。它还引入了`sysinit.target`，这将使我们能够轻松地添加服务单元。另一方面，我们有`halt.target`，在系统准备关闭时被激活，并引入了`halt.service`来执行实际的关闭操作。

这些文件总共不到30行；以下是它们的完整内容（您也可以在[GitHub](https://github.com/sgrj/systemd-by-example/tree/main/minimization)上找到它们）。

```
[Unit] Description=A minimal default target Requires=systemd-journald.service sysinit.target 
```

`default.target`

```
[Unit] Description=Journal Service DefaultDependencies=no   [Service] ExecStart=/lib/systemd/systemd-journald Sockets=systemd-journald.socket 
```

`systemd-journald.service`

```
[Unit] Description=Journal Socket DefaultDependencies=no   [Socket] ListenStream=/run/systemd/journal/stdout ListenDatagram=/run/systemd/journal/socket 
```

`systemd-journald.socket`

```
[Unit] Description=Empty sysinit target 
```

`sysinit.target`

```
[Unit] Description=A minimal halt target Requires=halt.service 
```

`halt.target`

```
[Unit] Description=Halt systemd DefaultDependencies=no   [Service] ExecStart=systemctl --force halt 
```

`halt.service`

这六个单元足以使systemd运行，尽管勉强如此。当然，这种设置远非用于生产环境，因为它缺少组成真正系统和使其可靠所需的许多部分。它被有意设计为最小主义，这对理解新概念很有帮助。

从这里，我们现在可以朝多个方向前进。一个方向是进一步使用`systemctl`工具调查我们的最小系统。如果你执行

```
podman exec systemd systemctl list-units --all 
```

当你看到systemd拥有的单元多于我们定义的六个时，你可以尝试找出它们的用途。或者，如果你执行

```
podman exec systemd systemctl list-dependencies default.target --all 
```

你将看到`default.target`的所有依赖项，包括所有传递依赖关系。同样，它们比我们明确定义的要多，因此你可以沿着这条路径继续探索。

另一个方向是向系统添加更多单元，以了解systemd的不同方面。这是我将在未来的文章中做的事情，敬请关注。

—由Sebastian Jambor撰写。关注我的Mastodon账号[@crepels@mastodon.social](https://mastodon.social/@crepels)，获取新博文的更新信息。
