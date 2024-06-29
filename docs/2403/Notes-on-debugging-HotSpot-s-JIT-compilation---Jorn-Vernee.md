<!--yml

分类：未分类

日期：2024-05-29 12:42:16

-->

# 关于调试HotSpot的JIT编译的注释 | Jorn Vernee

> 来源：[https://jornvernee.github.io/hotspot/jit/2023/08/18/debugging-jit.html](https://jornvernee.github.io/hotspot/jit/2023/08/18/debugging-jit.html)

通常情况下，当您编译Java程序时，它首先由Java编译器编译为字节码。然而，这个字节码尚未经过优化。在HotSpot（OpenJDK的JVM）中，这些优化是在运行时进行的，由JIT（即时编译器）完成。这种做法允许JIT充分利用代码运行的条件，如硬件，甚至在一定程度上特定输入到程序中的数据。

理解JIT编译器的工作在尝试理解Java程序的性能特征时非常重要。在本博文中，我将展示几种调试JIT编译器操作的技术。这不是针对初学者的文章。不过，这可能会给初学者一些关于他们可能想要学习的内容的启发。如果你想了解更多关于某个主题的信息，你需要自行研究。

1.  [设置舞台](#1-setting-the-stage)

1.  [获取编译方法的汇编](#2-getting-the-assembly-of-a-compiled-method)

1.  [打印内联跟踪](#3-printing-inlining-traces)

1.  [深入研究编译命令](#4-a-closer-look-at-compile-commands)

1.  [追踪逃逸对象](#5-tracking-down-escaping-objects)

1.  [使用本地调试器调试编译](#6-debugging-compilation-using-a-native-debugger)

## 1\. 设置舞台

首先，让我们建立一个小型测试项目，我们可以轻松修改以测试不同的Java代码片段。我们的目标是能够触发特定Java代码片段的JIT编译，以便我们可以使用一些HotSpot的调试编译选项。

我定义了一个‘负载’方法，它将包含我们希望进行JIT编译的代码。然后，我们通过多次调用该方法来简单地触发这个负载的JIT编译。这是代码：

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

Tier 4 JIT编译由C2 JIT编译器完成，在x64平台上的最高/最优化层级，在10K次调用后发生。这个数字可以在`./src/hotspot/cpu/x86/c2_globals.hpp`文件的`CompileThreshold` [1](https://github.com/openjdk/jdk/blob/752121114f424d8e673ee8b7bb85f7705a82b9cc/src/hotspot/cpu/x86/c2_globals_x86.hpp#L41)找到。它也是一个虚拟机标志，因此阈值也可以从命令行配置。我在这里调用了20K次负载，以确保安全，因为调用计数器不一定是100%准确的。

我只是用`javac`将程序编译成字节码。然后，在运行它时，需要传递一些重要的标志。首先是：`-XX:CompileCommand=dontinline,TestJIT::payload`。这个标志禁用了`payload`方法的内联。这样做很重要，可以单独编译`payload`方法，这是必需的，以便能够独立于`main`中的循环来检查该特定方法的编译。

我们需要传递的另一个重要标志是`-Xbatch`。 JIT编译默认是在后台线程中完成的。这意味着您的代码可以继续运行，而编译在进行时也会发生，但在我们的情况下，这也意味着代码可能在编译完成之前就运行完毕，这意味着我们无法调试编译。`-Xbatch`使得请求编译的线程在编译进行时停止。顺便说一句，`-Xbatch`是`-XX:-BackgroundCompilation`的别名，即关闭后台编译 [2](https://github.com/openjdk/jdk/blob/752121114f424d8e673ee8b7bb85f7705a82b9cc/src/hotspot/share/runtime/globals.hpp#L272-L274)。

如果您使用这2个标志（以及任何其他必需的标志）运行测试程序，输出目前还不是很有趣：

```
>  java  -cp  classes  -XX:CompileCommand=dontinline,TestJIT::payload  -Xbatch  TestJIT  CompileCommand:  dontinline  TestJIT.payload  bool  dontinline  =  true 
```

接下来，让我们看看如何开始使用这个测试设置来从有效负载方法的编译中获取有趣的信息。

## 2\. 获取编译方法的汇编

JIT编译器输出机器码，这种格式可以说是难以阅读的。幸运的是，这种机器码可以被反汇编成更易读的格式，其中每个CPU指令用助记符表示。要做到这一点，需要一个名为`hsdis`的HotSpot反汇编插件。您可以通过[OpenJDK构建说明](https://github.com/openjdk/jdk/blob/master/src/utils/hsdis/README.md)或者我的[之前关于这个主题的帖子](/hsdis/2022/04/30/hsdis.html)找到如何构建hsdis的指导。我推荐使用基于Capstone的hsdis，因为它最容易构建，也是我在使用的。我已经将hsdis库文件放在了`PATH`上，这样HotSpot可以自动加载它。

现在，我们再次运行程序，使用几个额外的 VM 标志来打印出 `payload` 方法的汇编代码。我使用 `-XX:CompileCommand=print,TestJIT::payload` 来打印出汇编代码，并且还添加了 `-XX:-TieredCompilation` 这个标志，用于禁用 HotSpot C1 编译器的编译。后者用于减少输出量。在这种情况下，我只关注由 C2 JIT 生成的汇编代码，这是更优化的版本。相反地，如果您只关注由 C1 生成的汇编代码，可以使用 `-XX:TieredStopAtLevel=3` 来禁用第四层编译，即 C2\. 最后，我更喜欢 Intel 汇编语法，所以我还传递了 `-XX:PrintAssemblyOptions=intel`。这个最后的标志是诊断性的，所以我们还必须在 `PrintAssemblyOptions` 标志之前传递 `-XX:+UnlockDiagnosticVMOptions`。将所有内容整合在一起，我们运行程序如下：

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

请注意，我正在使用 PowerShell。反引号是 PowerShell 中相当于 Unix Shell 中的 `\`。我将这个命令放在脚本文件中，以便更容易编辑和重新运行。

生成的输出是与架构相关的。我使用的是 x64 机器，这给我提供了 x64 汇编。如果您使用的是 AArch64 机器，例如 Apple 的 M1，输出将不同。我上面测试程序的输出如下：

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

需要关注的是输出的 `[Disassembly]` 部分。尽管我们的 `payload` 方法是空的，但 JIT 生成了相当多的代码。当然，我们正在运行在虚拟机中，并且需要一些额外的代码使其工作。如果我们只关心为 `payload` 方法内容生成的汇编代码，那么上述大部分内容都与此无关。然而，我会先浏览一遍，以便知道哪些部分通常可以忽略：

```
0x000001c6bacb0c00:   sub             rsp, 0x18
0x000001c6bacb0c07:   mov             qword ptr [rsp + 0x10], rbp 
```

设置堆栈帧。在线程的堆栈上分配一些内存，并将 `rbp` 寄存器的内容保存到堆栈上。

```
0x000001c6bacb0c0c:   cmp             dword ptr [r15 + 0x20], 0
0x000001c6bacb0c14:   jne             0x1c6bacb0c43 
```

`NMethod` 进入屏障。‘nmethod’ 是 HotSpot 中已编译的 Java 方法的名称。`nmethod` 进入屏障需要使一些 GC 工作。我现在不会深入讨论这个问题。

```
0x000001c6bacb0c1a:   add             rsp, 0x10
0x000001c6bacb0c1e:   pop             rbp 
```

清理帧。

```
0x000001c6bacb0c1f:   cmp             rsp, qword ptr [r15 + 0x378]
                                                            ;   {poll_return}
0x000001c6bacb0c26:   ja              0x1c6bacb0c2d 
```

安全点轮询。VM 需要线程偶尔轮询安全点。在安全点，当前线程的 JVM 状态是完全已知且可恢复的。这是代码中的一个点，VM 可能希望检查线程以执行各种 VM 操作。

返回指令

我现在要忽略其余部分。重要的部分是，`payload` 方法内容的代码主要在此块中找到，介于 `nmethod` 进入屏障和帧清理之间。当尝试找到片段生成的代码时，一个好策略是查找 `nmethod` 进入屏障并从那里向前工作，或者查找返回指令并从那里向后工作。

修改我们的 `payload` 方法，并查看生成的汇编代码会发生什么。我将修改我的 payload 方法，将两个数相加并返回结果：

```
public static int payload(int a, int b) {
    return a + b;
} 
```

我只需在 `main` 方法中使用一些虚拟参数调用它。由于我们已禁用 `payload` 方法的内联，所以参数的实际值对我们来说并不重要，因此 JIT 编译器无法“看到”参数的实际值，并且必须假定它们可以是任意值。出于类似的原因，在 `main` 方法中安全地丢弃返回值是可以的：

```
for (int i = 0; i < 20_000; i++) {
    payload(1, 2);
} 
```

使用 `javac` 重新编译，并使用上述标志重新运行程序，可以得到这段汇编：

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

我只复制了相关部分，从方法入口到返回指令。注意前几行，告诉我们参数传递到哪些寄存器中。HotSpot JIT 使用自定义的 Java 方法调用约定。因此，这可能与例如 C 使用的调用约定不同：

```
 # parm0:    rdx       = int
  # parm1:    r8        = int 
```

代码 `return a + b;` 的汇编可以在 nmethod 入口屏障和帧清理之间找到：

```
 0x0000016ad9230c1a:   lea             eax, [rdx + r8] 
```

一种聪明的方法是在单个指令中将两个值相加，并将结果存储在 `eax` 寄存器中，这是 Java 编译调用约定中用于返回整数值的寄存器。

这应该给你一个开始分析特定 Java 代码片段由 JIT 编译器生成的代码的基本概念。

这里需要注意的是，此汇编是使用发布版本生成的，即您可以从 [jdk.java.net](https://jdk.java.net/) 下载的版本。HotSpot 还有 'fastdebug' 和 'slowdebug' 版本。在打印汇编时，使用调试版本可以打印更详细的信息。如果您能够获取 'fastdebug' 版本，则建议至少使用这个版本。获取 'fastdebug' 版本的最简单方法是按照 [构建指南](https://github.com/openjdk/jdk/blob/master/doc/building.md) 中的步骤构建 JDK。对于 'fastdebug' 构建，只需确保使用 `--with-debug-level=fastdebug` 配置构建即可。

## 3\. 打印内联跟踪

JIT 编译器可以提供的下一个有用信息是内联跟踪。此跟踪指示编译的 Java 代码调用的方法是否被内联。内联是一种重要的优化，允许进行其他优化，因此在尝试理解 Java 代码片段的性能时，了解哪些方法被内联可能很重要。虽然此信息在汇编中也是存在的，但内联跟踪提供了一个更好的高级概述。

首先，让我们修改我们的 payload 方法来调用另一个方法：

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

我们可以使用另一个`CompileCommand`选项为此编译生成一个内联跟踪。而不是使用`-XX:CompileCommand=print,TestJIT::payload`打印汇编，我将在该标志中将`print`选项更改为`PrintInlining`，这将给我们提供一个内联跟踪：`-XX:CompileCommand=PrintInlining,TestJIT::payload`。我得到的输出只是：

```
@ 0   TestJIT::otherMethod (4 bytes)   inline (hot)
  @ 0   TestJIT::yetAnotherMethod (3 bytes)   inline (hot) 
```

内联跟踪的格式实际上并未有详细记录，不幸的是。理解它的最佳信息源是在[`CompileTask::print_inlining_inner`](https://github.com/openjdk/jdk/blob/bcba5e97857fd57ea4571341ad40194bb823cd0b/src/hotspot/share/compiler/compileTask.cpp#L412)中找到的源代码。内联跟踪中的每一行将指示一个成功内联的方法，或者一个失败内联的方法。目前，成功和失败之间的差异并未很好地指示（我希望将来改变），我们必须解释行末的消息以决定内联是否成功。在这种情况下，消息是`inline (hot)`，从中我们可以确定内联成功。该行还列出了被内联的方法的名称，以及调用该方法的字节码位置在调用方法中的位置。在这种情况下，在`payload`中对`otherMethod`的调用位于bci（字节码索引）`0`，而在`otherMethod`中对`yetAnotherMethod`方法的调用也位于bci `0`。行的缩进表示发生的内联的‘级别’。由于`yetAnotherMethod`方法是通过`otherMethod`间接内联的，其行缩进了额外的2个空格。

看看如果我们使用`-XX:CompileCommand=dontinline,TestJIT::otherMethod`标志禁用`otherMethod`的内联会发生什么。现在内联跟踪看起来像这样：

```
@ 0   TestJIT::otherMethod (4 bytes)   disallowed by CompileCommand 
```

这应该涵盖了内联跟踪的基础知识。

一些调试配置污染的注意事项：例如，当调用虚方法时会发生分析。JVM会记录接收器的类型，如果始终是两种类型之一，那么C2 JIT可以使用内联缓存内联方法调用。配置文件附加到特定的字节码上，这意味着当我们在某些经常共享的代码中看到虚方法调用站点时，看到了很多不同的接收器类型，C2可能无法内联此类方法调用。这有时被称为配置污染：调用的配置有许多不同的接收器类型。

内联跟踪对诊断性能污染很有用。污染的方法将显示为内联跟踪中的`virtual call`。然而，为了获得准确的性能分析（性能分析也发生在较低的层次），重要的是再次打开分层编译，这样可以使用`-XX:+TieredCompilation`（将`-`替换为`+`）。但是，这也会导致内联跟踪包含来自多个编译的跟踪。为了能够区分它们，我们可以使用`-XX:CompileCommand=PrintCompilation,TestJIT::payload`，这将在特定编译的内联跟踪开头输出一些关于编译的信息，使跟踪更容易区分（例如` 2591 308 b 4 AbsMapProfiling::payload (139 bytes)`后跟随该编译的内联跟踪）。对于最相关的信息，您可能希望查看该方法最后一次编译的跟踪。

## 4\. 更近距离查看编译命令

到目前为止，你可能已经注意到`-XX:CompileCommand=...`选项有多么有用了。这个命令用于基于每个方法来控制编译器设置。要获取有关`CompileCommand`标志的更多信息，我们可以使用：

```
java  -XX:CompileCommand=help 
```

这将输出标准的Java帮助信息，但是如果你在控制台输出中向上滚动，你也应该能找到`CompileCommand`的帮助信息。这描述了标志的语法，并列出了可以与该标志结合使用的所有选项。许多标志也在[`./src/hotspot/share/compiler/compilerDirectives.hpp`文件](https://github.com/openjdk/jdk/blob/bcba5e97857fd57ea4571341ad40194bb823cd0b/src/hotspot/share/compiler/compilerDirectives.hpp)中列出。

除了在命令行上指定这些编译命令之外，还可以通过json文件指定它们，这样稍微增加了灵活性，可以应用于哪个编译器。有关此更多信息，请参见[JEP](https://openjdk.org/jeps/165)

如果您查看compilerDirectives文件，您可能会注意到一些选项仅在非产品构建中可用。

## 5\. 追踪逃逸对象

对于这一部分，我将使用HotSpot的‘fastdebug’构建。如果您想跟进，您将无法使用发布版本构建。

当使用`-prof gc`运行[JMH](https://github.com/openjdk/jmh)基准测试时，您可能会遇到这种情况。您的基准测试如下：

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

而在使用`-prof gc`运行时，您会看到一些分配发生：

```
MyBenchmark.testMethod:gc.alloc.rate.norm  avgt   50    96.000 ±   0.001    B/op 
```

每次操作分配了96字节。但是，HotSpot有逃逸分析，可以消除分配，并且据我们所见，没有逃离基准方法的分配的对象。那么，为什么我们仍然看到分配？为了调查这个问题，我们将使用`TraceEscapeAnalysis`编译命令（在发布版本中不可用）。该命令会打印逃逸分析算法的跟踪，我们可以用它来追踪逃逸的对象以及原因。

让我们从修改我们的有效载荷开始，包括基准代码：

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

请注意，我已将有效载荷方法转换为实例方法，以使其工作。我只需创建TestJIT类的一个实例，并在其上调用有效载荷方法：

```
TestJIT recv = new TestJIT();
for (int i = 0; i < 20_000; i++) {
    recv.payload();
} 
```

为了获取逃逸分析跟踪，我将基本命令中的`PrintInlining`选项更改为`TraceEscapeAnalysis`：`-XX:CompileCommand=TraceEscapeAnalysis,TestJIT::payload`。我还将输出重定向到文件`... > EA.txt`，因为输出内容相当长。

我获得的输出非常长，我这里不会全部包含。要注意查找类似以下内容的行：

```
+++++ Initial worklist for virtual void TestJIT.payload() (ea_inv=0) 
```

请注意结尾的`ea_inv=0`。逃逸分析可以运行多次迭代。但是为了找到逃逸对象，只有最后一次迭代是相关的。因此，我只搜索`ea_inv`，并找到最后一次迭代，即`ea_inv=1`，并删除该之前的所有跟踪内容。

其余跟踪应分为两部分：初始工作列表，其开始由上述文本行标记，然后是实际的计算跟踪，由以下文本行标记：

```
+++++ Calculating escape states and scalar replaceability 
```

下一步是在初始工作列表中查找我们的对象分配。对象分配在C2中表示为`Allocate`节点，因此我们可以查找字符串`'Allocate ==='`。确保只查看初始工作列表中的分配。对于我展示的测试程序，应有2个分配：

```
JavaObject(9) NoEscape(NoEscape) [ [ 37 ]]     25  Allocate  === 5 6 7 8 1 (23 21 22 1 1 10 1 1 1 ) [[ 26 27 28 35 36 37 ]]  rawptr:NotNull ( int:>=0, java/lang/Object:NotNull *, bool, top, bool ) TestJIT::payload @ bci:0 (line 12) !jvms: TestJIT::payload @ bci:0 (line 12)
JavaObject(10) NoEscape(NoEscape) [ [ 102 ]]     90  Allocate  === 39 36 63 8 1 (88 87 22 1 1 10 1 1 1 42 1 42 ) [[ 91 92 93 100 101 102 ]]  rawptr:NotNull ( int:>=0, java/lang/Object:NotNull *, bool, top, bool ) TestJIT$Scope::<init> @ bci:5 (line 20) TestJIT::payload @ bci:4 (line 12) !jvms: TestJIT$Scope::<init> @ bci:5 (line 20) TestJIT::payload @ bci:4 (line 12) 
```

这些分配最初是不逃逸的，但在随后的逃逸分析中发现它们在某个时刻逃逸了。在Allocate节点的调试字符串中，我们可以看到分配发生在代码中的位置：`TestJIT::payload @ bci:0 (line 12)`和`TestJIT$Scope::<init> @ bci:5 (line 20)`。所以我们有2个分配，一个在第12行的有效载荷方法中，另一个在第20行的Scope构造函数中。这些是行：

```
try (Scope scope = new Scope()) { 
```

并且：

```
final List<Runnable> resources = new ArrayList<>(); 
```

正是我们在使用`new`操作符的地方！

现在，让我们试着找出为什么这些对象会逃逸。我将从搜索“在‘计算逃逸状态…’消息中查找JavaObject(9)”开始。我找到了这个：

```
JavaObject(9) NoEscape(NoEscape) -> NoEscape(GlobalEscape) propagated from: LocalVar(28) ...
JavaObject(9) NoEscape(GlobalEscape) NSR -> ArgEscape(GlobalEscape) propagated from: LocalVar(28) ... 
```

我们可以看到JavaObject(9)的状态更新为ArgEscape(GlobalEscape)，这使其不可替换为标量。这也由“NSR”（不可标量替换）表示。为了找到原因，我们必须沿着日志中的状态更新链追溯到根源。在这条消息中，我们可以看到状态是从LocalVar(28)传播而来。当我搜索它时，我找到了这个：

```
LocalVar(28) NoEscape(NoEscape) -> NoEscape(GlobalEscape) propagated from: LocalVar(41) ... 
```

即`LocalVar(28)`的状态本身是从`LocalVar(41)`传播而来。如果我继续追踪链条，最终会到达：

```
LocalVar(41) ArgEscape(ArgEscape) -> ArgEscape(GlobalEscape) escapes as arg to: 1020  CallStaticJava  === 414 407 408 8 1 (42 1 1 417 1 ) [[ 1021 1022 1023 ]] # Static  TestJIT$Scope::close void ( TestJIT$Scope (java/lang/AutoCloseable):NotNull * ) TestJIT::payload @ bci:25 (line 12) !jvms: TestJIT::payload @ bci:25 (line 12) 
```

即我们的对象通过对`TestJIT$Scope::close`的不合适调用而逃逸！而且，如果我们对`JavaObject(10)`执行相同的过程，我们最终也会到达同样的调用。因此，这两个对象都通过此调用逃逸。

我会说，跟踪日志的过程有些乏味。幸运的是，我最近编写了一个可以解析追踪并报告有关逃逸对象信息的脚本。您可以在[这里](https://cr.openjdk.org/~jvernee/TraceEAParser.java)找到它。如果我在追踪上调用该脚本：`java .\TraceEAParser.java EA.Txt`，我会得到以下结果：

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

对逃逸分配的一个很好的总结（如果我可以这么说的话 ;))。同样，我们可以看到这里有2个逃逸分配，它们通过对`TestJIT$Scope::close`的调用逃逸。

现在，关于为什么这个不合适的调用在这里，我们可以生成一个内联追踪，如[第 3 节](#3-printing-inlining-traces)所示。如果我们在追踪中查找`TestJIT$Scope::close`，我们可以找到这样的内容：

```
@ 25   TestJIT$Scope::close (39 bytes)   already compiled into a medium method 
```

不幸的是，`TestJIT$Scope::close`没有被内联，这使得我们的对象逃逸了。事实上，我们刚刚诊断了[JDK-8267532](https://bugs.openjdk.org/browse/JDK-8267532)，截至撰写时还没有解决方案。但是，我们现在至少知道对象逃逸的原因*为什么*。在其他情况下，这可能是一个更有用的工具，用来追踪问题代码并应用临时修复。但是，不幸的是，在处理性能时，彩虹的尽头并不总是有一锅金子。

## 6\. 使用本地调试器调试编译

最后，让我们打开最终的逃逸通道：在编译期间直接调试源代码。通过调试器逐步执行源代码是检查JIT编译器正在执行的操作的终极工具，当其他方法失效时。为此，我们需要一个‘slowdebug’构建（以类似于快速调试构建的方式获取）。这是必需的，以便VM实际上可以使用本地调试工具进行调试而不会出现太多问题。

要设置VM的调试，我首先通过在JDK构建系统中运行`make vscode-project`来设置一个VSCode项目。这将生成一个`./build/<config>/jdk.code-workspace`文件，我可以通过`File -> Open Workspace from File...`在VSCode中打开。

接下来，在我们的测试程序的`main`方法的开头添加这两行代码：

```
System.out.println("pid: " + ProcessHandle.current().pid());
System.in.read(); 
```

我打印出当前进程的ID，然后通过从标准输入读取来‘等待’。

接下来，我添加 `-XX:CompileCommand=BreakAtCompile,TestJIT::payload,true` 命令行标志，这将在编译载荷方法时停止本地调试器。如果我运行程序，控制台会打印出进程号。此时我可以通过 VSCode 附加调试器。我转到‘Run and Debug’选项卡，并点击窗口左上方的小齿轮。这应该打开一个带有几个运行/调试配置的 .json 文件。底部右侧应该有一个‘Add Configuration…’选项，通过它我可以为要使用的调试器添加配置。我正在使用来自 `C/C++` VSCode 插件的 `C/C++: (Windows) Attach` 配置。添加运行配置只需要一次，之后我只需点击窗口左上角的带有‘(Windows) Attach’运行配置的小绿色箭头即可。

使用打印的进程号附加本地调试器后，在测试程序控制台按‘enter’以使其继续执行，VSCode 窗口弹出并将我扔到 HotSpot 编译器代码的某个位置。此时我可以在编译器代码中设置其他断点，并开始调试载荷方法的编译。

这应该给你一个如何使用本地调试器调试编译的基本概念。

## 结论

目前就这些了。希望展示我使用的一些调试技术能为你自己调试 HotSpot 的 JIT 编译器提供一些灵感。

## 感谢阅读
