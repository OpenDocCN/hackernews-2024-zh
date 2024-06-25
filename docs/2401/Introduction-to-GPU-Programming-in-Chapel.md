<!--yml

category: 未分类

时间：2024-05-27 14:50:54

-->

# Chapel 中 GPU 编程简介

> 来源：[`chapel-lang.org/blog/posts/intro-to-gpus/`](https://chapel-lang.org/blog/posts/intro-to-gpus/)

Chapel 是一种用于高效并行计算的编程语言。近年来，并行计算的一个特定子领域变得异常流行：GPU 计算。因此，Chapel 团队一直在努力增加对 GPU 的支持，使得创建厂商中立且性能优异的 GPU 程序变得轻松。自首次发布以来，Chapel 编译器的这一方面已经迅速改进，并在时间上不断接收增强和性能修复。本教程将介绍 Chapel 的 GPU 编程功能。

虽然诸如 CUDA 和 HIP 等框架通常用于编写 GPU 程序，但本教程并不假设读者熟悉它们。示例将利用 Chapel 的一些通用功能，并在此过程中进行解释。想要更加深入地了解 Chapel，请参阅本博客上的 Advent of Code 系列，或查看 [Learning Chapel](https://chapel-lang.org/learning.html) 页面以获取更多资源。

在本文中，我们将直接使用 Chapel 的 GPU 编程功能。如果你对 GPU 适用于解决哪些问题感兴趣，请查看 *Appendix* 部分包含此信息。

### Locales 和 `on` 语句：Chapel 的基础

当 GPU 参与时，代码可能在不同的地方执行：它可能在 CPU 上执行，就像大多数程序员习惯的那样，也可能在 GPU 上执行。GPU 和 CPU 有很大的不同；对其中一个很合适的代码可能对另一个不合适。Chapel 通过其 *locales* 概念使程序员能够控制其代码在何处运行。引用自 [Chapel 规范](https://chapel-lang.org/docs/primers/locales.html)：

> 在 Chapel 中，`locale` 类型指的是程序运行所在的机器资源单元。

换句话说，locale 是计算机上可以运行代码的部分；这可能代表一个 GPU 或一个 CPU。当给定一个 locale 时，Chapel 的 *`on` 语句* 可以用于明确声明特定代码应该在何处执行。例如，下面的代码计算并打印出前十个偶数，将计算在第一个 GPU locale 上执行。

|

```
 1 2 3 4 5 6 7 8 9 10 11 12 
```

|

```
config  const  n  =  5;  // use 5-element arrays in examples by default, for brevity  on  here.gpus[0]  {  var  A:  [1..n]  int;  // declare an array with n elements // (to benefit from GPUs, you'd probably want n >> 5) foreach  i  in  1..n  do A[i]  =  i  *  2;   writeln("The whole array A is: ",  A); for  i  in  1..n  do writeln("A[",  i,  "] = ",  A[i]); } 
```

|

代码以一个针对 GPU *子区域* 的 `on` 块开始。此块引用 `here`，一个特殊的变量，指的是当前运行代码的区域。当启用 GPU 支持时，区域包括一个名为 `gpus` 的字段，该字段是一个子区域数组，每个子区域代表一个已安装的 GPU。在具有单个 GPU 的区域上，`here.gpus` 将是一个单元素数组。在具有多个 GPU 的区域上（这是 [注意：我之前提到超级计算机是有原因的。 Chapel 的 GPU 支持已在大型机器上进行了测试，包括 Frontier，目前发布时 TOP500 列表中唯一的 Exascale 超级计算机。]），`here.gpus` 将具有与 GPU 数量相同的元素。因此，`here.gpus[0]` 是机器的第一个 GPU。

<details><summary>**（如何启用 GPU 支持？）**</summary>

要启用 GPU 支持，Chapel 必须使用设置为 `gpu` 的 `CHPL_LOCALE_MODEL` 环境变量构建。在 Bash 会话中，可以如下设置该变量：

```
export CHPL_LOCALE_MODEL=gpu 
```

如果你有 NVIDIA GPU，这应该足以编译和运行启用 GPU 的程序。对于 AMD GPU，你还需要使用 `CHPL_GPU_ARCH` 环境变量指定 GPU 的架构：

```
export CHPL_GPU_ARCH=your_arch_here 
```

最后，即使你没有 GPU，Chapel 也提供了一种称为 'cpu-as-device' 的模式。在此模式下，你仍然会得到一个 `here.gpus` 数组，并且可以编写针对 GPU 的代码；然而，你的计算机 CPU 将用于执行所有代码。这使得可以在没有 GPU 的情况下开发支持 GPU 的程序。设置 `CHPL_GPU` 环境变量为 `cpu` 就可以启用 'cpu-as-device' 模式：

请参阅关于 [构建 Chapel](https://chapel-lang.org/docs/usingchapel/building.html) 的页面以及 [GPU 技术说明](https://chapel-lang.org/docs/technotes/gpu.html) 以获取有关与 GPU 相关的环境变量的更多信息。</details>

该代码通过对索引从一到五进行迭代来生成偶数，并将每个数字乘以二。GPU 擅长并行解决相同的子问题；使每个数字加倍可以成为其自己的子问题，使示例中的循环非常适合在 GPU 上执行。乘法循环使用 `foreach` 关键字编写，告诉 Chapel 可以安全地并行执行。语言会处理剩下的事情。Chapel 还有传统的 `for` 循环，就像示例中的第二个循环一样。我们将在后面进一步讨论两者之间的区别。

在示例中，`foreach` 循环在 GPU 上运行，而第 9 行的 `writeln` 并没有。与乘法不同，向控制台打印单个字符串并不适合在 GPU 上执行。通常，适合在 GPU 上执行的代码可以分解为许多相似且独立的部分。对于循环来说，这转化为 *无序性*。如果循环是无序的，则不会有任何迭代影响其他迭代。回顾我们的示例，让我们再次检查乘法循环：

```
 foreach  i  in  1..n  do A[i]  =  i  *  2; 
```

我们可以观察到`5*2`的结果不会影响`3*2`的计算。每次迭代访问`A`的不同元素，因此不存在数据竞争。因此，我们的示例是无序循环的一个实例。Chapel 让程序员指示哪些循环具有这个特性；为了断言循环是无序的，应该写成 [注意：Chapel 还具有`forall`循环。这些循环允许被遍历的数据结构决定如何并行迭代。Chapel 标准库提供的数据结构足够智能，可以利用无序性，因此，在 GPU 环境中合格的`forall`循环也会在 GPU 上执行。

查看[循环入门](https://chapel-lang.org/docs/main/primers/loops.html)以了解有关 Chapel 各种类型循环的更多信息。

很容易看出为什么无序循环很适合在 GPU 上执行。如果循环的迭代顺序无关紧要，那么我们可以将每个迭代视为要交给 GPU 核心处理的独立子问题。这一观察是 Chapel GPU 支持的基础：**无序循环可以在 GPU 上并行执行**。实际上，Chapel 会自动将无序循环转换为 GPU 代码，只要可能。

<details><summary>**（我了解 CUDA/HIP。你能告诉我更多关于它是如何工作的吗？）**</summary>

在 CUDA 和 HIP 中，编写启用 GPU 的程序通常涉及创建一个标记为`device`或`global`的函数，并在核心启动中使用此函数。在底层，Chapel 也是这样做的。

当 Chapel 遇到一个适合 GPU 的循环时，它会将其主体转换为一个函数，命名为类似`chpl_gpu_kernel_filename_linenumber`的函数。如果循环主体/新定义的核函数包含对其他函数或方法的调用，Chapel 还会生成这些函数的`device`版本。

Chapel 在原始循环旁边插入一个核心启动。由于相同的代码可以从`here.gpus[0]`（GPU）或默认的环境（CPU）执行，Chapel 也保留了循环。因此，在 GPU 上执行核心启动，在 CPU 上则退回到循环。

相比之下，示例中的第二个循环是*顺序相关的*。

```
 for  i  in  1..n  do writeln("A[",  i,  "] = ",  A[i]); 
```

我们特意希望按顺序打印`A`的元素。因此，打印第五个元素的迭代必须发生在打印第四个之后；存在依赖关系。在 Chapel 中，需要按顺序执行迭代的循环（称为*串行循环*）使用`for`关键字编写。使用`for`编写的循环不被编译器视为 GPU 执行的对象。

现在我们几乎审查了示例的每个部分。还有一个重要的部分：我们还声明了一个包含偶数的数组`A`：

```
on  here.gpus[0]  {  var  A:  [1..n]  int;   // ... the rest of the even numbers example } 
```

`on` 语句的一个重要方面是，在 `on` 块内部声明的变量在逻辑上位于语句指定的区域上。通常，这意味着从同一区域访问这些变量会更快，而从其他区域访问会更慢。这里有一个例子：

```
on  firstLocale  {  var  A:  [0..10]  int; A[0]  =  1;  // Cheap to access 'A': // 'A' is accessed from the same locale that it lives on  on  anotherLocale  { A[0]  =  2;  // More expensive to access 'A': // 'A' is accessed from another locale } } 
```

[注意：未来，Chapel 的目标是使 GPU 代码能够访问在 GPU 区域外声明的数组。但是，这将需要 GPU 发起的 GPU 和 CPU 之间的一些通信。类似的 *GPU 驱动通信* 已经在我们的路线图上，但在撰写本文时不受支持。] 为了让在 GPU 上运行的代码（如我们的 `foreach` 循环）可以访问数组，该数组必须位于 GPU 区域上。这就是为什么我们在 `on` 块内部声明 `A`，而不是在外部声明它的原因。如果 `A` 是在 `on` 块外部声明的，它将位于 CPU 区域上，分配在 CPU 内存中。

现在我们已经介绍了整个入门示例，演示了如何使用 `on` 块和 `foreach` 循环在 Chapel 中针对 GPU 进行定位。然而，这个例子是一个非常简单的程序，旨在介绍 Chapel 的 GPU 支持的关键概念。到目前为止，我们所看到的只是开始。在下一节中，我们将看看与 GPU 编程无缝结合的 Chapel 的其他功能。

### 你还能在 GPU 上做什么？

Chapel 语言的大部分功能都可以在 GPU 上运行。在本节中，我将快速浏览一下可以完成的工作。要了解我将展示的个别语言功能，请参阅本文开头提到的资源。

当我们逐个例子进行时，读者可能会想知道：我们如何确保代码是在 GPU 上运行的？跟踪代码在哪里运行是一个稍微高级的话题。为了简单起见，我将定义一个自定义函数，`numKernelLaunches`。 *内核* 一词通常指在 GPU 上运行的代码片段。 *内核启动* 是 CPU 启动在 GPU 上执行内核的过程。你不必理解我的辅助函数是如何工作的；把 `numKernelLaunches` 想象成返回自上次检查以来启动的内核总数就足够了。我将使用这个函数来确保代码在我们期望的时候在 GPU 上运行。

<details><summary>**(我想看看函数是如何定义的！)**</summary>

|

```
14 15 16 17 18 19 20 21 22 23 24 25 26 27 28 
```

|

```
use  GpuDiagnostics;   proc  startCountingKernelLaunches()  {  resetGpuDiagnostics(); startGpuDiagnostics(); }   proc  numKernelLaunches()  {  stopGpuDiagnostics(); var  result  =  +  reduce  getGpuDiagnostics().kernel_launch; startCountingKernelLaunches(); return  result; }   startCountingKernelLaunches(); 
```

|</details>

本节中的所有示例都将在 GPU 区域上执行。

让我们从一些有趣的事情开始。在本文的开头，我们使用 GPU 通过 `foreach` 将一些数字乘以两个。我们可以使用 Chapel 的一个特性称为 [*提升*](https://chapel-lang.org/docs/users-guide/datapar/promotion.html) 来更简洁地完成这个任务。提升允许我们将一个数组传递给需要标量值的操作。当我们这样做时，操作会自动在数组的每个元素上执行。这个操作会自动并行执行 —— 可能在 GPU 上。

如果我们的操作是“乘以二”，我们可以编写如下代码：

|

```
32 33 34 35 36 
```

|

```
 var  Evens  =  2  *  [1,2,3,4,5]; writeln(Evens);   // One kernel launch from the promoted initializer assert(numKernelLaunches()  ==  1); 
```

|

仅用一行简单的代码，我们就能编写在 GPU 上运行的代码。

让我们继续下一个示例。我们可以从在 GPU 上运行的代码中调用大多数 Chapel 函数。以下示例在区间\([0, 2\pi)\)内的十个增量上对内置正弦函数`sin`进行采样：

|

```
38 39 40 41 42 43 44 45 46 
```

|

```
 use  Math;  // Include the 'Math' module for access to 'sin' and 'pi'  const  numSamples  =  10; var  A  =  [i  in  0..#numSamples]  sin(2  *  pi  *  i  /  numSamples);   writeln(A);   // One kernel launch from the loop expression initializer assert(numKernelLaunches()  ==  1); 
```

|

从 GPU 调用的函数可以是用户自定义的，也可以是任意复杂的。在以下示例中，我们再次使用提升来计算前 20 个斐波那契数。这个例子【注：该示例在算法和 GPU 特定意义上都未经过优化。

在算法方面，熟悉算法分析的人可能会知道，存在一个 \(O(n)\) 算法来计算第 \(n\) 个斐波那契数。与此同时，我们的朴素算法是 \(O(2^n)\) ——相当糟糕。此外，我们的实现没有使用任何记忆化技术，在数组元素之间执行了大量冗余计算。

在 GPU 特定方面，我们正在编写将导致*线程分歧*的代码。每个线程将执行略有不同的步骤序列，这使得 GPU 难以执行它们。] 但是作为使用核函数中的任意函数的良好示例。

|

```
48 49 50 51 52 53 54 55 56 57 
```

|

```
 proc  fib(x:  int):  int  { if  x  <=  1  then  return  1; return  fib(x-1)  +  fib(x-2); }   var  Fibs  =  fib(0..#20); writeln(Fibs);   // One kernel launch from the promoted expression in the initializer assert(numKernelLaunches()  ==  1); 
```

|

这里，`fib`是一个普通的 Chapel 函数，通过使用整数范围`0..#20`来进行提升。请注意，我们可以在 GPU 核函数中使用它，而不需要进行任何特殊处理。总的来说，在 Chapel 中，一旦定义了函数，就可以从 GPU 和 CPU 中调用它。

循环也可以作为核函数的一部分执行。对于我们的最后一个示例，我们将定义一个二维数组`Square`，然后在 GPU 上对其每一列求和。以下代码将初始化、填充和打印这个新的方阵数组：

|

```
59 60 61 62 63 64 65 66 67 68 
```

|

```
 var  rows,  cols  =  1..5; var  Square:  [rows,  cols]  int; foreach  (r,  c)  in  Square.indices  do Square[r,  c]  =  r  *  10  +  c;   writeln("Original array:"); writeln(Square);   // Two kernel launches: one from initializing Square, one from the loop assert(numKernelLaunches()  ==  2); 
```

|

拿着新的`Square`数组，我们继续对其列求和。在`foreach`循环内部，我们像往常一样使用另一个`for`循环。

|

```
70 71 72 73 74 75 76 77 78 79 80 81 82 83 
```

|

```
 var  ColSums:  [cols]  int; foreach  c  in  cols  do  { var  sum  =  0;   for  r  in  rows  do sum  +=  Square[r,  c];   ColSums[c]  =  sum; } writeln("Column sums:"); writeln(ColSums);   // Two kernel launches: one from initializing ColSums, one from the loop assert(numKernelLaunches()  ==  2); 
```

|

经过几个示例后，我们可以结束 `on` 块，并返回到在 CPU 上计算。

|

```
85 
```

|

```
}  // end of `on here.gpus[0]` 
```

|

现在我们已经看到了几个使用 Chapel 的 GPU 支持的示例计算。我们可以在哪里运行所有这些代码？让我们接下来谈谈这个问题。

### 我可以在哪里运行 Chapel 代码以使用 GPU？

你可能还记得介绍中提到 Chapel 的 GPU 支持是供应商中立的。这意味着可以在 NVIDIA 和 AMD 的 GPU 上执行启用了 GPU 的 Chapel 代码，而无需进行任何修改！这包括本文中提到的所有代码。

实际上，你甚至不需要 GPU。Chapel 支持一种称为“CPU 作为设备”的模式，它允许针对 GPU 的代码在 CPU 上透明地执行。你可以在没有专用图形处理器的笔记本电脑上原型化启用了 GPU 的代码，然后随时切换到启用了 GPU 的机器。事实上，这就是撰写本文的方式。

最后，Chapel 的 GPU 支持与语言的其余部分一样，是可扩展的。在笔记本电脑上原型化的程序可以轻松地在超级计算机上执行，并获得良好的性能。尽管以可扩展的方式编写代码需要注意[注意：不仅使用 `here.gpus[0]` 可以访问第一个 GPU。]，但 Chapel 让创建能够在任何地方运行的并行代码变得轻松。

### 总结

Chapel 的 GPU 支持更深入，但这可能是介绍性文章的一个好的停顿点 —— 我们已经涵盖了很多内容！让我们回顾一下我们已经涵盖的一些内容：

+   Chapel 的*locale*表示可以运行代码和存储变量的机器的部分。

+   `on` 语句指定代码应该在哪里执行，包括在 GPU 上。

+   使用 `foreach` 编写的*无序*循环会自动在 GPU 上执行。

+   Chapel 语言的大部分功能都可以在 GPU 代码中使用。

    +   这包括*推广*，函数（普通和递归）以及循环。

+   所有这些都是与供应商无关的；Chapel 可与 NVIDIA 和 AMD 的 GPU 配合使用。

如果您想查看有关 Chapel 的 GPU 支持的更多信息，特别是 [技术说明](https://chapel-lang.org/docs/technotes/gpu.html) 包含了许多 GPU 代码的细节和示例，以及 Chapel 1.32 发行说明的 [**GPU 编程**部分](https://chapel-lang.org/releaseNotes/1.31-1.32/05-gpus.pdf) 包含了“GPU 编程速成课程”。

当然，我们还没有看到在 GPU 上解决问题的实际示例。我们也还没有看到如何分析 Chapel 中启用 GPU 的程序的性能，或者如何提高该性能。最后，我们没有看到 Chapel 的 GPU 支持如何无缝集成到语言的其余部分，以允许编写在所有 GPU **和**计算节点上运行的代码。关于最后一点的一点小提示，可以查看系统中所有 GPU 上运行的“偶数”示例的版本：

|

```
87 88 89 90 91 92 
```

|

```
coforall  loc  in  Locales  do  on  loc  {  coforall  gpu  in  here.gpus  do  on  gpu  { var  Evens  =  2  *  [1,2,3,4,5]; writeln("Even numbers computed on ",  gpu,  ": ",  Evens); } } 
```

|

仅用六行代码，我们编写了一个可以利用整个超级计算机的计算资源的程序。

我们将在随后的文章中回到上述所有主题，从编写多节点、多 GPU 程序开始。敬请关注！

**本文中介绍的整个 Chapel 程序可以在此处查看：**

<details>|

```
 1 2 3 4 5 6 7 8 9 10 11 12 13 14 15 16 17 18 19 20 21 22 23 24 25 26 27 28 29 30 31 32 33 34 35 36 37 38 39 40 41 42 43 44 45 46 47 48 49 50 51 52 53 54 55 56 57 58 59 60 61 62 63 64 65 66 67 68 69 70 71 72 73 74 75 76 77 78 79 80 81 82 83 84 85 86 87 88 89 90 91 92 
```

|

```
config  const  n  =  5;  // use 5-element arrays in examples by default, for brevity  on  here.gpus[0]  {  var  A:  [1..n]  int;  // declare an array with n elements // (to benefit from GPUs, you'd probably want n >> 5) foreach  i  in  1..n  do A[i]  =  i  *  2;   writeln("The whole array A is: ",  A); for  i  in  1..n  do writeln("A[",  i,  "] = ",  A[i]); }   use  GpuDiagnostics;   proc  startCountingKernelLaunches()  {  resetGpuDiagnostics(); startGpuDiagnostics(); }   proc  numKernelLaunches()  {  stopGpuDiagnostics(); var  result  =  +  reduce  getGpuDiagnostics().kernel_launch; startCountingKernelLaunches(); return  result; }   startCountingKernelLaunches();   on  here.gpus[0]  {   var  Evens  =  2  *  [1,2,3,4,5]; writeln(Evens);   // One kernel launch from the promoted initializer assert(numKernelLaunches()  ==  1);   use  Math;  // Include the 'Math' module for access to 'sin' and 'pi'  const  numSamples  =  10; var  A  =  [i  in  0..#numSamples]  sin(2  *  pi  *  i  /  numSamples);   writeln(A);   // One kernel launch from the loop expression initializer assert(numKernelLaunches()  ==  1);   proc  fib(x:  int):  int  { if  x  <=  1  then  return  1; return  fib(x-1)  +  fib(x-2); }   var  Fibs  =  fib(0..#20); writeln(Fibs);   // One kernel launch from the promoted expression in the initializer assert(numKernelLaunches()  ==  1);   var  rows,  cols  =  1..5; var  Square:  [rows,  cols]  int; foreach  (r,  c)  in  Square.indices  do Square[r,  c]  =  r  *  10  +  c;   writeln("Original array:"); writeln(Square);   // Two kernel launches: one from initializing Square, one from the loop assert(numKernelLaunches()  ==  2);   var  ColSums:  [cols]  int; foreach  c  in  cols  do  { var  sum  =  0;   for  r  in  rows  do sum  +=  Square[r,  c];   ColSums[c]  =  sum; } writeln("Column sums:"); writeln(ColSums);   // Two kernel launches: one from initializing ColSums, one from the loop assert(numKernelLaunches()  ==  2);   }  // end of `on here.gpus[0]`  coforall  loc  in  Locales  do  on  loc  {  coforall  gpu  in  here.gpus  do  on  gpu  { var  Evens  =  2  *  [1,2,3,4,5]; writeln("Even numbers computed on ",  gpu,  ": ",  Evens); } } 
```

|</details>

### 附录：哪些问题受益于 GPU？

GPU 和 CPU 之间的一个巨大区别是*核心数*。核心是处理器的一部分，负责执行指令/机器代码。虽然我现在用的电脑大约有 10 个 CPU 核心，但粗略的谷歌搜索显示，NVIDIA RTX 4070 GPU（通过搜索“消费级 NVIDIA GPU”随意选择）有 5888 个核心 — [注：事实上，由于大量并发执行代码的核心数量，GPU 是[大规模并行](https://en.wikipedia.org/wiki/Massively_parallel)架构的一个例子。] 当然，CPU 核心和 GPU 核心并不相同：GPU 核心在个体上往往要弱得多。

对于可以分解为大量[注：对于 GPU，子问题的相似性实际上非常重要。单个 GPU 核心实际上在*同时*解决多个子问题的实例，以锁步方式执行它们的代码。一旦锁步执行不再可能，并且两个子问题需要不同的方法，GPU 就会遇到很大的困难。这个问题被称为*线程分歧*。]片段的问题，拥有大量弱核心是理想的。每个核心可以获得自己问题的一部分，并且可以[注：解决问题时，GPU 核心之间可能有小程度的协调并不少见。GPU 能够分配共享内存，用于核心间的同步执行。然而，这比起这篇入门文章稍微复杂一些。]工作，因为每个子问题都是独立的，所有核心都可以在完全相同的时间工作在各自的份额上。这可能会比单个强大的核心获得非常显著的加速：这样的核心需要逐个处理每个片段。

一个适合在 GPU 上执行的问题的示例是将图像渲染到屏幕上。事实上，正如*图形处理单元*这个名字所暗示的，这正是 GPU 开发出来解决的问题。在渲染时，每个像素的颜色可以相对独立地计算，基于深度和纹理信息；因此，GPU 的每个核心可以同时并行地处理一些像素，大大加速了这个过程。

一个完全适合在 GPU 上执行的问题域是线性代数。在矩阵乘法中，例如，输出矩阵中的每个单元格都可以独立于其他所有单元格进行计算；再一次，这意味着每个 GPU 核心可以同时处理一些单元格。许多东西，包括神经网络，都可以用矩阵运算来表述；快速的线性代数导致更快的机器学习模型。

上述描述应该让你了解到 GPU 可以做些什么。下一个显而易见的问题是我们如何用 Chapel 实现所有这些。这就带我们来到这篇博客的主题：让 Chapel 在 GPU 上运行代码。
