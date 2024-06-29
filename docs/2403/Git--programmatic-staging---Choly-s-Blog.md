<!--yml

category: 未分类

date: 2024-05-27 14:31:16

-->

# Git：程序化暂存 · Choly的博客

> 来源：[https://choly.ca/post/git-programmatic-staging/](https://choly.ca/post/git-programmatic-staging/)

# Git：程序化暂存

2024年3月1日 星期五

在过去的一年中，我一直在使用大量工具自动重写/重构代码。这些工具包括[semgrep](https://semgrep.dev/)、[ast-grep](https://ast-grep.github.io/)、LLMs和一次性脚本。在大型代码库上运行这些工具后，通常会出现许多额外的意外更改。这些范围从格式/空白符到LLMs的未请求修改。

后续的“清理”步骤非常手动且繁琐。我基本上是在运行`git add -p`并逐个暂存区块。有时候，这一步骤感觉比重写工具本身带来的生产力增益更抵消。

做了几次后，我意识到我暂存的大多数区块都包含一些常见文本。如果我能自动暂存包含搜索词的区块，我就能自动化很多工作！Git本身不支持这一功能，但可以通过[expect](https://linux.die.net/man/1/expect)工具轻松实现：

```
#!/usr/bin/expect -f  # Set timeout to prevent the script from hanging set timeout -1   # Get the search pattern as a command line argument if {[llength $argv] != 0} {  set pattern [lindex $argv 0] } else {  puts "Error: search pattern not provided" exit 1 }   # Open the interaction with git add -p spawn git add -p   # This is the main loop that handles the user interaction expect {  # This expect block is for the hunk that contains the provided pattern "*$pattern*Stage this hunk*" { send "y\r" exp_continue } # This expect block is for continuing to the next hunk "*Stage this hunk*" { send "n\r" exp_continue } eof } 
```

要安装此脚本，请将其保存在名为`git-add-match`的`PATH`中。安装完成后，使用方法如下：

运行此命令后，所有包含字符串“foo”的区块将被暂存。

### 编辑：

[lobste.rs上的用户建议](https://lobste.rs/s/2iogwz/git_programmatic_staging#c_q5btxo)使用[grepdiff](https://linux.die.net/man/1/grepdiff)替代：

```
git diff | grepdiff --output-matching=hunk PATTERN | git apply --cached 
```
