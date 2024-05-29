<!--yml
category: 未分类
date: 2024-05-27 14:26:15
-->

# Release Nvim 0.9.5 · neovim/neovim · GitHub

> 来源：[https://github.com/neovim/neovim/releases/tag/v0.9.5](https://github.com/neovim/neovim/releases/tag/v0.9.5)

```
NVIM v0.9.5
Build type: Release
LuaJIT 2.1.1692716794 
```

This is a maintenance release, focusing on bugfixes.
Notably, fixes were made for issues with using and testing Nvim on less common platforms, like big endian platforms.

## Release notes

## Install

### Windows

#### Zip

1.  Download **nvim-win64.zip**
2.  Extract the zip
3.  Run `nvim-qt.exe`

#### MSI

1.  Download **nvim-win64.msi**
2.  Run the MSI
3.  Search and run `nvim-qt.exe` or run `nvim.exe` on your CLI of choice

### macOS

1.  Download **nvim-macos.tar.gz**
2.  Run `xattr -c ./nvim-macos.tar.gz` (to avoid "unknown developer" warning)
3.  Extract: `tar xzvf nvim-macos.tar.gz`
4.  Run `./nvim-macos/bin/nvim`

### Linux (x64)

#### AppImage

1.  Download **nvim.appimage**
2.  Run `chmod u+x nvim.appimage && ./nvim.appimage`
    *   If your system does not have FUSE you can [extract the appimage](https://github.com/AppImage/AppImageKit/wiki/FUSE#type-2-appimage):

        ```
        ./nvim.appimage --appimage-extract
        ./squashfs-root/usr/bin/nvim 
        ```

#### Tarball

1.  Download **nvim-linux64.tar.gz**
2.  Extract: `tar xzvf nvim-linux64.tar.gz`
3.  Run `./nvim-linux64/bin/nvim`

### Other

## SHA256 Checksums

```
44ee395d9b5f8a14be8ec00d3b8ead34e18fe6461e40c9c8c50e6956d643b6ca  nvim-linux64.tar.gz
0c82e5702af7a11fbb916a11b4a82e98928abf8266c74b2030ea740340437bf9  nvim.appimage
e3f3174d75c038915330db86bf685c704cb9be86863ee592a07e21203d32ced2  nvim.appimage.zsync
19d2366e0d6da001583bd0b8a3db59f69ce3dda5fa41f3064c6778cef3edd34c  nvim-macos.tar.gz
de6dc1f0edb45f5f225ee24ce80a4fcbc3a337932037e98ae143975fca2556bf  nvim-win64.zip
006b8578f0b6717bc5a987f12bc0746c61c20e6ba777fde6d4aa53ee54b937cd  nvim-win64.msi 
```