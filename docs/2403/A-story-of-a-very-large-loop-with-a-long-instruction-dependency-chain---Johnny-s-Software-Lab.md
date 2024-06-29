<!--yml

category: 未分类

date: 2024-05-27 14:29:47

实验

# 一个关于具有长指令依赖链的非常大循环的故事 - 约翰尼软件实验室

> 来源：[https://johnnysswlab.com/a-story-of-a-very-large-loop-with-a-long-instruction-dependency-chain/](https://johnnysswlab.com/a-story-of-a-very-large-loop-with-a-long-instruction-dependency-chain/)

*我们在**约翰尼软件实验室有限责任公司**是性能方面的专家。如果性能在您的软件项目中是一个问题，请随时[联系我们](https://johnnysswlab.com/contact/)。*

由于所有内存访问都是顺序的，我们调查了循环以查找指令依赖关系。在循环内部，每个内部函数的输入数据都依赖于前一个内部函数的输出数据。从循环体的第一行开始，到最后一行，存在一长串的依赖链。没有循环承载的依赖关系 - 每次迭代与所有其他迭代无关。

尽管如此，我们测量的CPI（每指令周期数）数值表明效果相当差。CPI低的最常见原因是（1）因为数据无法从内存子系统中获取而导致的停顿和（2）指令依赖关系。

| 15 | 2.46 |

## 对于这个实验，我们故意添加了嵌套余弦，以使指令依赖链变得任意长！

为了本文的目的，我们重新创建了一个类似的循环 - 一个具有长依赖链的循环，从第一个指令到最后一个指令，但没有循环承载的依赖关系（[完整源代码](https://github.com/ibogosavljevic/johnysswlab/blob/master/2024-02-ilp-loopfission/cos_test.cpp)）。首先让我们声明一个`cos_vector`函数，它接受一个包含四个双精度值的向量，并计算四个余弦值。该函数如下所示：

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

不要花太多时间分析函数的作用，因为这并不重要。重要的是注意指令依赖链。在这个函数中的许多内部函数中，内部函数 X 使用的输入数据是由前一个内部函数生成的。因此，存在一个从函数开始到函数结束的指令依赖链。

接下来，我们使用这个函数来计算嵌套余弦，就像这样：

```
for (int i = 0; i < cnt1; i += VECTOR_SIZE)
{
    double_vec v = load_vec(v_ptr + i);
    sum_vec += cos_vector(cos_vector(...(cos_vector(v)) ...);
}
```

| 嵌套余弦数量 | CPI |

当我们将CPI（每指令周期数，数值越小越好，0.25为最佳）作为嵌套余弦数的函数进行测量时，我们得到以下数值：

| 5 | 1.47 |
| --- | --- |
| 1 | 0.65 |
| 一段时间以前，我们曾经为一个客户调查过一个性能问题。问题代码是一个循环。这个循环非常长，大约有1000行的向量内部函数。向量化处理非常高效，因此我们无法使用更高效的内部函数来加快速度。仅仅通过看它，似乎一切都很好，我们达到了最高性能。 |
| 10 | 2.14 |
| --> |
| 20 | 2.63 |

当只有一个余弦函数时，如果CPU没有足够的工作量，它可以从循环的下一次迭代开始执行指令。但随着嵌套余弦函数数量的增加，依赖链变得更长，CPU 需要做的工作越来越少，CPI 指标也会变得更糟。

## 交错执行

增加 ILP 的一个解决方案是交错执行：我们创建了一个名为 [`cos_vector_interleaved`](https://github.com/ibogosavljevic/johnysswlab/blob/master/2024-02-ilp-loopfission/cos_test.cpp#L133) 的新版本，它不再使用一个输入双向量，而是计算两个结果。因此，在我们的新版本中，有两个并行运行的依赖链。这增加了可用的 ILP，从而提高了性能。

我们之前已经讨论过[交错执行](https://johnnysswlab.com/when-an-instruction-depends-on-the-previous-instruction-depends-on-the-previous-instructions-long-instruction-dependency-chains-and-performance/)。对于这个特定的例子，交错执行可能不错，但对于我们的客户来说并不是一个好方法：循环体已经相当复杂，而交错执行会使其更加复杂。此外，由于循环体已经使用了大量常数并需要大量寄存器，存在寄存器溢出和性能损失的风险。

*喜欢阅读我们的内容？在 [LinkedIn](https://www.linkedin.com/company/johnysswlab)、[Twitter](https://twitter.com/johnnysswlab) 或 [Mastodon](https://mastodon.online/@johnnysswlab) 上关注我们，并在新内容发布时得到通知。

需要软件性能的帮助？[联系我们](https://johnnysswlab.com/contact/)！

## 另一种方法

是否有其他方法？为简便起见，让我们将分析限制在三个嵌套余弦函数上。我们的原始循环看起来是这样的：

```
for (int i = 0; i < cnt1; i += VECTOR_SIZE)
{
    double_vec v = load_vec(v_ptr + i);
    sum_vec += cos_vector(cos_vector(cos_vector(v)));
}
```

我们将原始循环拆分为三个更小的循环，如下所示。每个循环计算一个余弦函数。这种方法称为*循环分裂*。由于每个循环产生的结果被下一个循环使用，我们需要一个临时数组来存储中间结果：

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

原始的大循环被分成了三个更小的循环。新引入的 `tmp_vec` 保存了第二个和第三个循环的中间结果。

这种方法有效吗？以下是原始循环、交错循环和分裂循环在各种嵌套余弦函数数量（从 1 到 49 和 1 百万）的运行时间：

我们的新版本比原始版本更好，但不及交错版本。交错版本仍然更好。但请注意，当存在 1 百万个嵌套余弦函数的长依赖链时会有异常值。这需要进一步的调查。

由于我们编写测试的方式，输入数组的大小取决于嵌套余弦的数量。对于一个嵌套余弦，输入数组的大小是6000万个元素，对于两个嵌套余弦，输入数组的大小是3000万个元素，对于N个嵌套余弦，输入数组的大小是6000万除以N。所以，对于1百万个嵌套余弦，输入数组的大小仅为60。

## 内存子系统

由于我们临时缓冲区的大小与输入数组的大小相同，并且我们多次从缓冲区读取和写入，这导致大量数据在内存子系统中移动。而性能取决于数据来自内存子系统的哪个部分。如果来自L1缓存，循环将会很快，如果来自内存，则循环将会很慢。

我们可以很容易地使用LIKWID测量组L2CACHE检查数据的来源。有一个叫做*L2请求率*的测量值，表示每个执行指令的L2请求次数。对于10个嵌套余弦来说，原始版本和交错版本的数字都是0.011，而我们的版本是0.3。哎呀！

## 一个更小的临时缓冲区

解决办法是让我们的临时缓冲区更小，以适应L1缓存。L1缓存的大小约为16 kB，所以我们需要保持临时缓冲区小于这个数字。

如果我们的输入缓冲区更小，我们就需要一个更小的临时缓冲区。而我们可以让输入缓冲区更小！我们只需以块的方式处理我们的输入数据。第一个块可以是从输入中获取的前128个值，第二个块可以是接下来的128个值，依此类推。这个技术被称为*循环阻塞*，因为数据被以块的形式处理。阻塞后，我们的处理循环如下：

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

外循环在块级别上进行操作，内部循环在块内部处理数据。以下是进行循环分裂+循环阻塞的性能数字：

现在看起来更好了！最后一个版本，使用分裂和阻塞，速度最快！

*喜欢你正在阅读的内容吗？在[LinkedIn](https://www.linkedin.com/company/johnysswlab)，[Twitter](https://twitter.com/johnnysswlab)或[Mastodon](https://mastodon.online/@johnnysswlab)关注我们，并在新内容发布时立即收到通知。

需要软件性能帮助吗？[联系我们](https://johnnysswlab.com/contact/)！
