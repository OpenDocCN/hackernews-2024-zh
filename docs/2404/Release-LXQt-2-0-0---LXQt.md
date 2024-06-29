<!--yml

category: 未分类

date: 2024-05-27 13:16:49

-->

# 发布LXQt 2.0.0 | LXQt

> 来源：[https://lxqt-project.org/release/2024/04/15/release-lxqt-2-0-0/](https://lxqt-project.org/release/2024/04/15/release-lxqt-2-0-0/)

<main id="content">

<hgroup>

# 发布LXQt 2.0.0

</hgroup>

2024年4月15日星期一

<main>

LXQt团队宣布发布了轻量级Qt桌面环境LXQt 2.0.0。

## 亮点

### General

+   LXQt 2.0.0基于Qt ≥ 6.6。为了支持带有Qt5应用程序的样式和LXQt文件对话框，在LXQt 2.0.0中可以与其Qt6版本并行安装`libqtxdg-3.12.0`、`lxqt-qtplugin-1.4.1`和`libfm-qt-1.4.0`。

+   PCManFM-Qt的桌面模块，LXQt Runner和LXQt桌面通知现已完全适用于支持“层壳协议”的Wayland合成器，如LabWC，Wayfire，kwin_wayland，Hyprland，Sway等。

+   Wayland将是LXQt 2.1.0的主要目标，就像Qt6是LXQt 2.0.0的一样。尚未准备好支持Wayland的组件包括ScreenGrab、LXQt全局快捷键、LXQt面板的任务栏和键盘指示器（但LXQt面板在Wayland下也可以使用，不需要这些插件），部分输入设置以及监视器、电源按钮和屏幕锁定设置。不过，大多数Wayland合成器已经有了可以替代它们的工具，因此高级用户已经可以使用LXQt-Wayland会话。

+   LXQt面板有了一个名为Fancy Menu的新默认应用菜单，包括“收藏夹”、“所有应用程序”和改进的搜索功能。

+   仅QTerminal将单独发布其Qt6版本 - 由于Qt6移除了传统编码，因此遇到了一些问题。在此之前，可以使用其Qt5版本1.4.0。

**注意事项给发行版：** 为了在LXQt 2.0.0中使用带有Qt5样式和Qt5文件对话框，可能需要将基于Qt5的`lxqt-build-tools`、`libqtxdg`、`lxqt-qtplugin`和`libfm-qt`的可安装包重命名，以便与其Qt6版本并行安装。此外，正确的Qt5字体设置需要Qt5版本的`lxqt-qtplugin-1.4.1`点版本（参见其发布说明）。

### LibFM-Qt / PCManFM-Qt

+   感谢`layer-shell-qt6`，桌面模块已完全准备好用于Wayland。现在，PCManFM-Qt不仅可以与桌面不可知的X11窗口管理器一起提供真正的桌面，还可以与实现“层壳协议”的Wayland合成器（如基于`wlroots`或KWin的合成器）一起使用。此外，请参阅[在Wayland下使用桌面](https://github.com/lxqt/pcmanfm-qt/wiki#using-desktop-under-wayland)中的Wiki页面。

+   在LibFM-Qt中更新了LXQt Archiver和Arqiver的MIME类型。

+   添加了一些菜单图标。

### LXQt面板

+   Fancy Menu作为新的默认应用菜单添加进来了。（旧菜单保留，但不再是默认菜单。）

+   使用层壳通过Wayland支持定位面板。

+   杂项修复。

### LXQt Runner

+   全面添加了Wayland支持。

### LXQt桌面通知

+   全面添加了Wayland支持。

## 发布说明

请查看每个 LXQt 组件的发布页面以获取其发布说明。

</main>

</main>
