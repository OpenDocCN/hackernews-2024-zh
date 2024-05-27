<!--yml
category: 未分类
date: 2024-05-27 13:09:22
-->

# Building a Linux Container using Namespaces :: Part - 1

> 来源：[https://www.polarsparc.com/xhtml/Containers-1.html](https://www.polarsparc.com/xhtml/Containers-1.html)

Building a Linux Container using Namespaces :: Part - 1

* * *

Ever wondered how Linux Containers worked ???

Currently, [Docker](https://www.polarsparc.com/xhtml/Docker.html) is one of the most popular and prevalent container implementations.

Containers run on top of the same Operating System kernel, but isolate the application processes running inside them from one another. One of the secret sauces behind containers is [Namespaces](http://man7.org/linux/man-pages/man7/namespaces.7.html).

A Namespace abstracts global system resources, such as, host names, user IDs, group IDs, process IDs, network ports, etc., in a way that it appears to the processes (within the namespace) as though they have their own isolated instance of the global system resources. One of the primary goals of namespaces is to support the implementation of containers (lightweight virtualization).

Currently, in Linux there are 6 types of namespaces - IPC, Network, Mount, PID, User, and UTS.

The following are brief descriptions for each of the namespaces:

*   IPC :: This namespace isolates certain interprocess communication (IPC) resources, namely, Message Queues, Semaphores, and Shared Memory

*   Network :: This namespace provides isolation of the system resources associated with networking, such as, Network devices, IP addresses, IP routing tables, /proc/net directory, port numbers, and so on

*   Mount :: This namespace isolates the set of filesystem mount points seen by a group of processes. Processes in different mount namespaces can have different views of the filesystem hierarchy

*   PID :: This namespace isolates the process ID number space. This allows processes in different PID namespaces to have the same PID

*   User :: This namespace isolates the user and group ID number spaces, such that, a process's user and group IDs can be different inside and outside the user namespace

*   UTS :: This namespace isolates two system identifiers — the hostname and the domainname. For containers, the UTS namespaces allows each container to have its own hostname and NIS domain name

For the demonstration in this article, we will be using the unshare Linux command as well as implement, build, and execute a simple container using golang.

The installation is on a Ubuntu 18.04 LTS based Linux desktop.

We will need two commands newuidmap and newgidmap to demonstrate User namespace. For this, we need to install the package uidmap.

To install the package uidmap, execute the following command:

$ sudo apt install -y uidmap

Next, we will need the brctl command to create a bridge network interface. For this, we need to install the package bridge-utils.

To install the package bridge-utils, execute the following command:

$ sudo apt install -y bridge-utils

To develop, build, and execute the simple container in go programming language, we need to install the golang package.

To check the version of golang available to install, execute the following command:

$ sudo apt-cache policy golang

The following would be a typical output:

#### Output.1

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

To install golang, execute the following command:

$ sudo apt install -y golang

The above installation procedure installs golang from the official Ubuntu repository.

Create a directory for developing, building, and running go programs by executing the following commands:

$ mkdir $HOME/projects/go

$ export GOPATH=$HOME/projects/go

We will need one of the popular go packages on netlink for networking.

To download the go package, execute the following command:

$ go get github.com/vishvananda/netlink

Open two Terminal windows - we will refer to them as TA and TB respectively. TB is where we will demonstrate the simple container.

We need to download a minimal root filesystem (rootfs) that will be used as the base image for the simple container. For our demonstration, we will choose the latest [Ubuntu Base 18.04.4 LTS](http://cdimage.ubuntu.com/ubuntu-base/releases/18.04.4/release/ubuntu-base-18.04.4-base-amd64.tar.gz) at the time of this article.

We will assume the latest Ubuntu Base is downloaded to the directory $HOME/Downloads.

The unshare command executes the specified program with the indicated namespace(s) isolated from the parent process.

To display the options for the unshare command, execute the following command in TA:

The following would be a typical output:

#### Output.2

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

Each process (with [PID]) has associated with it a sub-directory /proc/[PID]/ns that contains one entry for each of the namespaces.

To list all the namespaces associated with a process, execute the following command in TA :

The following would be a typical output:

#### Output.3

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

To launch a simple container whose host name is isolated from the parent host name, execute the following command in TB:

$ sudo unshare -u /bin/sh

The -u option enables the UTS namespace.

The command prompt will change to a #.

To check the PID of the simple container, execute the following command in TB:

The following would be a typical output:

To list all the namespaces associated with the simple container, execute the following command in TB:

The following would be a typical output:

#### Output.5

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

Comparing Output.5 to Output.3, we see a change in the uts namespace, which is expected and correct.

To change the host name of the simple container, execute the following command in TB:

To display the host name of the parent host, execute the following command in TA:

The following would be a typical output:

To display the host name of the simple container, execute the following command in TB:

The following would be a typical output:

This demonstrates to us that we have isolated the host name of the simple container from the parent host name.

To exit the simple container, execute the following command in TB:

Next, we will mimic the above UTS namespace isolation using the following go program:

<fieldset id="sc-fieldset"><legend>Listing.1</legend>

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

The Command function from the exec package allows one to run the specified command (1st parameter) with the supplied arguments (2nd parameter). It returns an instance of the Cmd struct.

One can set the standard input (os.Stdin), the standard output os.Stdout, the standard error os.Stderr, and some operating system specific attributes on the returned Cmd instance. In this case, we specify the syscall.CLONE_NEWUTS OS attribute to indicate the command be run in a new UTS namespace.

IMPORTANT : When the main process starts, it internally spawns another main process (with the CLONE argument) in a new namespace. It is this spawned main process (running in the new namespace) that is overlayed (syscall.Exec) with the shell command by invoking the function execContainerShell.

Create and change to the directory $GOPATH/uts by executing the following commands in TB:

$ mkdir -p $GOPATH/uts

$ cd $GOPATH/uts

Copy the above code into the program file main.go in the current directory.

To compile the program file main.go, execute the following command in TB:

To run program main, execute the following command in TB:

The following would be a typical output:

#### Output.8

```
2020/03/07 12:49:11 Starting process ./main with args: [./main]
2020/03/07 12:49:11 Ready to run command ...
2020/03/07 12:49:11 Starting process ./main with args: [./main CLONE]
2020/03/07 12:49:11 Ready to exec container shell ...
->
```

The command prompt will change to a ->.

To display the host name of the simple container, execute the following command in TB:

The following would be a typical output:

To exit the simple container, execute the following command in TB:

SUCCESS !!! We have demonstrated the UTS namespace using both the unshare command and a simple go program.

Let us layer the User namespace on top of the UTS namespace.

To launch a simple container whose user/group IDs as well as the host name are isolated from the parent namespace, execute the following command in TB:

$ sudo unshare -uU /bin/sh

The -U option enables the User namespace.

To display the user ID and group ID in the new namespace, execute the following command in TB:

The following would be a typical output:

#### Output.10

```
uid=65534(nobody) gid=65534(nogroup) groups=65534(nogroup)
```

When a User namespace is created, it starts without a mapping for the user/group IDs in the new namespace to the parent user/group IDs. The unmapped user/group ID is assigned the default value of the overflow user/group ID. The default value for the overflow user ID is read from /proc/sys/kernel/overflowuid (which is 65534). Similarly, the default value for the overflow group ID is read from /proc/sys/kernel/overflowgid (which is 65534).

To fix the mapping for the user/group ID to the parent user/group ID, exit the simple container by executing the following command in TB:

To re-launch the simple container with the current effective user/group ID mapped to the superuser user/group ID in the new namespace, execute the following command in TB:

$ sudo unshare -uUr /bin/sh

The -r option enables the mapping of the user/group IDs in the new namespace to the parent namespace user/group IDs.

The command prompt will change to a #.

To display the user ID and group ID in the new namespace, execute the following command in TB:

The following would be a typical output:

#### Output.11

```
uid=0(root) gid=0(root) groups=0(root)
```

To list all the namespaces associated with the simple container, execute the following command in TB:

The following would be a typical output:

#### Output.12

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

Comparing Output.12 to Output.3, we see a change in both the uts namespace as well as the user namespace, which is what is expected and correct.

To exit the simple container, execute the following command in TB:

Next, we will mimic the above UTS and User namespace isolation using the following go program:

<fieldset id="sc-fieldset"><legend>Listing.2</legend>

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

As indicated previously, the Command function returns an instance of the Cmd struct.

In this example, we specify the additional syscall.CLONE_NEWUSER OS attribute to indicate the command be run in a new User namespace.

In addition, we set the user ID map UidMappings as an array of syscall.SysProcIDMap struct entries, each consisting of the user ID mapping in the container (ContainerID) to the user ID in the host namespace (HostID). In this case, we map the root user ID 0 in the container to the root user ID 0 of the host namespace. Similarly, we set the group ID map GidMappings

Create and change to the directory $GOPATH/user by executing the following commands in TB:

$ mkdir -p $GOPATH/user

$ cd $GOPATH/user

Copy the above code into the program file main.go in the current directory.

To compile the program file main.go, execute the following command in TB:

To run program main, execute the following command in TB:

The following would be a typical output:

#### Output.13

```
2020/03/07 13:17:02 Starting process ./main with args: [./main]
2020/03/07 13:17:02 Ready to run command ...
2020/03/07 13:17:02 Starting process ./main with args: [./main CLONE]
2020/03/07 13:17:02 Ready to exec container shell ...
->
```

The command prompt will change to a ->.

To display the user ID and group ID in the new namespace, execute the following command in TB:

The following would be a typical output:

#### Output.14

```
uid=0(root) gid=0(root) groups=0(root)
```

To list all the namespaces associated with the simple container, execute the following command in TB:

The following would be a typical output:

#### Output.15

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

To display the host name of the simple container, execute the following command in TB:

The following would be a typical output:

To exit the simple container, execute the following command in TB:

SUCCESS !!! We have demonstrated the combined UTS and User namespaces using both the unshare command and a simple go program.

Let us now layer the PID namespace on top of the User namespace and the UTS namespace.

To launch a simple container whose process IDs as well as the user/group IDs and the host name are isolated from the parent namespace, execute the following command in TB:

$ sudo unshare -uUrpf --mount-proc /bin/sh

The -p option enables the PID namespace.

The -f option enables spawning (or forking) of new processes in the new namespace.

The --mount-proc option mounts the proc filesystem as a private mount at /proc in the new namespace. This means the /proc pseudo directory only shows information only about processes within that PID namespace.

#### ATTENTION

```
Ensure the option -f is *SPECIFIED*. Else will encounter the following error:

/bin/sh: 4: Cannot fork
```

The command prompt will change to a #.

To display all the processes in the new namespace, execute the following command in TB:

The following would be a typical output:

#### Output.17

```
USER       PID %CPU %MEM    VSZ   RSS TTY      STAT START   TIME COMMAND
root         1  0.0  0.0   4628   880 pts/1    S    09:08   0:00 /bin/sh
root         6  0.0  0.0  37368  3340 pts/1    R+   09:12   0:00 ps -fu
```

To display all the processes in the parent namespace, execute the following command in TA:

The following would be a typical output:

#### Output.18

```
USER         PID %CPU %MEM    VSZ   RSS TTY      STAT START   TIME COMMAND
polarsparc  8695  0.0  0.0  22840  5424 pts/1    Ss   08:43   0:00 bash
polarsparc  8681  0.0  0.0  22708  5096 pts/0    Ss   08:43   0:00 bash
polarsparc  9635  0.0  0.0  37368  3364 pts/0    R+   09:12   0:00  \_ ps -fu
```

Comparing Output.17 to Output.18, we see the isolation between the new namespace and the parent namespace, which is what is expected and correct.

To exit the simple container, execute the following command in TB:

Next, we will mimic the above UTS, User, and PID namespace isolation using the following go program:

<fieldset id="sc-fieldset"><legend>Listing.3</legend>

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

</fieldset>

As indicated previously, the Command function returns an instance of the Cmd struct.

In this example, we specify the additional syscall.CLONE_NEWNS and syscall.CLONE_NEWPID OS attributes to indicate the command be run in a new PID namespace.

Create and change to the directory $GOPATH/pid by executing the following commands in TB:

$ mkdir -p $GOPATH/pid

$ cd $GOPATH/pid

Copy the above code into the program file main.go in the current directory.

To compile the program file main.go, execute the following command in TB:

To run program main, execute the following command in TB:

The following would be a typical output:

#### Output.19

```
2020/03/07 13:38:02 Starting process ./main with args: [./main]
2020/03/07 13:38:02 Ready to run command ...
2020/03/07 13:38:02 Starting process ./main with args: [./main CLONE]
2020/03/07 13:38:02 Ready to exec container shell ...
->
```

The command prompt will change to a ->.

To display the host name of the simple container, execute the following command in TB:

The following would be a typical output:

To display the user ID and group ID in the new namespace, execute the following command in TB:

The following would be a typical output:

#### Output.21

```
uid=0(root) gid=0(root) groups=0(root)
```

To display all the processes in the simple container, execute the following command in TB:

The following would be a typical output:

#### Output.22

```
USER       PID %CPU %MEM    VSZ   RSS TTY      STAT START   TIME COMMAND
root         1  0.0  0.0   4628   776 pts/1    S    09:41   0:00 
root         6  0.0  0.0  37368  3400 pts/1    R+   09:41   0:00 ps -fu
```

To list all the namespaces associated with the simple container, execute the following command in TB:

The following would be a typical output:

#### Output.23

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

To exit the simple container, execute the following command in TB:

SUCCESS !!! We have demonstrated the combined UTS, User, and PID namespaces using both the unshare command and a simple go program.

We will now setup the minimal Ubuntu Base image for use in the new namespace in the /tmp directory.

To create and copy the base image to a directory in /tmp, execute the following commands in TA:

$ mkdir -p /tmp/rootfs/.old_root

$ tar -xvf $HOME/Downloads/ubuntu-base-18.04.4-base-amd64.tar.gz --directory /tmp/rootfs

cd /tmp

Now let us now layer the Mount namespace on top of the User, the UTS, and the PID namespaces.

To launch a simple container whose mount points as well as the process IDs, the user/group IDs, and the host name are isolated from the parent namespace, execute the following command in TB:

$ sudo unshare -uUrpfm --mount-proc /bin/sh

The -m option enables the Mount namespace.

The command prompt will change to a #.

To list all the mount points in the parent namespace, execute the following command in TA:

$ cat /proc/mounts | sort

The following would be a typical output:

#### Output.24

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

Now, let us list all the mount points in the new namespace by executing the following command in TB:

# cat /proc/mounts | sort

The following would be a typical output:

#### Output.25

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

Comparing Output.25 and Output.24, we see the one difference for proc. When a new Mount namespace is created, the mount points of the new namespace is a copy of the mount points in the parent's namespace.

We will now demonstrate any changes to the new namespace will not affect the parent namespace.

To make the mount point / (and its children recursively) to be private to the new namespace, execute the following command in TB:

# mount --make-rprivate /

To recursive bind the mount point rootfs/ to rootfs/ in the new namespace, execute the following command in TB:

# mount --rbind rootfs/ rootfs/

We need the proc filesystem in the new namespace for making changes to mounts. To mount /proc as the proc filesystem proc in the new namespace, execute the following command in TB:

# mount -t proc proc rootfs/proc

Next, we need to make rootfs/ the root filesystem in the new namespace and move the parent root filesystem to rootfs/.old_root using the pivot_root command. To do that, execute the following commands in TB:

# pivot_root rootfs/ rootfs/.old_root

# cd /

To list all the file(s) under / in the parent namespace, execute the following command in TA:

The following would be a typical output:

#### Output.26

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

To list all the file(s) under / in the new namespace, execute the following command in TB:

The following would be a typical output:

#### Output.27

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

Comparing Output.26 and Output.27, we see the root filesystems are totally different.

To mount /tmp as the temporary filesystem tmpfs in the new namespace, execute the following command in TB:

# mount -t tmpfs tmpfs /tmp

To create a text file /tmp/leopard.txt in the directory /tmp of the new namespace, execute the following command in TB:

# echo 'leopard' > /tmp/leopard.txt

To list the properties of the file /tmp/leopard.txt in the new namespace, execute the following command in TB:

The following would be a typical output:

#### Output.28

```
-rw-r--r-- 1 root root 7 Mar 14 22:05 /tmp/leopard.txt
```

To list the properties of the file /tmp/leopard.txt in the parent namespace, execute the following command in TA:

The following would be a typical output:

#### Output.29

```
ls: cannot access '/tmp/leopard.txt': No such file or directory
```

Finally, to completely remove the parent root filesystem rootfs/.old_root from the new namespace, execute the following commands in TB:

# mount --make-rprivate /.old_root

# umount -l /.old_root

To list all the mount points in the new namespace by executing the following command in TB :

# cat /proc/mounts | sort

The following would be a typical output:

#### Output.30

```
/dev/sda1 / ext4 rw,relatime,errors=remount-ro,data=ordered 0 0
proc /proc proc rw,relatime 0 0
tmpfs /tmp tmpfs rw,relatime 0 0
```

To exit the new namespace, execute the following command in TB:

SUCCESS !!! We have demonstrated the combined UTS, User, PID, and Mount namespaces using the unshare command.

* * *