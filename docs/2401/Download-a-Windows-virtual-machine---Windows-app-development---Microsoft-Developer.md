<!--yml
category: 未分类
date: 2024-05-27 14:48:29
-->

# Download a Windows virtual machine - Windows app development | Microsoft Developer

> 来源：[https://developer.microsoft.com/en-us/windows/downloads/virtual-machines/](https://developer.microsoft.com/en-us/windows/downloads/virtual-machines/)

# Get a Windows 11 development environment

Start building Windows applications quickly by using a virtual machine with the latest versions of Windows, the developer tools, SDKs, and samples ready to go.

## Download a virtual machine

We currently package our virtual machines for four different virtualization software options: [VMWare](https://www.vmware.com/products/desktop-virtualization.html), [Hyper-V (Gen2)](https://learn.microsoft.com/virtualization/hyper-v-on-windows/about/), [VirtualBox](https://www.virtualbox.org/), and [Parallels](https://www.parallels.com/). These virtual machines contain [an evaluation version of Windows](https://www.microsoft.com/evalcenter) that expires on the date posted. If the evaluation period expires, the desktop background will turn black, you will see a persistent desktop notification indicating that the system is not genuine, and the PC will shut down every hour.

Expiration date: July 15, 2024

### The evaluation virtual machine includes:

*   Windows 11 Enterprise (Evaluation)
*   Visual Studio 2022 Community Edition with UWP, .NET Desktop, Azure, and Windows App SDK for C# workloads enabled
*   Windows Subsystem for Linux 2 enabled with Ubuntu installed
*   Windows Terminal installed
*   Developer mode enabled

### File hashes

| Name | Length (bytes) | [File Hash - SHA256](https://learn.microsoft.com/powershell/module/microsoft.powershell.utility/get-filehash) |
| --- | --- | --- |
| WinDev2404Eval.HyperV.zip | 22947676843 | 42FF4A7A8D3D0AD8B739705830F01AA06E5ADE56A05553677B3F9C6A255BFFCA |
| WinDev2404Eval.Parallels.zip | 21907827044 | 5FE7E9054F83D0BB7129F78CE9E0E9770328EBC355E581E53E08618693F157BE |
| WinDev2404Eval.VirtualBox.zip | 23201160423 | D1270E7DDFB7B343D81BC29D55972CD8CA00417C863FFD9FD3ADDE75BA829CF2 |
| WinDev2404Eval.VMWare.zip | 24822900753 | 174B1A7862BB37E4E3F923BEC786A3560773D89ECA2AD2AA7ACB9DD3E4D6F659 |

Note

By using the virtual machines, you are accepting the [EULAs](https://aka.ms/windowsdevelopervirtualmachineeula) for all the installed products listed above

## Frequently Asked Questions

#### What are the requirements for the VM?

The VM will require a minimum of 8GB of RAM and at least 70GB of disk space.

#### What is the user password for these VMs?

There is no password set up for the user account. However, some software, especially those used to connect remotely to the VM, may require a password. In those cases, you will need to set up a password for the user account first before using that software.

#### Is it possible to activate the Windows license in these images for long term use?

No, these VM images use Windows Enterprise Evaluation Edition and do not support activation with a product key.

#### Is it possible to get an ARM version of the VM?

Unfortunately, we don't have an ARM version available at the moment. We understand that this may be disappointing news, but we don't have any short term plans to create these. However, we're always open to feedback and suggestions from our users and will take them into consideration when planning future updates.

#### I see strange rendering quirks when using these VMs on Virtualbox. Known issue?

Yes, we have noticed that there are some rendering quirks when using VirtualBox to run these developer images. The Start menu may also look different than expected. We are currently investigating this behavior. In the meantime, we appreciate your patience and understanding.

#### I've downloaded and booted the VM within the evaluation window, but the VM claims the evaluation license has already expired. Help?

Assuming the image is running within the evaluation window, Windows should ultimately fix itself. To fix it immediately, please run the [Windows Activation Troubleshooter.](https://support.microsoft.com/en-us/windows/using-the-windows-activation-troubleshooter-d717cdff-cf19-9770-7198-40119c2a696c)

Yes, you can get evaluation images here:

[https://www.microsoft.com/evalcenter/evaluate-windows-10-enterprise](https://www.microsoft.com/evalcenter/evaluate-windows-10-enterprise)
[https://www.microsoft.com/evalcenter/evaluate-windows-11-enterprise](https://www.microsoft.com/evalcenter/evaluate-windows-11-enterprise)

Your feedback can help us build great products. Please send your feedback to [WinDevVMFeedback@microsoft.com](mailto:WinDevVMFeedback@microsoft.com)