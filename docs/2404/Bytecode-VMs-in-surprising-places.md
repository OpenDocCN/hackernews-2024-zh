<!--yml
category: 未分类
date: 2024-05-27 13:40:55
-->

# Bytecode VMs in surprising places

> 来源：[https://dubroy.com/blog/bytecode-vms-in-surprising-places/](https://dubroy.com/blog/bytecode-vms-in-surprising-places/)

April 30, 2024

In response to a question on Twitter^(, Richard Hipp wrote about [why SQLite uses a bytecode VM](https://sqlite.org/draft/whybytecode.html) for executing SQL statements.)

Most people probably associate bytecode VMs with general-purpose programming languages, like JavaScript or Python. But sometimes they appear in surprising places! Here are a few that I know about.

## eBPF

Did you know that inside the Linux kernel, there’s an extension mechanism that includes a bytecode interpreter and a JIT compiler?

I had no idea. Well, it’s called [eBPF](https://en.wikipedia.org/wiki/EBPF), and it’s pretty interesting: a register-based VM with ten general-purpose registers and over a hundred different opcodes.

The “BPF” in eBPF stands for *Berkeley packet filter*, and the basic idea is described in [a 1993 USENIX paper](https://www.tcpdump.org/papers/bpf-usenix93.pdf):

> Many versions of Unix provide facilities for user-level packet capture, making possible the use of general purpose workstations for network monitoring. Because network monitors run as user-level processes, packets must be copied across the kernel/user-space protection boundary. This copying can be minimized by deploying a kernel agent called a packet filter, which discards unwanted packets as early as possible. The original Unix packet filter was designed around a stack-based filter evaluator that performs sub-optimally on current RISC CPUs. The BSD Packet Filter (BPF) uses a new, register-based filter evaluator that is up to 20 times faster than the original design.

So it was originally designed for a pretty restricted use case: a directed, acyclic control flow graph representing a filter function for network packets. And for a long time, the Linux implementation was equally simple: two general-purpose registers, a switch-style interpreter, and no backwards branches.

A patch in 2011 added [a JIT compiler for x86-64](https://lwn.net/Articles/437981/). In 2012, [the first non-networking use case appeared](https://lwn.net/Articles/475043/). Then, in 2014, the BPF implementation was substantially extended on its way to becoming [the universal in-kernel virtual machine](https://lwn.net/Articles/599755/):

> It expands the set of available registers from two to ten, adds a number of instructions that closely match real hardware instructions, implements 64-bit registers, makes it possible for BPF programs to call a (rigidly controlled) set of kernel functions, and more. Internal BPF is more readily compiled into fast machine code and makes it easier to hook BPF into other subsystems.

## DWARF expressions

DWARF is a file format used by compilers like GCC and LLVM to include debug information in compiled binaries. Say you’re trying to debug the following C++ code:

```
void add_two(int x)
{
    int ans = x + 2;
    return ans + 2; // Oops!
}
```

In a debugger, you might want to print the value of the `ans` variable. But depending on the compiler and the code, this could be surprisingly difficult! It could be in a register, on the stack, or it might have been optimized away (to name just a few possibilities).

The solution is allow the compiler to specify an *expression* that will compute the value of the local variable. So the [DWARF spec](https://dwarfstd.org/doc/DWARF5.pdf) includes an expression language:

> **2.5 DWARF Expressions**
> 
> DWARF expressions describe how to compute a value or specify a location. They are expressed in terms of DWARF operations that operate on a stack of values.
> 
> A DWARF expression is encoded as a stream of operations, each consisting of an opcode followed by zero or more literal operands. The number of operands is implied by the opcode.

It’s up to the debugger to evaluate the expressions. GDB and LLDB both have a switch-based interpreter for DWARF expressions.

## GDB agent expressions

But it turns out that GDB has another bytecode interpreter!

> Using GDB’s `trace` and `collect` commands, the user can specify locations in the program, and arbitrary expressions to evaluate when those locations are reached.
> 
> When GDB is debugging a remote target, the GDB agent code running on the target computes the values of the expressions itself. To avoid having a full symbolic expression evaluator on the agent, GDB translates expressions in the source language into a simpler bytecode language, and then sends the bytecode to the agent; the agent then executes the bytecode, and records the values for GDB to retrieve later.
> 
> The bytecode language is simple; there are forty-odd opcodes, the bulk of which are the usual vocabulary of C operands (addition, subtraction, shifts, and so on) and various sizes of literals and memory reference operations. The bytecode interpreter operates strictly on machine-level values — various sizes of integers and floating point numbers — and requires no information about types or symbols; thus, the interpreter’s internal data structures are simple, and each bytecode requires only a few native machine instructions to implement it. The interpreter is small, and strict limits on the memory and time required to evaluate an expression are easy to determine, making it suitable for use by the debugging agent in real-time applications.

*(From [Debugging with GDB, Appendix F: The GDB Agent Expression Mechanism](https://sourceware.org/gdb/current/onlinedocs/gdb.html/Agent-Expressions.html))*

## WinRAR

WinRAR ^(is a file compression utility for Windows with a proprietary file format. Tavis Ormandy, a vulnerability researcher at Google, [discovered that the RAR format includes a bytecode encoding for data transformation](https://blog.cmpxchg8b.com/2012/09/fun-with-constrained-programming.html):)

> Believe it or not, RAR files can contain bytecode for a simple x86-like virtual machine called the RarVM. This is designed to provide filters (preprocessors) to perform some reversible transformation on input data to increase redundancy, and thus improve compression.

His [rarvmtools repo](https://github.com/taviso/rarvmtools) includes some details on the architecture:

> Familiarity with x86 (and preferably intel assembly syntax) would be an advantage.
> 
> RarVM has 8 named registers, called r0 to r7\. r7 is used as a stack pointer for stack related operations (such as push, call, pop, etc). However, as on x86, there are no restrictions on setting r7 to whatever you like, although if you do something stack related it will be masked to fit within the address space for the duration of that operation.

Some others:

*   The [TrueType](https://developer.apple.com/fonts/TrueType-Reference-Manual/) font specification includes a set of over 200 instructions used for glyph rendering and hinting.
*   [PostScript](https://en.wikipedia.org/wiki/PostScript) is not only a page description language, but a relatively powerful stack-based programming language. PostScript files are plain text, so a PostScript renderer doesn’t necessarily use bytecode, but the spec also includes a binary encoding.

👉 *[Discuss on Hacker News](https://news.ycombinator.com/item?id=40211205).*