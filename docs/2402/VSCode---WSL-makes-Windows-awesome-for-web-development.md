<!--yml

类别：未分类

日期：2024-05-29 13:25:40

-->

# VSCode + WSL 让 Windows 在 web 开发中变得出色

> 来源：[https://world.hey.com/dhh/vscode-wsl-makes-windows-awesome-for-web-development-9bc4d528](https://world.hey.com/dhh/vscode-wsl-makes-windows-awesome-for-web-development-9bc4d528)

我有点震惊。Windows 真的变得非常适合 web 开发人员了。在

[VSCode](https://code.visualstudio.com/)

,

[WSL](https://learn.microsoft.com/en-us/windows/wsl/about)

，以及英特尔的最新桌面芯片，我已经使用 PC 工作了一周多，这台机器比 M3 Max 运行我的编程测试更快，配备了出色的窗口管理器，总体感觉完全可以作为 macOS 的一种完全可行的替代品来处理 web 开发。

哎呀，不仅仅是可行，而且在许多方面更好。WSL 让您能够原生运行真正的 Linux 发行版，因此您可以使用与您将部署到的相同软件包管理器。哦，而且由于它是 x86 架构，您构建 Docker 镜像的速度将是在 arm64 上多体系结构构建的一小部分时间。

现在确实如此，要想达到这种速度，你需要使用台式机。我购买的 Dell 配备了英特尔 i9-14900K，这就是你可以获得 500+ 分的方法。

[Speedometer 2.1](https://browserbench.org/Speedometer2.1/)

得分，超快的 Docker 构建和创纪录的编程测试运行。这台“野兽”的功耗显然远高于 M 系列芯片。

但我某种程度上认为 M3 Max 是当今主流计算机中无可争议的速度之王。事实并非如此。那

[价值 $3,200 的 M3 Max MacBook](https://www.apple.com/shop/buy-mac/macbook-pro/14-inch-space-black-apple-m3-max-with-14-core-cpu-and-30-core-gpu-36gb-memory-1tb)

实际上可以被一款

[价值 $2,349 的 Dell XPS](https://www.dell.com/en-us/shop/desktop-computers/new-xps-desktop/spd/xps-8960-desktop/usexpsthcto8960rpl27)

在大多数 web 开发人员看来最为重要的测试中表现出色。特别是对于那些 95% 的时间都把笔记本电脑放在桌面上的人来说。

我也尝试了

[Framework 13 笔记本](https://frame.work/)

所打动，该设备采用了 AMD Ryzen 7040 CPU。这是一款非常不错的芯片，但在性能上仍稍逊 M 系列。它在 Speedometer 2.1 上的跑分约为 300-350，而在我们的 HEY 测试套件上约为 3 分 31 秒。我的 M2 Pro 笔记本在 Speedometer 上可以达到超过 500 分(!)，并且能在 2 分 16 秒内完成测试套件（尽管在重复运行时会迅速降频到约 3 分钟）。

因此，苹果显然在笔记本领域仍具有一定优势。尽管我觉得 Framework 13 的电池续航已经非常充足了。

这些表现出色的桌面设备，例如 i9-14900K，展示了 Windows 在超越 M 系列芯片的可能性。这款 CPU 在我们的 HEY 测试套件中跑出了我见过的最快速度，仅需 1 分 37 秒。而且在 Speedometer 2.1 上也能超过 500 分。

但即使在笔记本电脑方面，备选方案也有很多值得喜欢的地方。Framework 在可修复性和升级方面有一个非常新颖的方法。你可以轻松地自己更换任何组件，通常无需工具。在蝴蝶键盘的恐怖时代经历了多次键盘更换的人一定会感受到这有多么有帮助。

更令人印象深刻的是，你甚至可以

[持续升级 CPU 和主板](https://frame.work/marketplace/mainboards)

on the Framework! 他们首先推出了第 11 代英特尔芯片，然后发布了第 12 代主板，现在又推出了这个 AMD 变体。你可以在所有这些代数中保持同一台笔记本电脑。非常酷。这确实让我感到有点不好意思，因为我从 M1 到 M2 再到 M3 都买了全新的笔记本电脑。

所以在 Windows 阵营近期有很多值得喜欢的地方。M芯片的震撼统治已经减弱，即使在笔记本领域仍然有优势，但远不及以往。这让我感到意外。但更让我感到意外的是微软如何成功地将 Linux 与 Windows 整合。我本以为在 WSL 下会有性能损失。或者会显得笨拙和不稳定。但事实并非如此。

VSCode 在这里的关键在于。它出厂自带了与 WSL 非常好的集成，所有文件实际上都存储在 Linux 子系统内部，因此没有我记得的上一次使用 Windows 时那种跨文件系统的旧慢速度。

你所需做的一切就是打开 Powershell，输入“wsl --install”，选择你的用户名/密码，你就可以运行 Ubuntu 22.04 了。然后安装 VSCode，它会自动检测到已安装 WSL，然后你可以在 Ubuntu 终端中输入“code .”来打开 Windows 端任何目录中的 VSCode。这就是如此流畅！

在看到这一切，并且与之共处了一周之后，我认真考虑每天使用桌面驱动程序。但最终我没有这样做。原因是我还不愿意（还是说现在不愿意？）放弃

[TextMate](https://macromates.com/)

对于 VSCode 或者不想处理 Windows 上的字体渲染。我怀疑这两者都是相当特别的原因，几乎肯定不适用于广大的 web 开发者。大多数已经接受了 VSCode，可能对字体渲染并不像我这样挑剔。

而这真是令人兴奋的！Mac 终于为那些需要在网页工作并需要可靠的 Unix 基础的人提供了一个无需道歉的选择。对于苹果来说，迫切需要竞争来吸引 web 开发者的注意力。

哦，我提到了你实际上可以在 PC 上玩 AAA 游戏吗？那台 $2,349 的 Dell 配备了 NVIDIA 4070，足以驱动从 Fortnite 到 Cyberpunk 2077 的所有内容。这些游戏在 Mac 上根本不存在，也许永远都不会存在。

野生的是我们已经走了一整圈。对于我认识的所有科技人员来说，微软在90年代末是头号大反派，而苹果则是被宠爱的小弟弟。现在情况正好相反。在2001年转向OSX感觉像是反叛的举动。而现在，2024年如果你敢跑Windows，可能会成为许多社区中的异类。至少我希望在未来的技术会议上看到大量的Windows机器，甚至是Framework笔记本电脑。

我也在家里保留这两台PC。谁知道，如果有人最终移植了TextMate的全色完美版本的

[万圣节](https://github.com/textmate/themes.tmbundle/blob/master/Themes/All%20Hallow's%20Eve.tmTheme)

主题方面，我可能会再给VSCode一个机会。如果以某种方式能实现Mac风格的字体渲染，我看不出有什么能阻止我的（不，MacType的黑客并不行）。

因此，向微软致敬！通过与Linux和开源的和解，你们为像我这样的开发人员提供了一组极其高效的工具组合。而现在，来自Intel和AMD的芯片终于在硬件方面给了苹果一场竞争。我真是喜欢看到这一切，也喜欢这种积极的惊喜感。
