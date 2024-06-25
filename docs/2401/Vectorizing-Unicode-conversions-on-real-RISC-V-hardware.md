<!--yml

类别：未分类

日期：2024-05-27 15:14:03

-->

# 在实际 RISC-V 硬件上实现向量化 Unicode 转换

> 来源：[`camel-cdr.github.io/rvv-bench-results/articles/vector-utf.html`](https://camel-cdr.github.io/rvv-bench-results/articles/vector-utf.html)

# 在实际 RISC-V 硬件上实现向量化 Unicode 转换

在本文中，我们将讨论如何利用 RISC-V 向量扩展实现 UTF-8 到 UTF-16 转换的加速。

我从上面的图表中排除了纯 ASCII 情况，因为这使得其他结果变得不太可读，速度提升为：C908 8 倍，C920 11 倍。更全面的测量结果在文章的结尾。

绝大多数文本都以 UTF-8 格式编码，但某些语言和 API 使用 UTF-16 作为其本机格式（JavaScript、Java、Windows 等）。这个和其他原因可能导致你需要在不同的 Unicode 编码之间进行转换。正如[simdutf](https://github.com/simdutf/simdutf/)所示，这个转换过程具有很多优化潜力。

在这里，我们将专注于 UTF-8 到 UTF-16 转换，并旨在开发一个经过优化的 RISC-V 实现，可以上游到[simdutf](https://github.com/simdutf/simdutf/)库中，该库被 Node.js 和 Bun 等项目使用。（[更改](https://github.com/simdutf/simdutf/pull/373)现已上游）

RISC-V 向量扩展（RVV）添加了一组 32 个向量寄存器，每个寄存器的宽度为 VLEN 位，其中 VLEN 是大于或等于 128 的 2 的幂（在标准 V 扩展中）。向量寄存器可以解释为多个 8/16/32/64 位元素，并相应地以有符号/无符号整数或单精度/双精度浮点数进行操作。由于我们可以一次操作多个元素，因此相对于标量代码，可以实现大幅加速。

在撰写本文时（2024 年初），几乎没有硬件支持 RVV。

+   唯一支持 RVV 的硬件，普通消费者可以购买的是[Kendryte K230](https://www.canaan.io/product/k230)。它具有来自旋铁的 1.6GHz 的有序核[C908](https://www.t-head.cn/product/%E7%8E%84%E9%93%81C908)，VLEN 为 128 位。（C908 基准测试页面)

+   你也可以购买另外两款支持早期不兼容草案版本（0.7.1）的向量扩展的 CPU，即[C906](https://www.t-head.cn/product/C906?lang=en)和[C920](https://www.t-head.cn/product/C910?lang=en)（C910 带有 RVV）。由于 C906 的性能特征与 C908 非常相似，我们不会为此提供基准测试。(C906 基准测试页面)

+   然而，[C920](https://www.t-head.cn/product/c910?lang=en)是一个速度更快的乱序核心，频率为 2GHz，其结果比未来硬件的 C908 更加有趣。目标是 RVV 0.7.1，这是很困难的，因为没有官方的工具链支持，但我花了时间手动翻译生成的汇编代码，并且用一个[较旧的 GCC 分支](https://github.com/brucehoult/riscv-gnu-toolchain)来汇编它。（C920 基准测试页面）

+   有[一些开源的 RVV 实现](https://github.com/stars/camel-cdr/lists/rvv-implementations)，但大多数仍处于开发中/不完整状态。唯一一个完整的，我们可以在本地模拟的是 Tenstorrents bobcat（以前是 ocelot），但它是一个概念验证设计，我们将使用的一些指令明确未经过优化。（我们稍后将讨论此事）

幸运的是，我拥有 Kendryte K230，并且能够通过 [perfXlab](http://www.perfxlab.com/) 获得对拥有 64 个 C920 核心的 Milk-V Pioneer 服务器的 ssh 访问权限。开发是通过 qemu 模拟完成的，因为这比使用真实硬件要简单得多，目前是这样的。

接下来的两节将介绍 RVV 和 Unicode 的基础知识，如果你已经对这些主题很熟悉，可以随意跳过。

我将尝试使用一个简短的例子来解释 RVV 的基础知识：

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

在这里，我们使用 C 内部函数 API 来计算提供数据中的 UTF-8 字符数。下一节将更详细地描述这一点，但是要计算 UTF-8 字符的数量，我们只需要计算不匹配模式`**0b10xxxxxx**`的字节数，假设输入是有效的。

**从现在开始，当我提到“字符”时，我指的是一个 Unicode 代码点。这并不直接映射到屏幕上的单个字符，例如，这个表情符号 "🧙‍♀️" 由三个 Unicode 代码点构成：🧙 + 零宽度连接器 + ♀️ = 🧙‍♀️*

如上所述，RVV 支持不同的向量长度。为了使相同的代码能够在具有不同向量长度的机器上运行，RVV 提供了`vsetvl*`指令。你需要给出你的输入元素的数量和元素宽度，它将给你一个小于或等于你提供的数量的计数，以便将其装入一个向量寄存器中。

上面的代码使用了这个功能来迭代输入，`vl = vsetvl_e8m8(len)`代表一次迭代处理的元素数量。接下来的`vle8_v_u8m8()`从我们的输入中加载`vl`个 8 位整数元素到一个向量寄存器中。

然后创建一个掩码，其中每个活动元素（掩码中对应位设置的元素）与模式`**0b10xxxxxx**`不匹配。vsrl 代表*右移逻辑*，vmsne 代表*不等于*，所以`vmsne(vsrl(v, **6**, vl), **0b10**, vl)`在每个元素上执行`(x >> **6**) != **0b10**`。RVV 没有单独的掩码寄存器，而是将掩码存储在向量寄存器的低位中，内置 API 添加了`vboolN_t`类型以提供更多的类型安全性。最后，我们使用`vcpop`来计算我们的掩码中活动元素的数量，并将其加到我们的总和中。

你可能想知道"m8"是什么意思，到目前为止我还没有提到过。RVV 有 32 个 VLEN 位宽的向量寄存器，但是通过`vsetvl`你也可以配置 LMUL（长度乘数），并使处理器将这些寄存器分组。这意味着后续指令将在一个寄存器组上执行，并且`vsetvl`将返回与 LMUL 对应的`vl`。

当 LMUL=1 时，我们有 32 个 VLEN 位宽的寄存器，当 LMUL=2 时，它们现在就像是 16 个 VLEN*2 位宽的寄存器，所以对于"m8"（LMUL=8），我们有 4 个 VLEN*8 位宽的寄存器。在这里，我们使用少于五个向量寄存器，所以使用 LMUL=8 基本上就给了我们免费的循环展开，这使得标量和掩码操作的成本降低了。展开不是 LMUL 的唯一优势，它还允许我们轻松地处理混合宽度的数据，我们稍后将大量使用它。

这也解释了为什么掩码存储在向量寄存器中，更具体地说是存储在 LMUL=1 向量寄存器中，即使它们每个元素只存储一个比特，它们所指的。对于 LMUL=8 和 8 位元素，你需要一个完整的 LMUL=1 寄存器来存储足够的比特来表示其掩码。

除了已经讨论过的 RVV 功能之外，我们将使用的其他功能包括：

+   **减少**元素：对所有元素应用操作以产生一个标量结果。例如，对所有元素求和。

+   **窄化/扩展**算术运算: 减小/增加元素宽度和 LMUL 的操作。

+   **排列**: 移动元素的指令，RVV 支持滑动、合并（混合）、压缩和聚集（洗牌）。

希望这不会让你太困惑，如果你觉得自己还不够准备好跟上文章的后续内容，这里有一些关于 RVV 更深入的参考资料：

Unicode 定义了一组约 150,000 个字符，并为它们分配了一个唯一的 32 位数字，称为代码点。仅存储代码点本身称为 UTF-32。实际上不这样做，因为较低的代码点更常见，这会浪费大量空间。还有两种其他的编码方案：UTF-8 和 UTF-16。

UTF-8 使用一到四个字节来表示一个代码点，使用下面可视化的格式：

注意小的无效范围。代码点在 0xD800-0xDFFF 之间是未分配和无效的，这用于允许 UTF-16 编码。

UTF-16 直接将 1、2 和 3 字节的 UTF-8 字符的代码点直接编码为单个 16 位字符。四字节 UTF-8 代码点通过利用部分无效字符范围来信号化为多字节 UTF-16 字符的两个 16 位字符进行编码。

在范围 0xD800-0xDBFF 内的 16 位字称为高代理，而在范围 0xDC00-0xDFFF 内的字称为低代理。高代理总是后跟低代理，编码如下：

最后，这里是字符串"rνṿ🧙"的编码的对比，其中包括所有 UTF-8 字符长度：

我们的目标是实现一个快速的矢量化验证 UTF-8 到 UTF-16 的转换例程，但让我们首先解决非验证通用 UTF-8 到 UTF-32 的转换，看看这会带给我们什么。我们可能最终会简单地将 Unicode 代码点（UTF-32）转换为 UTF-16，或者找出一些中间变量的用法来得到 UTF-16。

所以，这给我们留下了一些事情要做：

+   确定字符位置

+   移除前缀

+   组合到 UTF-32 代码点

最重要的问题似乎是，我们如何处理不同的字符大小。

起初，我有两个想法来解决这个问题：

+   **vdecompress:** 浏览规范，看起来符合条件的第一条指令是 vdecompress。尽管它实际上并不是一条指令，而是`viota`和`vrgather`指令的组合，用于合成一个`vcompress`的逆过程。它使用一个掩码将向量的每个第 n 个元素移动到源寄存器中的第 n 个活动元素。这可以使我们将每个 UTF-8 字符扩展为四个字节长，这样我们就可以统一处理不同大小的字符。

+   **vcompress:** 或者我们也可以`vcompress`每个 UTF-8 字符的第 n 个字节到四个单独的向量寄存器的第 n 个字节。然后我们也可以编写操作所有字符大小的代码，但我们需要重新组合寄存器以存储最终的代码点。

第一种方法对我来说似乎很有前途，所以我草绘了创建解压缩掩码的过程，但卡在了如何继续的地方。问题是，现在你从一个输入寄存器转到了，暂时假设 LMUL=1，到一个 LMUL=4 寄存器，仍然需要做所有去除前缀并将位移动到位的逻辑。这使得我们进行的每个操作都慢了四倍，而且我们需要进行相当多的操作。加上`vrgather`对于较大的 LMUL 的速度较慢（见后续讨论），这似乎不再是一个好主意了。

让我们再次考虑 `vcompress` 方法。一旦我们从四个寄存器中移除了前缀，我们基本上就可以免费获得“将位移动到正确位置”的部分，因为无论如何我们都需要重新组合它们。使用扩展乘法使得在移位六位的同时组合它们变得更容易，甚至比交错字节（移位八位）还要容易，因为我们无法用 `**1**<<**8**` 乘以 8 位。

我本来希望使用掩码扩展加法和乘法，但结果证明并不值得，因为我们需要指定一个已经扩展的目标操作数。另一个复杂之处在于，将前两个字节与后两个字节组合需要将前两个字节左移 0、6 或 12 位，这并不能很好地转换为掩码操作。

但是我们总是可以假装我们有一个四字节的 UTF-8 字符，然后计算并应用一个正确的右移量。这也消除了需要屏蔽加法操作的必要性，因为任何剩余的位都会被移位掉。

所以我们的游戏计划如下：

我们想要以块的形式处理输入，但是如果我们直接将数据加载到单个矢量寄存器中，我们需要弄清楚寄存器中最后一个完整字符的结束位置。我们可以始终向前看三个字节，只将它们视为连续字节，稍后你会看到为什么这样做非常合适。以下是我们将要构建的框架：

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

*/* IMPL: 提取 b1/b2/b3/b4 */*

vuint8m2_t v1 = __riscv_vslide1down(v0, src[vl+**0**], vl);

vuint8m2_t v2 = __riscv_vslide1down(v1, src[vl+**1**], vl);

vuint8m2_t v3 = __riscv_vslide1down(v2, src[vl+**2**], vl);

*/* 非连续字节的掩码 */*

vbool4_t m = __riscv_vmsne(__riscv_vsrl(v0, **6**, vl), **0b10**, vl);

*/* 提取第三和第四个字节 */*

vuint8m2_t b1 = __riscv_vcompress(v0, m, vl);

vuint8m2_t b2 = __riscv_vcompress(v1, m, vl);

vuint8m2_t b3 = __riscv_vcompress(v2, m, vl);

vuint8m2_t b4 = __riscv_vcompress(v3, m, vl);

```

Removing the prefixes of b2/b3/b4 is trivial, we only need to mask out the two MSBs.

```

*/* IMPL: 移除前缀 */*

*/* 移除尾部字节的前缀 */*

vlOut = __riscv_vcpop(m, vl);

b2 = __riscv_vand(b2, **0b00111111**, vlOut);

b3 = __riscv_vand(b3, **0b00111111**, vlOut);

b4 = __riscv_vand(b4, **0b00111111**, vlOut);

*/* TODO: 移除前导字节的前缀 */*

```

For b1 we need to determine how many bytes to mask-off, depending on the prefix. Fortunately, the first four bytes are enough to determine which bits to mask-off, and we could simply use a `vrgater` lookup table. On current hardware however another approach seems to be faster.

Instead of a mask, we can also use left and then immediately right shifts by the same value to remove prefix bits. The shift amounts for this can almost be calculated using a single saturating subtract 10:

There is one outlier, we can easily address using a merge:

```

*/* IMPL: 移除前导字节的前缀 */*

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

*/* IMPL: 合并到 b1234 */*

*/* 无条件地扩展和合并到 b1234 */*

vuint16m4_t b34   = __riscv_vwaddu_wv(__riscv_vwmulu(b3,  **1**<<**6**,  vlOut), b4,  vlOut);

vuint16m4_t b12   = __riscv_vwaddu_wv(__riscv_vwmulu(b1,  **1**<<**6**,  vlOut), b2,  vlOut);

vuint32m8_t b1234 = __riscv_vwaddu_wv(__riscv_vwmulu(b12, **1**<<**12**, vlOut), b34, vlOut);

*/* TODO: 计算移位量 */*

b1234 = __riscv_vsrl(b1234, __riscv_vzext_vf4(shift, vlOut), vlOut);

```

Computing the shift amount it's a bit more involved than last time, but it can be done using as follows:

This maps directly into the following code:

```

*/* IMPL: 计算移位量 */*

*/* 从 `shift` 派生所需的右移量以减少 */

** b1234 到所需的字节数 */*

shift = __riscv_vmul(__riscv_vrsub(__riscv_vssubu(shift, **2**, vlOut), **3**, vlOut), **6**, vlOut);

```

For validation, we use the method described in ["Validating UTF-8 In Less Than One Instruction Per Byte"](https://arxiv.org/abs/2010.03090). I won't go into the full detail, but I'll summarise how it works:

There are seven error patterns in a 2 byte UTF-8 sequence, they can all be detected by only looking at the first 12 bits. We use three 4-bit lookup tables that map to a bitmask of the errors matching that pattern and AND them together to determine if the was an error. The only error not detected by this are related to 3-4 byte UTF-8 characters that have the wrong amount of continuation bytes. To detect those, the last remaining bit in the bitmask of the LUTs is used to indicate a pair of continuation bytes, which combined with a few comparisons manages to detect all invalid UTF-8 sequences.

Instead of doing a three LMUL=2 vrgather, we do six LMUL=1 `vrgather`, as we don't need to cross any lanes, and this performs better because it needs to do a less complex task (see later discussion). To help with that, we define the `VRGATHER_u8m1x2` macro, which unrolls the `vrgather` for us. Before we get into the implementation we also need to define the error lookup tables, and some constants we'll use later:

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

`vuint8m2_t idx2 = __riscv_vand(v2, **0xF**, vl);`

`vuint8m2_t idx1 = __riscv_vand(s1, **0xF**, vl);`

`vuint8m2_t idx3 = __riscv_vand(s3, **0xF**, vl);`

`vuint8m2_t err1 = VRGATHER_u8m1x2(err1tbl, idx1);`

`vuint8m2_t err2 = VRGATHER_u8m1x2(err2tbl, idx2);`

`vuint8m2_t err3 = VRGATHER_u8m1x2(err3tbl, idx3);`

`vint8m2_t  errs = __riscv_vreinterpret_i8m2(`

`__riscv_vand(__riscv_vand(err1, err2, vl), err3, vl));`

`*/* TODO: 检测 3/4 字节错误 */*``` 
```

`检查 3/4 字节错误，我们检查前一个输入是否有 3 或 4 字节的字符，其后应该是两个继续字节。如果我们期望两个继续字节并且得到它们，没有错误，如果我们不期望两个继续字节并且没有得到它们，这就完美地映射到异或操作。我们利用了错误位集的最高位指示了期望的两个继续字节。将字节解释为有符号数，让我们轻松地检查 MSB 位是否设置（`x < 0`），以及是否有其他位被设置（`x > 0`）。`

最后，我们测试错误掩码是否包含错误，并使用错误代码退出函数。

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

`最后，我们不能忘记前三个字节。它们可以是全部继续字节，这将导致我们的循环忽略它们。在我们的循环开始之前，我们找到第三个 UTF-8 字符的结尾并将其传递给标量验证例程，这里我们为简单起见重用 `utf8_to_utf32_scalar`：`

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

现在到了有趣的部分，让事情变得更快。

如果我们读取了全 ASCII 向量，则可以跳过验证过程，直接扩展和存储向量。我们使用最大归约来确定是否只有 ASCII 字节，其结果我们还可以用于其他快速路径：

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

`这个快速路径需要在验证之后发生。我们只关心从 b1 和 b2 创建 b12，这使我们可以大大简化代码，从而简化了一般情况。

我们不需要为了从 b1 中移除前缀而进行位移，只有两种可能性，而且其中一种是不进行任何操作，因此掩码和适合得很好。

在扩展目标操作数之前，我们仍然不能使用掩码扩展乘法，但是我们可以使用简单的`vmerge`来在两个可能的位移值之间进行选择。由于目的地已经扩展了，因此可以使用掩码扩展加法来进行加法。

现在我们需要再次进行零扩展，然后就完成了：

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

`我将这个留给读者去理解，注意所有三字节及以下字节的 UTF-8 字符的代码点都适合于 16 个字节。

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

*/* 转换为 [000000000000aaaa|aaaaaabbbbbbbbbb]*

** 转换为      [110111bbbbbbbbbb|110110aaaaaaaaaa] */*

vuint32m8_t sur = __riscv_vsub(b1234, **0x10000**, vlOut);

sur = __riscv_vor(__riscv_vsll(sur, **16**, vlOut),

__riscv_vsrl(sur, **10**, vlOut), vlOut);

sur = __riscv_vand(sur, **0x3FF03FF**, vlOut);

sur = __riscv_vor(sur, **0xDC00D800**, vlOut);

*/* 合并 1 字节 b1234 和 2 字节 sur */*

vbool4_t m4 = __riscv_vmsgtu(b1234, **0xFFFF**, vlOut);

b1234 = __riscv_vmerge(b1234, sur, m4, vlOut);

*/* 交换 b1234 的两个字节对 */*

*/* 压缩并存储 */*

vbool2_t mOut = __riscv_vmor(__riscv_vmsne(DOWN(b1234), **0**, vlOut***2**), m2even, vlOut***2**);

b1234 = UP(__riscv_vcompress(DOWN(b1234), mOut, vlOut***2**));

size_t vlDest = __riscv_vcpop(mOut, vlOut***2**);

__riscv_vse16_v_u16m8(dest, DOWN(b1234), vlDest);

```

Finally, we need to adjust the tail handling to account for UTF-16 output. Since we want to reparse the last character, we need to decrement the destination pointer, if the last output word was a high surrogate.

```

*/* 重新解析最后一个字符 + 尾巴 */*

*if (count > tail) {*

*if ((src[0] >> 6) == 0b10) --dest;*

*while ((src[0] >> 6) == 0b10 && tail < count)*

*--src, ++tail;*

*/* 在高代理时再回退一个 */*

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
