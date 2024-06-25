<!--yml

category: 未分类

date: 2024-05-27 14:38:33

-->

# Picking the Widevine Locks: Acquiring and Using an L3 CDM | Mo Ismailzai

> 来源：[https://www.ismailzai.com/blog/picking-the-widevine-locks](https://www.ismailzai.com/blog/picking-the-widevine-locks)

数字版权管理（DRM）的世界是一个故意模糊的领域，部分依赖于混淆视听的安全策略。这对于负责提供付费媒体的开发者是一个挑战，特别是因为很多文档只通过特定供应商的企业门户交付。尽管如此，DRM和安全内容传递对现代互联网至关重要，是值得理解的重要内容。我最近不得不深入研究了[Widevine](https://www.widevine.com/solutions/widevine-drm)，这可能是最流行的DRM平台，因为它是Chrome和Android设备的本地支持，但基本原理在主要供应商（包括Microsoft的[PlayReady](https://www.microsoft.com/playready/)和Apple的[Fairplay](https://developer.apple.com/streaming/fps/)）中基本相同。

数字版权管理（DRM）是管理数字内容合法访问的技术，比管理物理介质的访问更为棘手。数字文件的一个基本特性是可以完美复制，使得复制品与原件无法区分。尽管如此，数字世界的转型极具吸引力，因为它的成本显著低于生产、存储、分发和销售物理介质。

至于Widevine，整个流程相对简单。要分发的媒体以加密格式存储，如果没有有效密钥，将无法访问。当您登录像Netflix这样的流媒体服务并尝试播放视频时，您的浏览器会向内容许可证服务器请求许可证。许可证服务器评估此请求并检查您的设备是否授权并满足内容的安全要求。Widevine根据设备的安全级别将其分为三个级别：L1、L2和L3，基于加密操作和密钥处理的安全程度不同。大多数网络浏览器和一些设备只支持L3，这意味着所有加密操作都是在软件而非硬件中执行的。L3被认为是最不安全的级别，因为内容和密钥在可能更易受攻击的软件环境中处理，因此通常对于可以在L3设备上播放的内容会有一些限制，例如不允许1080p或4K播放。

如果您的请求通过了许可检查，则解密密钥将发送到您的Widevine DRM客户端，该客户端开始解密过程。解密由您设备上的Widevine客户端实时进行，使用从许可证服务器获取的密钥。媒体播放器渲染解密后的内容以供播放。在L3设备上，内容在渲染和输出过程中没有得到像L1设备那样的安全保护，使其比L1设备更容易受到盗版。在播放过程中，Widevine客户端可能会持续确保许可证的有效性和遵守使用规则。这可以包括维护安全环境并验证内容是否被非法录制或传输等检查。

在本文中，我将演示一个Widevine工作流程示例，包括如何从模拟的Android设备中获取L3内容解密模块。不幸的是，这是一个简单的过程，突出显示了通过深奥的安全确实没有任何安全性。在我深入之前，先简要提醒一下：这篇文章全是关于DRM的细枝末节，并旨在引发关于创建*无法被盗版的用户体验*的更广泛讨论。***我不是在支持或建议您绕过DRM保护或法律***。这些东西在法律上可能会让您陷入麻烦，这篇文章旨在作为开发者资源，深入了解DRM的复杂世界。始终保持冷静和合法。

## 从Android设备获取L3 CDM

### 设置环境

第一步是建立必要的环境。我在Linux上工作，运行Manjaro，所以这些是我将包含的说明，但总体过程对于任何操作系统都是相同的。我们需要安装[Android Studio](https://developer.android.com/studio)，它可以从大多数软件包管理器或网页下载免费获得。我们还需要`android-tools`和`xz`软件包来与模拟的Android设备进行交互，并解压[Frida Server](https://frida.re/)存档。

```
yay -Syu android-studio android-tools xz
```

一旦成功安装了Android Studio，启动它，创建一个新项目，然后通过`Tools > Device Manager`导航来设置一个新的Android设备模拟器。我选择`Pixel 7 Pro`，API级别为`28`，目标为`Android 9（Google APIs）`，因为这是一个已知的可以生成L3 CDMs的工作配置。IDE应该开始下载必要的依赖项；一旦完成，启动模拟的Android设备，并确保通过终端中的`adb devices`命令被系统识别。

### 使用Frida Server工作

Frida 是安全研究人员和反向工程师使用的动态仪器工具包，他们需要钩入黑盒未记录功能。它允许您监视应用程序流量，例如，在加密并发送到网络之前，因此在 WireShark 无法满足的地方可能会有所帮助。您可以直接从他们的 [GitHub 发布](https://github.com/frida/frida/releases/) 页面安装 Frida Server。您选择的版本必须与下一步下载的 Widevine `wvdumper` 编排脚本的版本对应，因此如果偏离我的示例，请务必更新两者：

```
wget https://github.com/frida/frida/releases/download/16.0.8/frida-server-16.0.8-android-x86_64.xz
unxz frida-server-16.0.8-android-x86_64.xz 
```

下载并解压缩发布版后，您需要将文件推送到模拟的 Android 设备：

```
 adb push frida-server-16.0.8-android-x86_64 /sdcard
```

最后，您需要进入模拟设备的 shell，提升权限，将 Frida Server 移动到合适的目录，并确保它具有执行权限，然后运行它以开始监听仪器命令。

```
adb push frida-server-16.0.8-android-x86_64 /sdcard
adb shell
su
mv /sdcard/frida-server-16.0.8-android-x86_64 /data/local/tmp
chmod  +x /data/local/tmp/frida-server-16.0.8-android-x86_64
/data/local/tmp/frida-server-16.0.8-android-x86_64
```

### 使用 Widevine 密钥转储程序进行密钥转储

由于 Frida Server 现在正在运行，我们需要设置我们的 Python 编排脚本与其进行交互（显然，您需要保持该终端打开并保持进程运行，因此打开一个新终端）。

首先创建一个专用的项目目录。在此目录中，设置一个 Python 虚拟环境并安装所有必要的依赖项：

```
using python -m venv ./.
bin/pip3 install frida==16.0.8 frida-tools==12.0.4 protobuf==3.19.0 pycryptodome
```

这些版本很重要，如果您想使用 Frida Server 的较新版本，则可能需要尝试依赖项，除非您编写自己的编排脚本。

环境现在设置好了，克隆 Widevine `wvdumper/dumper` 仓库，然后运行 `dump_keys.py` 脚本：

```
git clone https://github.com/wvdumper/dumper l3keydump
cd l3keydump
../bin/python3 dump_keys.py
```

忽略所有助手，这个脚本实际上归结为这些行：

```
device = frida.get_usb_device()
scanner = Scan(device.name)
for process in device.enumerate_processes():
    logging.debug(process)
    if 'drm' in process.name:
        libraries = scanner.find_widevine_process(device, process.name)
        if libraries:
            for library in libraries:
                scanner.hook_to_process(device, process.name, library) 
```

它钩入 Frida Server 可以识别的任何 Widevine DRM 进程，并开始扫描密钥。 现在您只需在模拟的 Android 设备上启动 Google Chrome 浏览器，并导航到触发 Widevine DRM 工作流程的网站，例如，[https://bitmovin.com/demos/drm](https://bitmovin.com/demos/drm)。

如果一切配置正确，这将触发脚本创建一个新的 `./key_dumps` 目录并转储您的 CDM 客户端 ID 和私钥，分别为 `client_id.bin` 和 `client_id.pem`。 就所有目的而言，您现在可以作为合法的 L3 CDM，有效地绕过 Widevine 安全模型。

## 使用 L3 CDM 解密 Widevine 加密内容

使用 CDM 请求并解密来自流媒体服务的内容取决于该特定服务如何实现其 Widevine 工作流程。 Widevine 要求客户端向许可证服务器识别自己并请求特定内容的许可证。一旦识别和授权，许可证服务器将返回相应的解密密钥。除此之外的一切完全取决于实施网页开发人员，并且在各个服务之间可能会有很大差异。

尽管如此，通过HTTP传输信息的方式只有少数几种，无论是GET参数、POST负载还是HTTP标头，因此，通过一些试验和错误，一个积极主动的客户端可以快速识别所需内容，并在浏览器之外模拟相同的请求，从而绕过任何内置的浏览器安全机制。为了证明这一点，我们可以使用另一个Python脚本。在您上面创建的同一目录中，克隆解密代码库并安装相关依赖项：

```
git clone https://github.com/medvm/widevine_keys.git l3decrypt
bin/pip3 install pycryptodomex xmltodict requests google google-api-python-client
```

[Widevine密钥存储库](https://github.com/medvm/widevine_keys)粗糙而陈旧，包含的L3 CDM早已被列入黑名单，因此，为了进行此演示，您需要将代码库中的CDM替换为我们从上述模拟的安卓设备中获取的CDM：

```
cp l3keydump/key_dumps/*/private_keys/*/*/client_id.bin l3decrypt/cdm/devices/device_client_id_blob
cp l3keydump/key_dumps/*/private_keys/*/*/private_key.pem l3decrypt/cdm/devices/device_private_key
```

现在我们需要获取我们的加密内容及其解密密钥。导航到您想观看的Widevine保护内容并识别相关的Media Presentation Description（MPD）和许可证服务器网络请求。

MPD是描述媒体呈现结构的清单文件，包括有关可用比特率、分辨率和字幕的详细信息。这告诉我们可以获取哪些内容以及在其加密形式中从哪里下载它。毫不奇怪，流媒体平台希望您难以做到这一点，但最终，使用工具如[mitmproxy](https://github.com/mitmproxy/mitmproxy)、[zaproxy](https://github.com/zaproxy/zaproxy)、[httptoolkit](https://github.com/httptoolkit/httptoolkit)甚至是您的浏览器的基本网络请求标签，都是轻而易举的。在最简单的情况下，MPD和许可证请求都是对端点的简单GET调用。在更复杂的情况下，它们已知需要特定的标头、负载或HTTP动词，但归根结底，这些都是从您的浏览器发出的网络调用，因此完全透明，因此通过一些试验和错误，您总是可以识别出适当的请求。

对于简单的GET调用，您只需要复制相关的URL。对于需要特定标头的请求，您需要修改此存储库中的`headers.py`文件，以包含运行`l3.py`脚本所需的标头，然后运行该脚本。

一旦您确定了必要的组件，您可以使用`l3.py`从Widevine许可服务器获取解密密钥：

```
 cd l3decrypt
../bin/python3 l3.py 
```

您将被要求输入MPD和许可证URL，如果您一切都做对了，您将收到一组解密密钥：

```
PSSH obtained. 

TG9yZW0gaXBzdW0gZG9sb3Igc2l0IGFtZXQsIGNvbnNlY3RldHVyIGFkaXBpc2NpbmcgZWxpdCwgc2VkIGRvIGVpdXNtb2QgdGVtcG9yIGluY2lkaWR1bnQgdXQgbGFib3JlIGV0IGRvbG9yZSBtYWduYSAg

license response status: Response [200];

KID:KEY 47b53cd2729d42eb92c2ec9f24a0375f:1d58376bbb424d75a16f9ab20cc7bba8
```

保护系统特定标头（PSSH）是用于加密媒体流的数据块。它包含DRM系统解密内容所需的信息，例如密钥系统、密钥ID和其他DRM特定数据。在此之下，您将看到相关的密钥ID和相应的密钥。

### 下载和解密内容

拿到密钥后，我们现在可以下载加密内容并解密它。我们可以手动查看 MPD 文件并识别我们感兴趣的媒体文件，但使用像 [yt-dlp](https://github.com/yt-dlp/yt-dlp) 这样支持 MPD URL 的工具会更容易。我们还需要 [bento4](https://www.bento4.com/) 工具来解密我们下载的媒体和 [ffmpeg](https://ffmpeg.org/) 来将我们解密的音频和视频合并成一个文件。

首先，我们需要安装这些依赖项：

```
yay -Syu yt-dlp ffmpeg bento4
```

现在使用 `yt-dlp` 下载加密媒体：

```
yt-dlp --allow-unplayable-formats https://www.example.com/some/media.mpd
```

默认情况下，这将获取列出的最高分辨率视频和音频，但您当然可以使用标志修改。在此示例中，我们受到 L3 CDM 可用的比特率的限制，但真正的硬件 Android 设备可以被强制放弃 L1 密钥，从而允许访问最高质量的内容。要解密这些文件，我们将使用`mp4decrypt`（作为`bento4`软件包的一部分安装），并在上一步中由`l3.py`标识的每个密钥重复使用`--key`标志：

```
mp4decrypt --key KID:KEY encrypted_video.mp4 decrypted_video.mp4
mp4decrypt --key KID:KEY encrypted_audio.m4a decrypted_audio.m4a
```

此时，我们有了解密的音频和视频流，可以像这样捆绑在一起：

```
ffmpeg -i decrypted_video.mp4 -i decrypted_audio.m4a -vcodec copy -acodec copy decrypted_bundle.mp4
```

## 结语：

忽略显而易见的问题，比如模拟的 Android 设备为什么会生成真实的 Widevine 密钥，我的主要问题是 DRM 的目标是谁？显然，对于不是开发人员的人来说，Widevine 造成了重大障碍，但它为合法付费客户引入了限制。例如，内置到浏览器中的 L3 CDM 受设计限制 —— 这是可以理解的，因为在您的计算机上运行的软件可以被操纵以泄露其 CMD —— 但对于付费客户来说，这意味着他们无法在计算机上流式传输 4K 或 HDR 内容。具有讽刺意味的是，盗版用户不会遇到这个问题，因为他们的内容将是无 DRM 的。

在 PC 游戏世界中，DRM 也是如此，在其中出版商推出了具有妨碍性的 DRM 系统，这些系统经常无法正常工作，导致崩溃，需要互联网连接，或者以其他方式干扰软件的合法拥有者。倾向于这样做的用户只需下载无 DRM 的盗版版本，就能比付费客户有更好的整体用户体验，这显然是一个次优结果。作为开发者，我们必须优先考虑用户体验，并更多地思考激励而不是惩罚。

有明显的成功案例可以借鉴。例如，iTunes 能够与 Napster 和 Limewire 这样的盗版竞争，因为数字媒体本身，音乐，只是整体产品的一部分。iTunes 与其他苹果产品进行了深度集成，对终端用户来说更容易和更安全。它增加了价值而不是减少价值，并且这种价值非常引人入胜，以至于改变了整个音乐产业。同样的产业再次被流媒体服务颠覆，这些服务成为了无限音乐的门户，一种发现我们从未想过会喜欢的歌曲的方式，以及完全符合我们口味的播放列表的来源。

在PC游戏世界中，Steam从任何想玩《半条命2》的人必备的安装程序发展为一个深受欢迎的生态系统，游戏生死攸关。Steam通过提供及时更新和补丁、跨计算机和设备同步游戏存档、在网络上流式传输到其他设备的能力、社区评价和策划，甚至内置的支持mods，显著增强了用户体验。这改变了最终用户的计算方式，他们*可能*会盗版游戏，但无法盗版体验，因此决定这个价格是值得的。

最终，那些想要盗版的人会这样做。如果他们简单地买不起你在卖的产品，那么你几乎没有什么办法让他们成为付费客户。但对于那些没有看到明确价值主张的人群，我们需要创造出值得付费的用户体验。
