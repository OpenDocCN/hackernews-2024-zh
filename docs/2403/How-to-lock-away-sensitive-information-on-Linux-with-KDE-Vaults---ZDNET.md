<!--yml

类别：未分类

日期：2024-05-27 14:49:03

-->

# 如何在 Linux 上使用 KDE Vaults 锁定敏感信息 | ZDNET

> 来源：[https://www.zdnet.com/article/how-to-lock-away-sensitive-information-on-linux-with-kde-vaults/](https://www.zdnet.com/article/how-to-lock-away-sensitive-information-on-linux-with-kde-vaults/)

DBenitostock/Getty Images

您可能有些文档希望远离窥探的眼睛。这些可能是合同、遗嘱、银行账户信息、契据，或者任何包含关于您、您的家人或生活细节的内容，不应该落入错误手中的东西。

锁定文档的最佳方法之一是使用加密。不幸的是，并非每个操作系统都能轻松管理这样的任务。幸运的是，如果你选择使用[使用 KDE Plasma 桌面环境](https://www.zdnet.com/article/kde-neon-shows-that-the-plasma-6-linux-distro-is-something-truly-special/)的 Linux 发行版，你将会惊喜地发现，由于 KDE Vaults 的存在，创建加密文件夹非常简单。

**同时：[每个新用户都应该学会的前5个Linux命令](https://www.zdnet.com/article/the-first-5-linux-commands-every-new-user-should-learn/)**

KDE Vaults 是一个内置系统，提供了一个用户友好的 GUI 来帮助您创建加密保险库，用于锁定您更敏感的文档。只需几次鼠标点击，您就可以创建一个加密保险库，准备好容纳所有不应该被桌面上任何有权限访问的人查看的文档（图片和其他文件）。

我将引导您完成在展示最新桌面版本 KDE Plasma 6 的 KDE Neon 上创建第一个保险库的过程。

## 如何创建您的第一个保险库

**你需要的东西**：唯一需要的是运行 KDE Plasma 桌面环境的 Linux 发行版实例。我建议使用[KDE Neon](https://www.zdnet.com/article/kde-neon-shows-that-the-plasma-6-linux-distro-is-something-truly-special/)发行版，因为它包含了 KDE 开发者的最新发布。

访问初始 KDE Vault 创建的最快方式是通过通知弹出窗口。在您的 KDE Plasma 面板上，点击向上的箭头以显示“状态和通知”弹出窗口。然后点击“Vaults”。

创建了第一个保险库后，您将不会在通知弹出窗口中找到“Vaults”入口。

ZDNET/Jack Wallen

从弹出的窗口中选择“创建新的保险库”。

这使您可以访问“Vaults”创建向导。

ZDNET/Jack Wallen

接下来的交互屏幕需要您命名新的保险库。输入一个名称并点击“下一步”。

您可以随意命名您的保险库。

ZDNET/Jack Wallen

接下来的屏幕为您提供一个安全通知，基本上提醒您，虽然 CryFS 被认为是安全的，但尚未进行独立的安全审计来确认这一说法。继续点击“下一步”。

随后您将被提示为新保险库输入并验证密码。完成后，点击“下一步”。

输入并验证新保险库的密码。

ZDNET/Jack Wallen

默认情况下，保险库将创建在~/.local/share/plasma-vault/NAME目录下（其中NAME是您给保险库起的名字），挂载点为~/Vaults/NAME。您可以保留这些默认设置，或者根据需要进行更改。例如，您可能希望将您的保险库放在非标准目录中。确认位置后，点击“下一步”。

您可以保留或更改新保险库的默认设置。

ZDNET/Jack Wallen

在接下来的窗口中，您可以为加密选择不同的密码，限制保险库的某些活动，并在打开保险库时强制使您的计算机脱机。完成这些配置后，点击“创建”，您的第一个保险库将准备就绪。

## 使用你的新保险库

如果您在面板中点击保险库图标（一个带锁的文件夹），将弹出一个窗口显示您的新保险库。在此列表中，您可以点击锁定保险库（默认情况下，登录时是解锁的），或者点击名称并（从展开的菜单中）打开Dolphin文件管理器到新保险库，强制锁定保险库，或配置保险库。

您的新保险库将显示在KDE Plasma面板中的“保险库”弹出菜单中。

ZDNET/Jack Wallen

有一件事可能会让您困扰：一旦保险库被锁定，您仍然会在文件管理器中看到它。如果您点击该条目，似乎您仍然可以访问该文件夹。然而，您只能访问挂载点，里面什么都没有。您只能在保险库解锁时（这时会将解密后的保险库挂载到挂载点上）查看保险库的内容。

**另外：[Linux 上是否需要安装防病毒软件？](https://www.zdnet.com/article/do-you-need-antivirus-on-linux/)**

这就是创建和使用KDE保险库功能的全部内容。一旦您创建了新保险库，请将所有需要安全存放的文件添加进去，然后将保险库锁起来。一旦锁定，只有知道加密密码的人才能访问其中的内容。
