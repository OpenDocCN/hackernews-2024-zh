<!--yml
category: 未分类
date: 2024-05-27 13:11:16
-->

# c++ - Fastest way to get IPv4 address from string - Stack Overflow

> 来源：[https://stackoverflow.com/questions/31679341/fastest-way-to-get-ipv4-address-from-string](https://stackoverflow.com/questions/31679341/fastest-way-to-get-ipv4-address-from-string)

Since we are speaking about maximizing throughput of IP address parsing, I suggest using a vectorized solution.

Here is x86-specific fast solution (needs SSE4.1, or at least SSSE3 for poor):

```
__m128i shuffleTable[65536];    //can be reduced 256x times, see @IwillnotexistIdonotexist

UINT32 MyGetIP(const char *str) {
    __m128i input = _mm_lddqu_si128((const __m128i*)str);   //"192.167.1.3"
    input = _mm_sub_epi8(input, _mm_set1_epi8('0'));        //1 9 2 254 1 6 7 254 1 254 3 208 245 0 8 40 
    __m128i cmp = input;                                    //...X...X.X.XX...  (signs)
    UINT32 mask = _mm_movemask_epi8(cmp);                   //6792 - magic index
    __m128i shuf = shuffleTable[mask];                      //10 -1 -1 -1 8 -1 -1 -1 6 5 4 -1 2 1 0 -1 
    __m128i arr = _mm_shuffle_epi8(input, shuf);            //3 0 0 0 | 1 0 0 0 | 7 6 1 0 | 2 9 1 0 
    __m128i coeffs = _mm_set_epi8(0, 100, 10, 1, 0, 100, 10, 1, 0, 100, 10, 1, 0, 100, 10, 1);
    __m128i prod = _mm_maddubs_epi16(coeffs, arr);          //3 0 | 1 0 | 67 100 | 92 100 
    prod = _mm_hadd_epi16(prod, prod);                      //3 | 1 | 167 | 192 | ? | ? | ? | ?
    __m128i imm = _mm_set_epi8(-1, -1, -1, -1, -1, -1, -1, -1, -1, -1, -1, -1, 6, 4, 2, 0);
    prod = _mm_shuffle_epi8(prod, imm);                     //3 1 167 192 0 0 0 0 0 0 0 0 0 0 0 0
    return _mm_extract_epi32(prod, 0);
//  return (UINT32(_mm_extract_epi16(prod, 1)) << 16) + UINT32(_mm_extract_epi16(prod, 0)); //no SSE 4.1
} 
```

And here is the required precalculation for `shuffleTable`:

```
void MyInit() {
    memset(shuffleTable, -1, sizeof(shuffleTable));
    int len[4];
    for (len[0] = 1; len[0] <= 3; len[0]++)
        for (len[1] = 1; len[1] <= 3; len[1]++)
            for (len[2] = 1; len[2] <= 3; len[2]++)
                for (len[3] = 1; len[3] <= 3; len[3]++) {
                    int slen = len[0] + len[1] + len[2] + len[3] + 4;
                    int rem = 16 - slen;
                    for (int rmask = 0; rmask < 1<<rem; rmask++) {
//                    { int rmask = (1<<rem)-1;    //note: only maximal rmask is possible if strings are zero-padded
                        int mask = 0;
                        char shuf[16] = {-1, -1, -1, -1, -1, -1, -1, -1, -1, -1, -1, -1, -1, -1, -1, -1};
                        int pos = 0;
                        for (int i = 0; i < 4; i++) {
                            for (int j = 0; j < len[i]; j++) {
                                shuf[(3-i) * 4 + (len[i]-1-j)] = pos;
                                pos++;
                            }
                            mask ^= (1<<pos);
                            pos++;
                        }
                        mask ^= (rmask<<slen);
                        _mm_store_si128(&shuffleTable[mask], _mm_loadu_si128((__m128i*)shuf));
                    }
                }
} 
```

Full code with testing is avaliable [here](http://pastebin.com/XSiPqHuN). On Ivy Bridge processor it prints:

```
C0A70103
Time = 0.406   (1556701184)
Time = 3.133   (1556701184) 
```

It means that the suggested solution is **7.8 times faster** in terms of throughput than the code by OP. It processes **336 millions of addresses per second** (single core of 3.4 Ghz).

Now I'll try to explain how it works. Note that on each line of the listing you can see contents of the value just computed. All the arrays are printed in little-endian order (though `set` intrinsics use big-endian).

First of all, we load 16 bytes from unaligned address by `lddqu` instruction. Note that in 64-bit mode memory is allocated by 16-byte chunks, so this works well automatically. On 32-bit it may theoretically cause issues with out of range access. Though I do not believe that it really can. The subsequent code would work properly regardless of the values in the after-the-end bytes. Anyway, you'd better ensure that each IP address takes at least 16 bytes of storage.

Then we subtract '0' from all the chars. After that '.' turns into -2, and zero turns into -48, all the digits remain nonnegative. Now we take bitmask of signs of all the bytes with `_mm_movemask_epi8`.

Depending on the value of this mask, we fetch a nontrivial 16-byte shuffling mask from lookup table `shuffleTable`. The table is quite large: 1Mb total. And it takes quite some time to precompute. However, it does not take precious space in CPU cache, because only 81 elements from this table are really used. That is because each part of IP address can be either one, two, three digits long => hence 81 variants in total. Note that random trashy bytes after the end of the string may in principle cause increased memory footprint in the lookup table.

*EDIT*: you can find a version modified by @IwillnotexistIdonotexist in comments, which uses lookup table of only 4Kb size (it is a bit slower, though).

The ingenious `_mm_shuffle_epi8` intrinsic allows us to reorder the bytes with our shuffle mask. As a result XMM register contains four 4-byte blocks, each block contains digits in little-endian order. We convert each block into a 16-bit number by `_mm_maddubs_epi16` followed by `_mm_hadd_epi16`. Then we reorder bytes of the register, so that the whole IP address occupies the lower 4 bytes.

Finally, we extract the lower 4 bytes from the XMM register to GP register. It is done with SSE4.1 intrinsic (`_mm_extract_epi32`). If you don't have it, replace it with other line using `_mm_extract_epi16`, but it will run a bit slower.

Finally, here is the generated assembly (MSVC2013), so that you can check that your compiler does not generate anything suspicious:

```
lddqu   xmm1, XMMWORD PTR [rcx]
psubb   xmm1, xmm6
pmovmskb ecx, xmm1
mov ecx, ecx               //useless, see @PeterCordes and @IwillnotexistIdonotexist
add rcx, rcx               //can be removed, see @EvgenyKluev
pshufb  xmm1, XMMWORD PTR [r13+rcx*8]
movdqa  xmm0, xmm8
pmaddubsw xmm0, xmm1
phaddw  xmm0, xmm0
pshufb  xmm0, xmm7
pextrd  eax, xmm0, 0 
```

P.S. If you are still reading it, be sure to check out comments =)