<!--yml
category: 未分类
date: 2024-05-27 14:46:03
-->

# How to Lose Control of your Shell - by Thorsten Ball

> 来源：[https://registerspill.thorstenball.com/p/how-to-lose-control-of-your-shell](https://registerspill.thorstenball.com/p/how-to-lose-control-of-your-shell)

A few weeks ago I was hacking on language server support in Zed, trying to get Zed to detect when a given language server binary, such as `gopls`, is already present in `$PATH`. If so, it should use that instead of downloading a new binary.

The challenge: `$PATH` is often dynamically modified by tools such as `direnv`, `asdf`, `mise` and others, which allow you to set a specific `$PATH` in a given folder. (Why do these tools do that? Because it gives you the ability to, say, prepend `./my_custom_binaries` to `$PATH` when you’re in `my-cool-project`.) So we can’t just use the `$PATH` associated with the Zed process, we need the `$PATH` as it is when you `cd` into your project directory.

Easy, I thought. Just launch a `$SHELL`, `cd` into the project to trigger `direnv` and whathaveyou, run `env`, store the environment, pick out `$PATH`, find binaries in there.

And easy it was. Here’s some of the code, the part that launches `$SHELL`, `cd`s and gets the `env`:

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

Except for one thing: after starting a Zed instance in my terminal that executed this function, I could no longer kill Zed by hitting `Ctrl-C`.

What?

I could spam the terminal with `^C` and nothing happened. Lines and lines of desperate `^C`s that never hear their own echo.

How? Why? … What?

After saying “What?” 20 times and hitting `Ctrl-c` even more, I asked [Piotr](https://zed.dev/team#piotr-osiewicz) for help, because I wasn’t 100% confident in how Rust spawns processes and he’s a Rust wizard. What I did know was that there would have to be `fork` and `exec` syscalls somewhere inside `std::process::Command` but I wasn’t sure whether Rust doesn’t do something clever with the signal handlers or has default signal handlers setup that mess with `Ctrl-c`. Because `Ctrl-c`  *should* result in an [interrupt signal being sent](https://en.wikipedia.org/wiki/Signal_(IPC)#Sending_signals) to the processes which should cause it to terminate, but clearly that stopped working.

We started to poke at all kinds of things to test all kinds of hypotheses, as outlandish as they might be.

Are we *sure* that the shell is not running anymore? Yes, we are, because `.output()` up there only returns once the command has finished running.

Is this about `cd`? Do `direnv` or `asdf` or other tools fire some hooks that take control of the terminal? No, turns out when we just run `/usr/bin/env -0;` without `cd` it also takes control over the shell.

So is it the `-0` that we pass to `env`? It shouldn’t be, clearly, because that’s just formatting. But: desperate times breed desparate debugging attempts. So we tried it and it wasn’t `-0` either.

Wait, is it `env`? Does it do something weird with my terminal? Huh.

So we changed the `command` from

```
`let command = format!("/usr/bin/env;");`
```

to

```
`let command = format!("echo lol");`
```

… and guess what? `Ctrl-c` worked again.

What?

Okay, another attempt. What if we do *both*?

```
`let command = format!("/usr/bin/env; echo lol");`
```

That *also* worked. WHAT!

Okay, wait a second… my gut is telling me something. `/usr/bin/env` isn’t a shell built-in, is it? But `echo` is. Is that a clue?

Let’s try this one:

```
`let command = format!("ls");`
```

Good old `ls`. Probably the command I’ve ran the most in my life. It’s always there when I need it and on every machine I gain access to I immediately run `ls` just to see that it works. I’d trust `ls` with my life.

And yet: after running `ls` in that subshell, `Ctrl-c` stopped working. Et tu, `ls`?

Next hypothesis: is it something in Zed? Do we setup some signal handlers? Let’s find out. We copied the function to a new, bare-bones Rust project, ran it and… it reproduced. `Ctrl-c` stopped working in that project too.

Okay, is it Rust then? I rewrote the function to Go and in Go too `Ctrl-c` lost control.

At this point we had spent nearly 2 hours on this and couldn’t figure it out. But we did have a workaround:

```
`let command = format!("/usr/bin/env; exit 0;");`
```

`exit` is a built-in in all the different shells, so it’s safe to run and it fixes the problem. Okay, fair enough. We slapped one hell of a comment above that line to let the next person to come along know that the `exit 0` is now load-bearing and moved on.

But this puzzle got to me. I asked [fellow shell-nerds](https://twitter.com/thorstenball/status/1760693619164393907) whether they know what’s happening but no one had an answer ready. So in my mornings I started to investigate.

I setup a [repository](https://github.com/mrnugget/strange-subshell) in which a small Rust program reproduced the problem: it spawns a shell process, waits for it to exit, then idles for 5 seconds so I can test whether `Ctrl-c` still works. The hunt was on.

The first big light bulb moment came when I realized that I don’t have to send a signal via `Ctrl-c`: I can use the `kill` command. And, alas, it’s *not* the signal handling that’s borked! When I used `kill -INT` the signal arrived and the process stopped. It’s not that my process doesn’t react to signals anymore, but rather that `Ctrl-c` doesn’t deliver the right signals after launching the shell process.

Next attempt: is the terminal stated borked after launching the shell? Okay, so something about the terminal state. [Someone in the tweet replies](https://twitter.com/hugelgupf/status/1760704715111944347) did point me to `stty`, which lets you set options on your terminal device, such as the baud rate (yes) and other things. I modified my program to run `stty -a` before and after the shell process. No luck: no changes in the output.

Desperate, I also used [Ghostty’s terminal inspector](https://mitchellh.com/writing/ghostty-devlog-005) to see whether some state changes in the terminal that results in `Ctl-c` going up in smoke. But no luck there either.

After days of going back and forth on this with ChatGPT (which I [wrote about the last time](https://registerspill.thorstenball.com/p/how-i-use-ai)) it finally gave me a clue:

> The spawned shell inherits the terminal (TTY) control, and since it’s an interactive shell (-i flag), it sets itself as the foreground process group leader for the terminal. This changes how signals, especially SIGINT generated by Ctrl-C, are handled.

Huh. Foreground process group leader. Interesting. Hmmm. Here’s what [Advanced Programming in the Unix Environment (APUE)](https://en.wikipedia.org/wiki/Advanced_Programming_in_the_Unix_Environment), which I pulled out today while writing this, says on process groups:

> A process group is a collection of one or more processes, usually associated with the same job (job control is discussed in Section 9.8), that can receive signals from the same terminal. Each process group has a unique process group ID. Process group IDs are similar to process IDs: they are positive integers and can be stored in a `pid_t` data type. The function `getpgrp` returns the process group ID of the calling process.

The important part: “that can receive signals from the same terminal.”

It’s been a while since I last looked up something in a physical book. Pictured: Advanced Programming in the Unix Environment. A fantastic book.

APUE has more clues:

> It is possible for a process group leader to create a process group, create processes in the group, and then terminate.

So is that what happens? The shell spawns, claims it’s the process group leader when it doesn’t run a built-in command, exits, and then doesn’t restore the previous process group leader?

It felt like I was getting closer. So I kept asking ChatGPT how to confirm this and it led me to `tcgetprg`:

> The function `tcgetpgrp()` returns the process group ID of the foreground process group on the terminal associated to `fd`, which must be the controlling terminal of the calling process.

Okay, now we’re talking, this sounds like it could lead us somewhere. I asked ChatGPT to generate me some Rust code for that `tcgetpgrp` call:

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

I plugged that into my program so it would print the process group ID associated with STDIN (file descriptor `0`) before and after the `$SHELL` process has run. This is what it printed:

```
`process group before: 54530
shell exited with status: exit status: 0
process group after: 54571`
```

Well, hello there! This certainly looks like the murder weapon. How can I confirm that it *is* what kills my `Ctrl-c` though? Is there some way I could stop the shell from taking over as process group leader? ChatGPT said that I could use the `pre_exec` hook on `std::process::Command` to put the shell process in a new, separate process session, which will put it in a new process group, which in turn means it won’t be able to become the process group leader of the group associated with STDIN. Like this:

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

Right there, in the middle: `setsid`. That’s called right after we create a new process with `fork` but before that process is turned into `$SHELL`.

APUE on what happens when a process calls `setsid`:

> 1.  The process becomes the session leader of this new session. […]
>     
>     
> 2.  The process becomes the process group leader of a new process group. […]
>     
>     
> 3.  The process has no controlling terminal. […] If the process had a controlling terminal before calling `setsid`, that association is broken.`

That makes sense. By calling `setsid` it would break any association the newly-spawned shell process has with the terminal and that could help me confirm whether the shell mucking with the process groups leader is the problem.

And — boom! fireworks! loud noises! a small child saying: “ta-da!” — with the `pre_exec` hook this is what the program printed:

```
`process group before: 54530
shell exited with status: exit status: 0
process group after: 54530`
```

And `Ctrl-C` still worked!

The foreground process group ID *is* the murder weapon. At this point it was clear *what* happens: the shell that’s spawned takes control of the terminal, by setting the foreground process group ID, which means the signal resulting from `Ctrl-C` is sent to the shell process. But if the shell runs a non-built-in command as its last command, it doesn’t clean up after itself and its process ID stays associated with the terminal, leading to all of our `Ctrl-C`s ending up in the void.

With that *What?* the next question is: why?

Why does ZSH (the shell with which this happened for me) not reset the foreground process group leader when it runs a non-built in command?

On my Linux machine I ran `strace -f` to see which syscalls my process and, more importantly, its child processes (including the spawned shell) were making. What I could figure out was this:

When `zsh` is run with `-c` and the last command in that passed command is a non-built-in, such as `ls` or `env`, then ZSH `execve`s into that last process. Meaning: it doesn’t create a child process to run `ls`. No, instead it turns itself into that command. That means at the point in time when `ls` is run in `zsh -c 'echo lol; ls'` the `zsh` process is gone and turned into `ls` and there’s no one left to reset the foreground process group leader.

But when you run `zsh -c '/usr/bin/env; echo lol'`, i.e.: first non-built-in, then built-in, then ZSH doesn’t disappear. It `forks` and `execs`  `/usr/bin/env` and then executes the `echo lol` and, somewhere in there, cleans up the foreground process group leader.

Now, listen. I wish I could continue here and end with “… and *this* is why ZSH does it that way!” and someone would finally PayPal me $100 with the message “thanks for your newsletter”, but I have to disappoint you.

I don’t know how and why exactly ZSH does what it does. I cloned the repo, I compiled it, I tried to run it from source, but somehow failed and man `cmake` is a lot and also the folders have names like `Src` and `Doc` and who the hell capitalizes the first letter in a folder name and there’s also a `./configure` you have to run and then you need to make sure it doesn’t use your system library and… You see this shell investigation stuff isn’t easy and I gave up, sorry.

What I *did*  [find though](https://sourcegraph.com/github.com/zsh-users/zsh@fa9b3ad5977ede0a4635cd86276dd0f0c2f6f03e/-/blob/Src/jobs.c?L3151-3169) is that ZSH does actively set the process group id for job control. And it also [remembers the original one and resets it](https://sourcegraph.com/github.com/zsh-users/zsh@fa9b3ad5977ede0a4635cd86276dd0f0c2f6f03e/-/blob/Src/jobs.c?L3213-3227). But I gave up when I saw [this part here](https://sourcegraph.com/github.com/zsh-users/zsh@fa9b3ad5977ede0a4635cd86276dd0f0c2f6f03e/-/blob/Src/exec.c?L1114-1153) that does job control stuff in ZSH and realized that I’m not getting paid for this.

I await your letters with the explanation.