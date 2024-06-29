<!--yml

category: 未分类

date: 2024-05-27 14:48:08

-->

# 在Macbook Apple Silicon上构建DOS、OS/2

> 来源：[https://retrocoding.net/building-for-dos-os2-and-dos-on-a-macbook-apple-silicon](https://retrocoding.net/building-for-dos-os2-and-dos-on-a-macbook-apple-silicon)

我之前写过关于[OpenWatcom作为通往古老世界的入口](https://retrocoding.net/openwatcom-gateway-to-ancient-world-of-x86)。然而，那时，我专注于使用Docker安装它，因为OpenWatcom 1.9只在Windows和Linux上工作。我不是全职的Windows或Linux用户，所以当我发现OpenWatcom的2.0系列可以在macOS上编译和运行，并且可以在ARM macOS上运行时，我感到非常激动。

它没有出现在他们的网站上，只出现在像GitHub Issues这样不那么显眼的地方。然而，经过测试，我相信值得与其他像我一样热衷于写老式Windows（和DOS）应用程序但仍然想使用他们最新、最闪亮的MacBook的人分享。

## 准备工作

在在Apple Silicon上构建OpenWatcom源代码之前，你需要做一些准备工作。

**首先，编译器和平台SDK**。要在macOS上构建任何东西，你需要他们的官方编译器和IDE。Xcode。最近，他们试图精简一切，所以从[developer.apple.com](http://developer.apple.com)下载大约2GB的内容。

**其次，homebrew**。如果你知道你在做什么，这是可选的。然而，为了顺利编译OpenWatcom，你需要DOSBOX命令行，我觉得获取DosBox最简单的方法是使用homebrew。从[brew.sh](https://brew.sh)安装它。

## 获取依赖项

要获取依赖项，首先我们安装DosBox。在终端中，使用以下命令安装dosbox。

```
brew install dosbox 
```

这是你需要的唯一依赖项。

## 编译

编译实际上非常简单。你只需要获取“latest” OpenWatcom v2.0系列的源代码。我正在使用2024年2月的存档。转到发布页面并下载源代码（.tar.gz）。解压它

```
tar xvzf open-watcom-v2-2024-02-02-Build.tar.gz 
```

在我们做所有事情之前，现在打开文件 `setenv.h` 并搜索带有 `export OWTOOLS` 的行并更改为：

```
export OWTOOLS=CLANG 
```

这将改变编译为clang，Xcode内置的编译器，然后搜索带有 `export OWDOSBOX` 的行，取消注释并更改为。

```
export OWDOSBOX=dosbox 
```

从这一点上，你已经准备好了。最后一步只是运行这个命令

```
./build.sh 
```

等待。这将使用你的ARM64平台上的 `clang` 编译器编译OpenWatcom编译器。如果一切都完成了，你可以通过以下命令将编译后的产品放置。

```
./build.sh rel 
```

这将复制生成的编译器、支持所有支持平台的头文件以及库到 `rel` 目录中。

我通常在一个我放置在路径上的目录上使用watcom。对我来说，它在 `/opt/watcom` 上。所以如果你还没有做过，你可以运行这个命令来为你的watcom安装创建一个目录，并把所有东西复制到那个地方。

```
mkdir -p /opt/watcom
cd rel && copy -a . /opt/watcom 
```

要使用二进制文件，你需要将工具链添加到 `PATH` 中。Apple Silicon Mac 的工具链位于 `armo64` 子目录中。你还需要设置 `WATCOM` 环境变量，指向我们的安装目录。

```
export WATCOM=/opt/watcom
export PATH=$PATH:$WATCOM/armo64
export EDDAT=$WATCOM/eddat 
```

对于每个平台（16位 Windows、32位 Windows、DOS、OS2），你需要设置不同的环境变量来配置 `INCLUDE` 和 `LIBS`。这里是针对 32位 Windows 平台的设置。

```
export INCLUDE=$WATCOM/h:$WATCOM/h/nt:$WATCOM/h/nt/ddk:$WATCOM/h/nt/sdk:$WATCOM/h/nt/sdk/ntddk:$WATCOM/h/nt/sdk/wdm:$WATCOM/h/nt/sdk/wdm/dd
export LIBS=$WATCOM/lib386:$WATCOM/lib386/nt:$WATCOM/lib386/nt/ddk:$WATCOM/lib386/nt/sdk:$WATCOM/lib386/nt/sdk/ntddk:$WATCOM/lib386/nt/sdk/wdm:$WATCOM/lib386/nt/sdk/wdm/ddk 
```

请参考 OpenWatcom/Watcom 手册，查找头文件和库文件的位置。你可以从 `/h` 目录和 `/libxxx` 目录基本猜出来。

完成后，你可以运行 `wcl386` 看看是否能正常工作。

```
❯ wcl386 -h
Open Watcom C/C++ x86 32-bit Compile and Link Utility
Version 2.0 beta Feb 10 2024 00:00:59
Copyright (c) 2002-2024 The Open Watcom Contributors. All Rights Reserved.
Portions Copyright (c) 1988-2002 Sybase, Inc. All Rights Reserved.
Source code is available under the Sybase Open Watcom Public License.
See https://github.com/open-watcom/open-watcom-v2 
```

这意味着你的 Watcom 编译器已经准备好用于旧平台的目标。
