<!--yml

类别：未分类

日期：2024-05-27 14:50:38

-->

# shelly/README.md at main · naskovai/shelly · GitHub

> 来源：[`github.com/paletov/shelly/blob/main/README.md`](https://github.com/paletov/shelly/blob/main/README.md)

# Shelly：用英语编写终端命令

[](#shelly-write-terminal-commands-in-english)

Shelly 是一个强大的工具，它将英语翻译成可以在您的终端中无缝执行的命令。您不再需要记住确切的命令了。

调用 Shelly 并给出指令后，它会在终端中打开一个像 nano/vim 的编辑器，并显示建议的命令。保存后会执行最终版本。

Shelly 使用人工智能来解释普通英语并将其翻译成准确的终端命令。特别是，Shelly 使用 OpenAI 的 GPT-4，因此您需要一个 OpenAI API 密钥。

```
shelly 'git last 5 commits'
  $ "git log -5"

shelly 'check size of files in this directory but format it with gb/mb, etc'
  $ "du -sh *"

shelly 'find "class" in files in this directory without the .venv dir'
  $ grep -r --exclude-dir=.venv "class" .
```

通过 Homebrew 安装 Shelly：

```
brew untap paletov/shelly
brew tap paletov/shelly https://github.com/paletov/shelly
brew install shelly
```

享受使用 Shelly 吧！
