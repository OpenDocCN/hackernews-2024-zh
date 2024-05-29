<!--yml
category: 未分类
date: 2024-05-27 14:32:12
-->

# CellLVM // -dealloc

> 来源：[https://belkadan.com/blog/2023/12/CellLVM/](https://belkadan.com/blog/2023/12/CellLVM/)

A few weeks ago I posted this:

<https://belkadan.com/blog/2023/12/CellLVM/CellLVM.mp4>

[(screen recording)](https://belkadan.com/blog/2023/12/CellLVM/CellLVM.mp4)

Which, if you’re not interested in watching a video right now, is a proof-of-concept LLVM to Excel spreadsheet compiler.

### What.

The night before, I was talking with friends about [CSV](https://en.wikipedia.org/wiki/Comma-separated_values), in particular joking about alignment charts and what counted as “true” CSV. Someone pointed out that assembly, as conventionally printed, could be a CSV, to which I responded

> ooh, asm in Excel nearly wraps around to being a good idea
> your labels are row references

As I lay in bed, I realized that this was a better match than I initially thought. Forget the CSV part, row references are what let Excel do *computation.* And while you could make that work with assembly, there’s an alternative that’s a much better fit: LLVM.

### SSA Format

[LLVM](https://llvm.org) is a library for building compilers and related tools—the one used by Swift and Rust, actually. ^(At its core is a stripped-down language also called LLVM, or maybe “[the LLVM instruction set](https://llvm.org/docs/LangRef.html)”. What’s unique about this language, besides being designed as an intermediate stage for compiling higher-level languages, is that every local variable is assigned a value exactly once, as a simple expression that can only depend on the variables that come before it. This is called [static single-assignment form](https://en.wikipedia.org/wiki/Static_single-assignment_form).)

My insight, which I don’t think is new, is that Excel formulas work the same way. Any cell with a formula has that formula defined up front, and values flow through the spreadsheet based on the references set in the formulas—just like SSA. So it should be possible, for operations supported by both LLVM and Excel, to rewrite an LLVM function as an Excel sheet that performs the same computation!

I went to sleep excited about that idea. I woke up and realized the primary problem with it: what about loops?

### Phi Nodes

In order for SSA to represent branching control flow (such as a conditional increment), it has to have some notion of history when the branches join back up. The conventional way to do this is with a special kind of expression called a *phi node,* which basically says “if we came from the true block, use x[1] as the value; if we came from the else block, use x[2] as the value”. The name “phi” isn’t short for anything; it’s apparently just meant to be close to “fi”, as in “if” backwards. ^(This form works for switches as well (there are just more possible predecessors), and even loops: the predecessor of a loop body might be the entry of the loop, or it might be the last block in the *previous* time through the loop.)

But spreadsheets don’t have loops, do they? I searched around a bit and discovered I was incorrect: Excel spreadsheets *do* support loops, in the form of “iterative calculation”. As long as the formulas converge on a fixed point within a certain number of steps, Excel will find it. So now I need to figure out how to encode loops in such a way that they do, in fact, converge.

At this point I got up from my bed and started messing around in Google Sheets. (It was a weekend, I didn’t have to go to work.) And I hit on a solution: a variation of the classic “program counter” used by real CPUs. If you keep track of the number of blocks you’ve visited, counting repeats, then you always know what the “current” block is, and more importantly what the “previous” block was. Which means you can implement phi nodes.

Here’s what a phi expression looks like in Excel:

| `=CHOOSE(` | A phi is a choice… |
| `XMATCH(` | of values based on a source… |
| `MAX(` | which is the most recent (max) PC… |
| `IF(B5=ROW(),C5,0),` | of the branch in row 5, if it was coming here… |
| `IF(B10=ROW(),C10,0),` | the branch in row 10, if it was coming here… |
| `C7-0.5),` | and the current row, 7, as a last resort… |
| `{C5,C10,C7-0.5}),` | which we get as an index with `XMATCH`… |
| `B4,B8,B7)` | and `CHOOSE` the correct value |

Not pretty but it gets the job done. And with that, I knew it was possible, and I set off to build the compiler.

### The Compiler

The actual compiler is a scant 150 lines of code, partly because it barely implements anything, but also because it’s not actually doing much work. All of the hard parts are in LLVM (and its wrapper, [LLVMSwift](https://github.com/llvm-swift/LLVMSwift), which [I did have to fork](https://belkadan.com/source/LLVMSwift/)) and [xslxwriter](https://libxlsxwriter.github.io/) (and [its own wrapper](https://github.com/damuellen/xlsxwriter.swift)). Without these pre-existing libraries, doing this in a day would have been impossible.

The compiler makes two passes over a single LLVM function: one to assign rows to instructions and basic blocks, and one to translate each instruction into a formula, 1:1\. There are only three relevant columns: a label with the instruction type, the value of each instruction, and the “program counter” described above. [You can read the whole thing if you want.](https://belkadan.com/source/CellLVM/)

The result is a command-line tool that takes LLVM bitcode as input and produces an xlsx file as output. It throws an error if there’s more than one function in the input, or if there’s an operation it doesn’t support (like, say, subtraction). But I do consider it a valid proof of concept! And a successful project, of course—which is important when [I can’t spend much time on computers outside of work these days.](https://belkadan.com/blog/2021/07/Keyboard-Pants/)

[The output from the video above is on Google Sheets](https://docs.google.com/spreadsheets/d/1_K4gMtS0GGviPAIFkhGZmXXFXvuaAatxcx2ulM1XZXk/edit), though you’ll have to make your own copy if you want to “run” it. It adds its two inputs, then doubles them until the result is greater than 50\. (Which is about all the current implementation knows how to handle.)

### Future Directions: alloca

One thing that’s *not* implemented in this proof-of-concept is alloca, i.e. local variables. This is both inconvenient, because that’s the default for a non-optimized build, and a definite missing piece for *truly* compiling LLVM to a spreadsheet. The thing is, LLVM’s load and store instructions are *definitely* imperative, in a way that spreadsheets aren’t. So to actually represent a memory location, we’d probably need to express a load as “the value of the most recent store with a matching location”, similar to how we represented a phi as “one of several values based on the most recent basic block”. There’s probably some trickiness around loops as well—maybe the “program counter” actually should count instructions and not just basic blocks, so “most recent” can include “but not in my future”.

Once here, it’s still a jump to *arbitrary* memory allocation, but maybe not as big of one as it could be. As long as stores are broken up into individual fields, we could say that column H represents heap memory, and column I the instant when it was last modified. This would be a formula involving *every store instruction in the program,* but it might work. I haven’t tried to work out the details, though.

### Future Directions: A Call Stack

Another major omission is function calls. For non-recursive functions this is mostly a weird kind of branch/phi combination, but for recursive functions we have a problem: all our local SSA variables need to do double duty! I don’t have a good idea of how to do this one short of using *columns* to represent stack frames. (In which a stack overflow would be running out of columns that have the right formulas.) That would also be neat because you could *see the call stack,* but I haven’t thought through if it would actually work.

This entry was posted on [December](https://belkadan.com/blog/2023/12) 28, [2023](https://belkadan.com/blog/2023) and is filed under [Technical](https://belkadan.com/blog/technical). Tags: [Compilers](https://belkadan.com/blog/tags/compilers), [LLVM](https://belkadan.com/blog/tags/llvm), [Spreadsheets](https://belkadan.com/blog/tags/spreadsheets), [Source code](https://belkadan.com/blog/tags/source-code)