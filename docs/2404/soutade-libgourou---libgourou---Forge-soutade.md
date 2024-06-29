<!--yml

类别：未分类

date: 2024-05-27 13:05:07

-->

# soutade/libgourou - libgourou - Forge soutade

> 来源：[https://forge.soutade.fr/soutade/libgourou](https://forge.soutade.fr/soutade/libgourou)

## 介绍

libgourou 是 Adobe ADEPT 协议的免费实现，用于向 ePub/PDF 文件添加 DRM。它弥补了 Adobe 在 Linux 平台上的支持不足。

## 架构

类似于 RMSDK，libgourou 采用客户端/服务器模式。所有平台特定功能（加密、网络...）必须在客户端类中实现（派生自 DRMProcessorClient），而服务器则实现 ADEPT 协议。提供了使用 cURL、OpenSSL 和 libzip 的参考实现（在 *utils* 目录中）。

使用 gourou::DRMProcessor 的主要功能包括：

+   从一个 ACSM 文件获取 ePub：*fulfill()* 和 *download()*

+   创建新设备：*createDRMProcessor()*

+   注册新设备：*signIn()* 和 *activateDevice()*

+   去除 DRM：*removeDRM()*

+   归还借出的书籍：*returnLoan()*

你可以至少从以下位置导入配置：

+   Kobo 设备：.adept/device.xml、.adept/devicesalt 和 .adept/activation.xml

+   Bookeen 设备：.adobe-digital-editions/device.xml、root/devkey.bin 和 .adobe-digital-editions/activation.xml

或者创建一个新设备。注意：每个帐户最多可以创建有限数量的设备。

ePub 使用共享密钥加密：一个帐户 / 多个设备，因此您可以在计算机上创建并注册一个设备，并使用相同的 AdobeID 帐户配置您的电子阅读器来读取下载的（和加密的）ePub 文件。

对于想要使用 Calibre 和其 DeDRM 插件去除 DRM 的人，你可以导出你的私钥并将其导入到 [Calibre](https://calibre-ebook.com/) 中。

## 依赖项

对于 libgourou：

*externals*：

*internals*：

对于 utils：

+   libcurl

+   OpenSSL

+   libzip

+   libpugixml

内部库会在第一次运行时自动获取并静态编译。当你更新 libgourou 仓库时，**不要忘记更新内部库**，方法如下：

```
make update_lib 
```

## 编译

使用 *make* 命令

```
make [CROSS=XXX] [DEBUG=(0*|1)] [STATIC_UTILS=(0*|1)] [BUILD_UTILS=(0|1*)] [BUILD_STATIC=(0*|1)] [BUILD_SHARED=(0|1*)] [all*|clean|ultraclean|build_utils|install|uninstall] 
```

CROSS 可以定义一个交叉编译器前缀（如 arm-linux-gnueabihf-）

DEBUG 可以设置为调试模式编译

BUILD_UTILS 用于构建 utils 或不构建

STATIC_UTILS 用于使用静态库（libgourou.a）构建 utils，而不是默认的动态库（libgourou.so）

BUILD_STATIC 如果为 1，则构建 libgourou.a，为 0 则不构建，可以与 BUILD_SHARED 结合使用

BUILD_SHARED 如果为 1，则构建 libgourou.so，为 0 则不构建，可以与 BUILD_STATIC 结合使用

其他变量包括 DESTDIR 和 PREFIX，用于处理目标安装目录

## Utils

首先，将 libgourou.so 添加到你的 LD_LIBRARY_PATH 中

```
export LD_LIBRARY_PATH=$LD_LIBRARY_PATH:$PWD 
```

您可以选择性地指定您的 .adept 目录

```
export ADEPT_DIR=/home/XXX 
```

然后，使用以下工具：

你可以从你的电子阅读器导入配置或使用 *utils/adept_activate* 创建新的配置：

```
./utils/adept_activate -u <AdobeID USERNAME> 
```

然后会创建 */home//.config/adept* 目录及所有配置文件

下载 ePub/PDF：

```
./utils/acsmdownloader <ACSM_FILE> 
```

导出你的私钥（用于 DeDRM 软件）：

```
./utils/acsmdownloader --export-private-key [-o adobekey_1.der] 
```

去除 ADEPT DRM：

```
./utils/adept_remove <encryptedFile> 
```

列出借出的书籍：

```
./utils/adept_loan_mgt [-l] 
```

归还借出的书籍：

```
./utils/adept_loan_mgt -r <id> 
```

您可以通过 -h 或 --help 开关获取 utils 的完整选项说明

## Docker

一个由 bcliang 制作的 Docker 镜像可以在 [https://github.com/bcliang/docker-libgourou/](https://github.com/bcliang/docker-libgourou/) 找到

## 版权

Grégory Soutadé

## 许可证

libgourou：遵循 LGPL v3 或更高版本许可协议

utils：遵循 BSD 许可协议

## 特别鸣谢

+   *Jens* 贡献了所有的测试样本和实用工具测试

+   *Milian* 用于调试和编码

+   *Berwyn H* 贡献了所有的测试样本、反馈、补丁以及慷慨捐赠
