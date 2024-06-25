<!--yml

category: 未分类

date: 2024-05-27 14:40:48

-->

# 虚拟化技术是如何应用到苹果芯片 Mac 的 – The Eclectic Light Company

> 来源：[`eclecticlight.co/2024/01/11/how-virtualisation-came-to-apple-silicon-macs/`](https://eclecticlight.co/2024/01/11/how-virtualisation-came-to-apple-silicon-macs/)

在 2020 年 6 月苹果宣布推出首款搭载苹果芯片的 Mac 之前的几周，外界就对它们如何运行现有的 macOS 软件展开了激烈的猜测。尽管大多数人正确地认识到这将通过翻译 x86 代码来实现，就像 2006 年从 PowerPC 过渡到 Intel 时的 Rosetta 所做的那样，但也有人看到了虚拟化的作用。当时，[我写道](https://eclecticlight.co/2020/06/12/what-would-an-arm-based-mac-mean-to-us/)，“苹果可以提供一个完整的虚拟化层，让用户运行大多数的英特尔 Mac 应用程序，甚至可以安装并运行 Windows。”

几天后，Craig Federighi 描述了支持在苹果芯片上运行各种应用程序的三个支柱：包含两种架构代码的通用应用程序，Rosetta 2 用于翻译 x86 代码，还有虚拟化。当时，虚拟化主要用于托管 Linux 和 Docker，Andreas Wendker 展示了 Parallels Desktop 的预览版作为 Linux 的客户端。

#### Hypervisor

苹果最初在 2014 年的 OS X 10.10 Yosemite 中就为虚拟化的核心需求，[虚拟机监控程序](https://developer.apple.com/documentation/hypervisor)提供了支持。它提供了在用户空间虚拟化的 C API，无需内核扩展。在 Intel Mac 上，此 API 提供了 VT-x 功能集的硬件支持，包括扩展页表（EPT）和无约束模式。但这个 API 并没有提供 VirtIO 设备驱动程序的支持。

#### VirtIO

与此同时，为苹果芯片 Mac 的准备工作一直在紧锣密鼓地进行。大约在 Mojave 的时候、也就是 2018-19 年之间，出现了两个新的内核扩展，显然是为了在 AppleVirtIO 和 AppleVirtualGraphics 中提供对 VirtIO 的初始支持，在 2019 年 7 月的 10.14.6 升级中，它们达到了 2.1.3 版本，这是在首个开发者转换套件发行的一年前。在 Catalina 中这两个迅速发展，不久之后当苹果宣布针对 Apple 芯片时达到了 16.140.6 版本。

Big Sur 为 macOS 带来了进一步的增强，使得在虚拟机中运行 Linux 成为可能。Parallels 公开表示与苹果合作。内核扩展的数量从最初的 2 个增加到 5 个，新增了 AppleParavirtGPU、AppleParavirtGPUMetal 和 AppleVirtualMCA。VirtIO、AppleVirtIO 和 AppleVirtualGraphics 也升级到了 74.50.1 版本。两个框架加入了这些内核扩展：ParavirtualizedGraphics 和 Virtualization。

#### 虚拟化扩展

更根本地说，Big Sur 引入了苹果所称的“虚拟化扩展”，可能是由 Arm 记录的 AArch64 虚拟化所推动。这个功能包括额外的“异常级别”，EL2 hypervisor，提供第 2 阶段转换，EL1/0 指令和寄存器访问捕获以及虚拟异常生成。第 2 阶段转换允许 hypervisor 控制 VM 可访问的内存映射资源，并且在 VM 的地址空间中出现的位置，增强了由操作系统控制的第 1 阶段转换。捕获允许 hypervisor 捕获操作，例如配置低级控制，并在必要时对其进行仿真。

虽然 Big Sur 为 Andreas Wendker 的简短演示提供了足够的支持，但仍然不足以满足发布所需的要求，即使在 11.5.2 的附加支持之后也是如此。在那个阶段，有十个内核扩展

+   AppleParavirtGPU

+   AppleParavirtGPUIOGPUFamily

+   AppleParavirtGPUMetal

+   AppleParavirtGPUMetalIOGPUFamily

+   AppleParavirtIOSurface

+   AppleVirtIO

+   AppleVirtIOStorage

+   AppleVirtualGraphics

+   AppleVirtualMCA

+   AppleVirtualPlatform

还有一些仍处于第一版本的内核扩展。苹果还添加了一个音频插件，AppleVirtIOSound。

#### Launch

在 WWDC 2022 上，以本杰明·普兰（Benjamin Poulain）的演示来解释如何简单实现 macOS 和 Linux 在 Apple 硅片上的轻量级虚拟化，并在 Monterey 上提供了有限的支持。他总结说虚拟化团队“迫不及待地想看到你们下一步会做什么”。尽管引人注意的是，macOS 客户端和主机之间共享文件夹的支持仍然未到位，这需要等待 Ventura 添加 virtio 文件系统，以及内建神经引擎（ANE）的支持，后者在同一时间增加了自己的内核扩展 AppleVirtIONeuralEngineDevice。此后，Sonoma 还增加了两个内核扩展，AppleVideoToolboxParavirtualization 和 AppleVirtIOBiometrics，以及用于 EventTap、Installation、LinuxRosetta 和 VirtualMachine 的虚拟化框架的 XPC 服务。

除了 hypervisor 和 VirtIO 支持之外，Apple 硅片 Mac 运行客户端 macOS 还有其他障碍需要克服。标准的 macOS 安装程序应用程序不足以构建一个可工作的 VM，因为它不包含大部分所需的“固件”组件。幸运的是，这些已经包含在用于在 DFU 模式下对 Apple 硅片 Mac 进行完全恢复的 IPSW 镜像中，并提供了安装 macOS VM 的所有组件。

截至目前，索诺玛（Sonoma）版本仍有几个尚未解决的问题，苹果尚未解决。其中最重要的是苹果 ID 支持的棘手问题，对此苹果一直保持沉默。对于那些在他们的超级芯片上有足够性能核心的人来说，苹果在任何 Mac 上同时运行的最多两个 macOS 虚拟机的许可限制是不必要的限制，因为他们可以虚拟化任意数量的 Linux 实例。然而，少数人希望能够通过在另一个虚拟机内运行虚拟机来进行嵌套虚拟化。最后，欧洲和美国以外的许多国家的人都希望苹果在 macOS 虚拟机中支持 ISO 键盘布局，这仍然是一个令人惊讶的短板。我相信未来还会有更多的内容。

#### 里程碑

+   2014 年 OS X 10.10：Hypervisor API（英特尔）。

+   2018 年 macOS 10.14：VirtIO 内核扩展。

+   2020 年 macOS 11 Big Sur：演示 Linux 虚拟机，为苹果硅提供虚拟化扩展。

+   2021 年 macOS 12 Monterey：首次公开支持。

+   2022 年 macOS 13 Ventura：包括 virtiofs 共享在内的全面支持。

+   2023 年 macOS 14 Sonoma：虚拟显示和其他增强功能。
