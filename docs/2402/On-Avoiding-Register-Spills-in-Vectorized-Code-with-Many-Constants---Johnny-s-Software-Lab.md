<!--yml
category: 未分类
date: 2024-05-27 14:36:08
-->

# On Avoiding Register Spills in Vectorized Code with Many Constants - Johnny's Software Lab

> 来源：[https://johnnysswlab.com/on-avoiding-register-spills-in-vectorized-code-with-many-constants/](https://johnnysswlab.com/on-avoiding-register-spills-in-vectorized-code-with-many-constants/)

*We at **Johnny’s Software Lab LLC** are experts in performance. If performance is in any way concern in your software project, feel free to [contact us](https://johnnysswlab.com/contact/).*

Consider the following code that calculates *cosine* using numerical methods:

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

In this code, there is total of six constants: (1) 1\. / (2\. * M_PI), (2) 0.25, (3) 16.0, (4) 0.5, (5) 0.225 and (6) 1.0\. When vectorizing this code, the straightforward approach is to store the constants to vector variables outside of the loop:

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

In many cases this can be a good approach, but not in all of them. This kind of code reserves in total six registers to hold constants. On SSE/AVX this can be a big issue, since the number of registers is 16\. It is a bit less of a problem on NEON and AVX512, where the number of registers is 32\. But, in many scientific applications, codes with many constants are quite common so the number of constants easily exceeds the number of available registers.

When the compiler doesn’t have enough registers to generate efficient assembly, it [spills registers to the stack](https://johnnysswlab.com/decreasing-the-number-of-memory-accesses-the-compilers-secret-life-2-2/) and later reloads the data to the registers when they are again needed. Spilling can be done outside of the loop, but reloading needs to be done within the loop. In our case, reloading constants will be done in each iteration of the loop.

With regards to reloading, reloading is a simple load instruction, but loads on AVX have a latency of 4 cycles, and similar number is for NEON. And constants are first candidates for the compiler to spill, since they are typically used only once or twice in each iteration of the loop.

*Like what you are reading? Follow us on [LinkedIn](https://www.linkedin.com/company/johnysswlab) , [Twitter](https://twitter.com/johnnysswlab) or [Mastodon](https://mastodon.online/@johnnysswlab) and get notified as soon as new content becomes available.
Need help with software performance? [Contact us](https://johnnysswlab.com/contact/)!*

## Avoiding Register Spills

Here we present a workaround on how to avoid register spills by decreasing the number of constants. The trick is to store constants in the vector register, but each lane should store a different constant. With this approach, you can rematerialize the constant inside the loop much quicker than with reloading. So, from our previous example, we store value tp in lane 0, 0.25 in lane 1, 16.0 in lane 2 and 0.5 in lane 3\. The code looks like this

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

We have stored constants like this, but most instructions don’t work with single lanes. We need to rematerialize the constants. By rematerializing, we mean loading the constant into all lanes of an available vector register to be used by subsequent intrinsics.

When in it comes to storing constants, notice that for AVX, the values are repeated, i.e. lane 0 and 4 have the same value, as well as lane 1 and 5, 2 and 6, and 3 and 7\. We will explain later why this is the case.

### Rematerialization on AVX

On AVX, we use the fast `_mm256_shuffle_ps` intrinsic to rematerialize the constant, like this:

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

Materialization of value in lane X is performed using `_mm256_shuffle_ps(constant, constant, _MM_SHUFFLE(X, X, X, X))`. You can also use `_mm256_permute_ps(constant, _MM_SHUFFLE(X, X, X, X)` intrinsic with the same effect. This intrinsic loads lane X from register `constant` to all the lanes of the output register. Its latency is 1 cycle and throughput 0.5 cycles and it is faster than using loads to rematerialize constants.

A question you might be asking yourselves is why don’t we store 8 different values instead of two pairs of identical four values. The reason is performance compromises in AVX. In AVX, there is a performance penalty if data is moved from lower half of the 256 bit register to the higher half and vice-versa. So we have two copies of the same value, one in the lower 128 bit part, and another in the higher 128 bit part. And that’s why we use intrinsics that don’t cross 128 bit boundaries.

### Rematerialization on NEON

On NEON, materialization is performed using `vdupq_laneq_f32` intrinsic like this:

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

Rematerialization of lane X is performed using `vdupq_laneq_f32(constant, X)`. Again, this instruction has a lower latency than loading data from memory.

This code, however, is not optimal. On NEON, multiply (`vmul_XX`) and multiply-accumulate (`vmla_XX` and `vmls_XX`) operations have a scalar variant, which multiply lanes from register `a` with lane X from register `b`. So, instead of using `vdupq_laneq_f32` to rematerialize the value and multiplying it with `vmulq_f32`, we can omit rematerialization and use `vmulq_laneq_f32`, like this:

```
for (size_t i = 0; i < n; i+=4) {
    float32x4_t x = vld1q_f32(x + i);
    x = vmulq_f32(x, c0, 0);
    ...
}
```

Intrinsic `vmulq_f32(x, c0, 0)` multiplies each lane of register `x` with lane 0 from register `c0`.

## Conclusion

Simple codes will become a bit slower when using the above technique to reduce the number of needed registers, because rematerialization adds a small performance penalty. But, in case of complex loops, this approach yields better performance since rematerialization is faster.

Being aware of constants and register spills is especially important if you are writing functions that accept and return vector arguments. In this case, the compiler will often inline the vector function, but code at the place of call can be quite complex and causes compiler to spill registers. The described technique will help lower the register pressure and allow the compiler to emit more efficient assembly. You should optionally consult the [compiler optimization report](https://johnnysswlab.com/loop-optimizations-interpreting-the-compiler-optimization-report/) to check if the spilling actually happens.

*Like what you are reading? Follow us on [LinkedIn](https://www.linkedin.com/company/johnysswlab) , [Twitter](https://twitter.com/johnnysswlab) or [Mastodon](https://mastodon.online/@johnnysswlab) and get notified as soon as new content becomes available.
Need help with software performance? [Contact us](https://johnnysswlab.com/contact/)!*