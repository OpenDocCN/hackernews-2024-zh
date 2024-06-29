<!--yml

category: 未分类

date: 2024-05-27 14:46:03

-->

# 如何失去对你的 shell 控制权 - 作者：Thorsten Ball

> 来源：[https://registerspill.thorstenball.com/p/how-to-lose-control-of-your-shell](https://registerspill.thorstenball.com/p/how-to-lose-control-of-your-shell)

几周前，我正在 Zed 中开发语言服务器支持，试图让 Zed 能够检测给定语言服务器二进制文件（如 `gopls`）是否已经存在于 `$PATH` 中。如果是，它应该使用该二进制文件，而不是下载一个新的二进制文件。

挑战：`$PATH` 经常被工具如 `direnv`、`asdf`、`mise` 等动态修改，这些工具允许你在特定文件夹中设置特定的 `$PATH`。（为什么这些工具这么做呢？因为它让你有能力在你进入 `my-cool-project` 时，比如说在 `$PATH` 前面加上 `./my_custom_binaries`。）所以我们不能仅仅使用 Zed 进程关联的 `$PATH`，我们需要在你 `cd` 到项目目录时的 `$PATH`。

简单，我想。只需启动一个 `$SHELL`，`cd` 到项目目录以触发 `direnv` 和其他操作，运行 `env`，保存环境变量，提取出 `$PATH`，在其中找到二进制文件。

就是这么简单。这里是部分代码，启动 `$SHELL`，`cd` 并获取 `env` 的部分：

```
`fn load_shell_environment(dir: &Path) -> Result<HashMap<String, String>> {
    // Get the $SHELL
    let shell = std::env::var("SHELL")?;

    // Construct the command we want the $SHELL to execute
    let command = format!("cd {:?}; /usr/bin/env -0;", dir);

    // Launch the $SHELL as an interactive shell (so the user's rc files are used)
    // and execute `command`:
    let output = std::process::Command::new(&shell)
        .args(["-i", "-c", &command])
        .output()?;

    // [... check exit code, get stdout, turn stdout into HashMap, etc. ...]
}`
```

除了一件事：在我的终端中启动一个执行此函数的 Zed 实例后，我再也无法通过按 `Ctrl-C` 来终止 Zed 了。

什么？

我可以用 `^C` 塞满终端，却什么也没有发生。绝望的 `^C` 行和行，从未听到它们自己的回声。

如何？为什么？…… 什么？

说了“什么？” 20 次，按了 `Ctrl-C` 更多次后，我向 [Piotr](https://zed.dev/team#piotr-osiewicz) 寻求帮助，因为我对 Rust 如何生成进程不是百分之百的信心，而他是 Rust 的高手。我知道的是，在 `std::process::Command` 内必然存在 `fork` 和 `exec` 系统调用，但我不确定 Rust 是否会在信号处理程序上做些聪明的事情，或者是否设置了默认的信号处理程序来干扰 `Ctrl-C`。因为 `Ctrl-C` *应该* 导致发送中断信号到进程，从而导致其终止，但显然这种方式停止工作了。

我们开始探讨各种事情，测试各种可能的假设，尽管它们可能有些不可思议。

我们确定了吗，shell 真的不再运行了吗？是的，因为上面的 `.output()` 仅在命令运行完成后才返回。

这和 `cd` 有关吗？ `direnv` 或 `asdf` 或其他工具是否触发了一些控制终端的钩子？不，结果表明，即使我们只运行 `/usr/bin/env -0;` 而没有 `cd`，它也会控制住终端。

所以是我们传递给 `env` 的 `-0` 引起了这个问题吗？不应该，显然不是，因为那只是格式化。但：绝望的时候会引发绝望的调试尝试。所以我们尝试了一下，结果不是 `-0` 的问题。

等等，是 `env` 吗？它是否对我的终端做了一些奇怪的事情？哼。

所以我们把 `command` 从

```
`let command = format!("/usr/bin/env;");`
```

to

```
`let command = format!("echo lol");`
```

... 猜猜看？`Ctrl-c`又恢复工作了。

什么？

好的，再试一次。如果我们*两者*都做呢？

```
`let command = format!("/usr/bin/env; echo lol");`
```

那个*也*奏效了。什么！

好了，等一下... 我的直觉告诉我点什么。`/usr/bin/env`不是一个内置的shell命令，对吧？但`echo`是。这是一个线索吗？

让我们试试这个：

```
`let command = format!("ls");`
```

那个老牌的`ls`。可能是我一生中运行得最多的命令。每当我需要它的时候，无论我访问哪台机器，我都会立即运行`ls`来确认它能正常工作。我可以用我的生命来信赖`ls`。

然而：在那个子shell中运行`ls`后，`Ctrl-c`停止工作了。连你也，`ls`吗？

下一个假设：是Zed里面的问题吗？我们需要设置一些信号处理程序吗？让我们找出来。我们把这个函数复制到一个新的、最简单的Rust项目中运行，结果... 它也复现了。在那个项目中`Ctrl-c`也停止工作了。

好了，难道是Rust的问题吗？我用Go重写了这个函数，在Go中`Ctrl-c`也失去了控制。

到这一步，我们已经花了将近2个小时，还是解决不了。但我们确实找到了一个解决方法：

```
`let command = format!("/usr/bin/env; exit 0;");`
```

`exit`在所有不同的shell中都是内置命令，所以可以安全地运行并且修复了问题。好的，公平。我们在那行上方加了一行非常重要的注释，让下一个遇到这个问题的人知道`exit 0`现在是有负载的，然后继续。

但这个难题困扰着我。我问了一些[shell爱好者](https://twitter.com/thorstenball/status/1760693619164393907)，是否知道发生了什么，但没有人有准备好的答案。所以在早晨，我开始调查。

我在一个[存储库](https://github.com/mrnugget/strange-subshell)中设置了一个小的Rust程序，重现了这个问题：它生成一个shell进程，等待它退出，然后空闲5秒，这样我就可以测试`Ctrl-c`是否仍然起作用。这是一场追踪之旅。

第一个重大的灵光乍现时刻是当我意识到我不必通过`Ctrl-c`发送信号：我可以使用`kill`命令。然后，啊，问题并不在于信号处理出了问题！当我使用`kill -INT`时，信号传递并且进程停止了。问题并不是我的进程不再响应信号，而是在启动shell进程后，`Ctrl-c`没有传递正确的信号。

下一次尝试：启动shell后，终端状态是否出错？嗯，应该是终端状态的问题。[推特回复中的某人](https://twitter.com/hugelgupf/status/1760704715111944347)指向了`stty`，它可以让你设置终端设备的选项，如波特率（是的）和其他选项。我修改了我的程序，在shell进程之前和之后运行了`stty -a`。没用：输出没有变化。

绝望之际，我还使用了[Ghostty的终端检查工具](https://mitchellh.com/writing/ghostty-devlog-005)来查看终端中是否有某些状态改变导致`Ctrl-c`失效。但也没有运气。

经过与ChatGPT反复讨论数天后（我[上次写到过](https://registerspill.thorstenball.com/p/how-i-use-ai)），它终于给了我一个线索：

> 生成的 shell 继承了终端（TTY）控制权，并且因为是交互式 shell（-i 标志），它将自己设定为终端的前台进程组组长。这改变了信号的处理方式，特别是由 Ctrl-C 生成的 SIGINT 信号的处理。

哦。前台进程组组长。有意思。嗯。这里是《Unix 环境高级编程》（APUE），我在写这篇文章时找出来的，关于进程组的内容：

> 进程组是一个或多个进程的集合，通常与同一作业相关联（作业控制在第 9.8 节中讨论），它们可以从同一终端接收信号。每个进程组都有一个唯一的进程组 ID。进程组 ID 类似于进程 ID：它们是正整数，并且可以存储在 `pid_t` 数据类型中。函数 `getpgrp` 返回调用进程的进程组 ID。

重要的部分是：“它们可以从同一终端接收信号。”

已经有一段时间没有在实体书中查找信息了。图片：《Unix 环境高级编程》。一本绝佳的书。

APUE 给出了更多线索：

> 进程组领导者可以创建一个进程组，创建组内的进程，然后终止。

那么就是这样发生的吗？shell 生成，声称它是进程组领导者，当它不运行内建命令时退出，然后不恢复之前的进程组领导者吗？

我觉得我离真相越来越近了。所以我继续询问 ChatGPT 如何确认这一点，并它引导我到了 `tcgetprg`：

> 函数 `tcgetpgrp()` 返回与调用进程的控制终端（fd 必须是调用进程的控制终端）关联的前台进程组的进程组 ID。

好了，现在我们开始谈论，这听起来可能会给我们带来一些线索。我让 ChatGPT 为我生成了一些 Rust 代码来调用 `tcgetpgrp`：

```
`fn get_process_group_id(fd: i32) -> io::Result<libc::pid_t> {
    let pgid = unsafe { libc::tcgetpgrp(fd) };
    if pgid == -1 {
        Err(io::Error::last_os_error())
    } else {
        Ok(pgid)
    }
}`
```

我将它插入到我的程序中，以便在 `$SHELL` 进程运行之前和之后打印与 STDIN（文件描述符 `0`）关联的进程组 ID。这是它打印出来的内容：

```
`process group before: 54530
shell exited with status: exit status: 0
process group after: 54571`
```

哦，你好！这看起来确实像是凶器。但我怎样确认它 *确实* 是导致我的 `Ctrl-c` 失效的东西？有没有办法我可以阻止 shell 接管进程组领导者的角色？ChatGPT 说我可以在 `std::process::Command` 的 `pre_exec` 钩子上使用，将 shell 进程置于一个新的、独立的进程会话中，这将把它置于一个新的进程组，进而意味着它无法成为与 STDIN 关联的进程组的进程组领导者。就像这样：

```
`let cmd = std::process::Command::new("/bin/zsh");
cmd.args(["-i", "-c", "/usr/bin/env"]);

// Set a hook that will be executed right after `fork`, but before `exec`:
unsafe {
    cmd.pre_exec(|| {
        if libc::setsid() == -1 {
            return Err(std::io::Error::last_os_error());
        }
        Ok(())
    });
}

// Run the command
let output = cmd.output().unwrap();`
```

就在这里，在中间：`setsid`。这是在我们用 `fork` 创建新进程后但在将该进程转换为 `$SHELL` 之前调用的。

APUE 讨论进程调用 `setsid` 时发生的情况：

> 1.  这个进程成为了新会话的会话领导者。[…]
> 1.  
> 1.  进程成为了一个新进程组的进程组领导者。[…]
> 1.  
> 1.  进程没有控制终端。[…] 如果进程在调用 `setsid` 前有一个控制终端，那么这种关联会被断开。

这很有道理。通过调用 `setsid`，可以打破新生成的 shell 进程与终端的任何关联，这有助于我确认 shell 操纵进程组领导者是否是问题所在。

然后 — 砰！烟花！大声的噪音！一个小孩说：“ta-da！” — 通过 `pre_exec` 钩子，程序打印了这个：

```
`process group before: 54530
shell exited with status: exit status: 0
process group after: 54530`
```

并且 `Ctrl-C` 仍然有效！

前台进程组 ID *就是* 凶器。这时候，*发生了什么* 清楚了：生成的 shell 接管了终端，通过设置前台进程组 ID，这意味着由 `Ctrl-C` 导致的信号被发送到 shell 进程。但是，如果 shell 作为其最后一个命令运行非内建命令，它不会自我清理，其进程 ID 保持与终端关联，导致所有我们的 `Ctrl-C` 都消失无踪。

有了 *什么？* 接下来的问题是：为什么？

为什么 ZSH（这是发生在我身上的 shell）在运行非内建命令时不重置前台进程组领导者？

在我的 Linux 机器上，我运行了 `strace -f` 来查看我的进程以及其子进程（包括生成的 shell）都在进行哪些系统调用。我能弄清楚的是：

当以 `-c` 运行 `zsh` 并且传递的命令中最后一个命令是非内建命令，比如 `ls` 或 `env`，那么 ZSH 就会 `execve` 进入到该最后的进程中。这意味着：它不会创建一个子进程来运行 `ls`。不，它会将自己变成那个命令。这意味着在 `zsh -c 'echo lol; ls'` 中运行 `ls` 时，`zsh` 进程已经消失并变成了 `ls`，没有人留下来重新设置前台进程组领导者。

但是当你运行 `zsh -c '/usr/bin/env; echo lol'`，即：先是非内建，然后是内建命令，那么 ZSH 不会消失。它会 `fork` 和 `exec` `/usr/bin/env`，然后执行 `echo lol`，在某个地方，清理前台进程组领导者。

现在，请听我说。我希望我能继续并以 “… 这就是为什么 ZSH 那样做！” 结束，然后有人终于通过 PayPal 给我 100 美元并留言 “感谢你的通讯”，但我必须让你失望了。

我不知道 `ZSH` 为什么以及如何做这些事情。我克隆了仓库，编译了它，试图从源码运行它，但不知何故失败了，而且 `cmake` 真是个大问题，还有文件夹的名字像 `Src` 和 `Doc`，谁会大写文件夹名字的第一个字母，还有一个 `./configure` 你必须运行，然后确保它不使用你的系统库… 你看，这些 shell 调查的事情并不容易，我放弃了，抱歉。

我*确实* [找到了](https://sourcegraph.com/github.com/zsh-users/zsh@fa9b3ad5977ede0a4635cd86276dd0f0c2f6f03e/-/blob/Src/jobs.c?L3151-3169)，ZSH确实主动设置作业控制的进程组ID。而且它也[记住并重置了原始ID](https://sourcegraph.com/github.com/zsh-users/zsh@fa9b3ad5977ede0a4635cd86276dd0f0c2f6f03e/-/blob/Src/jobs.c?L3213-3227)。但是当我看到[ZSH中这部分内容](https://sourcegraph.com/github.com/zsh-users/zsh@fa9b3ad5977ede0a4635cd86276dd0f0c2f6f03e/-/blob/Src/exec.c?L1114-1153)处理作业控制的时候，意识到我做这些工作是不会得到报酬的。

我期待着您解释的来信。
