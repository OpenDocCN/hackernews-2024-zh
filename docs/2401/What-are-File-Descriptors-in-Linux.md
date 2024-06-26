<!--yml

分类：未分类

日期：2024-05-27 14:44:10

-->

# Linux 中的文件描述符是什么

> 来源：[`linuxtldr.com/file-descriptors-linux/`](https://linuxtldr.com/file-descriptors-linux/)

在本文中，你将学习关于文件描述符的一切，比如它们在 Linux 中的用途、文件描述符表是什么、如何在特定进程下查看文件描述符，以及如何在 Linux 中更改文件描述符的限制。

## Linux 中的文件描述符是什么？

文件描述符是一个正整数，作为“文件”和其他 I/O 资源（如管道、套接字、块、设备或终端 I/O）的唯一标识符（或句柄）。

所有文件描述符记录都保存在内核中的文件描述符表中。当一个文件被打开时，在文件描述符表中为该文件分配一个新的文件描述符（或整数值）。

例如，如果你打开一个“`example_file1.txt`”文件（其实就是一个进程），它将被分配一个可用的文件描述符（例如 101），并在文件描述符表中创建一个新条目。

当你打开另一个文件像“`example_file2.txt`”时，它将被分配给另一个可用的文件描述符，如 102，并在文件描述符表中创建另一个条目。

| 文件描述符 | 进程 |
| --- | --- |
| 101 | example_file1.txt |
| 102 | example_file2.txt |

引用文件的文件描述符在你关闭文件后将可供另一个进程使用。

因此，当你在 Linux 系统中打开数百个文件或其他 I/O 资源时，文件描述符表中将有 100 个条目，并且每个条目都将引用一个唯一的文件描述符（或类似 100、102、103... 的整数值）来标识文件。

## Linux 中的文件描述符表是什么？

当一个进程或 I/O 设备发出成功请求时，内核会向该进程返回一个文件描述符，并将当前和所有运行中的进程文件描述符列表保存在文件描述符表中，该表位于内核的某个地方。

现在，你的进程可能依赖于其他系统资源，如输入和输出；由于这个事件也是一个进程，它也有一个文件描述符，该文件描述符将附加到文件描述符表中的你的进程上。

文件描述符表中的每个文件描述符都指向内核全局文件表中的一个条目。文件表条目维护文件（或其他 I/O 资源）模式的记录，如 (`r`)ead、(`w`)rite 和 (`e`)xecute。

此外，文件表条目指向一个称为 inode 表的第三个表，该表指向实际文件信息，如大小、修改日期、指针等。

## 预定义文件描述符

默认情况下，文件描述符表中存在三种类型的标准 POSIX 文件描述符，你可能已经熟悉它们作为 [Linux 中的数据流](https://linuxtldr.com/understanding-streams-in-linux/)：

| 文件描述符 | 名称 | 缩写 |
| --- | --- | --- |
| `0` | 标准输入 | stdin |
| `1` | 标准输出 | stdout |
| `2` | 标准错误 | stderr |

除了它们之外，每个其他进程都有自己的一组文件描述符，但其中少数（除了一些守护进程）也利用了上述提到的文件描述符来处理进程的输入、输出和错误。

要确保进程正在使用上述文件描述符，只需在“`/proc/PID/fd/`”下查找上述文件描述符（以整数格式表示），其中 PID 代表“进程标识符”。

例如，我在我的系统上启动了 GEDIT 编辑器，它使用了上述所有文件描述符，如图所示。

## 列出运行进程的所有文件描述符

正如您刚刚学到的，Linux 中的每个运行进程都有自己的一组文件描述符，但也使用其他文件描述符来标识与内核空间通过系统调用或库调用进行通信的特定文件。

### 查找进程 ID（或 PID）

首先，使用 [ps 命令](https://linuxtldr.com/ps-command/)找出您的进程标识符（或 PID），然后查看其下的文件描述符。

```
$ ps aux | grep gedit
```

用您正在运行的进程名称替换“`gedit`”，或者您可以放置“`$$`”以传递当前 bash 会话。

现在，您有两种方法来列出特定进程下的文件描述符，接着是：

### 使用 ls 命令

使用 [ls 命令](https://linuxtldr.com/ls-command/)列出“`/proc/PID/fd/`”路径下某个 PID 下的所有文件描述符及其所指向的文件。

```
$ ls -la /proc/11472/fd/
```

输出：

### 使用 lsof 命令

lsof 命令用于列出系统中运行进程的信息，并且也可用于列出特定 PID 下的文件描述符。

为此，请使用“`-d`”标志指定文件描述符的范围，“`-p`”选项指定 PID。要组合此选择，请使用“`-a`”标志。

```
$ lsof -a -d 0-2147483647 -p 11472
```

输出：

## 首先，文件描述符的目的是什么？

文件描述符与文件表一起，跟踪系统中每个运行进程的权限，并维护数据完整性。

一个运行中的进程可以通过继承其文件描述符来继承另一个进程的功能，正如您在本文中所学到的。

## **如果文件描述符用尽会发生什么？**

这一点至关重要，因为文件描述符是内核在成功打开文件后返回给进程（或其他 I/O 资源）的整数值。

对于一个进程，可以给它分配的文件描述符（或整数值）数量是有限制的。当达到该限制时，数据可能会丢失。

通常情况下，在 Linux 中有两种类型的文件描述符：进程级文件描述符和系统级文件描述符。

### 进程级文件描述符限制

使用 ulimit 命令检查当前进程级文件描述符限制。

```
$ ulimit -n
```

输出：

通过在命令之后添加自定义正数来重置限制。

```
$ ulimit -n 3276800
```

注意，非 root 用户也可以使用上述命令来更改进程级限制（<Kernel 2.4.x），但您需要在“`/etc/security/limits.conf`”中添加以下行以分配用户修改权限：

```
soft nofile 2048
hard nofile 8192
```

### 系统级文件描述符限制

使用[cat 命令](https://linuxtldr.com/cat-command/)检查系统级描述符的限制。

```
$ cat /proc/sys/fs/file-max
```

输出：

使用“`>`” [重定向符号](https://linuxtldr.com/understanding-streams-in-linux/)修改文件中的新值。

```
$ echo 90000 > /proc/sys/fs/file-max
```

修改上述文件后，在“`nr_open`”文件中修改该值。

```
$ echo "50000" > /proc/sys/fs/nr_open
```

## 最终提示！

希望本文能够让您清楚地了解 Linux 计算中文件描述符的工作原理。

如果您对此主题有任何疑问或查询，请在评论部分随时提出。
