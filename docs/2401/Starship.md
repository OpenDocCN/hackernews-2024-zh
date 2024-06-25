<!--yml

类别：未分类

日期：2024-05-27 15:12:58

-->

# Starship

> 来源：[https://starship.rs/](https://starship.rs/)

+   将初始化脚本添加到您的Shell配置文件中：

    #### Bash

    将以下内容添加到`~/.bashrc`的末尾：

    #### Fish

    将以下内容添加到`~/.config/fish/config.fish`的末尾：

    #### Zsh

    将以下内容添加到`~/.zshrc`的末尾：

    #### Powershell

    将以下内容添加到`Microsoft.PowerShell_profile.ps1`的末尾。您可以通过在PowerShell中查询`$PROFILE`变量来检查此文件的位置。通常路径为`~\Documents\PowerShell\Microsoft.PowerShell_profile.ps1`或-Nix上的`~/.config/powershell/Microsoft.PowerShell_profile.ps1`。

    #### Ion

    将以下内容添加到`~/.config/ion/initrc`的末尾：

    #### Elvish

    警告

    仅支持elvish v0.18或更高版本。

    将以下内容添加到`~/.elvish/rc.elv`的末尾：

    #### Tcsh

    将以下内容添加到`~/.tcshrc`的末尾：

    #### Nushell

    警告

    这将在未来更改。仅支持Nushell v0.78+。

    将以下内容添加到您的Nushell环境文件的末尾（通过在Nushell中运行`$nu.env-path`找到该文件）：

    并将以下内容添加到您的Nushell配置的末尾（通过运行`$nu.config-path`在其中找到它）：

    #### Xonsh

    将以下内容添加到`~/.xonshrc`的末尾：

    #### Cmd

    您需要使用[Clink](https://chrisant996.github.io/clink/clink.html)（v1.2.30+）与Cmd。将以下内容添加到名为`starship.lua`的文件中，并将此文件放置在Clink脚本目录中：
