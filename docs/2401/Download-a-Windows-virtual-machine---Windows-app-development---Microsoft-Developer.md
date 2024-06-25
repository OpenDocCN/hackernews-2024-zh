<!--yml

类别：未分类

日期：2024-05-27 14:48:29

-->

# 下载 Windows 虚拟机 - Windows 应用程序开发 | 微软开发人员

> 来源：[`developer.microsoft.com/en-us/windows/downloads/virtual-machines/`](https://developer.microsoft.com/en-us/windows/downloads/virtual-machines/)

# 获取 Windows 11 开发环境

通过使用预装有最新版本的 Windows、开发工具、SDK 和示例的虚拟机，快速构建 Windows 应用程序。

## 下载虚拟机

我们目前为四种不同的虚拟化软件选项打包我们的虚拟机：[VMWare](https://www.vmware.com/products/desktop-virtualization.html)、[Hyper-V（Gen2）](https://learn.microsoft.com/virtualization/hyper-v-on-windows/about/)、[VirtualBox](https://www.virtualbox.org/)，和 [Parallels](https://www.parallels.com/)。 这些虚拟机包含一个 [Windows 评估版本](https://www.microsoft.com/evalcenter) ，在发布日期到期后会失效。如果评估期到期，桌面背景将变黑，您将看到一个持久的桌面通知，指示系统不是正版，并且计算机将每小时关闭一次。

到期日期：2024 年 7 月 15 日

### 评估虚拟机包括：

+   Windows 11 企业版（评估版）

+   Visual Studio 2022 社区版启用 UWP、.NET 桌面、Azure 和用于 C# 工作负载的 Windows 应用程序 SDK

+   启用带有安装的 Ubuntu 的 Windows 子系统为 Linux 2

+   安装了 Windows 终端

+   启用开发者模式

### 文件哈希

| 名称 | 长度（字节） | [文件哈希-SHA256](https://learn.microsoft.com/powershell/module/microsoft.powershell.utility/get-filehash) |
| --- | --- | --- |
| WinDev2404Eval.HyperV.zip | 22947676843 | 42FF4A7A8D3D0AD8B739705830F01AA06E5ADE56A05553677B3F9C6A255BFFCA |
| WinDev2404Eval.Parallels.zip | 21907827044 | 5FE7E9054F83D0BB7129F78CE9E0E9770328EBC355E581E53E08618693F157BE |
| WinDev2404Eval.VirtualBox.zip | 23201160423 | D1270E7DDFB7B343D81BC29D55972CD8CA00417C863FFD9FD3ADDE75BA829CF2 |
| WinDev2404Eval.VMWare.zip | 24822900753 | 174B1A7862BB37E4E3F923BEC786A3560773D89ECA2AD2AA7ACB9DD3E4D6F659 |

注意

使用这些虚拟机，即表示您接受上述所有已安装产品的 [最终用户许可协议](https://aka.ms/windowsdevelopervirtualmachineeula)

## 常见问题

#### 这些虚拟机的要求是什么？

虚拟机将需要最少 8GB 的 RAM 和至少 70GB 的磁盘空间。

#### 这些虚拟机的用户密码是什么？

用户帐户没有设置密码。不过，一些软件，尤其是用于远程连接到虚拟机的软件，可能需要密码。在这种情况下，您需要首先为用户帐户设置密码，然后再使用该软件。

#### 这些映像是否可以激活 Windows 许可证并长期使用？

不，这些 VM 映像使用 Windows 企业评估版，并不支持使用产品密钥进行激活。

#### 是否有可能获得虚拟机的 ARM 版本？

很遗憾，目前我们没有 ARM 版本可用。我们理解这可能是令人失望的消息，但我们没有短期计划去创建这些。然而，我们一直乐于接受用户的反馈和建议，并将在计划未来的更新时予以考虑。

#### 我在 Virtualbox 上使用这些 VM 时看到了一些奇怪的渲染问题。已知问题？

是的，我们注意到在使用 VirtualBox 运行这些开发者映像时存在一些渲染怪异的问题。开始菜单可能看起来也与预期不同。我们目前正在调查这种行为。与此同时，我们感谢您的耐心和理解。

#### 我已经下载并在评估时间窗口内启动了 VM，但该 VM 声称评估许可证已经过期。求助？

假设该映像在评估时间窗口内运行，Windows 应该最终会自行修复。要立即修复，请运行[Windows 激活故障排除程序](https://support.microsoft.com/en-us/windows/using-the-windows-activation-troubleshooter-d717cdff-cf19-9770-7198-40119c2a696c)。

是的，您可以在这里获取评估映像：

[`www.microsoft.com/evalcenter/evaluate-windows-10-enterprise`](https://www.microsoft.com/evalcenter/evaluate-windows-10-enterprise)

[`www.microsoft.com/evalcenter/evaluate-windows-11-enterprise`](https://www.microsoft.com/evalcenter/evaluate-windows-11-enterprise)

您的反馈可以帮助我们打造出色的产品。请将您的反馈发送至 WinDevVMFeedback@microsoft.com
