<!--yml
category: 未分类
date: 2024-05-27 13:24:40
-->

# Finding the First 10-digit Prime in (a Billion) Digits of e

> 来源：[https://www.hanshq.net/eprime.html](https://www.hanshq.net/eprime.html)

Back in 2004, Google ran a [recruitment campaign](https://googleblog.blogspot.com/2004/07/warning-we-brake-for-number-theory.html) where they posted the following billboard along the main freeway running through Silicon Valley, and later at [other locations](http://web.archive.org/web/20040912012847/http://www.boston.com/business/technology/articles/2004/09/09/comprehension_test/) in the country:

For those who managed to find the answer, a second problem awaited on the secret web site, and those who solved that were then encouraged to send in a job application.

Effectively [nerd sniped](http://xkcd.com/356/), I started playing with this problem sometime last year, and it led down a path to some excellent programming exercises.

This post describes a few different ways of solving the problem; from a Perl one-liner, to using hand-rolled fixed-point arithmetic (including an implementation of [Improved division by invariant integers](https://gmplib.org/~tege/division-paper.pdf)) or using binary splitting with GMP to compute a billion decimals of *e*.

The code is available in [eprime.c](files/eprime.c).

## Table of Contents

## Finding the 10-digit Prime With Perl

Searching for ["first 10-digit prime in e"](https://www.google.com/search?q=first+10-digit+prime+in+e) quickly yields the answer: 7427466391 is the number we're looking for. But let's assume we found this problem early on, and that the solution had yet to be posted online. We can still benefit from the web by searching for ["many digits of e"](https://www.google.com/search?q=many+digits+of+e). The [first hit](https://apod.nasa.gov/htmltest/gifcity/e.2mil) provides two million digits in which we can search for the solution, for example with a Perl one-liner:

```
$ curl -s https://apod.nasa.gov/htmltest/gifcity/e.2mil | tr -d '[:space:]' | \
          perl -MMath::Prime::Util=is_prime -MList::Util=first -nle \
          'print first { is_prime($_) } /(?=([1-9]\d{9}))/g'
7427466391 
```

(To install the required Perl module on a Debian system: sudo apt-get install libmath-prime-util-perl)

How does this work?

*   [curl](https://curl.haxx.se/docs/manpage.html) downloads the e.2mil file, which is then piped to the next command. The -s (for silent) flag makes it not print any other output.
*   [tr](https://linux.die.net/man/1/tr) deletes (-d) all whitespace characters, making sure the digits of *e* from the file end up on one line with no spaces in between them.
*   With Perl's [-M](https://perldoc.perl.org/perlrun.html#*-M*%5b*-*%5d_module_) flag, we import two Perl module subroutines: [Math::Prime::Util::is_prime](http://search.cpan.org/~danaj/Math-Prime-Util-0.65/lib/Math/Prime/Util.pm#is_prime) and [List::Util::first](http://search.cpan.org/~pevans/Scalar-List-Utils-1.48/lib/List/Util.pm#first).
*   [-n](https://perldoc.perl.org/perlrun.html#*-n*) makes Perl loop over the input, executing our code for each line, with the current line in [$_](https://perldoc.perl.org/perlvar.html#%24_).
*   [-l](https://perldoc.perl.org/perlrun.html#*-l*%5b_octnum_%5d) causes a newline to be added to each print statement.
*   [-e](https://perldoc.perl.org/perlrun.html#*-e*-_commandline_) specifies the code we want to run.
*   The /.../g [match operator](https://perldoc.perl.org/perlop.html#%2f_PATTERN_%2fmsixpodualngc) matches a regular expression against $_, which holds the current line. The [/g](https://perldoc.perl.org/perlop.html#Matching-in-list-context) (global matching) modifier makes it return a list of all the strings matched by capture groups.
*   The capture group in our regex, ([1-9]\d{9}), matches a non-zero digit followed by nine digits, i.e. a ten-digit number (leading zeros wouldn't count).
*   Perl starts looking for the next regex match where the previous match ends, and since we want to find all ten-digit numbers in the string, including those that overlap, we put the pattern in a [lookahead assertion](https://perldoc.perl.org/perlre.html#Lookaround-Assertions): (?=..). Our regex matches an empty string followed by a ten-digit number; the number isn't considered part of the match, but it does get captured.
*   first returns the first element from the list of regex matches for which is_prime returns true.
*   Finally, that element is printed.

## Computing *e* Ourselves

Computing mathematical constants to a large number of digits has been a popular sport (in select circles) for a long time. Pi is [especially popular](https://en.wikipedia.org/wiki/Chronology_of_computation_of_%CF%80), with the current record at 22.5 trillion digits, but *e* too has been computed to extreme precision ([at least](http://www.numberworld.org/digits/E/) to 5 trillion digits).

One way to approach the computation is from [Taylor's theorem](https://en.wikipedia.org/wiki/Taylor's_theorem), which for *e* gives us

where *R*, the remainder term, is bounded by

The factorial in the denominator of the remainder term means the series converges quickly.

By choosing a large enough *n* and adding the terms of the series, we can approximate *e* to any precision we want. To get *k* correct decimals, we need an *n* such that

Taking the logarithm of both sides and using [Stirling's approximation for factorials](https://en.wikipedia.org/wiki/Stirling's_approximation) gives us

Since the left-hand side is monotonic, we can use binary search to quickly find the smallest *n* that fulfills the inequality:

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

To compute 10,000 decimals, we need 3,249 terms.

For the computation of *e*, we cannot use the regular double data type, since it only has enough precision for a handful of decimals. Instead, we will use [GMP](https://gmplib.org/)'s arbitrary-precision [floating-point functions](https://gmplib.org/manual/Floating_002dpoint-Functions.html).

How many bits of precision do we need? Each decimal digit requires

bits, so to compute 10,000 decimals we define:

```
#ifndef NUM_DECIMALS
#define NUM_DECIMALS 10000ULL
#endif
#define NUM_DIGITS (1 + NUM_DECIMALS)
#define NUM_BITS (NUM_DIGITS * 3322 / 1000 + 999) 
```

(Note that we have to be careful not to lose any bits due to the truncating integer division.)

We need two floating-point variables: one for the sum of the terms (which approaches *e*), and one for the current term. The current term starts at 1, and then we divide it by 1, 2, 3, etc. so that in each loop iteration, its value is *1/i!*

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

We convert the final value to a string of decimal digits using [mpf_get_str](https://gmplib.org/manual/Converting-Floats.html#index-mpf_005fget_005fstr):

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

The string we get back starts with "2718..", i.e. there is an implicit decimal point after the first character.

Finally, we iterate over the string, checking for 10-digit primes with [mpz_probab_prime_p](https://gmplib.org/manual/Number-Theoretic-Functions.html#index-Prime-testing-functions):

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

To install GMP, build and run the program on Debian:

```
$ sudo apt-get install libgmp-dev
$ gcc -O3 -DNDEBUG eprime.c -lm -lgmp
$ ./a.out
7427466391 is prime 
```

On Mac, to install GMP from [MacPorts](https://www.macports.org/), build and run the program:

```
$ sudo port install gmp
$ clang -O3 -DNDEBUG -I/opt/local/include -L/opt/local/lib eprime.c -lm -lgmp
$ ./a.out
7427466391 is prime 
```

To print the digits, pass along -DPRINT_E:

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

The result can be compared against for example [this page](http://www-history.mcs.st-andrews.ac.uk/HistTopics/e_10000.html).

(Note that mpf_get_str has rounded the last decimal up to 9, and so has that page. If you print more decimals, you will see that the 10,000th decimal is actually 8, followed by 5674..)

As it turns out, the 10-digit prime occurs early in *e*, starting at the 99th decimal (in the second row above), so there is no need to compute all 10,000 decimals. But more decimals is more fun!

## Computing *e* Ourselves Without GMP

It seems amazing that just by repeatedly dividing and adding some numbers together, we end up with this sophisticated mathematical constant.

GMP did all the work in the code above. Perhaps if we implemented all of it ourselves, it would be even more satisfying? This seems like a worthwhile programming exercise.

### Fixed-Point Representation

Remember how numbers are represented in our regular decimal system. For example:

As long as we remember the position of the decimal point, we could simply store the number as an array of digits:

```
int nbr[6] = {1,2,3,4,5,6}; 
```

This is called fixed-point representation because the implicit decimal point occurs in a fixed position of the number (as opposed to floating-point, where it can change).

Since the instructions in modern computers perform arithmetic on 64-bit values, it's much more efficient to store numbers as arrays of 64-bit values rather than decimals.

As we know that the integer part of *e* is 2, we only concern ourselves with computing and storing the fractional part. We will represent it as an array of *n* 64-bit words:

### Fixed-Point Addition

To compute *e*, we only need to perform two operations: addition and division (what we did with mpf_add and mpf_div_ui in the GMP version).

Addition is straight-forward; we add the numbers place by place, starting at the least significant end and carrying the one as necessary (this is Algorithm A in The Art of Computer Programming, Section 4.3.1):

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

### Fixed-Point Division

Performing the division is trickier. Luckily, we only need to divide our n-word term number by a single word. That means the algorithm is straight-forward, essentially how we do [short division](https://en.wikipedia.org/wiki/Short_division) in school.

However, the algorithm relies on being able to divide a two-word divisor by a one-word dividend (dividing 128 bits by 64 bits in our case), "two-by-one division". Some CPUs have an instruction for that; for example, Intel x86's DIV does exactly what we need as long as the result fits in a single word (which it always will in our case). But it is not possible to express that division in standard C, and many CPUs don't have an instruction for it.

Instead, we will perform two-by-one division using a clever technique that relies on multiplying the divisor with the approximate reciprocal of the dividend. Not only does this solve the division problem for us, it solves it efficiently because we can reuse the reciprocal when dividing by the same value multiple times. Using X86's DIV instruction would be much slower since it can take up to 100 cycles.

The algorithm is described in Möller and Granlund "[Improved division by invariant integers](https://gmplib.org/~tege/division-paper.pdf)" (IEEE Trans. Comput. 2011). The paper is an improved version of Granlund and Montgomery "[Division by Invariant Integers using Multiplication"](https://gmplib.org/~tege/divcnst-pldi94.pdf) (PLDI'94) which is also the subject of Chapter 10 in [Hacker's Delight](http://www.hackersdelight.org/).

The approximate reciprocal is defined as

which when *d* is "normalised", meaning that its highest bit is set, fits in a 64-bit word. Basically it's *1/d* shifted up 128 bits, with some adjustments to make it fit in a 64-bit word.

The algorithms in the paper rely on performing 64-bit by 64-bit multiplications, the results of which can be up to 128 bits. Standard C only provides the lower 64 bits of such multiplications, which the paper refers to as umullo:

```
uint64_t umullo(uint64_t a, uint64_t b)
{
        return a * b;
} 
```

Most CPUs do provide the full result of the multiplication. With GCC or Clang we can use the non-standard __uint128_t type to implement umulhi (the high 64-bits of the result) and umul (both the high and low 64-bits). With Microsoft Visual C++ we can use intrinsics, and when none of those options are available, we can compute the result by hand using four 32-bit multiplications:

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

With these functions in place, we can proceed to implement reciprocal_word, which computes *v* using carefully implemented (and hard to follow in the paper) [Newton iteration](https://en.wikipedia.org/wiki/Newton's_method).

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

If we do have access to the X86 DIV instruction, it can be used to compute the approximate reciprocal directly. We use this in the assert above to check our work.

Using the reciprocal, 2-by-1 division is implemented as below, using two multiplications and some adjustments.

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

And with that in place, we can finally implement our n-by-1 division.

To normalize the dividend, it is convenient to have a function for getting the number of leading zeros in a word. Many CPUs have an instruction for that (X86 has BSR), and otherwise we can do it ourselves.

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

Since we shift the dividend to normalize it, we must also shift the divisor the same amount to get the correct result. We can do this in-place while performing the division (this "short division" algorithm is Exercise 16 in The Art of Computer Programming, Section 4.3.1):

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

### Computing *e* With Our Fixed-Point Routines

Now we can finally proceed to computing *e*. Note that since we're only computing the fraction, we skip the first terms (which add up to 2), and start with the third one. To initialize efrac and term to 0.5, we need to figure out what that is in the base we're using:

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

### Decimal Conversion

The code above leaves us with an array of carefully computed 64-bit words that represent the fractional part of *e*. How do we turn that into a decimal string?

We have:

If we multiply it by 10, the most significant digit, 7, gets moved to the integer position (in our case there is no integer position, so the 7 arrives as an overflow of the multiplication). We can repeat the process to extract one decimal at the time.

Instead of multiplying by 10 to get one decimal, we can multiply by 100 to get two, or any power of 10 to extract multiple decimals at the time. The largest power of 10 that fits in a 64-bit word is 10^19, so we will use that to extract 19 decimals at a time.

(The time complexity of this is quadratic: *O(n)* multiplications are performed *O(n)* times. GMP implements faster [algorithms](https://gmplib.org/manual/Binary-to-Radix.html#Binary-to-Radix) for binary to decimal conversion.)

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

Back in eprime_manual:

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

### Primality Testing

To check whether a number is prime, we will implement Algorithm P from The Art of Computer Programming, Section 4.5.4, also known as the [Miller-Rabin primality test](https://en.wikipedia.org/wiki/Miller%E2%80%93Rabin_primality_test).

This is a probabilistic test, which means it cannot tell us that a number is prime with absolute certainty, but it's good enough for our purposes, and it's fast.

The algorithm relies on modular exponentiation, which we can implement with [repeated squaring](https://en.wikipedia.org/wiki/Exponentiation_by_squaring) (also known as binary exponentiation). What makes it a little difficult is that we're dealing with 64-bit numbers, so we need to be able to compute 128-bit products and divide those by a 64-bit dividend. Luckily, we implemented exactly the necessary tools with umul and div2by1 above.

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

Back in eprime_manual:

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

### A Million Decimals

We can use our program to compute a million decimals:

```
$ gcc -O3 -march=native -DNDEBUG -DNUM_DECIMALS=1000000ULL -lm -lgmp eprime.c
$ time ./a.out
7427466391 is prime

real	2m0.800s
user	2m0.872s
sys	0m0.000s 
```

On my laptop, the "manual" version takes about 2 minutes. If I use X86's DIV instruction instead of multiplying by the reciprocal, it takes 4 minutes. The GMP version takes just over 1 minute; it uses fancier and more carefully implemented algorithms.

## Computing *e* Using Binary Splitting

While the algorithms above suffice for computing a million decimals, computing a billion decimals calls for a faster approach. The programs used for calculating *e*, *pi*, and other constants to trillions of decimals normally use a technique called *binary splitting*. My implementation is based on [the description](http://numbers.computation.free.fr/Constants/Algorithms/splitting.html) ([PostScript version](http://numbers.computation.free.fr/Constants/Algorithms/splitting.ps)) by Xavier Gourdon and Pascal Sebah.

Consider the series below.

When *a = 0* and *b = n*, it is equivalent to the Taylor series for *e* except that it's missing the first term, *1/0!*

To compute the fraction resulting from adding the terms together, we need to rewrite each term to use the common denominator, *Q(a,b)*:

To compute *Q(a,b)*, we can use binary splitting. Instead of iteratively multiplying with *a+1*, *a+2*, and so on, we can use a recursive approach, splitting the computation into two halves at each step and combining the results (binary splitting is really just another name for [divide and conquer](https://en.wikipedia.org/wiki/Divide_and_conquer_algorithm)):

When *Q(a,b)* is small, this isn't any faster than the iterative computation, because binary splitting performs the same number of operations. However, when the numbers are larger, the time required for each operation grows (often super-linearly) with the size of the numbers involved, and then we benefit from the binary splitting method reducing the size of the sub-problems.

(Besides reducing the size of the sub-problems, binary splitting is also a nice way to parallelize the computation by computing the sub-problems on separate threads.)

Computing the numerator of the series, *P(a,b)* is a little trickier. To rewrite each term on the common denominator, we need to multiply them with the factors missing from their denominators. For example, for the first term, we need to multibly by *a+2*, *a+3*, and so on all the way to *b*. For the second term, we start with *a+3*, and so on:

If we split *P(a,b)* into *P(a,m)* and *P(m,b)*, how can we combine them? Look at what we get:

*P(m,b)* matches the last terms of *P(a,b)*, so that seems fine.

*P(a,m)* looks like the first terms of *P(a,b)* except that the numerators are wrong; we need them to be *b!*. To fix this, we can multiply them by *b!/m!*, which we also know as *Q(m,b)*. So, we end up with:

In order for the computation to be fast, it's necessary to use a multiplication algorithm with good time complexity (in particular, the naive schoolbook algorithm with *O(mn)* time won't do). GMP implements [fancy multiplication algorithms](https://gmplib.org/manual/Multiplication-Algorithms.html), so we will rely on that for performance:

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

### A Billion Decimals

Using this version to compute a million decimals is significantly faster than what we saw [above](#million):

```
$ gcc -O3 -march=native -DNDEBUG -DNUM_DECIMALS=1000000ULL -lm -lgmp eprime.c
$ time ./a.out
7427466391 is prime

real	0m0.379s
user	0m0.376s
sys	0m0.004s 
```

How about a billion decimals? On my laptop, it takes about 20 minutes:

```
$ gcc -O3 -march=native -DNDEBUG -DNUM_DECIMALS=1000000000ULL -lm -lgmp eprime.c
$ time ./a.out
7427466391 is prime

real	21m27.575s
user	20m57.308s
sys	0m13.956s 
```

## Further Reading