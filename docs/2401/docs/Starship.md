<!--yml
category: 未分类
date: 2024-05-27 15:12:58
-->

# Starship

> 来源：[https://starship.rs/](https://starship.rs/)

*   Add the init script to your shell's config file:

    #### Bash

    Add the following to the end of `~/.bashrc`:

    #### Fish

    Add the following to the end of `~/.config/fish/config.fish`:

    #### Zsh

    Add the following to the end of `~/.zshrc`:

    #### Powershell

    Add the following to the end of `Microsoft.PowerShell_profile.ps1`. You can check the location of this file by querying the `$PROFILE` variable in PowerShell. Typically the path is `~\Documents\PowerShell\Microsoft.PowerShell_profile.ps1` or `~/.config/powershell/Microsoft.PowerShell_profile.ps1` on -Nix.

    #### Ion

    Add the following to the end of `~/.config/ion/initrc`:

    #### Elvish

    WARNING

    Only elvish v0.18 or higher is supported.

    Add the following to the end of `~/.elvish/rc.elv`:

    #### Tcsh

    Add the following to the end of `~/.tcshrc`:

    #### Nushell

    WARNING

    This will change in the future. Only Nushell v0.78+ is supported.

    Add the following to the end of your Nushell env file (find it by running `$nu.env-path` in Nushell):

    And add the following to the end of your Nushell configuration (find it by running `$nu.config-path`):

    #### Xonsh

    Add the following to the end of `~/.xonshrc`:

    #### Cmd

    You need to use [Clink](https://chrisant996.github.io/clink/clink.html) (v1.2.30+) with Cmd. Add the following to a file `starship.lua` and place this file in Clink scripts directory: