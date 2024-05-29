<!--yml
category: 未分类
date: 2024-05-27 14:49:51
-->

# Laurence Tratt: Faster Shell Startup With Shell Switching

> 来源：[https://tratt.net/laurie/blog/2024/faster_shell_startup_with_shell_switching.html](https://tratt.net/laurie/blog/2024/faster_shell_startup_with_shell_switching.html)

A few days ago Thorsten Ball wrote a post exhorting Unix users to [optimise their shell’s startup time](https://registerspill.thorstenball.com/p/how-fast-is-your-shell). It’s excellent advice and well worth following.

The Unix shell has two major use cases: as an interactive prompt (what we often call “the command line”); and as a “scripting” or “non-interactive command” language. We normally pick one shell (bash, fish, zsh, etc.) and use it for both use cases.

However, we can use different shells for each use case. People don’t normally bother doing so because there is little functional utility in doing so. There is, however, a small performance reason for doing so, which I’m going to look at in this post. I’m going to call the technique I describe in this post “shell switching” since I’m not aware that it has an existing name.

## Minimum Shell Overhead

Let’s start by looking at the absolute minimum cost of starting and stopping a non-interactive Unix shell when there’s no user configuration . If I create a fresh user account with no configuration (other than an empty `~/.zshrc` to stop zsh asking me to do setup) bash, fish, and zsh take the following times to run on my OpenBSD laptop:

```
$ hyperfine --shell=none "bash -c exit" Benchmark 1: bash -c exit
 Time (mean ± σ):       5.1 ms ±   0.2 ms    [User: 0.7 ms, System: 3.5 ms] Range (min … max):     4.6 ms …   7.3 ms    472 runs  $ hyperfine --shell=none "fish -c exit" Benchmark 1: fish -c exit
 Time (mean ± σ):      14.7 ms ±   0.6 ms    [User: 6.3 ms, System: 7.4 ms] Range (min … max):    13.3 ms …  17.8 ms    184 runs  $ hyperfine --shell=none "zsh -c exit" Benchmark 1: zsh -c exit
 Time (mean ± σ):       7.5 ms ±   3.1 ms    [User: 1.3 ms, System: 4.2 ms] Range (min … max):     5.9 ms …  21.1 ms    364 runs 
```

Benchmarking numbers from laptops are bad enough, but I’m also using a weird operating system — those numbers are clearly untrustworthy. Here’s what happens if I run the same benchmarks on a fresh user account on a Debian server:

```
$ hyperfine --shell=none "bash -c exit" Benchmark 1: bash -c exit
 Time (mean ± σ):       3.3 ms ±   0.1 ms    [User: 1.5 ms, System: 1.4 ms] Range (min … max):     3.1 ms …   3.9 ms    828 runs   $ hyperfine --shell=none "fish -c exit" Benchmark 1: fish -c exit
 Time (mean ± σ):      14.0 ms ±   0.9 ms    [User: 8.8 ms, System: 4.8 ms] Range (min … max):     8.7 ms …  15.9 ms    206 runs  $ hyperfine --shell=none "zsh -c exit" Benchmark 1: zsh -c exit
 Time (mean ± σ):       4.2 ms ±   0.2 ms    [User: 1.5 ms, System: 2.3 ms] Range (min … max):     3.6 ms …   5.0 ms    718 runs 
```

The numbers between the two machines tell a fairly consistent story: bash and zsh are fairly fast, but fish is several times slower. This seems bad for me, because I use fish on my local machines, but should I really worry about a mere 3-15ms minimum overhead? Thorsten argues that I should care for interactive prompts, but personally I care just as much about “non-interactive command” execution.

I run lots of programs which themselves run lots of commands. This can range from me running external commands in my editor to running commands on a remote machine (with `ssh example.com "cmd arg1 ... argn"`). Although we often overlook this, the commands we specify are generally passed to a shell to run them with `shell -c cmd "arg1 ... argn"` — in other words these commands are one-line shell scripts. The overhead of the shell is a cost I am forced to bear each time I run such a command.

An obvious question is: which shell is used to run sub-commands? Surprisingly, there is no simple answer to this, and different programs use different shells to run sub-commands. Some use the contents of the `$SHELL` environment variable; some use the shell set in the user database (i.e. `/etc/passwd` and friends); and some use a hard-coded shell (e.g. `/bin/sh`). In most cases, `$SHELL` and the shell in the user database are the same, often a “big” shell like bash, fish, or zsh. In many of the cases I care about, my non-interactive commands are run with one of these big shells, so the overhead of them is something I care about.

In practice, the overhead I experience when running non-interactive commands is much higher than the bare minimum would suggest. As luck would have it, I have fairly comparable fish and zsh configurations (I migrated from the latter to the former a year or two back). Executing a command as my normal user account shows what the overhead of the shell plus my configuration is:

```
$ hyperfine --shell=none "fish -c exit" Benchmark 1: fish -c exit
 Time (mean ± σ):      88.2 ms ±   3.6 ms    [User: 38.1 ms, System: 92.9 ms] Range (min … max):    83.1 ms …  99.5 ms    31 runs  $ hyperfine --shell=none "zsh -c exit" Benchmark 1: zsh -c exit
 Time (mean ± σ):      49.5 ms ±   1.5 ms    [User: 14.7 ms, System: 75.4 ms] Range (min … max):    46.9 ms …  53.0 ms    57 runs 
```

Not only are those times 10-20x more than the minimum but they’re slow enough to be visible to me slow, aged human eyes! And it gets worse, depending on the machine. One of the servers I use frequently is fairly slow (in CPU and IO): if the relevant files aren’t in cache, fish and zsh with my configuration add 300ms (a third of a second!) overhead.

## Reducing Shell Overhead

Last year I got particularly frustrated when I was executing huge numbers of commands, locally and remotely. My normal user shell (depending on the machine, fish or zsh) was adding noticeable overhead.

The first thing I did was to change some of my configuration so that it was skipped in non-interactive (i.e. “scripting”) mode. That helped a bit, but it’s harder than it looks. For example, with zsh I always get confused by when, and in which order, the `~/.zlogin`, `~/.zprofile`, `~/.zshenv`, and `~/.zshrc` configuration files are loaded. fish has a different model of which files are executed when, overloading my poor brain. Whatever shells I might use in the future will probably do something different again.

I then had a different idea. I want a fully featured shell like fish or zsh for interactive prompts, but I don’t care what’s used for non-interactive commands. Indeed, at a basic level, most Unix shells implement [the same base language](https://pubs.opengroup.org/onlinepubs/9699919799/utilities/V3_chap02.html). Even fish, which is explicitly not compatible with this standard, is in practice mostly compatible.

Most modern Unices come with a small shell installed in `/bin/sh` — for example, ksh in OpenBSD or dash in Debian. Here’s our fresh user account running `/bin/sh` on my OpenBSD laptop:

```
$ hyperfine --shell=none "/bin/sh -c exit" Benchmark 1: sh -c exit
 Time (mean ± σ):       2.0 ms ±   0.2 ms    [User: 0.3 ms, System: 1.1 ms] Range (min … max):     1.6 ms …   3.3 ms    996 runs 
```

That’s faster than any of the “big” shells, even if only by a couple of milliseconds. The performance story is similar on the Linux server:

```
$ hyperfine --shell=none "/bin/sh -c exit" Benchmark 1: sh -c exit
 Time (mean ± σ):       2.0 ms ±   0.1 ms    [User: 1.2 ms, System: 0.6 ms] Range (min … max):     1.6 ms …   2.5 ms    1462 runs 
```

Both ksh and dash are too bare-bones for me to want to use them as an interactive shell, so I don’t want to switch to them wholesale. Instead, what I want to do is use `/bin/sh` for non-interactive commands and fish (or another “big” shell) for interactive prompts.

## Shell Switching

It turns out to be straightforward to use one shell for non-interactive commands and another for interactive prompts.

First, I set my user shell to `/bin/sh` (with `chsh -c /bin/sh`). However, when I login at a terminal I’m now using ksh or dash, when I want to use fish or zsh.

`/bin/sh` always starts by loading and executing the shell script in `~/.profile`. In that file we need to work out if we’re running a non-interactive command or an interactive prompt. This stymied me for a while, but it turns out to be fairly easy. Shells set an environment variable `$-` which will contain the letter “i” if the shell is an interactive prompt.

With this knowledge, the minimum version of shell switching is to put the following in `~/.profile`:

```
case $- in
 *i* ) exec fish ;; esac 
```

If I’m starting an interactive prompt, this `exec`s fish (i.e. overwrites the `/bin/sh` process with `fish`), otherwise it sticks with `/bin/sh`. Interestingly, because `login` sets `$SHELL`, and fish doesn’t change that value, my fish shell reports `/bin/sh` as the value for `$SHELL`. Thus, whether a program uses `$SHELL` or the shell set in the user database to run a sub-command they always get a value of `/bin/sh`, which is what I want!

Happily my small `.profile` is so quick to execute that it has no meaningful impact on our benchmark:

```
$ hyperfine --shell=none "sh -c exit" Benchmark 1: sh -c exit
 Time (mean ± σ):       2.1 ms ±   0.1 ms    [User: 0.4 ms, System: 1.1 ms] Range (min … max):     1.8 ms …   2.7 ms    1111 runs 
```

I can even see how long it takes shell switching to execute fish for an interactive prompt:

```
$ hyperfine --shell=none "sh -il -c exit" Benchmark 1: sh -il -c exit
 Time (mean ± σ):     101.5 ms ±   7.7 ms    [User: 44.1 ms, System: 96.6 ms] Range (min … max):    94.5 ms … 123.2 ms    29 runs 
```

If I run this benchmark with fish directly, the resulting benchmark numbers are statistically indistinguishable. Shell switching is fast!

One question is: do we *need* some configuration even in `~/.profile`? Personally I don’t. If you do, the good news is that most of the time spent in shell startup tends to be spent on executing things humans want in interactive prompts. Adding a line or two to `~/.profile` which, say, adds directories to `$PATH` seems unlikely to add much overhead.

## Protecting Against Failing Shell Execution

A handful of times over the years I’ve found myself using a system where the shell binary doesn’t work: either it doesn’t exist; or it fails to execute. In both cases, this tends to happen during a system update and the end result is that I can’t login.

It’s possible to minimally extend “shell switching” so that `~/.profile` catches the obvious cases of this, meaning that `/bin/sh` ends up being my fall-back interactive prompt if my “big” shell doesn’t work:

```
case $- in
 *i* )
  if command -v fish > /dev/null; then
  fish --version > /dev/null && exec fish  echo "Couldn't run 'fish'" > /dev/stderr
  fi
 ;; esac 
```

You may notice a degree of paranoia here. First this does a fast check that a binary called “fish” exists (`command -v fish` returns non-zero if it doesn’t). Second it tries actually running fish to see if the binary at least minimally runs without exiting (using the `--version` flag as a proxy for this). Only if that succeeds does it `exec fish`.

## Summary

If you care about the speed of non-interactive command execution then shell switching is a simple technique for optimising this without effecting the configuration of your interactive prompt. It involves:

1.  setting your shell to `/bin/sh`.
2.  putting code into `~/.profile` which `exec`s a different shell when an interactive prompt is detected.

I’ve been running with this setup for nearly 12 months, and it’s caused me no problems, but has given me a useful speed boost!

*Update (2024-01-16)* The original code I gave for `~/.profile` contained `[[...]]` which isn’t POSIX compatible, and causes dash (and potentially other strict shells) to error. I’ve replaced it with `case ... esac` which is POSIX compatible. I reran the two benchmarks affected, though neither changed in statistically significant ways.

*Update (2024-01-18)* Fixed typo: `login` sets `$SHELL`, not `/bin/sh`.

[Newer](/laurie/blog/2024/some_reflections_on_writing_unix_daemons.html)

2024-01-16 14:50

[Older](/laurie/blog/2024/choosing_what_to_read.html)

If you’d like updates on new blog posts: follow me on

[Mastodon](https://mastodon.social/@ltratt)

or

[Twitter](https://twitter.com/laurencetratt)

; or

[subscribe to the RSS feed](../blog.rss)

; or

[subscribe to email updates](/laurie/newsletter/)

:

### Footnotes

### Comments