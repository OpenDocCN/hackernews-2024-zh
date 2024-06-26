<!--yml

类别：未分类

日期：2024 年 5 月 27 日 14:23:40

-->

# Coproc 教程，重制

> 来源：[`www.zsh.org/mla/users/2011/msg00095.html`](https://www.zsh.org/mla/users/2011/msg00095.html)

**Zsh 邮件列表存档**按照消息的顺序排序：逆序日期，日期，主题，作者

# Coproc 教程，重制

* * *

+   *X-seq*：zsh-users 15773

+   *来自*：巴特·舍弗 <schaefer@xxxxxxxxxxxxxxxx>

+   *收件人*：zsh-users@xxxxxxx

+   *主题*：Coproc 教程，重制

+   *日期*：星期六，2011 年 2 月 5 日 15:26:45 -0800

+   *列表帮助*：<mailto:zsh-users-help@zsh.org>

+   *列表 ID*：Zsh 用户列表 <zsh-users.zsh.org>

+   *列表发布*：<mailto:zsh-users@zsh.org>

+   *邮件列表*：联系 zsh-users-help@xxxxxxx；由 ezmlm 运行

* * *

```
A very long time ago (11+ years) I wrote an zsh-users post about how to
use the coproc command.  It might be time to repeat/update it, especially
given that I now understand the reason for one of the problems I noted in
the original article.

* What is a "coproc"?

It's short for "co-process" which means a second process cooperating with
the shell.  It's very similar to a background job started with an "&" at
the end of the command, except that instead of sharing the same standard
input and output as its parent shell, its standard I/O is connected to
the parent shell by a special kind of pipe called a FIFO (first in, first
out).  This allows the parent shell to communicate with the coproc.

(In fact in ksh you start a coprocess by placing "|&" at the end of the
command, but that token is already used by zsh for another purpose.)

One starts a coproc in zsh with

	coproc command

The command has to be prepared to read from stdin and/or write to stdout,
or it isn't of much use as a coproc.  Generally speaking, the command also
should not be one that uses buffered writes on its output, or you may end
up waiting for output that never appears (more on this below).

After it's running, you have several choices:

Write to the coproc with "print -p ..."
Read from the coproc with "read -p ..."
Redirect output to the coproc with "othercommand >&p"
Redirect input from the coproc with "othercommand <&p"

* Why is a coproc useful?

A coproc allows more explicit control over the order of execution of
commands and permits incremental communication between the parent shell
and the coproc.  A command substitution ( `...` or $(...) ) is the
more common way of capturing the output of another command, but must be
given all of its input up front and must run to completion so that all
of its output can be consumed at once.  Replacing it with a coproc
runs it in parallel with the shell, so the shell needn't wait.

Coproc can be used to assemble a pipeline in a specific order.  In zsh,
a "while ...; do" loop at the right-hand-end of a pipeline is run in
the current shell (unless backgrounded), so you can set and export
parameters and so on and they remain set when the loop finishes; but
zsh is unique in this.  In other shells, such a loop would be run in a
subshell, and the left side of the pipe in the current shell.  With a
coproc, you can choose which side of the "pipeline" to run in the
coproc and use redirection for the other side, and thus explicitly
control which portion is run in the current shell.

Coprocs are also useful if you ever have two commands that each want to
consume the other's output:

    coproc command1
    command2 <&p >&p

I've never had reason to do that, but I suppose it would be good if one
was testing a pair of network protocol daemons.

There are of course other paradigms that can be implemented with coproc.
For example, something like:

    if condition1; then
        coproc command1
    elif condition2; then
        coproc command2
    else
        coproc command3
    fi
    # ... do a bunch of other setup for command4, then ...
    command4 <&p

In practice I find that a coproc is rarely an absolute requirement, it
is just more efficient or convenient in some circumstances.

* How does it work?

Here's a very simple example of a coproc that converts all the text sent
to it into upper case ("zsh% " is the shell prompt):

    zsh% coproc while read line; do print -r -- "$line:u"; done

Note that you can put an entire control structure into a coproc; it works
just like putting "&" at the end.  The coproc shows up in the job table
as a background job; you can bring it into the foreground, kill it, etc.
In fact, it's a no-op to put an "&" at the end of a "coproc ...", because
zsh is going to background the job already.

With that coproc running, I can say

    zsh% print -p foo ; read -ep
    FOO

(Using "read -e" means to immediately echo what was just read.)

About that output buffering thing:  You might wonder why I didn't use:

    zsh% coproc tr a-z A-Z
    zsh% print -p foo ; read -ep

It's because of the output buffering done by "tr".  The "print -p foo" is
happily consumed by "tr", but it doesn't produce any output until it has
either processed a whole buffer-full of bytes (usually 1024) or until it
has seen end-of-file on its input and is about to exit.  So "read -ep"
sits there forever, waiting for "tr", which is also sitting there forever
waiting for someone to send it some more bytes.

* Where might I go wrong with coproc?

This brings us to an oddity about zsh's coproc:  It sees end-of-file on
its input only when a new coproc is started.  In other shells, using the
equivalent of the "othercommand >&p" redirection causes the shell to
discard its own copy of the coproc descriptor, so the coproc gets an
EOF when "othercommand" closes its output (exits).  Zsh, however, keeps
the coproc descriptor open so that you can repeatedly direct new output
to the same coproc.  However, there can only be one magic "p" descriptor,
so when you issue a new "coproc ..." command, zsh finally does close its
copy of the descriptor.  (Some "othercommand" may still have it open.)

One idiom for closing off a coproc's input and output is to use:

    zsh% coproc exit

That starts a new coproc (which immediately exits), causing the input and
output of the old coproc (if any) to be shut down.

However, there is a "gotcha" here:  When the coproc is a shell construct
(a complex command like a loop, or a shell function) zsh must fork a new
subshell to run that construct in the background.  All versions of zsh up
through 4.3.11 have bug in that this forked subshell still has the magic
"p" descriptor open.  This means that "coproc exit" in the parent won't
close the input of the shell construct, because the intermediate subshell
is also holding it open; so the parent can't force the coprocess to see
an end-of-file on its input, and the coprocess may deadlock.  This does
not affect external commands run as coprocesses.

The workaround for this gotcha is to close the coproc inside the coproc
itself:

    zsh% coproc { coproc :
    while read line; do print -r -- "$line:u"; done }

This demonstrates that you can use coproc on an entire brace construct.
It also shows that you can use any command that immediately exits (":",
"return", "exit", "true", "false", etc.) to close off the coproc.

This bug is related to another oddity:  You may think from reading the
"Why is a coproc useful?" section that you can build up your own
pipelines by chaining coproc together like this:

    coproc tail
    coproc head >&p

That appears to say "start `tail' as a coproc, and then start `head' as a
new coproc with its output connected to the input of the old coproc."
However, that doesn't work; zsh recreates the coproc descriptors before
processing the redirection, so what "coproc head >&p" actually does is run
"head" with its output connected back to its own input.  This is a good
way to create either deadlock or an extremely CPU-intensive loop, so I
don't recommend doing it.

It's possible that one or both of these bugs will be fixed in 4.3.12
and later.

Now a word about input buffering:  In my example, I sent a line to the
coproc with "print -p foo ;" leaving the "print" in the foreground.
That's because I know for a fact that the coproc will consume one line
of input (the "while read line" loop) before producing any output at all,
so I'm sure that "print" will finish successfully.  Some other coproc
might read only a few bytes before stopping to do some other work, in
which case my "print" would block and "read -ep" might never run.  It's
more usual, therefore, to send input to the coproc from a background
job:

    zsh% cat /etc/termcap >&p &

(I picked /etc/termcap because it's usually a huge file, so that command
will almost certainly block if not backgrounded.)

* Anything else?

No, that about covers it.  Enjoy.

```

* * *

* * *

按照消息的顺序排序：逆序日期，日期，主题，作者
