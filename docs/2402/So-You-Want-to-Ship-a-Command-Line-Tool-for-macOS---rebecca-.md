<!--yml

category: 未分类

date: 2024-05-27 15:04:02

-->

# 所以你想要在macOS上发布一个命令行工具 @ rebecca®

> 来源：[https://becca.ooo/blog/so-you-want-to-ship-a-command-line-tool-for-macos/](https://becca.ooo/blog/so-you-want-to-ship-a-command-line-tool-for-macos/)

<main id="main">

一个建议：**不要**。

在工作中，我编写了一个命令行工具，用于设置开发环境。它安装了Nix包管理器，设置了本地的Postgres实例，并处理了所有复杂的配置细节。在Linux和macOS上都可以运行，并支持`bash`、`zsh`和`fish`作为shell配置。

我们使用GitHub actions构建和发布发布版本，这样工程师们可以在需要设置新机器或修复现有机器的开发环境时下载并运行最新版本的工具。

在Linux上，这些都很顺利（除了在NixOS上运行任何东西的通常问题之外](https://wiki.nixos.org/wiki/Packaging/Binaries)，但macOS要求程序必须经过代码签名和公证。

在macOS上进行代码签名真的是一场*噩梦*。作为对非对称加密略有了解的人，我本来以为过程大致是这样的：

1.  生成公钥和私钥（这是一大堆二进制数据，或者大致来说是“一个文件”）。

1.  让Apple用我的公钥签署一个证书（这个证书也是一个文件）。

1.  运行一个程序，指向包含我的私钥和证书的文件，“签署”一个可执行文件以便分发。

1.  当用户运行我的程序时，操作系统可以看到它使用有效的苹果证书签名，无需担心任何问题。

事实上，并非如此。

第一个问题是，苹果提供了不少于八种不同类型的证书，但很少有文档说明它们的用途，所以在成功之前，预计要生成半打证书并撤销其中几个。

另一个问题是，`codesign`命令行工具有一个非常不透明和复杂的界面。实际上，要运行代码，有几个你必须选择的选项，并不是所有的选项都明确表明是必需的（例如，`-o runtime`和`-timestamp`对于公证是必需的）。

没有图形界面访问的情况下，比如在CI中签署代码，获取证书进入钥匙串的说明很少，而且苹果关于用Xcode之外的工具签署任何内容的文档也不多。

我成功解决了大部分签署代码的问题，使用了由Gregory Szorc编写的*出色的*[`rcodesign`工具](https://gregoryszorc.com/blog/2022/04/25/expanding-apple-ecosystem-access-with-open-source,-multi-platform-code-signing/)，它具有非常合理的命令行界面，实际上接受所有作为文件的密钥和证书。我将摘录一段来自链接博文的真实描述（强调是我自己的）：

> 我已经对苹果代码签名的细枝末节了解得太多了。**对于安全领域来说，机制过于复杂了。**在过去的一年中，至少有一个引起关注的Gatekeeper bug允许未正确签名的代码运行。我怀疑会有更多：可利用的面积实在太大了。

一旦我成功签名并验证了可执行文件，我立即遇到了另一个问题：[Gatekeeper阻止点击时运行命令行工具。](https://developer.apple.com/forums/thread/706379)显然这是macOS中的一个“已知bug”，尽管因为它被记录在苹果的专有“Radar”错误跟踪器中，我无法看到任何细节或者是否有修复计划。

这特别令人遗憾，因为当工程师在macOS上从GitHub下载二进制文件时，下载会从屏幕底部弹出，几乎让人无法抵挡点击。

解决方案来自于苹果开发者论坛的Quinn告诉我们，“将您的工具嵌入到一个应用程序中。”据我所知，这并不起作用，但让我们来看看。

有一个关于[在沙箱应用中嵌入命令行工具的长指南](https://developer.apple.com/documentation/xcode/embedding-a-helper-tool-in-a-sandboxed-app)，所以我遵循了那个指南，然后缓慢而痛苦地将Xcode剔除，这样我就不必想办法将一个10GB的Xcode安装到CI机器上（请记住，您需要登录到Apple ID才能下载Xcode，而且没有办法通过命令行进行）。

运用[Cassie的](https://www.witchoflight.com/)帮助，我编写了一个简短的Swift脚本（我希望它）能够实现我想要的功能：首先，它使用[`Bundle.url(forAuxiliaryExecutable:)`](https://developer.apple.com/documentation/foundation/bundle/1411412-url)方法查找嵌入的命令行工具二进制文件。然后，将该URL传递给[`NSWorkspace.open(urls:, withApplicationAt:)`](https://developer.apple.com/documentation/appkit/nsworkspace/3172702-open)方法，在新窗口中运行嵌入的命令行工具。最后，`completionHandler`在终端窗口打开后关闭应用程序。

（Swift中有[一个bug](https://github.com/apple/swift/issues/55127)，使得SwiftUI应用程序必须使用`@main`属性，否则无法工作，所以我需要自己运行`swiftc`，并添加神秘的`-parse-as-library`选项来修复这个bug。我还需要[阅读Swift源代码](https://stackoverflow.com/questions/46532610/swiftc-possible-values-for-target-command-line-option)来确定`-target`命令行选项的可能值。）

现在，我们必须进行代码签名：

1.  这是Swift的包装脚本。

1.  原始的命令行工具。

1.  包含上述两者的`.app`。

1.  包含`.app`的`.dmg`，因为`.app`只是一个目录，所以您需要将其压缩以分发。

我们还需要对 `.app` 和 `.dmg` 进行认证。有趣的是，你只能对 `.pkg`、`.dmg` 和 `.app` 文件（在 `.zip` 中）进行认证 —— 如果命令行工具嵌入其中，才能进行认证。

大多数认证文档告诉你使用 `altool`，但只有在你真的上传某些内容进行认证后，它才会告诉你它已经被 `notarytool` 取代。如果任何事情失败，`notarytool` 将打印 `status: Invalid`，而没有额外的细节。（有一个单独的 `notarytool` 子命令可以使用来获取更多信息的日志，但输出中没有任何东西告诉你这是一个选项，包括打开详细/调试日志。）

无论如何，一旦应用程序被组装并通过了所有苹果的验证工具，它，嗯，继续不工作！

你可以双击应用程序来运行它，但是当它尝试启动嵌入的命令行工具时，我们会收到一个错误消息：“无法打开 `mytool-cli` 因为无法确认开发者身份”，接着是“无法打开应用程序‘Terminal’。-128”。

对于这些文件，我确实在苹果的工具中得到一个错误；尽管 `.app` 本身通过了所有检查，但嵌入的工具未通过 `spctl` 的验证：

```
$ spctl -a -v --raw ./mytool
./mytool: rejected (the code is valid but does not seem to be an app)
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE plist PUBLIC "-//Apple//DTD PLIST 1.0//EN" "http://www.apple.com/DTDs/PropertyList-1.0.dtd">
<plist version="1.0">
<dict>
	<key>assessment:authority</key>
	<dict>
		<key>assessment:authority:flags</key>
		<integer>0</integer>
		<key>assessment:authority:source</key>
		<string>obsolete resource envelope</string>
		<key>assessment:authority:weak</key>
		<true/>
	</dict>
	<key>assessment:cserror</key>
	<integer>-67002</integer>
	<key>assessment:remote</key>
	<true/>
	<key>assessment:verdict</key>
	<false/>
</dict>
</plist> 
```

搜索“obsolete resource envelope”会得到几年来不同的错误，但都不适用。而且 `spctl -a -v --raw` 显示了与 macOS 预装的 `/bin/ls` 相同的错误。

这就是我们今天的情况。我已经在[Apple开发者论坛上报告了这个问题，](https://developer.apple.com/forums/thread/713932)但我不指望会得到任何可操作的建议，因为这个分发路径已经试图解决几个已知的 macOS bug。

所有这些工具及其所有错误信息都是*垃圾*。(`rcodesign` 是一个显著的例外，因为它不是由苹果编写的。) 以下是一些你可能运行的命令来检查代码签名：

```
spctl -a -v --raw mytool.app
codesign -verify -vvvv mytool.app
codesign -d -vvv --entitlements :- mytool.app
codesign -vvvv -R=notarized --check-notarization mytool.app 
```

只有一堆字母绝对的涌出来，没有明显的意义，当然也没有合理的直觉。

在苹果开发者论坛上，[MirrorMan 发帖谈论了大致相同的问题](https://developer.apple.com/forums/thread/696235)，并表达了他的沮丧：

> > 如果产品已签名、已认证和正确装订，一切应该正常运行。否则，你需要调查 Gatekeeper 为什么不满意（2），解决这个问题，然后重新测试。
> > 
> “修复那个”！！！！不！绝对不行！Gatekeeper/spctl 必须有办法准确描述是否会在任意客户机器上运行某个东西（应用程序、命令行工具…），立即、马上，从开发机器上绕过任何缓存或其他任何东西。为了一个苹果开发者花几天更新 spctl 以产生100%准确和有用的信息，有多少第三方开发者日需要浪费？例如，notarytool 的信息非常好！它告诉我接下来必须跳过的半打困难（钥匙串！应用程序特定密码！）。仅仅是做研究来弄清楚该做什么，就遇到了难以理解的错误。每年2370亿美元，开发者还得猜测？苹果可以做得更好。抱歉发牢骚，但这是必要的。

除了我对那些因为开发者不友好的行为而完全抛弃苹果生态系统的朋友们充满同情之外，我没有更多要补充的。

## 接下来

那么，我们可以用这些做些什么呢？我们有几个选择，但都不是特别吸引人。

1.  保持现状；告诉用户执行 `chmod +x` 和 `xattr -d com.apple.quarantine` 命令来跳过代码签名机制，但这需要用户复制/粘贴魔法命令。

    最终一切都还好 — 用户是工程师，所以他们可以管理终端，或者我们可以教他们 — 但这并不特别“干净”，尤其是当苹果（理论上）为此目的提供了代码签名机制，而我雇主已经为此付费。

1.  使用 `curl` 下载文件。像 `curl` 和 `tar` 这样的 Unix 命令行工具在 macOS 上创建的文件不会添加隔离位，因此使用它们下载的软件非常容易运行。

    不幸的是，开发此工具的仓库是私有的，因此未经身份验证的 `curl` 下载无法工作。我正在认真考虑是否可以让首席技术官将仓库公开，**只为了**能够使用 `curl` 作为分发机制。我甚至可以编写一个平台检测的 shell 脚本！

1.  重新用 Swift 实现整个工具，并将其作为使用 Xcode 构建的 App 分发，而不包含任何嵌入式工具。

    哈哈哈哈。开玩笑的。

1.  将该工具作为自定义 Homebrew tap 或类似的东西进行分发。

    这个工具负责*安装* Homebrew，并且我在一家 Nix 初创公司工作，因此许多工程师*拒绝*安装 Homebrew。我觉得这有点儿荒谬，但我的工作是支持他们，提供尽可能顺畅的开发体验，所以我们实际上不能使用 Homebrew 进行分发。

我想我可能会分发一个经过代码签名的可执行文件，尽管 macOS 会说它无法确定制造者是谁，即使他们自己的工具说签名没问题，晚上我也会哭着入睡。

## 结论

1.  不要试图在 macOS 上“创建软件”。他们不希望你这样做。如果你想要运行自己的程序，请安装 Linux，并像你应该的那样受苦。

1.  如果你需要为苹果电脑签署软件，请使用[`rcodesign`](https://gregoryszorc.com/blog/2022/04/25/expanding-apple-ecosystem-access-with-open-source,-multi-platform-code-signing/)。

</main>
