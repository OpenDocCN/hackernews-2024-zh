<!--yml

类别：未分类

日期：2024-05-29 12:43:16

-->

# Flatpak构建不具备复现性，这是一个实际问题

> 来源：[https://ranfdev.com/blog/flatpak-builds-are-not-reproducible/](https://ranfdev.com/blog/flatpak-builds-are-not-reproducible/)

<hgroup>

# Flatpak构建不具备复现性，这是一个实际问题

## 阅读时间6分钟。发布日期：2022-05-09

</hgroup>

<details class="toc"><summary>目录</summary></details>

## *更新 2022-05-10*

我刚刚与一个flatpak的贡献者交谈过，他给了我更多关于这个主题的信息。flatpak在*技术上* **是可以复现的！** 所有重现构建所需的元数据都存在于构建的包中 **内部**。

> 已经生成了一个锁定文件，并且是最终构建的一部分。您可以在/runtime下找到它们，位于/usr/manifest.json和/app/manifest.json，分别用于运行时和应用程序。

实际上，这些文件修复了运行时、sdk和其他依赖项的确切提交：

```
[📦 org.gnome.Podcasts ~]$ cat /app/manifest.json {
 "id" : "org.gnome.Podcasts", "runtime" : "org.gnome.Platform", "runtime-version" : "41", "runtime-commit" : "a306cb5f76bba5e87223de4c98881ff65f32c2fba3dfac7af2673507d18dd741", "sdk" : "org.gnome.Sdk", "sdk-commit" : "be925b0b2a0092bd1372f130c72e7d2fffb53b238953469bb7943e75a0808bd7", ... } 
```

此外，他们承认并非一切都完美，但这只是因为在所有地方都实现复现性是一个难题（对每个人来说都是）。他们正在跟踪复现性状态的问题，并且还在与设计用于遵守归档法规的其他技术合作：

> Buildstream，我们用于运行时，但也用于构建endlessos和许多嵌入式系统的工具，在这方面做得更好。它被设计用于能够处理例如汽车行业中的归档法规，这是一种更复现性的方法。

### 但是...

虽然重现构建所需的信息确实存在于构建的包内部，但**这些信息未提交到源代码树**，这就是我在原文中列出的问题。Flatpak是可以复现的，但在构建工具方面仍有改进的空间，以提升开发者的体验。

* * *

## 介绍

根据[flatpak.org](https://flatpak.org)，"Flatpak是一种在Linux上构建和分发桌面应用程序的先进技术"。我相信flatpak正在成为事实上的标准应用程序分发格式：只需打开[flathub.org](https://flathub.org)看看已经有多少flatpak可用；它得到了广泛的发行版支持，完全开源（甚至是服务器端）；总体上，它得到了用户和开发者的高度赞赏。我希望flatpak能够成功，但我认为他们在某些基本事项上搞错了。幸运的是，这些问题是可以解决的。

要创建一个flatpak，作为开发者，你需要两个工具：`flatpak` 和 `flatpak-builder`。

> "Flatpak-builder是一个从源代码构建flatpaks的工具。"

而`flatpak`工具则主要用于安装flatpaks和管理flatpak仓库。

## flatpak清单

要打包flatpak应用程序，您需要声明一个清单，即一个JSON文件，将由`flatpak-builder`读取。此文件必须声明您的应用程序需要的每个依赖项：运行时（一个常见的软件捆绑包）、运行时扩展（用于扩展运行时的常见软件）和自定义依赖项。

让我们看一个（简化的）例子，来自我的一个应用程序，[Geopard](https://ranfdev.com/projects/geopard)：

```
{
  "app-id" : "com.ranfdev.Geopard.Devel",
  "runtime" : "org.gnome.Platform",
  "runtime-version" : "42",
  "sdk" : "org.gnome.Sdk",
  "sdk-extensions" : [
  "org.freedesktop.Sdk.Extension.rust-stable"
 ],  "command" : "geopard",
  "finish-args" : [
  "--share=ipc",
  "--socket=fallback-x11",
  "--socket=wayland",
  "--device=dri",
  "--share=network",
  "--filesystem=xdg-download/Geopard"
 ],  ...   "modules" : [
 {  "name" : "Geopard",
  "buildsystem" : "meson",
  "sources" : [
 {  "type" : "git",
  "url" : "../",
  "branch": "master"
 } ] } ] } 
```

## 问题1：我应该从哪里获取运行时？

`flatpak-builder`将解析先前的清单并抱怨... 运行时`org.gnome.Sdk`未安装！好...但是我应该从哪里下载运行时？Flatpak软件是从存储库下载的，但在这种情况下，哪个存储库？无法确定，但当您不确定时，应始终尝试`https://flathub.org`，这是最大的公共存储库。

建立库并安装其他所需软件（请注意，我们必须手动完成！）：

```
flatpak --user remote-add flathub https://flathub.org/repo/flathub.flatpakrepo flatpak --user install org.gnome.Platform//42 # I've manually added the version (stated in the manifest) as a suffix. # ... 
```

### 解决方案

清单中应该有一个“repos”字段，以便人们（和工具）确切知道从哪里获取东西：注意：此字段不是可选的。**没有所需存储库的清单应被视为无效**。

```
{
 "app-id" : "com.ranfdev.Geopard.Devel", "repos" : ["https://flathub.org/repo/flathub.flatpakrepo"], ... } 
```

## 问题2：它的版本是多少？

在安装所需的运行时和扩展时，我们已经到了这一步：

```
$ flatpak --user install org.freedesktop.Sdk.Extension.rust-stable Looking for matches… Similar refs found for ‘org.freedesktop.Sdk.Extension.rust-stable’ in remote ‘flathub’ (user):   1) runtime/org.freedesktop.Sdk.Extension.rust-stable/x86_64/1.6 2) runtime/org.freedesktop.Sdk.Extension.rust-stable/x86_64/18.08 3) runtime/org.freedesktop.Sdk.Extension.rust-stable/x86_64/19.08 4) runtime/org.freedesktop.Sdk.Extension.rust-stable/x86_64/20.08 5) runtime/org.freedesktop.Sdk.Extension.rust-stable/x86_64/21.08   Which do you want to use (0 to abort)? [0-5]: 
```

好问题... 我想使用哪个“ref”？... 我不知道。

### 解决方案

没有版本后缀的ref *必须* 被视为无效。

无效：

```
{"sdk-extensions" : [
  "org.freedesktop.Sdk.Extension.rust-stable" ]} 
```

有效：

```
{"sdk-extensions" : [
  "org.freedesktop.Sdk.Extension.rust-stable//21.08" ]} 
```

## 问题3：可重现性

由于没有足够的信息来知道所需的依赖关系，很明显没有可重现性。官方flatpak网站上的文字误导：“在与用户相同的环境中开发和测试您的应用程序。”甚至开发人员都没有一致的开发环境。

### 解决方案

一旦`flatpak-builder`成功构建清单，它应该将所使用的确切依赖项写入锁定文件。这种模式对于关心可重现性的每个软件包管理器都很常见：Nix、cargo、npm等等... 我明白`flatpak-builder`不是一个软件包管理器，但是在flatpak构建链中的*某个东西*应该创建锁定文件。

我也明白为什么人们可能实际上*不想*锁定他们的运行时到特定版本：如果运行时被锁定，没有程序员的干预就无法更新，因此应用程序无法自动重建以获取安全更新。这确实是一个问题，但可以通过软件存储库轻松解决，方法是简单地覆盖锁定文件。这样，开发人员就可以获得可重现的调试构建（以便实际上可以追踪错误），而用户可以接收安全更新。这仍然比现在好。

## 结论

这些（概念上）是简单的解决方案，但我认为它们可以极大地改善 Flatpak 生态系统。Nix 和 Guix，这两个最先进的包管理器，已经解决了这些问题。Flatpak 正在尝试成为既是包管理器又是应用分发系统。后者部分很好，前者部分只是…… 糟糕。

* * *

版权：Ranfdev 2019-2023
