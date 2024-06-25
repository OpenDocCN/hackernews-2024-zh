<!--yml

类别：未分类

日期：2024 年 5 月 27 日 14:56:12

-->

# 驱动程序篡改使您可以在 Windows BSOD 后运行 Linux，无需重新启动-故障检查失败允许在操作系统崩溃后运行程序|汤姆的硬件

> 来源：[`www.tomshardware.com/software/operating-systems/driver-hack-lets-you-run-linux-after-windows-bsods-no-reboot-required`](https://www.tomshardware.com/software/operating-systems/driver-hack-lets-you-run-linux-after-windows-bsods-no-reboot-required)

通常，在 Windows 中发生蓝屏死机（BSOD）会导致您的 PC 自动重启，但[程序员 NSG650](https://github.com/NSG650/BugCheck2Linux)（通过[NTDEV](https://www.youtube.com/watch?v=XUsfSQPrqrM'))已经创建了一个驱动程序，它会让你的 PC 启动 Linux 模拟器。 尽管这个驱动程序更多是一个新奇之举，而不是实际上有用，但它是对如何简单地利用软件功能的巧妙展示，只是通过以一种非预期的方式使用它们。

这个驱动程序的工作原理实际上相当简单——它只是利用了 Windows 中内置的 Bug Check 回调功能。 Bug Check 只是崩溃或蓝屏的技术名称，当发生 Bug Check 时，Windows 想要知道原因。 作为 Bug Check 回调例程的一部分，[驱动程序](https://www.tomshardware.com/tag/drivers)可以"将设备重置为已知状态"，根据[Microsoft 的 Windows 编码手册](https://click.linksynergy.com/deeplink?id=kXQk6%2AivFEQ&mid=24542&u1=tomshardware-us-1222065713088901764&murl=https%3A%2F%2Flearn.microsoft.com%2Fen-us%2Fwindows-hardware%2Fdrivers%2Fkernel%2Fwriting-a-bug-check-callback-routine)。 换句话说，它可以在崩溃后仍运行代码。

虽然大多数驱动程序会利用这个机会向崩溃转储文件添加诊断数据，NSG650 的驱动程序插入了一个 RISC-V Linux 模拟器，这可能是对[Microsoft](https://www.tomshardware.com/tag/microsoft)的 Bug Check 回调函数的轻微误用。 显而易见，这与 PC 的[双引导](https://www.tomshardware.com/how-to/dual-boot-linux-and-windows-11)不同，双引导意味着它同时安装了 Windows 和 Linux。

在这个模拟器中，你不能做太多事情，因为它基本上只是 DOS 或命令行操作系统，而不是完全成熟的 Ubuntu 或 Arch Linux。 如果你打错字，甚至不能用退格键，而且你必须使用大写锁定键而不是 Shift 键来大写字母；这就是这个模拟器有多受限制。

然而，这个[RISC-V Linux 模拟器](https://github.com/cnlohr/mini-rv32ima)非常小，只有 400 行代码。 相比之下，全面的 Linux 内核本身就有数百万行代码。 这似乎要么无法运行完整的 Linux 发行版，要么是要让它工作太麻烦了，鉴于这不是任何人可能会认真使用的东西，这都是一个合理的理由。

虽然这个驱动程序更多或更少只是关于 Windows 和 Linux 的一个有趣的笑话，但它确实提出了利用相同的 bug 检查回调功能做更多事情的可能性。目前还不清楚能做什么和不能做什么，但是如果在崩溃后能够运行模拟器，那么肯定也可以做其他事情。这一切都是在假设微软不再审视 Windows 的这个功能，并得出结论说这个功能实在太容易被利用了。

获取 Tom's Hardware 最好的新闻和深度评论，直接发送到您的收件箱。
