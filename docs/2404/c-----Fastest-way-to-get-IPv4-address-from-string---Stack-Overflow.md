<!--yml

category: 未分类

date: 2024-05-27 13:11:16

-->

# c++ - 从字符串获取 IPv4 地址的最快方法 - Stack Overflow

> 来源：[https://stackoverflow.com/questions/31679341/fastest-way-to-get-ipv4-address-from-string](https://stackoverflow.com/questions/31679341/fastest-way-to-get-ipv4-address-from-string)

由于我们正在讨论最大化 IP 地址解析的吞吐量，我建议使用矢量化解决方案。

这里是 x86 特定的快速解决方案（需要 SSE4.1，或至少对于贫穷来说 SSSE3）：

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

这意味着建议的解决方案在吞吐量上比 OP 的代码快了**7.8倍**。它每秒处理**336百万个地址**（3.4 GHz 的单核）。

现在我将尝试解释它的工作原理。请注意，列表每一行显示的是刚刚计算的值的内容。所有数组按照小端顺序打印（虽然 `set` 内置函数使用大端顺序）。

首先，我们通过 `lddqu` 指令从非对齐地址加载 16 字节。请注意，在 64 位模式下，内存自动以 16 字节块分配，因此这自动有效。在 32 位模式下，理论上可能会导致超出范围的访问问题。尽管我不认为这真的可能发生。无论如何，最好确保每个 IP 地址至少占用 16 字节的存储空间。

然后我们从所有字符中减去 '0'。之后，'.' 变成 -2，零变成 -48，所有数字保持非负。现在我们使用 `_mm_movemask_epi8` 获取所有字节的符号位掩码。

根据这个掩码的值，我们从查找表 `shuffleTable` 中获取一个非平凡的 16 字节洗牌掩码。该表总共有 1Mb，预计计算时间也相当长。然而，它并不占用 CPU 缓存中的宝贵空间，因为仅使用了这个表的 81 个元素。这是因为 IP 地址的每一部分都可以是一位、两位或三位数 => 因此总共有 81 种变体。请注意，字符串末尾的随机垃圾字节可能会增加查找表的内存占用量。

*   *EDIT*: you can find a version modified by @IwillnotexistIdonotexist in comments, which uses lookup table of only 4Kb size (it is a bit slower, though).

精巧的 `_mm_shuffle_epi8` 内在函数允许我们使用我们的洗牌掩码重新排序字节。结果是 XMM 寄存器包含四个 4 字节块，每个块按小端顺序包含数字。我们通过 `_mm_maddubs_epi16` 后跟 `_mm_hadd_epi16` 将每个块转换为 16 位数字。然后重新排序寄存器的字节，使整个 IP 地址占用较低的 4 字节。

最后，我们从XMM寄存器中提取低4字节到GP寄存器。这是通过SSE4.1内置函数（`_mm_extract_epi32`）完成的。如果你没有它，可以使用`_mm_extract_epi16`替换，但运行速度会慢一些。

最后，这里是生成的汇编代码（MSVC2013），这样你可以检查你的编译器是否生成了任何可疑的内容：

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

P.S. 如果你还在阅读，请务必查看评论 =)
