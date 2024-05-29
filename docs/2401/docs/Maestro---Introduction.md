<!--yml
category: 未分类
date: 2024-05-27 14:28:47
-->

# Maestro - Introduction

> 来源：[https://blog.lenot.re/a/introduction](https://blog.lenot.re/a/introduction)

Thanks to the internet, I can learn how most things I am interested in work. However, something stayed a mystery to me for a long time: computers.

Computers are amongst the most complex tools that humanity has ever built. They are a marvel of engineering that we take for granted because we use them in our everyday lives.

I like to dig into complexity, and I like to learn by doing. On top of that, I spend a lot of time on the computer. Wouldn’t it be cool if I had a system that I would know from A to Z and that I could customise as much as I wanted to fit my expectations?

This is why I decided to build [Maestro](https://github.com/llenotre/maestro). A Unix-like operating system that is meant to be lightweight and compatible-enough with Linux to be usable in everyday life.

## A bit of history

The first commit of the kernel dates back to December 22nd, 2018, at 3:18 in the morning (the best time to write code, of course). It started as a school project.

It was originally implemented using the **C language** and continued to be for roughly a year and a half, until the codebase became too hard to keep clean.

At that moment, I decided to switch to **Rust** (my first project in this language), which represented several advantages:

*   Restart the project from the beginning, using lessons learned from previous mistakes
*   Be a bit more innovative than just writing a Linux-like kernel in C. After all, just use Linux at that point
*   Use the safety of the Rust language to leverage some difficulty of kernel programming. Using Rust’s typing system allows to shift some responsibility over memory safety from the programmer to the compiler

In kernel development, debugging is very hard for several reasons:

*   Documentation is often hard to find, and BIOS implementations may be flawed (more often than you would think)
*   On boot, the kernel has full access to the memory and is allowed to write where it should not (its own code, for example)
*   Troubleshooting memory leaks is not easy. Tools such as *valgrind* cannot be used
*   *gdb* can be used with *QEMU* and *VMWare*, but the kernel may have a different behaviour when running on a different emulator or virtual machine. Also, those emulators may not support gdb (example *VirtualBox*)
*   Some features in the support for gdb in QEMU or VMWare are missing and gdb might even crash sometimes

All those issues are reasons for using a memory-safe language, to avoid them as much as possible.

Overall, the use of Rust in the kernel allowed for the implementation of a lot of safeguards. And I believe that it is, to this day, the best decision I have made for this project.

### Timelapse

 <https://blog.lenot.re/assets/article/gource.mp4> 

Created using [Gource](https://gource.io/). Music: *Many Moons of Saturn, Mike Cole*

## The current state of the project

Maestro is a monolithic kernel, supporting only the x86 (in 32 bits) architecture for now.

At the time of writing, **135** out of **437** Linux system calls (roughly 31%) are more or less implemented. The project has **48 800** lines of code across **615** files (all repositories combined, counted using the `cloc` command).

The OS currently has the following components, aside from the kernel:

*   [Solfège](https://github.com/llenotre/solfege): a boot system and daemon manager (kind of similar to systemd, but lighter)
*   [maestro-utils](https://github.com/llenotre/maestro-utils): system utility commands
*   [blimp](https://github.com/llenotre/blimp): a package manager
*   And more components that are available on [my github](https://github.com/llenotre)

So far, the following third-party software has been tested and is working on the OS:

*   musl (C standard library)
*   bash
*   Some GNU coreutils commands such as `ls`, `cat`, `mkdir`, `rm`, `rmdir`, `uname`, `whoami`, etc…
*   neofetch (a patched version, since the original neofetch does not know about my OS)

## Test it yourself!

> **Disclaimer**: It is important to note that the OS is still in a very early stage of development and is highly unstable. I discourage trying to install it on a machine with important data on it.
> 
> So far, it has been tested mostly on QEMU, VMWare and VirtualBox.

There are two ways you can install the OS:

The ISO provides an installer for the OS. You can use it on QEMU, VMWare or VirtualBox for example.

> You should run the ISO with sufficient RAM (**1GB** should be more than enough).
> 
> Such an amount of memory is required because packages to be installed are stored in RAM (on the initramsfs) instead of the disk. This is currently the best method since the OS is *not yet* able to read on a USB stick or CD-ROM by itself, so it relies on the bootloader for this.

## What this blog is about

The aim of this blog is **not** to write tutorials about how to create an OS. This is already well covered by other websites/blogs. I recommend in particular:

The goal is to explore more advanced subjects (since most people/blogs tend to stop at the basics), to push the subjects as far as I am able to, to write articles about problems I encounter and how I solve them, to discover how computers work underneath, but also operating systems, the internet, and much more… Plenty of things to talk about!

## What’s coming next?

Cleaning of the codebase and performance optimisations are in order. Since the OS started as a school project, I had to cut corners in order to finish it on time. But now is the time to pay back the technical debt I accumulated.

Some memory leaks are also lying around and have to be fixed. Performance optimisations will probably be a subject for blog articles.

The next leap forward will be to have the package manager fully working on the OS. To do so, some features are required:

*   Network support, which is currently under development. And will probably be the subject of numerous articles
*   Shared library support. This currently does not work because it requires mapping files directly into memory, which is not currently supported by the implementation of the `mmap` system call on the kernel

After that, I will be able to install (without pain) and test programmes such as compilers (gcc/g++, clang, rustc), make, Git, Vim, etc… And then develop the kernel while using it!

The development of the kernel largely follows a simple procedure:

*   **1**: Run a programme on the kernel and see if it works correctly
*   **2**: If it does not work, then:
    *   **3**: Run the programme while printing system calls and search for the first system call that is causing troubles (not implemented or buggy)
    *   **4**: Implement or fix the system call in question
    *   **5**: Go to step 1
*   **6**: Else: Yay!

The more programmes running correctly on the kernel, the more stable and complete it becomes!

## How *you* can help

You can leave a star ⭐ on the [Github repository of the kernel](https://github.com/llenotre/maestro) ❤️

And stay in touch by:

Do not hesitate to join Discord! If you have feedback to make, advice to give, or questions to ask, I will be glad to answer!