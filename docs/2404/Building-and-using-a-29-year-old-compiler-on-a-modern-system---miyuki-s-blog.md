<!--yml
category: 未分类
date: 2024-05-27 12:51:28
-->

# Building and using a 29-year-old compiler on a modern system · miyuki's blog

> 来源：[https://miyuki.github.io/2017/10/04/gcc-archaeology-1.html](https://miyuki.github.io/2017/10/04/gcc-archaeology-1.html)

# Building and using a 29-year-old compiler on a modern system

04 Oct 2017

In this post I’ll share my experience of building and using one of the earliest versions of the GNU C Compiler — GCC 1.27 (released in 1988) on a modern system.

## The environment

For my experiments I used an LXC container based on Debian 8 (Why not 9? Because I had started working on this post before Debian 9 was released). I decided to use an i386 container (not an amd64 one to save the efforts; I’m sure that everything would work out for amd64 too after some dances with paths and symlinks).

Debian 8 ships GCC 4.9.2 as a host compiler, Binutils 2.25 and Glibc 2.19.

## Obtaining the source code

Old releases of GCC are available from the official server at [gcc.gnu.org](https://gcc.gnu.org/pub/gcc/old-releases/). The first available release is version [0.9](https://gcc.gnu.org/pub/gcc/old-releases/gcc-1/gcc-0.9.tar.bz2). Unfortunately, this release is not very interesting for us, because it does not support the i386 architecture (or any compatible one).

Version [1.27](https://gcc.gnu.org/pub/gcc/old-releases/gcc-1/gcc-1.27.tar.bz2) of the GNU C Compiler (notice that back then it was called the GNU C Compiler rather than the GNU Compiler Collection) is the first available version that supports the i386 architecture. It was released on 05.09.1988 (Linux did not exist back then).

## Building and testing

Unlike modern versions, GCC 1.27 does not include any huge configury scripts and configuration is done manually. Nevertheless, it is very straightforward and well documented. In fact, it involves creating just 4 symlinks.

It is amazing, how well the C Standard compatibility is maintained in the GNU toolchain. Furthermore, essential Glibc headers are also backwards compatible with old compilers. Owing to this fact it is possible to compile GCC 1.27 using a modern compiler after patching only a dozen (out of ~92000) lines of code. Most of them are related to changes in the C library, and some are due to more strict C syntax rules implemented in modern C compilers (see [gcc-1.27.patch](https://gist.github.com/miyuki/9eab2c6a43e23c95183eb39e1f5e6833)).

Another problem was a missing header called `syms.h`, which apparently defines some constants used for generating debug information in the SDB format. It is unsurprising that the header is missing: on Linux SDB was superseded by the DWARF format a long ago. I managed to find these headers on the MIT website: [syms.h](http://www.mit.edu/afs.new/athena/system/rs_aix32/os/usr/include/syms.h). The URL suggests that these files have something to do with the IBM AIX OS.

### Headers and specs

Making the source compilable on a modern system is not enough to get a working compiler. As you probably know, the GNU toolchain includes other tools, such as an assembler and a linker with which the compiler interacts. Luckily, the syntax of generated assembly code is fully compatible with a contemporary version of the GNU assembler (except for debug information). As for the linker, some tweaks to the so-called linker specs (i.e. command line options used by the GCC driver) were needed.

With these fixes applied, we now have a compiler able to produce valid `elf` binaries.

### Bootstrapping

A quick reminder: bootstrapping a compiler means compiling it using itself. For more details please refer to the "[Bootstrapping (compilers)](https://en.wikipedia.org/wiki/Bootstrapping_(compilers))" Wikipedia article.

The first minor problem with bootstrap comes from glibc headers: modern versions of glibc assume that the compiler supports 64-bit integral types, which is not true for GCC 1.27\. The culprit is `/usr/include/bits/byteswap.h`. It’s inclusion can be easily disabled by passing a flag `-D_BITS_BYTESWAP_H` to the compiler.

Now, attempting to bootstrap the compiler leads to a failure during compilation of a file named `cccp.c` related to the C preprocessor: the compiler crashes with a segmentation fault. After digging a bit into the cause of this failure, I managed to produce a minimal failing test case using [C-Reduce](https://embed.cs.utah.edu/creduce/):

```
struct file_buf { } fn1(), a;
fn2() { a = fn1(); }
```

The compiler crashes attempting to dereference a null pointer while translating expression `a = fn1()` from an AST into its intermediate representation (RTL). Apperently the i386 back end has a bug in the code dealing with calls to functions that return structs by value.

It turned out that fixing this bug is relatively straightforward. Adding a single check, which is present in GCC 1.31 fixes the bug and bootstrap now succeeds.

By the way, this could mean that no one have ever tried to bootstrap GCC 1.27 on x86 before.

Moreover, I managed to perform bootstrap comparison, i.e. to build:

*   stage 1 compiler, i.e. GCC 1.27 compiled by the host compiler (GCC 4.9.2)
*   stage 2 compiler, i.e. GCC 1.27 compiled by the stage 1 compiler
*   stage 3 compiler, i.e. GCC 1.27 compiled by the stage 2 compiler

As I expected stage 2 and stage 3 were identical.

### Playing around

I tried to find some code (besides GCC itself) to compile and play with. Remember: we need code written in so-called K&R C (because the ANSI C89 Standard did not exist back then).

For example, I used a [program](https://people.sc.fsu.edu/~jburkardt/c_src/mandelbrot_ascii/mandelbrot_ascii.html) that produces an ASCII image of the Mandelbrot set (unfortunately, I failed to find out who the author of this code is).

Here is the code, formatted for better readability:

```
main (n)
{
  float r, i, R, I, b;
  for (i = -1; i < 1; i += .06, puts (""))
    for (r = -2; I = i, (R = r) < 1; r += .03, putchar (n + 31))
      for (n = 0; b = I * I, 26 > n++ && R * R + b < 4;
        I = 2 * R * I + i, R = R * R - b + r);
}
```

As you can see, it involves quite complex control flow and floating point arithmetic. GCC 1.27 compiles it without errors and the output matches that of the same code compiled by a modern version of GCC.

GCC 1.27 includes many features typical for modern compilers, such as:

*   Compiler warnings
*   Optimizations
*   Debug information output
*   Instrumentation for code profiling
*   Command-line flags controlling all of the above

## Exploring the GCC source code

Another thing that surprised me, is that a lot of ideas and even much of the code of these old versions of GCC are still used today.

Multiple back ends are supported in the compiler by means of switchable header files (i.e., `config-i386v.h` is used on i386 System V, and `config-sparc.h` is used on Sparc Sun). Machine description `.md` files describe CPU instruction patterns. During the build stage, they are parsed and transformed into C source files, which later get linked with the compiler. Of course, during the past decades the DSL of `.md` has evolved, but the principle remains the same. Furthermore, the LLVM compiler infrastructure [uses](http://llvm.org/docs/CodeGenerator.html#using-tablegen-for-target-description) a similar technique.

The two ubiquitous data types `tree` and `rtl` which are used for representing the program in the compiler front end and back end respectively still serve their purpose.

Of course this does not mean that current versions of GCC are stuck in the eighties. Despite some similarities, the number of major enhancements is really huge and I won’t bother listing them (it’s a good topic for a post with some benchmarks, charts and infographics).

Old versions of GCC did not have a bug-tracker site, so instead, a list of bugs and enhancement requests was kept in a file called `PROBLEMS`. As of version 1.27 it contains 27 entries with many gaps in numbering. Actually, the last item has number 122\. This probably means that the remaining 95 problems had been fixed before this release.