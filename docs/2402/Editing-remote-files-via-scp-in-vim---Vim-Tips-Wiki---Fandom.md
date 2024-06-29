<!--yml

category: 未分类

date: 2024-05-29 13:23:17

-->

# 通过 scp 在 vim 中编辑远程文件 | Vim 技巧维基 | Fandom

> 来源：[https://vim.fandom.com/wiki/Editing_remote_files_via_scp_in_vim](https://vim.fandom.com/wiki/Editing_remote_files_via_scp_in_vim)

**重复的提示**

此提示与以下内容非常相似：

这些提示需要合并 - 请参阅[合并指南](/wiki/Vim_Tips_Wiki:Merge_guidelines "Vim 技巧维基：合并指南")。

Vim 6.x 已安装 netrw 插件作为标准插件。它允许您通过 ftp、rcp、scp 或 http 编辑文件。然而，如果您在远程主机上的用户名不同，并且尝试使用 scp，事情可能会变得有些奇怪，特别是如果您不是在用户树下编辑文档。

要绕过这一点，请尝试以下方式打开文件：

```
vim scp://remoteuser@server.tld//absolute/path/to/document

```

类似地，您可以通过运行以下命令在 Vim 中的新缓冲区中打开文件：

```
:e scp://remoteuser@server.tld//absolute/path/to/document

```

或者在新标签页中使用：

```
:tabe scp://remoteuser@server.tld//absolute/path/to/document

```

注意两件事情：

1.  `remoteuser@`：这用于指定远程服务器上的用户名。如果没有这个，它将使用本地计算机上的用户名。通常这将来自 $USERNAME 环境变量。如果本地计算机和远程服务器上的用户名称相同，则此部分是不必要的。如果您不确定是否需要，请使用它以确保安全。

1.  主机名和文件路径之间的双斜杠("`//`")：至少需要一个斜杠来分隔远程服务器的主机名和文件路径。该斜杠不包括在用于引用远程服务器上文件的路径中。如果文件路径是绝对的，则必须以斜杠开头，给出主机名和文件路径之间的两个斜杠如上所示。

    然而，如果要编辑的文件位于远程用户的主目录中，可以使用相对路径，不应使用第二个斜杠。例如，如果要编辑的文件的绝对路径为 `/users/remoteuser/relative/path/to/document`，并且 remoteuser 的主目录为 `/users/remoteuser`，则以下命令将打开该文件：

    ```
    vim scp://remoteuser@server.tld/relative/path/to/document
    ```

## [[](/wiki/Editing_remote_files_via_scp_in_vim?veaction=edit&section=1 "编辑部分：评论")]

[latest netrw.vim](http://www.drchip.org/astronaut/vim/index.html#NETRW) 已有几个改进。支持后续 Windows ftp，新协议（rsync、cadaver、fetch），用户修复函数等等。

* * *

我们如何存储密码？它每次保存时都会提示！

* * *

我刚刚在 Win2k 上使用 PuTTY 的命令行 scp 程序 [http://www.chiark.greenend.org.uk/~sgtatham/putty/](http://www.chiark.greenend.org.uk/~sgtatham/putty/) 得以运行。

+   将 pscp.exe 复制到您的路径中，命名为 scp.exe

+   将 "let g:netrw_cygwin= 0" 放入您的 $VIM/_vimrc 文件中

* * *

可能会出现的陷阱：

如果未按照提示指定的路径（并注意），您可能会遇到非直观的错误：在编辑文件路径的绝对路径之间不放置 "//" 可能会导致 vim 尝试通过 rcp 检索文件，如下所示

```
:!rcp scp://m@mymachine.com:t1

```

并导致错误。还要注意，要在远程机器上放置文件的绝对路径，而不是相对于远程用户主目录的路径。

* * *

使用相对路径是非常正常且得到良好支持的。尝试

```
:r scp://m@machine/t1

```

* * *

有人问你是否可以为ftp定义端口

```
vim ftp://[user@]machine[[:#]portnumber]/path

```

尝试那个……就像任何其他的网址一样。

```
vim ftp://stankonia@domainname.com:6090/public_html/index.html

```

我猜这样会起作用。

* * *

有一个保存密码的好方法：在你的主目录下创建`.netrc`文件，并放入一行，每个ftp机器一个，就像这样一个：

```
machine yourftp.somewhere.org login yourlogin password "yoursecret"

```

* * *

ftp远程编辑没问题。

运行命令：

```
gvim <host>//<path_2_file> ftp://<host>//<path_2_file>;

```

然后输入用户名和密码。

* * *

如果一切看起来设置正确，但你仍然无法通过ftp访问文件，请检查你的`.netrc`文件的权限。如果`.netrc`被除所有者外的其他人读取，则ftp自动失败。

```
chmod 600 .netrc

```

* * *

在vimrc中为我解决了在putty中的问题：

```
let g:netrw_cygwin = 0
let g:netrw_scp_cmd = "\"C:\\Program Files\\PuTTY\\pscp.exe\" -pw mypasswd "

```

现在从wincommander运行：

```
gvim scp://user@server.xxx.cz/file.txt

```

* * *

获取最新版本来解决你遇到的任何问题。

* * *

当我在远程端以root身份使用scp连接并编辑远程站点上的本地用户文件时，因为不得不输入远程用户文件的完整路径而感到有点烦恼，我发现我可以这样做，而vim会“做正确的事情”。

```
vim scp://root@example.com/~user/public_html/.htaccess

```

这比起不得不麻烦：

```
vim scp://root@example.com//home/user/public_html/.htaccess

```

在这个例子中可能并不是那么痛苦，但如果你正在使用Ensim for Linux系统，你得到的一切都被chroot，这使得你不得不输入一个非常长的路径，比如：

```
vim scp://root@example.com//home/virtual/site2/fst/var/www/html/.htaccess (yawn)

```

* * *

要更改scp端口，有几个选项。一个快速的方法是当你打开vim时输入这个：

```
:let g:netrw_scp_cmd="scp -q -P <desired_new_port>"

```

然后只需输入：

```
:e scp://my_user@remote_hostname//path/to/remote/file

```

--> 我认为一个更好的解决方案是使用ssh机制，即`~/.ssh/config`文件：

```
Host lala
 HostName test.machine.example.net
 User remoteuser
 IdentityFile ~/.ssh/id_for_test.machine
 Port 57

```

* * *

我已经发现如何使被动模式的ftp工作。参见[http://alecthegeek.github.io/blog/2007/02/06/handy-hack-how-to-use-vim-netrw-in-ftp-passive-mode/](http://alecthegeek.github.io/blog/2007/02/06/handy-hack-how-to-use-vim-netrw-in-ftp-passive-mode/)

* * *

尝试在WinSCP中使用`"C:\Program Files\Vim\vim71\gvim.exe" --remote-tab !.!`以在同一个gVim实例的不同标签中打开每个文件。我还勾选了“外部编辑器在一个窗口中打开多个文件”复选框。还有另一个选项`--remote-silent`，它将抑制第一个警告，即没有已经运行的gVim，但你不能与`--remote-tab`选项一起使用它。我更喜欢确保在打开任何东西之前我在新标签页中而不是消除一个警告。----

--前面的[未签名](https://en.wikipedia.org/wiki/Signatures "wikipedia:Signatures")评论由[161.88.255.139](/wiki/User:161.88.255.139 "User:161.88.255.139")（讨论 • [贡献](/wiki/Special:Contributions/161.88.255.139 "Special:Contributions/161.88.255.139")）于2008年4月18日20:49添加。

* * *

在WinSCP中使用`"C:\Program Files\Vim\vim71\gvim.exe" --remote-tab-silent !..!`作为外部编辑器完全可行。

* * *

对于Windows用户，我建议将PuTTY添加到系统路径（或不添加），并简单设置g:netrw_cygwin=0，g:netrw_scp_cmd=<PATH TO PSCP>，并使用Pageant管理您的私钥；这样您就不必将密码保存在vimrc文件中。当PSCP尝试连接时，Pageant会提供密钥给PSCP。

-- 前 [未签名](https://zh.wikipedia.org/wiki/%E7%94%A8%E6%88%B7%E8%A8%8A%E6%81%AF) 评论由 [216.145.54.7](/wiki/User:216.145.54.7 "User:216.145.54.7")（讨论 • [贡献](/wiki/Special:Contributions/216.145.54.7 "Special:Contributions/216.145.54.7")）添加，UTC时间 2009年5月20日 11:56

* * *

[bcvi](http://sshmenu.sourceforge.net/articles/bcvi/) 是一个与SSH/SCP和上述Vim NetRW插件一起工作的实用程序。当您使用SSH+bcvi登录远程服务器时，您可以`cd`进入任何目录，然后输入`vi filename`，gvim命令将在您的工作站上启动，并使用正确的SCP URL指向服务器上的文件。如果这听起来有点混乱，[bcvi文章](http://sshmenu.sourceforge.net/articles/bcvi/)通过示例和图片来阐明。

* * *
