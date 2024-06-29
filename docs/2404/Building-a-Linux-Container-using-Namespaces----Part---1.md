<!--yml

category: 未分类

date: 2024-05-27 13:09:22

-->

# 构建 Linux 容器使用 Namespaces :: 第一部分

> 来源：[https://www.polarsparc.com/xhtml/Containers-1.html](https://www.polarsparc.com/xhtml/Containers-1.html)

构建 Linux 容器使用 Namespaces :: 第一部分

* * *

你是否曾经想过 Linux 容器是如何工作的？

目前，[Docker](https://www.polarsparc.com/xhtml/Docker.html) 是最流行和普遍使用的容器实现之一。

容器运行在相同的操作系统内核上，但将其内部运行的应用程序进程互相隔离。容器背后的秘密之一是 [Namespaces](http://man7.org/linux/man-pages/man7/namespaces.7.html)。

命名空间将全局系统资源，如主机名、用户 ID、组 ID、进程 ID、网络端口等，在进程看来抽象为它们拥有自己的隔离实例。命名空间的主要目标之一是支持容器的实现（轻量级虚拟化）。

目前，Linux 中存在 6 种类型的命名空间 — IPC、网络、挂载、PID、用户和 UTS。

下面是每个命名空间的简要描述：

+   IPC :: 此命名空间隔离了某些进程间通信（IPC）资源，即消息队列、信号量和共享内存。

+   网络 :: 此命名空间提供了与网络相关的系统资源隔离，例如网络设备、IP 地址、IP 路由表、/proc/net 目录、端口号等。

+   Mount :: 此命名空间隔离了一组进程所看到的文件系统挂载点集合。不同挂载命名空间中的进程可以有不同的文件系统层次结构视图。

+   PID :: 此命名空间隔离了进程 ID 号空间。这使得不同 PID 命名空间中的进程可以拥有相同的 PID。

+   用户 :: 此命名空间隔离了用户和组 ID 号空间，使得进程的用户和组 ID 在用户命名空间内外可以不同。

+   UTS :: 此命名空间隔离了两个系统标识符 — 主机名和域名。对于容器来说，UTS 命名空间允许每个容器拥有自己的主机名和 NIS 域名。

在本文的演示中，我们将使用 unshare Linux 命令，以及使用 golang 实现、构建和执行一个简单的容器。

安装在基于 Ubuntu 18.04 LTS 的 Linux 桌面系统上。

我们将需要两个命令 newuidmap 和 newgidmap 来演示用户命名空间。为此，我们需要安装 uidmap 软件包。

要安装 uidmap 软件包，请执行以下命令：

$ sudo apt install -y uidmap

接下来，我们将需要 brctl 命令来创建一个桥接网络接口。为此，我们需要安装 bridge-utils 软件包。

要安装 bridge-utils 软件包，请执行以下命令：

$ sudo apt install -y bridge-utils

要开发、构建和执行go编程语言中的简单容器，我们需要安装golang包。

要检查可安装的golang版本，请执行以下命令：

$ sudo apt-cache policy golang

以下将是典型的输出：

#### 输出。1

```
golang:
  Installed: (none)
  Candidate: 2:1.13~1ubuntu1ppa1~bionic
  Version table:
 *** 2:1.13~1ubuntu1ppa1~bionic 500
        500 http://ppa.launchpad.net/hnakamur/golang-1.13/ubuntu bionic/main amd64 Packages
        500 http://ppa.launchpad.net/hnakamur/golang-1.13/ubuntu bionic/main i386 Packages
        100 /var/lib/dpkg/status
     2:1.10~4ubuntu1 500
        500 http://archive.ubuntu.com/ubuntu bionic/main amd64 Packages
```

要安装golang，请执行以下命令：

$ sudo apt install -y golang

上述安装过程从官方Ubuntu存储库安装golang。

通过执行以下命令为开发、构建和运行go程序创建一个目录：

$ mkdir $HOME/projects/go

$ export GOPATH=$HOME/projects/go

我们需要一个名为netlink的流行go包进行网络连接。

要下载go包，请执行以下命令：

$ go get github.com/vishvananda/netlink

打开两个终端窗口 - 我们将分别称它们为TA和TB。TB是我们将演示简单容器的地方。

我们需要下载一个作为简单容器基础镜像的最小根文件系统（rootfs）。对于我们的演示，我们将选择此文章撰写时的最新[Ubuntu Base 18.04.4 LTS](http://cdimage.ubuntu.com/ubuntu-base/releases/18.04.4/release/ubuntu-base-18.04.4-base-amd64.tar.gz)。

我们将假设最新的Ubuntu Base已下载到目录$HOME/Downloads。

unshare命令将指定的程序与从父进程隔离的指定命名空间一起执行。

要显示unshare命令的选项，请在TA中执行以下命令：

以下将是典型的输出：

#### 输出。2

```
Usage:
 unshare [options] [<program> [<argument>...]]

Run a program with some namespaces unshared from the parent.

Options:
 -m, --mount[=<file>]      unshare mounts namespace
 -u, --uts[=<file>]        unshare UTS namespace (hostname etc)
 -i, --ipc[=<file>]        unshare System V IPC namespace
 -n, --net[=<file>]        unshare network namespace
 -p, --pid[=<file>]        unshare pid namespace
 -U, --user[=<file>]       unshare user namespace
 -C, --cgroup[=<file>]     unshare cgroup namespace
 -f, --fork                fork before launching <program>
     --mount-proc[=<dir>]  mount proc filesystem first (implies --mount)
 -r, --map-root-user       map current user to root (implies --user)
     --propagation slave|shared|private|unchanged
                           modify mount propagation in mount namespace
 -s, --setgroups allow|deny  control the setgroups syscall in user namespaces

 -h, --help                display this help
 -V, --version             display version
```

每个进程（带有[PID]）都有一个子目录/proc/[PID]/ns，其中包含每个命名空间的一个条目。

要列出与进程相关的所有命名空间，请在TA中执行以下命令：

以下将是典型的输出：

#### 输出。3

```
total 0
lrwxrwxrwx 1 alice alice 0 Mar  7 12:17 cgroup -> 'cgroup:[4026531835]'
lrwxrwxrwx 1 alice alice 0 Mar  7 12:17 ipc -> 'ipc:[4026531839]'
lrwxrwxrwx 1 alice alice 0 Mar  7 12:17 mnt -> 'mnt:[4026531840]'
lrwxrwxrwx 1 alice alice 0 Mar  7 12:17 net -> 'net:[4026531993]'
lrwxrwxrwx 1 alice alice 0 Mar  7 12:17 pid -> 'pid:[4026531836]'
lrwxrwxrwx 1 alice alice 0 Mar  7 20:41 pid_for_children -> 'pid:[4026531836]'
lrwxrwxrwx 1 alice alice 0 Mar  7 12:17 user -> 'user:[4026531837]'
lrwxrwxrwx 1 alice alice 0 Mar  7 12:17 uts -> 'uts:[4026531838]'
```

要启动一个简单容器，其主机名与父主机名隔离，请在TB中执行以下命令：

$ sudo unshare -u /bin/sh

-u选项启用UTS命名空间。

命令提示符将更改为#。

要检查简单容器的PID，请在TB中执行以下命令：

以下将是典型的输出：

要列出与简单容器相关的所有命名空间，请在TB中执行以下命令：

以下将是典型的输出：

#### 输出。5

```
total 0
lrwxrwxrwx 1 root root 0 Mar  7 12:36 cgroup -> 'cgroup:[4026531835]'
lrwxrwxrwx 1 root root 0 Mar  7 12:36 ipc -> 'ipc:[4026531839]'
lrwxrwxrwx 1 root root 0 Mar  7 12:36 mnt -> 'mnt:[4026531840]'
lrwxrwxrwx 1 root root 0 Mar  7 12:36 net -> 'net:[4026531993]'
lrwxrwxrwx 1 root root 0 Mar  7 12:36 pid -> 'pid:[4026531836]'
lrwxrwxrwx 1 root root 0 Mar  7 12:36 pid_for_children -> 'pid:[4026531836]'
lrwxrwxrwx 1 root root 0 Mar  7 12:36 user -> 'user:[4026531837]'
lrwxrwxrwx 1 root root 0 Mar  7 12:36 uts -> 'uts:[4026533064]'
```

比较输出。5和输出。3，我们看到了UTS命名空间的变化，这是预期的和正确的。

要更改简单容器的主机名，请在TB中执行以下命令：

要显示父主机的主机名，请在TA中执行以下命令：

以下将是典型的输出：

要显示简单容器的主机名，请在TB中执行以下命令：

以下将是典型的输出：

这向我们展示了我们已将简单容器的主机名与父主机名隔离开来。

要退出简单容器，请在TB中执行以下命令：

接下来，我们将使用以下go程序模仿上述UTS命名空间隔离：

<fieldset id="sc-fieldset"><legend>清单.1</legend>

```
package main

import (
    "log"
    "os"
    "os/exec"
    "syscall"
)

func execContainerShell() {
    log.Printf("Ready to exec container shell ...\n")

    if err := syscall.Sethostname([]byte("leopard")); err != nil {
       panic(err)
    }

    const sh = "/bin/sh"

    env := os.Environ()
    env = append(env, "PS1=-> ")

    if err := syscall.Exec(sh, []string{""}, env); err != nil {
        panic(err)
    }
}

func main() {
    log.Printf("Starting process %s with args: %v\n", os.Args[0], os.Args)

    const clone = "CLONE"

    if len(os.Args) > 1 && os.Args[1] == clone {
        execContainerShell()
    }

    log.Printf("Ready to run command ...\n")

    cmd := exec.Command(os.Args[0], []string{clone}...)
    cmd.Stdin = os.Stdin
    cmd.Stdout = os.Stdout
    cmd.Stderr = os.Stderr
    cmd.SysProcAttr = &syscall.SysProcAttr{
        Cloneflags: syscall.CLONE_NEWUTS,
    }

    if err := cmd.Run(); err != nil {
        panic(err)
    }
}
```

</fieldset>

exec包中的Command函数允许运行指定的命令（第一个参数）并使用提供的参数（第二个参数）。它返回Cmd结构体的实例。

可以在返回的Cmd实例上设置标准输入（os.Stdin）、标准输出os.Stdout、标准错误os.Stderr以及一些操作系统特定的属性。在本例中，我们指定syscall.CLONE_NEWUTS操作系统属性以指示在新的UTS命名空间中运行命令。

重要提示：当主进程启动时，它会在新的命名空间中内部生成另一个主进程（带有CLONE参数）。正是这个在新命名空间中运行的生成的主进程，通过调用execContainerShell函数将其覆盖（syscall.Exec）为shell命令。

通过在TB中执行以下命令创建并切换到目录$GOPATH/uts：

$ mkdir -p $GOPATH/uts

$ cd $GOPATH/uts

将以上代码复制到当前目录中的程序文件main.go中。

要编译程序文件main.go，请在TB中执行以下命令：

要运行主程序，请在TB中执行以下命令：

下面是典型的输出：

#### 输出.8

```
2020/03/07 12:49:11 Starting process ./main with args: [./main]
2020/03/07 12:49:11 Ready to run command ...
2020/03/07 12:49:11 Starting process ./main with args: [./main CLONE]
2020/03/07 12:49:11 Ready to exec container shell ...
->
```

命令提示符将变更为->。

要显示简单容器的主机名，请在TB中执行以下命令：

下面是典型的输出：

要退出简单容器，请在TB中执行以下命令：

成功！！！我们已经展示了使用unshare命令和简单go程序的UTS命名空间。

让我们将用户命名空间层叠在UTS命名空间上。

要启动一个简单容器，其用户/组ID以及主机名与父命名空间隔离，请在TB中执行以下命令：

$ sudo unshare -uU /bin/sh

-U选项启用了用户命名空间。

要在新的命名空间中显示用户ID和组ID，请在TB中执行以下命令：

下面是典型的输出：

#### 输出.10

```
uid=65534(nobody) gid=65534(nogroup) groups=65534(nogroup)
```

创建用户命名空间时，默认情况下，新命名空间中的用户/组ID不会映射到父命名空间的用户/组ID。未映射的用户/组ID将分配默认值溢出用户/组ID。溢出用户ID的默认值从/proc/sys/kernel/overflowuid读取（为65534）。类似地，溢出组ID的默认值从/proc/sys/kernel/overflowgid读取（为65534）。

要修复用户/组ID到父用户/组ID的映射，请通过在TB中执行以下命令退出简单容器：

要使用当前有效的用户/组ID映射到新命名空间中的超级用户/组ID重新启动简单容器，请在TB中执行以下命令：

$ sudo unshare -uUr /bin/sh

-r选项启用在新命名空间中用户/组ID到父命名空间用户/组ID的映射。

命令提示符将更改为#。

要显示新命名空间中的用户ID和组ID，请在TB中执行以下命令：

以下是典型的输出：

#### 输出.11

```
uid=0(root) gid=0(root) groups=0(root)
```

要列出与简单容器关联的所有命名空间，请在TB中执行以下命令：

以下是典型的输出：

#### 输出.12

```
total 0
lrwxrwxrwx 1 root root 0 Mar 7 13:09 cgroup -> 'cgroup:[4026531835]'
lrwxrwxrwx 1 root root 0 Mar 7 13:09 ipc -> 'ipc:[4026531839]'
lrwxrwxrwx 1 root root 0 Mar 7 13:09 mnt -> 'mnt:[4026531840]'
lrwxrwxrwx 1 root root 0 Mar 7 13:09 net -> 'net:[4026531993]'
lrwxrwxrwx 1 root root 0 Mar 7 13:09 pid -> 'pid:[4026531836]'
lrwxrwxrwx 1 root root 0 Mar 7 13:09 pid_for_children -> 'pid:[4026531836]'
lrwxrwxrwx 1 root root 0 Mar 7 13:09 user -> 'user:[4026532892]'
lrwxrwxrwx 1 root root 0 Mar 7 13:09 uts -> 'uts:[4026533401]'
```

将输出.12与输出.3进行比较，我们看到了UTS命名空间和用户命名空间的变化，这是预期且正确的。

要退出简单容器，请在TB中执行以下命令：

接下来，我们将使用以下go程序模仿上述的UTS和用户命名空间隔离：

<fieldset id="sc-fieldset"><legend>列表.2</legend>

```
package main

import (
    "log"
    "os"
    "os/exec"
    "syscall"
)

func execContainerShell() {
    log.Printf("Ready to exec container shell ...\n")

    if err := syscall.Sethostname([]byte("leopard")); err != nil {
        panic(err)
    }

    const sh = "/bin/sh"

    env := os.Environ()
    env = append(env, "PS1=-> ")

    if err := syscall.Exec(sh, []string{""}, env); err != nil {
        panic(err)
    }
}

func main() {
    log.Printf("Starting process %s with args: %v\n", os.Args[0], os.Args)

    const clone = "CLONE"

    if len(os.Args) > 1 && os.Args[1] == clone {
        execContainerShell()
        os.Exit(0)
    }

    log.Printf("Ready to run command ...\n")

    cmd := exec.Command(os.Args[0], []string{clone}...)
    cmd.Stdin = os.Stdin
    cmd.Stdout = os.Stdout
    cmd.Stderr = os.Stderr
    cmd.SysProcAttr = &syscall.SysProcAttr{
        Cloneflags: syscall.CLONE_NEWUTS | syscall.CLONE_NEWUSER,
        UidMappings: []syscall.SysProcIDMap{
            {ContainerID: 0, HostID: 0, Size: 1},
        },
        GidMappings: []syscall.SysProcIDMap{
            {ContainerID: 0, HostID: 0, Size: 1},
        },
    }

    if err := cmd.Run(); err != nil {
        panic(err)
    }
}
```

</fieldset>

如前所述，Command函数返回Cmd结构的一个实例。

在此示例中，我们指定了额外的syscall.CLONE_NEWUSER操作系统属性，以指示在新的用户命名空间中运行命令。

此外，我们将用户ID映射UidMappings设置为syscall.SysProcIDMap结构条目的数组，每个条目包含容器中的用户ID映射（ContainerID）到主机命名空间中的用户ID（HostID）。在本例中，我们将容器中的根用户ID 0映射到主机命名空间中的根用户ID 0。类似地，我们设置组ID映射GidMappings

通过在TB中执行以下命令来创建并切换到目录$GOPATH/user：

$ mkdir -p $GOPATH/user

$ cd $GOPATH/user

将上述代码复制到当前目录中的程序文件main.go中。

要编译程序文件 main.go，请在TB中执行以下命令：

要运行程序 main，请在TB中执行以下命令：

以下是典型的输出：

#### 输出.13

```
2020/03/07 13:17:02 Starting process ./main with args: [./main]
2020/03/07 13:17:02 Ready to run command ...
2020/03/07 13:17:02 Starting process ./main with args: [./main CLONE]
2020/03/07 13:17:02 Ready to exec container shell ...
->
```

命令提示符将更改为->。

要显示新命名空间中的用户ID和组ID，请在TB中执行以下命令：

以下是典型的输出：

#### 输出.14

```
uid=0(root) gid=0(root) groups=0(root)
```

要列出与简单容器关联的所有命名空间，请在TB中执行以下命令：

以下是典型的输出：

#### 输出.15

```
total 0
lrwxrwxrwx 1 root root 0 Mar 13 21:17 cgroup -> 'cgroup:[4026531835]'
lrwxrwxrwx 1 root root 0 Mar 13 21:17 ipc -> 'ipc:[4026531839]'
lrwxrwxrwx 1 root root 0 Mar 13 21:17 mnt -> 'mnt:[4026531840]'
lrwxrwxrwx 1 root root 0 Mar 13 21:17 net -> 'net:[4026531993]'
lrwxrwxrwx 1 root root 0 Mar 13 21:17 pid -> 'pid:[4026531836]'
lrwxrwxrwx 1 root root 0 Mar 13 21:17 pid_for_children -> 'pid:[4026531836]'
lrwxrwxrwx 1 root root 0 Mar 13 21:17 user -> 'user:[4026532666]'
lrwxrwxrwx 1 root root 0 Mar 13 21:17 uts -> 'uts:[4026532723]'
```

要在简单容器中显示主机名，请在TB中执行以下命令：

以下是典型的输出：

要退出简单容器，请在TB中执行以下命令：

成功！我们展示了使用unshare命令和一个简单的go程序组合UTS和用户命名空间。

现在让我们在用户命名空间和UTS命名空间之上添加PID命名空间。

要启动一个简单容器，其进程ID、用户/组ID和主机名与父命名空间隔离，请在TB中执行以下命令：

$ sudo unshare -uUrpf --mount-proc /bin/sh

-p选项启用PID命名空间。

-f选项启用在新命名空间中生成（或分叉）新进程。

--mount-proc选项在新的命名空间中将proc文件系统作为私有挂载点挂载到/proc。这意味着/proc伪目录仅显示有关该PID命名空间内进程的信息。

#### 注意

```
Ensure the option -f is *SPECIFIED*. Else will encounter the following error:

/bin/sh: 4: Cannot fork
```

命令提示符将变为 #。

要显示新命名空间中的所有进程，请在 TB 中执行以下命令：

典型输出如下：

#### 输出.17

```
USER       PID %CPU %MEM    VSZ   RSS TTY      STAT START   TIME COMMAND
root         1  0.0  0.0   4628   880 pts/1    S    09:08   0:00 /bin/sh
root         6  0.0  0.0  37368  3340 pts/1    R+   09:12   0:00 ps -fu
```

要显示父命名空间中的所有进程，请在 TA 中执行以下命令：

典型输出如下：

#### 输出.18

```
USER         PID %CPU %MEM    VSZ   RSS TTY      STAT START   TIME COMMAND
polarsparc  8695  0.0  0.0  22840  5424 pts/1    Ss   08:43   0:00 bash
polarsparc  8681  0.0  0.0  22708  5096 pts/0    Ss   08:43   0:00 bash
polarsparc  9635  0.0  0.0  37368  3364 pts/0    R+   09:12   0:00  \_ ps -fu
```

比较输出.17 和输出.18，我们可以看到新命名空间与父命名空间之间的隔离，这是预期的正确行为。

要退出简单容器，请在 TB 中执行以下命令：

接下来，我们将使用以下 Go 程序模拟上述 UTS、用户和 PID 命名空间隔离：

`<fieldset id="sc-fieldset"><legend>Listing.3</legend>`

```
package main

import (
    "log"
    "os"
    "os/exec"
    "syscall"
)

func execContainerShell() {
    log.Printf("Ready to exec container shell ...\n")

    if err := syscall.Sethostname([]byte("leopard")); err != nil {
        panic(err)
    }

    if err := syscall.Mount("proc", "/proc", "proc", 0, ""); err != nil {
        panic(err)
    }

    const sh = "/bin/sh"

    env := os.Environ()
    env = append(env, "PS1=-> ")

    if err := syscall.Exec(sh, []string{""}, env); err != nil {
        panic(err)
    }
}

func main() {
    log.Printf("Starting process %s with args: %v\n", os.Args[0], os.Args)

    const clone = "CLONE"

    if len(os.Args) > 1 && os.Args[1] == clone {
        execContainerShell()
        os.Exit(0)
    }

    log.Printf("Ready to run command ...\n")

    cmd := exec.Command(os.Args[0], []string{clone}...)
    cmd.Stdin = os.Stdin
    cmd.Stdout = os.Stdout
    cmd.Stderr = os.Stderr
    cmd.SysProcAttr = &syscall.SysProcAttr{
        Cloneflags: syscall.CLONE_NEWUTS | syscall.CLONE_NEWUSER | syscall.CLONE_NEWNS | syscall.CLONE_NEWPID,
        UidMappings: []syscall.SysProcIDMap{
            {ContainerID: 0, HostID: 0, Size: 1},
        },
        GidMappings: []syscall.SysProcIDMap{
            {ContainerID: 0, HostID: 0, Size: 1},
        },
    }

    if err := cmd.Run(); err != nil {
        panic(err)
    }
}
```

`</fieldset>`

如前所述，Command 函数返回 Cmd 结构体的一个实例。

在本例中，我们指定了附加的 syscall.CLONE_NEWNS 和 syscall.CLONE_NEWPID 操作系统属性，以指示在新 PID 命名空间中运行命令。

通过在 TB 中执行以下命令，创建并切换到目录 $GOPATH/pid：

`$ mkdir -p $GOPATH/pid`

`$ cd $GOPATH/pid`

将以上代码复制到当前目录中的程序文件 main.go。

编译程序文件 main.go，需在 TB 中执行以下命令：

要运行程序 main，请在 TB 中执行以下命令：

典型输出如下：

#### 输出.19

```
2020/03/07 13:38:02 Starting process ./main with args: [./main]
2020/03/07 13:38:02 Ready to run command ...
2020/03/07 13:38:02 Starting process ./main with args: [./main CLONE]
2020/03/07 13:38:02 Ready to exec container shell ...
->
```

命令提示符将变为 ->。

要显示简单容器的主机名，请在 TB 中执行以下命令：

典型输出如下：

要显示新命名空间中的用户 ID 和组 ID，请在 TB 中执行以下命令：

典型输出如下：

#### 输出.21

```
uid=0(root) gid=0(root) groups=0(root)
```

要显示简单容器中的所有进程，请在 TB 中执行以下命令：

典型输出如下：

#### 输出.22

```
USER       PID %CPU %MEM    VSZ   RSS TTY      STAT START   TIME COMMAND
root         1  0.0  0.0   4628   776 pts/1    S    09:41   0:00 
root         6  0.0  0.0  37368  3400 pts/1    R+   09:41   0:00 ps -fu
```

要列出与简单容器关联的所有命名空间，需在 TB 中执行以下命令：

典型输出如下：

#### 输出.23

```
total 0
lrwxrwxrwx 1 root root 0 Mar 14 09:44 cgroup -> 'cgroup:[4026531835]'
lrwxrwxrwx 1 root root 0 Mar 14 09:44 ipc -> 'ipc:[4026531839]'
lrwxrwxrwx 1 root root 0 Mar 14 09:44 mnt -> 'mnt:[4026532366]'
lrwxrwxrwx 1 root root 0 Mar 14 09:44 net -> 'net:[4026531993]'
lrwxrwxrwx 1 root root 0 Mar 14 09:44 pid -> 'pid:[4026532368]'
lrwxrwxrwx 1 root root 0 Mar 14 09:44 pid_for_children -> 'pid:[4026532368]'
lrwxrwxrwx 1 root root 0 Mar 14 09:44 user -> 'user:[4026532365]'
lrwxrwxrwx 1 root root 0 Mar 14 09:44 uts -> 'uts:[4026532367]'
```

要退出简单容器，请在 TB 中执行以下命令：

成功！我们已经演示了使用 unshare 命令和一个简单的 Go 程序结合使用 UTS、用户和 PID 命名空间。

我们现在将在 /tmp 目录中为新命名空间设置最小化的 Ubuntu 基础映像。

要创建并将基础映像复制到 /tmp 目录中的目录，请在 TA 中执行以下命令：

`$ mkdir -p /tmp/rootfs/.old_root`

`$ tar -xvf $HOME/Downloads/ubuntu-base-18.04.4-base-amd64.tar.gz --directory /tmp/rootfs`

`cd /tmp`

现在让我们将 Mount 命名空间叠加到 UTS、用户和 PID 命名空间上。

要启动一个简单容器，其挂载点以及进程 ID、用户/组 ID 和主机名与父命名空间隔离，请在 TB 中执行以下命令：

`$ sudo unshare -uUrpfm --mount-proc /bin/sh`

`-m` 选项启用 Mount 命名空间。

命令提示符将变为 #。

要列出父命名空间中的所有挂载点，请在 TA 中执行以下命令：

`$ cat /proc/mounts | sort`

以下是典型的输出：

#### 输出.24

```
cgroup /sys/fs/cgroup/blkio cgroup rw,nosuid,nodev,noexec,relatime,blkio 0 0
cgroup /sys/fs/cgroup/cpu,cpuacct cgroup rw,nosuid,nodev,noexec,relatime,cpu,cpuacct 0 0
cgroup /sys/fs/cgroup/cpuset cgroup rw,nosuid,nodev,noexec,relatime,cpuset 0 0
cgroup /sys/fs/cgroup/devices cgroup rw,nosuid,nodev,noexec,relatime,devices 0 0
cgroup /sys/fs/cgroup/freezer cgroup rw,nosuid,nodev,noexec,relatime,freezer 0 0
cgroup /sys/fs/cgroup/hugetlb cgroup rw,nosuid,nodev,noexec,relatime,hugetlb 0 0
cgroup /sys/fs/cgroup/memory cgroup rw,nosuid,nodev,noexec,relatime,memory 0 0
cgroup /sys/fs/cgroup/net_cls,net_prio cgroup rw,nosuid,nodev,noexec,relatime,net_cls,net_prio 0 0
cgroup /sys/fs/cgroup/perf_event cgroup rw,nosuid,nodev,noexec,relatime,perf_event 0 0
cgroup /sys/fs/cgroup/pids cgroup rw,nosuid,nodev,noexec,relatime,pids 0 0
cgroup /sys/fs/cgroup/rdma cgroup rw,nosuid,nodev,noexec,relatime,rdma 0 0
cgroup /sys/fs/cgroup/systemd cgroup rw,nosuid,nodev,noexec,relatime,xattr,name=systemd 0 0
cgroup /sys/fs/cgroup/unified cgroup2 rw,nosuid,nodev,noexec,relatime,nsdelegate 0 0
configfs /sys/kernel/config configfs rw,relatime 0 0
debugfs /sys/kernel/debug debugfs rw,relatime 0 0
devpts /dev/pts devpts rw,nosuid,noexec,relatime,gid=5,mode=620,ptmxmode=000 0 0
/dev/sda1 / ext4 rw,relatime,errors=remount-ro,data=ordered 0 0
/dev/sdb1 /home ext4 rw,relatime,data=ordered 0 0
/dev/sdc1 /home/data ext4 rw,relatime,data=ordered 0 0
fusectl /sys/fs/fuse/connections fusectl rw,relatime 0 0
gvfsd-fuse /run/user/1000/gvfs fuse.gvfsd-fuse rw,nosuid,nodev,relatime,user_id=1000,group_id=1000 0 0
hugetlbfs /dev/hugepages hugetlbfs rw,relatime,pagesize=2M 0 0
mqueue /dev/mqueue mqueue rw,relatime 0 0
proc /proc proc rw,nosuid,nodev,noexec,relatime 0 0
pstore /sys/fs/pstore pstore rw,nosuid,nodev,noexec,relatime 0 0
securityfs /sys/kernel/security securityfs rw,nosuid,nodev,noexec,relatime 0 0
sysfs /sys sysfs rw,nosuid,nodev,noexec,relatime 0 0
systemd-1 /proc/sys/fs/binfmt_misc autofs rw,relatime,fd=25,pgrp=1,timeout=0,minproto=5,maxproto=5,direct,pipe_ino=28210 0 0
tmpfs /dev/shm tmpfs rw,nosuid,nodev 0 0
tmpfs /run/lock tmpfs rw,nosuid,nodev,noexec,relatime,size=5120k 0 0
tmpfs /run tmpfs rw,nosuid,noexec,relatime,size=3293620k,mode=755 0 0
tmpfs /run/user/1000 tmpfs rw,nosuid,nodev,relatime,size=3293616k,mode=700,uid=1000,gid=1000 0 0
tmpfs /sys/fs/cgroup tmpfs ro,nosuid,nodev,noexec,mode=755 0 0
udev /dev devtmpfs rw,nosuid,relatime,size=16402556k,nr_inodes=4100639,mode=755 0 0
```

现在，让我们通过在 TB 中执行以下命令列出新命名空间中的所有挂载点：

# cat /proc/mounts | sort

以下是典型的输出：

#### 输出.25

```
cgroup /sys/fs/cgroup/blkio cgroup rw,nosuid,nodev,noexec,relatime,blkio 0 0
cgroup /sys/fs/cgroup/cpu,cpuacct cgroup rw,nosuid,nodev,noexec,relatime,cpu,cpuacct 0 0
cgroup /sys/fs/cgroup/cpuset cgroup rw,nosuid,nodev,noexec,relatime,cpuset 0 0
cgroup /sys/fs/cgroup/devices cgroup rw,nosuid,nodev,noexec,relatime,devices 0 0
cgroup /sys/fs/cgroup/freezer cgroup rw,nosuid,nodev,noexec,relatime,freezer 0 0
cgroup /sys/fs/cgroup/hugetlb cgroup rw,nosuid,nodev,noexec,relatime,hugetlb 0 0
cgroup /sys/fs/cgroup/memory cgroup rw,nosuid,nodev,noexec,relatime,memory 0 0
cgroup /sys/fs/cgroup/net_cls,net_prio cgroup rw,nosuid,nodev,noexec,relatime,net_cls,net_prio 0 0
cgroup /sys/fs/cgroup/perf_event cgroup rw,nosuid,nodev,noexec,relatime,perf_event 0 0
cgroup /sys/fs/cgroup/pids cgroup rw,nosuid,nodev,noexec,relatime,pids 0 0
cgroup /sys/fs/cgroup/rdma cgroup rw,nosuid,nodev,noexec,relatime,rdma 0 0
cgroup /sys/fs/cgroup/systemd cgroup rw,nosuid,nodev,noexec,relatime,xattr,name=systemd 0 0
cgroup /sys/fs/cgroup/unified cgroup2 rw,nosuid,nodev,noexec,relatime,nsdelegate 0 0
configfs /sys/kernel/config configfs rw,relatime 0 0
debugfs /sys/kernel/debug debugfs rw,relatime 0 0
devpts /dev/pts devpts rw,nosuid,noexec,relatime,gid=5,mode=620,ptmxmode=000 0 0
/dev/sda1 / ext4 rw,relatime,errors=remount-ro,data=ordered 0 0
/dev/sdb1 /home ext4 rw,relatime,data=ordered 0 0
/dev/sdc1 /home/data ext4 rw,relatime,data=ordered 0 0
fusectl /sys/fs/fuse/connections fusectl rw,relatime 0 0
gvfsd-fuse /run/user/1000/gvfs fuse.gvfsd-fuse rw,nosuid,nodev,relatime,user_id=1000,group_id=1000 0 0
hugetlbfs /dev/hugepages hugetlbfs rw,relatime,pagesize=2M 0 0
mqueue /dev/mqueue mqueue rw,relatime 0 0
proc /proc proc rw,nosuid,nodev,noexec,relatime 0 0
proc /proc proc rw,nosuid,nodev,noexec,relatime 0 0
pstore /sys/fs/pstore pstore rw,nosuid,nodev,noexec,relatime 0 0
securityfs /sys/kernel/security securityfs rw,nosuid,nodev,noexec,relatime 0 0
sysfs /sys sysfs rw,nosuid,nodev,noexec,relatime 0 0
systemd-1 /proc/sys/fs/binfmt_misc autofs rw,relatime,fd=25,pgrp=0,timeout=0,minproto=5,maxproto=5,direct,pipe_ino=28210 0 0
tmpfs /dev/shm tmpfs rw,nosuid,nodev 0 0
tmpfs /run/lock tmpfs rw,nosuid,nodev,noexec,relatime,size=5120k 0 0
tmpfs /run tmpfs rw,nosuid,noexec,relatime,size=3293620k,mode=755 0 0
tmpfs /run/user/1000 tmpfs rw,nosuid,nodev,relatime,size=3293616k,mode=700,uid=1000,gid=1000 0 0
tmpfs /sys/fs/cgroup tmpfs ro,nosuid,nodev,noexec,mode=755 0 0
udev /dev devtmpfs rw,nosuid,relatime,size=16402556k,nr_inodes=4100639,mode=755 0 0
```

比较输出 Output.25 和 Output.24，我们看到 proc 的一个区别。创建新的挂载命名空间时，新命名空间的挂载点是父命名空间挂载点的副本。

现在我们将演示新命名空间中的任何更改不会影响父命名空间。

要使挂载点 /（及其递归子项）成为新命名空间的私有，请在 TB 中执行以下命令：

# mount --make-rprivate /

在 TB 中，要将挂载点 rootfs/ 递归绑定到新命名空间中的 rootfs/，请执行以下命令：

# mount --rbind rootfs/ rootfs/

我们需要在新命名空间中的 proc 文件系统来进行挂载更改。要在新命名空间中将 /proc 挂载为 proc 文件系统，请在 TB 中执行以下命令：

# mount -t proc proc rootfs/proc

接下来，我们需要将 rootfs/ 设为新命名空间的根文件系统，并使用 pivot_root 命令将父根文件系统移动到 rootfs/.old_root。要执行此操作，请在 TB 中执行以下命令：

# pivot_root rootfs/ rootfs/.old_root

# cd /

要列出父命名空间中 / 目录下的所有文件，请在 TA 中执行以下命令：

以下是典型的输出：

#### 输出.26

```
total 96
drwxr-xr-x   2 root root  4096 Mar  1 10:58 bin
drwxr-xr-x   3 root root  4096 Mar 16 21:15 boot
drwxr-xr-x   2 root root  4096 Sep 13  2019 cdrom
drwxr-xr-x  22 root root  4560 Mar 21 06:59 dev
drwxr-xr-x 163 root root 12288 Mar 20 10:01 etc
drwxr-xr-x   5 root root  4096 Sep 13  2019 home
lrwxrwxrwx   1 root root    33 Mar 16 21:15 initrd.img -> boot/initrd.img-4.15.0-91-generic
lrwxrwxrwx   1 root root    33 Feb 17 14:08 initrd.img.old -> boot/initrd.img-4.15.0-88-generic
drwxr-xr-x  25 root root  4096 Mar 16 13:37 lib
drwxr-xr-x   2 root root  4096 Jul 29  2019 lib64
drwx------   2 root root 16384 Sep 13  2019 lost+found
drwxr-xr-x   3 root root  4096 Nov 10 13:00 media
drwxr-xr-x   2 root root  4096 Jul 29  2019 mnt
drwxr-xr-x   7 root root  4096 Mar 13 08:04 opt
dr-xr-xr-x 328 root root     0 Mar 21 06:59 proc
drwx------   9 root root  4096 Feb 23 13:25 root
drwxr-xr-x  36 root root  1140 Mar 21 07:04 run
drwxr-xr-x   2 root root 12288 Mar 16 13:37 sbin
drwxr-xr-x   2 root root  4096 Jul 29  2019 srv
dr-xr-xr-x  13 root root     0 Mar 21 06:59 sys
drwxrwxrwt  20 root root  4096 Mar 21 11:10 tmp
drwxr-xr-x  11 root root  4096 Jul 29  2019 usr
drwxr-xr-x  11 root root  4096 Jul 29  2019 var
lrwxrwxrwx   1 root root    30 Mar 16 21:15 vmlinuz -> boot/vmlinuz-4.15.0-91-generic
lrwxrwxrwx   1 root root    30 Feb 17 14:08 vmlinuz.old -> boot/vmlinuz-4.15.0-88-generic
```

要列出新命名空间中 / 目录下的所有文件，请在 TB 中执行以下命令：

以下是典型的输出：

#### 输出.27

```
total 72
drwxr-xr-x   2 nobody nogroup 4096 Feb  3 20:24 bin
drwxr-xr-x   2 nobody nogroup 4096 Apr 24  2018 boot
drwxr-xr-x   2 nobody nogroup 4096 Feb  3 20:24 dev
drwxr-xr-x  29 nobody nogroup 4096 Feb  3 20:24 etc
drwxr-xr-x   2 nobody nogroup 4096 Apr 24  2018 home
drwxr-xr-x   8 nobody nogroup 4096 May 23  2017 lib
drwxr-xr-x   2 nobody nogroup 4096 Feb  3 20:23 lib64
drwxr-xr-x   2 nobody nogroup 4096 Feb  3 20:23 media
drwxr-xr-x   2 nobody nogroup 4096 Feb  3 20:23 mnt
drwxr-xr-x   2 nobody nogroup 4096 Feb  3 20:23 opt
dr-xr-xr-x 328 root   root       0 Mar 21 14:10 proc
drwx------   2 nobody nogroup 4096 Feb  3 20:24 root
drwxr-xr-x   4 nobody nogroup 4096 Feb  3 20:23 run
drwxr-xr-x   2 nobody nogroup 4096 Feb  3 20:24 sbin
drwxr-xr-x   2 nobody nogroup 4096 Feb  3 20:23 srv
drwxr-xr-x   2 nobody nogroup 4096 Apr 24  2018 sys
drwxrwxr-x   2 nobody nogroup 4096 Feb  3 20:24 tmp
drwxr-xr-x  10 nobody nogroup 4096 Feb  3 20:23 usr
drwxr-xr-x  11 nobody nogroup 4096 Feb  3 20:24 var
```

比较输出 Output.26 和 Output.27，我们看到根文件系统完全不同。

要将 /tmp 挂载为新命名空间的临时文件系统 tmpfs，请在 TB 中执行以下命令：

# mount -t tmpfs tmpfs /tmp

要在新命名空间的 /tmp 目录下创建文本文件 /tmp/leopard.txt，请在 TB 中执行以下命令：

# echo 'leopard' > /tmp/leopard.txt

要列出新命名空间中文件 /tmp/leopard.txt 的属性，请在 TB 中执行以下命令：

以下是典型的输出：

#### 输出.28

```
-rw-r--r-- 1 root root 7 Mar 14 22:05 /tmp/leopard.txt
```

要列出父命名空间中文件 /tmp/leopard.txt 的属性，请在 TA 中执行以下命令：

以下是典型的输出：

#### 输出.29

```
ls: cannot access '/tmp/leopard.txt': No such file or directory
```

最后，要完全从新命名空间中删除父根文件系统 rootfs/.old_root，请在 TB 中执行以下命令：

# mount --make-rprivate /.old_root

# umount -l /.old_root

要列出新命名空间中的所有挂载点，请在 TB 中执行以下命令：

# cat /proc/mounts | sort

以下是典型的输出：

#### 输出.30

```
/dev/sda1 / ext4 rw,relatime,errors=remount-ro,data=ordered 0 0
proc /proc proc rw,relatime 0 0
tmpfs /tmp tmpfs rw,relatime 0 0
```

要退出新命名空间，请在 TB 中执行以下命令：

成功！我们已经演示了使用 unshare 命令结合 UTS、用户、PID 和挂载命名空间。

* * *
