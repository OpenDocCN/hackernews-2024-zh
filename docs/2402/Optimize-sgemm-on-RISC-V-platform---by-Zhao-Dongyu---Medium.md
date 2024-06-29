<!--yml

category: 未分类

date: 2024-05-29 13:26:17

-->

# 在 RISC-V 平台上优化 sgemm | 作者：赵东宇 | Medium

> 来源：[https://medium.com/@zhaodongyu/optimize-sgemm-on-risc-v-platform-b0098630b444](https://medium.com/@zhaodongyu/optimize-sgemm-on-risc-v-platform-b0098630b444)

# **准备**

相关代码位于[./prepare/](https://github.com/Zhao-Dongyu/sgemm_riscv/tree/main/prepare) 。

**测试交叉编译**

我使用**全志麒麟 D1**开发板，并从[这里](https://xuantie.t-head.cn/community/download?id=4090445921563774976)下载了交叉编译链接。

有关详细说明，请参阅[readme](https://github.com/Zhao-Dongyu/sgemm_riscv/blob/main/prepare/README.md)。

**内存带宽测试**

我在开发板上进行了内存带宽测试，使用了以下项目：

**Roofline 模型**

[Roofline](https://en.wikipedia.org/wiki/Roofline_model) 提出了一种使用“**操作强度**”进行量化分析的方法，并提供了一个公式，用于计算计算平台上可达到的理论性能极限。

根据[OpenPPL 公开课 | RISC-V 技术分析](https://zhuanlan.zhihu.com/p/474684731)：

+   D1 的计算能力可达**4 GFlops**（@ 1GHz）。

+   内存速度：**2.727 GB/s**（DDR3 792 MHz）。

尽管我测得的最高速度为**2.592 GB/s**，但可能某处存在问题？暂且相信商汤，暂时接受其值。

# **SGEMM 优化**

相关代码位于[./sgemm/](https://github.com/Zhao-Dongyu/sgemm_riscv/tree/main/sgemm) 。

**用法**

以*step0*为例，您需要首先编辑**Makefile**来配置您的交叉编译链。

```
$ cd sgemm/step0/
$ make
$ adb push test_bl_sgemm_step0.x ./.
$ adb shell './test_bl_sgemm_step0.x'
```

**版本 0: 初始版本**

对我来说，这个版本似乎是**最直观的**，毕竟这是我学习、理解和计算矩阵乘法的方式：

> 将 A 的一行与 B 的一列相乘，得到 C 的一个元素。

```
for ( i = 0; i < m; i ++ ) {              // Start 2-th loop
    for ( j = 0; j < n; j ++ ) {          // Start 1-nd loop
        for ( p = 0; p < k; p ++ ) {      // Start 0-st loop
            C( i, j ) += A( i, p ) * B( p, j );
        }                                 // End   0-th loop
    }                                     // End   1-st loop
}                                         // End   2-nd loop
```

我认为*版本 0*很好地解释了公式 $C_{mn} = \sum_{k=1}^{K} A_{mk}B_{kn}$。

然而，这个版本显然存在明显的缺陷：在理论计算能力为**4 GFLOPS**的平台上，它仅实现了最大计算性能为**0.03 GFLOPS**。这是因为**对矩阵 B 的访问具有非常低的缓存命中率，即“内存访问的空间局部性差”**。在整个计算过程中，相当于多次访问矩阵 B。

建议按顺序访问多维数组的元素。这可以改善内存访问的空间局部性，并使其对缓存更友好。

此外，可以观察到随着尺寸增加，性能波动显著。数据分析显示，当*m=n=k*为128 164 192 228 256 288 320 352 384时，性能较差。这些数字相差32，32 * 4（sizeof(float)) = 128 B。

推测性能波动与**缓存行**和**硬件预取**有关——缓存行 = 64B，在缓存未命中后，硬件预取，即HWPrefetcher，读取一个额外的缓存行。

**版本1：循环互换版本**

在缓存中重复使用数据是对缓存的最基本和有效利用。对于嵌套循环，可以使用*循环互换*、*循环反转*、*循环融合*、*循环分布*、*循环切片*、*循环展开与合并*等方法来提高程序性能。

选择适当的循环转换方法既可以保持程序的语义，又可以提高其性能。

```
for ( i = 0; i < m; i ++ ) {              // Start 2-th loop
    for ( p = 0; p < k; p ++ ) {          // Start 1-st loop
        for ( j = 0; j < n; j ++ ) {      // Start 0-nd loop
            C( i, j ) += A( i, p ) * B( p, j );
        }                                 // End   0-th loop
    }                                     // End   1-st loop
}                                         // End   2-nd loop
```

与*版本0*相比，*版本1*对矩阵B的操作具有更好的空间局部性，并且性能得到了大幅提升（尤其是对于更大的尺寸，而对于 m=n=k <= 68，*版本0*的效率更高）。

调整m、n和k的顺序**不会**影响**结果**（即保持程序的语义），但会影响**性能**。

测试不同的循环顺序的性能（以全志 摩尔 Nezha D1平台，m=n=k=512为例）

然而，*版本1*的硬件利用率仍然很低，需要进一步优化。

**版本2：阻塞版本**

```
for ( i = 0; i < m; i += DGEMM_MR ) {          // Start 2-nd loop
    for ( j = 0; j < n; j += DGEMM_NR ) {      // Start 1-st loop
        AddDot_4x4_opt( k, &A( i, 0 ), lda, &B( 0, j ), ldb, &C( i, j ), ldc );
    }                                          // End   1-st loop
}                                              // End   2-nd loop
```

为了避免不必要的缓存交换，进行阻塞处理。[讨论为什么阻塞矩阵优化有效](https://zhuanlan.zhihu.com/p/342923482)是一篇很棒的文章，我建议从中学习。

在*版本2*中进行块操作后，性能仍然不尽人意。这是因为，虽然这个版本表面上实现了阻塞逻辑，但在块内的计算中仍然有一些小技巧尚未应用。

**版本3：阻塞优化版本**

已添加**AddDot_4x4_opt**。

[**BLISlab-tutorial**](https://github.com/flame/blislab/blob/master/tutorial.pdf)中提到了几个技巧：

> 2.4.2 循环展开
> 
> 每次通过内部循环更新循环索引i和指针cp会造成相当大的开销。因此，编译器会执行循环展开。
> 
> 2.4.3 寄存器变量
> 
> 请注意，只有在数据存储在寄存器中时才能进行计算。编译器会自动转换代码，以便插入放置某些数据在寄存器中的中间步骤。

使用这些技巧后，这个版本的性能显著提高了！

但是，对于更大的矩阵大小，这个版本的性能仍然相对较低。经过调查，例如，在访问B[0,0]、B[0,1]、B[0,2]、B[0,3]之后，在访问B[1,0]时，当大小很大时，必定会发生**缓存未命中**。因此，提前重新排列数据将是很好的。

**版本4：B预打包版本**

假设矩阵B是**参数**，所以我们可以提前进行*打包*操作。*版本4*提前打包矩阵B，从而进一步提高了性能！

性能改善的原因显而易见：访问矩阵B时缓存未命中显著减少。这是我第一次深刻意识到在模型推断之前**预打包神经网络权重**的重要性。

当大小较大时，可以看出性能仍然下降。这可能是由于访问矩阵A时高缓存未命中率造成的。我们是否应该对A进行打包？

我假设矩阵A是**输入**，因此无法提前对A进行打包，必须包含在总体计时中。这是必要的吗？

**第五版：A包 & B预包版**

基于*第四版*，*第五版*在矩阵A上执行打包。

在这里，由于矩阵A被假设为输入，因此在计算过程中需要对A进行打包，并且这种时间消耗需要计入计时中。

结果仍然令人满意，尤其是在大矩阵尺寸下，进一步实现了性能提升。

我最初以试错的心态接近这个实验，考虑**额外读取A**和**写入packA**。看起来未来的主要挑战在于对抗缓存未命中。

当前的优化方向已经达到了极限。值得尝试在计算过程中进行一些**预加载**。

接下来，我们将转向汇编，进行向量计算，并在汇编中实现**预加载**。

**第六版：汇编版**

简要解释：A**未打包**，但B预先使用了16个数字**预打包**。

```
for ( i = 0; i < m; i += DGEMM_MR ) {       // Start 2-nd loop
    int mb = DGEMM_MR;
    if((m - i) < DGEMM_MR) mb = m - i; 
    for ( j = 0; j < n; j += DGEMM_NR ) {   // Start 1-st loop
        int nb = DGEMM_NR;
        if((n - j) < DGEMM_NR) nb = n - j; 
        RvvSgemm4x16(   nb,                 // nr <= 16, a0
                        mb,                 // mr <= 4,  a1
                        k,                  // astride = k*sizeof(float), a2
                        &A[i * k],          // mr * k,   a3
                        &packB[j * k],      // k * 16,   a4
                        &C( i, j ),         // mr * nr,  a5
                        n * sizeof(float),  // Len(N) * sizeof(float), a6
                        bias
                    );
    }                                       // End   1-st loop
}                                           // End   2-nd loop
```

关于使用rvv指令，我认为**vsetvli**至关重要，**vfmacc.vf**是主要支柱。

我从[OpenPPL课程 | RISC-V技术分析](https://zhuanlan.zhihu.com/p/474684731)中学到了很多。他们真的很专业！我推荐从他们那里学习理论指导和知识点，向OpenPPL致敬！

至于汇编操作符，在汇编中有许多细节，我强烈抱怨：**写汇编真的很烦人！特别是调试过程，简直是折磨。**我上次写汇编是在本科课程期间。再次拾起它带来了一些新奇和兴奋感，能够在非常精细的粒度上控制操作符的执行给人一种巨大的成就感。

关于如何具体实现汇编文件，我认为最快的方法是查看汇编代码。这里不再详细解释。

需要注意的是，这个版本的性能非常差。为什么？这是另一个**循环顺序**的问题。

**第七版：循环顺序交换的汇编版**

简要解释：A**未打包**，但B预先使用了16个数字**预打包**。

```
for ( j = 0; j < n; j += DGEMM_NR ) {       // Start 2-st loop
    int nb = DGEMM_NR;
    if((n - j) < DGEMM_NR) nb = n - j; 
    for ( i = 0; i < m; i += DGEMM_MR ) {   // Start 1-nd loop
        int mb = DGEMM_MR;
        if((m - i) < DGEMM_MR) mb = m - i; 
        RvvSgemm4x16(   nb,                 // nr <= 16, a0
                        mb,                 // mr <= 4,  a1
                        k,                  // astride = k*sizeof(float), a2
                        &A[i * k],          // mr * k,   a3
                        &packB[j * k],      // k * 16,   a4
                        &C( i, j ),         // mr * nr,  a5
                        n * sizeof(float),  // Len(N) * sizeof(float), a6
                        bias
                    );
    }                                       // End   1-st loop
}                                           // End   2-nd loop
```

将循环顺序颠倒，从**n方向**开始，然后是**m方向**，显著改善了性能。

然而，大尺寸矩阵的性能仍然不是很好。根本原因仍在于**内存访问**。在屋顶线模型中，大尺寸矩阵的计算被认为是**计算受限**的，理想情况下，**计算时间**和**内存访问时间**应尽可能重叠。目前，大量时间花费在内存访问上（主要是由于**缓存未命中**！）。

**版本 8：带有预载入的汇编版本**

简要解释：矩阵 A **未打包**，而矩阵 B 经历了包含 16 个元素的**预打包**，并包括**预载入**操作。

性能爆炸性增长！达到最大**2.212 GFLOPS**。

核心操作：

```
vfmacc.vf v16,  ft0, v0
vlw.v v4, (bp0)         # b0'->v4
flw fs4, 384(bp0)       # pre-load B
addi bp0,bp0,64
vfmacc.vf v20,  ft1, v0
```

在**vfmacc.vf**指令之间插入一些**load**操作，将稍后将使用的数据预先加载到**缓存**中，显著减少**缓存未命中**。

最初，我感到困惑 —— 当代码似乎按顺序执行时，**计算时间**和**内存访问时间**如何重叠？直到后来我才理解这里的实质，这在**缓存行**的原理中。的确，基础知识至关重要！

**版本 9：带有 A Packed 的汇编版本**

基于之前的经验，尝试对矩阵 A 进行**打包**，但令人惊讶的是，结果并不理想。简要分析表明，这个版本的汇编代码中矩阵 A 的预载入可能并未优化。

在前一个版本中，虽然 A 没有打包，但 A 的 4 行进行了预载入，这也解决了矩阵 A 缓存未命中的痛点。
