<!--yml

category: 未分类

date: 2024-05-27 14:30:08

-->

# Chris's Wiki :: blog/tech/ServersSpeedOfChangeDown

> 来源：[https://utcc.utoronto.ca/~cks/space/blog/tech/ServersSpeedOfChangeDown](https://utcc.utoronto.ca/~cks/space/blog/tech/ServersSpeedOfChangeDown)

One of the bits of technology news that I saw recently was that AWS was changing how long it ran servers, from five years to six years. Obviously one large motivation for this is that it will save Amazon a nice chunk of money. However, I suspect that one enabling factor for this is that old servers are more similar to new servers than they used to be, as part of what could be called the great slowdown in computer performance improvement.

New CPUs and to a lesser extent memory are somewhat better than they used to be, both on an absolute measure and on a performance per watt basis, but the changes aren't huge the way they used to be. SATA SSD performance has been more or less stagnant for years; NVMe performance has improved, but from a baseline that was already very high, perhaps higher than many workloads could take advantage of. Network speeds are potentially better but it's already hard to truly take advantage of 10G speeds, especially with ordinary workloads and software.

(I don't know if SAS SSD bandwidth and performance has improved, although raw SAS bandwidth has and is above what SATA can provide.)

对于 AWS 和像 [我们](https://support.cs.toronto.edu/) 这样运行物理服务器的人来说，还有一个问题，那就是有多少人需要更快的 CPU 和更多的内存，以及与此相关的他们愿意为此支付多少的问题。长期以来观察到，人们在服务器上运行的许多内容并不是对 CPU、内存（和 IO 带宽）的大量消耗者。如果你的 VPS 大部分时间只是 5% 或 10% 的 CPU 负载，那么你可能对为了更快的 CPU 而支付更多的 VPS 不是很感兴趣，因为它几乎全天候只会以 2.5% 的负载运行。

(Now that I've written this it strikes me that this is one possible motivation for cloud providers to push 'function as a service' computing, because it potentially allows them to use those faster CPUs more effectively. If they're renting you CPU by the second and only when you use it, faster CPUs likely mean more people can be packed on to the same number of CPUs and machines.)

[We](https://support.cs.toronto.edu/) have a few uses for very fast single-core CPU performance, but other than those cases (and [our compute cluster](/~cks/space/blog/sysadmin/SlurmHowWeUseIt)) it's hard to identify machines that could make much use of faster CPUs than they already have. It would be nice if [our fileservers](/~cks/space/blog/linux/ZFSFileserverSetupIII) had [U.2 NVMe drives](/~cks/space/blog/tech/ServerNVMeU2U3AndOthers2022) instead of SATA SSDs but I'm not sure we'd really notice; the fileservers only rarely see high IO loads.

PS：由于我对这些东西并不是特别了解，可能会漏掉一些重要的改进。一个可能的领域是由系统的CPU直接支持的PCIe通道，这使得所有这些快速的NVMe驱动器、多个10G或更快的网络连接等成为可能。
