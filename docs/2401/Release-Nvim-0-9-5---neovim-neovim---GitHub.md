<!--yml

分类：未分类

日期：2024-05-27 14:26:15

-->

# 发布 Nvim 0.9.5 · neovim/neovim · GitHub

> 来源：[`github.com/neovim/neovim/releases/tag/v0.9.5`](https://github.com/neovim/neovim/releases/tag/v0.9.5)

```
NVIM v0.9.5
Build type: Release
LuaJIT 2.1.1692716794 
```

这是一个维护版本，重点是修复了 bug。

特别是，修复了在使用和测试 Nvim 时出现的问题，这些问题在不常见的平台上，如大端平台上。

## 发布说明

## 安装

### Windows

#### Zip

1.  下载 **nvim-win64.zip**

1.  解压缩 zip

1.  运行 `nvim-qt.exe`

#### MSI

1.  下载 **nvim-win64.msi**

1.  运行 MSI

1.  搜索并运行 `nvim-qt.exe` 或者在你选择的 CLI 上运行 `nvim.exe`

### macOS

1.  下载 **nvim-macos.tar.gz**

1.  运行 `xattr -c ./nvim-macos.tar.gz`（以避免“未知开发者”警告）

1.  解压缩：`tar xzvf nvim-macos.tar.gz`

1.  运行 `./nvim-macos/bin/nvim`

### Linux（x64）

#### AppImage

1.  下载 **nvim.appimage**

1.  运行 `chmod u+x nvim.appimage && ./nvim.appimage`

    +   如果你的系统没有 FUSE，你可以 [解压缩 appimage](https://github.com/AppImage/AppImageKit/wiki/FUSE#type-2-appimage)：

        ```
        ./nvim.appimage --appimage-extract
        ./squashfs-root/usr/bin/nvim 
        ```

#### Tarball

1.  下载 **nvim-linux64.tar.gz**

1.  解压缩：`tar xzvf nvim-linux64.tar.gz`

1.  运行 `./nvim-linux64/bin/nvim`

### 其他

## SHA256 校验和

```
44ee395d9b5f8a14be8ec00d3b8ead34e18fe6461e40c9c8c50e6956d643b6ca  nvim-linux64.tar.gz
0c82e5702af7a11fbb916a11b4a82e98928abf8266c74b2030ea740340437bf9  nvim.appimage
e3f3174d75c038915330db86bf685c704cb9be86863ee592a07e21203d32ced2  nvim.appimage.zsync
19d2366e0d6da001583bd0b8a3db59f69ce3dda5fa41f3064c6778cef3edd34c  nvim-macos.tar.gz
de6dc1f0edb45f5f225ee24ce80a4fcbc3a337932037e98ae143975fca2556bf  nvim-win64.zip
006b8578f0b6717bc5a987f12bc0746c61c20e6ba777fde6d4aa53ee54b937cd  nvim-win64.msi 
```
