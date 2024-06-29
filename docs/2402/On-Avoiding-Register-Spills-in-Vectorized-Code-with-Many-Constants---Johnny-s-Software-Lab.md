<!--yml

类别：未分类

日期：2024-05-27 14:36:08

-->

# 如何避免向量化代码中的寄存器溢出 - Johnny's Software Lab

> 来源：[https://johnnysswlab.com/on-avoiding-register-spills-in-vectorized-code-with-many-constants/](https://johnnysswlab.com/on-avoiding-register-spills-in-vectorized-code-with-many-constants/)

我们在**Johnny’s Software Lab LLC**是性能专家。如果您的软件项目中有性能方面的任何问题，请随时[联系我们](https://johnnysswlab.com/contact/)。

考虑以下计算*余弦*的数值方法代码：

```
void cos(float* x, size_t n) {
    constexpr float tp = 1\. / (2\. * M_PI);
    for (size_t i = 0; i < n; i++) {
        x = x * tp;
        x = x - (float(.25) + std::floor(x + float(.25)));
        x = x * (float(16.) * (std::abs(x) - float(.5)));
        x = x + (float(.225) * x * (std::abs(x) - float(1.)));
    }
}
```

在这段代码中，总共有六个常量：(1) 1. / (2. * M_PI)，(2) 0.25，(3) 16.0，(4) 0.5，(5) 0.225和(6) 1.0。在向量化此代码时，直接的方法是在循环外将常量存储到向量变量中：

```
void cos_avx(float* x, size_t n) {
    const __m256 tp = _mm256_set1_ps(1\. / (2\. * M_PI));
    const __m256 c0p25 = _mm256_set1_ps(0.25);
    const __m256 c16 = _mm256_set1_ps(16.0);
    const __m256 c0p5 = _mm256_set1_ps(0.5);
    ...
    for (size_t i = 0; i < n; i+=4) {
        __m256 x = _mm256_loadu_ps(x + i);
        x = _mm256_mul_ps(x, tp);
        ...
    }
}

void cos_neon(float* x, size_t n) {
    const float32x4_t tp = vdupq_n_f32(1\. / (2\. * M_PI));
    const float32x4_t c0p25 = vdupq_n_f32(0.25);
    const float32x4_t c16 = vdupq_n_f32(16.0);
    const float32x4_t c0p5 = vdupq_n_f32(0.5);
    ...
    for (size_t i = 0; i < n; i+=4) {
        float32x4_t x = vld1q_f32(x + i);
        x = vmulq_f32(x, tp);
        ...
    }
}
```

在许多情况下，这可能是一个好的方法，但并非所有情况都是如此。这种代码总共保留了六个寄存器来保存常量。在SSE/AVX上这可能是一个大问题，因为寄存器数量是16个。在NEON和AVX512上这个问题稍微小一些，因为寄存器数量是32个。但是，在许多科学应用程序中，具有许多常量的代码非常常见，因此常量的数量很容易超过可用寄存器的数量。

当编译器没有足够的寄存器生成高效的汇编时，会将寄存器[溢出到堆栈](https://johnnysswlab.com/decreasing-the-number-of-memory-accesses-the-compilers-secret-life-2-2/)，然后在再次需要时重新加载数据到寄存器。溢出可以在循环外完成，但重新加载需要在循环内完成。在我们的情况下，每次循环迭代都会重新加载常量。

关于重载，重载是一个简单的加载指令，但AVX上的加载延迟为4个周期，NEON上也是类似的。常量通常是编译器在每次循环迭代中首选溢出的候选项，因为它们通常仅在循环中使用一次或两次。

*喜欢这篇文章吗？在[LinkedIn](https://www.linkedin.com/company/johnysswlab)，[Twitter](https://twitter.com/johnnysswlab)或[Mastodon](https://mastodon.online/@johnnysswlab)上关注我们，并在新内容推出时及时收到通知。

需要帮助优化软件性能？[联系我们](https://johnnysswlab.com/contact/)！

## 避免寄存器溢出

这里我们提出了一种方法来避免通过减少常量数量来减少寄存器溢出的方法。诀窍是将常量存储在向量寄存器中，但每个lane应存储不同的常量。采用这种方法，您可以比重新加载更快地在循环中重新生成常量。因此，根据我们之前的示例，我们将值tp存储在lane 0中，0.25存储在lane 1中，16.0存储在lane 2中，0.5存储在lane 3中。代码如下所示：

```
void cos_avx(float* x, size_t n) {
    // Note that lanes pairs (0, 4), (1, 5), (2, 6), (3.7)
    // hold identical values.
    const __m256 c0 = _mm256_setr_ps(tp, 0.25, 16.0, 0.5, tp, 0.25, 16.0, 0.5);
    ...
    for (size_t i = 0; i < n; i+=4) {
        ...
    }
}

void cos_neon(float* x, size_t n) {
    float c0A[4] = { tp, 0.25, 16.0, 0.5 };
    const float32x4_t c0 = vld1q_f32(c0A);
    ...
    for (size_t i = 0; i < n; i+=4) {
        ...
    }
}
```

我们已经像这样存储常数，但大多数指令不能处理单个轨道。我们需要重新材料化常数。通过重新材料化，我们的意思是将常数加载到可用矢量寄存器的所有轨道中，以供后续内在函数使用。

在存储常数方面，注意对于 AVX，值是重复的，即轨道 0 和 4 有相同的值，轨道 1 和 5 也是如此，以此类推。我们稍后会解释这样做的原因。

### 在 AVX 上的材料化

在 AVX 上，我们使用快速的 `_mm256_shuffle_ps` 内在函数来材料化常数，就像这样：

```
void cos_avx(float* x, size_t n) {
    // Note that lanes pairs (0, 4), (1, 5), (2, 6), (3.7)
    // hold identical values.
    const __m256 c0 = _mm256_setr_ps(tp, 0.25, 16.0, 0.5, tp, 0.25, 16.0, 0.5);
    ...
    for (size_t i = 0; i < n; i+=4) {
        __m256 x = _mm256_loadu_ps(x + i);
        __m256 tp = _mm256_shuffle_ps(c0, c0, _MM_SHUFFLE(0, 0, 0, 0));
        x = _mm256_mul_ps(x, tp);
        ...
    }
}
```

使用 `_mm256_shuffle_ps(constant, constant, _MM_SHUFFLE(X, X, X, X))` 材料化 X 轨道的值。您还可以使用具有相同效果的 `_mm256_permute_ps(constant, _MM_SHUFFLE(X, X, X, X)` 内在函数。这种内在函数将寄存器 `constant` 的轨道 X 加载到输出寄存器的所有轨道中。其延迟为 1 个周期，吞吐量为 0.5 个周期，比使用加载来重新材料化常数更快。

你们可能会问为什么我们不存储 8 个不同的值而是存储两对相同的四个值。原因在于 AVX 中的性能折衷。在 AVX 中，如果数据从 256 位寄存器的低半部分移动到高半部分，反之亦然，会产生性能损失。因此，我们有两个相同值的副本，一个在低 128 位部分，另一个在高 128 位部分。这就是为什么我们使用不跨越 128 位边界的内在函数。

### NEON 上的材料化

在 NEON 上，材料化使用 `vdupq_laneq_f32` 内在函数，像这样：

```
void cos_neon(float* x, size_t n) {
    float c0A[4] = { tp, 0.25, 16.0, 0.5 };
    const float32x4_t c0 = vld1q_f32(c0A);
    ...
    for (size_t i = 0; i < n; i+=4) {
        float32x4_t x = vld1q_f32(x + i);
        float32x4_t tp = vdupq_laneq_f32(c0, 0);
        x = vmulq_f32(x, tp);
        ...
    }
}
```

重建 X 轨道的材料化使用 `vdupq_laneq_f32(constant, X)` 完成。再次，这条指令比从内存加载数据具有更低的延迟。

然而，这段代码并不是最优的。在 NEON 上，乘法 (`vmul_XX`) 和乘-累加 (`vmla_XX` 和 `vmls_XX`) 操作有一个标量变体，它们将寄存器 `a` 的轨道与寄存器 `b` 的轨道 X 相乘。因此，我们可以省略材料化并使用 `vmulq_laneq_f32`，像这样：

```
for (size_t i = 0; i < n; i+=4) {
    float32x4_t x = vld1q_f32(x + i);
    x = vmulq_f32(x, c0, 0);
    ...
}
```

内在函数 `vmulq_f32(x, c0, 0)` 将寄存器 `x` 的每个轨道与寄存器 `c0` 的轨道 0 相乘。

## 结论

使用上述技术减少所需寄存器的简单代码会变得稍慢，因为材料化会增加一些性能开销。但是，在复杂循环的情况下，这种方法提供了更好的性能，因为材料化速度更快。

如果您正在编写接受和返回向量参数的函数，则特别需要注意常量和寄存器溢出情况。在这种情况下，编译器通常会内联向量函数，但调用处的代码可能相当复杂，导致编译器溢出寄存器。所述的技术将有助于降低寄存器压力，并允许编译器生成更有效的汇编代码。您可以选择查阅[编译器优化报告](https://johnnysswlab.com/loop-optimizations-interpreting-the-compiler-optimization-report/)，以确认是否确实发生了寄存器溢出。

*喜欢我们的内容吗？请在[LinkedIn](https://www.linkedin.com/company/johnysswlab)，[Twitter](https://twitter.com/johnnysswlab)或者[Mastodon](https://mastodon.online/@johnnysswlab)上关注我们，以便第一时间获取新内容通知。

如果您需要软件性能方面的帮助，请[联系我们](https://johnnysswlab.com/contact/)！*
