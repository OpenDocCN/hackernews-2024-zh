<!--yml
category: 未分类
date: 2024-05-27 14:59:57
-->

# Use Midnight Commander like a pro – Igor Klimer’s personal site

> 来源：[https://klimer.eu/2015/05/01/use-midnight-commander-like-a-pro/](https://klimer.eu/2015/05/01/use-midnight-commander-like-a-pro/)

If you’ve used an `*nix` system, at some point you’ve stumbled upon [Midnight Commander](http://www.midnight-commander.org/), a file manager based on the venerable Norton Commander. You’re probably familiar with the basic operations (`F5` for copying, `F6` for moving, `F8` for deleting, etc.) and how to switch panels (ummm, the `Tab` key). But `mc` offers so much more than that. This article aims to show all the useful (YMMV) shortcuts and functionalities that are often overlooked. Most of them can be accessed using the menu (`F9`), but who has the time to do that?

Before we get started, let’s establish some facts. This article was written and tested on the following software:

*   Midnight Commander 4.8.13
*   GNU bash 4.2.53

Oh, and make sure you’re running a modern and UTF-8 friendly terminal – for example, rxvt-unicode.

# Hold your horses

There’s actually one thing I’d recommend doing before you run `mc`. `mc` has the ability to exit to its current directory. Meaning, you can navigate the filesystem using `mc` (sometimes it’s easier than `cd`ing into that one directory buried deep down *somewhere*) and when you quit `mc` (`F10`), your shell will automagically `cd` to that directory. This is done thanks to the `mc-wrapper` script that should be bundled with your installation of `mc`. The exact location is dependent on your distribution – in mine (Gentoo) it’s `/usr/libexec/mc/`, in Ubuntu supposedly it’s in `/usr/share/mc/bin/`. Once found, modify your `~/.bashrc`:

```
alias mc='. /usr/libexec/mc/mc-wrapper.sh'

```

Restart your shell, launch `mc`, change to another directory, exit and your shell should be set to that new directory.

# Selecting files

*   `Insert` (`Ctrl + t` alternatively) – select files (for example, for copying, moving or deleting).
*   `+` – select files based on a pattern.
*   `\` –*un*select files based on a pattern.
*   `*` – reverse selection. If nothing was selected, all files will get selected.

# Accessing the shell

*   There’s a shell awaiting your command at the bottom of the screen – just start typing (when no other command dialog is open, of course).
*   Since `Tab` is bound to switching panels (or moving the focus in dialogs), you have to use `Esc Tab` to use autocompletion. Hit it twice to get all the possible completions (just like in a shell). This works in dialogs too.
*   If you want inspect the output of the command, do some input or just prefer a bigger console, no need to quit `mc`. Just hit `Ctrl + o` – the effect will be similar to putting `mc` in the background but with a nice perk. Your current working directory from `mc` will be passed on to the shell… and vice versa! Hit `Ctrl + o` again to return to `mc`.
*   `Ctrl + Enter` or `Alt + Enter` – copy the currently selected file’s name to the shell.
*   `Ctrl + Shift + Enter` – same as above, but the full path is copied.

# Internal viewer (`F3`) and editor (`F4`)

*   The internal viewer has many built in modes for “previewing” the content of the file. Try “viewing” a binary, an archive, a DOC document or an image. In some cases, external programs are needed in order for this “previewing” to work.
*   If you want to preview the “raw” contents of the file, hit `Shift + F3`.
*   While the internal viewer and editor are powerful, sometimes you want to use your preferred software (*cough*vim*cough*). You can do so by setting the `PAGER` (for viewer) and `EDITOR` (for editor) variables (for example, in your `~/.bashrc` file). Then toggle the `Options -> Configuration -> Use interal edit/view` option (access the top menu by pressing `F9`).

# Panels

*   `Alt + ,` – switch `mc`’s layout from left-right to top-bottom. Mind = blown. Useful for operating on files with long names.
*   `Alt + t` – switch the panel’s listing mode in a loop: default, brief, long, user-defined. “long” is especially useful, because it maximises one panel so that it takes full width of the window and longer filenames fit on screen.
*   `Alt + i` – synchronize the active panel with the other panel. That is, show the current directory in the other panel.
*   `Ctrl + u` – swap panels.
*   `Alt + o` – if the currently selected file is a directory, load that directory on the other panel and move the selection to the next file. If the currently selected file is not a directory, load the parent directory on the other panel and moves the selection to the next file. This is useful for quick checking the contents of a list of directories.
*   `Ctrl + PgUp` (or just left arrow, if you’ve enabled `Lynx-like motion`, see later) – move to the parent directory.
*   `Alt + Shift + h` – show the directory history. Might be easier to navigate than going back one entry at a time.
*   `Alt + y` – move to the previous directory in history.
*   `Alt + u` – move to the next directory in history.

# Searching files

*   `Alt + ?` – shows the full Find dialog.
*   `Alt + s` or `Ctrl + s` – quick search mode. Start typing and the selection will move to the first matching file. Press the shortcut again to jump to another match. Use wildcards (`*`, `?`) for easier matching.

# Common actions

*   `Ctrl + Space` – calculate the size of the selected directories. Press this shortcut when the selection is on `..` to calculate the size of all the directories in the current directory.
*   `Ctrl + x s` (that is press `Ctrl + x`, [let it go](https://www.youtube.com/watch?v=L0MK7qz13bU) and then press `s`) – create a symbolic link (change `s` to `l` for a hardlink). I find it very useful and intuitive – the link will, of course, be created in the other panel. You can change it’s destination and name, like with any other file operation.
*   `Ctrl + x c` – open the `chmod` dialog.
*   `Ctrl + x o` – open the `chown` dialog.

# Virtual File System (VFS)

`mc` has a concept known as Virtual File System. Try “entering” an archive (`*.tar.gz`, `*.rpm` or even `*.jar`) – you’ll be able to browse the contents of the archive like a normal folder, without unpacking it first. You extract selected files from the archive by just copying them to the other panel. Bonus points: try “entering” a… `*.patch` file.

This concept is even more powerful when you realize that remote locations can be viewed the same way. A quick way to browse an FTP location is to just `cd` to it: `cd ftp://mirrors.tera-byte.com/pub/gentoo` (first Gentoo FTP mirror I found). You’ll be able to interact with files as you normally do. To exit this remote location, `cd` to a local directory. Just typing `cd` will suffice as it will take you to your home directory.

VFS works for SFTP and Samba shares too. Check the manpages for more information on how to specify user/pass, etc.

# Useful options

*   **Configuration**
    *   `Verbose operation` and `Compute totals` – so that operations like copy/move have a more detailed progress dialogs.
*   **Layout**
    *   `Equal split` – uncheck to define your own ratio for panels. Maybe you prefer one panel bigger than the other? Useful especially if you keep one of the panels in tree mode (or maybe info/quick view, too).
    *   Uncheck `Hintbar visible` – one more line available, one less line of noise.
*   **Panel options**
    *   `Show backup files` and `Show hidden files` – I keep both enabled, as I often work with configuration files, etc.
    *   `Lynx-like motion` – mentioned above, makes left arrow go to parent directory, while the right arrow enters the directory under selection. Faster than `Home`, `Enter`, `Home`, `Enter`, etc. This options is quite smart, that is if the shell command line is not empty, the arrows work as usual and allow moving the cursor in the command line.
    *   `File highlight` -> `File types` is useful, as it uses a different color for example for executable files. `Permissions`, for me, is not that useful, but I can definitely see it’s use, for example, for sysadmins.
*   **Appearance**
    *   Only one option here, `Skins`. You can check out different skins shipped with `mc` – just select one from the list. I prefer `gotar`, because it plays well with my [solarized](http://ethanschoonover.com/solarized) terminal colors.
    *   Useful tip – set up a different skin when logged in as the `root` user. It’ll be easier to differentiate between root’s and normal user’s session, when you’re swapping between them (as is often the case).

# Bonus assignments

*   Define your own listing mode (`Right/Left` -> `Listing mode...` -> `User defined`). Hit `F1` to see available columns and options.
*   Play around in tree mode: `Right/Left` -> `Tree` or `Command` -> `Directory tree`.
*   Compare directories (`Ctrl + x d`)
*   Fill up the directory hotlist (`Ctrl + \`)

Well, that was a lot to take in. Of course, this list is not complete (that’s what `man mc` is there for), but I’ve selected the commands and functionalities that are the most useful to *me*. Embrace the ones you find useful, forget the rest and learn about the other ones I’ve missed!

Happy browsing!

**Published** May 1, 2015January 1, 2021