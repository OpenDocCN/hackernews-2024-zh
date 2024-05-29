<!--yml
category: 未分类
date: 2024-05-27 14:32:31
-->

# SIMD in Pure Python | Blog

> 来源：[https://www.da.vidbuchanan.co.uk/blog/python-swar.html](https://www.da.vidbuchanan.co.uk/blog/python-swar.html)

# SIMD in Pure Python

*By David Buchanan, 4^(th) January 2024*

First of all, this article is an exercise in recreational "because I can" programming. If you just want to make your Python code go fast, this is perhaps not the article for you. And perhaps Python is not the language you want, either!

By the end, I'll explain how I implemented [Game of Life](https://en.wikipedia.org/wiki/Conway%27s_Game_of_Life) in pure Python (plus pysdl2 for graphics output) running in 4K resolution at 180fps, which represents a ~3800x speedup over a naive implementation.

![](img/7b57f311eb4ea0c2294d0c82ecd87168.png)

If you're already familiar with SIMD and vectorization, you might want to skip this next section.

### A Brief Introduction to SIMD

If you want to get the most performance out of a modern CPU, you're probably going to be using [SIMD](https://en.wikipedia.org/wiki/Single_instruction,_multiple_data) instructions - Single Instruction, Multiple Data.

For example, if you have two arrays of length 4, containing 32-bit integers, most modern CPUs will let you add the pairwise elements together in a single machine instruction (assuming you've already loaded the data into registers). Hopefully it's obvious why this is more efficient than looping over the individual array elements. Intel CPUs have had this *specific* capability since [SSE2](https://en.wikipedia.org/wiki/SSE2) was introduced in the year 2000, but SIMD as a concept predates it by [a lot](https://en.wikipedia.org/wiki/Single_instruction,_multiple_data#History).

Newer CPUs cores have been expanding on these capabilities ever since, meaning SIMD instructions are more relevant than ever for maximising CPU throughput.

If you're programming in a language like C, an optimising compiler will recognise code that can be accelerated using SIMD instructions, and automatically emit appropriate machine code. Compilers can't optimise everything perfectly, though, so anyone who wants to squeeze out the maximum performance might end up using [Intrinsics](https://www.intel.com/content/www/us/en/docs/intrinsics-guide/index.html) to explicitly tell the compiler which instructions to use. And if *that* still isn't enough, you might end up programming directly in assembly.

There are many ways to express SIMD programs, but the rest of them are out of scope for this article!

"Vectorization" is the process of transforming a typical program into one that operates over whole arrays of data (i.e. vectors) at once (for example, using SIMD). The work done by the optimising compiler described above, or the human writing Intrinsic operations, is vectorization.

### A Brief Introduction to CPython

[CPython](https://github.com/python/cpython) is the reference implementation of the Python language, written mostly in C, hence the name. Other implementations exist, but when people say "Python" they're often implicitly referring to CPython. I'll try to only say Python when I'm referring to the language as a whole, and CPython when I'm talking about a CPython implementation detail (which we'll be getting into later).

The TL;DR is that CPython compiles your code into a [bytecode](https://docs.python.org/3/glossary.html#term-bytecode) format, and then interprets that bytecode at run-time. I'll be referring to that bytecode interpreter as the ["VM"](https://docs.python.org/3/glossary.html#term-virtual-machine).

### SIMD in Python

Python does not natively have a concept of SIMD. However, libraries like [NumPy](https://numpy.org/) exist, allowing for relatively efficient vectorized code. NumPy lets you define vectors, or even n-dimensional arrays, and perform operations on them in a single API call.

|  |  
```
import numpy as np

a = np.array([1, 2, 3, 4])
b = np.array([2, 4, 6, 8])

print(a + b)  # [ 3  6  9 12] 
```

 |

Without NumPy, the above example would require a loop over array elements (or a list comprehension, which is fancy syntax for the same thing, more or less).

Internally, NumPy is implemented using native C extensions, which in turn use Intrinsics to express SIMD operations. I'm not an expert on NumPy implementation details, but you can peruse their SIMD code [here](https://github.com/numpy/numpy/tree/main/numpy/_core/src/common/simd). Note that the code has been customised for various CPU architectures.

CPython itself, being an interpreted Python implementation, is slow. But if you can structure your program so that all the "real work" gets done inside a library like NumPy, it can be surprisingly efficient overall.

NumPy is excellent and widely used for getting real work done. However, NumPy is not "pure" Python!

### SIMD in *Pure* Python

By "pure," I mean using only functionality built into the [Python language](https://docs.python.org/3/reference/) itself, or the [Python standard library](https://docs.python.org/3/library/index.html).

This is an entirely arbitrary and self-imposed constraint, but I think it's a fun one to work within. It's also vaguely useful, since libraries like NumPy aren't available in certain environments.

Earlier, I said Python doesn't natively have a concept of SIMD. This isn't entirely true; otherwise the article would end here. Python supports bitwise operations over pairs of integers: AND (`&`), OR (`|`), XOR (`^`). If you think about these as operations over vectors of booleans, each bit being one bool, it is SIMD!

Unlike many other programming languages, Python integers have unlimited precision. That is, they can accurately represent integers containing arbitrarily many digits—at least, until you run out of memory. This means we can evaluate an unlimited number of conceptually-parallel boolean operations with a single python operator.

SIMD over booleans might sound esoteric, but it's an idea we can immediately put to work. A common operation in cryptography is to XOR two byte buffers together. An idiomatic implementation might look like this:

|  |  
```
def xor_bytes(a: bytes, b: bytes) -> bytes:
	assert(len(a) == len(b))
	return bytes(x ^ y for x, y in zip(a, b)) 
```

 |

This takes each individual byte (as an integer between 0 and 255) in a and b, applies the xor operator to each, and constructs a new `bytes` object from the results. Arguably, we are already using the boolean-SIMD concept here; each of the 8 bits in a byte are getting XORed in parallel. But we can do better than that:

|  |  
```
def xor_bytes_simd(a: bytes, b: bytes) -> bytes:
	assert(len(a) == len(b))
	return (
		int.from_bytes(a, "little") ^ int.from_bytes(b, "little")
	).to_bytes(len(a), "little") 
```

 |

This might look a bit ridiculous, and that's because it is. We convert each `bytes` object into an arbitrary-precision integer, XOR those two integers together, and then convert the resulting integer back to `bytes`. The python `^` operator only gets executed once, processing all the data in one step (or more explicitly, one CPython VM operation). I'm going to call this approach "pseudo-SIMD". But with all this conversion between bytes and integers, surely it must be slower overall? Let's benchmark it.

Here are my results. The number is time to execute 1 million iterations. I tested on CPython 3.11 on an M1 Pro macbook. [Try it on your own machine!](https://gist.github.com/DavidBuchanan314/51bb8f6219ea8bb7a603e0ad19725f6d)

```
naive, n=1 0.3557752799242735
simd,  n=1 0.21655898913741112
numpy, n=1 0.798536550020799

naive, n=16 0.8749550790525973
simd,  n=16 0.23561427788808942
numpy, n=16 0.7937424059491605

naive, n=128 4.5441425608005375
simd,  n=128 0.5077524171210825
numpy, n=128 0.8012108108960092

naive, n=1024 34.96425646613352
simd,  n=1024 2.811028849100694
numpy, n=1024 0.9388492209836841

```

Even for the trivial case of n=1, somehow our wacky pseudo-SIMD function wins, by about a third!

The difference becomes even more pronounced as the buffer size increases, with the pseudo-SIMD function being 12x faster for n=1024.

I also threw a numpy implementation into the mix. This isn't really fair on numpy, because converting the bytes to and from numpy arrays appears to have quite a high constant overhead. In more realistic numpy code, you'd end up keeping your data in numpy format throughout. Because of this, numpy ended up slower all the way until n=1024, where it became 3x faster. Evidently, those constant overheads become less relevant as <math xmlns="http://www.w3.org/1998/Math/MathML" display="inline"><mrow><mi>n</mi></mrow></math> grows.

But what if we eliminate the conversion overheads, allowing our functions to input and output data in their "native" formats, rather than bytes? For pseudo-SIMD, that format is integers, and for numpy it's a `np.array` object.

```
simd,  n=1 0.03484031208790839
numpy, n=1 0.2326297229155898

simd,  n=16 0.042713511968031526
numpy, n=16 0.23679199093021452

simd,  n=128 0.046673570992425084
numpy, n=128 0.23861141502857208

simd,  n=1024 0.08742194902151823
numpy, n=1024 0.28949279501102865

simd,  n=32768 0.9535991169977933
numpy, n=32768 0.9617231499869376

simd,  n=131072 4.984845655970275
numpy, n=131072 4.609246583888307

```

Pseudo-SIMD has the edge for small inputs (perhaps because it doesn't have to do any [FFI](https://en.wikipedia.org/wiki/Foreign_function_interface)), but for large buffers, numpy edges ahead. But only barely! How is our pure-python XOR function (which at this point is just the XOR operator itself) able to keep up with NumPy's optimised SIMD code?

### CPython Internals

Let's take a closer look. Here's the [inner loop](https://github.com/python/cpython/blob/v3.11.5/Objects/longobject.c#L5050-L5051) of the code for XORing together two arbitrary-precision integers in CPython 3.11.

|  |  
```
for  (i  =  0;  i  <  size_b;  ++i)
  z->ob_digit[i]  =  a->ob_digit[i]  ^  b->ob_digit[i]; 
```

 |

It's a simple loop over arrays. No SIMD instructions? Well, there aren't any explicit ones, but what does the C compiler do to it? Let's take an *even closer* look.

I loaded libpython into Ghidra and had a look around. The library on my system didn't have full symbols, so I searched for cross-references to the exported symbol `_PyLong_New`. There were 82 matches, but by an extremely weird stroke of luck it was the first function I clicked on.

On my system, the (aarch64) assembly corresponding to the above loop is as follows:

```
.      00299ec0 01 03 80 d2     mov        x1,#0x18
       00299ec4 00 00 80 d2     mov        x0,#0x0

  ,->LAB_00299ec8                                    XREF[1]:     00299ee4(j)  
  |    00299ec8 80 6a e1 3c     ldr        q0,[x20,x1]
  |    00299ecc 00 04 00 91     add        x0,x0,#0x1
  |    00299ed0 a1 6a e1 3c     ldr        q1,[x21,x1]
  |    00299ed4 00 1c 21 6e     eor        v0.16B,v0.16B,v1.16B
  |    00299ed8 e0 6a a1 3c     str        q0,[x23,x1]
  |    00299edc 21 40 00 91     add        x1,x1,#0x10
  |    00299ee0 5f 00 00 eb     cmp        x2,x0
   \_  00299ee4 21 ff ff 54     b.ne       LAB_00299ec8
       00299ee8 61 f6 7e 92     and        x1,x19,#-0x4
       00299eec 7f 06 40 f2     tst        x19,#0x3
       00299ef0 e0 02 00 54     b.eq       LAB_00299f4c

     LAB_00299ef4                                    XREF[1]:     0029a2a0(j)  
       00299ef4 20 f4 7e d3     lsl        x0,x1,#0x2
       00299ef8 22 04 00 91     add        x2,x1,#0x1
       00299efc 85 02 00 8b     add        x5,x20,x0
       00299f00 a4 02 00 8b     add        x4,x21,x0
       00299f04 e0 02 00 8b     add        x0,x23,x0
       00299f08 86 18 40 b9     ldr        w6,[x4, #0x18]
       00299f0c a3 18 40 b9     ldr        w3,[x5, #0x18]
       00299f10 63 00 06 4a     eor        w3,w3,w6
       00299f14 03 18 00 b9     str        w3,[x0, #0x18]
       00299f18 7f 02 02 eb     cmp        x19,x2
       00299f1c 8d 01 00 54     b.le       LAB_00299f4c
       00299f20 83 1c 40 b9     ldr        w3,[x4, #0x1c]
       00299f24 21 08 00 91     add        x1,x1,#0x2
       00299f28 a2 1c 40 b9     ldr        w2,[x5, #0x1c]
       00299f2c 42 00 03 4a     eor        w2,w2,w3
       00299f30 02 1c 00 b9     str        w2,[x0, #0x1c]
       00299f34 7f 02 01 eb     cmp        x19,x1
       00299f38 ad 00 00 54     b.le       LAB_00299f4c
       00299f3c 82 20 40 b9     ldr        w2,[x4, #0x20]
       00299f40 a1 20 40 b9     ldr        w1,[x5, #0x20]
       00299f44 21 00 02 4a     eor        w1,w1,w2
       00299f48 01 20 00 b9     str        w1,[x0, #0x20]
     LAB_00299f4c

```

If you're not a reverse engineer, or even if you are, you're probably thinking "WTF is going on here?" This is a fairly typical result of compiler [auto-vectorization](https://en.wikipedia.org/wiki/Automatic_vectorization). Ghidra does a terrible job of converting it to meaningful pseudocode, so I'll provide my own version (note, this is not a 1:1 mapping, but it should convey the general idea)

|  
```
 1
 2
 3
 4
 5
 6
 7
 8
 9
10
11
12
13
14
15
```

 |  
```
i = 0
words_left_to_xor = size_b
while words_left_to_xor > 3:
	# xor 4 words (16 bytes) concurrently using q0, q1 registers
	z[i:i+4] = a[i:i+4] ^ b[i:i+4]
	i += 4
	words_left_to_xor -= 4

# deal with remaining 32-bit words individually
if words_left_to_xor > 0:
	z[i] = a[i] ^ b[i]
if words_left_to_xor > 1:
	z[i+1] = a[i+1] ^ b[i+1]
if words_left_to_xor > 2:
	z[i+2] = a[i+2] ^ b[i+2] 
```

 |

The main loop operates using the `q0` and `q1` registers, which according to [ARM docs](https://developer.arm.com/documentation/dht0002/a/Introducing-NEON/NEON-architecture-overview/NEON-registers) are 128-bit wide NEON registers. As far as I can tell, NEON doesn't stand for anything in particular, but it's what ARM calls its SIMD features (by the way, their "next-gen" SIMD instruction set is called [SVE](https://developer.arm.com/documentation/102476/0100/Introducing-SVE)).

After the main loop, it xors the remaining 32-bit words, one at a time.

The key observation here is that our "pseudo-SIMD" implementation is using real SIMD instructions under the hood! At least, it is on my system; this may depend on your platform, compiler, configuration, etc.

If we limit ourselves to bitwise operators, and our numbers are big enough that the interpreter overhead is small (in relative terms), we can get real SIMD performance speedups in pure Python code. Well, kinda. If you were implementing a specific algorithm in assembly, you could generate much more tightly optimised SIMD routines, keeping data in registers where possible, avoiding unnecessary loads and stores from memory, etc.

Another big caveat with this approach is that it involves creating a whole new integer to contain the result. This wastes memory space, puts pressure on the memory allocator/gc, and perhaps most importantly, it wastes memory bandwidth. Although you can write `a ^= b` in Python to denote an in-place XOR operation, it still ends up internally allocating a new object to store the result.

If you're wondering why the NumPy implementation was still slightly faster, I believe the answer lies in the way CPython represents its integers. Each entry in the `ob_digit` array only represents 30 bits of the overall number. I'm guessing this makes handling carry propagation simpler during arithmetic operations. This means the in-memory representation has a ~7% overhead compared to optimal packing. While I haven't checked NumPy's implementation details, I imagine they pack array elements tightly.

### Doing Useful Work

Now that we know we can do efficient-ish bitwise SIMD operations, can we build something useful from that?

One use case is bitsliced cryptography. [Here's](https://github.com/DavidBuchanan314/python-bitsliced-aes) my implementation of bitsliced AES-128-ECB in pure Python. It's over 20x faster than the next fastest pure-python AES implementation I could find, and in theory it's more secure too, due to not having any data-dependent array indexing (but I still wouldn't trust it as a secure implementation; use a [proper cryptography library!](https://cryptography.io/en/latest/))

For a more detailed introduction to bitslicing, check out [this article](https://timtaubert.de/blog/2018/08/bitslicing-an-introduction/). The idea is to express your whole algorithm as a circuit of logic gates, or in other words, a bunch of boolean expressions. You can do this for any computation, but AES is particularly amenable to it. Once you have a boolean expression, you can use bit-parallel operations (i.e., bitwise SIMD operations) to compute multiple instances of your algorithm in parallel. Since AES is a block-based cipher, you can use this idea to compute multiple AES cipher blocks concurrently.

### SWAR

We can use Python integers for more than just parallel bitwise operations. We can use them for parallel additions, too!

|  
```
 1
 2
 3
 4
 5
 6
 7
 8
 9
10
11
12
13
14
15
```

 |  
```
a = 0x4567 # this represents four 4-bit unsigned integers, [4, 5, 6, 7]
b = 0x6789 # as does this: [6, 7, 8, 9]

print(hex(a + b))  # 0xacf0 => [0xa, 0xc, 0xf, 0x0] == [10, 12, 15, 0]

# oh no, that's the wrong answer... 6+8 should be 14, not 15.
# it's wrong because the result of 9+7 was 16 (0x10), causing carry propagation
# into the adjacent "lane".

# solution: padding and masking:
a = 0x04050607
b = 0x06070809
m = 0x0f0f0f0f

print(hex((a + b) & m)) # 0xa0c0e00 => [0xa, 0xc, 0xe, 0x0] == [10, 12, 14, 0] 
```

 |

As shown here, we can pack multiple fixed-width integers into a single Python integer, and add them all together at once. However, if they're tightly packed and an integer overflow occurs, this causes unwanted carry propagation between lanes. The solution is simple: space them out a bit. In this example I use generous 4-bit-wide padding to make things more obvious, but in principle you only need a single bit of padding. Finally, we use the bitwise AND operator to mask off any overflow bits. If we didn't do this, the overflowing bits could accumulate over the course of multiple additions and start causing overflows between lanes again. The more padding bits you use between lanes, the more chained additions you can survive before masking is required.

You can do similar things for subtraction, multiplication by a small constant, and bit shifts/rotations, so long as you have enough padding bits to prevent overflows in each scenario.

The general term for this concept is SWAR, which stands for [SIMD Within A Register](https://en.wikipedia.org/wiki/SWAR). But here, rather than using a machine register, we're using an arbitrarily long Python integer. I'm calling this variant SWAB: SIMD Within A Bigint.

SWAB is a useful idea in Python because it maximises the amount of work done per VM instruction, reducing the interpreter overhead; the CPU gets to spend the majority of its time in the fast native code that implements the integer operations.

### Doing ~~Useful Work~~ Something Fun

That's enough theory; now it's time to make Game of Life go fast. First, let me describe the "problem" I'm trying to solve. I'll assume you're already broadly familiar with Game of Life, but if not, go read the [Wikipedia article.](https://en.wikipedia.org/wiki/Conway%27s_Game_of_Life)

There are some very clever algorithms for making GoL go fast, the most famous being [Hashlife](https://johnhw.github.io/hashlife/index.md.html), which works by spotting repeating patterns in both space and time. However, my favourite GoL pattern to simulate is ["soup"](https://conwaylife.com/wiki/Soup), i.e., a large random starting grid. These chaotic patterns aren't well suited to the Hashlife algorithm, so we need to go back to the basics.

When you simulate soup in the classic GoL ruleset, it typically dies out after a few thousand generations, producing a rather boring arrangement of oscillating ["ash"](https://conwaylife.com/wiki/Soup#Ash) (which is back to something Hashlife can simulate quickly). I prefer my simulations to live on in eternal chaos, and there's a variant of the classic ruleset that more or less guarantees this, called [DryLife](https://conwaylife.com/wiki/OCA:DryLife). I like DryLife because it still exhibits most of the familiar GoL behaviours (for example, gliders) and yet the soup lives on indefinitely, creating a pleasing screensaver-like animation.

The "obvious" implementation of the inner loop of the GoL algorithm looks something like this:

|  
```
 1
 2
 3
 4
 5
 6
 7
 8
 9
10
11
12
13
14
15
```

 |  
```
for y in range(1, height + 1):
    for x in range(width):
        neighbor_count = sum(
            get_cell(state, (x + dx) % width, y + dy)
            for dx, dy in [
                (-1, -1), (0, -1), (1, -1),
                (-1,  0),          (1,  0),
                (-1,  1), (0,  1), (1,  1)
            ]
        )
        this_cell = get_cell(state, x, y)
        next_value = neighbor_count == 3 or (this_cell and neighbor_count == 2)
        if cfg.drylife: # another opportunity for dead cells to come alive
            next_value &#124;= (not this_cell) and neighbor_count == 7
        set_cell(next_state, x, y, next_value) 
```

 |

It iterates over every cell in the grid, counts up its immediate 8 neighbours, and applies the rules to decide if the cell should be alive or not in the next iteration. In big-O terms, this is an <math xmlns="http://www.w3.org/1998/Math/MathML" display="inline"><mrow><mi>O</mi><mo stretchy="false">(</mo><mi>n</mi><mo stretchy="false">)</mo></mrow></math> algorithm, where <math xmlns="http://www.w3.org/1998/Math/MathML" display="inline"><mrow><mi>n</mi></mrow></math> is the number of cells in the grid (i.e., width <math xmlns="http://www.w3.org/1998/Math/MathML" display="inline"><mrow><mi>×</mi></mrow></math> height).

Although it isn't made explicit in the snippet above, the state of the cells is being stored in a big array, accessed via the `get_cell` and `set_cell` helper functions. What if instead of using an array, we stored the whole state in one very long integer, and used SWAB arithmetic to process the whole thing at once? The trickiest part of this process will be counting up the neighbours, which can sum to up to 8 (or 9 if we also count the initial cell value). That's a 4-bit value, and we can be sure it will never overflow into a 5-bit value, so we can store each cell as a 4-bit wide "SWAB lane". Without further ado, here's the equivalent inner-loop code:

|  
```
 1
 2
 3
 4
 5
 6
 7
 8
 9
10
11
12
13
14
15
16
17
18
19
20
21
22
23
24
25
26
```

 |  
```
# count neighbors
summed = state
summed += (summed >> 4) + (summed << 4)
summed += (summed >> COLSHIFT) + (summed << COLSHIFT)

# check if there are exactly 3 neighbors
has_3_neighbors = summed ^ MASK_NOT_3 # at this point, a value of all 1s means it was initially 3
has_3_neighbors &= has_3_neighbors >> 2 # fold in half
has_3_neighbors &= has_3_neighbors >> 1 # fold in half again

# check if there are exactly 4 neighbors
has_4_neighbors = summed ^ MASK_NOT_4 # at this point, a value of all 1s means it was initially 4
has_4_neighbors &= has_4_neighbors >> 2  # fold in half
has_4_neighbors &= has_4_neighbors >> 1  # fold in half again

if cfg.drylife:
    # check if there are exactly 7 neighbors
    has_7_neighbors = summed ^ MASK_NOT_7 # at this point, a value of all 1s means it was initially 7
    has_7_neighbors &= has_7_neighbors >> 2  # fold in half
    has_7_neighbors &= has_7_neighbors >> 1  # fold in half again

    # variable name here is misleading...
    has_3_neighbors &#124;= (~state) & has_7_neighbors

# apply game-of-life rules
state = (has_3_neighbors &#124; (state & has_4_neighbors)) & MASK_CANVAS 
```

 |

(I've omitted some details like wraparound handling; you can see the full code, along with definitions of those magic constants, [here](https://github.com/DavidBuchanan314/pyswargol/blob/main/swargol.py))

Aside from the different state representation, this achieves exactly the same thing as the previous snippet. The <math xmlns="http://www.w3.org/1998/Math/MathML" display="inline"><mrow><mi>O</mi><mo stretchy="false">(</mo><mi>n</mi><mo stretchy="false">)</mo></mrow></math> loop over every cell has been completely eliminated! Well, almost. There are <math xmlns="http://www.w3.org/1998/Math/MathML" display="inline"><mrow><mi>O</mi><mo stretchy="false">(</mo><mn>1</mn><mo stretchy="false">)</mo></mrow></math> CPython VM operations, but there is still <math xmlns="http://www.w3.org/1998/Math/MathML" display="inline"><mrow><mi>O</mi><mo stretchy="false">(</mo><mi>n</mi><mo stretchy="false">)</mo></mrow></math> work being done, hidden away inside the SIMD-accelerated native code of CPython's bigint arithmetic routines. This is a *huge* performance win, which I'll quantify later. But first, how do we turn that `state` integer into pixels on the screen?

### Blitting

To get pixels on the screen, we need to convert the data into a more standard format. The very first step is easy: Python integers have a `to_bytes` method that serialises them into bytes (like I used in the XOR function example earlier in this article). What to do with those bytes next is less obvious.

In the spirit of only using "pure" Python, I came up with a ridiculous hack: craft a compressed gzip stream that, when decompressed, converts the weird 4-bits-per-pixel buffer into a more standard 8-bits-per-pixel grayscale buffer, and surrounds it in the necessary framing data to be a YUV4MPEG video stream. The output of the Python script can be piped into a gzip decompressor, and subsequently, a video player. That code is [here](https://gist.github.com/DavidBuchanan314/acae2aab38953759aacc114b417ed0b9), and probably deserves an article of its own, but I'm not going to go into it today.

While this was a great hack, gzip is not an especially efficient [blitter](https://en.wikipedia.org/wiki/Blitter). I was able to get about 24fps at full-screen resolution on my 2021 macbook, but I really wanted at least 60fps, and that approach wasn't going to cut it.

The *ideal* approach would probably be to ship the 4bpp data off to the GPU as a texture, as-is, and write a [fragment shader](https://www.khronos.org/opengl/wiki/Fragment_Shader) capable of unpacking it onto the screen pixels. The only reason I didn't do this is because it feels silly. It feels silly because if we're doing GPU programming, we might as well just implement the whole GoL algorithm on the GPU. It'd be way faster, but it wouldn't be within the spirit of the completely arbitrary constraints I'd set for myself.

My compromise here was to use SDL2, via the `pysdl2` bindings. Sure, it's not "pure Python," but it *is* very standard and widely available. It feels "right" because I can use it to its full extent without completely defeating the purpose (like running GoL entirely on the GPU would do).

SDL2 supports a pixel format called `SDL_PIXELFORMAT_INDEX4LSB`, which is a 4-bit palleted mode. If we set up an appropriate palette (specifying the colours for "dead" and "alive" cells, respectively), then we can pass our 4bpp buffer to SDL as-is, and it'll know how to convert it into the right format for sending to the GPU (in this case, `SDL_PIXELFORMAT_ARGB8888`). This process isn't terribly efficient, because the conversion still happens on the CPU, and the amount of data sent over to the GPU is much larger than necessary. Despite that, it's a whole lot faster than the gzip method, getting around 48fps.

### Parallelization

We still haven't hit the 60fps target, and to get there I added some parallelization. I summarised my approach in a code comment:

|  
```
 1
 2
 3
 4
 5
 6
 7
 8
 9
10
11
12
13
14
15
16
17
18
19
20
21
22
23
24
25
26
27
28
29
30
31
32
33
34
35
36
37
38
39
```

 |  
```
"""

┌───────────┐             Graphics Process
│ ┌────┐    │    ┌───────────────────────────────┐
│ │  ┌─▼────┴─┐  │  ┌─────────┐  ┌────────────┐  │
│ │  │  Life  ├─────► Blitter ├──►            │  │
│ │  └─┬────▲─┘  │  └─────────┘  │            │  │
▼ │  ┌─▼────┴─┐  │  ┌─────────┐  │            │  │
│ ▲  │  Life  ├─────► Blitter ├──►            │  │
│ │  └─┬────▲─┘  │  └─────────┘  │    GUI     │  │
│ │  ┌─▼────┴─┐  │  ┌─────────┐  │  Renderer  │  │
│ │  │  Life  ├─────► Blitter ├──►            │  │
│ │  └─┬────▲─┘  │  └─────────┘  │            │  │
▼ │  ┌─▼────┴─┐  │  ┌─────────┐  │            │  │
│ ▲  │  Life  ├─────► Blitter ├──►            │  │
│ │  └─┬────▲─┘  │  └─────────┘  └────────────┘  │
│ └────┘    │    └───────────────────────────────┘
└───────────┘

"Life" threads implement the SWAR life algorithm, for a horizontal strip of
the overall canvas.

Blitter threads use SDL2 functions to unpack the SWAR buffers into RGBA8888
surfaces, which are passed to the main Renderer thread.

The renderer thread is responsible for uploading the surfaces to a GPU texture,
and making it show up on the screen. It's also responsible for dealing with SDL
events.

Each Life thread lives in its own process, to avoid the GIL. They
talk to each other (for overlap/wraparound), and to the Blitter threads (to
report their results), using Pipes.

Everything else happens in the main process, so the Blitters can talk to the
main thread using standard Queues - the hard work here is done inside SDL2,
which does not hold the GIL (meaning we can use multithreading, as opposed
to multiprocessing).

""" 
```

 |

That's enough to smash past the 60fps target, reaching ~250fps at my full-screen resolution of 3024x1890, using 8 parallel Life processes. At 4K resolution (3840x2160), it can reach 180fps.

There are plenty of remaining inefficiencies here that could be improved on, but 250fps is already *way* faster than I care about, so I'm not going to optimise any further. I think 60fps is the most visually interesting speed to watch at, anyway (which can be achieved by turning on vsync).

> **Edit:** After having some people test on non-Apple-silicon systems, many are struggling to hit 4K60fps. I haven't done any profiling, but my guess is that the bottleneck is the CPU->GPU bandwidth, or maybe just memory bandwidth in general. I might revisit this in the future, perhaps implementing the buffer-unpacking fragment shader idea I mentioned.

Earlier, I said that upgrading from the naive implementation to SWAB gave "huge" performance wins, so just how huge are they? If I go back to the naive approach, I get an incredible 0.4fps at 4K resolution (and that's *with* 8x parallelism), representing a ~450x performance difference. If I get extra mean and force it to run on a single thread, it's a 3800x difference relative to the fully optimised version. Damn!

As a reminder, both approaches are <math xmlns="http://www.w3.org/1998/Math/MathML" display="inline"><mrow><mi>O</mi><mo stretchy="false">(</mo><mi>n</mi><mo stretchy="false">)</mo></mrow></math>, but the faster version gets to spend more time within efficient native code.

If I rewrote this whole thing in C using SIMD intrinsics (combined with SWAR or other tricks), I predict that I'd get somewhere between 10x and 100x further speedup, due to more efficient use of SIMD registers and memory accesses. A GPU implementation could be faster still. Those speedups would be nice to have, but I think it's interesting how far I was able to get just by optimising the Python version.

The source code is available [here](https://github.com/DavidBuchanan314/pyswargol).