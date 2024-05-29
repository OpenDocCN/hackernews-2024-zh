<!--yml
category: 未分类
date: 2024-05-27 15:14:03
-->

# Vectorizing Unicode conversions on real RISC-V hardware

> 来源：[https://camel-cdr.github.io/rvv-bench-results/articles/vector-utf.html](https://camel-cdr.github.io/rvv-bench-results/articles/vector-utf.html)

# Vectorizing Unicode conversions on real RISC-V hardware

In this article we'll discuss how to achieve the speedup below for UTF-8 to UTF-16 conversion, using the RISC-V Vector extension.

I excluded the plain ASCII case from the graph above because it made the other results less readable, the speedup was: 8x for C908, and 11x for C920\. More comprehensive measurements are at the [end](#finbench) of the article.

The vast majority of text you'll come across will be encoded in the UTF-8 format, but some languages and APIs use UTF-16 as their native format instead (JavaScript, Java, Windows, ...). This and other reasons, might cause you to convert between different Unicode encodings. As demonstrated by [simdutf](https://github.com/simdutf/simdutf/), this conversion process has a lot of optimization potential.

Here we'll focus on UTF-8 to UTF-16 conversion, and aim to develop an optimized RISC-V implementation that can be upstreamed to the [simdutf](https://github.com/simdutf/simdutf/) library, which is used among others by Node.js and Bun. ([The changes](https://github.com/simdutf/simdutf/pull/373) are now upstream)

The RISC-V Vector extension (RVV) adds a set of 32 vector registers that are each VLEN bits wide, where VLEN is a power-of-two greater or equal to 128 (in the standard V extension). Vector registers can be interpreted as multiple 8/16/32/64 bit elements, and operated on accordingly, as signed/unsigned integers or single/double precision floating point numbers. Since we can operate on multiple elements at a time large speedup over scalar code is possible.

At the time of writing (early 2024) there is almost no hardware available that supports RVV.

*   The only hardware with RVV support, regular consumers can buy, is the [Kendryte K230](https://www.canaan.io/product/k230). It has a [C908](https://www.t-head.cn/product/%E7%8E%84%E9%93%81C908) in-order core from Xuantie running at 1.6GHz with a VLEN of 128 bits. ([C908 benchmark page](../canmv_k230/index.html))
*   You can also buy two other CPUs that support an early incompatible draft version (0.7.1) of the vector extension, the [C906](https://www.t-head.cn/product/C906?lang=en) and the [C920](https://www.t-head.cn/product/C910?lang=en) (C910 with RVV). Since the C906s performance characteristics are very similar to the C908, we won't include benchmarks for this one. ([C906 benchmark page](../mangopi_mq_pro/index.html))
*   The [C920](https://www.t-head.cn/product/c910?lang=en) however is a much faster out-of-order core at 2GHz, the results of which are more interesting than the ones from C908 for future hardware. Targeting RVV 0.7.1 is a pain, as there is no official toolchain support, but I've taken the time to manually translate the generated assembly and assemble it with an [older GCC branch](https://github.com/brucehoult/riscv-gnu-toolchain). ([C920 benchmark page](../milkv_pioneer/index.html))
*   There are [a few open-source RVV implementations](https://github.com/stars/camel-cdr/lists/rvv-implementations), but most are still in development/incomplete. The only one that is complete, and we could simulate locally is Tenstorrents bobcat (formally ocelot), but it was a proof-of-concept design and some instructions we'll be using were explicitly not optimized. (we'll discuss that [later](#gather))

Fortunately, I own the Kendryte K230, and have ssh access to a Milk-V Pioneer server with 64 C920 cores, thanks to [perfXlab](http://www.perfxlab.com/). The development was done via qemu emulation, as that's far simpler than using real hardware, for now.

The next two sections will cover the basics of RVV and Unicode, feel free to [skip ahead](#attack) if you are already familiar with the topics.

I'll try to explain the RVV basics using a short example:

```
size_t utf8_count_rvv(char const *buf, size_t len) {
 size_t sum = **0**;
 for (size_t vl; len > **0**; len -= vl, buf += vl) {
 vl = __riscv_vsetvl_e8m8(len);
 vuint8m8_t v = __riscv_vle8_v_u8m8((uint8_t*)buf, vl);
 vbool1_t b = __riscv_vmsne(__riscv_vsrl(v, **6**, vl), **0b10**, vl); 
 sum += __riscv_vcpop(b, vl);
 }
 return sum;
}

```

```
*// the corresponding assembly code*:
**utf8_count_rvv**:
 li      a2, **0**
**loop**:
 vsetvli a3, a1, e8, m8, ta, ma
 vle8.v  v8, (a0)
 vsrl.vi v8, v8, **6**
 vmsne.vi        v16, v8, **0b10**
 vcpop.m a4, v16
 add     a2, a2, a4
 sub     a1, a1, a3
 add     a0, a0, a3
 bnez    a1, loop
 mv      a0, a2
 ret

```

Here we're using the C intrinsics API to count the number of UTF-8 characters* in the supplied data. The next section will describe this in more detail, but to count the number of UTF-8 characters we just need to count the number of bytes that don't match the pattern `**0b10xxxxxx**`, assuming the input is valid.

**From here on out when I refer to a "character" I mean a Unicode code point. This doesn't directly map into a single character on screen, for example, this emoji "🧙‍♀️" is built with three Unicode code points: 🧙 + Zero Width Joiner + ♀️ = 🧙‍♀️*

As mentioned above RVV supports different vector lengths. To facilitate having the same code work on machines with different vector lengths RVV has the `vsetvl*` instruction. You give it the element count of your input, an element width, and it will give you a count smaller or equal to your supplied count that it can fit into a vector register.
The code above uses this to iterate over the input, `vl = vsetvl_e8m8(len)` represents the of number elements one iteration processes. Next `vle8_v_u8m8()` loads `vl` 8-bit integer elements from our input into a vector register.
Then a mask is created where each active element (element where the coresponding bit in the mask is set) doesn't match the pattern `**0b10xxxxxx**`. vsrl stands for *shift right logical*, and vmsne for *not equal to*, so `vmsne(vsrl(v, **6**, vl), **0b10**, vl)` does `(x >> **6**) != **0b10**` on each element. RVV doesn't have separate mask registers, instead, masks are stored in the lower bits of a vector register, the intrinsics API adds the `vboolN_t` types to give this more type safety. Finally, we use `vcpop` to count the number of active elements in our mask and add that to our sum.

You might be wondering what the "m8" means, I've omitted that so far. RVV has 32 VLEN bits wide vector registers, but with `vsetvl` you can also configure the LMUL (length multiplier), and cause the processor to group these registers. This means that subsequent instructions will act on a register group, and `vsetvl` will return a `vl` corresponding to LMUL.
When LMUL=1 we've got 32 VLEN bits wide registers, for LMUL=2 they now act like 16 VLEN*2 bit wide registers, so for "m8" (LMUL=8), we've got 4 VLEN*8 bit wide registers. Here we use less than five vector registers, so using LMUL=8 gives us essentially free loop unrolling, which makes the scalar- and mask operations less expensive. Unrolling isn't the only advantage of LMUL, it also allows us to easily work with mixed-width data, we'll be heavily using this later.
This also explains why masks are stored in a vector register, more specifically in an LMUL=1 vector register, even though they only store one bit per element they are referring to. For LMUL=8 and 8-bit elements you need a full LMUL=1 register to have enough bits to represent its mask.

Apart from the RVV feature discussed already, the other features we'll be using are:

*   **reductions** over elements: Applies an operation to all elements to produce a scalar result. E.g. sum all elements.
*   **narrowing/widening** arithmetic operations: Operation that decreases/increases element width and LMUL.
*   **permutations**: Instructions to move elements, RVV supports slides, merge (blend), compress, and gather (shuffle)

I hope this wasn't too confusing, here are some more in-depth references on RVV if you don't feel prepared to follow along with the rest of the article:

Unicode defines a set of ~150,000 characters and assigns them a unique 32-bit number, called code point. Storing just the code points themself is called UTF-32\. This isn't done in practice, because lower code points occur more often, and this wastes a lot of space. There are two other encoding schemes: UTF-8, and UTF-16.

UTF-8 uses one- to four-bytes to represent a code point using the format visualized below:

Notice the small Invalid range. Code points between 0xD800-0xDFFF are unassigned and invalid, this is used to allow for the UTF-16 encoding.
UTF-16 encodes the code points from 1, 2 and 3 byte UTF-8 characters directly as a single 16-bit character. Four-byte UTF-8 code points are encoded in two 16-bit characters by leveraging part of the invalid character range to signal that it's a multi-word UTF-16 character.
16-bit words in the range 0xD800-0xDBFF are called high surrogates and in the range 0xDC00-0xDFFF low surrogates. A high surrogate is always followed by a low surrogate, the code points are encoded as follows:

Lastly, here is a side-by-side comparison of the encodings for the string "rνṿ🧙", which includes all UTF-8 character lengths:

Our goal is to implement a fast vectorized validating UTF-8 to UTF-16 conversion routine, but let's tackle non-validating general UTF-8 to UTF-32 conversion first and see where that leads us. We might end up simply converting the Unicode code point (UTF-32) to UTF-16, or figure out a use of some intermediate variables to get to UTF-16.

So, this leaves us with a few things to do:

*   identify character positions
*   remove prefixes
*   combine to UTF-32 code point

The most important question seems to be, how we deal with the different character sizes.
Initially, I had two ideas on how to approach this:

*   **vdecompress:** Skimming through the specification, the first instruction that seemed to fit the bill was vdecompress. Although it isn't really an instruction, but rather a combination of the `viota` and `vrgather` instructions to synthesize a `vcompress` inverse. It uses a mask to move every nth element of a vector to the nth active element in the source register. This could allow us to widen every UTF-8 character to four bytes long, so we can work on the different-sized characters uniformly.

*   **vcompress:** Alternatively we could also `vcompress` every nth byte of all UTF-8 characters into the nth of four separate vector registers. Then we could also write code that operates on all character sizes uniformly, but we'd need to recombine the registers to store the final code point.

The first approach seemed quite promising to me, so I sketched out the creation of the decompress-mask but got stuck on how to proceed from there. The problem is, that now you go from an input register of, let's for now assume LMUL=1, to an LMUL=4 register, and still need to do all of the logic to remove prefixes and shift the bits into place. That makes every operation we do four times slower, and we'd need to quite a few operations. Add to that, that `vrgather` is slow with larger LMULs (see [later discussion](#gather)), and this doesn't seem like that good of an idea anymore.

Let's consider the `vcompress` approach again. Once we've removed the prefixes from our four registers, we get the "shifting the bits into the right position" part basically for free, because we need to recombine them anyway. Using a widening multiply makes combining them while shifting by six bits even easier than interleaving the bytes (shifting by eight bits), because we can't multiply by `**1**<<**8**` since it doesn't fit into 8-bits.

I was hoping to use masked widening adds and multiplies, but that didn't end up being worth it, as we need to specify a destination operand that is already widened. Another complication is that combining the first two bytes with the last two bytes needs to shift the first two by 0, 6 or 12 bits, which doesn't nicely translate into a masked operation.
We can however always act like we have a four-byte UTF-8 character and later calculate and apply a correction right shift amount. This also removes the need to mask the add operations, as any residual bits are shifted away.

So here is the game plan:

We want to process the input in chunks, but if we were to load the data directly into a single vector register, we'd need to figure out where the last full character ends in the register. Instead of doing that we can always lookahead three bytes, and only consider them as continuation bytes, you'll see why that works out quite well later. Here is the framework we'll be building on top of:

```
size_t utf8_to_utf32(char const *src, size_t count, uint32_t *dest) {
 size_t tail = **3**;
 if (count < tail) return utf8_to_utf32_scalar(src, count, dest);
 `size_t n = count - tail;`
 `uint32_t *destBeg = dest;`
 ``for (size_t vl, vlOut; n > **0**; n -= vl, src += vl, dest += vlOut) {`
 `vl = __riscv_vsetvl_e8m2(n);`
 `vuint8m2_t v0 = __riscv_vle8_v_u8m2((uint8_t const*)src, vl);`
 ``*/* TODO: extract b1/b2/b3/b4 */*`
 `*/* TODO: remove prefixes */*`
 `*/* TODO: combine to b1234 */*`
 ``__riscv_vse32(dest, b1234, vlOut);`
 `}`
 ``*/* reparse last character + tail */*`
 `if (count > tail) {`
 `if ((src[**0**] >> **6**) == **0b10**) --dest;`
 `while ((src[**0**] >> **6**) == **0b10** && tail < count)`
 `--src, ++tail;`
 `}`
 `size_t ret = utf8_to_utf32_scalar(src, tail, dest);`
 `if (ret == **0**) return **0**;`
 `return (size_t)(dest - destBeg) + ret;`
`}````` 
```

 ``The loop structure itself is relatively simple, but we also need to make sure that we handle our lookahead of three correctly. For simplicity, we'll fall back to a scalar implementation for the tail, but we need to make sure we pass it a pointer to the beginning of a UTF-8 character.
Notice how we are loading `vuint8m2_t`, this is a bit optimistic, but the idea is that we can get away with using 16 instead of 32 registers. This would essentially unroll the loop and make the scalar and mask operations less expensive.

Initially, my thought was to create a mask of all the first bytes of all UTF-8 characters, compress those, and shift the mask right to extract the second bytes, and so on. Obtaining the mask is trivial, you just find all bytes that aren't continuation bytes: `x >> **6** != **0b10**`. RVV can't shift masks though, unless you treat the mask vector as a normal vector and assume it fits in 64-bit elements, or handle the carry.
Instead, we can left-shift (`vslide1down`) the elements of the vector itself and leave the mask constant. This now comes back to why the lookahead works so well, as we can specify a scalar to shift into the rightmost element.
The following brings the above concepts together:

```
*/* IMPL: extract b1/b2/b3/b4 */*
vuint8m2_t v1 = __riscv_vslide1down(v0, src[vl+**0**], vl);
vuint8m2_t v2 = __riscv_vslide1down(v1, src[vl+**1**], vl);
vuint8m2_t v3 = __riscv_vslide1down(v2, src[vl+**2**], vl);
*/* mask of non-continuation bytes */*
vbool4_t m = __riscv_vmsne(__riscv_vsrl(v0, **6**, vl), **0b10**, vl);
*/* extract third and fourth bytes */*
vuint8m2_t b1 = __riscv_vcompress(v0, m, vl);
vuint8m2_t b2 = __riscv_vcompress(v1, m, vl);
vuint8m2_t b3 = __riscv_vcompress(v2, m, vl);
vuint8m2_t b4 = __riscv_vcompress(v3, m, vl);

```

Removing the prefixes of b2/b3/b4 is trivial, we only need to mask out the two MSBs.

```
*/* IMPL: remove prefixes */*
*/* remove prefix from trailing bytes */*
vlOut = __riscv_vcpop(m, vl);
b2 = __riscv_vand(b2, **0b00111111**, vlOut);
b3 = __riscv_vand(b3, **0b00111111**, vlOut);
b4 = __riscv_vand(b4, **0b00111111**, vlOut);
*/* TODO: remove prefix from leading bytes */*

```

For b1 we need to determine how many bytes to mask-off, depending on the prefix. Fortunately, the first four bytes are enough to determine which bits to mask-off, and we could simply use a `vrgater` lookup table. On current hardware however another approach seems to be faster.

Instead of a mask, we can also use left and then immediately right shifts by the same value to remove prefix bits. The shift amounts for this can almost be calculated using a single saturating subtract 10:

There is one outlier, we can easily address using a merge:

```
*/* IMPL: remove prefix from leading bytes */*
vuint8m2_t shift = __riscv_vsrl(b1, **4**, vlOut);
shift = __riscv_vmerge(__riscv_vssubu(shift, **10**, vlOut), **3**,
 __riscv_vmseq(shift, **12**, vlOut), vlOut);
b1 = __riscv_vsll(b1, shift, vlOut);
b1 = __riscv_vsrl(b1, shift, vlOut);

```

The `vrgather` implementation should probably be revisited once more hardware is available.

As described above, well first treat all elements as four-byte UTF-8 characters and then right shift this into place.

The first part is trivially using widening operations, and assuming we've already calculated the correct right shift amount that we can just widen and apply:

```
*/* IMPL: combine to b1234 */*
*/* unconditionally widen and combine to b1234 */*
vuint16m4_t b34   = __riscv_vwaddu_wv(__riscv_vwmulu(b3,  **1**<<**6**,  vlOut), b4,  vlOut);
vuint16m4_t b12   = __riscv_vwaddu_wv(__riscv_vwmulu(b1,  **1**<<**6**,  vlOut), b2,  vlOut);
vuint32m8_t b1234 = __riscv_vwaddu_wv(__riscv_vwmulu(b12, **1**<<**12**, vlOut), b34, vlOut);
*/* TODO: compute shift amount */*
b1234 = __riscv_vsrl(b1234, __riscv_vzext_vf4(shift, vlOut), vlOut);

```

Computing the shift amount it's a bit more involved than last time, but it can be done using as follows:

This maps directly into the following code:

```
*/* IMPL: compute shift amount */*
*/* derive required right-shift amount from `shift` to reduce*
 ** b1234 to the required number of bytes */*
shift = __riscv_vmul(__riscv_vrsub(__riscv_vssubu(shift, **2**, vlOut), **3**, vlOut), **6**, vlOut);

```

For validation, we use the method described in ["Validating UTF-8 In Less Than One Instruction Per Byte"](https://arxiv.org/abs/2010.03090). I won't go into the full detail, but I'll summarise how it works:

There are seven error patterns in a 2 byte UTF-8 sequence, they can all be detected by only looking at the first 12 bits. We use three 4-bit lookup tables that map to a bitmask of the errors matching that pattern and AND them together to determine if the was an error. The only error not detected by this are related to 3-4 byte UTF-8 characters that have the wrong amount of continuation bytes. To detect those, the last remaining bit in the bitmask of the LUTs is used to indicate a pair of continuation bytes, which combined with a few comparisons manages to detect all invalid UTF-8 sequences.

Instead of doing a three LMUL=2 vrgather, we do six LMUL=1 `vrgather`, as we don't need to cross any lanes, and this performs better because it needs to do a less complex task (see [later discussion](#gather)). To help with that, we define the `VRGATHER_u8m1x2` macro, which unrolls the `vrgather` for us. Before we get into the implementation we also need to define the error lookup tables, and some constants we'll use later:

```
#define VRGATHER_u8m1x2(tbl, idx) \
 __riscv_vset(__riscv_vlmul_ext_u8m2( \
 __riscv_vrgather(tbl, __riscv_vget_u8m1(idx, **0**), vl8m1)), **1**, \
 __riscv_vrgather(tbl, __riscv_vget_u8m1(idx, **1**), vl8m1));
 `static const uint64_t err1m[] = { **0x0202020202020202**, **0x4915012180808080** };`
`static const uint64_t err2m[] = { **0xCBCBCB8B8383A3E7**, **0xCBCBDBCBCBCBCBCB** };`
`static const uint64_t err3m[] = { **0x0101010101010101**, **0x01010101BABAAEE6** };`
`const vuint8m1_t err1tbl = __riscv_vreinterpret_u8m1(__riscv_vle64_v_u64m1(err1m, **2**));`
`const vuint8m1_t err2tbl = __riscv_vreinterpret_u8m1(__riscv_vle64_v_u64m1(err2m, **2**));`
`const vuint8m1_t err3tbl = __riscv_vreinterpret_u8m1(__riscv_vle64_v_u64m1(err3m, **2**));`
`const size_t vl8m1 = __riscv_vsetvlmax_e8m1();`
`const size_t vl16m2 = __riscv_vsetvlmax_e16m2();` 
```

 `Now we need to extract the 4-bits and do the lookup three times.

```
*...*
*vuint8m2_t v3 = __riscv_vslide1down(v2, src[vl+2], vl);*
 `vuint8m2_t s1 = __riscv_vreinterpret_u8m2(__riscv_vsrl(__riscv_vreinterpret_u16m2(v2), **4**, vl16m2));`
`vuint8m2_t s3 = __riscv_vreinterpret_u8m2(__riscv_vsrl(__riscv_vreinterpret_u16m2(v3), **4**, vl16m2));`
 ``vuint8m2_t idx2 = __riscv_vand(v2, **0xF**, vl);`
`vuint8m2_t idx1 = __riscv_vand(s1, **0xF**, vl);`
`vuint8m2_t idx3 = __riscv_vand(s3, **0xF**, vl);`
 ``vuint8m2_t err1 = VRGATHER_u8m1x2(err1tbl, idx1);`
`vuint8m2_t err2 = VRGATHER_u8m1x2(err2tbl, idx2);`
`vuint8m2_t err3 = VRGATHER_u8m1x2(err3tbl, idx3);`
`vint8m2_t  errs = __riscv_vreinterpret_i8m2(`
 `__riscv_vand(__riscv_vand(err1, err2, vl), err3, vl));`
`*/* TODO: detect 3/4 byte errors */*``` 
```

 ``To check for 3/4 byte errors, we check if the previous input had a 3 or 4 byte character, which should be followed by two continuation bytes. There is no error, if we expect two continuations and get them, and if we don't expect two continuations and don't get them, this maps perfectly to an XOR operation. We use the fact, that the upper bit of our error bit set indicates the expectation of two continuations. Interpreting the byte as a signed number, lets us easily check if the MSB bit is set (`x < 0`) and if any of the other bits are set (`x > 0`).
Finally, we test if our error mask contains an error, and exit the function with an error code.

```
*/* IMPL: detect 3/4 byte errors */*
vbool4_t is_3 = __riscv_vmsgtu(v1, **0b11100000**-1, vl);
vbool4_t is_4 = __riscv_vmsgtu(v0, **0b11110000**-1, vl);
vbool4_t is_34 = __riscv_vmor(is_3, is_4, vl);
vbool4_t err34 = __riscv_vmxor(is_34, __riscv_vmslt(errs, **0**, vl), vl);
vbool4_t errm = __riscv_vmor(__riscv_vmsgt(errs, **0**, vl), err34, vl);
 `if (__riscv_vfirst(errm, vl) >= **0**)`
 `return **0**;` 
```

 `Lastly, we can't forget about the first three bytes. They could e.g. be all continuation bytes, which would cause our loop to ignore them. Before the start of our loop, we find the end of the thired UTF-8 character and pass that to a scalar validation routine, here we reuse `utf8_to_utf32_scalar` for simplicity:

```
*...*
*if (count < tail) return utf8_to_utf32_scalar(src, count, dest);*
*/* validate first three bytes */*
{
 size_t idx = tail;
 while (idx < count && (src[idx] >> **6**) == **0b10**)
 ++idx;
 uint32_t buf[**10**];
 if (idx > tail + **3** || !utf8_to_utf32_scalar(src, idx, buf))
 return **0**;
}

```

Now for the fun part, making things faster.

If we read an all ASCII vector, then we can skip the validation pass, and simply widen and store the vector. We use a max reduction to determine if we have only ASCII bytes, the result of which we can also use for our other fast paths:

```
*...*
*vuint8m2_t v0 = __riscv_vle8_v_u8m2((uint8_t const*)src, vl);*
uint64_t max = __riscv_vmv_x(__riscv_vredmaxu(v0, __riscv_vmv_s_x_u8m1(**0**, vl), vl));
 `*/* fast path: ASCII */*`
`if (max < **0b10000000**) {`
 `vlOut = vl;`
 `__riscv_vse32(dest, __riscv_vzext_vf4(v0, vlOut), vlOut);`
 `continue;`
`}` 
```

 `This fast path needs to happen after validation. We are only concerned with creating b12 from b1 and b2, which allows us to simplify the code from the general case a lot.
We don't need to bother with shifting to remove the prefix from b1, there are only two possibilities, and one is to do nothing, hence a masked and fits perfectly.
We still can't use a masked widening multiply without first widening the destination operand, but we can use a simple `vmerge` to select between the two possible shift values. The addition can then be done using a masked widening add because the destination is already widened.
Now we need to zero extend again and we are done:

```
*...*
*vuint8m2_t b2 = __riscv_vcompress(v1, m, vl);*
vlOut = __riscv_vcpop(m, vl); */* needs to be moved up from previous place to here */*
 `*/* fast path: one and two byte */*`
`if (max < **0b11100000**) {`
 `b2 = __riscv_vand(b2, **0b00111111**, vlOut);`
 `vbool4_t m1 = __riscv_vmsgtu(b1, **0b10111111**, vlOut);`
 `b1 = __riscv_vand_mu(m1, b1, b1, **63**, vlOut);`
 `vuint16m4_t b12 = __riscv_vwmulu(b1, __riscv_vmerge(__riscv_vmv_v_x_u8m2(**1**, vlOut), **1**<<**6**, m1, vlOut), vlOut);`
 `b12 = __riscv_vwaddu_wv_mu(m1, b12, b12, b2, vlOut);`
 `__riscv_vse32(dest, __riscv_vzext_vf2(b12, vlOut), vlOut);`
 `continue;`
`}` 
```

 `I'll leave understanding this one as an exercise to the reader, note that the code points of all three and below byte UTF-8 characters fit into 16 bytes.

```
*/* fast path: one and two byte */*
*...*
 `*/* fast path: one, two and three byte */*`
`if (max < **0b11110000**) {`
 `vuint8m2_t b3 = __riscv_vcompress(v2, m, vl);`
 `b2 = __riscv_vand(b2, **0b00111111**, vlOut);`
 `b3 = __riscv_vand(b3, **0b00111111**, vlOut);`
 ``vbool4_t m1 = __riscv_vmsgtu(b1, **0b10111111**, vlOut);`
 `vbool4_t m3 = __riscv_vmsgtu(b1, **0b11011111**, vlOut);`
 ``vuint8m2_t t1 = __riscv_vand(m1, b1, b1, **63**, vlOut);`
 `b1 = __riscv_vand(m3, t1, b1, **15**, vlOut);`
 ``vuint16m4_t b12 = __riscv_vwmulu(b1, __riscv_vmerge(__riscv_vmv_v_x_u8m2(**1**, vlOut), **1**<<**6**, m1, vlOut), vlOut);`
 `b12 = __riscv_vwaddu_wv(m1, b12, b12, b2, vlOut);`
 `vuint16m4_t b123 = __riscv_vwaddu_wv_(m3, b12, __riscv_vsll_vx_u16m4_mu(m3, b12, b12, **6**, vlOut), b3, vlOut);`
 `__riscv_vse32(dest, __riscv_vzext_vf2(b123, vlOut), vlOut);`
 `continue;`
`}```` 
```

 ``Every 1 to 3 byte UTF-8 character can be represented in a single word UTF-16 character, so for the fast paths above, we need to widen and store to 16 instead of 32 bits.
Hey! The fast paths just got a bit faster!

The next step starts right after we've computed the UTF-32 code point in the general path. We treat all characters as two-word UTF-16 and convert them to surrogate pairs accordingly. Then we `vmerge` between the original UTF-32 code point and the two-word UTF-16 code points. Afterward, we use the original UTF-32 code point to create a compression mask for our UTF-32 words. This can be done by matching the UTF-32 code points that fit into 16 bits, their upper two bytes are zero.

We predefine a mask `m2even` that masks all odd indexes and two helpers, as we'll need to switch between treating the vector as a vector of 16-bit and 32-bit elements.

```
size_t vl8m4 = __riscv_vsetvlmax_e8m4();
const vbool2_t m2even = __riscv_vmseq(__riscv_vand(__riscv_vid_v_u8m4(vl8m4), **1**, vl8m4), **0**, vl8m4);

#define DOWN __riscv_vreinterpret_u16m8
#define UP __riscv_vreinterpret_u32m8

```

Now we shift the bits around to create the surrogate representation of each code point and then select between this and the UTF-32 code point using `vmerge`. To obtain the correct UTF-16 output, we need to compress all odd words that are zero, as those are only present in the one single UTF-16 character currently stored as UTF-32.

```
*...*
*b1234 = __riscv_vsrl(b1234, ...);*
*/* convert [000000000000aaaa|aaaaaabbbbbbbbbb]*
 ** to      [110111bbbbbbbbbb|110110aaaaaaaaaa] */*
vuint32m8_t sur = __riscv_vsub(b1234, **0x10000**, vlOut);
sur = __riscv_vor(__riscv_vsll(sur, **16**, vlOut),
 __riscv_vsrl(sur, **10**, vlOut), vlOut);
sur = __riscv_vand(sur, **0x3FF03FF**, vlOut);
sur = __riscv_vor(sur, **0xDC00D800**, vlOut);
*/* merge 1 byte b1234 and 2 byte sur */*
vbool4_t m4 = __riscv_vmsgtu(b1234, **0xFFFF**, vlOut);
b1234 = __riscv_vmerge(b1234, sur, m4, vlOut);
*/* swap b1234 two byte pairs */*
*/* compress and store */*
vbool2_t mOut = __riscv_vmor(__riscv_vmsne(DOWN(b1234), **0**, vlOut***2**), m2even, vlOut***2**);
b1234 = UP(__riscv_vcompress(DOWN(b1234), mOut, vlOut***2**));
size_t vlDest = __riscv_vcpop(mOut, vlOut***2**);
__riscv_vse16_v_u16m8(dest, DOWN(b1234), vlDest);

```

Finally, we need to adjust the tail handling to account for UTF-16 output. Since we want to reparse the last character, we need to decrement the destination pointer, if the last output word was a high surrogate.

```
*/* reparse last character + tail */*
*if (count > tail) {*
 *if ((src[0] >> 6) == 0b10) --dest;*
 *while ((src[0] >> 6) == 0b10 && tail < count)*
 *--src, ++tail;*
 */* go back one more, when on high surrogate */*
 if (dest[-**1**] >= **0xD800** && dest[-**1**] <= **0xDBFF**)
 --dest;
*}*

```

Let's do one last optimization. If you look at the general case again, you might notice that we are doing twice the needed work needed, if the average input character size is equal to or above two. To circumvent that, we can lower the LMUL from two to one and unroll the code with an early exit between the iterations.

This is a simple transformation, and you can browse the [final code here](https://github.com/camel-cdr/rvv-bench/blob/main/vector-utf/8toN_gather.c). It's a bit more complex because it implements both the UTF-8 to UTF-16 and UTF-8 to UTF-32 conversions in one function.

I used the Lemires [unicode_lipsum](https://github.com/lemire/unicode_lipsum/) dataset to measure the performance for inputs of different languages. You can look at the [benchmark code here](https://github.com/camel-cdr/rvv-bench/blob/main/vector-utf/bench.c), it reads an input file into a buffer and measures the average time a conversion of that input takes.

The code for the `C920` needed to be manually backported to the draft 0.7.1 RVV version, which introduced a few pessimization:

*   `vmvNr` needed to be replaced with one `vmv.v.v` and two `vsetvli` instructions.
*   `vzext.vN` needed to be replaced with N/2 `vwaddu.vx` and N/2+1 `vsetvli` instructions.
*   `vredmax` needed to be replaced with one `vmsgtu.vx` and one `vfirst.m` instruction.
    This shouldn't be needed for RVV 0.7.1, but the hardware I've got access to seems to produce the wrong result for vredmax. I haven't looked into this further.

The [unicode_lipsum](https://github.com/lemire/unicode_lipsum/) dataset is split into two parts. One contains [Lorem ipsum](https://en.wikipedia.org/wiki/Lorem_ipsum) style text in different languages and was created by Hans Kratz, for the [simdutf8](https://github.com/lemire/unicode_lipsum/blob/main/simdutf8) project.
This gives us very dense input, the average character sizes are as follows: Arabic: 1.8 Chinese: 3.0 Emoji: 4.0 Hebrew: 1.8 Hindi: 2.7 Japanese: 2.9 Korean: 2.5 Latin: 1.0 Russian: 1.8

Excluding the, all ASCII, Latin input, we get an average speedup of 3x over scalar on the C920, and 3.5x on the C908\. That's a lot!
We are a bit lacking in the Emoji test case, which solely consists of 4-byte UTF-8 emojis. We could add an extra fast path for that, but such input will rarely occur in the real world, so I don't think it's worth optimizing for.

The second part of the data set includes the source code of different translations for the [Mars](https://en.wikipedia.org/wiki/Mars) Wikipedia entries. This is less dense, no average character length is above 2.0, but probably more representative, as a lot of text comes in some sort of structural format (XML, JSON, ...) or has Arabic numerals, links, ... in between the native language characters.

Here, we get an average speedup of 3.6x over scalar on the C920, and 4.0x on the C908, excluding the all ASCII English case.

We managed to get a good speedup on real hardware, but we are still in the early days of RVV implementations. Will the code also have similar speedups on future hardware that might support bigger vector lengths?

Most RVV instructions can be expected to perform well across implementations, but the permutation instructions, specifically `vrgather.vv` and `vcompress.vm`, are harder to scale and already exhibit a variety of performance characteristics. I've compiled cycle estimates for all implementations I could get the data for:

**bobcat: Note, that it was explicitly stated, that they didn't optimize the permutation instructions*

**x280: the numbers are from llvm-mca, but I was told they match reality. There is also supposed to be a `vrgather.vv` fast path for vl<=256\. I think they didn't have much incentive to optimize this, as the x280 targets AI workloads w.*

Now you might see why we've decided to use six LMUL=1 gathers instead of three LMUL=2 gathers for our validation lookup tables. Current implementations don't scale well to larger LMUL, and some won't perform well enough for our implementation on any LMUL.

I think that the C920 results are the most representative of what to expect of future desktop CPUs and we can safely ignore the bobcat/x280 cycle counts for that use case. I suspect we'll see `vrgather.vv` perform well for LMUL=1, and then grow exponentially per element with higher LMUL, as an all-to-all mapping is quite expensive to scale.
`vcompress.vm` should be better scalable than `vrgather.vv`, since the work is subdividable, and I think we might see a range of implementations that scale almost linearly in respect to LMUL.

We've seen that `vrgather.vv` is really useful for quick lookup tables, but provides a way more powerful all-to-all mapping than required for that. Four-bit lookup tables specifically are very likely to be used by a bunch of other implementations as well, as the standard V extension guarantees a VLEN of at least 128, so a 4-bit LUT can always be assumed to fit into one vector register.

I don't know anything about hardware design, but implementing a LMUL=1 `vrgather.vv` for very long vector implementations that performs, per element, as fast as one for a smaller implementation, seems almost impossible. If you expect to use it for its full capabilities, it isn't a big problem, sometimes you just need to get the elements into a specific permutation. But for 4-bit lookup tables we only care about selecting one of 16 values.

Hardware might be able to have a special case/fast path implementation for smaller indices, but we can't expect most implementations to support it. I think adding a dedicated `vrgatherei4`, which only uses the four LSB for indices, to the specification might be a good idea.

We've managed to effectively use the RISC-V Vector extension to speed up UTF-8 to UTF-16 conversion by on average 3 to 4 times on real hardware. [This and other conversion functions](https://github.com/simdutf/simdutf/pull/373) have been successfully upstreamed to the [simdutf](https://github.com/simdutf/simdutf/) library. Hopefully, we can see this contribute to bridging the optimization gap of RISC-V compared to other architectures, and maybe even inspire some people to give porting to RVV a shot.

I hope you could follow this write up, have a nice day :3

If you are wondering how the RISC-V hardware compares to existing hardware of other architectures, here are the measurements for the dataset from above including simdutf implementations for the other architectures.``````````