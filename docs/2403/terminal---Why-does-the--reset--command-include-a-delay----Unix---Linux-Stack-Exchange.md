<!--yml
category: 未分类
date: 2024-05-27 14:44:46
-->

# terminal - Why does the `reset` command include a delay? - Unix & Linux Stack Exchange

> 来源：[https://unix.stackexchange.com/questions/335648/why-does-the-reset-command-include-a-delay](https://unix.stackexchange.com/questions/335648/why-does-the-reset-command-include-a-delay)

Real (hardware) terminals need that. For instance, with some, the only way to reset them is to do a hardware-reset.

It's harmless with a terminal emulator, and since there's no conventional way to tell the difference (and too hard to determine if some escape sequence might do a hardware-reset), `reset` assumes your terminal is real.

The time-delay dates back to `tset` in 3BSD in 1979, like this:

```
 /* output startup string */
    if (!RepOnly && !NoInit)
    {
            bufp = buf;
            if (tgetstr("is", &bufp) != 0)
                    prs(buf);
            bufp = buf;
            if (tgetstr("if", &bufp) != 0)
                    cat(buf);
            sleep(1);       /* let terminal settle down */
    } 
```

It's evolved somewhat in ncurses, but using the same guideline:

```
 if (!noinit) {
            if (send_init_strings(my_fd, &oldmode)) {
                (void) putc('\r', stderr);
                (void) fflush(stderr);
                (void) napms(1000);         /* Settle the terminal. */
            }
        } 
```

Further reading: