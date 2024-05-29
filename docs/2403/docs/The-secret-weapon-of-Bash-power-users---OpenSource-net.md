<!--yml
category: 未分类
date: 2024-05-29 12:40:28
-->

# The secret weapon of Bash power users - OpenSource.net

> 来源：[https://opensource.net/the-secret-weapon-of-bash-power-users/](https://opensource.net/the-secret-weapon-of-bash-power-users/)

I’m one of those Linux dinosaurs who likes to start every project from the command line. OK, maybe not quite every project, but many.

I’m also a member of that community that prefers [vi](https://en.wikipedia.org/wiki/Vi_(text_editor)), or **vim**, to **emacs** as my text editor.

Vi, or more accurately vi mode in the Bash shell, becomes a secret weapon for bash power users for a couple of reasons. Vi keystrokes are designed for rapid navigation and editing within a text file and applying these same keystrokes to the Bash command history lets you move quickly through past commands, saving time and keystrokes.

For quite some time now, I’ve combined both preferences by using vi mode in the [bash](https://opensource.net/programming-with-bash-part-3-loops/) shell. This means using vi keystrokes like `k` to move back one line – “up” – in the command history and `j` to move forward one line – “down”. Using `0` (that’s a zero) to move to the beginning of a command line and `$` to move to the end; or `cw` to change the word starting at the current character position in the command line, and so forth.

Basically vi mode in the bash shell treats the command history as though it were the file of text being edited in vi, or vim and vi keystroke sequences operate on the history line where the cursor is currently positioned. It sounds clunky but it’s incredibly awesome. For people who are used to working in vi, it takes about 10 seconds to realize just how cool it is and after that…

By default in the bash shell, the user is in “insert” mode, typing commands and causing them to be executed by pressing return. To exit “insert” mode, the user presses the `Esc` key to enter “command” mode, where keystrokes are interpreted as vi commands. When done editing the command line, the user presses the `Esc` key once again to exit “command” mode and enter “insert” mode.

The secret to getting vi mode working with bash is to create a file called .inputrc in the home directory and put these lines in it:

`set editing-mode vi
set keymap vi-insert`

Then start up a new shell where these settings will take effect.

That’s it! For those interested in more discussion of this topic, check out [this informative thread on stackoverflow](https://stackoverflow.com/questions/46161918/bash-vim-mode-instead-of-vi-mode) and check out our series of [Bash tutorials](https://opensource.net/?s=bash).