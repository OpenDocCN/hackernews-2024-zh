<!--yml

类别：未分类

日期：2024-05-27 13:12:59

-->

# Gregory Szorc 的数字之家 | 实现完全开源的苹果代码签名和认证的实现

> 来源：[https://gregoryszorc.com/blog/2022/08/08/achieving-a-completely-open-source-implementation-of-apple-code-signing-and-notarization/](https://gregoryszorc.com/blog/2022/08/08/achieving-a-completely-open-source-implementation-of-apple-code-signing-and-notarization/)

作为我之前在 [纯 Rust 实现的 Apple 代码签名](/blog/2021/04/14/pure-rust-implementation-of-apple-code-signing/)（2021-04-14）和 [扩展苹果生态系统访问权限的开源、多平台代码签名](/blog/2022/04/25/expanding-apple-ecosystem-access-with-open-source,-multi-platform-code-signing/)（2022-04-25）中博客过的内容，我一直在使用 Rust 编程语言进行 Apple 代码签名和认证的开源实现。这体现在 `apple-codesign` 的库及其 `rcodesign` 的 CLI 可执行文件中。([文档](https://gregoryszorc.com/docs/apple-codesign/stable/) / [GitHub 项目](https://github.com/indygreg/apple-platform-rs/tree/main/apple-codesign) / [crates.io](https://crates.io/crates/apple-codesign))。

我在四月最新的那篇文章中表示，对于这个实现的相对稳定性我感到非常满意：我们能够对 Mach-O 二进制文件、目录捆绑包（如 `.app`、`.framework` 捆绑包等）、XAR 存档/平面包装/.pkg 安装程序和 DMG 磁盘映像进行签名、认证和加固。除了已知的限制之外，如果苹果的官方 `codesign` 和 `notarytool` 工具支持它，我们也支持。**这使得人们能够在像 Linux 和 Windows 这样的非苹果操作系统上签名、认证和发布苹果软件**。这为访问苹果平台打开了新的途径。

在之前版本的 `apple-codesign` 库中的一个主要限制是我们依赖于苹果的 [Transporter](https://help.apple.com/itc/transporteruserguide/) 工具进行认证。Transporter 是一款 Java 应用程序，可用于 macOS、Linux 和 Windows，并可与苹果的服务器通信，可以上传资产到他们的认证服务。当时我使用这个工具，因为它似乎是苹果官方支持的，并且是建立认证的最小阻力路径。但是 Transporter 使用起来有点棘手，是一个额外需要安装的依赖。

在 WWDC 2022 上，苹果[宣布](https://developer.apple.com/videos/play/wwdc2022/10109/)了一个新的[Notary API](https://developer.apple.com/documentation/notaryapi)，作为 App Store Connect API 的一部分。苹果甚至明确提到利用该 API 从 Linux 进行公证，这让我觉得他们直接在向我眨眼！我一看到这个消息就知道，只是时间问题，我就能用一个纯 Rust 客户端替换 Transporter 来使用新的 HTTP API。（我已经在考虑使用`notarytool`使用的未发布的 HTTP API。从 WWDC 之前我有的有限反向工程笔记看，新的官方 Notary API 看起来非常类似 - 可能与`notarytool`使用的内容完全相同。所以感谢苹果开放了这一访问！）

**我非常激动地宣布，我们现在在 `apple-codesign` crate 中有了一个纯 Rust 实现的 Apple Notary API 客户端！这意味着，我们现在可以从任何能编译 Rust crate 的机器上公证苹果软件。这意味着我们不再依赖第三方的 Apple Transporter 应用程序。与代码签名一样，公证也完全使用开源 Rust 代码实现。**

尽管我对宣布这一新功能感到非常激动，**更让我兴奋的是，这主要是由贡献者 Robin Lambertz / [@roblabla](https://github.com/roblabla) 实现的！** 他们在 WWDC 2022 进行期间提出了一个 GitHub 功能请求，几天后提交了一个 PR。我花了几个月才找时间审查它（我尽量避免在夏天使用电脑屏幕），但鉴于变更的范围，这确实是一个了不起的 PR。每当有人突然为开源项目贡献出伟大的东西时，我都感到无比欣慰。

因此，从刚刚发布的[`0.17 release`](https://github.com/indygreg/PyOxidizer/releases/tag/apple-codesign%2F0.17.0)开始，`apple-codesign` Rust crate 及其对应的 `rcodesign` CLI 工具，您现在可以使用纯 Rust 客户端执行 `rcodesign notary-submit` 与 Apple 的 Notary API 通信。不再需要第三方专有软件。您只需要一个自包含的 `rcodesign` 可执行文件和一台运行 Linux、Windows、macOS、BSD 等操作系统的计算机即可对苹果应用程序进行签名和公证。

我非常激动地宣布终于实现了这一里程碑！可能有成千上万家公司和个人希望从非 macOS 操作系统发布苹果软件。（像[fastlane](https://fastlane.tools/)这样的工具的存在似乎证实了这一点。）长期以来，缺乏一个在 macOS 之外工作的苹果代码签名和公证解决方案一直阻碍着这一进程。现在，这个障碍正式消除了。

发行说明、文档和（自签名的）主要平台上的`rcodesign`可执行文件的预构建版本可以在[0.17 release page](https://github.com/indygreg/PyOxidizer/releases/tag/apple-codesign%2F0.17.0)上找到。
