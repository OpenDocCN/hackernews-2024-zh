<!--yml
category: 未分类
date: 2024-05-27 14:29:47
-->

# A story of a very large loop with a long instruction dependency chain - Johnny's Software Lab

> 来源：[https://johnnysswlab.com/a-story-of-a-very-large-loop-with-a-long-instruction-dependency-chain/](https://johnnysswlab.com/a-story-of-a-very-large-loop-with-a-long-instruction-dependency-chain/)

*We at **Johnny’s Software Lab LLC** are experts in performance. If performance is in any way concern in your software project, feel free to [contact us](https://johnnysswlab.com/contact/).*

Some time ago we had to investigate a performance problem for a customer. The problematic piece of code was a loop. The loop was very long, about 1000 lines of vector intrinsics. The vectorization was done efficiently so there were no more efficient intrinsics that we could use to make it faster. Just by looking at it, it seemed that it everything was fine and we reached peak performance.

Nevertheless, we measured CPI (cycles per instructions) number to find out it was quite bad. The most common reasons for low CPI are (1) stalls because data couldn’t be fetched from the memory subsystem and (2) instruction dependencies.

Since all memory accesses were sequential, we investigated the loop for instruction dependencies. And behold, inside the loop, the input data for each intrinsic depended on the output data of the previous intrinsic. There was a long chain of dependencies starting from the first line in the loop body and going to the last line. There were no loop-carried dependencies – each iteration was independent of all the other iterations.

## The experiment

For the purposes of this post, we recreated a similar loop – a loop with a long dependency chain going from the first instruction to the last instruction but without loop-carried dependency ([full source code](https://github.com/ibogosavljevic/johnysswlab/blob/master/2024-02-ilp-loopfission/cos_test.cpp)). First let’s declare a `cos_vector` function which takes a vector of four double values and calculates four cosines. The function looks like this:

```
__m256d cos_vector(__m256d x) noexcept
{
    constexpr double tp_scalar = 1\. / (2\. * M_PI);
    __m256d tmp;

    // x = x * tp;
    const __m256d tp = _mm256_set1_pd(tp_scalar);
    x = _mm256_mul_pd(x, tp);

    // x = x - (double(.25) + std::floor(x + double(.25)));
    const __m256d v_25 = _mm256_set1_pd(0.25);
    tmp = _mm256_add_pd(x, v_25);

    tmp = _mm256_floor_pd(tmp);
    tmp = _mm256_add_pd(v_25, tmp);

    x = _mm256_sub_pd(x, tmp);

    // x = x * (double(16.) * (std::abs(x) - double(.5)));
    const __m256d v_5 = _mm256_set1_pd(0.5);
    const __m256d v_16 = _mm256_set1_pd(16.0);

    tmp = _mm256_abs_pd(x);
    tmp = _mm256_sub_pd(tmp, v_5);
    tmp = _mm256_mul_pd(v_16, tmp);
    x = _mm256_mul_pd(x, tmp);

    // x = x + (double(.225) * x * (std::abs(x) - double(1.)));
    const __m256d v_225 = _mm256_set1_pd(0.225);
    const __m256d v_1 = _mm256_set1_pd(1.0);
    tmp = _mm256_abs_pd(x);
    tmp = _mm256_sub_pd(tmp, v_1);

    tmp = _mm256_mul_pd(x, tmp);
    tmp = _mm256_mul_pd(v_225, tmp);
    x = _mm256_add_pd(x, tmp);

    return x;
}
```

Don’t spend too much time analyzing what the function does, since it is not important. What is important is to notice the instruction dependency chain. For many intrinsics in this function, the input data used by intrinsic X is produced by the intrinsic that comes immediately before it. So there is an instruction dependency chain that starts at the beginning of the function and ends at the end of the function.

Next, we use this function to calculate nested cosines, like this:

```
for (int i = 0; i < cnt1; i += VECTOR_SIZE)
{
    double_vec v = load_vec(v_ptr + i);
    sum_vec += cos_vector(cos_vector(...(cos_vector(v)) ...);
}
```

For this experiment. we added nested cosines here intentionally, to make the instruction dependency chain arbitrary long!

When we measured CPI (cycles per instruction metric, smaller number is better, 0.25 is the best possible) as a function of number of nested cosines, we got following values:

| Number of nested cosines | CPI |
| --- | --- |
| 1 | 0.65 |
| 5 | 1.47 |
| 10 | 2.14 |
| 15 | 2.46 |
| 20 | 2.63 |

When there is only one cosine, the CPU can start executing instructions from the next iteration of the loop if it doesn’t have enough work. But as the dependency chain becomes longer with the growing number of nested cosines, the CPU has less and less work to do and the CPI metric gets worse.

## Interleaving

One solution to increasing ILP is interleaving: we create a new version of `cos_vector` that instead of one takes two double vectors for inputs and calculate two results. So, inside our new [`cos_vector_interleaved`](https://github.com/ibogosavljevic/johnysswlab/blob/master/2024-02-ilp-loopfission/cos_test.cpp#L133) we have two dependency chains running in parallel. This increases available ILP and consequently increases performance.

We already covered [interleaving in earlier post](https://johnnysswlab.com/when-an-instruction-depends-on-the-previous-instruction-depends-on-the-previous-instructions-long-instruction-dependency-chains-and-performance/). Interleaving would be fine for this specific example, but it wasn’t a good approach with our customer: the loop body was already quite complex and interleaving makes it even more complex. Also, there was fear of register spilling and possible loss of performance because the loop body was already using a lot of constants and needed a lot of registers.

*Like what you are reading? Follow us on [LinkedIn](https://www.linkedin.com/company/johnysswlab) , [Twitter](https://twitter.com/johnnysswlab) or [Mastodon](https://mastodon.online/@johnnysswlab) and get notified as soon as new content becomes available.
Need help with software performance? [Contact us](https://johnnysswlab.com/contact/)!*

## A different approach

Are there other approaches? Let’s limit the analysis for simplicity to three nested cosines. Our original loop looks like this:

```
for (int i = 0; i < cnt1; i += VECTOR_SIZE)
{
    double_vec v = load_vec(v_ptr + i);
    sum_vec += cos_vector(cos_vector(cos_vector(v)));
}
```

We took our original loop and split it into three smaller loops, as you can see bellow. Each loop calculates one cosine. This approach is called *loop fission*. Since each loop produces a result that is consumed by the next loop, we need a temporary array to store the intermediate result:

```
std::vector<double_vec> tmp_vec(cnt1);

for (int i = 0; i < cnt1; i += VECTOR_SIZE) {
    tmp_vec[i] = cos_vector(load_vec(v_ptr + i));
}

for (int i = 0; i < cnt1; i += VECTOR_SIZE) {
    tmp_vec[i] = cos_vector(tmp_vec[i]);
}

for (int i = 0; i < cnt1; i += VECTOR_SIZE) {
    sum_vec += cos_vector(tmp_vec[i]);
}
```

The original big loop got split into three smaller loops. The newly introduced `tmp_vec` holds the intermediate results for the second and the third loop.

Does this approach work? Here are the runtimes for the original loop, interleaved loop and fissioned loop for various numbers of nested cosines, from 1 to 49 and 1 M:

Our new version is better than the original version, but not than the interleaved version. Interleaved version is still better. But notice the outlier for the really long dependency chain of 1 M nested cosines. This requires some additional investigation.

Because of the way we wrote our test, the size of the input array depends on the number of nested cosines. For one nested cosine, the size of the input array is 60 M elements, for two nested cosines the size of the input array is 30 M elements, for N nested cosines the size of the input array is 60 M divided by N. So, for 1 M nested cosines, the size of the input array is only 60.

## The memory subsystem

Since the size of our temporary buffer is the size of the input array, and we read from the buffer and write to it many times, this makes a lot of data moving in and out of the memory subsystem. And the performance depends on the which part of the memory subsystem the data is coming from. If it is from L1 cache, the loop will be fast, if it is from memory, the loop will be slow.

We can easily check where the data is coming from using LIKWID measurement group L2CACHE. There is a measurement called *L2 Request Rate* which is number of L2 requests made per executed instruction. For 10 nested cosines, this number is 0.011 for the original and interleaved versions and 0.3 for our version. Ouch!

## A smaller temporary buffer

The solution is to make our temporary buffer smaller, so it fits L1 cache. The size of L1 cache is about 16 kB, so we need to keep our temporary buffer smaller than this number.

If our input buffer was smaller, we would need a smaller temporary buffer. And we can make our input buffer smaller! We just need to process our input data in blocks. The first block is e.g. first 128 values from the input, second block is next 128 values in the input, etc. This technique is called *loop blocking*, because data is processed in blocks. After blocking, our processing loop looks like this:

```
constexpr int BLOCK_SIZE = 128;

std::vector<double_vec> tmp_vec2(BLOCK_SIZE);

for (int ii = 0; ii < cnt1; ii += BLOCK_SIZE) {
    int i_end = std::min(ii + BLOCK_SIZE, cnt1);

    for (int i = ii; i < i_end; i += VECTOR_SIZE) {
        tmp_vec2[i - ii] = cos_vector(load_vec(v_ptr + i));
    }

    for (int i = ii; i < i_end; i+= VECTOR_SIZE) {
        tmp_vec2[i - ii] = cos_vector(tmp_vec2[i - ii]);
    }

    for (int i = ii; i < i_end; i+= VECTOR_SIZE) {
        sum_vec += cos_vector(tmp_vec2[i - ii]);
    }
}
```

The outer loop works on a block level, and the inner loops process data within blocks. Here are the performance numbers with loop fission + loop blocking:

Now this looks better! The last version, with fission and blocking is the fastest!

*Like what you are reading? Follow us on [LinkedIn](https://www.linkedin.com/company/johnysswlab) , [Twitter](https://twitter.com/johnnysswlab) or [Mastodon](https://mastodon.online/@johnnysswlab) and get notified as soon as new content becomes available.
Need help with software performance? [Contact us](https://johnnysswlab.com/contact/)!*