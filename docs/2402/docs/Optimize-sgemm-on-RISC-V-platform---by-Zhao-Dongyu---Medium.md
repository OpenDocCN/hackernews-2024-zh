<!--yml
category: 未分类
date: 2024-05-29 13:26:17
-->

# Optimize sgemm on RISC-V platform | by Zhao Dongyu | Medium

> 来源：[https://medium.com/@zhaodongyu/optimize-sgemm-on-risc-v-platform-b0098630b444](https://medium.com/@zhaodongyu/optimize-sgemm-on-risc-v-platform-b0098630b444)

# **Prepare**

The related code is located in [./prepare/](https://github.com/Zhao-Dongyu/sgemm_riscv/tree/main/prepare) .

**Test Cross-Compilation**

I use the **Allwinner Nezha D1** development board, and downloaded the cross-compilation link from [here](https://xuantie.t-head.cn/community/download?id=4090445921563774976).

For detailed instructions, please refer to the [readme](https://github.com/Zhao-Dongyu/sgemm_riscv/blob/main/prepare/README.md).

**Memory Bandwidth Test**

I conducted memory bandwidth tests on the development board using the following projects:

**Roofline Model**

[Roofline](https://en.wikipedia.org/wiki/Roofline_model) proposes a method for quantitative analysis using “**Operational Intensity**” and provides a formula for the theoretical performance limit achievable on computational platforms.

According to [OpenPPL Public Course | RISC-V Technical Analysis](https://zhuanlan.zhihu.com/p/474684731):

*   The computing power of D1 can reach **4 GFlops** (@ 1GHz).
*   Memory: **2.727 GB/s** (DDR3 792 MHz).

Although I measured the highest as **2.592 GB/s**, there may be some problems somewhere? Let’s trust Sensetime for now, temporarily accept its value.

# **SGEMM Optimization**

Related code is located in [./sgemm/](https://github.com/Zhao-Dongyu/sgemm_riscv/tree/main/sgemm) .

**Usage**

Take *step0* as an example, you need to edit the **Makefile** first to configure your cross-compilation chain.

```
$ cd sgemm/step0/
$ make
$ adb push test_bl_sgemm_step0.x ./.
$ adb shell './test_bl_sgemm_step0.x'
```

**Version 0: Naive Version**

This version seems to be **the most intuitive** to me, after all, this is how I learned, understood, and computed matrix multiplication:

> Multiply one row of A by one column of B to get one element of C.

```
for ( i = 0; i < m; i ++ ) {              // Start 2-th loop
    for ( j = 0; j < n; j ++ ) {          // Start 1-nd loop
        for ( p = 0; p < k; p ++ ) {      // Start 0-st loop
            C( i, j ) += A( i, p ) * B( p, j );
        }                                 // End   0-th loop
    }                                     // End   1-st loop
}                                         // End   2-nd loop
```

I think *version 0* very well explains the formula $C_{mn} = \sum_{k=1}^{K} A_{mk}B_{kn}$.

However, this version has obvious shortcomings: on a platform with a theoretical computing power of **4 GFLOPS**, it only achieves a maximum computational performance of **0.03 GFLOPS**. This is because **the access to matrix B has a very low cache hit rate, i.e., “poor spatial locality”**. Throughout the calculation, it is equivalent to accessing matrix B many, many times.

It is advisable to access the elements of multi-dimensional arrays in sequential order. This can improve the spatial locality of memory access and make it more friendly to the cache.

Furthermore, it can be observed that with the increase in size, the performance fluctuates significantly. Analysis of the data shows that when *m=n=k* is 128 164 192 228 256 288 320 352 384, the performance is poor. These numbers differ by 32, and 32 * 4 (sizeof(float)) = 128 B.

It is speculated that the performance fluctuation is related to **cacheline** and **hardware prefetching** — cacheline = 64B, after cache miss, hardware prefetching, i.e., HWPrefetcher, reads one more cacheline.

**Version 1: Loop Interchange Version**

Reusing data in the cache is the most basic and efficient use of cache. For nested loops, *loop interchange*, *loop reversal*, *loop fusion*, *loop distribution*, *loop tiling*, *loop unrolling and jam*, etc., can be used to improve program performance.

Selecting an appropriate loop transformation method can both maintain the semantics of the program and improve its performance.

```
for ( i = 0; i < m; i ++ ) {              // Start 2-th loop
    for ( p = 0; p < k; p ++ ) {          // Start 1-st loop
        for ( j = 0; j < n; j ++ ) {      // Start 0-nd loop
            C( i, j ) += A( i, p ) * B( p, j );
        }                                 // End   0-th loop
    }                                     // End   1-st loop
}                                         // End   2-nd loop
```

Compared with *version 0*, *version 1* has better spatial locality for the operation on matrix B, and the performance has been greatly improved (especially for larger sizes, while for m = n = k <= 68, the efficiency of *version 0* is higher).

Adjusting the order of m, n, and k does **not** affect the **result** (i.e., maintaining the semantics of the program), but it can affect the **performance**.

Testing the performance of different loop orders (using the Allwinner Nezha D1 platform, with m=n=k=512 as an example)

However, the hardware utilization of *version 1* is still very low, and further optimizations are needed.

**Version 2: Blocking Version**

```
for ( i = 0; i < m; i += DGEMM_MR ) {          // Start 2-nd loop
    for ( j = 0; j < n; j += DGEMM_NR ) {      // Start 1-st loop
        AddDot_4x4_opt( k, &A( i, 0 ), lda, &B( 0, j ), ldb, &C( i, j ), ldc );
    }                                          // End   1-st loop
}                                              // End   2-nd loop
```

To avoid unnecessary cache swapping, blocking processing is performed. [Discussing Why Blocking Matrix Optimization Works](https://zhuanlan.zhihu.com/p/342923482) is a good read, I recommend learning from it.

After performing block operations in *version 2*, the performance is still not satisfactory. This is because, although this version superficially implements blocking logic, there are still some small tricks in the calculation within the block that have not been applied.

**Version 3: Blocked Optimization Version**

**AddDot_4x4_opt** has been added.

Several tricks are mentioned in [**BLISlab-tutorial**](https://github.com/flame/blislab/blob/master/tutorial.pdf):

> 2.4.2 Loop unrolling
> 
> Updating loop index i and the pointer cp every time through the inner loop creates considerable overhead. For this reason, a compiler will perform loop unrolling.
> 
> 2.4.3 Register variables
> 
> Notice that computation can only happen if data is stored in registers. A compiler will automatically transform code so that the intermediate steps that place certain data in registers is inserted.

After using these tricks, this version has significantly improved performance!

However, for larger matrix sizes, the performance of this version is still relatively low. Upon investigation, for example, after accessing B[0,0], B[0,1], B[0,2], B[0,3], when accessing B[1,0], when the size is large, there must be a **cache miss**. Therefore, it would be great if the data could be rearranged in advance.

**Version 4: B prepack Version**

I assume matrix B is **parameter**, so we can perform the *pack* operation in advance. Version 4 **prepack** matrix B, leading to further performance improvement!

The reason for the performance improvement is evident: there is a significant reduction in **cache misses** when accessing matrix B. This is the first time I deeply realized the importance of **prepacking neural network weights** before model inference.

It can be seen that when the size is relatively large, the performance still declines. This should be due to a high number of cache misses when accessing matrix A. Should we pack A?

I assume matrix A is **input**, so packing A cannot be done in advance and must be included in the overall timing. Is it necessary?

**Version 5: A pack & B prepack Version**

Based on *Version 4*, *Version 5* performs packing on matrix A.

Here, since matrix A is assumed to be an input, packing A needs to be performed during computation, and this time consumption needs to be included in the timing.

The results are still pleasing, especially with large matrix sizes, achieving further performance improvements.

I initially approached this experiment with a trial-and-error mindset, considering **the additional read of A** and **writing of packA**. It seems the main challenge ahead lies in combating cache misses.

The current optimization direction has reached its limit. It’s worth trying to do some **preload** during the calculation process.

Next, we’ll move to assembly, work on vector calculations, and implement **preload** in assembly.

**Version 6: Assembly Version**

Brief explanation: A is **not packed**, but B is **prepacked** with 16 numbers.

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

Regarding the use of rvv instructions, I believe **vsetvli** is essential, and **vfmacc.vf** is the mainstay.

I have learned a lot from [OpenPPL Course | RISC-V Technical Analysis](https://zhuanlan.zhihu.com/p/474684731). They are truly professional! I recommend learning theoretical guidance and knowledge points from them, paying tribute to OpenPPL!

As for assembly operators, there are many details in assembly, and I strongly complain: **writing assembly is really annoying! Especially the debugging process, it’s torturous.** The last time I wrote assembly was during my undergraduate classes. Picking it up again brings some novelty and excitement, and being able to control the execution of operators at a very fine granularity gives a great sense of accomplishment.

Regarding how the assembly files are implemented specifically, I believe the fastest way is to look at the assembly code. I won’t explain it further here.

It should be noted that this version’s performance is very poor. Why is that? It’s another issue of **loop order**.

**Version 7: Assembly Version with Loop Order Swapped**

Brief explanation: A is **not packed**, but B is **prepacked** with 16 numbers.

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

Reversing the order of loops, starting with the **n-direction** followed by the **m-direction**, significantly improves performance.

However, the performance of large-sized matrices is still not very good. The root cause remains in **memory access**. The computation of large-sized matrices in the roofline model is considered **compute-bound**, where ideally the **compute time** and **memory access time** should overlap as much as possible. Currently, a significant amount of time is spent on memory access (mostly due to **cache miss**!).

**Version 8: Assembly Version with Preload**

Brief Explanation: Matrix A is **not packed**, while Matrix B undergoes **prepackaging** of 16 elements and includes a **preload** operation.

The performance is explosive! It reaches a maximum of **2.212 GFLOPS**.

Core operations:

```
vfmacc.vf v16,  ft0, v0
vlw.v v4, (bp0)         # b0'->v4
flw fs4, 384(bp0)       # pre-load B
addi bp0,bp0,64
vfmacc.vf v20,  ft1, v0
```

Inserting some **load** operations between **vfmacc.vf** instructions preloads the data that will be used later into the **cache**, significantly reducing **cache miss**.

Initially, I was puzzled — how can the **compute time** and **memory access time** overlap when the code seems to execute sequentially? It wasn’t until later that I understood the essence here, which lies in the principle of **cacheline**. Indeed, foundational knowledge is crucial!

**Version 9: Assembly Version with A Packed**

Based on previous experience, an attempt was made to **pack** Matrix A, but surprisingly, the results were not very good. A brief analysis suggests that the preload for Matrix A in this version of the assembly code might not be optimized.

In the previous version, although A wasn’t packed, there was preload for A’s 4 rows, which also addressed the pain point of cache miss for Matrix A.