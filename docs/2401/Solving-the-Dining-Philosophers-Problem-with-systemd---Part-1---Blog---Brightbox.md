<!--yml

类别：未分类

日期：2024 年 5 月 27 日 14:54:18

-->

# 使用 systemd 解决哲学家就餐问题 - 第一部分 - 博客 - Brightbox

> 来源：[`www.brightbox.com/blog/2024/01/10/solving-dining-philosophers-with-systemd-part-1/`](https://www.brightbox.com/blog/2024/01/10/solving-dining-philosophers-with-systemd-part-1/)

# 使用 systemd 解决哲学家就餐问题 - 第一部分

由**Neil Wilson**发布 • **2024 年 1 月 10 日**

多年来，开发人员似乎已经忘记了 Unix 是一个编程环境，而不仅仅是你必须安装才能访问 Web 服务器的东西。Unix 是一个强大的多用户系统，拥有许多经受时间考验的工具，并且随着 systemd 的出现，我们有了一个可以轻松建模状态机的新工具。现在，我们可以仅仅使用 Unix 来解决新类别的问题。

在本博客系列中，我们将使用 Unix 编程环境中一些被遗忘的机制和工具、systemd 的一些高级功能以及一些 shell 脚本来解决经典的[“哲学家就餐问题”](https://zh.wikipedia.org/wiki/%E5%93%B2%E5%AD%A6%E5%AE%B6%E5%B0%B1%E9%A4%90%E9%97%AE%E9%A2%98)并且全部操作不会出现死锁。

让我们首先构建一个简单的状态机，使用用户级别的 systemd 设施在`思考`、`饥饿`和`吃饭`的状态之间进行转换。

## 创建服务器

在这些示例中，我们将使用 Brightbox 上的 Ubuntu，但是任何运行 systemd 的现代发行版都应该可以工作。

创建一个服务器并用默认的`ubuntu`用户登录：

```
$ ssh ubuntu@public.srv-hgy08.gb1.brightbox.com
Welcome to Ubuntu 23.10 (GNU/Linux 6.5.0-10-generic x86_64)
...
```

当您登录时，systemd 会自动启动一个用户实例来管理用户级别的单元。您可以通过 systemctl 查看这一点：

```
ubuntu@srv-hgy08:~$ systemctl --user status
● srv-hgy08
    State: running
    Units: 171 loaded (incl. loaded aliases)
     Jobs: 0 queued
   Failed: 0 units
    Since: Mon 2023-11-20 09:41:56 UTC; 1min 19s ago
  systemd: 253.5-1ubuntu6
   CGroup: /user.slice/user-1000.slice/user@1000.service
           └─init.scope
             ├─857 /lib/systemd/systemd --user
             └─858 "(sd-pam)"
```

## 创建状态

我们可以使用 systemd 目标单元来模拟哲学家就餐问题的三个状态。通过声明它们彼此冲突，我们可以使目标相互排斥：

```
# thinking.target [Unit]
Description=Thinking...
Conflicts=eating.target hungry.target
```

这意味着任何时候只有一个目标可以保持活动状态。为了使一个目标转移到另一个目标，我们需要一个计时器单元。

```
# thinking.timer [Unit]
Description=Thinking Period
PartOf=thinking.target

[Timer]
Unit=hungry.target
RandomizedDelaySec=60
OnActiveSec=0
AccuracySec=1us
RemainAfterElapse=false
```

我们将计时器单元设为目标的一部分，以便它在目标停止时也停止。这个`RandomizedDelay`、`OnActive`和`Accuracy`的组合将转换时间从 0 到 60 秒分布。

我们想知道哲学家在想些什么，我们可以通过运行一个调用脚本的服务来模拟。

```
# contemplation.service [Unit]
Description=Contemplation
PartOf=thinking.target
Before=hungry.target

[Service]
Type=oneshot
SyslogFacility=local0
ExecStart=/usr/local/bin/contemplation
```

我们将脚本的输出记录到不同的 syslog 设施中，这样可以让我们稍后在日志中轻松找到它们。一个`Before`限制确保哲学家在处理饥饿之前记录他们的想法。

创建冥想脚本来输出适当的想法：

```
#!/bin/sh
# /usr/local/bin/contemplation
set -e

default_thought="the enigmatic nature of reality:
the existential grasp for an elusive fork, the categorical imperative
of simulated existence, and the irony in questioning the true essence
of the held object."
{
    printf '%s' 'Currently contemplating '
    printf '%s\n' "${default_thought}"
} | fmt
exit 0
```

现在，我们更新目标以引入这些新单元：

```
# thinking.target [Unit]
Description=Thinking...
Conflicts=eating.target hungry.target
Wants=thinking.timer contemplation.service
```

并重复这个过程来满足吃和饥饿的目标。

## 运行状态机

要开始模拟，请将文件放在`$HOME/.config/systemd/user`并激活`thinking`目标：

```
$ systemctl --user start thinking.target
```

查看日志以查看模拟进度：

```
$ journalctl -f
Nov 20 10:49:15 srv-hgy08 systemd[1127]: Started thinking.timer - Thinking Period.
Nov 20 10:49:15 srv-hgy08 systemd[1127]: Stopped target eating.target - Eating....
Nov 20 10:49:15 srv-hgy08 systemd[1127]: Stopped eating.timer - Eating Period.
Nov 20 10:49:15 srv-hgy08 systemd[1127]: Starting contemplation.service - Contemplation...
Nov 20 10:49:15 srv-hgy08 contemplation[2163]: Currently contemplating the enigmatic nature of reality: the existential
Nov 20 10:49:15 srv-hgy08 contemplation[2163]: grasp for an elusive fork, the categorical imperative of simulated
Nov 20 10:49:15 srv-hgy08 contemplation[2163]: existence, and the irony in questioning the true essence of the held
Nov 20 10:49:15 srv-hgy08 contemplation[2163]: object.
Nov 20 10:49:15 srv-hgy08 systemd[1127]: Finished contemplation.service - Contemplation.
Nov 20 10:49:15 srv-hgy08 systemd[1127]: Reached target thinking.target - Thinking....
Nov 20 10:49:17 srv-hgy08 systemd[1127]: Started hungry.timer - Hungry Period.
Nov 20 10:49:17 srv-hgy08 systemd[1127]: Stopped target thinking.target - Thinking....
Nov 20 10:49:17 srv-hgy08 systemd[1127]: Stopped thinking.timer - Thinking Period.
Nov 20 10:49:17 srv-hgy08 systemd[1127]: Starting hunger.service - Hunger...
Nov 20 10:49:17 srv-hgy08 hunger[2165]: Getting hungry...
Nov 20 10:49:17 srv-hgy08 systemd[1127]: Finished hunger.service - Hunger.
Nov 20 10:49:17 srv-hgy08 systemd[1127]: Reached target hungry.target - Hungry....
Nov 20 10:49:17 srv-hgy08 systemd[1127]: Started eating.timer - Eating Period.
Nov 20 10:49:17 srv-hgy08 systemd[1127]: Stopped target hungry.target - Hungry....
Nov 20 10:49:17 srv-hgy08 systemd[1127]: Stopped hungry.timer - Hungry Period.
Nov 20 10:49:17 srv-hgy08 consumption[2166]: Eating...
Nov 20 10:49:17 srv-hgy08 systemd[1127]: Starting eating-food.service - Eating Food...
Nov 20 10:49:17 srv-hgy08 systemd[1127]: Finished eating-food.service - Eating Food.
Nov 20 10:49:17 srv-hgy08 systemd[1127]: Reached target eating.target - Eating....
```

我们可以通过仅查看来自我们的脚本的输出来过滤状态噪音：

```
$ journalctl -f --facility local0
Nov 20 10:50:44 srv-hgy08 contemplation[2176]: Currently contemplating the enigmatic nature of reality: the existential
Nov 20 10:50:44 srv-hgy08 contemplation[2176]: grasp for an elusive fork, the categorical imperative of simulated
Nov 20 10:50:44 srv-hgy08 contemplation[2176]: existence, and the irony in questioning the true essence of the held
Nov 20 10:50:44 srv-hgy08 contemplation[2176]: object.
Nov 20 10:51:14 srv-hgy08 hunger[2177]: Getting hungry...
Nov 20 10:51:15 srv-hgy08 consumption[2178]: Eating...
Nov 20 10:51:18 srv-hgy08 contemplation[2181]: Currently contemplating the enigmatic nature of reality: the existential
Nov 20 10:51:18 srv-hgy08 contemplation[2181]: grasp for an elusive fork, the categorical imperative of simulated
Nov 20 10:51:18 srv-hgy08 contemplation[2181]: existence, and the irony in questioning the true essence of the held
Nov 20 10:51:18 srv-hgy08 contemplation[2181]: object.
```

或者，如果我们只想查看特定的一个脚本，我们可以按标识符进行过滤：

```
$ journalctl -f --identifier=contemplation
Nov 20 10:51:18 srv-hgy08 contemplation[2181]: existence, and the irony in questioning the true essence of the held
Nov 20 10:51:18 srv-hgy08 contemplation[2181]: object.
Nov 20 10:52:11 srv-hgy08 contemplation[2192]: Currently contemplating the enigmatic nature of reality: the existential
Nov 20 10:52:11 srv-hgy08 contemplation[2192]: grasp for an elusive fork, the categorical imperative of simulated
Nov 20 10:52:11 srv-hgy08 contemplation[2192]: existence, and the irony in questioning the true essence of the held
Nov 20 10:52:11 srv-hgy08 contemplation[2192]: object.
Nov 20 10:52:37 srv-hgy08 contemplation[2197]: Currently contemplating the enigmatic nature of reality: the existential
Nov 20 10:52:37 srv-hgy08 contemplation[2197]: grasp for an elusive fork, the categorical imperative of simulated
Nov 20 10:52:37 srv-hgy08 contemplation[2197]: existence, and the irony in questioning the true essence of the held
Nov 20 10:52:37 srv-hgy08 contemplation[2197]: object.
```

要停止模拟，请停止目标：

```
$ systemctl --user stop thinking.target eating.target hungry.target
```

或者退出。

## 摘要

在这篇博客中，我们使用 systemd 创建了一个简单的状态机，它可以在一段时间内整洁地在各个状态之间移动，并将其活动记录在日志中。这几乎不是一个完全建模的哲学家，但这是一个很好的开端。

在下一部分中，我们将扩展这个简单的模型，涵盖多个哲学家，并介绍共享的用于他们思考的餐厅。
