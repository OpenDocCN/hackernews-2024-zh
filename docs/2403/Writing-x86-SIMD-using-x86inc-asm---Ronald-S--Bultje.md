```
#include <stddef.h>
#include <stdint.h>
#include <stdlib.h>

typedef uint8_t pixel;
static unsigned sad_16x16_c(const pixel *src, ptrdiff_t src_stride,
                            const pixel *dst, ptrdiff_t dst_stride)
{
    unsigned sum = 0;
    int y, x;

    for (y = 0; y < 16; y++) {
        for (x = 0; x < 16; x++)
            sum += abs(src[x] - dst[x]);
        src += src_stride;
        dst += dst_stride;
    }

    return sum;
} 
```

```
%include "x86inc.asm"

SECTION .text

INIT_XMM sse2
cglobal sad_16x16, 4, 7, 5, src, src_stride, dst, dst_stride, \
                            src_stride3, dst_stride3, cnt
    lea    src_stride3q, [src_strideq*3]
    lea    dst_stride3q, [dst_strideq*3]
    mov            cntd, 4
    pxor             m0, m0
.loop:
    mova             m1, [srcq+src_strideq*0]
    mova             m2, [srcq+src_strideq*1]
    mova             m3, [srcq+src_strideq*2]
    mova             m4, [srcq+src_stride3q]
    lea            srcq, [srcq+src_strideq*4]
    psadbw           m1, [dstq+dst_strideq*0]
    psadbw           m2, [dstq+dst_strideq*1]
    psadbw           m3, [dstq+dst_strideq*2]
    psadbw           m4, [dstq+dst_stride3q]
    lea            dstq, [dstq+dst_strideq*4]
    paddw            m1, m2
    paddw            m3, m4
    paddw            m0, m1
    paddw            m0, m3
    dec            cntd
    jg .loop
    movhlps          m1, m0
    paddw            m0, m1
    movd            eax, m0
    RET 
```

```
cglobal sad_16x16, 4, 7, 5, src, src_stride, dst, dst_stride, \
                            src_stride3, dst_stride3, cnt 
```

```
%macro SAD_FN 2 ; width, height
cglobal sad_%1x%2, 4, 7, 5, src, src_stride, dst, dst_stride, \
                            src_stride3, dst_stride3, cnt
    lea    src_stride3q, [src_strideq*3]
    lea    dst_stride3q, [dst_strideq*3]
    mov            cntd, %2 / 4
    pxor             m0, m0
.loop:
    mova             m1, [srcq+src_strideq*0]
    mova             m2, [srcq+src_strideq*1]
    mova             m3, [srcq+src_strideq*2]
    mova             m4, [srcq+src_stride3q]
    lea            srcq, [srcq+src_strideq*4]
    psadbw           m1, [dstq+dst_strideq*0]
    psadbw           m2, [dstq+dst_strideq*1]
    psadbw           m3, [dstq+dst_strideq*2]
    psadbw           m4, [dstq+dst_stride3q]
    lea            dstq, [dstq+dst_strideq*4]
    paddw            m1, m2
    paddw            m3, m4
    paddw            m0, m1
    paddw            m0, m3
    dec            cntd
    jg .loop

%if mmsize >= 16
%if mmsize >= 32
    vextracti128    xm1, m0, 1
    paddw           xm0, xm1
%endif
    movhlps         xm1, xm0
    paddw           xm0, xm1
%endif
    movd            eax, xm0
    RET
%endmacro

INIT_MMX mmxext
SAD_FN 8, 4
SAD_FN 8, 8
SAD_FN 8, 16

INIT_XMM sse2
SAD_FN 16, 8
SAD_FN 16, 16
SAD_FN 16, 32

INIT_YMM avx2
SAD_FN 32, 16
SAD_FN 32, 32
SAD_FN 32, 64 
```

