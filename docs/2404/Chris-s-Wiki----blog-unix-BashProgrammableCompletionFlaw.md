<!--yml

category: 未分类

date: 2024-05-27 13:06:47

-->

# Chris's Wiki :: blog/unix/BashProgrammableCompletionFlaw

> 来源：[https://utcc.utoronto.ca/~cks/space/blog/unix/BashProgrammableCompletionFlaw](https://utcc.utoronto.ca/~cks/space/blog/unix/BashProgrammableCompletionFlaw)

[Bash](https://en.wikipedia.org/wiki/Bash_(Unix_shell)) 具有一个有趣且广泛有用的功能称为 '[programmable completion](https://www.gnu.org/software/bash/manual/html_node/Programmable-Completion.html)'（这在以前已经提到过）。可编程完成使得 Bash 能够为当前程序自动完成命令行选项之类的事物。不幸的是，Bash 的可编程完成的一个缺陷是它不理解足够的 Bash 命令行语法，因此可能会妨碍您。

假设，非假设地，您正在 Debian-based Linux 系统上的 Bash 命令行中输入以下内容，并且在按下 TAB 键之前已输入 **粗体** 部分：

> `# **apt-get install $(grep -v '^#' somefi**`<TAB>

当您按下 TAB 键以完成文件名时，不会发生任何事情。这是因为 Bash 已经知道 apt-get 的参数，并且 '知道' 它们不包括文件（尽管这在今天已经不正确了，但不要紧）。Bash 不够聪明，不能识别您通过 '`$(`' 开始编写命令替换并且现在位于完全不同的完成上下文中，即 `grep` 的完成上下文，您应该允许完成文件名。

理论上 Bash 可以变得很聪明，但在实践中可能会遇到许多障碍。例如，在这里我们没有一个格式良好的命令替换，因为我们还没有输入闭合的 '`)`'；Bash 实际上需要做类似于编辑器自动完成的即时解析不完整代码的事情。也许 Bash 可以通过一些启发式方法做得更好，但当前的情况广泛易于解释和推理，即使结果有时会令人沮丧。

至少有两种方法可以禁用 Bash 中的这些可编程完成。您可以通过 '`shopt -u progcomp`' 完全关闭该功能，或者使用 '`complete -r`' 刷新所有注册的可编程完成。理论上，您可以使用 '`complete -p`' 查看当前完成列表，然后使用 '`complete -r <name>`' 删除其中一个，但实际上，直到我开始尝试使用它们进行完成，'complete -p' 并不总是列出程序。

(定义完成的 shell 片段的位置取决于系统，但 Linux 系统通常会将它们放在 '`/usr/share/bash-completion/completions`' 中。)

比我记忆力好的人还可以使用 M-/ 强制完成文件名，无论 Bash 的可编程完成认为应该放在那里。M-! 将完成命令名称，M-$ 将完成变量名称，M-~ 将完成用户名。您可以在 Bash 手册页中找到这些作为各种 'complete-*' readline（键绑定）命令名称。

（关于可编程完成的另一个一般性缺陷是，它依赖于为命令提供完成定义的人员是否正确执行，但他们并不总是做到这一点，[正如我在过去看到的那样](/~cks/space/blog/unix/TooSmartShellAutocompleteFailure)。）
