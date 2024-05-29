<!--yml
category: 未分类
date: 2024-05-29 13:20:31
-->

# Iterating over Bit Sets quickly

> 来源：[https://alexharri.com/blog/bit-set-iteration](https://alexharri.com/blog/bit-set-iteration)

<main class="css-fkkl8v">

# Iterating over Bit Sets quickly

January 6, 2024

Welcome to part 2 of my 2-part series on bit sets! If you're not very familiar with bit manipulation — or don't know what bit sets are — I recommend reading part 1 first: [Bit Sets: An introduction to bit manipulation](/blog/bit-sets).

If you know your bit manipulation, then you can freely skip part 1.

[Bit sets](https://en.wikipedia.org/wiki/Bit_array) — also known as bit arrays or bit vectors — are a highly compact data structure that stores a list of bits. They are often used to represent a set of integers or an array of booleans.

In [part 1](/blog/bit-sets) we started writing a `BitSet` class, and implemented a few basic methods:

```
class  BitSet  {  private words:  number[];  add(index:  number):  void;  remove(index:  number):  void;  has(index:  number):  boolean}
```

The bits of our bit set are stored in a `number[]` called `words`. Since JavaScript only [supports 32-bit integers](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Number#fixed-width_number_conversion), each [word](https://en.wikipedia.org/wiki/Word_(computer_architecture)) stores 32 bits (the first word stores bits 1-32, the second word stores bits 33-64, and so on).

In this post, we'll tackle `BitSet.forEach`. We'll start off implementing the naive approach where we iterate over the bits and see how far we can optimize that approach.

We'll then learn about [two's complement](https://en.wikipedia.org/wiki/Two%27s_complement) and [Hamming weights](https://en.wikipedia.org/wiki/Hamming_weight), and how exploiting those lets us iterate over bit sets *really* quickly.

## Implementing BitSet.forEach

The `BitSet.forEach` method should invoke a callback for every bit that is set to 1, with the index of that bit.

```
class  BitSet  {  forEach(callback:  (index:  number)  =>  void)  {  }}
```

To start, we'll iterate over every word in `words`:

```
const words =  this.words;for  (let wordIndex =  0; wordIndex < words.length; wordIndex++)  {  const word = words[wordIndex];}
```

For each word, we'll run through the bits in ascending order:

```
for  (let i =  0; i <  WORD_LEN; i++)  {}
```

`WORD_LEN` is set to 32: the number of bits in each `word`

We can check whether the bit is set via `(word & (1 << i)) !== 0`:

```
const bitIsSetToOne =  (word &  (1  << i))  !==  0;
```

If the bit is non-zero, we'll want to invoke `callback` with the index of the bit:

```
for  (let i =  0; i <  WORD_LEN; i++)  {  const bitIsSetToOne =  (word &  (1  << i))  !==  0;  if  (bitIsSetToOne)  {  callback(index)

             // Cannot find name 'index'.

  }}
```

We can compute the bit's `index` in the bit set like so:

```
const index =  (wordIndex <<  WORD_LOG)  + i;
```

`WORD_LOG` is set to 5: the base 2 logarithm of 32

The expression `wordIndex << WORD_LOG` is equivalent to `wordIndex * WORD_LEN` because left shifting by one is equivalent to multiplying by 2 (and `2 ** WORD_LOG` equals `WORD_LEN`).

```
1  <<  WORD_LOG3  <<  WORD_LOG
```

And so, we have a basic implementation.

```
class  BitSet  {  forEach(callback:  (index:  number)  =>  void)  {  const words =  this.words;  for  (let wordIndex =  0; wordIndex < words.length; wordIndex++)  {  const word = words[wordIndex];  for  (let i =  0; i <  WORD_LEN; i++)  {  const bitIsSetToOne =  (word &  (1  << i))  !==  0;  if  (bitIsSetToOne)  {  const index =  (wordIndex <<  WORD_LOG)  + i;  callback(index);  }  }  }  }}
```

## Optimizing `BitSet.forEach`

We've now got a working implementation for `BitSet.forEach`. Can we optimize it further?

The first thing that pops out to me is that we always iterate over every bit in every word. We can skip words with no set bits with a cheap `word === 0` check.

```
for  (let wordIndex =  0; wordIndex < words.length; wordIndex++)  {  const word = words[wordIndex];  if  (word ===  0)  continue;  }
```

This won't do much for dense sets where most words have some bits set, but this skips a lot of work for sparse sets.

```
[  00000000,  01000000,  00000000,  00000000,  0000001  ][  11101101,  01110001,  10110101,  11010001,  0101101  ]
```

But how significant are the performance gains from this optimization? Let's figure it out by running some benchmarks.

We'll run our benchmarks for bit sets with various densities:

```
const densities =  [1,  0.75,  0.5,  0.25,  0.1,  0.05,  0.01,  0.001];
```

For each density, we'll create a bit set with 100 million bits.

```
const bitsetsAndDensities = densities.map((density)  =>  ({ density, bitset:  makeBitSet(100_000_000, density),}));
```

The `makeBitSet` method is implemented like so:

```
function  makeBitSet(size:  number, density:  number)  {  const bitset =  new  BitSet();  for  (let i =  0; i < size; i++)  {  if  (Math.random()  < density)  { bitset.add(i);  }  }  return bitset;}
```

Now that we've created our bit sets, we'll run the `forEach` method for each of them and log out how long it takes.

```
for  (const  { bitset, density }  of bitsetsAndDensities)  {  profile(  ()  => bitset.forEach(()  =>  {}),  (time)  =>  console.log(`${percentage(density)} density: ${time}`),  );}
```

For the unoptimized version, we get:

```
100.0% density:     95.2 ms 75.0% density:    250.7 ms 50.0% density:    343.3 ms 25.0% density:    221.8 ms 10.0% density:    141.6 ms 1.0% density:     78.5 ms 0.1% density:     66.7 ms
```

With the `ìf (word === 0) continue;` optimization, we get:

```
100.0% density:     95.4 ms 75.0% density:    245.5 ms 50.0% density:    336.3 ms 25.0% density:    213.9 ms 10.0% density:    132.4 ms 1.0% density:     34.6 ms 0.1% density:      5.6 ms
```

Let's put this into a table and compare the performance:

|  | Unoptimized (baseline) | Skip 0s |
| Density | Runtime | Speed ^* | Runtime | Speed ^* |
| 100.0% | 95.2 ms | 1.0x | 95.4 ms | 1.0x |
| 75.0% | 250.7 ms | 1.0x | 245.5 ms | 1.0x |
| 50.0% | 343.3 ms | 1.0x | 336.3 ms | 1.0x |
| 25.0% | 221.8 ms | 1.0x | 213.9 ms | 1.0x |
| 10.0% | 141.6 ms | 1.0x | 132.4 ms | 1.0x |
| 5.0% | 114.5 ms | 1.0x | 95.9 ms | 1.2x |
| 1.0% | 78.5 ms | 1.0x | 34.6 ms | 2.3x |
| 0.1% | 66.7 ms | 1.0x | 5.6 ms | 11.9x |

* Speed compared to baseline

We observe no significant difference in performance for densities above 10%, but once we reach densities of ≤5% we start to see significant performance improvements: **>2x faster** at 1% density and **>10x faster** at 0.1% density.

### Skipping halves

We can take this method of optimization further by skipping *each half* of a word if it's all 0s. We'll create bitmasks for each half of a word:

```
export  const  WORD_FIRST_HALF_MASK  =  0x0000ffff;export  const  WORD_LATTER_HALF_MASK  =  0xffff0000;console.log(WORD_FIRST_HALF_MASK);console.log(WORD_LATTER_HALF_MASK);
```

Using them, we want to

*   only iterate over bits 1-16 if there are any set bits in the first half, and
*   only iterate over bits 17-32 if there are any set bits in the latter half.

We can determine whether to iterate over the halves like so:

```
const iterFirstHalf =  (word &  WORD_FIRST_HALF_MASK)  !==  0;const iterLatterHalf =  (word &  WORD_LATTER_HALF_MASK)  !==  0;
```

Which we use to determine the range of bits we iterate over:

```
const start = iterFirstHalf ?  0  :  WORD_LEN_HALF;const end = iterLatterHalf ?  WORD_LEN  :  WORD_LEN_HALF;for  (let i = start; i < end; i++)  {}
```

Let's see the difference this makes:

|  | Unoptimized (baseline) | Skip 0s | Skip 0s and halves |
| Density | Runtime | Speed ^* | Runtime | Speed ^* | Runtime | Speed ^* |
| 100.0% | 95.2 ms | 1.0x | 95.4 ms | 1.0x | 99.2 ms | 1.0x |
| 75.0% | 250.7 ms | 1.0x | 245.5 ms | 1.0x | 246.3 ms | 1.0x |
| 50.0% | 343.3 ms | 1.0x | 336.3 ms | 1.0x | 346.3 ms | 1.0x |
| 25.0% | 221.8 ms | 1.0x | 213.9 ms | 1.0x | 215.5 ms | 1.0x |
| 10.0% | 141.6 ms | 1.0x | 132.4 ms | 1.0x | 133.6 ms | 1.1x |
| 5.0% | 114.5 ms | 1.0x | 95.9 ms | 1.2x | 93.7 ms | 1.2x |
| 1.0% | 78.5 ms | 1.0x | 34.6 ms | 2.3x | 28.5 ms | 2.8x |
| 0.1% | 66.7 ms | 1.0x | 5.6 ms | 11.9x | 5.7 ms | 11.7x |

* Speed compared to baseline

We receive a tiny performance penalty for high-density sets, but we see a slight performance boost for sets at a sweet spot of roughly 1% density.

This optimization may or may not be worth it depending on your average set density, but it doesn't move the needle all that much.

* * *

It was at this point in my bit set journey that I discovered a different approach for iterating over bits that yields significantly better results across all set densities.

## Two's complement

Iterating over individual bits is expensive and requires an `if` statement at each iteration to check whether to invoke the callback or not. This `if` statement creates a [branch](https://johnnysswlab.com/how-branches-influence-the-performance-of-your-code-and-what-can-you-do-about-it/) that further degrades performance.

If we were able to somehow "jump" to the next set bit, we would eliminate the need to iterate over 0 bits and perform a bunch of `if` statements.

As it turns out, there's a very cheap method to find the least significant bit set to 1, which looks like so:

When I first saw this, it made no sense to me. I thought:

> *“Doesn't making a number negative just set the sign bit to 1?*
> 
> *If so, then `x & -x` just yields `x`.”*

That would be true if signed integers were represented using [sign-magnitude](https://en.wikipedia.org/wiki/Signed_number_representations#Sign%E2%80%93magnitude), where the leftmost bit is the sign bit and the rest of the bits denote the value (magnitude).

| Numbers represented using sign-magnitude |
| Positive | Negative |
| Bits | Value | Bits | Value |
| 00000000 | 0 | 10000000 | -0 |
| 00000001 | 1 | 10000001 | -1 |
| 00000010 | 2 | 10000010 | -2 |
| 00000011 | 3 | 10000011 | -3 |
| 00001001 | 9 | 10001001 | -9 |
| 01111111 | 127 | 11111111 | −127 |

But as I learned, signed integers are most commonly represented using [two's complement](https://en.wikipedia.org/wiki/Two%27s_complement).

Two's complement is different from sign-magnitude (and [one's complement](https://en.wikipedia.org/wiki/Signed_number_representations#Ones'_complement)) in that it only has one representation for 0 (there's no -0).

| Numbers represented using two's complement |
| Positive | Negative |
| Bits | Value | Bits | Value |
| 00000000 | 0 |  |  |
| 00000001 | 1 | 11111111 | -1 |
| 00000010 | 2 | 11111110 | -2 |
| 00000011 | 3 | 11111101 | -3 |
| 00001001 | 9 | 11110111 | -9 |
| 01111111 | 127 | 10000001 | −127 |
|  |  | 10000000 | −128 |

The two's complement of an integer is computed by:

1.  inverting the bits (including the sign bit), and
2.  adding 1 to the number.

```
00010011  1110110011101101 
```

Note: This also works in the opposite direction (from negative to positive)

The binary representation of `19` has a 1 as the least-significant bit. Inverting makes the least significant bit become 0, so adding one will always make the first bit 1\. This makes `x & -x` yield the 1st set bit for any number where the least-significant bit is 1.

Let's take a look at a number with some leading 0s:

```
00110000  1100111111010000 
```

Here we observe that all the bits before the least-significant set bit become 1 when inverted. When 1 is added to the number, the 1s are carried until they reach the least-significant 0 (which was the least-significant 1 pre-inversion). This makes `x & -x` yield the 1st set bit for any number with leading 0s.

So when iterating over the bits of a word, we can always find the least significant set bit via `word & -word`. What's neat is that we can then use bitwise XOR to unset the bit:

```
while  (word !==  0)  {  const lsb = word &  -word; word ^= lsb;  }
```

We can now iterate over the set bits of a word, but we've got a small problem. We want to invoke the callback with the *index of* the set bits, not the set bits themselves.

We'll find the index of the set bit through the use of [Hamming weights](https://en.wikipedia.org/wiki/Hamming_weight).

### Hamming weight

Given an integer with one set bit, we want to be able to quickly find the index of said bit:

```
indexOfFirstSetBit(0b00000100)indexOfFirstSetBit(0b00000001)indexOfFirstSetBit(0b00100000)
```

The brute-force approach would be to walk over the bits one by one, but then we're back to iterating over bits. That's a no-go.

One observation to make is that the index of the set bit is equal to the number of leading 0s.

Consider what happens when we subtract 1\. The leading 0s turn into 1s, and the set bit becomes unset.

This transforms the problem from finding the index of the set bit in `x` into computing the number of set bits in `x - 1`. The number of non-zero bits is known as the [Hamming weight](https://en.wikipedia.org/wiki/Hamming_weight), and it turns out that we can [compute the Hamming weight](https://en.wikipedia.org/wiki/Hamming_weight#Efficient_implementation) of an integer very cheaply:

```
function  hammingWeight(n:  number):  number  { n -=  (n >>  1)  &  0x55555555; n =  (n &  0x33333333)  +  ((n >>>  2)  &  0x33333333);  return  (((n +  (n >>>  4))  &  0xf0f0f0f)  *  0x1010101)  >>  24;}
```

How this works, precisely, is something that we won't get into. We'll just trust that this works.

We now have all the pieces we need.

### Making BitSet.forEach go fast

As before, we iterate over each word:

```
class  BitSet  {  forEach(callback:  (index:  number)  =>  void)  {  const words =  this.words;  for  (let wordIndex =  0; wordIndex < words.length; wordIndex++)  {  let word = words[wordIndex];  }  }}
```

While `word` is non-zero, we find the least-significant bit `lsb`:

```
while  (word !==  0)  {  const lsb = word &  -word;}
```

Using `lsb` we can compute the index using the hamming weight of `lsb - 1`:

```
const index =  (wordIndex <<  WORD_LOG)  +  hammingWeight(lsb -  1);callback(index);
```

Before the next iteration, we unset the least significant bit via `word XOR lsb`, making `word` ready for the next iteration:

The full implementation looks like so:

```
forEach(callback:  (index:  number)  =>  void)  {  const words =  this.words;  for  (let wordIndex =  0; wordIndex < words.length; wordIndex++)  {  let word = words[wordIndex];  while  (word !==  0)  {  const lsb = word &  -word;  const index =  (wordIndex <<  WORD_LOG)  +  hammingWeight(lsb -  1);  callback(index); word ^= lsb;  }  }}
```

But how fast is this optimized version? Let's run our benchmark and compare.

|  | Unoptimized (baseline) | Skip 0s | Optimized |
| Density | Runtime | Speed ^* | Runtime | Speed ^* | Runtime | Speed ^* |
| 100.0% | 95.2 ms | 1.0x | 95.4 ms | 1.0x | 53.4 ms | 1.8x |
| 75.0% | 250.7 ms | 1.0x | 245.5 ms | 1.0x | 91.9 ms | 2.7x |
| 50.0% | 343.3 ms | 1.0x | 336.3 ms | 1.0x | 68.4 ms | 5.0x |
| 25.0% | 221.8 ms | 1.0x | 213.9 ms | 1.0x | 44.4 ms | 5.0x |
| 10.0% | 141.6 ms | 1.0x | 132.4 ms | 1.0x | 30.1 ms | 4.7x |
| 5.0% | 114.5 ms | 1.0x | 95.9 ms | 1.2x | 24.8 ms | 4.6x |
| 1.0% | 78.5 ms | 1.0x | 34.6 ms | 2.3x | 10.0 ms | 7.8x |
| 0.1% | 66.7 ms | 1.0x | 5.6 ms | 11.9x | 4.0 ms | 16.7x |

* Speed compared to baseline

For densities of 5-50%, we receive a **~5x increase in performance**. Higher densities of 75% and above receive a notable speedup of >2x, while the densities below 5% see a **5-17x increase in performance**.

## A full BitSet implementation

I've recently published a performant and feature-complete `BitSet` package [on npm](https://www.npmjs.com/package/bitset-mut).

If you want to explore the full implementation, take a look at the [GitHub repo](https://github.com/alexharri/bitset-mut).

## Further reading

[Daniel Lemire](https://lemire.me/blog/) has written [lots](https://lemire.me/blog/2018/02/21/iterating-over-set-bits-quickly/) [of](https://lemire.me/blog/2012/11/13/fast-sets-of-integers/) [posts](https://lemire.me/blog/2019/05/03/really-fast-bitset-decoding-for-average-densities/) about bit sets. His [`FastBitSet` implementation](https://github.com/lemire/FastBitSet.js) is where I discovered this trick for optimizing over set bits. If you're into software performance, he's written *a lot* on that topic.

## Parting thoughts

Any given piece of code can be optimized, but taking a different approach will often outperform those local optimizations.

Different algorithms will often favor some inputs over others, as we saw with low vs high-density sets in this post. It's useful to keep these sorts of trade-offs in mind when considering which way to go. Benchmark when possible!

* * *

Anyway, thanks for reading this short series on bit set! If you haven't read part 1 yet, you can find it here: [Bit Sets: An introduction to bit manipulation](/blog/bit-sets).

I may write a part 3, taking an in-depth look at bit set performance for boolean operations (and, or, xor, andNot, etc), but I've thought about bit sets quite enough for now.

— Alex Harri

</main>