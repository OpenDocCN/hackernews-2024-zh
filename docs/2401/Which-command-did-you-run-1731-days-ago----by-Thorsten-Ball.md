<!--yml

类别：未分类

日期：2024 年 05 月 27 日 14:59:50

-->

# 你 1731 天前运行了哪个命令？– 作者：Thorsten Ball

> 来源：[`registerspill.thorstenball.com/p/which-command-did-you-run-1731-days`](https://registerspill.thorstenball.com/p/which-command-did-you-run-1731-days)
> 
> *亨利·琼斯教授*：找到圣杯的人必须面对最后的挑战。
> 
> *印第安纳·琼斯*：什么最后的挑战？
> 
> *亨利·琼斯教授*：如此致命的三个装置。
> 
> *印第安纳·琼斯*：陷阱？
> 
> *亨利·琼斯教授*：哦，记得。但我在《圣安瑟姆编年史》中找到了能安全地引导我们通过它们的线索。
> 
> *印第安纳·琼斯*：[高兴]那么，它们是什么？
> 
> [亨利试图回忆时短暂停顿]
> 
> *印第安纳·琼斯*：你不记得了吗？
> 
> *亨利·琼斯教授*：我把它们记在日记中，这样就不必*记住*。

*******

在外壳中过上美好生活的秘诀：

1.  确保它运行速度快。

1.  确保它的历史几乎可以无穷无尽地增长，并且可以通过模糊搜索来查找。

第一个–快速外壳–[我们上周讨论过](https://registerspill.thorstenball.com/p/how-fast-is-your-shell)，所以这一次，让我们来谈谈外壳的历史。

在我的机器上，`~/.zsh_history` 文件包含了 26278 条命令。它最早的记录是在 2019 年 4 月 25 日。当我在 ZSH 中按下`Ctrl-r`时，[fzf](https://github.com/junegunn/fzf)弹出并让我通过所有 26278 条命令进行模糊搜索，查找我过去 5 年的外壳历史。真是太棒了。

那个使用这四个奇怪的 HTTP 头部组成的`curl`命令，我必须发送它才能确保通过反向代理正确与我的本地开发环境通信 – 它就在那里，只需几次击键和一些模糊的记忆。另一个使用正确的 SSH 客户端二进制文件并保留时间戳和权限的`rsync` – 也在其中。那个将`git grep`与`find`结合以及 8 个丑陋的 AWK 咒语链在一起的令人讨厌的命令 – 也被锁在其中。昨天我运行的测试命令和 3 年前我运行的具有 12 个`&&`的命令，都得益于一个长长的外壳历史文件。

实际上：`~/.zsh_history` 是我机器上最重要的文件之一。我每天使用它数十次，每当我按`Ctrl-r`时都会用到它。如果我不是一个傻瓜，它可能包含了更多的 26278 条命令，因为我几乎十年来一直保留着长时间的外壳历史。但由于我 *是* 一个傻瓜，我忘记了备份它（或者删除备份的太早）几次，因此我在打字的这台机器上的`~/.zsh_history`只包含了 1034 条命令。

有人可能会说：“好吧，如果这个命令那么重要，为什么不将它保存到脚本中呢？” 对此，我只能说：听着，伙计，如果我提前知道哪个命令会变得重要，那么我们就不会在这里了，因为我早就升上了星体领域。

此外：为什么我要将其记录在脚本中，而不是简单地配置我的 shell 来为我记录所有这些命令呢？这很廉价，速度很快，基本上没有任何缺点：我最大的`~/.zsh_history`只有 2MB，我可以在*毫秒*内对其进行模糊搜索。

*******

所以，重申一遍：你的 shell——ZSH、Bash、Fish 或其他——可以记录你在其中执行的每个命令，我恳求你确保它被配置为这样做。

确保它能记录*很多*命令。那么多的命令，以至于当你在你的 rc 文件中设置这个数字时，你会想：这太荒谬了，为什么我需要记录那么多命令？把这个数字翻倍。作为下一步和最后一步，你需要获取一个模糊搜索器来搜索它。

这是我多年来一直使用的[ZSH 配置](https://github.com/mrnugget/dotfiles/blob/2bdf21b659cbf34f21a0716bfac1f90914426a87/zshrc#L18-L35)：

```
`##########
# HISTORY
##########

HISTFILE=$HOME/.zsh_history
HISTSIZE=50000
SAVEHIST=50000

# Immediately append to history file:
setopt INC_APPEND_HISTORY

# Record timestamp in history:
setopt EXTENDED_HISTORY

# Expire duplicate entries first when trimming history:
setopt HIST_EXPIRE_DUPS_FIRST

# Dont record an entry that was just recorded again:
setopt HIST_IGNORE_DUPS

# Delete old recorded entry if new entry is a duplicate:
setopt HIST_IGNORE_ALL_DUPS

# Do not display a line previously found:
setopt HIST_FIND_NO_DUPS

# Dont record an entry starting with a space:
setopt HIST_IGNORE_SPACE

# Dont write duplicate entries in the history file:
setopt HIST_SAVE_NO_DUPS

# Share history between all sessions:
setopt SHARE_HISTORY

# Execute commands using history (e.g.: using !$) immediatel:
unsetopt HIST_VERIFY` 
```

这与[fzf](https://github.com/junegunn/fzf)结合起来，可以在按下`ctrl-r`时模糊搜索历史记录。

将你的 shell 配置大致如此，你就能享受美好的 shell 生活了。

或者你可以使用[Atuin](https://github.com/atuinsh/atuin)，我最近几周一直在尝试并且喜欢上了它。它是你 shell 历史功能的一个即插即用的增强，不仅提供了我上面描述的所有功能——无比长的 shell 历史记录和模糊搜索——而且还可以在多台机器之间同步和备份（是的，需要这个功能），按照机器过滤命令，工作目录特定的历史记录，可能还有我忘记的 13 件其他事情。它非常棒。强烈推荐。

无论你使用什么：确保你的 shell 历史记录被记录下来，确保你的 shell 历史记录可以变得非常长，然后获取一个模糊搜索功能。