```
%macro ABSW 2 ; dst/src, tmp
%if cpuflag(ssse3)
    pabsw   %1, %1
%else
    pxor    %2, %2
    psubw   %2, %1
    pmaxsw  %1, %2
%endif
%endmacro

%macro SAD_8x8_FN 0
cglobal sad_8x8, 4, 7, 6, src, src_stride, dst, dst_stride, \
                          src_stride3, dst_stride3, cnt
    lea    src_stride3q, [src_strideq*3]
    lea    dst_stride3q, [dst_strideq*3]
    mov            cntd, 2
    pxor             m0, m0
.loop:
    mova             m1, [srcq+src_strideq*0]
    mova             m2, [srcq+src_strideq*1]
    mova             m3, [srcq+src_strideq*2]
    mova             m4, [srcq+src_stride3q]
    lea            srcq, [srcq+src_strideq*4]
    psubw            m1, [dstq+dst_strideq*0]
    psubw            m2, [dstq+dst_strideq*1]
    psubw            m3, [dstq+dst_strideq*2]
    psubw            m4, [dstq+dst_stride3q]
    lea            dstq, [dstq+dst_strideq*4]
    ABSW             m1, m5
    ABSW             m2, m5
    ABSW             m3, m5
    ABSW             m4, m5
    paddw            m1, m2
    paddw            m3, m4
    paddw            m0, m1
    paddw            m0, m3
    dec            cntd
    jg .loop
    movhlps          m1, m0
    paddw            m0, m1
    pshuflw      m1, m0, q1010
    paddw            m0, m1
    pshuflw      m1, m0, q0000 ; qNNNN is a base4-notation for imm8 arguments
    paddw            m0, m0
    movd            eax, m0
    movsxwd         eax, ax
    RET
%endmacro

INIT_XMM sse2
SAD_8x8_FN

INIT_XMM ssse3
SAD_8x8_FN 
```

```
[..]
    mova           m0, [srcq+src_strideq*0]
    mova           m1, [srcq+src_strideq*1]
    punpckhbw  m2, m0, m1
    punpcklbw      m0, m1
[..] 
```

```
[..]
    vmovdqa    xmm0, [rdi]
    vmovdqa    xmm1, [rdi+rsi]
    vpunpckhbw xmm2, xmm0, xmm1
    vpunpcklbw xmm0, xmm0, xmm1
[..] 
```

```
[..]
    movdqa    xmm0, [rdi]
    movdqa    xmm1, [rdi+rsi]
    movdqa    xmm2, xmm0
    punpckhbw xmm2, xmm1
    punpcklbw xmm0, xmm1
[..] 
```

+   如果我们在通用寄存器（GPR）中保存了原始栈指针，我们在函数体中仍然可以访问到原始的`m`和`mp`命名的寄存器别名。然而，这意味着一个通用寄存器（保存栈指针的那个）在函数体内无法用于其他用途。具体来说，在32位上，这将限制函数体内可用的GPR数量为6个。要使用此选项，请指定一个正的栈大小；

```
cglobal sad_16x16, 4, 7, 5, 64, src, src_stride, dst, dst_stride, \
                                src_stride3, dst_stride3, cnt 
```

+   如果我们在（后对齐的）栈上保存了原始栈指针，我们将无法在函数体中访问`m`或`mp`命名的寄存器别名，但我们不占用GPR。要使用此选项，请指定一个负的栈大小。请注意，我们将一个负数写为`0 - 64`，而不仅仅是`-64`，因为在一些旧版本的`yasm`中存在一个错误。

```
cglobal sad_16x16, 4, 7, 5, 0 - 64, src, src_stride, dst, dst_stride, \
                                    src_stride3, dst_stride3, cnt
```

在栈内存分配之后，可以使用`[rsp]`来访问它（在32位上是`[esp]`的别名）。

## 结论

这篇文章旨在介绍`x86inc.asm`，一个为开发人员（有些）熟悉手写汇编或内部函数的x86手写汇编助手。它受到了一个名为[x264asm](https://wiki.videolan.org/X264_asm_intro/)的早期指南的启发。
