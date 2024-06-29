<!--yml

category: 未分类

date: 2024-05-27 14:31:08

-->

# IPC - Shared Memory | Goodness's blog

> 来源：[https://goodyduru.github.io/os/2024/01/31/ipc-shared-memory.html](https://goodyduru.github.io/os/2024/01/31/ipc-shared-memory.html)

在之前的文章中，我们介绍了[消息队列](https://goodyduru.github.io/os/2023/11/13/ipc-message-queues.html)的IPC机制。本文将介绍另一种称为共享内存的机制。

在讨论共享内存及其在IPC中的用途之前，我们必须了解为什么会存在“共享内存”这种东西，因为我们知道所有进程共享它们应该共享的RAM。问题在于，它们既共享又不共享。这种明显矛盾的现象归因于内存寻址的魔力。让我们稍微谈谈这个问题。

### 内存寻址

访问内存有两种模式：**物理**和**虚拟**^([[1]](#footer-note-1))。在物理寻址模式下，当一个进程从内存地址读取数据时，CPU从RAM中的相同地址获取数据并交给进程。写入也是如此。早期微处理器只能通过物理寻址方式访问RAM，在许多嵌入式计算机中仍在使用。

当只有一个进程预期在计算机上运行时，物理寻址是可以接受的。但对于多进程系统来说，它是灾难性的。例如，如果同时运行两个不同的进程A和B，A可能会写入B也读取的内存地址。更糟糕的是，即使B是内核进程，A也可能会写入包含B可执行代码一部分的内存地址。

上述问题是为什么虚拟寻址是当今许多微处理器架构中使用的内存地址模式之一的原因之一^([[2]](#footer-note-2))。在虚拟寻址模式下，当一个进程从内存地址读取数据时，这个内存地址会被转换为物理地址，由[内存管理单元（MMU）](https://zh.wikipedia.org/wiki/%E5%86%85%E5%AD%98%E7%AE%A1%E7%90%86%E5%8D%95%E5%85%83)来完成，然后从该物理地址读取数据并交给进程。这一切都在进程的不知情下完成。

这种地址转换是通过[**页表**](https://zh.wikipedia.org/wiki/%E9%A1%B5%E8%A1%A8)来完成的。页表以[页](https://zh.wikipedia.org/wiki/%E9%A1%B5_(%E8%AE%A1%E7%AE%97%E6%9C%BA%E5%86%85%E5%AD%98))单元存储虚拟地址和物理地址之间的映射^([[3]](#footer-note-3))。每个进程都有自己的页表，并允许进程引用接近**几乎^([[4]](#footer-note-4))**所有的地址直到表的最大地址^([[5]](#footer-note-5))。

虚拟寻址允许多个进程共存而不会互相干扰。如果我们的两个进程尝试访问同一内存地址，那么该地址将在RAM中指向两个不同的地址。因此，内存地址4523可能对应于A的9619和B的86443。这意味着它们不共享虚拟内存但共享物理内存。

如果进程只能使用无法共享的虚拟内存，它们如何使用共享内存进行通信呢？让我们来看看这个问题。

### 共享内存

共享内存是在进程之间共享的一段内存。物理内存段通过它们各自的页表映射到每个进程的虚拟内存中。一旦完成映射，进程就可以像处理其他段一样读写这一段。写入此段的任何数据对其他参与进程可见。虚拟地址可能因参与进程的不同而不同，尽管只有一个物理地址。

与[消息队列](https://goodyduru.github.io/os/2023/11/13/ipc-message-queues.html#message-queues)一样，共享内存分为两种 - System V 和 POSIX。本文将重点介绍 System V。

创建或访问共享内存段需要一个唯一的整数键。如果没有使用唯一键，外部进程可能与您的进程进行通信，这可能很烦人，甚至更糟糕。此键可以在每个参与进程中硬编码，或者可以生成。硬编码键会降低唯一性的机会，因为外部进程可能在其中也硬编码相同的键。键的生成应该确保所有参与进程都能生成相同的键。幸运的是，有一个（几乎）保证唯一性且是确定性的函数存在。此函数称为[`ftok`](https://man7.org/linux/man-pages/man3/ftok.3.html)。它接受一个现有文件路径和一个整数作为参数。只要路径指向的文件未被重新创建，`ftok` 就会返回相同的结果。所有参与进程只需要使用相同的文件路径和整数调用 `ftok` 函数，就会生成相同的键。额外的安全步骤可以包括确保文件仅对参与进程可访问。

一旦生成了一个键，进程可以使用它来通过调用[`shmget`](https://man7.org/linux/man-pages/man3/shmget.3p.html)函数获取或创建一个共享内存段。此函数接受三个参数：键、共享内存的大小和一个标志。如果键未与共享内存段关联，并且标志中设置了**IPC_CREAT**位，将创建一个大小为 size 参数的共享内存段。在创建段时，函数返回一个 id。如果键具有现有的共享内存段，则返回该现有段的 id，即使标志设置了**IPC_CREAT**位也是如此。有多种原因可能导致错误，这些在 man 手册中有所涵盖。权限可以通过与[file permissions](https://www.multacom.com/faq/password_protection/file_permissions.htm)相同的格式在标志中定义。

成功获取共享内存 id 后，共享内存段的虚拟内存地址将由 [`shmat`](https://man7.org/linux/man-pages/man3/shmat.3p.html) 函数返回。此函数接受 3 个参数：从 `shmget` 获取的共享内存 id，进程希望将段映射到的虚拟内存地址，以及一个标志。如果地址参数设置为零，操作系统将选择一个地址。函数执行成功后返回一个内存地址。

返回内存地址后，进程可以通过仅读取和写入地址与其他进程通信。读取和写入地址与读取和写入其他地址使用相同的方法进行，没有特殊的函数。

当进程完成通信时，可以通过调用 [`shmdt`](https://man7.org/linux/man-pages/man3/shmdt.3p.html) 函数将段从其地址空间分离。该函数仅接受一个参数，即从 `shmat` 获取的虚拟内存地址。一旦调用此函数，地址将变为无效，与之交互可能导致段错误。

分离段不会销毁它，即使所有参与的进程都从中分离。通过调用销毁 `shmctl(id, IPC_RMID, NULL);`

> `shmctl(id, IPC_RMID, NULL);`

`id` 参数是从 `shmget` 获取的共享内存段 id。`IPC_RMID` 是一个常量，表示删除段。除了销毁段外，[`shmctl`](https://man7.org/linux/man-pages/man3/shmctl.3p.html) 还可用于配置段。

通过这个命令 `ipcs -m` 可以得到系统上所有的共享内存段的列表。

足够的讨论。让我们看看 Python 中的一个示例。

### 给我看看代码

此示例将演示两个进程在 Python 中使用共享内存进行通信。Python 不提供开箱即用的共享内存支持。相反，我使用了优秀的 [sysv-ipc](https://semanchuk.com/philip/sysv_ipc/#shared_memory) 库。你可以在 [这里](https://pypi.org/project/sysv-ipc/) 找到 pip 包。

这里是客户端代码：

```
import os
import time

import sysv_ipc

ROUNDS = 100

def run():
    path = '/tmp/example'
    fd = os.open(path, flags=os.O_CREAT)
    os.close(fd)
    key = sysv_ipc.ftok(path, 42)
    mem = sysv_ipc.SharedMemory(key, size=20, flags=sysv_ipc.IPC_CREAT, mode=0o644)
    i = 0
    message = b"ping"
    while i != ROUNDS:
        mem.write(message)
        print("Client: Sent ping")
        data = mem.read(4)
        while data == message:
            time.sleep(1e-6)
            data = mem.read(4)
        data = data.decode()
        print(f"Client: Received {data}")
        i += 1
    mem.write(b"end")
    mem.detach()

run() 
```

为了确保临时文件存在（可选），调用 ftok 函数时使用文件路径和一个整数。共享内存段被（可选）创建并访问。这将返回一个共享内存对象。执行一个循环，其中发送包含字节字符串的消息。为防止进程处理它发送的消息，运行一个 while 循环，使进程进入睡眠并接收消息。如果消息与发送的消息不同，则此 while 循环停止，表示该消息来自另一个进程。此 while 循环是非常原始的通信同步形式。接收到的消息被解码并打印，然后循环继续。当发送 *ROUNDS* 消息后，循环结束。然后发送一个 *end* 消息，表示客户端已完成。

发送和接收的消息是字节字符串，而不是常规字符串，因此必须进行相应的编码和解码。

下面是服务器端的代码

```
import os
import time

import sysv_ipc

def run():
    path = '/tmp/example'
    fd = os.open(path, flags=os.O_CREAT) # create file
    os.close(fd)
    key = sysv_ipc.ftok(path, 42)
    mem = sysv_ipc.SharedMemory(key, size=20, flags=sysv_ipc.IPC_CREAT, mode=0o644)
    data = mem.read(4)
    message = b"pong"
    while data != b"ping":
        time.sleep(5e-6)
        data = mem.read(4)
    data = data.decode()
    while data[0] != 'e':
        print(f"Server: Received {data}")
        mem.write(message)
        print(f"Server: Sent pong")
        data = mem.read(4)
        while data == message:
            time.sleep(1e-6)
            data = mem.read(4)
        data = data.decode()
    mem.remove()
    mem.detach()
    os.unlink(path)

run() 
```

服务器端的代码与客户端代码类似地设置了共享内存段。因为 `ftok` 使用的参数与客户端代码中的完全相同，生成的密钥值也将相同。发送一条消息，运行一个 while 循环，确保接收到的消息不和发送的消息相同。运行一个循环，检查接收到的消息的第一个字符是否不等于 *e*，这意味着没有接收到 *end*。在这个循环中，打印接收到的消息，并发送另一条消息。发送后，再次运行消息检查的循环。然后接收并解码消息。

一旦接收到 *end* 消息，循环就会停止。用于生成密钥和共享内存段的文件将被删除。然后程序退出。

### 性能

共享消息速度超快。与最快的 IPC 机制（mmap）之间的差异不大。[IPC-Bench](https://github.com/goldsborough/ipc-bench#benchmarked-on-intelr-coretm-i5-4590s-cpu--300ghz-running-ubuntu-20041-lts) 在运行 Ubuntu 20.04.1 LTS 的 Intel(R) Core(TM) i5-4590S CPU 上对每秒传送 1,659,291 个大小为 1KB 的消息进行了基准测试。速度超快。

由于其极快的速度和其类似于标准的内存地址读写的特性，使用共享内存时必须像上面的代码片段一样包括某种形式的同步机制。

### 演示代码

您可以在[GitHub](https://github.com/goodyduru/ipc-demos)找到我展示共享内存的代码。

### 结论

共享内存是一种简单且快速的 IPC 机制。它有一个简单直接的模型来发送和接收消息。即使如此，使用它需要包括同步机制的复杂性。但一旦掌握了这一点，它的速度就物有所值。

下一篇文章将介绍另一个极快且常见的 IPC 机制，叫做[内存映射文件](https://goodyduru.github.io/os/2024/03/18/ipc-mmap.html)。在那之前，好好照顾自己，保持水分！✌🏾

* * *
