<!--yml
category: 未分类
date: 2024-05-27 14:35:58
-->

# TIL: Go’s CompareAndSwap is not always Compare-and-swap

> 来源：[https://lu.sagebl.eu/notes/go-cas/](https://lu.sagebl.eu/notes/go-cas/)

# TIL: Go’s CompareAndSwap is not always Compare-and-swap

Published on 7 January 2024

Go's standard package `sync/atomics` provides programmers with functions to use the underlying CPU-level atomic operations such as *compare-and-swap* (CAS), through `atomic.CompareAndSwapT` (where `T` is an integer type).

**Problem**: Not all CPU architectures offers a CAS instruction to rely on to implement `atomic.CompareAndSwapT`. However, Go must compile that function code to something semantically equivalent—let's see what.

<details><summary>What is a compare-and-swap?</summary>

Conceptually, CAS is an operation defined as followed, except it's atomic.

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

Let's take a look at how `atomic.CompareAndSwapInt64` works with `int64`. The function calls a lower-level function `Cas64` which is defined in assembly language as follows. It's a mapping to x86_64's `LOCK CMPXCHG` ("Compare and Exchange") instruction that performs the CAS.

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

The code is from [Go 1.21.5 source](https://cs.opensource.google/go/go/+/refs/tags/go1.21.5:src/runtime/internal/atomic/atomic_amd64.s;l=46-53).

Remarks: `LOCK` is not an instruction, it's a prefix to make `CMPXCHG` atomic in concurrent contexts. `AX`, `BX`, `CX` and `FP` are translated into real corresponding registers during compilation; The Go assembler does not use the actual architecture's regsiter names directly.

Not much more to say, the Go function translates to the equivalent CPU instruction, here.

## Arm64 (AArch64)

The Arm64 case is interesting. The function `Cas64` defined for Arm64 has two ways to perform a CAS operation.

*   With the `CASAL` instruction, which is similar to AMD64's `CMPXCHG`.
*   With the `LDAXR` and `STLXR` instructions, enclosing the hand-coded CAS logic in-between.

The `CASAL` instructions is part of the [Armv8.1-A Large System Extension (LSE)](https://learn.arm.com/learning-paths/servers-and-cloud-computing/lse/intro/) that provides CAS operations. If the CPU supports it, this path is taken. If the CPU does not support LSE, then the function fallbacks on another path, performing an "emulated" CAS with the help of `LDAXR` and `STLXR`.

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

(I added the comments.)

I was curious about the Arm64 case, because typical RISC architectures are not usually keen to provide instructions that operate directly on memory like x86_64. With RISC, one usually needs to load a value from memory, operates on it, and then stores the value into memory in distinct steps, unlike CAS does.

In theory LSE instructions such as `CASAL` should boost code performance. It appeared to be true on micro-benchmarks simulating cases of high contention, but according to some [benchmarks conducted by the author of the *MySQL on ARM* blog](https://mysqlonarm.github.io/ARM-LSE-and-MySQL/), the results are more mitigated on real workloads.

## RISC-V 64 (RV64A)

RV64A's `LRD` and `SCD` instructions are equivalent to Arm64's `LDAXR` and `STLXR` seen previously. The hand-coded logic is similar, too.

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

Code from [Go source](https://cs.opensource.google/go/go/+/refs/tags/go1.21.5:src/runtime/internal/atomic/atomic_riscv64.s;l=59-73).

It seems that RISC-V is purer than Arm64; it does not provide a ready-to-use CAS instruction. CAS must be done by hand with the LL/SC pattern. I found an interesting rationale in the RISC-V instruction manual:

<q>Both compare-and-swap (CAS) and LR/SC can be used to build lock-free data structures. After extensive discussion, we opted for LR/SC for several reasons: 1) CAS suffers from the ABA problem, which LR/SC avoids because it monitors all accesses to the address rather than only checking for changes in the data value; 2) CAS would also require a new integer instruction format to support three source operands (address, compare value, swap value) as well as a different memory system message format, which would complicate microarchitecture</q>

## Additional notes

I looked at Go's implementation because that's what I use. Unsurprisingly, other implementations take the same approach: use LL/SC when CAS does not exist *in silico*, and for Arm64 use one or another pattern depending on the availability of LSE. The previously linked article says as follows:

<q>GCC auto emits a code with dynamic check with both variants (lse and non-lse). Runtime a decision is taken and accordingly said variant is executed.</q>