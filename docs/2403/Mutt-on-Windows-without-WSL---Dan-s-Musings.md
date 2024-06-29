<!--yml

category: 未分类

date: 2024-05-29 12:40:33

-->

# 在Windows上使用MSYS2而无需WSL — 丹的思考

> 来源：[https://blog.djhaskin.com/blog/mutt-on-windows-without-wsl/](https://blog.djhaskin.com/blog/mutt-on-windows-without-wsl/)

## 介绍

我有一个奇怪的爱好，就是试图让好的Unix工具在Windows上本地运行。我发现这是我笔记本电脑资源使用效率最高的方式，因此是最愉快的体验。WSL很好用，但它在我的系统上占用了大量内存。我更喜欢使用[MSYS2](https://www.msys2.org/docs/what-is-msys2/)。在MSYS2下正确地使用Unix工具是一个挑战，正如我们将看到的，对我来说，这也是一部分乐趣。

## 先决条件

这些先决条件不仅适用于使用mutt，而且适用于在任何开发或其他工作中成功使用msys2。这些是我遵循的步骤，以最大化在msys2上成功的机会。

+   Msys2需要安装。我喜欢使用[scoop](https://scoop.sh/)安装它（以及Windows上的其他所有东西，最好）。只需确保运行`scoop hold msys2`。当你可以只需在msys2内运行`pacman -Syu`获取所有所需的更新时，你不希望scoop每隔半秒钟就重新安装msys2以进行“更新”。

+   在你的USERPROFILE路径中不应该有空格。我知道这很难，但这很值得。我通常通过创建一个名为`nospaces`或类似的本地用户来实现这一点，在那个用户中登录，然后在我的`C:\Users\<me>`文件夹创建后，再登录我的Microsoft Windows账户。

+   你的MSYS2安装应该将此作为其`/etc/fstab`，这是确保你的MSYS2主目录也是你的`USERPROFILE`目录的唯一正确方法：

```
# For a description of the file format, see the Users Guide
# https://cygwin.com/cygwin-ug-net/using.html#mount-table

# DO NOT REMOVE NEXT LINE. It remove cygdrive prefix from path
none / cygdrive binary,posix=0,noacl,user 0 0
C:/Users    /home   ntfs    binary,posix=0,noacl,user 0 0 
```

如果你使用scoop安装msys2，你可以将上述文件放在`C:\Users\<you>\scoop\apps\msys2\current\etc\fstab`下，应该就能工作。

## 说明

+   从mingw64 MSYS2提示符中，输入以下内容以安装gpg、mux和msmtp：

```
pacman -S gnupg mutt mingw64/mingw-w64-x86_64-msmtp w3m 
```

+   我们按照通常的方法配置mutt，但我发现mutt由于某种原因无法发送电子邮件。因此，我配置它使用[msmtp](https://marlam.de/msmtp/)处理此部分，然后一切正常。这是我的gmail配置（位于`~/.mutt/home.account`下）：

```
set imap_user = "djhaskin987@gmail.com"
set folder = "imaps://imap.gmail.com/"
set spoolfile="+INBOX"
set postponed = "+[Gmail]/Drafts"
set record = "+[Gmail]/Sent Mail"
set trash = "+[Gmail]/Trash"
set imap_pass = `multipass imaps://imap.gmail.com:993/`

set realname = "Daniel Jay Haskin"
set from="djhaskin987@gmail.com"
set sendmail="/mingw64/bin/msmtp -a home"

set crypt_use_gpgme
set pgp_default_key="443A163BD11CEAE798BAAB94D7268D49D06594F4"

source ~/.muttrc 
```

在这个文件中，我使用一个名为[multipass](https://git.sr.ht/~skin/dotfiles/tree/main/item/dot-local/bin/multipass)的shell脚本，它通过[git-credential-keepassxc](https://github.com/Frederick888/git-credential-keepassxc)从[KeePassXC](https://keepassxc.org/)中查找密码。

我们还看到我使用`msmtp`配置为使用home账户发送邮件。

这是我的`msmtp`配置：

```
defaults
auth on
tls on
tls_trust_file C:/Users/bhw/scoop/apps/msys2/current/usr/ssl/certs/ca-bundle.crt
logfile ~/.msmtp.log

account home
host smtp.gmail.com
port 465
tls_starttls off
from djhaskin987@gmail.com
user djhaskin987@gmail.com
passwordeval "C:\Users\bhw\Executables\multipass.bat smtps://smtp.gmail.com:465/ djhaskin987@gmail.com"

account migadu
host smtp.migadu.com
port 465
tls_starttls off
from dan@djhaskin.com
user dan@djhaskin.com
passwordeval "C:\Users\bhw\Executables\multipass.bat smtps://smtp.migadu.com:465/ dan@djhaskin.com"

account default: home 
```

由于“msys2”子系统而不是“mingw64”子系统编译，所以`msmtp`程序必须使用[*不同的多通道（multipass）*](https://git.sr.ht/~skin/dotfiles/tree/main/item/Windows/Executables/multipass.bat)，因为它是“Windows感知”的而不是“msys2感知”的。顺便说一句，上述文件必须写入`~/msmtprc.txt`，这是我遇到的另一个注意点。正是在这种来回窗口-非窗口的边界上，msys2才显得如此重要，因为它的主目录必须与“Windows”主目录相同。我发现这样有时候会让事情不那么混乱。

最后，这是我的`~/.muttrc`：

```
set move = no
set confirmappend = no
alternative_order text/plain text/html
auto_view text/html
set mailcap_path = ~/.mailcap

set envelope_from=yes 
```

无论如何，我能够在msys2 mingw64终端上启动并使用mutt：

```
mutt -F ~/.mutt/home.account 
```
