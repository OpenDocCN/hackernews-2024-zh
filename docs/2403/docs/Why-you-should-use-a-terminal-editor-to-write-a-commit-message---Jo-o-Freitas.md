<!--yml
category: 未分类
date: 2024-05-29 12:33:31
-->

# Why you should use a terminal editor to write a commit message | João Freitas

> 来源：[https://joaomagfreitas.link/why-you-should-use-a-terminal-editor-to-write-a-commit-message/](https://joaomagfreitas.link/why-you-should-use-a-terminal-editor-to-write-a-commit-message/)

 <content>Ever since starting to only use Git in the command line, I’ve been writing commit messages through a **terminal editor** (`nano`), instead of passing the message (`-m`) flag. Some colleagues find it odd that I do it this way, so here are **seven** reasons in favor of using a terminal editor:

1.  First of all, you need to **pass** the `-m` flag, and that alone is costing you time.
2.  You need to **wrap** your message in quotes/ticks (""). Finding them on your keyboard can be somewhat hard.
3.  You can’t break lines in your message. Your brain is always under pressure to not click the **enter** key, otherwise, you will need to *amend* the commit message.
4.  If the repository is configured with `pre-commit` hooks, they won’t be executed until you click the **enter** key. This is particularly bad if commit messages need to follow a specific structure.
5.  Oh, you managed to click the enter key and **accidentally created** a commit? You need to repeat the whole process to amend the commit message (this includes copying/writing the message and passing the `--amend` flag).
6.  Your terminal shell interpreter (`bash/zsh/csh...`) is very **sensitive** to **special characters**. To include them, you will need to think of ways to either escape them or make sure the interpreter won’t evaluate them. (e.g., imagine you need to write this message: `fix: apply "$HOME" instead of using single ticks (''))`)
7.  Copy-pasting a message might give you a hard time. If the message includes **Unicode** characters that your terminal has issues to **render**, like emojis, it will be a pain to **correct** them.

All of these issues can be avoided if you learn the basics of a terminal editor (`nano` is really simple!) and type `git commit` (or even better, declare a shorthand alias like `gc` that will save you a ton of time).</content>