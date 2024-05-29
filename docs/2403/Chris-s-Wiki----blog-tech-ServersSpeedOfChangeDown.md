<!--yml
category: 未分类
date: 2024-05-27 14:30:08
-->

# Chris's Wiki :: blog/tech/ServersSpeedOfChangeDown

> 来源：[https://utcc.utoronto.ca/~cks/space/blog/tech/ServersSpeedOfChangeDown](https://utcc.utoronto.ca/~cks/space/blog/tech/ServersSpeedOfChangeDown)

One of the bits of technology news that I saw recently was that AWS was changing how long it ran servers, from five years to six years. Obviously one large motivation for this is that it will save Amazon a nice chunk of money. However, I suspect that one enabling factor for this is that old servers are more similar to new servers than they used to be, as part of what could be called the great slowdown in computer performance improvement.

New CPUs and to a lesser extent memory are somewhat better than they used to be, both on an absolute measure and on a performance per watt basis, but the changes aren't huge the way they used to be. SATA SSD performance has been more or less stagnant for years; NVMe performance has improved, but from a baseline that was already very high, perhaps higher than many workloads could take advantage of. Network speeds are potentially better but it's already hard to truly take advantage of 10G speeds, especially with ordinary workloads and software.

(I don't know if SAS SSD bandwidth and performance has improved, although raw SAS bandwidth has and is above what SATA can provide.)

For both AWS and people running physical servers (like [us](https://support.cs.toronto.edu/)) there's also the question of how many people need faster CPUs and more memory, and related to that, how much they're willing to pay for them. It's long been observed that a lot of what people run on servers is not a voracious consumer of CPU and memory (and IO bandwidth). If your VPS runs at 5% or 10% CPU load most of the time, you're probably not very enthused about paying more for a VPS with a faster CPU that will run at 2.5% almost all of the time.

(Now that I've written this it strikes me that this is one possible motivation for cloud providers to push 'function as a service' computing, because it potentially allows them to use those faster CPUs more effectively. If they're renting you CPU by the second and only when you use it, faster CPUs likely mean more people can be packed on to the same number of CPUs and machines.)

[We](https://support.cs.toronto.edu/) have a few uses for very fast single-core CPU performance, but other than those cases (and [our compute cluster](/~cks/space/blog/sysadmin/SlurmHowWeUseIt)) it's hard to identify machines that could make much use of faster CPUs than they already have. It would be nice if [our fileservers](/~cks/space/blog/linux/ZFSFileserverSetupIII) had [U.2 NVMe drives](/~cks/space/blog/tech/ServerNVMeU2U3AndOthers2022) instead of SATA SSDs but I'm not sure we'd really notice; the fileservers only rarely see high IO loads.

PS: It's possible that I've missed important improvements here because I'm not all that tuned in to this stuff. One possible area is PCIe lanes directly supported by the system's CPU(s), which enable all of those fast NVMe drives, multiple 10G or faster network connections, and so on.