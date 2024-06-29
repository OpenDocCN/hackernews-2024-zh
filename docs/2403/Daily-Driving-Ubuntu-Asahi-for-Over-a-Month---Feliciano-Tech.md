<!--yml

category: 未分类

date: 2024-05-29 12:35:30

-->

# 在过去一个多月每天驱动Ubuntu Asahi | Feliciano.Tech

> 来源：[https://www.feliciano.tech/blog/daily-driving-ubuntu-asahi-for-over-a-month/](https://www.feliciano.tech/blog/daily-driving-ubuntu-asahi-for-over-a-month/)

最近我被裁员了，全面掌控了我的MacBook Pro。我不是macOS的粉丝，但我使用它是因为苹果系统被锁定。不再如此。Asahi Linux已经走了很长一段路，衍生出了Fedora Asahi和最近的[Ubuntu Asahi](https://ubuntuasahi.org/)。Ubuntu是我主要的操作系统已经约14年了，所以能在我的MacBook Pro上本地运行它是非常吸引我的。

有了完全的管理员访问权限，我安装了Ubuntu Asahi。安装程序版本是v0.7.1（本文撰写时的当前版本）。这个过程令人耳目一新地简单。遵循[网站上的说明](https://ubuntuasahi.org/)，您就可以开始使用了。

## 硬件支持

总体来说，还不错。大部分让我不满意的问题都有解决方法。

就像在macOS上一样，默认情况下，功能键操作为媒体键。要执行所需的任务，需要按下‘fn’按钮与功能键。尽管Ubuntu没有像macOS那样的UI方法来更改这种行为，但传奇的Arch Linux wiki提供了[终端解决方案](https://wiki.archlinux.org/title/Apple_Keyboard#Function_keys_do_not_work)。总结页面，要进行临时切换，您可以运行：

|

```
1 
```

|

```
echo 2 >> /sys/module/hid_apple/parameters/fnmode 
```

|

要使切换永久生效，您需要放置以下文本：

|

```
1 
```

|

```
options hid_apple fnmode=2 
```

|

在文件中：

|

```
1 
```

|

```
/etc/modprobe.d/hid_apple.conf 
```

|

永久解决方案对我还没有起作用。每次重启后我都需要执行临时解决方案。

我还遇到了声音和麦克风问题。对于声音，80%的时间都工作正常。在其他20%的时间里，声音会出问题，我无法调整音量，音频听起来很响亮刺耳，似乎要把扬声器吹爆。我不确定原因，但这很烦人。幸运的是，我经常在笔记本电脑上使用耳机，所以这并不经常影响我。

对于麦克风，我遇到了一个问题，当使用Google Meet时它不起作用。如果我在Firefox中尝试，麦克风被检测到但不起作用。当我在Chromium中使用Google Meet时根本找不到麦克风。奇怪的问题。

我最近的硬件笔记与电池寿命有关。我在某处读到一篇关于Asahi Linux的笔记，提到它在使用苹果芯片时与macOS有着相同的电池续航时间。我发现这是真的，但并非总是如此。我经常观察到电池寿命以我熟悉的macOS的速度消耗。对我来说问题出在关闭笔记本盖子时。我不知道背后发生了什么，但有时候，几个小时后，电池仅会消耗2%，有时候会是30%。

## 软件支持

到目前为止，软件是我在Ubuntu Asahi上遇到的最大问题，但这里有个好消息。问题不在于Ubuntu自带的软件包，但有一个例外。Gnome Shell 偶尔会崩溃。类似于电池问题，只有在锁定屏幕时（通过键盘或关闭盖子）发生才会出现。发生时，我必须重新打开之前打开的任何应用程序。这个Gnome Shell问题是我不推荐Ubuntu Asahi作为日常驱动器的原因。如果你正在处理重要文件而Gnome Shell崩溃，那将会令人沮丧。

我的问题主要在于arm64架构上第三方软件的可用性。你可能会认为一些不那么依赖Linux的软件可能会遇到这个问题，但即使是像Firefox和PostgreSQL这样的老手也会出现问题。

让我们从Google Chrome开始。像许多现代技术人员一样，我在浏览器中花费了异常大的时间。我选择的浏览器是Google Chrome。今天我不会谈论浏览器政治，但我喜欢Chrome，并使用了许多其特性。其中一些在Chromium中没有。Google Chrome并未为Linux/arm64编译。macOS有适用于Arm的构建吗？有。Windows呢？有。但Linux没有。这种不均衡的平台支持对我来说是个大麻烦。

### Snaps

我不情愿地决定在Ubuntu Asahi上使用Firefox，但我想快速更改一下。几个版本前，Canonical决定强制使用Firefox的snap而不是.deb。[我不是snap的粉丝](/blog/breaking-up-with-snap/)，所以我试图切换到使用Mozilla官方的deb仓库，但遇到了问题。该仓库目前不支持arm64。我不得不改用PPA。

我目前使用的几个snap都是图形应用程序，我尝试安装它们。Mattermost可以使用，但Slack、Todoist、Simple Note、Spotify和Telegram都不能。再次强调，这些snap不支持arm64。

幸运的是，我找到了两个Telegram的解决方案。尽管snap不支持arm64，但[Flatpak支持](https://flathub.org/apps/org.telegram.desktop)。😉不久之后，我在[Linux Matters](https://linuxmatters.sh/)播客中听到，其中一位主持人Alan专门为Ubuntu Asahi制作了他自己的[Telegram snap](https://snapcraft.io/telegram-asahi)。虽然这很棒，但这两个选项都是社区制作的，所以请自行决定是否使用。

最后，我要谈一下PostgreSQL。由于Ubuntu Asahi最新支持的Ubuntu版本是Ubuntu 23.10，它所包含的最新版本的Postgres是v15。我需要v16。Postgres提供了仓库，但有一个有趣的转折。他们提供了arm64构建，但仅适用于Ubuntu LTS版本，而我没有使用这些版本。这对我来说是另一个与Ubuntu Asahi的兼容性问题。

## 结论

目前对我来说，Ubuntu Asahi 只有两个主要问题。其一，Gnome Shell 偶尔会崩溃。其二，没有Google Chrome。

基于我的评估，很容易会认为运行 Ubuntu Asahi 存在许多问题。我只列出了我遇到的问题，因为列出所有正常运行的内容会占据一本书的篇幅。Ubuntu Asahi 基于 Asahi Linux 团队、Canonical 和 Ubuntu 社区的卓越工作构建，为他们提供了一个很好的起点。

如果现在问我是否推荐日常使用 Ubuntu Asahi，我会说如果你有多台机器可以使用的话，我会推荐。如果你只有一台电脑可用，我建议暂时等等。虽然我认为这个项目离稳定还有一段距离。根据他们目前的进展，我可以在 2024 年结束前给出肯定的答复。我对这个项目的发展感到非常兴奋。
