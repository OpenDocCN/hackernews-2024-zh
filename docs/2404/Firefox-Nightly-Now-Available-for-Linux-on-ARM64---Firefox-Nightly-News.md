<!--yml

分类：未分类

日期：2024-05-27 13:22:10

-->

# Firefox Nightly 现已支持 Linux ARM64 – Firefox Nightly 新闻

> 来源：[https://blog.nightly.mozilla.org/2024/04/19/firefox-nightly-now-available-for-linux-on-arm64/](https://blog.nightly.mozilla.org/2024/04/19/firefox-nightly-now-available-for-linux-on-arm64/)

我们很高兴向在 ARM64（也称为 AArch64）架构上运行 Linux 的用户分享更新。

### ARM64 架构现已提供二进制版本

[发布 Firefox Nightly .deb 软件包](https://blog.nightly.mozilla.org/2023/10/30/introducing-mozillas-firefox-nightly-deb-packages-for-debian-based-linux-distributions/)后，反馈显示用户对 ARM64 构建的需求很高。为此，我们很高兴地宣布，现在可以提供 Firefox Nightly 的 ARM64 版本，包括 `.tar` 归档和 `.deb` 软件包。请继续提出建议 – 我们随时欢迎您的反馈！

+   **.tar 归档：** 偏爱我们传统的 `.tar.bz2` 二进制文件吗？您可以从 [我们的下载页面](https://www.mozilla.org/firefox/all/#product-desktop-nightly) 上选择 `Linux ARM64/AArch64` 的 `Firefox Nightly` 下载它们。

+   **.deb 软件包：** 欲通过我们的 APT 仓库进行更新和安装，请[按照这些说明操作](https://support.mozilla.org/kb/install-firefox-linux#w_install-firefox-deb-package-for-debian-based-distributions)，并安装 `firefox-nightly` 软件包。

### 关于 ARM64 构建的稳定性

我们希望诚实地告知大家我们目前 ARM64 构建的状况。尽管我们对 Firefox 在这一架构上的质量有信心，但我们仍在将全面的 ARM64 测试融入到 Firefox 的持续集成和发布流程中。我们的目标是将 ARM64 构建整合到 Firefox 广泛的自动化测试套件中，从而能够在 beta、发布和 ESR 渠道中提供这一架构版本。

### 您的反馈至关重要

我们鼓励您下载新的 ARM64 Firefox Nightly 二进制文件，进行测试，并与我们分享您的发现。通过使用这些构建并报告任何问题，您将帮助我们的开发人员更好地支持和测试这一架构，最终实现稳定可靠的 ARM64 版 Firefox。请通过 [Bugzilla](https://bugzilla.mozilla.org/enter_bug.cgi?format=__default__&blocked=1867368&product=Release%20Engineering&component=General&rep_platform=ARM64&op_sys=Linux) 分享您的发现，并关注更多更新。感谢您在 Firefox Nightly 社区中的持续参与！
