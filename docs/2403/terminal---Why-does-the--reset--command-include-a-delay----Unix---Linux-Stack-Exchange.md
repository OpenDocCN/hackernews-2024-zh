<!--yml

category: 未分类

date: 2024-05-27 14:44:46

-->

# 终端 - 为什么`reset`命令包含延迟？ - Unix & Linux Stack Exchange

> 来源：[https://unix.stackexchange.com/questions/335648/why-does-the-reset-command-include-a-delay](https://unix.stackexchange.com/questions/335648/why-does-the-reset-command-include-a-delay)

真实（硬件）终端需要这样做。例如，对于某些终端，唯一的重置方法是进行硬件重置。

它在终端仿真器中是无害的，并且因为没有传统的方法可以区分（并且难以确定某些转义序列是否可能执行硬件复位），`reset`假设你的终端是真实的。

时间延迟可以追溯到1979年的3BSD中的`tset`，像这样：

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

它在ncurses中有所发展，但遵循相同的准则：

```
 if (!noinit) {
            if (send_init_strings(my_fd, &oldmode)) {
                (void) putc('\r', stderr);
                (void) fflush(stderr);
                (void) napms(1000);         /* Settle the terminal. */
            }
        } 
```

进一步阅读：
