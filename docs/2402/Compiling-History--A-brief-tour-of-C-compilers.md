<!--yml
category: 未分类
date: 2024-05-27 14:48:37
-->

# Compiling History: A brief tour of C compilers

> 来源：[https://www.deusinmachina.net/p/compiling-history-a-brief-tour-of](https://www.deusinmachina.net/p/compiling-history-a-brief-tour-of)

As the story of C’s birth goes hand in hand with the creation of Unix, the first C compiler can be traced back to the early 1970’s. I've detailed the history of C in my previous article [Tracing the Lines: From the Telephone to Unix](https://www.deusinmachina.net/p/history-of-unix), which includes a brief summary of this history.

> Around 1971, Ken decided that Unix needed to be ported to a higher level language. Dennis Ritchie took on the task, evolving Ken’s B language into something more feature rich. It was first called New B (NB), but each time Ken tried to rewrite the kernel in New B he would run into a roadblock. He would then ask Dennis to add more features. Eventually after structures were invented, there were enough features that Ken could rewrite version 4 of the whole Unix kernel in it. After a new compiler was written for this new language, it was renamed to C, and the rest is history. This was a significant breakthrough, as, until then, kernels were written in Assembly. For perspective, as late as 1983, Microsoft was still programming [MS-DOS v2.0](https://github.com/microsoft/MS-DOS/tree/master/v2.0/source) in Assembly. Unix was truly ahead of its time.

Since Unix was created on a PDP-10/11, it makes sense that the first compiler for C was created for the PDP-11\. This is usually just referred to as the PDP C Compiler. The earliest known version of this compiler’s source code can still [be viewed here](https://github.com/mortdeus/legacy-cc), and is an interesting time capsule of computing history.

This was succeeded by the Portable C Compiler, developed by Stephen C. Johnson of Bell Labs, and one of the first compilers that was capable of generating machine independent C code. In Section 2.1 of Bjarne Stroustrup’s article titled [Sibling Rivalry: C and C++](https://stroustrup.com/sibling_rivalry.pdf), he details key aspects of this compiler and why it was so important

> Pre-ANSI C is often referred to as K&R C. However, that is slightly incorrect. The C described in [Kernighan,1978] lacks three features of the language used by almost all C programmers before the emergenceof C89: void, enumerations, and structure assignment. These three features were added in PCC, the Portable C Compiler, developed by Steve Johnson and distributed as the C compiler by Bell Labs (with the ‘‘blessing’’ of Dennis Ritchie).
> 
> Adding void (used as a possible return type for functions only) allows a programmer to directly express that a function doesn’t return a value, and allows the compiler to check that. Similarly, adding enumerations allows a programmer to directly express that a group of values in some way belong together. It also supports the notion of manifest constants in a way that does not rely on macros.
> 
> Adding structure assignment (and also structure copy initialization, argument passing, and function return) makes struct values first-class citizens of C.
> 
> Thus, two of the three last additions to Classic C add to the expressive power of the type system without actually allowing a programmer to express any new computations. The third makes user-defined types, as then existing, equal to built-in types. In addition, one of the additions provides an alternative to the use of macros. These are all themes that recur in the design of C++.

The Portable C Compiler was distributed with version 7 of Unix, the last version of Unix released before it was commercialized. Due to its early mover advantage, and the fact that it could be adapted to produce assembly for different architectures, meant that it enjoyed much success in the nascent years of C.

But it was not the only compiler to pop up during that time. The 1970s also saw the Small-C compiler created by Ron Cain. It was a minimalist subset of C that could run on 8-bit microcomputers. It’s hard to believe now a days that computers at one point struggled to compile C code, let alone a subset of it, but that was indeed the case. The PDP-11 that C was developed on was a 16-bit broom closet sized computer, which was still considerably more powerful than the 8-bit home computers of the day. This is often why programs from that era were written in Assembly, Basic, and Pascal instead of C.

Another commercial compiler to come out during this time was the Lattice C compiler, one of the first C compilers written for the IBM Personal Computer. It was created by [Lifeboat Associates](https://en.wikipedia.org/wiki/Lifeboat_Associates) and retailed for $500 ($1,628 in today’s money), and it ran on PC-DOS and MS-DOS. Microsoft used this as the basis for their Microsoft C Compiler (MSC). During this time many compilers were produced including, the Mark Williams Compiler, the Green Hills compiler, the Aztec C compiler and many others.

These developments however, navigated a landscape devoid of an official C standard, leading to varied interpretations and implementations. They were based on "The C Programming Language" book by Brian Kernighan and Dennis M. Ritchie published on February 22, 1978\. The eventual release of the standard, known as C89 or C90, brought much needed uniformity and clarity to the language. The preface in the 2nd edition of the book, published in April of 1988, highlights the importance.

> The standard formalizes constructions that were hinted but not described in the first edition, particularly structure assignment and enumerations. It provides a new form of function declaration that permits cross-checking of definition with use. It specifies a standard library, with an extensive set of functions for performing input and output, memory management, string manipulation, and similar tasks. It makes precise the behavior of features that were not spelled out in the original definition, and at the same time states explicitly which aspects of the language remain machine-dependent.

With the publishing of the standard, C became a much more consistent language to program across environments.

Fast forward to the present, the GNU Compiler Collection (GCC) stands as a testament to the evolution of compilers, supporting not just multiple platforms, but also multiple languages.

One of my favorite things about computer lore is that many of the most instrumental people are still alive today, and we still have records of when they made history. This is no different for GCC, and we actual have the text [Richard Stallman sent](https://gcc.gnu.org/wiki/History), introducing the GCC beta back in 87\.

```
 Date: Sun, 22 Mar 87 10:56:56 EST
   From: rms (Richard M. Stallman)

   The GNU C compiler is now available for ftp from the file
   /u2/emacs/gcc.tar on prep.ai.mit.edu.  This includes machine
   descriptions for vax and sun, 60 pages of documentation on writing
   machine descriptions (internals.texinfo, internals.dvi and Info
   file internals).

   This also contains the ANSI standard (Nov 86) C preprocessor and 30
   pages of reference manual for it.

   This compiler compiles itself correctly on the 68020 and did so
   recently on the vax.  It recently compiled Emacs correctly on the
   68020, and has also compiled tex-in-C and Kyoto Common Lisp.
   However, it probably still has numerous bugs that I hope you will
   find for me.

   I will be away for a month, so bugs reported now will not be
   handled until then.

   If you can't ftp, you can order a compiler beta-test tape from the
   Free Software Foundation for $150 (plus 5% sales tax in
   Massachusetts, or plus $15 overseas if you want air mail).

   Free Software Foundation
   1000 Mass Ave
   Cambridge, MA  02138
```

This feels like computer archeology to me.

Today, GCC is more than just a C compiler. It’s a Compiler Collection, and it supports these programming languages: C, C++, Objective-C, Objective-C++, Fortran, Ada, D, and Go. It has support for the most platforms, and the most CPU architectures out of all compilers today, and is still being actively developed.

But GCC isn’t the only cross platform industrial grade compiler on the block. LLVM provides a great experience as well, and benefits from decades of hindsight in compiler construction. It was created by Vikram Adve and his PhD student Chris Lattner at the [University of Illinois at Urbana–Champaign](https://en.wikipedia.org/wiki/University_of_Illinois_at_Urbana%E2%80%93Champaign) in 2000\. It started as a research project in December while Chris was on winter break. Over the course of the next year, Chris and Vikram continued to work on the compiler before publishing their first paper on it titled, [Automatic Pool Allocation for Disjoint Data Structures](https://llvm.org/pubs/2002-06-AutomaticPoolAllocation.pdf). Though they didn’t know it at the time, they were making history. A lot had happened in the field of compilers by the early 2000s, and this allowed LLVM to enjoy many benefits.

1.  **LLVM IR**

    *   Front-end developers only need to understand LLVM IR, its workings, and invariants, making it easy to create new front ends for LLVM. Unlike other compilers like GCC, LLVM IR is self-contained, eliminating the need to manipulate complex data structures and global variables from other parts of the compiler

2.  **Modular Library-Based Design**

    *   The LLVM infrastructure consists of loosely coupled libraries instead of a monolith, including the optimizer, allowing developers to choose and order optimization passes for their specific needs. Only the necessary optimization passes are linked into the final application, optimizing compile times and avoiding unnecessary bloat

3.  **Retargetable Code Generator**

    *   The LLVM code generator transforms LLVM IR into target specific machine code. It employs a modular approach with individual passes for instruction selection, register allocation, scheduling, code layout optimization, and assembly emission. This flexibility enables target-specific optimizations, such as register pressure reduction for x86 and latency optimization for PowerPC, without requiring a complete code generator rewrite

And lastly it’s 13 years younger than GCC, and benefits from not having to support as many architectures. This means that this, along with LLVM’s modular natures, allows LLVM’s code base to be about 3.5 times smaller than GCC (5million vs 1.6 million lines of code). While these numbers might seem staggering, it pales in comparison to the mighty Ford F150 which has 150 million lines of code much of which is compiled with LLVM and GCC I’m sure.

But while LLVM might be newer, it still faces stiff competition from GCC. Both GCC and LLVM support many modern C and C++ standards, and they both have a large suite of tools for working with their output. In the great article by Jeremy Bennett titled [How Much Does a Compiler Cost?](https://www.embecosm.com/2018/02/26/how-much-does-a-compiler-cost/) We see the many tools that these compilers bring, as well as the hundreds of thousands of lines of code it takes to create the supporting software suite.

*   **Debugger:** Either GDB (800k lines) or LLDB (600k lines)

*   **Linker:** GNU ld (160k lines), gold (140k lines) or lld (60k lines)

*   **Assembler/disassembler:** GNU gas (850k lines) or the built in LLVM assembler

*   **Binary utilities:** GNU (90k lines) and/or LLVM (included in main LLVM source)

*   **Emulation library:** libgcc (included in GCC source) or CompilerRT (340k lines)

*   **Standard C library:** newlib (850k lines), glibc (1.2M lines), musl (82k lines) or uClibC-ng (251k lines)

In Matt Godbolt’s CppCon talk titled *What Has My Compiler Done for Me Lately? Unbolting the Compiler's Lid*, we see how many of the clever optimization tricks we used to need to do in code, can be done at the compiler level. This allows us to have simpler and easier to maintain code, without making tradeoffs on performance

I’m grateful for all the work compilers maintainers have put in optimizing our programs. We’ve come a long way to get here, and have built upon decades of hard won programming experience to come up with the robust solutions we have today. As our programming languages continue to evolve, our compilers will be right their with us, tirelessly optimizing our code and catching bugs, so we can get the best performance we can

Hi 👋 my name is Diego Crespo and I like to talk about technology, niche programming languages, and AI. I have a [Twitter](https://twitter.com/deusinmach) and a [Mastodon](https://mastodon.social/deck/@DiegoCrespo), if you’d like to follow me on other social media platforms. If you liked the article, consider liking and subscribing. And if you haven’t why not check out another article of mine listed below! Thank you for reading and giving me a little of your valuable time. A.M.D.G

[Share](https://www.deusinmachina.net/p/compiling-history-a-brief-tour-of?utm_source=substack&utm_medium=email&utm_content=share&action=share)

[Share Deus In Machina](https://www.deusinmachina.net/?utm_source=substack&utm_medium=email&utm_content=share&action=share)