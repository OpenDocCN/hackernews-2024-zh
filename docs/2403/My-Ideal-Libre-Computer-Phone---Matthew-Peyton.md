<!--yml

category: 未分类

date: 2024-05-29 12:47:29

-->

# 我的理想自由计算机+手机 | 马修·佩顿

> 来源：[https://mpeyton.com/posts/my_ideal_libre_computer_phone/](https://mpeyton.com/posts/my_ideal_libre_computer_phone/)

<content>我对我的“理想”计算机有非常清晰的想法。它将融合我今天已经使用的所有不同计算机（包括智能手机！）的最佳特性，集成到一台机器中。

## 什么

首先，它将具有典型智能手机的外形因子。这将使我能够随身携带，并且单手和双手均可使用。我甚至可以将其设计为“热狗式折叠”，以便更舒适地阅读。这种形式因子还将使我能够将其放在桌子上，而无需使用外部支架。

第二，它将拥有强大且开源的内部配置。它将具有与苹果M系列相当的性能和效率的芯片。尽可能多的RAM（> 16GB）、存储（> 1TB）和电池（> 4000 mAh），以适应其尺寸。一个USB-C端口。没有多余的东西……没有花哨的东西。所有这些都将是100%开源的，文档齐全，并且没有黑匣子。

第三，它将运行基于GNU+Linux的完全自由开源操作系统。用户将在这个操作系统上拥有完全的根权限，它将是一个真正的Linux发行版，而不是Android的衍生版。它将使用一种声明式的软件包管理风格。它将能够实时切换其显示和窗口管理器，以适应正在使用的任何形态因子。仿真器和兼容性层将允许无缝使用为其他系统编写的应用程序和程序。

第四，它将能够“插入”外部模块。想象一下将其插入到USB-C坞站在桌上。该坞站随后连接到鼠标、键盘、显示器、网络摄像头和外部GPU。这将允许计算机用于编码或图形设计。类似的笔记本外壳形式的坞站也可以使用。靠近电视的坞站可以用来将其转变为家庭娱乐系统或视频游戏主机。甚至可以连接VR头显。蓝牙和WiFi将打开更多可能性。

## 为什么

这台计算机将解决我对计算机的许多直接问题，以及许多二阶问题。

它将许多设备整合为一个单一设备。这消除了在设备之间同步文件和程序的需求。这反过来意味着许多需要同步的专有云软件可以被放弃。包括那些因不得不使用云中介而臭名昭著的手机应用程序。这也意味着程序可以与它们操作的数据分离。让笔记应用只是一个简单的文本编辑器在一个 .txt 文件上，无需同步，将大大简化做笔记的过程。（假设 UI/UX 能够适应不同的形态因素。）这一好处将延伸到我使用的所有其他应用程序和程序。想象一下电影、书籍、电视节目、游戏、论文、文件等。无需同步。可由您选择的任何程序读写。

它还将显著改善隐私、安全性和自由度。如果设备上的一切都是 FOSS，包括其硬件，那么一切都可以完全审计。不同步也极大地提高了数据隐私和安全性。操作系统和软件都是 FOSS 还为设备的使用提供了很多自由。想在手机上运行 cron 作业？随你去吧！

此外，它鼓励我投入更多时间使计算机变成我自己的。我经常不费力地将所有文件、视频、游戏、程序、桌面自定义、工具、脚本等同步到所有设备上。有时不可能（例如，传统智能手机上无法运行有用的 Python 脚本）或者只是非常痛苦（例如，在智能手机和笔记本电脑上重新创建相同的系统颜色主题）。使用声明性 Linux 操作系统也将大大有助于此。

最后，它的灵活性和模块化为我们开启了许多选择。如果你总是有一个大小与手机相同的设备，并且上面装有所有你的游戏，那么随时随地玩游戏将会变得更好。特别是如果你可以将蓝牙控制器连接到它上面。功率不足？只需将其连接到电视的 eGPU……然后玩任何你想玩的游戏。或者在厨房看烹饪视频时，拔下插头并将其放在上面。然后稍后将其插入其笔记本外壳，在沙发上写作。

这听起来可能微不足道，但我认为消除这些摩擦将极大地提高我日常计算的使用体验。一切都是完全开源、经过审计和安全的事实也会让我无比放心。

## 这怎么实现的

拼图的各个部分已经存在，但没有人能够将它们组合在一起。

Apple 在统一他们的各种设备方面做得很出色……但它们仍然是你必须在之间切换的独立硬件。

[Purism](https://puri.sm/) 在试图加速开发融合 Linux 手机和其他设备方面也做得很出色……但他们还有很长的路要走。[许多 Linux 手机操作系统和软件正在崭露头角](https://linmob.net/)，但仍处于早期阶段。

折叠手机等设备目前仍然是Android OEM的专属领域&mldr; 并且 [Linux上的eGPU确实可行](https://wiki.archlinux.org/title/External_GPU)，但有很多限制。对于所有这些用例，USB-C扩展坞也还不完美。笔记本外壳虽然存在，但只是勉强存在。

房间里的大象是自由开源硬件。RISC-V似乎正在赢得人气&mldr; 但现在还处于早期阶段，并且 [可能会面临一些阻力](https://cset.georgetown.edu/article/risc-v-what-it-is-and-why-it-matters/)。其余硬件也需要同样是开源的。这可能是最远的一部分。

## **时间节点**

我在关注影响这台“终极”计算机制造能力的发展。我认为虽然各个组成部分的进展缓慢&mldr; 它们都在平行地向前推进。鉴于此，我预计很多年来看起来都是无望的&mldr; 直到突然间局势改变，每个组件达到某个关键阈值。然后只需将它们整合在一起。

记住，在Proton出现之前，Linux上的游戏看起来无望。[现在五年过去了，](https://www.gamingonlinux.com/2023/08/5-years-ago-valve-released-proton-forever-changing-linux-gaming/) 在许多情况下，Linux上的游戏 [超过了Windows上的游戏。](https://www.reddit.com/r/linux_gaming/comments/16vhe6a/why_does_proton_sometimes_run_games_better_than/) 传统厂商总是看似无法撼动，新进入者总是显得无望，直到突然局势逆转。

我具体追踪的事项包括：

+   **硬件：** [Google Pixels](https://store.google.com/us/category/phones?hl=en-US), [Pixel Fold](https://store.google.com/us/product/pixel_fold?hl=en-US&pli=1), [Purism Librem 5](https://puri.sm/products/librem-5/), [Pinephone Pro](https://pine64.org/devices/pinephone_pro/)

+   **操作系统：** [GrapheneOS](https://grapheneos.org/), [PureOS](https://www.pureos.net/), [NixOS](https://nixos.org/), [PostmarketOS](https://postmarketos.org/)

+   **桌面环境：** [Wayland](https://arewewaylandyet.com/), [Maui Shell](https://github.com/Nitrux/maui-shell), [GNOME](https://www.gnome.org/), [sxmo](https://sxmo.org/)

+   **融合Linux软件：** [众多&mldr;](https://tuxphones.com/convergent-linux-phone-apps/)

+   **兼容层：** [Wine](https://www.winehq.org/), [Proton](https://github.com/ValveSoftware/Proton), [Waydroid](https://waydro.id/), [Darling](https://www.darlinghq.org/)

+   **扩展坞：** [众多&mldr;](https://www.pcworld.com/article/402858/the-best-usb-c-hubs-for-your-laptop-tablet-or-2-in-1.html)

+   **外接GPU：** [众多&mldr;](https://egpu.io/best-egpu-buyers-guide/)

+   **笔记本壳：** [Nexdock](https://nexdock.com/), [其他选择](https://liliputing.com/5-laptop-docks-that-let-you-use-a-smartphone-like-a-notebook/)

希望我能在一年或两年后发布一个跟进，看看事情的进展如何。</content>
