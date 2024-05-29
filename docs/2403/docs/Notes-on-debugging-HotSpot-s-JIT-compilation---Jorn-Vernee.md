<!--yml
category: 未分类
date: 2024-05-29 12:42:16
-->

# Notes on debugging HotSpot’s JIT compilation | Jorn Vernee

> 来源：[https://jornvernee.github.io/hotspot/jit/2023/08/18/debugging-jit.html](https://jornvernee.github.io/hotspot/jit/2023/08/18/debugging-jit.html)

Typically when you compile a Java program, it is first compiled into bytecode by a Java compiler. However, this bytecode is not optimized yet. In HotSpot (OpenJDK’s JVM) this instead happens at runtime, and is done by a JIT, a Just-In-Time compiler. This way of doing things allows the JIT to take maximum advantage of the conditions that the code is running in, such as hardware, and even to a certain degree the specific data that is fed into the program.

Understanding what the JIT compilers do is important when trying to understand the performance characteristics of a Java program. In this blog post I will show off several techniques that I use to debug what a JIT compiler is doing. This is not a post focussed on beginners. Though, it might give some inspiration to a beginner for things they might want to learn about. If you wish to know more about a certain topic, you’ll have to research it yourself.

1.  [Setting the stage](#1-setting-the-stage)
2.  [Getting the assembly of a compiled method](#2-getting-the-assembly-of-a-compiled-method)
3.  [Printing inlining traces](#3-printing-inlining-traces)
4.  [A closer look at compile commands](#4-a-closer-look-at-compile-commands)
5.  [Tracking down escaping objects](#5-tracking-down-escaping-objects)
6.  [Debugging compilation using a native debugger](#6-debugging-compilation-using-a-native-debugger)

## 1\. Setting the stage

First, let’s set up a small test project that we can easily modify to test different snippets of Java code. Our goal is to be able to trigger a JIT compilation for a particular piece of Java code, so that we can use some of HotSpot’s options for debugging the compilation.

I define a ‘payload’ method, which will hold the code that we wish to JIT compile. Then, we trigger the JIT compilation of this payload simply by invoking the method a bunch of times. Here is the code:

```
public class TestJIT {
    public static void main(String[] args) {
        for (int i = 0; i < 20_000; i++) {
            payload();
        }
    }

    public static void payload() {
        // test code here
    }
} 
```

Tier 4 JIT compilation, done by the C2 JIT compiler, which is the highest/most optimized tier, happens after 10K invocations on x64\. The number can be found in the `./src/hotspot/cpu/x86/c2_globals.hpp` file as `CompileThreshold` [1](https://github.com/openjdk/jdk/blob/752121114f424d8e673ee8b7bb85f7705a82b9cc/src/hotspot/cpu/x86/c2_globals_x86.hpp#L41). It is also a VM flag, so the threshold can be configured from the command line as well. I’m invoking the payload 20K times here to be on the safe side, since the invocation counter is not necessarily 100% accurate.

I just compile the program to bytecode with `javac`. Then, when running it, there are a few important flags to pass. First: `-XX:CompileCommand=dontinline,TestJIT::payload`. This flag disables inlining of `payload`. Doing this is important in order to get a standalone compilation of the `payload` method, which is required to be able to inspect the compilation of that particular method in isolation from the loop in `main`.

The other important flag we need to pass is `-Xbatch`. JIT compilation is by default done in a background thread. This means that your code can just keep running while the compilation happens, but in our case it also means that the code might finish running before the compilation is done, meaning we would not be able to debug the compilation. `-Xbatch` makes it so the thread that requests a compilation is stopped while the compilation is happening. FWIW, `-Xbatch` is an alias for `-XX:-BackgroundCompilation`, i.e. turning off background compilation [2](https://github.com/openjdk/jdk/blob/752121114f424d8e673ee8b7bb85f7705a82b9cc/src/hotspot/share/runtime/globals.hpp#L272-L274)

If you run the test program with these 2 flags (and any other needed flags), the output is not very interesting yet:

```
>  java  -cp  classes  -XX:CompileCommand=dontinline,TestJIT::payload  -Xbatch  TestJIT  CompileCommand:  dontinline  TestJIT.payload  bool  dontinline  =  true 
```

Next, let’s see how we can start using this test setup to get interesting information out of the compilation of the payload method.

## 2\. Getting the assembly of a compiled method

The JIT compilers output machine code, which is, let’s say, a hard to read format. Luckily this machine code can be dis-assembled into a more readable format where each CPU instruction is represented with a mnemonic. To do that, a dis-assembler plugin for HotSpot is needed called `hsdis`. You can find instructions on how to build hdis through the [OpenJDK build instructions](https://github.com/openjdk/jdk/blob/master/src/utils/hsdis/README.md), or through my [previous post on the topic](/hsdis/2022/04/30/hsdis.html). I recommend using the capstone-based hsdis, as it’s easiest to build, and also what I’m using. I’ve put the hsdis library file on the `PATH`, where it can be loaded automatically by HotSpot.

Now we just run the program again with a couple of additional VM flags to print out the assembly of the `payload` method. I’m using `-XX:CompileCommand=print,TestJIT::payload` to print out the assembly, and I’m also adding `-XX:-TieredCompilation` which disables compilation by the HotSpot C1 compiler. This latter flag is used to reduce the amount of output. In this case I’m only interested in the assembly generated by C2 JIT, which is the more optimized version. Conversely, if you’re only interested in the assembly generated by C1, you can use `-XX:TieredStopAtLevel=3` which disables tier 4 compilation, i.e. C2\. Lastly, I prefer the intel assembly syntax, so I also pass `-XX:PrintAssemblyOptions=intel`. This last flag is diagnostic, so we also have to pass `-XX:+UnlockDiagnosticVMOptions` *before* the `PrintAssemblyOptions` flag. Putting it all together, we run the program as follows:

```
java  `
  -cp  classes  `
  -Xbatch  `
  -XX:-TieredCompilation  `
  -XX:CompileCommand=dontinline,TestJIT::payload  `
  -XX:CompileCommand=print,TestJIT::payload  `
  -XX:+UnlockDiagnosticVMOptions  `
  -XX:PrintAssemblyOptions=intel  `
  TestJIT 
```

Note that I’m using powershell. The ticks are the powershell equivalent of `\` in a unix shell. I’ve put this command in a script file to make it easier to edit and re-run.

The generated output is architecture specific. I’m using an x64 machine, which gives me x64 assembly. If you are using an AArch64 machine, such as Apple’s M1, the output will be different. The output I get for my test program above as follows:

```
CompileCommand: dontinline TestJIT.payload bool dontinline = true
CompileCommand: print TestJIT.payload bool print = true

============================= C2-compiled nmethod ==============================
----------------------------------- Assembly -----------------------------------

Compiled method (c2)      54   13             TestJIT::payload (1 bytes)
 total in heap  [0x000001c6bacb0a90,0x000001c6bacb0ca0] = 528
 relocation     [0x000001c6bacb0bd8,0x000001c6bacb0be8] = 16
 main code      [0x000001c6bacb0c00,0x000001c6bacb0c50] = 80
 stub code      [0x000001c6bacb0c50,0x000001c6bacb0c68] = 24
 oops           [0x000001c6bacb0c68,0x000001c6bacb0c70] = 8
 scopes data    [0x000001c6bacb0c70,0x000001c6bacb0c78] = 8
 scopes pcs     [0x000001c6bacb0c78,0x000001c6bacb0c98] = 32
 dependencies   [0x000001c6bacb0c98,0x000001c6bacb0ca0] = 8

[Disassembly]
--------------------------------------------------------------------------------
[Constant Pool (empty)]

--------------------------------------------------------------------------------

[Verified Entry Point]
  # {method} {0x000001c6d88002e8} 'payload' '()V' in 'TestJIT'
  #           [sp+0x20]  (sp of caller)
  0x000001c6bacb0c00:   sub             rsp, 0x18
  0x000001c6bacb0c07:   mov             qword ptr [rsp + 0x10], rbp
  0x000001c6bacb0c0c:   cmp             dword ptr [r15 + 0x20], 0
  0x000001c6bacb0c14:   jne             0x1c6bacb0c43
  0x000001c6bacb0c1a:   add             rsp, 0x10
  0x000001c6bacb0c1e:   pop             rbp
  0x000001c6bacb0c1f:   cmp             rsp, qword ptr [r15 + 0x378]
                                                            ;   {poll_return}
  0x000001c6bacb0c26:   ja              0x1c6bacb0c2d
  0x000001c6bacb0c2c:   ret
  0x000001c6bacb0c2d:   movabs          r10, 0x1c6bacb0c1f  ;   {internal_word}
  0x000001c6bacb0c37:   mov             qword ptr [r15 + 0x390], r10
  0x000001c6bacb0c3e:   jmp             0x1c6bac7ad80       ;   {runtime_call SafepointBlob}
  0x000001c6bacb0c43:   call            0x1c6bac586e0       ;   {runtime_call StubRoutines (2)}
  0x000001c6bacb0c48:   jmp             0x1c6bacb0c1a
  0x000001c6bacb0c4d:   hlt
  0x000001c6bacb0c4e:   hlt
  0x000001c6bacb0c4f:   hlt
[Exception Handler]
  0x000001c6bacb0c50:   jmp             0x1c6baca3f00       ;   {no_reloc}
[Deopt Handler Code]
  0x000001c6bacb0c55:   call            0x1c6bacb0c5a
  0x000001c6bacb0c5a:   sub             qword ptr [rsp], 5
  0x000001c6bacb0c5f:   jmp             0x1c6bac7a020       ;   {runtime_call DeoptimizationBlob}
  0x000001c6bacb0c64:   hlt
  0x000001c6bacb0c65:   hlt
  0x000001c6bacb0c66:   hlt
  0x000001c6bacb0c67:   hlt
--------------------------------------------------------------------------------
[/Disassembly] 
```

The thing to focus on is the `[Disassembly]` section of the output. Even though our `payload` method is empty, there is still quite a bit of code generated by the JIT. We are of course running in a virtual machine, and there is some additional code needed to make it work. If we’re just interested in the assembly generated for the contents of the `payload` method, then most of the above is irrelevant. However, I will walk through it once so that we know which parts we can typically ignore:

```
0x000001c6bacb0c00:   sub             rsp, 0x18
0x000001c6bacb0c07:   mov             qword ptr [rsp + 0x10], rbp 
```

Setting up the stack frame. Allocates a bit of memory on the thread’s stack, and saves the contents of the `rbp` register to the stack.

```
0x000001c6bacb0c0c:   cmp             dword ptr [r15 + 0x20], 0
0x000001c6bacb0c14:   jne             0x1c6bacb0c43 
```

NMethod entry barrier. ‘nmethod’ is the name for a compiled Java method in HotSpot. The nmethod entry barrier is needed to make some GCs work. I won’t get into that right now.

```
0x000001c6bacb0c1a:   add             rsp, 0x10
0x000001c6bacb0c1e:   pop             rbp 
```

Cleaning up the frame.

```
0x000001c6bacb0c1f:   cmp             rsp, qword ptr [r15 + 0x378]
                                                            ;   {poll_return}
0x000001c6bacb0c26:   ja              0x1c6bacb0c2d 
```

Safepoint poll. The VM needs threads to occasionally poll for safe points. At a safe point, the JVM state for the current thread is fully known and recoverable. This is a point in the code where the VM might want to inspect the thread to do various VM operations.

Return instruction

I’m going to ignore the rest for now. The important part is that the code for the contents of the `payload` method will be primarily found in this block, between the nmethod entry barrier and the cleanup of the frame. A good strategy when trying to find the generated code for a snippet is to look for the nmethod entry barrier and work forwards from there, or to look for the return instruction and work backwards from there.

Let’s modify our `payload` method and see what happens to the generated assembly. I’m going to change my payload method to add two numbers together and return the result:

```
public static int payload(int a, int b) {
    return a + b;
} 
```

I just call it with some dummy arguments in the `main` method. The values don’t really matter since we have disabled inlining of the `payload` method, so the JIT compiler will not be able to ‘see’ the actual value of the arguments, and has to assume that they could be anything. For similar reasons, it is safe to discard the return value in the `main` method:

```
for (int i = 0; i < 20_000; i++) {
    payload(1, 2);
} 
```

Recompiling with `javac`, and re-running the program with the flags above, gets me this bit of assembly:

```
 # {method} {0x0000016af6c002f0} 'payload' '(II)I' in 'TestJIT'
  # parm0:    rdx       = int
  # parm1:    r8        = int
  #           [sp+0x20]  (sp of caller)
  0x0000016ad9230c00:   sub             rsp, 0x18
  0x0000016ad9230c07:   mov             qword ptr [rsp + 0x10], rbp
  0x0000016ad9230c0c:   cmp             dword ptr [r15 + 0x20], 0
  0x0000016ad9230c14:   jne             0x16ad9230c47
  0x0000016ad9230c1a:   lea             eax, [rdx + r8]
  0x0000016ad9230c1e:   add             rsp, 0x10
  0x0000016ad9230c22:   pop             rbp
  0x0000016ad9230c23:   cmp             rsp, qword ptr [r15 + 0x378]
                                                            ;   {poll_return}
  0x0000016ad9230c2a:   ja              0x16ad9230c31
  0x0000016ad9230c30:   ret 
```

I’ve just copied the relevant bits here, starting from the method entry to the return instruction. Note the first few lines, which tells us in which registers the arguments are being passed. The HotSpot JITs use a custom calling convention for Java methods. So, this can be different from the calling convention used by e.g. C:

```
 # parm0:    rdx       = int
  # parm1:    r8        = int 
```

The assembly for the code `return a + b;` can be found between the nmethod entry barrier and the frame cleanup:

```
 0x0000016ad9230c1a:   lea             eax, [rdx + r8] 
```

A clever way of adding two values together in a single instruction, and storing the result in the `eax` register, which is the register in which integer values are returned in the Java compiled calling convention.

That should give you a basic idea of what is needed to start analysing the code generated by the JIT compilers for a particular snippet of Java code.

A thing to note here is that this assembly was generated with a release build, i.e. the ones that you can download from [jdk.java.net](https://jdk.java.net/). There are also ‘fastdebug’ and ‘slowdebug’ versions of HotSpot. Using a debug build can result in more detailed information being printed when printing assembly. If you can get your hands on a fastdebug build, I recommend using at least that when printing assembly. The most straightforward way of getting a fastdebug build is building the JDK yourself. This is relatively easy, if you follow the steps in the [build guide](https://github.com/openjdk/jdk/blob/master/doc/building.md). For a fastdebug build, just make sure to configure the build with `--with-debug-level=fastdebug`.

## 3\. Printing inlining traces

The next useful bit of information we can get from a JIT compiler is an inlining trace. This trace indicates whether methods being called by the compiled Java code were inlined. Inlining is an important optimization that allows other optimizations to take place, so understanding which methods are inlined can be important when trying to understand the performance of a snippet of Java code. While this information is technically present in the assembly as well, the inlining trace gives a much nicer high-level overview.

To start, let’s modify our payload method to call another method:

```
public static int yetAnotherMethod() {
    return 42;
}

public static int otherMethod() {
    return yetAnotherMethod();
}

public static int payload() {
    return otherMethod() + 1;
} 
```

We can generate an inlining trace for this compilation using another `CompileCommand` option. Instead of using `-XX:CompileCommand=print,TestJIT::payload` to print the assembly, I’m going to change the `print` option in that flag to `PrintInlining`, which will give us an inlining trace: `-XX:CompileCommand=PrintInlining,TestJIT::payload`. The output I get is simply:

```
@ 0   TestJIT::otherMethod (4 bytes)   inline (hot)
  @ 0   TestJIT::yetAnotherMethod (3 bytes)   inline (hot) 
```

The format of an inlining trace is not really documented, unfortunately. The best source of information to understanding it is the source code found in [`CompileTask::print_inlining_inner`](https://github.com/openjdk/jdk/blob/bcba5e97857fd57ea4571341ad40194bb823cd0b/src/hotspot/share/compiler/compileTask.cpp#L412). Each line in the inlining trace will indicate a method that was either inlined successfully, or a method which failed to be inlined. Currently the difference between success and failure is not indicated very well (something I’m hoping to change in the future), and we have to interpret the message at the end of the line to decide whether inlining succeeded or failed. In this case, the message is `inline (hot)`, from which we can ascertain that inlining succeeded. The line also lists the name of the method which was inlined, and the bytecode location at which the method call is located in the caller method. In this case, the call to `otherMethod` in `payload` is located at bci (byte code index) `0`, and the `yetAnotherMethod` method call in `otherMethod` is also located at bci `0`. The indentation of the lines indicates the ‘level’ of inlining that occurred. Since the `yetAnotherMethod` method is inlined transitively through `otherMethod`, its line indented by 2 extra spaces.

Let’s see what happens if we disable inlining of `otherMethod` using the `-XX:CompileCommand=dontinline,TestJIT::otherMethod` flag. Now the inlining trace looks like this:

```
@ 0   TestJIT::otherMethod (4 bytes)   disallowed by CompileCommand 
```

That should cover the basics of inlining traces.

Some notes for debugging profile pollution: profiling happens for instance when a virtual method is called. The JVM will record the type of the receiver, and if it’s always one of two types, the C2 JIT can inline the method call using an inlining cache. Profiles are attached to particular bytecodes, this means that when we have a virtual method call site in some heavily shared code that sees a lot of different receiver types, C2 can fail to inlining such method calls. This is sometimes called profile pollution: the profile for the call is ‘polluted’ with many different receiver types.

Inlining traces can be useful to diagnose profile pollution. Polluted methods will show up as `virtual call` in the inlining trace. It is however important to turn on tiered compilation again for that in order to get an accurate profile (profiling also happens in lower tiers). So, instead of `-XX:-TieredCompilation` we need to use `-XX:+TieredCompilation` (replacing the `-` with a `+`). This will however also make it so the inlining trace contains traces from multiple compilations. To be able to differentiate them, we can use `-XX:CompileCommand=PrintCompilation,TestJIT::payload`, this will output some info about the compilation at the start of an inlining trace for a particular compilation, making the traces much easier to differentiate (e.g. ` 2591 308 b 4 AbsMapProfiling::payload (139 bytes)` followed by the inlining trace for that compilation). For the most relevant information, you probably want to look at the trace for that last compilation of the method.

## 4\. A closer look at compile commands

By now you’ve probably noticed how useful the `-XX:CompileCommand=...` option is. This command is used to control compiler settings on a per-method basis. To get more information about the `CompileCommand` flag, we can use:

```
java  -XX:CompileCommand=help 
```

This will output the standard Java help message, but if you scroll up above that in the console output, you should also find the `CompileCommand` help message. This describes the syntax of the flag, and lists all the options that can be used in combination with the flag. A lot of the flags are also listed in the [`./src/hotspot/share/compiler/compilerDirectives.hpp` file](https://github.com/openjdk/jdk/blob/bcba5e97857fd57ea4571341ad40194bb823cd0b/src/hotspot/share/compiler/compilerDirectives.hpp).

Besides specifying these compile commands on the command line, it’s also possible to specify them through a json file, which gives slightly more flexibility wrt. which compiler the option applies to. More information about that can be found in [the JEP](https://openjdk.org/jeps/165)

If you look at the compilerDirectives file, you might notice that some options are only available in non-product builds.

## 5\. Tracking down escaping objects

For this section, I’m going to be using a ‘fastdebug’ build of HotSpot. You will not be able to use a release build if you wish to follow along.

You might have encountered this when running [JMH](https://github.com/openjdk/jmh) benchmarks with `-prof gc`. You have a benchmark like so:

```
Runnable dummy = () -> {};

static class Scope implements AutoCloseable {
    final List<Runnable> resources = new ArrayList<>();

    void addCloseAction(Runnable runnable) {
        resources.add(runnable);
    }

    @Override
    public void close() {
        for (Runnable r : resources) {
            r.run();
        }
    }
}
@Benchmark
public void testMethod() throws InterruptedException {
    try (Scope scope = new Scope()) {
        scope.addCloseAction(dummy);
    }
} 
```

And when running it with `-prof gc` you see some allocation happening:

```
MyBenchmark.testMethod:gc.alloc.rate.norm  avgt   50    96.000 ±   0.001    B/op 
```

96 bytes per operation are being allocated. But, HotSpot has escape analysis which allows allocations to be eliminated, and as far as we can see there are no objects allocated that escape from the benchmark method. So, why are we still seeing allocations? To investigate this we are going to use the `TraceEscapeAnalysis` compile command (which is not available in release builds). This command prints a trace of the escape analysis algorithm, which we can use to track down which objects escape, and why.

Let’s start by modifying our payload to include the code from the benchmark:

```
 public void payload() {
    try (Scope scope = new Scope()) {
        scope.addCloseAction(dummy);
    }
}

Runnable dummy = () -> {};

static class Scope implements AutoCloseable {
    final List<Runnable> resources = new ArrayList<>();

    void addCloseAction(Runnable runnable) {
        resources.add(runnable);
    }

    @Override
    public void close() {
        for (Runnable r : resources) {
            r.run();
        }
    }
} 
```

Note that I’ve turned the payload method into an instance method, to make that work I’m simply creating an instance of the TestJIT class and invoking the payload method on that:

```
TestJIT recv = new TestJIT();
for (int i = 0; i < 20_000; i++) {
    recv.payload();
} 
```

To get an escape analysis trace, I change the `PrintInlining` option to `TraceEscapeAnalysis` in the base command: `-XX:CompileCommand=TraceEscapeAnalysis,TestJIT::payload`. I also pipe the output to a file `... > EA.txt`, since it’s quite long.

The output I get is long, and I wont include it in full here. The important thing to look for are the lines like this:

```
+++++ Initial worklist for virtual void TestJIT.payload() (ea_inv=0) 
```

Notice the `ea_inv=0` at the end. Escape analysis can run for multiple iterations. To find escaping objects however, only the last iteration is relevant. So, I just search for `ea_inv`, and find the last iteration which is `ea_inv=1`, and delete the rest of the trace before that.

The rest of the trace should be split into 2 parts: the initial work list, whose start is marked by the line of text above, and then the actual calculation trace, which is marked by the following line:

```
+++++ Calculating escape states and scalar replaceability 
```

The next step is to find our object allocations in the initial work list. Object allocations are represented by `Allocate` nodes in C2, so we can look for the string `'Allocate ==='`. Be sure to only look at the allocations in the initial work list. There should be 2 for the test program I’ve shown:

```
JavaObject(9) NoEscape(NoEscape) [ [ 37 ]]     25  Allocate  === 5 6 7 8 1 (23 21 22 1 1 10 1 1 1 ) [[ 26 27 28 35 36 37 ]]  rawptr:NotNull ( int:>=0, java/lang/Object:NotNull *, bool, top, bool ) TestJIT::payload @ bci:0 (line 12) !jvms: TestJIT::payload @ bci:0 (line 12)
JavaObject(10) NoEscape(NoEscape) [ [ 102 ]]     90  Allocate  === 39 36 63 8 1 (88 87 22 1 1 10 1 1 1 42 1 42 ) [[ 91 92 93 100 101 102 ]]  rawptr:NotNull ( int:>=0, java/lang/Object:NotNull *, bool, top, bool ) TestJIT$Scope::<init> @ bci:5 (line 20) TestJIT::payload @ bci:4 (line 12) !jvms: TestJIT$Scope::<init> @ bci:5 (line 20) TestJIT::payload @ bci:4 (line 12) 
```

These allocations start out as non-escaping, but at some point they are discovered to escape during the following escape analysis. In the debug string for the Allocate node, we can see where the allocation is happening in the code: `TestJIT::payload @ bci:0 (line 12)` and `TestJIT$Scope::<init> @ bci:5 (line 20)`. So we have 2 allocations, one in the payload method on line 12, and one in the constructor of `Scope` on line 20\. These are the lines:

```
try (Scope scope = new Scope()) { 
```

And:

```
final List<Runnable> resources = new ArrayList<>(); 
```

Exactly where we are using the `new` operator!

Now, let’s try and find why these objects are escaping. I’ll start by searching for `JavaObject(9)` in the ‘calculating escape states …’ messages. I find this:

```
JavaObject(9) NoEscape(NoEscape) -> NoEscape(GlobalEscape) propagated from: LocalVar(28) ...
JavaObject(9) NoEscape(GlobalEscape) NSR -> ArgEscape(GlobalEscape) propagated from: LocalVar(28) ... 
```

We can see here that the state of `JavaObject(9)` is updated to `ArgEscape(GlobalEscape)`, which makes it not scalar replaceable. This is also indicated by the ‘NSR’ (Not Scalar Replaceable). To find the reason for this, we have to follow back the chain of state updates in the log to the root. In this message we can see the state is propagated from `LocalVar(28)`. When I search for that, I find this:

```
LocalVar(28) NoEscape(NoEscape) -> NoEscape(GlobalEscape) propagated from: LocalVar(41) ... 
```

i.e. the state for `LocalVar(28)` was itself propagated from `LocalVar(41)`. If I keep following the chain back, I eventually get to:

```
LocalVar(41) ArgEscape(ArgEscape) -> ArgEscape(GlobalEscape) escapes as arg to: 1020  CallStaticJava  === 414 407 408 8 1 (42 1 1 417 1 ) [[ 1021 1022 1023 ]] # Static  TestJIT$Scope::close void ( TestJIT$Scope (java/lang/AutoCloseable):NotNull * ) TestJIT::payload @ bci:25 (line 12) !jvms: TestJIT::payload @ bci:25 (line 12) 
```

i.e. our object escapes through an out-of-line call to `TestJIT$Scope::close`! And, if we follow the same process for `JavaObject(10)`, our other allocation, we end up at the same call. So, both objects escape through this call.

I’ll say that the process of following the log is somewhat tedious. Lucky for you readers I’ve recently written a script that can parse the trace and report back information about escaping objects. You can find it [here](https://cr.openjdk.org/~jvernee/TraceEAParser.java). If I invoke that script on the trace: `java .\TraceEAParser.java EA.Txt`, I get the following:

```
Escaping allocations:

JavaObject(9) allocation in: TestJIT::payload @ bci:0 (line 12)
  -> LocalVar(28)
  -> LocalVar(41)
  Reason: EscapesAsArg[callNode=1020  CallStaticJava  === 414 407 408 8 1 (42 1 1 417 1 ) [[ 1021 1022 1023 ]] # Static  TestJIT$Scope::close void ( TestJIT$Scope (java/lang/AutoCloseable):NotNull * ) TestJIT::payload @ bci:25 (line 12) !jvms: TestJIT::payload @ bci:25 (line 12)]

JavaObject(10) allocation in: TestJIT$Scope::<init> @ bci:5 (line 20)
  -> Field(19)
  -> JavaObject(9)
  -> LocalVar(28)
  -> LocalVar(41)
  Reason: EscapesAsArg[callNode=1020  CallStaticJava  === 414 407 408 8 1 (42 1 1 417 1 ) [[ 1021 1022 1023 ]] # Static  TestJIT$Scope::close void ( TestJIT$Scope (java/lang/AutoCloseable):NotNull * ) TestJIT::payload @ bci:25 (line 12) !jvms: TestJIT::payload @ bci:25 (line 12)] 
```

A nice summary of the escaping allocations (if I do say so myself ;)). Again, we can see here that there are 2 escaping allocations which are both escaping through a call to `TestJIT$Scope::close`.

Now, for the reason why this out of line call is here, we can generate an inlining trace, as shown in [section 3](#3-printing-inlining-traces). If we look for `TestJIT$Scope::close` in the trace, we find this:

```
@ 25   TestJIT$Scope::close (39 bytes)   already compiled into a medium method 
```

Unfortunately, `TestJIT$Scope::close` is not being inlined, which makes our objects escape. Actually, we’ve just diagnosed [JDK-8267532](https://bugs.openjdk.org/browse/JDK-8267532) which doesn’t have a solution as of the time of writing. But, we now at least know the reason *why* the objects are escaping. In other cases this might be a more useful tool to track down a problematic piece of code in order to apply a spot fix. But, unfortunately when dealing with performance, there is not always a pot of gold at the end of the rainbow.

## 6\. Debugging compilation using a native debugger

Finally, let’s bust open the ultimate escape hatch: debugging the source code directly during a compilation. Stepping through the source code with a debugger is the ultimate tool for inspecting what a JIT compiler is doing, when all else falls short. For this, we need a ‘slowdebug’ build (obtained in a similar way to a fastdebug build). This is needed so that the VM is actually debugable without too many issues using a native debugging tool.

To set up debugging of the VM, I first set up a VSCode project by running `make vscode-project` in the JDK build system. That should generate a `./build/<config>/jdk.code-workspace` file which I can open in VSCode through `File -> Open Workspace from File...`.

Next, I add these 2 lines of code to the start of the `main` method in our test program:

```
System.out.println("pid: " + ProcessHandle.current().pid());
System.in.read(); 
```

I print out the current process id, and then ‘wait’ by reading from stdin.

Next, I add the `-XX:CompileCommand=BreakAtCompile,TestJIT::payload,true` command line flag, which will stop the native debugger during the compilation of the payload method. If I run the program, I will see the pid printed on the console. At this point I can attach a debugger through VSCode. I go to the ‘Run and Debug’ tab, and click the little gear in the top left-ish of the window. This should open a .json file with several run/debug configurations. There should be a ‘Add Configuration…’ in the bottom right, through which I can add a configuration for the debugger I want to use. I’m using the `C/C++: (Windows) Attach` configuration which comes from the `C/C++` VSCode plugin. Adding the run configuration is only needed once, after which I can just click the little green arrow in the top left with the ‘(Windows) Attach’ run configuration selected.

After attaching the native debugger using the printed pid, and pressing ‘enter’ in the test program console to make it continue execution, the VSCode window pops back up and throws me somewhere into the HotSpot compiler code. At this point I can set other breakpoints in the compiler code, and begin debugging the compilation of the payload method.

That should give you a basic idea of how to debug a compilation with a native debugger.

## Conclusion

That’s all I have for now. Hopefully showing some of the debugging techniques I use gave you some inspiration for debugging HotSpot’s JIT compilers yourself.

## Thanks for reading