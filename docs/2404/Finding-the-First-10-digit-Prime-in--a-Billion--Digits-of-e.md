<!--yml

category: 未分类

date: 2024-05-27 13:24:40

-->

# 在（十亿位）e中找到第一个10位质数

> 来源：[https://www.hanshq.net/eprime.html](https://www.hanshq.net/eprime.html)

2004年，谷歌运行了一个 [招聘活动](https://googleblog.blogspot.com/2004/07/warning-we-brake-for-number-theory.html)，在硅谷主要高速公路沿线的广告牌上发布了以下内容，后来也在 [其他地点](http://web.archive.org/web/20040912012847/http://www.boston.com/business/technology/articles/2004/09/09/comprehension_test/) 进行了发布：

对于那些找到答案的人，秘密网站上还等待着第二个问题，解决了这个问题的人随后被鼓励提交工作申请。

有效地 [怪胎抢夺](http://xkcd.com/356/)，我去年某个时候开始玩这个问题，它引导我进行了一些出色的编程练习。

本文描述了解决问题的几种不同方法；从Perl的一行代码，到使用手动编写的定点算术（包括[改进的不变整数除法](https://gmplib.org/~tege/division-paper.pdf)的实现），或者使用GMP进行二分拆分计算十亿位的*e*。

代码可以在 [eprime.c](files/eprime.c) 中找到。

## 目录

## 使用Perl找到10位质数

搜索 ["e中的第一个10位质数"](https://www.google.com/search?q=first+10-digit+prime+in+e) 很快就能找到答案：7427466391 是我们寻找的数字。但假设我们早期找到这个问题，并且解决方案尚未在网上发布。我们仍然可以通过搜索 ["e的许多位数"](https://www.google.com/search?q=many+digits+of+e) 在网上获益。[第一个链接](https://apod.nasa.gov/htmltest/gifcity/e.2mil) 提供了两百万位数字，我们可以在其中搜索解决方案，例如使用Perl的一行代码：

```
$ curl -s https://apod.nasa.gov/htmltest/gifcity/e.2mil | tr -d '[:space:]' | \
          perl -MMath::Prime::Util=is_prime -MList::Util=first -nle \
          'print first { is_prime($_) } /(?=([1-9]\d{9}))/g'
7427466391 
```

（在Debian系统上安装所需的Perl模块：sudo apt-get install libmath-prime-util-perl）

这是如何运作的？

+   [curl](https://curl.haxx.se/docs/manpage.html) 下载 e.2mil 文件，然后将其传输到下一个命令。-s（静默）标志使其不打印任何其他输出。

+   [tr](https://linux.die.net/man/1/tr) 删除（-d）所有空白字符，确保从文件中取出的*e*的数字都在一行上，中间没有空格。

+   使用Perl的 [-M](https://perldoc.perl.org/perlrun.html#*-M*%5b*-*%5d_module_) 标志，我们导入了两个Perl模块的子例程：[Math::Prime::Util::is_prime](http://search.cpan.org/~danaj/Math-Prime-Util-0.65/lib/Math/Prime/Util.pm#is_prime) 和 [List::Util::first](http://search.cpan.org/~pevans/Scalar-List-Utils-1.48/lib/List/Util.pm#first)。

+   [-n](https://perldoc.perl.org/perlrun.html#*-n*) 使Perl循环处理输入，对每行执行我们的代码，当前行存在 [$_](https://perldoc.perl.org/perlvar.html#%24_) 中。

+   [-l](https://perldoc.perl.org/perlrun.html#*-l*%5b_octnum_%5d)会在每个打印语句中添加一个换行符。

+   [-e](https://perldoc.perl.org/perlrun.html#*-e*-_commandline_)指定我们要运行的代码。

+   /.../g [匹配运算符](https://perldoc.perl.org/perlop.html#%2f_PATTERN_%2fmsixpodualngc)针对$_（当前行）匹配正则表达式。[/g](https://perldoc.perl.org/perlop.html#Matching-in-list-context)（全局匹配）修饰符使其返回所有由捕获组匹配的字符串列表。

+   我们正则表达式中的捕获组([1-9]\d{9})匹配一个非零数字后面跟着九个数字，即一个十位数（前导零不计）。

+   Perl开始寻找下一个正则表达式匹配，前一个匹配结束后，由于我们想在字符串中找到所有的十位数，包括那些重叠的数，我们将模式放在[前瞻断言](https://perldoc.perl.org/perlre.html#Lookaround-Assertions)中：(?=..)。我们的正则表达式匹配一个空字符串，后面跟着一个十位数；这个数字不被视为匹配的一部分，但确实被捕获。

+   首先返回正则表达式匹配列表中使is_prime返回true的第一个元素。

+   最后，打印该元素。

## 自己计算*e*

将数学常数计算到极大的位数一直是一个受欢迎的活动（在特定圈子内）。Pi是[特别流行的](https://en.wikipedia.org/wiki/Chronology_of_computation_of_%CF%80)，目前的记录为22.5万亿位，但*e*也已经计算到极端的精度（至少[到5万亿位](http://www.numberworld.org/digits/E/)）。

一种方法是从[泰勒定理](https://en.wikipedia.org/wiki/Taylor's_theorem)来计算*e*，对于*e*，我们得到

其中*R*，余项，受到限制

分母中的阶乘意味着余项快速收敛。

通过选择足够大的*n*并添加系列的项，我们可以按任意精度近似*e*。为了获得*k*位正确的小数，我们需要一个*n*，使得

取对数两边并使用[斯特林逼近公式计算阶乘](https://en.wikipedia.org/wiki/Stirling's_approximation)得到

由于左侧是单调的，我们可以使用二分搜索快速找到满足不等式的最小*n*：

```
uint64_t compute_n(uint64_t k)
{
        /* Find n such that
           (n + 1) * ln(n + 1) - (n + 1) > 1 + k * ln(10)
           using binary search.
         */

        uint64_t hi, lo, n;

        hi = UINT64_MAX;
        lo = 0;

        while (hi - lo > 1) {
                n = lo + (hi - lo) / 2;

                if ((n + 1) * log(n + 1) - (n + 1) <= 1 + k * log(10)) {
                        lo = n;
                } else {
                        hi = n;
                }
        }

        return hi;
} 
```

要计算10,000位小数，我们需要3,249个项。

对于计算*e*，我们不能使用常规的双精度数据类型，因为它只能保留少数几位小数的精度。相反，我们将使用[GMP](https://gmplib.org/)的任意精度[浮点函数](https://gmplib.org/manual/Floating_002dpoint-Functions.html)。

我们需要多少位的精度？每个十进制数字都需要

位，因此要计算10,000位小数我们定义：

```
#ifndef NUM_DECIMALS
#define NUM_DECIMALS 10000ULL
#endif
#define NUM_DIGITS (1 + NUM_DECIMALS)
#define NUM_BITS (NUM_DIGITS * 3322 / 1000 + 999) 
```

（注意，我们必须小心，不要因为截断的整数除法而丢失任何位。）

我们需要两个浮点变量：一个用于项的和（逼近 *e*），一个用于当前项。当前项从1开始，然后我们除以1, 2, 3等，以便在每次循环迭代中，其值为 *1/i!*

```
void eprime_gmp(void)
{
        uint64_t i, n;
        mpf_t e, term;
        char *s;
        mp_exp_t strexp;

        n = compute_n(NUM_DECIMALS);

        /* Compute e. */
        mpf_set_default_prec(NUM_BITS);
        mpf_init_set_ui(e, 1);
        mpf_init_set_ui(term, 1);

        for (i = 1; i <= n; i++) {
                mpf_div_ui(term, term, i);
                mpf_add(e, e, term);
        }

        mpf_clear(term); 
```

我们使用 [mpf_get_str](https://gmplib.org/manual/Converting-Floats.html#index-mpf_005fget_005fstr) 将最终值转换为十进制数字的字符串：

```
 /* Convert to string of decimal digits. */
        s = mpf_get_str(NULL, &strexp, 10, NUM_DIGITS, e);
        mpf_clear(e);
        assert(strexp == 1);
        assert(strlen(s) == NUM_DIGITS);
#ifdef PRINT_E
        printf("2.%s\n", &s[1]);
#endif 
```

我们获取的字符串以 "2718.." 开头，即第一个字符后有一个隐含的小数点。

最后，我们遍历字符串，使用 [mpz_probab_prime_p](https://gmplib.org/manual/Number-Theoretic-Functions.html#index-Prime-testing-functions) 检查10位素数：

```
 find_prime_gmp(&s[1]);
        free(s);
}

/* Find the first 10-digit prime in string s with length NUM_DECIMALS. */
void find_prime_gmp(const char *s)
{
        int i;
        mpz_t p;

        mpz_init(p);
        for (i = 1; i + 9 < NUM_DECIMALS; i++) {
                if (s[i] == '0' || (s[i + 9] - '0') % 2 == 0) {
                        /* Skip leading zeros and even numbers. */
                        continue;
                }

                gmp_sscanf(&s[i], "%10Zd", p);
                if (mpz_probab_prime_p(p, 20)) {
                        gmp_printf("%Zd is prime\n", p);
                        break;
                }
        }
        mpz_clear(p);
} 
```

要在Debian上安装GMP，请构建并运行程序：

```
$ sudo apt-get install libgmp-dev
$ gcc -O3 -DNDEBUG eprime.c -lm -lgmp
$ ./a.out
7427466391 is prime 
```

在Mac上，要从 [MacPorts](https://www.macports.org/) 安装GMP，请构建并运行程序：

```
$ sudo port install gmp
$ clang -O3 -DNDEBUG -I/opt/local/include -L/opt/local/lib eprime.c -lm -lgmp
$ ./a.out
7427466391 is prime 
```

要打印数字，请传递 -DPRINT_E：

```
$ gcc -O3 -DNDEBUG -DPRINT_E eprime.c -lm -lgmp
$ ./a.out
2.718281828459045235360287471352662497757247093699959574966967627724076630353547
59457138217852516642742746639193200305992181741359662904357290033429526059563073
81323286279434907632338298807531952510190115738341879307021540891499348841675092
44761460668082264800168477411853742345442437107539077744992069551702761838606261
33138458300075204493382656029760673711320070932870912744374704723069697720931014
...
91674197015125597825727074064318086014281490241467804723275976842696339357735429
30186739439716388611764209004068663398856841681003872389214483176070116684503887
21236436704331409115573328018297798873659091665961240202177855885487617616198937
07943800566633648843650891448055710397652146960276625835990519870423001794655367
89
7427466391 is prime 
```

可以将结果与例如 [此页面](http://www-history.mcs.st-andrews.ac.uk/HistTopics/e_10000.html) 进行比较。

（请注意，mpf_get_str已将最后一个小数四舍五入到9，该页面也是如此。如果打印更多小数，您将看到第10000个小数实际上是8，然后是5674..）

结果表明，10位素数出现在 *e* 的早期，从第99个小数（在上面的第二行）开始，因此无需计算全部10000位小数。但更多的小数更有趣！

## 自行计算 *e* 而不使用 GMP

只需反复分割和添加一些数字，我们就能得到这个复杂的数学常数，看起来很不可思议。

在上述代码中，GMP完成了所有工作。也许如果我们自己实现所有这些，会更加令人满意？这似乎是一个值得的编程练习。

### 固定点表示

记住我们常规十进制系统中数字的表示方式。例如：

只要记住小数点的位置，我们可以简单地将数字存储为数字数组：

```
int nbr[6] = {1,2,3,4,5,6}; 
```

这称为固定点表示，因为隐含的小数点位于数字的固定位置（与浮点表示不同，其小数点位置可以变化）。

由于现代计算机指令在64位值上执行算术运算，将数字存储为64位值数组而不是小数更有效率。

由于我们知道 *e* 的整数部分是2，我们只关心计算和存储其分数部分。我们将其表示为 *n* 个64位字的数组：

### 固定点加法

要计算 *e*，我们只需执行两种操作：加法和除法（类似于我们在GMP版本中使用mpf_add和mpf_div_ui所做的操作）。

加法很简单；我们按位加数，从最不显著位开始，并在必要时进行进位（这是《计算机程序设计艺术》第4.3.1节中的算法A）：

```
/* Add n-place integers u and v into w. */
void addn(int n, const uint64_t *u, const uint64_t *v, uint64_t *w)
{
        bool carry, carry_a, carry_b;
        uint64_t sum_a, sum_b;
        int j;

        carry = false;

        for (j = 0; j < n; j++) {
                sum_a = u[j] + carry;
                carry_a = (sum_a < u[j]);

                sum_b = sum_a + v[j];
                carry_b = (sum_b < sum_a);

                w[j] = sum_b;
                assert(carry_a + carry_b <= 1);
                carry = carry_a + carry_b;
        }
} 
```

### 固定点除法

执行除法更加棘手。幸运的是，我们只需要将我们的n字项数除以一个单字。这意味着算法非常直接，基本上就是我们在学校里做[短除法](https://en.wikipedia.org/wiki/Short_division)的方式。

然而，该算法依赖于能够将一个双字除数除以一个单字被除数（在我们的情况下是将128位除以64位），“双除一”操作。一些CPU有这样的指令；例如，Intel x86 的 DIV 指令正好满足我们的需求，只要结果能够适应单字（在我们的情况下总是可以的）。但是在标准C中无法表达这种除法，而且很多CPU也没有这样的指令。

相反，我们将使用一个巧妙的技术进行双对一除法，该技术依赖于将除数乘以被除数的近似倒数。这不仅解决了我们的除法问题，而且因为我们可以在多次除以相同值时重复使用倒数，所以它的效率也高。使用X86的DIV指令会慢得多，因为它可能需要多达100个周期。

该算法在 Möller 和 Granlund 的论文"[Improved division by invariant integers](https://gmplib.org/~tege/division-paper.pdf)"（IEEE Trans. Comput. 2011）中有描述。该论文是 Granlund 和 Montgomery 的"[Division by Invariant Integers using Multiplication](https://gmplib.org/~tege/divcnst-pldi94.pdf)"（PLDI'94）的改进版本，也是[Hacker's Delight](http://www.hackersdelight.org/)第10章的主题。

近似倒数定义为

当 *d* 被“标准化”时，意味着它的最高位被设置，适合于64位字。基本上它是将 *1/d* 向上移动了128位，做了一些调整使其适应64位字。

论文中的算法依赖于执行64位乘以64位的乘法，其结果可以达到128位。标准C只提供这种乘法的低64位，论文中称为umullo：

```
uint64_t umullo(uint64_t a, uint64_t b)
{
        return a * b;
} 
```

大多数CPU确实提供了乘法的完整结果。使用GCC或Clang，我们可以使用非标准的__uint128_t类型来实现umulhi（结果的高64位）和umul（高位和低位的64位）。使用Microsoft Visual C++，我们可以使用内部函数，在没有这些选项可用时，我们可以通过手动计算使用四个32位乘法来得到结果：

```
#if defined(__GNUC__)
void umul(uint64_t a, uint64_t b, uint64_t *p1, uint64_t *p0)
{
        __uint128_t p = (__uint128_t)a * b;
        *p1 = (uint64_t)(p >> 64);
        *p0 = (uint64_t)p;
}

uint64_t umulhi(uint64_t a, uint64_t b)
{
        return (uint64_t)(((__uint128_t)a * b) >> 64);
}
#elif defined(_MSC_VER) && defined(_M_X64)
#include <intrin.h> void umul(uint64_t a, uint64_t b, uint64_t *p1, uint64_t *p0)
{
        *p0 = _umul128(a, b, p1);
}

uint64_t umulhi(uint64_t a, uint64_t b)
{
        return __umulh(a, b);
}
#else
void umul(uint64_t x, uint64_t y, uint64_t *p1, uint64_t *p0)
{
        uint32_t x0 = (uint32_t)x, x1 = (uint32_t)(x >> 32);
        uint32_t y0 = (uint32_t)y, y1 = (uint32_t)(y >> 32);
        uint64_t p;
        uint32_t res0, res1, res2, res3;

        p = (uint64_t)x0 * y0;
        res0 = (uint32_t)p;
        res1 = (uint32_t)(p >> 32);

        p = (uint64_t)x0 * y1;
        res1 = (uint32_t)(p += res1);
        res2 = (uint32_t)(p >> 32);

        p = (uint64_t)x1 * y0;
        res1 = (uint32_t)(p += res1);
        p >>= 32;
        res2 = (uint32_t)(p += res2);
        res3 = (uint32_t)(p >> 32);

        p = (uint64_t)x1 * y1;
        res2 = (uint32_t)(p += res2);
        res3 += (uint32_t)(p >> 32);

        *p0 = ((uint64_t)res1 << 32) | res0;
        *p1 = ((uint64_t)res3 << 32) | res2;
}

uint64_t umulhi(uint64_t a, uint64_t b)
{
        uint64_t p0, p1;
        umul(a, b, &p1, &p0);
        return p1;
}
#endif 
```

有了这些函数，我们可以继续实现reciprocal_word，这个函数使用了精心实现的（在论文中难以理解的）[牛顿迭代法](https://en.wikipedia.org/wiki/Newton's_method)来计算*v*。

```
/* Algorithm 2 from Möller and Granlund
   "Improved division by invariant integers". */
uint64_t reciprocal_word(uint64_t d)
{
        uint64_t d0, d9, d40, d63, v0, v1, v2, ehat, v3, v4, hi, lo;

        static const uint64_t table[] = {
        /* Generated with:
           for (int i = (1 << 8); i < (1 << 9); i++)
                   printf("0x%03x,\n", ((1 << 19) - 3 * (1 << 8)) / i); */
        0x7fd, 0x7f5, 0x7ed, 0x7e5, 0x7dd, 0x7d5, 0x7ce, 0x7c6, 0x7bf, 0x7b7,
        0x7b0, 0x7a8, 0x7a1, 0x79a, 0x792, 0x78b, 0x784, 0x77d, 0x776, 0x76f,
        0x768, 0x761, 0x75b, 0x754, 0x74d, 0x747, 0x740, 0x739, 0x733, 0x72c,
        0x726, 0x720, 0x719, 0x713, 0x70d, 0x707, 0x700, 0x6fa, 0x6f4, 0x6ee,
        0x6e8, 0x6e2, 0x6dc, 0x6d6, 0x6d1, 0x6cb, 0x6c5, 0x6bf, 0x6ba, 0x6b4,
        0x6ae, 0x6a9, 0x6a3, 0x69e, 0x698, 0x693, 0x68d, 0x688, 0x683, 0x67d,
        0x678, 0x673, 0x66e, 0x669, 0x664, 0x65e, 0x659, 0x654, 0x64f, 0x64a,
        0x645, 0x640, 0x63c, 0x637, 0x632, 0x62d, 0x628, 0x624, 0x61f, 0x61a,
        0x616, 0x611, 0x60c, 0x608, 0x603, 0x5ff, 0x5fa, 0x5f6, 0x5f1, 0x5ed,
        0x5e9, 0x5e4, 0x5e0, 0x5dc, 0x5d7, 0x5d3, 0x5cf, 0x5cb, 0x5c6, 0x5c2,
        0x5be, 0x5ba, 0x5b6, 0x5b2, 0x5ae, 0x5aa, 0x5a6, 0x5a2, 0x59e, 0x59a,
        0x596, 0x592, 0x58e, 0x58a, 0x586, 0x583, 0x57f, 0x57b, 0x577, 0x574,
        0x570, 0x56c, 0x568, 0x565, 0x561, 0x55e, 0x55a, 0x556, 0x553, 0x54f,
        0x54c, 0x548, 0x545, 0x541, 0x53e, 0x53a, 0x537, 0x534, 0x530, 0x52d,
        0x52a, 0x526, 0x523, 0x520, 0x51c, 0x519, 0x516, 0x513, 0x50f, 0x50c,
        0x509, 0x506, 0x503, 0x500, 0x4fc, 0x4f9, 0x4f6, 0x4f3, 0x4f0, 0x4ed,
        0x4ea, 0x4e7, 0x4e4, 0x4e1, 0x4de, 0x4db, 0x4d8, 0x4d5, 0x4d2, 0x4cf,
        0x4cc, 0x4ca, 0x4c7, 0x4c4, 0x4c1, 0x4be, 0x4bb, 0x4b9, 0x4b6, 0x4b3,
        0x4b0, 0x4ad, 0x4ab, 0x4a8, 0x4a5, 0x4a3, 0x4a0, 0x49d, 0x49b, 0x498,
        0x495, 0x493, 0x490, 0x48d, 0x48b, 0x488, 0x486, 0x483, 0x481, 0x47e,
        0x47c, 0x479, 0x477, 0x474, 0x472, 0x46f, 0x46d, 0x46a, 0x468, 0x465,
        0x463, 0x461, 0x45e, 0x45c, 0x459, 0x457, 0x455, 0x452, 0x450, 0x44e,
        0x44b, 0x449, 0x447, 0x444, 0x442, 0x440, 0x43e, 0x43b, 0x439, 0x437,
        0x435, 0x432, 0x430, 0x42e, 0x42c, 0x42a, 0x428, 0x425, 0x423, 0x421,
        0x41f, 0x41d, 0x41b, 0x419, 0x417, 0x414, 0x412, 0x410, 0x40e, 0x40c,
        0x40a, 0x408, 0x406, 0x404, 0x402, 0x400
        };

        assert(d > UINT64_MAX / 2 && "d must be normalized.");

        d0 = d & 1;
        d9 = d >> 55;
        d40 = (d >> 24) + 1;
        d63 = (d >> 1) + d0;

        v0 = table[d9 - (1 << 8)];
        v1 = (v0 << 11) - (umullo(umullo(v0, v0), d40) >> 40) - 1;
        v2 = (v1 << 13) + (umullo(v1, (1ULL << 60) - umullo(v1, d40)) >> 47);
        ehat = (v2 >> 1) * d0 - umullo(v2, d63);
        v3 = (v2 << 31) + (umulhi(v2, ehat) >> 1);
        umul(v3, d, &hi, &lo);
        v4 = v3 - (hi + d + (lo + d < lo));

#if defined(__GNUC__) && defined(__x86_64__) && !defined(NDEBUG)
        uint64_t asmq, asmr;
        __asm__("divq %2" : "=a"(asmq), "=d"(asmr)
                          : "r"(d), "d"(UINT64_MAX-d), "a"(UINT64_MAX) : "cc");
        assert(v4 == asmq);
#endif

        return v4;
} 
```

如果我们有X86 DIV指令的访问权限，可以直接用它来计算近似倒数。我们在上面的断言中使用它来检查我们的工作是否正确。

使用倒数，2对1除法的实现如下，使用了两次乘法和一些调整。

```
/* Algorithm 4 from Möller and Granlund
   "Improved division by invariant integers".
   Divide u1:u0 by d, returning the quotient and storing the remainder in r.
   v is the approximate reciprocal of d, as computed by reciprocal_word. */
uint64_t div2by1(uint64_t u1, uint64_t u0, uint64_t d, uint64_t *r, uint64_t v)
{
        uint64_t q0, q1;

        assert(u1 < d && "The quotient must fit in one word.");
        assert(d > UINT64_MAX / 2 && "d must be normalized.");

        umul(v, u1, &q1, &q0);
        q0 = q0 + u0;
        q1 = q1 + u1 + (q0 < u0);
        q1++;
        *r = u0 - umullo(q1, d);

        q1 = (*r > q0) ? q1 - 1 : q1;
        *r = (*r > q0) ? *r + d : *r;

        if (*r >= d) {
                q1++;
                *r -= d;
        }

#if defined(__GNUC__) && defined(__x86_64__) && !defined(NDEBUG)
        uint64_t asmq, asmr;
        __asm__("divq %2" : "=a"(asmq), "=d"(asmr)
                          : "r"(d), "d"(u1), "a"(u0) : "cc");
        assert(q1 == asmq && *r == asmr);
#endif

        return q1;
} 
```

有了这一步，我们最终可以实现我们的n对1除法。

为了标准化被除数，有一个函数可以得到字中前导零的数量是很方便的。许多CPU有这种指令（X86有BSR），否则我们可以自己做。

```
/* Count leading zeros. */
#if defined(__GNUC__)
int clz(uint64_t x)
{
        assert(x != 0);
        return __builtin_clzll(x);
}
#elif defined(_MSC_VER) && defined(_M_X64)
#include <intrin.h> int clz(uint64_t x)
{
        assert(x != 0);
        return __lzcnt64(x);
}
#else
int clz(uint64_t x)
{
        int n = 0;
        assert(x != 0);
        while ((x << n) <= UINT64_MAX / 2) n++;
        return n;
}
#endif

/* Right-shift that also handles the 64 case. */
uint64_t shr(uint64_t x, int n)
{
        return n < 64 ? (x >> n) : 0;
} 
```

由于我们将被除数移位以使其标准化，我们必须同时移动除数相同的量以获得正确的结果。我们可以在执行除法时原地执行此操作（这种“短除法”算法是《计算机程序设计艺术》第4.3.1节中的练习16）：

```
/* Divide n-place integer u by d, yielding n-place quotient q. */
void divnby1(int n, const uint64_t *u, uint64_t d, uint64_t *q)
{
        uint64_t v, k, ui;
        int l, i;

        assert(d != 0);
        assert(n > 0);

        /* Normalize d, storing the shift amount in l. */
        l = clz(d);
        d <<= l;

        /* Compute the reciprocal. */
        v = reciprocal_word(d);

        /* Perform the division. */
        k = shr(u[n - 1], 64 - l);
        for (i = n - 1; i >= 1; i--) {
                ui = (u[i] << l) | shr(u[i - 1], 64 - l);
                q[i] = div2by1(k, ui, d, &k, v);
        }
        q[0] = div2by1(k, u[0] << l, d, &k, v);
} 
```

### 使用我们的固定点例程计算*e*

现在我们终于可以继续计算*e*了。请注意，由于我们只计算分数部分，我们跳过了前几项（它们加起来是2），并从第三项开始。为了将 efrac 和 term 初始化为 0.5，在我们使用的基数中，我们需要弄清楚那是什么：

```
void eprime_manual(void)
{
        int n, i;
        uint64_t *efrac;
        uint64_t *term;
        uint64_t d;
        char *s;

        efrac = calloc(NUM_WORDS, sizeof(*efrac));
        term = calloc(NUM_WORDS, sizeof(*efrac));

        /* Start efrac and term at 0.5\. */
        efrac[NUM_WORDS - 1] = (1ULL << 63);
        term[NUM_WORDS - 1] = (1ULL << 63);

        /* Sum the series. */
        n = compute_n(NUM_DECIMALS);
        for (i = 3; i <= n; i++) {
                divnby1(NUM_WORDS, term, (uint64_t)i, term);
                addn(NUM_WORDS, efrac, term, efrac);
        } 
```

### 十进制转换

上述代码给出了一个精心计算的64位字数组，它表示*e*的小数部分。我们如何将其转换为一个十进制字符串？

我们有：

如果我们乘以10，最高有效位7被移动到整数位置（在我们的例子中没有整数位置，所以7作为乘法溢出到达）。我们可以重复这个过程一次提取一个小数。

我们可以通过乘以10来获得一个小数，也可以乘以100来获得两个小数，或者任何幂的10来一次性提取多个小数。64位字中适合的最大的10的幂是10^19，所以我们将使用它来一次性提取19个小数。

（这个时间复杂度是二次的：*O(n)*次乘法执行了*O(n)*次。GMP实现了更快的[算法](https://gmplib.org/manual/Binary-to-Radix.html#Binary-to-Radix)用于二进制到十进制的转换。）

```
/* Multiply n-place integer u by x in place, returning the overflow word. */
uint64_t mulnby1(int n, uint64_t *u, uint64_t x)
{
        uint64_t k, p1, p0;
        int i;

        k = 0;

        for (i = 0; i < n; i++) {
                umul(u[i], x, &p1, &p0);
                u[i] = p0 + k;
                k = p1 + (u[i] < p0);
        }

        return k;
} 
```

回到eprime_manual：

```
 /* Convert to decimal. */
        s = malloc(19 * (NUM_DECIMALS / 19 + 18) + 1);
        for (i = 0; i < NUM_DECIMALS; i += 19) {
                d = mulnby1(NUM_WORDS, efrac, 10000000000000000000ULL);
                sprintf(&s[i], "%019" PRIu64, d);
        }
#ifdef PRINT_E
        printf("2.%.*s\n", NUM_DECIMALS, s);
#endif 
```

### 素性测试

要检查一个数是否为素数，我们将实现《计算机程序设计艺术》第4.5.4节中的算法P，也称为[Miller-Rabin素性测试](https://en.wikipedia.org/wiki/Miller%E2%80%93Rabin_primality_test)。

这是一个概率测试，这意味着它不能绝对确定一个数是素数，但对我们的目的来说已经足够了，而且速度很快。

该算法依赖于模指数运算，我们可以通过[重复平方法](https://en.wikipedia.org/wiki/Exponentiation_by_squaring)（也称为二进制指数运算）来实现。稍微困难的是我们处理64位数字，所以我们需要能够计算128位乘积，并将其除以64位被除数。幸运的是，我们以上正好实现了必要的工具，使用umul和div2by1。

```
/* Compute x * y mod n, where n << s is normalized and
   v is the approximate reciprocal of n << s. */
uint64_t mulmodn(uint64_t x, uint64_t y, uint64_t n, int s, uint64_t v)
{
        uint64_t hi, lo, r;

        assert(s >= 0 && s < 64);
        assert((n << s) > UINT64_MAX / 2 && "n << s is normalized.");
        assert(x < n && y < n);

        umul(x, y, &hi, &lo);
        div2by1((hi << s) | shr(lo, 64 - s), lo << s, n << s, &r, v);

        return r >> s;
}

/* Compute x^p mod n by means of left-to-right binary exponentiation. */
uint64_t powmodn(uint64_t x, uint64_t p, uint64_t n)
{
        uint64_t res, v;
        int i, l, s;

        assert(x > 0 && x < n);

        s = clz(n);
        v = reciprocal_word(n << s);

        res = x;
        l = 63 - clz(p);

        for (i = l - 1; i >= 0; i--) {
                res = mulmodn(res, res, n, s, v);
                if (p & (1ULL << i)) {
                        res = mulmodn(res, x, n, s, v);
                }
        }

        return res;
}

/* Miller-Rabin primality test a.k.a. "Algorithm P" in TAOCP 4.5.4\. */
bool is_prob_prime(uint64_t n)
{
        uint64_t q, x, y;
        int i, j, k;

        if (n == 2) {
                return true;
        } else if (n < 2 || n % 2 == 0) {
                return false;
        }

        /* Find q and k such that n = 1 + 2^k * q, where q is odd. */
        k = 0;
        q = n - 1;
        while (q % 2 == 0) {
                k++;
                q /= 2;
        }

        for (i = 0; i < 25; i++) {
                x = ((uint64_t)rand() << 32) | (uint64_t)rand();
                x = x % (n - 2) + 2;
                assert(x > 1 && x < n);

                j = 0;
                y = powmodn(x, q, n);

                for (;;) {
                        if (y == n - 1 || (j == 0 && y == 1)) {
                                /* Maybe prime; try another x. */
                                break;
                        } else if (y == 1 && j > 0) {
                                return false;
                        }

                        j++;
                        if (j >= k) {
                                return false;
                        }
                        y = powmodn(y, 2, n);
                }
        }

        return true;
} 
```

回到eprime_manual：

```
 /* Find the first prime. */
        for (i = 0; i + 9 < NUM_DECIMALS; i++) {
                if (s[i] == '0' || (s[i + 9] - '0') % 2 == 0) {
                        /* Skip leading zeros and even numbers. */
                        continue;
                }

                sscanf(&s[i], "%10" SCNu64, &d);
                if (is_prob_prime(d)) {
                        printf("%" PRIu64 " is prime\n", d);
                        break;
                }
        }

        free(term);
        free(efrac);
        free(s);
} 
```

### 一百万位小数

我们可以使用我们的程序计算一百万位小数：

```
$ gcc -O3 -march=native -DNDEBUG -DNUM_DECIMALS=1000000ULL -lm -lgmp eprime.c
$ time ./a.out
7427466391 is prime

real	2m0.800s
user	2m0.872s
sys	0m0.000s 
```

在我的笔记本电脑上，“手动”版本大约需要2分钟。如果我使用X86的DIV指令而不是乘以倒数，需要4分钟。GMP版本仅需超过1分钟；它使用更高级别和更谨慎实施的算法。

## 使用二分法拆分计算*e*

虽然上面的算法足以计算一百万位小数，但计算十亿位小数需要更快的方法。用于计算 *e*、*pi* 和其他常数到万亿位小数的程序通常使用一种称为“二分法”的技术。我的实现基于[Xavier Gourdon 和 Pascal Sebah 的描述](http://numbers.computation.free.fr/Constants/Algorithms/splitting.html)（[PostScript 版本](http://numbers.computation.free.fr/Constants/Algorithms/splitting.ps)）。

考虑下面的级数。

当 *a = 0* 且 *b = n* 时，这相当于 *e* 的泰勒级数，只是缺少第一项 *1/0!*。

要计算加起来的项得到的分数，我们需要将每一项重新写成使用公共分母 *Q(a,b)* 的形式：

要计算 *Q(a,b)*，我们可以使用二分法。不是迭代地乘以 *a+1*、*a+2* 等，而是使用递归方法，在每一步将计算拆分为两半并组合结果（二分法实际上就是[分治法](https://en.wikipedia.org/wiki/Divide_and_conquer_algorithm)的另一个名称）：

当 *Q(a,b)* 较小时，这并不比迭代计算更快，因为二分法执行的操作数量相同。然而，当数字变大时，每次操作所需的时间（通常是超线性增长）随着所涉及数字的大小而增加，此时我们从二分法中受益，它通过减小子问题的规模来提高效率。

（除了减小子问题的规模外，二分法还是一种将计算并行化的好方法，通过在不同线程上计算子问题来实现。）

计算级数的分子 *P(a,b)* 稍微复杂一些。为了将每一项重新写成公共分母的形式，我们需要将它们乘以其分母所缺失的因子。例如，对于第一项，我们需要乘以 *a+2*、*a+3*，一直到 *b*。对于第二项，我们从 *a+3* 开始，依此类推：

如果我们将 *P(a,b)* 分成 *P(a,m)* 和 *P(m,b)*，如何将它们组合起来？看看我们得到了什么：

*P(m,b)* 匹配 *P(a,b)* 的最后一项，所以看起来没问题。

*P(a,m)* 看起来像 *P(a,b)* 的前面几项，只是分子不对；我们需要它们是 *b!*。为了修正这一点，我们可以将它们乘以 *b!/m!*，也就是我们称之为 *Q(m,b)* 的东西。因此，最终我们得到：

为了使计算快速，需要使用时间复杂度好的乘法算法（特别是，简单的竖式乘法算法，时间复杂度为 *O(mn)*，是不够的）。GMP 实现了[高级乘法算法](https://gmplib.org/manual/Multiplication-Algorithms.html)，因此我们会依赖它来提升性能：

```
void binsplit(uint64_t a, uint64_t b, mpz_t p, mpz_t q)
{
        uint64_t m;
        mpz_t pmb, qmb;

        assert(b > a);

        if (b - a == 1) {
                mpz_set_ui(p, 1);
                mpz_set_ui(q, b);
                return;
        }

        m = a + (b - a) / 2;
        mpz_init(pmb);
        mpz_init(qmb);

        /* Compute p(a, m) and q(a, m), storing the results in p and q to avoid
         * allocating extra variables. */
        binsplit(a, m, p, q);

        /* Compute p(m, b) and q(m, b) */
        binsplit(m, b, pmb, qmb);

        /* p(a,b) = p(a,m) * q(m,b) + p(m,b) */
        mpz_mul(p, p, qmb);
        mpz_add(p, p, pmb);

        /* q(a,b) = q(a,m) * q(m,b) */
        mpz_mul(q, q, qmb);

        mpz_clear(pmb);
        mpz_clear(qmb);
}

void eprime_binsplit(void)
{
        uint64_t n;
        mpz_t p, q;
        mpf_t e, qf;
        char *s;
        mp_exp_t strexp;

        n = compute_n(NUM_DECIMALS);

        /* Compute the fraction. */
        mpz_init(p);
        mpz_init(q);
        binsplit(0, n, p, q);

        /* Divide to rational. */
        mpf_set_default_prec(NUM_BITS);
        mpf_init(e);
        mpf_init(qf);
        mpf_set_z(e, p);
        mpf_set_z(qf, q);
        mpf_div(e, e, qf);
        mpz_clear(p);
        mpz_clear(q);
        mpf_clear(qf);

        /* Add the missing 1 term. */
        mpf_add_ui(e, e, 1);

        /* Convert to string of decimal digits. */
        s = mpf_get_str(NULL, &strexp, 10, NUM_DIGITS, e);
        mpf_clear(e);
        assert(strexp == 1);
        assert(strlen(s) == NUM_DIGITS);
#ifdef PRINT_E
        printf("2.%s\n", &s[1]);
#endif
        find_prime_gmp(&s[1]);
        free(s);
} 
```

### 十亿位小数

使用这个版本计算一百万位小数比我们在[上文](#million)看到的要快得多：

```
$ gcc -O3 -march=native -DNDEBUG -DNUM_DECIMALS=1000000ULL -lm -lgmp eprime.c
$ time ./a.out
7427466391 is prime

real	0m0.379s
user	0m0.376s
sys	0m0.004s 
```

十亿位小数怎么样？在我的笔记本上，大约需要 20 分钟：

```
$ gcc -O3 -march=native -DNDEBUG -DNUM_DECIMALS=1000000000ULL -lm -lgmp eprime.c
$ time ./a.out
7427466391 is prime

real	21m27.575s
user	20m57.308s
sys	0m13.956s 
```

## 进一步阅读
