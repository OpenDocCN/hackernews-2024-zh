<!--yml

category: 未分类

date: 2024-05-27 12:54:06

-->

# 游戏程序员的XDP

> 来源：[https://mas-bandwidth.com/xdp-for-game-programmers/](https://mas-bandwidth.com/xdp-for-game-programmers/)

我是[Glenn Fiedler](https://www.linkedin.com/in/glenn-fiedler-11b735302/?ref=mas-bandwidth.com)，欢迎来到**Más Bandwidth**，我在游戏网络编程和可伸缩后端工程交汇处的新博客。

这个新博客是关于什么？拥有数千玩家的游戏。虚拟世界。以百万为单位的虚拟空间性能。**元宇宙**（不，不是那个元宇宙，是***真正的元宇宙，没有区块链***）。增强现实中的叠加世界。遥感和虚拟现实中的远程工作。游戏流媒体（*呸，令人作呕！*）。好吧，除了最后一个，我将在稍后的文章中解释为什么。

但说真的，在10年内，每个人都将拥有10gbps的互联网。这会如何改变游戏制作？我们将如何利用所有这些带宽？5v5射击游戏将不再适用。**接下来是什么？**

作为这个博客的首篇文章，如果你发现自己像我一样，需要应用程序的***绝对最大带宽***，那么你将需要使用内核旁路技术。为什么？因为否则，每个数据包在内核中处理并将其传递到用户空间，再返回到内核并传递到网卡的开销会限制您能达到的吞吐量。我们在这里谈论的是10gbps及以上的速度。

好消息是，在过去的5年里，作为Linux内核旁路技术的XDP/eBPF已经足够成熟，使得它已经从内核黑客的领域转移到现在2024年初，可以被像你我这样的普通人普遍使用。

因此，在本文中，我将为您快速概述XDP/eBPF的工作原理，展示您可以用XDP/eBPF做什么，*并*为一些简单的XDP程序提供一些工作示例代码（[https://github.com/mas-bandwidth/xdp](https://github.com/mas-bandwidth/xdp?ref=mas-bandwidth.com)），这样您就可以开始在应用程序中使用这项技术。

在XDP中做一些真正了不起的事情，所以请继续阅读！

* * *

### 什么是XDP，它是如何工作的？

XDP代表"快速数据路径"，基本上是一种你可以编写函数的方式，在数据包从网卡出来时最早的时刻调用该函数，就在Linux内核为数据包进行任何分配或处理之前。

这是不可思议的。这是强大的。这是程序员的毒品。你可以编写一个在Linux内核中运行的函数，你几乎可以做*任何你想做的事情*。你可以：

+   丢弃数据包

+   修改数据包内容，或完全替换数据包

+   在包头或包尾增加或减少数据包大小

+   发送响应数据包，或将数据包转发到另一个地址

*或*

+   *将数据包传递到内核进行常规处理*

最后一点至关重要。与 DPDK 等其他内核旁路技术不同，您需要安装第二个 NIC 来运行您的程序，或者基本上实现（或许可）一个完整的 TCP/IP 网络堆栈，以确保一切在幕后正常工作（NIC 不仅仅处理 UDP 数据包，例如您的游戏...）。

现在，您可以专注于调整您的 XDP 程序，例如，仅适用于发送到端口 40000 的 IPv4 UDP 数据包，并将其他所有数据包传递给 Linux 内核进行常规处理。简单吧。

***更正：** 显然，现在您可以使用 "双分支驱动程序" 与 DPDK 一起将某些数据包传递回操作系统。这在我上次使用 DPDK 时还不可用，那是相当长时间之前的事情了。不过我仍然更喜欢 XDP 而不是 DPDK。*

### 什么是 eBPF**?**

eBPF 代表"扩展伯克利数据包过滤器"，它是一种技术，可以让您在 Linux 内核中编译、链接和运行 XDP 程序。

简而言之，eBPF 是一个在 Linux 内核中运行函数的字节码和轻量级虚拟机。eBPF 函数可以插入许多不同的位置，而 XDP 只是其中之一。

因为 eBPF 函数在 Linux 内核中运行，它们不能崩溃，绝对不能停止。为了确保这一点成立，BPF 函数在加载到内核之前必须通过验证器。

在实践中，这意味着 XDP 函数在功能上有些受限。它们不是图灵完备的（停机问题），并且你必须做很多努力来向验证器证明你不会写出界。但实际上，只要保持简单，并且愿意与验证器创造性地对抗，通常可以说服它你的程序是安全的。

### 在 Ubuntu 22.04 LTS 上设置 eBPF/XDP

在您开始编写 XDP 程序之前，您需要设置您的机器，以便能够编译、链接和运行 eBPF 程序，并将它们加载到内核中。

从 Ubuntu 22.04 LTS 发行版开始。

首先，您需要确保您有 6.5 版本的 Linux 内核：

```
uname -r 
```

如果输出不是版本 6.5，请使用以下命令更新您的内核：

```
sudo apt install linux-generic-hwe-22.04 -y 
```

在命令行中运行以下命令：

```
# install necessary packages

sudo NEEDRESTART_SUSPEND=1 apt autoremove -y
sudo NEEDRESTART_SUSPEND=1 apt update -y
sudo NEEDRESTART_SUSPEND=1 apt upgrade -y
sudo NEEDRESTART_SUSPEND=1 apt dist-upgrade -y
sudo NEEDRESTART_SUSPEND=1 apt full-upgrade -y
sudo NEEDRESTART_SUSPEND=1 apt install libcurl3-gnutls-dev build-essential vim wget libsodium-dev flex bison clang unzip libc6-dev-i386 gcc-12 dwarves libelf-dev pkg-config m4 libpcap-dev net-tools -y
sudo NEEDRESTART_SUSPEND=1 apt install linux-headers-`uname -r` linux-tools-`uname -r` -y
sudo NEEDRESTART_SUSPEND=1 apt autoremove -y

# install libxdp and libbpf from source

cd ~
wget https://github.com/xdp-project/xdp-tools/releases/download/v1.4.2/xdp-tools-1.4.2.tar.gz
tar -zxf xdp-tools-1.4.2.tar.gz
cd xdp-tools-1.4.2
./configure
make -j && sudo make install

cd lib/libbpf/src
make -j && sudo make install
sudo ldconfig

# setup vmlinux btf

sudo cp /sys/kernel/btf/vmlinux /usr/lib/modules/`uname -r`/build/ 
```

总结一下，关键步骤是从源代码构建 **libxdp**，然后构建并安装包含在 **libxdp** 中的确切版本的 **libbpf**。

我只能猜测为什么这是必要的，但如果没有这个步骤，我找不到在 Ubuntu 22.04 上使 XDP 完全工作的其他方法，包括像 BTFs、kfuncs 和内核模块等所有功能。稍后会详细介绍这些内容。

### XDP 反射

现在我们将构建并运行一个简单的 XDP 程序。在这个程序中，我们将把发送到我们的端口 40000 的 UDP 数据包原样反射回发送者。所有其他数据包都传递给内核进行常规处理。

首先，从 GitHub 克隆我的 [XDP 示例库](https://github.com/mas-bandwidth/xdp?ref=mas-bandwidth.com)：

```
git clone https://github.com/mas-bandwidth/xdp 
```

切换到 reflect 目录并制作程序：

```
cd xdp/reflect && make 
```

运行UDP反射程序，传入要连接程序的网络接口的名称。您可以使用**ifconfig**列出Linux机器上的网络接口。

```
sudo ./reflect enp4s0 
```

打开另一个终端窗口以查看来自XDP程序的日志：

```
sudo cat /sys/kernel/debug/tracing/trace_pipe 
```

然后再在另一台机器上克隆XDP存储库，并运行相应的反射程序客户端，将192.168.1.40替换为运行XDP程序的Linux机器的IP地址：

```
git clone https://github.com/mas-bandwidth/xdp
cd xdp/reflect && go run client.go 192.168.1.40 
```

如果一切正常，您应该看到如下日志：

```
gaffer@batman reflect % go run client.go
sent 256 byte packet to 192.168.1.40:40000
sent 256 byte packet to 192.168.1.40:40000
sent 256 byte packet to 192.168.1.40:40000
sent 256 byte packet to 192.168.1.40:40000
sent 256 byte packet to 192.168.1.40:40000
sent 256 byte packet to 192.168.1.40:40000
sent 256 byte packet to 192.168.1.40:40000
sent 256 byte packet to 192.168.1.40:40000
sent 256 byte packet to 192.168.1.40:40000
sent 256 byte packet to 192.168.1.40:40000
received 256 byte packet from 192.168.1.40:40000
received 256 byte packet from 192.168.1.40:40000
received 256 byte packet from 192.168.1.40:40000
received 256 byte packet from 192.168.1.40:40000
received 256 byte packet from 192.168.1.40:40000
received 256 byte packet from 192.168.1.40:40000
received 256 byte packet from 192.168.1.40:40000
received 256 byte packet from 192.168.1.40:40000
received 256 byte packet from 192.168.1.40:40000
received 256 byte packet from 192.168.1.40:40000 
```

恭喜，您已经构建并运行了您的第一个XDP程序，而且它不是玩具。如果您只是注释掉**#define DEBUG 1**行，在reflect_xdp.c中，它能够在10G NIC上以线速率反射数据包。

### XDP Drop

接下来我们将运行一个程序，监听一个UDP端口，并丢弃不匹配模式的数据包。这种类型的XDP程序对于加固您的游戏服务器抵御DDoS攻击非常有用，尽管它肯定不是万能解药。

总体思路是对关键数据包进行哈希，例如：数据包长度、源和目标地址和端口，以及如果想要更高级的话，还可以包括每分钟变化的某些滚动魔法数字。虽然这不是完美的，并且无法防止数据包重放攻击，但至少随机生成的UDP数据包将无法通过模式检查。

关键是以可逆的方式将这个8字节哈希分散在数据包开头的15字节中，有效地执行与压缩相反的操作。我们将以非常低效的方式存储这些数据，使得标头中的每个字节都有一个非常窄的有效值范围，并且大多数不是有效值。现在我们有一个极低熵的模式可以检查，甚至无需计算哈希。

这里有一个示例，它使用哈希并填充了一个16字节的标头，其中包括一个低熵编码的哈希的15字节，以及用于数据包类型的第一个字节的保留字节：

```
func GeneratePacketHeader(packet []byte, sourceAddress *net.UDPAddr, destAddress *net.UDPAddr) {

    var packetLengthData [2]byte
    binary.LittleEndian.PutUint16(packetLengthData[:], uint16(len(packet)))

    hash := fnv.New64a()
    hash.Write(packet[0:1])
    hash.Write(packet[16:])
    hash.Write(sourceAddress.IP.To4())
    hash.Write(destAddress.IP.To4())
    hash.Write(packetLengthData[:])
    hashValue := hash.Sum64()

    var data [8]byte
    binary.LittleEndian.PutUint64(data[:], uint64(hashValue))

    packet[1] = ((data[6] & 0xC0) >> 6) + 42
    packet[2] = (data[3] & 0x1F) + 200
    packet[3] = ((data[2] & 0xFC) >> 2) + 5
    packet[4] = data[0]
    packet[5] = (data[2] & 0x03) + 78
    packet[6] = (data[4] & 0x7F) + 96
    packet[7] = ((data[1] & 0xFC) >> 2) + 100

    if (data[7] & 1) == 0 {
        packet[8] = 79
    } else {
        packet[8] = 7
    }
    if (data[4] & 0x80) == 0 {
        packet[9] = 37
    } else {
        packet[9] = 83
    }

    packet[10] = (data[5] & 0x07) + 124
    packet[11] = ((data[1] & 0xE0) >> 5) + 175
    packet[12] = (data[6] & 0x3F) + 33

    value := (data[1] & 0x03)
    if value == 0 {
        packet[13] = 97
    } else if value == 1 {
        packet[13] = 5
    } else if value == 2 {
        packet[13] = 43
    } else {
        packet[13] = 13
    }

    packet[14] = ((data[5] & 0xF8) >> 3) + 210
    packet[15] = ((data[7] & 0xFE) >> 1) + 17
} 
```

要运行xdp drop程序，只需切换到'drop'目录，并在您的网络接口上运行它：

```
cd xdp/drop && sudo ./drop enp4s0 
```

然后在另一台计算机上运行drop客户端，分别替换客户端和drop XDP程序地址：

```
cd xdp/drop && go run client.go 192.168.1.20 192.168.1.40 
```

在XDP机器上，您将看到日志显示数据包过滤器通过了：

```
sudo cat /sys/kernel/debug/tracing/trace_pipe 
```

尝试修改client.go以发送不带标头的随机生成的数据包。现在您将在日志中看到，数据包过滤器丢弃了数据包。哈希的编码非常低熵，以至于几乎不可能有随机生成的数据包通过数据包过滤器。

如果您要在生产环境中使用这种技术，请务必确保将低熵编码更改为游戏中独特的内容，因为脚本小子也会阅读这些文章。此外，确保您的编码是可逆的，这样您就可以在接收端重建哈希并在XDP中丢弃数据包，当哈希与预期值不匹配时。现在人们无法伪造其源地址或端口！

### XDP白名单

那么，更简单的方法呢？为什么不只是维护一个允许与游戏服务器通信的IP地址列表，丢弃任何不是来自白名单地址的数据包呢？

当然，您需要在连接之前在服务器上的后端进行一些工作来“打开”客户端地址，并且在客户端断开连接时进行一些工作来“关闭”地址...但这是可行的，现在，来自随机地址的数据包被XDP丢弃，而不是Linux内核进行任何处理。

要做到这一点，我们需要一种将白名单传递给XDP程序的方法。在这里，我们可以使用BPF的一个新功能：**Maps**。

Map是一组非常丰富的数据结构，您可以从BPF中使用。数组、哈希、每CPU数组、每CPU哈希等等。所有这些数据结构都是无锁的，您可以从BPF程序内部和用户空间程序中读取和写入它们。

如果你明白我现在的意图，你现在有了一种从BPF程序通信到用户空间并反之的方法。现在这几乎太容易了：只需调用用户空间程序中的函数来添加和删除白名单哈希映射中的条目。

运行白名单XDP程序，用您自己的接口名称替换：

```
cd xdp/whitelist && sudo ./drop enp4s0 
```

然后在另一台计算机上运行白名单客户端，分别用客户端和drop XDP程序地址替换地址：

```
cd xdp/whitelist && go run client.go 192.168.1.20 192.168.1.40 
```

如果您查看XDP程序的日志：

```
sudo cat /sys/kernel/debug/tracing/trace_pipe 
```

你会看到它打印出来说它正在丢弃包，因为它们不在白名单中。编辑**whitelist/whitelist.c**，添加运行client.go的机器的地址，然后重新加载XDP程序。再次运行client.go，数据包应该通过。此时，在XDP机器上将UDP套接字绑定到端口40000，它只会接收通过白名单检查的数据包。

如果您在生产中使用这个，您将需要编写自己的系统来添加和删除白名单条目。也许您的服务器定期向后端获取打开地址的列表？也许它订阅某个队列？此外，此示例中的白名单哈希值为空，但您可以在其中放置数据。对于每个客户端的秘密密钥以使数据包过滤器哈希更安全怎么样？您可以将白名单与数据包过滤器和哈希检查结合起来，阻止攻击者伪造其IPv4源地址以通过白名单。

### XDP中继

如果您的游戏服务器即使XDP正在使用白名单、数据包过滤检查和哈希检查丢弃数据包，仍然受到DDoS攻击的超载影响呢？

DDoS攻击变得更大了。大得多。恭喜，您的游戏**非常成功**。为什么不在游戏服务器前面放置一个中继，只转发有效数据包，完全隐藏游戏服务器的IP地址？您可以在每个数据中心保护您的游戏服务器，并且这些中继可以拥有10、40或100gbps的网卡。

我把这个留给读者作为练习。将上述白名单方法与足够的信息结合在白名单哈希值条目中，使中继能够从客户端到服务器和反向转发数据包。

现在尽快丢弃任何不在白名单中的地址的数据包。加分：跟踪每个客户端连接的序列号，以避免重放攻击，并限制每个客户端连接到某个最大带宽信封。这开始成为一个相当稳固的系统。

在这一点上，你基本上拥有了自己的版本的Steam数据中继（SDR），只是这个不是免费的。它很可能甚至比SDR更好。干得好！如果你有无限的资源并且像Valve一样在地下室里有自己的印钞机，你也能负担得起运行这个系统。

### 使用XDP进行网络加速

你知道吗，随时有大约5-10%的玩家遭遇网络性能不佳，例如比平常更高的延迟、高抖动或高丢包率？很难相信，但这是真的。我有来自超过五千万名独特玩家的数据来证明这一点。

更有趣的是，这种网络性能不佳每个月都在变动，影响大约90%的玩家。不是每天同样的5-10%的玩家。

这不仅仅是一小部分网络连接不佳的玩家，这是一个系统性问题。从一场比赛到下一场比赛，网络性能的不一致影响了大多数玩家。

我们的公司[Network Next](https://networknext.com/?ref=mas-bandwidth.com)通过引导游戏数据包经过中继来解决这个问题，而且……我们甚至超越了谷歌云到它们自己的数据中心使用的高级传输，亚马逊全球加速器到它们自己的数据中心，还大大超越了Steam数据中继（SDR）。

Network Next中继已在XDP中实现。

### 在XDP中的加密

实施Network Next中继时遇到的一个问题是，我如何从我的XDP程序内部访问加密？当然我可以快速转发或丢弃数据包，但我决定是否转发或丢弃数据包不仅基于白名单、数据包过滤器和哈希检查，还基于sha256和chachapoly等加密测试。

虽然我肯定可以与验证程序作斗争并直接在BPF中编写自己的加密基元，但这似乎是逆生产的。我将花费大量时间与验证程序作斗争，最终，也许甚至不能在验证程序的限制下实现给定的加密基元。它具有一种不察觉你的代码完全安全的神秘能力。说真的，写完XDP程序后，你真的会想揍它一顿。

救援即将到来的是本文将讨论的BPF的最后两个特性。**BTF**和**kfuncs**。

简而言之，你可以编写自己的内核模块，然后从该模块导出称为内核函数或（kfuncs）的函数，并从 XDP 内部调用它们。甚至可以注释这些函数，以便 BPF 验证器知道，好的，这个函数参数是一个 void 指针 *data*，这是数据的长度 int *data__sz.* 这些注释通过 BTF 完成，BTF 是一种轻量级的类型系统，它从 Linux 内核中导出类型数据，*包括内核模块*，因此可以从 BPF 访问。

使用这个方法，我们可以在我们自己的自定义 **crypto_module** 内核模块中实现这个函数，使用现有的 Linux 内核加密原语执行 sha256。

要查看这个过程，首先构建并加载内核模块：

```
cd xdp/crypto
make module 
```

接下来，构建并运行 XDP 程序，用你自己的网络接口名称替换它：

```
make && sudo ./crypto enp4s0 
```

现在在另一台计算机上，切换到 crypto 目录并运行客户端，用运行 XDP 程序的机器的 IP 地址替换地址：

```
cd xdp/crypto && go run client.go 192.168.1.40 
```

如果一切正常，你将看到 XDP 程序对每个发送的数据包都回应一个32字节的数据包，其中包含你的数据包前256字节的 sha256。为什么只有前256字节？好吧，要理解这一点，你需要了解 BPF 验证器的限制…

### BPF 验证器的限制

有了 kfuncs，似乎你现在应该能够将整个 *void * packet* 和 *int packet__sz* 从 XDP 传递给一个 kfunc，并在内核模块中完全处理它。

哦，别那么快。BPF 验证器有一些限制，这些限制限制了你能做的事情（至少在2024年是这样）。希望 Linux BPF 开发人员将来修复这些问题。

BPF 用于 XDP 程序的最佳描述是它极其“锚定”于从左到右处理数据包。通常，你从数据包的开头开始，可以检查数据包中是否有足够的字节来读取以太网头部，然后将指针向右移动一定的量，读取 IP 头部，再次向右移动一定的量并读取 UDP 头部，依此类推。

但是，如果你试图编写代码来读取数据包中的最后2个字节，我简直找不到任何方法使得这段代码通过验证器，尽管这段代码完全安全，并且不会读取超出边界的内存。

下一个限制是（在2024年看来）你似乎只能将数据包的常量大小部分传递给 kfuncs。例如，我可以检查 UDP 数据包载荷中是否至少有256字节，然后使用指向数据包数据和常量大小为256字节的指针调用 kfunc，这样可以通过验证器。但是，如果你传递从 XDP 上下文派生的数据包的 *实际大小*，似乎无法说服验证器是安全的。

这是一个巨大的遗憾，因为如果我们可以简单地将XDP数据包传递给内核模块并在那里执行一些操作，我们真的可以*大展宏图*。希望这在未来的BPF版本中得到修复。

### 结论

在本文中，我们探讨了XDP和eBPF，这是Ubuntu 22.04 LTS中的一种内核旁路技术，使用的内核版本为6.5。此前不稳定，并且只有“脖子上有胡子”的内核黑客才使用的领域，现在已经足够稳定和成熟，可以被游戏开发者广泛使用。

它是一个令人惊讶的强大且易于使用的系统，一旦你掌握了它的使用技巧。你可以编写一个XDP函数来反射数据包，丢弃它们，转发它们，运行数据包过滤器，执行白名单检查甚至一些加密操作。你可以在10gbps及以上的线速下以最小的工作量完成所有这些操作。查看文章的示例源代码（[https://github.com/mas-bandwidth/xdp](https://github.com/mas-bandwidth/xdp?ref=mas-bandwidth.com)）并自行查看。

我知道听起来很疯狂，但是在未来，我实际上正在探索在XDP几乎完全内部实现整个后端系统和可扩展的游戏服务器的可能性。通过使用映射、内核模块和内核函数，你几乎可以做到*几乎*任何你想做的事情，如果做不到，最坏的情况下可以将数据包传递到用户空间进行处理。例如，如果你在2024年创建一个新的MMO或超高玩家数量的游戏，我认为没有比XDP/eBPF更好的基础技术了。

希望这篇文章能帮助你开始编写XDP程序，它们非常强大且编写起来很有趣，即使验证器让你发疯。期待看到你将会用它创造出什么来！

最好的祝愿，

- Glenn
