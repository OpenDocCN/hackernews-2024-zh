<!--yml

类别：未分类

日期：2024-05-27 14:35:58

-->

# TIL：Go 的 CompareAndSwap 并不总是 Compare-and-swap。

> 来源：[`lu.sagebl.eu/notes/go-cas/`](https://lu.sagebl.eu/notes/go-cas/)

# TIL：Go 的 CompareAndSwap 并不总是 Compare-and-swap。

发布于 2024 年 1 月 7 日

Go 的标准包 `sync/atomics` 通过 `atomic.CompareAndSwapT`（其中 `T` 是整数类型）为程序员提供了使用底层 CPU 级别原子操作的函数，如 *compare-and-swap*（CAS）。

**问题**：并非所有的 CPU 架构都提供了 CAS 指令来实现 `atomic.CompareAndSwapT`。然而，Go 必须将该函数代码编译成语义上等效的东西——让我们看看是什么。

<details><summary>什么是比较和交换？</summary>

从概念上讲，CAS 是一个操作，如下所示，只不过它是原子性的。

```
func CAS(ptr *T, old, new T) bool {
    if (*ptr != old) {
        return false
    }
    *ptr = new
    return true
}
```</details>

## x86_64 (AMD64)

让我们看看 `atomic.CompareAndSwapInt64` 如何与 `int64` 一起工作。该函数调用一个低级别的函数 `Cas64`，它的汇编语言定义如下。它是对 x86_64 的 `LOCK CMPXCHG`（“比较并交换”）指令的映射，用于执行 CAS。

```
TEXT ·Cas64(SB), NOSPLIT, $0-25
  MOVQ  ptr+0(FP), BX
  MOVQ  old+8(FP), AX
  MOVQ  new+16(FP), CX
  LOCK
  CMPXCHGQ  CX, 0(BX)
  SETEQ ret+24(FP)
  RET 
```

代码来自[Go 1.21.5 源代码](https://cs.opensource.google/go/go/+/refs/tags/go1.21.5:src/runtime/internal/atomic/atomic_amd64.s;l=46-53)。

备注：`LOCK` 不是一条指令，它是一种前缀，使 `CMPXCHG` 在并发环境中变为原子操作。`AX`、`BX`、`CX` 和 `FP` 在编译期间会被转换为实际对应的寄存器；Go 汇编器不会直接使用实际架构的寄存器名称。

没有更多可说的了，Go 函数在这里转换为相应的 CPU 指令。

## Arm64 (AArch64)

Arm64 的情况很有趣。针对 Arm64 定义的函数 `Cas64` 有两种执行 CAS 操作的方式。

+   使用 `CASAL` 指令，类似于 AMD64 的 `CMPXCHG`。

+   使用 `LDAXR` 和 `STLXR` 指令，将手工编码的 CAS 逻辑封装在其中。

`CASAL` 指令是[Armv8.1-A 大系统扩展（LSE）](https://learn.arm.com/learning-paths/servers-and-cloud-computing/lse/intro/)的一部分，提供 CAS 操作。如果 CPU 支持它，就会采用这种方式。如果 CPU 不支持 LSE，则函数会退而求其次，通过 `LDAXR` 和 `STLXR` 的帮助执行一个“模拟”的 CAS。

```
TEXT ·Cas64(SB), NOSPLIT, $0-25
  MOVD  ptr+0(FP), R0
  MOVD  old+8(FP), R1
  MOVD  new+16(FP), R2

// Check support for LSE atomics
  MOVBU internal∕cpu·ARM64+const_offsetARM64HasATOMICS(SB), R4
  CBZ   R4, load_store_loop

// LSE supported, use CASAL
  MOVD  R1, R3
  CASALD  R3, (R0), R2
  CMP   R1, R3
  CSET  EQ, R0
  MOVB  R0, ret+24(FP)
  RET

// LSE not supported, use LDAXR/STLXR
load_store_loop:
  LDAXR (R0), R3
  CMP R1, R3
  BNE ok
  STLXR R2, (R0), R3
  CBNZ  R3, load_store_loop
ok:
  CSET  EQ, R0
  MOVB  R0, ret+24(FP)
  RET 
```

（我添加了注释。）

我对 Arm64 的情况很感兴趣，因为典型的 RISC 架构通常不太愿意直接在内存上进行操作，就像 x86_64 那样。在 RISC 中，通常需要分别从内存加载值，对其进行操作，然后将值存储到内存中，而不像 CAS 那样。

理论上，诸如`CASAL`之类的 LSE 指令应该提升代码性能。在模拟高争用情况的微基准测试中似乎是正确的，但根据一些[由 *MySQL on ARM* 博客的作者进行的基准测试](https://mysqlonarm.github.io/ARM-LSE-and-MySQL/)，结果在真实工作负载中更为平缓。

## RISC-V 64（RV64A）

RV64A 的`LRD`和`SCD`指令与之前看到的 Arm64 的`LDAXR`和`STLXR`是等效的。手动编写的逻辑也类似。

```
TEXT ·Cas64(SB), NOSPLIT, $0-25
  MOV ptr+0(FP), A0
  MOV old+8(FP), A1
  MOV new+16(FP), A2
cas_again:
  LRD  (A0), A3
  BNE  A3, A1, cas_fail
  SCD  A2, (A0), A4
  BNE  A4, ZERO, cas_again
  MOV  $1, A0
  MOVB A0, ret+24(FP)
  RET
cas_fail:
  MOVB  ZERO, ret+24(FP)
  RET 
```

代码来自[Go 源代码](https://cs.opensource.google/go/go/+/refs/tags/go1.21.5:src/runtime/internal/atomic/atomic_riscv64.s;l=59-73)。

看起来 RISC-V 比 Arm64 更纯粹；它不提供现成的 CAS 指令。CAS 必须用 LL/SC 模式手动完成。在 RISC-V 指令手册中我发现了一个有趣的原因：

<q>比较和交换（CAS）和 LR/SC 都可以用于构建无锁数据结构。经过广泛讨论，我们选择了 LR/SC，原因有几点：1）CAS 存在 ABA 问题，LR/SC 可以避免这个问题，因为它监视对地址的所有访问，而不仅仅检查数据值的变化；2）CAS 还需要一个新的整数指令格式来支持三个源操作数（地址、比较值、交换值），以及一个不同的内存系统消息格式，这会使微架构复杂化</q>

## 附加说明

我查看了 Go 的实现，因为那是我使用的。毫不奇怪，其他实现采用了相同的方法：在计算机中**不存在 CAS**时使用 LL/SC，对于 Arm64，根据 LSE 的可用性使用一种模式或另一种模式。之前链接的文章如下所示：

<q>GCC 自动使用两种变体（lse 和 non-lse）进行动态检查的代码。运行时根据情况做出决定，并相应地执行所选的变体。</q>
