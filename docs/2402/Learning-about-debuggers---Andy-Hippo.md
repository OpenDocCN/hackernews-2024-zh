<!--yml
category: 未分类
date: 2024-05-27 14:54:44
-->

# Learning about debuggers | Andy Hippo

> 来源：[https://werat.dev/blog/learning-about-debuggers/](https://werat.dev/blog/learning-about-debuggers/)

# Learning about debuggers

January 8, 2024

Today’s article is a collection of materials to learn more about debuggers: how they work, which technologies are under the hood, what kind of problems exist in this area. There is of course a big overlap with related components like compilers and linkers, so get ready to learn lots of new things 😃.

Of course, the list is not exhaustive by any means. These are just the links I’ve accumulated over the years and found useful for myself. If you’d like to add anything to the list, let me know!

> Note: many of the links here are blog posts – be sure to check out other articles in those blogs! They’re often worth reading even if not directly related to the topic.

* * *

## How debuggers work

All debuggers share the same basic principles: they need to attach to a process, parse the binary and debug information, handle breakpoints, etc. The implementations may vary significantly between the operating systems, but after you understand one of them, others will look very familiar.

*   **Writing a Linux Debugger**
    *   [https://blog.tartanllama.xyz/writing-a-linux-debugger-setup/](https://blog.tartanllama.xyz/writing-a-linux-debugger-setup/)
    *   A 10-part journey of writing a Linux debugger from scratch. You’ll learn the basics of `ptrace`, how breakpoints work, how to map source code to machine code and how to unwind the stack. Highly recommend starting with this one, if you wanna learn the fundamentals.
*   **Writing a Debugger From Scratch**
*   **How Does a C Debugger Work? (GDB Ptrace/x86 example)**
*   **Debugging with the natives**
*   **Engineering Record And Replay For Deployability** (aka “how does `rr` work?”)
*   **Implementation of Live Reverse Debugging in LLDB**
    *   [https://arxiv.org/abs/2105.12819](https://arxiv.org/abs/2105.12819)
    *   Great detailed paper about the implementation of the reverse debugging in LLDB. Gives a good overview of various options and their trade-offs and explains in details the approach chosed for LLDB.
*   **How does gdb call functions?**

## Stack unwinding

Producing a call stack is one of the most basic debugger features, yet it’s far from trivial. In order to understand how it works, you’ll need to learn about frame pointers, call frame information (CFI), DWARF expressions and many other things. Here are some article that explore this topic in great detail.

*   **Unwinding the stack the hard way**
*   **Stack unwinding**
*   **The Apple Compact Unwinding Format: Documented and Explained**
*   **Unwinding a Stack by Hand with Frame Pointers and ORC**
*   **DWARF-based Stack Walking Using eBPF**

## Random debugger trivia

I already gave up with the categorization, so dumping the rest under “random trivia”.

*   **Where Did My Variable Go? Poking Holes in Incomplete Debug Information**
*   **Separating debug symbols from executables**
*   **Debugger Lies: Stack Corruption**
*   **Everything Is Broken: Shipping rust-minidump at Mozilla**
*   **Fuzzing rust-minidump for Embarrassment and Crashes**
*   **Crash reporting in Rust**
*   **So you want to live-reload Rust**

## General useful knowledge

*   **Linkers**
*   **Design and implementation of mold**
*   **The Definitive Guide to Linux System Calls**
*   **System calls in the Linux kernel**

* * *

Discuss this article: