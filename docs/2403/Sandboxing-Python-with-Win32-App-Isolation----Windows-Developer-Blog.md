<!--yml

category: 未分类

date: 2024-05-27 14:55:46

-->

# 使用 Win32 App Isolation 隔离 Python - Windows Developer Blog

> 来源：[https://blogs.windows.com/windowsdeveloper/2024/03/06/sandboxing-python-with-win32-app-isolation/](https://blogs.windows.com/windowsdeveloper/2024/03/06/sandboxing-python-with-win32-app-isolation/)

Python 的沙箱化对开发人员非常有用，但由于 CPython 实现的灵活性，这一直是个挑战。在许多情况下，对 Python 进行沙箱化尤其有用。例如，在需要从用户执行任意代码的任何网站上，如编程练习站点或在线解释器上。它还可以有助于防止对大型语言模型（LLMs）的不断增加的攻击。

Win32 应用隔离（https://github.com/microsoft/win32-app-isolation）提供了一种不同的途径来对 Python 进行沙盒隔离 - 不是在 Python 库层面上，而是在应用程序/操作系统层面上。

Win32 应用隔离旨在通过创建应用程序和操作系统之间的安全边界来隔离 Win32 应用程序，使用 AppContainer、虚拟化资源和文件系统代理的组合，有助于防止应用程序威胁操作系统。

以下是最新的 Windows Insider 版本上使用 Win32 应用隔离对 CPython 进行沙箱隔离的具体要求和指南。这需要大约 30-60 分钟，任何具有 Windows 内部版本的用户都可以在本地机器上完成这个过程。

要使用 Win32 应用隔离，您需要 Windows 内部版本号大于 25357，并且需要从 https://github.com/microsoft/win32-app-isolation/releases/ 获取 MSIX 打包工具。

Step One:

从 python.org 下载 Windows 版本的 Python 安装程序。此示例使用了版本 3.12.2。

Step Two:

Package it by following the documentation ([https://github.com/microsoft/win32-app-isolation/blob/main/docs/packaging/msix-packaging-tool.md](https://github.com/microsoft/win32-app-isolation/blob/main/docs/packaging/msix-packaging-tool.md)). You may need a certificate to sign the package to install it (https://learn.microsoft.com/en-us/windows/msix/packaging-tool/create-app-package#signing-preference).

Add `isolatedWin32-PromptForAccess` capability for demonstration purposes, so you can allow it to access certain resources in prompts. Without the capability, all access to the user files that are not explicitly set to be available to the application would be denied.

Add an alias `python3.12.exe` to use it from PowerShell.

Step Three:

安装该包。在 PowerShell 中使用定义的别名 `python3.12.exe` 启动 Python 解释器。

启动 Python 解释器时，它将提示您请求访问当前工作目录的权限。Python 本身可以通过任一选项启动 - 但这决定了 Python 是否可以访问当前目录中的文件。

例如，如果您在这里点击了“否”，您将无法再列出当前目录中的文件：

选择将被记住，所以应用程序只会对任何文件/文件夹提示一次。用户可以在“设置”-“隐私与安全性”-“文件系统”中重置文件权限：

如果没有更改代码，Python解释器应该可以自行运行良好。

检查的更有趣的方面是“沙盒”功能-网络和文件系统。

但在此之前，请尝试使用隔离的Python执行脚本。

因为你正在尝试访问和执行一个用户文件（脚本），它将提示访问权限。如果被拒绝，脚本将不会运行：

允许它回顾一些示例，从网络访问开始。

因为在打包时没有提供任何网络功能，所以Python将无法访问网络。通过运行此脚本来确认：

```
def test_urllib(): 
    with urllib.request.urlopen("http://www.microsoft.com") as response: 
        print(response.read())
```

同样尝试访问文件系统。

```
def test_open(): 
    with open("../data/test.txt", "w") as f: 
        f.write("test")
```

显示了一个提示来请求写入文件的访问权限，你可以通过点击“否”来拒绝这个访问。

它也不会与本机API一起工作：

```
def test_native(): 
    handle = ctypes.windll.kernel32.CreateFileW("../data/test.txt", 0x10000000, 0, None, 1, 0x80, 0) 
    if handle > 0: 
        ctypes.windll.kernel32.CloseHandle(handle) 
    else: 
        raise RuntimeError("Failed to get the handle")
```

你也可以测试`os.system()`和`subprocess`：

```
def test_os_system(): 
    os.system("powershell -command cat c:/data/test/bin/data/test.txt") 
def test_subprocess(): 
    subprocess.run(["powershell", "-command", "cat", "c:/data/test/bin/data/test.txt"], shell=True)
```

通过修改`sys.path`导入模块也会触发提示：

```
def test_import(): 
    sys.path.append("../data") 
    import example
```

点击“否”将拒绝对模块的访问（`example.py`在数据目录中，点击“是”将成功导入它）：

从上述示例中，通过引入Win32应用程序隔离，你可以帮助防止Python在未经同意的情况下访问文件系统和网络。

有两个明显的问题：

1.  如果你想在服务器上部署这个功能，你如何“点击”提示？如上所述，`isolatedWin32-promptForAccess`仅用于演示目的，不需要用于隔离。如果没有它，你的系统将简单地拒绝对用户文件的任何访问，从而有助于防止勒索软件等攻击。

1.  那么你如何为应用程序授予对模块、脚本和数据的必需文件访问权限？有几种方法可以做到：

    +   你可以将需要的模块打包到你的应用程序中，然后它们将默认可访问。

    +   你可以将需要访问的文件放入应用程序配置文件所在的`%localappdata%`，应用程序将自动获得访问权限。

    +   你可以将需要访问的文件放入`c:\ProgramData`中以你的应用程序发布者ID结尾的“发布者目录”中。([win32-app-isolation/docs/fundamentals/consent.md at main · microsoft/win32-app-isolation (github.com)](https://github.com/microsoft/win32-app-isolation/blob/main/docs/fundamentals/consent.md))

    +   你可以使用`icalcs`设置文件和目录的访问权限，允许你的应用程序或所有应用程序访问它们。

使用这项技术，用户可以在尝试执行来自不受信任来源的代码时创建更轻量的解决方案。隔离的Python可以与常规Python并行运行，并作为低特权解释器运行。或者，在正确配置的情况下，它可以单独使用，有助于保护系统免受威胁，同时保留Python的所有功能。
