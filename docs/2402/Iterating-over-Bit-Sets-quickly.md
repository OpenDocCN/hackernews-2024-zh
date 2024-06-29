<!--yml

category: 未分类

date: 2024-05-29 13:20:31

-->

# 快速迭代位集

> 来源：[https://alexharri.com/blog/bit-set-iteration](https://alexharri.com/blog/bit-set-iteration)

<main class="css-fkkl8v">

# 快速迭代位集

2024年1月6日

欢迎来到我关于位集的两部分系列的第二部分！如果你对位操作不是很熟悉，或者不知道什么是位集，我建议你先阅读第一部分：[位集：介绍位操作](/blog/bit-sets)。

如果你了解位操作，你可以直接跳过第一部分。

[位集](https://en.wikipedia.org/wiki/Bit_array) — 也称为位数组或位向量 — 是一种高度紧凑的数据结构，用于存储一系列位。它们经常用来表示整数集合或布尔数组。

在[第一部分](/blog/bit-sets)中，我们开始编写一个`BitSet`类，并实现了一些基本方法：

```
class  BitSet  {  private words:  number[];  add(index:  number):  void;  remove(index:  number):  void;  has(index:  number):  boolean}
```

我们位集的位存储在名为`words`的`number[]`中。由于JavaScript只支持[32位整数](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Number#fixed-width_number_conversion)，每个[字](https://en.wikipedia.org/wiki/Word_(computer_architecture))存储32位（第一个字存储位1-32，第二个字存储位33-64，依此类推）。

在本文中，我们将处理`BitSet.forEach`。我们将从实现朴素方法开始，通过迭代位来看看我们能优化到什么程度。

然后我们将学习[二进制补码](https://en.wikipedia.org/wiki/Two%27s_complement)和[汉明权重](https://en.wikipedia.org/wiki/Hamming_weight)，以及如何利用它们让我们*非常快速*地迭代位集。

## 实现 BitSet.forEach

`BitSet.forEach`方法应该对每个被设置为1的位调用一个回调，带上该位的索引。

```
class  BitSet  {  forEach(callback:  (index:  number)  =>  void)  {  }}
```

首先，我们将遍历`words`中的每个字：

```
const words =  this.words;for  (let wordIndex =  0; wordIndex < words.length; wordIndex++)  {  const word = words[wordIndex];}
```

对于每个字，我们将按升序遍历位：

```
for  (let i =  0; i <  WORD_LEN; i++)  {}
```

`WORD_LEN`设为32：每个`word`中的位数

我们可以通过`(word & (1 << i)) !== 0`来检查位是否被设置：

```
const bitIsSetToOne =  (word &  (1  << i))  !==  0;
```

如果位非零，我们将想要调用带有位的索引的`callback`：

```
for  (let i =  0; i <  WORD_LEN; i++)  {  const bitIsSetToOne =  (word &  (1  << i))  !==  0;  if  (bitIsSetToOne)  {  callback(index)

             // Cannot find name 'index'.

  }}
```

我们可以这样计算位集中位的`index`：

```
const index =  (wordIndex <<  WORD_LOG)  + i;
```

`WORD_LOG`设为5：32的底2对数

表达式`wordIndex << WORD_LOG`等同于`wordIndex * WORD_LEN`，因为左移一位相当于乘以2（而`2 ** WORD_LOG`等于`WORD_LEN`）。

```
1  <<  WORD_LOG3  <<  WORD_LOG
```

因此，我们有了一个基本的实现。

```
class  BitSet  {  forEach(callback:  (index:  number)  =>  void)  {  const words =  this.words;  for  (let wordIndex =  0; wordIndex < words.length; wordIndex++)  {  const word = words[wordIndex];  for  (let i =  0; i <  WORD_LEN; i++)  {  const bitIsSetToOne =  (word &  (1  << i))  !==  0;  if  (bitIsSetToOne)  {  const index =  (wordIndex <<  WORD_LOG)  + i;  callback(index);  }  }  }  }}
```

## 优化`BitSet.forEach`

现在我们已经为`BitSet.forEach`实现了一个工作中的版本。我们能进一步优化吗？

我注意到的第一件事是，我们总是要遍历每个字中的每个位。我们可以通过一个廉价的`word === 0`检查来跳过没有设置位的字。

```
for  (let wordIndex =  0; wordIndex < words.length; wordIndex++)  {  const word = words[wordIndex];  if  (word ===  0)  continue;  }
```

对于大部分单词有一些位被设置的密集集合，这不会有太大帮助，但对于稀疏集合来说，这可以跳过大量工作。

```
[  00000000,  01000000,  00000000,  00000000,  0000001  ][  11101101,  01110001,  10110101,  11010001,  0101101  ]
```

但是，从这种优化中获得的性能提升有多显著？让我们通过运行一些基准测试来找出答案。

我们将为不同密度的位集运行我们的基准测试：

```
const densities =  [1,  0.75,  0.5,  0.25,  0.1,  0.05,  0.01,  0.001];
```

对于每个密度，我们将创建一个含有1亿位的位集。

```
const bitsetsAndDensities = densities.map((density)  =>  ({ density, bitset:  makeBitSet(100_000_000, density),}));
```

`makeBitSet`方法的实现如下：

```
function  makeBitSet(size:  number, density:  number)  {  const bitset =  new  BitSet();  for  (let i =  0; i < size; i++)  {  if  (Math.random()  < density)  { bitset.add(i);  }  }  return bitset;}
```

现在我们已经创建了我们的位集，我们将为每一个运行`forEach`方法并记录它花费的时间。

```
for  (const  { bitset, density }  of bitsetsAndDensities)  {  profile(  ()  => bitset.forEach(()  =>  {}),  (time)  =>  console.log(`${percentage(density)} density: ${time}`),  );}
```

对于未优化版本，我们得到：

```
100.0% density:     95.2 ms 75.0% density:    250.7 ms 50.0% density:    343.3 ms 25.0% density:    221.8 ms 10.0% density:    141.6 ms 1.0% density:     78.5 ms 0.1% density:     66.7 ms
```

使用`ìf (word === 0) continue;`优化后，我们得到：

```
100.0% density:     95.4 ms 75.0% density:    245.5 ms 50.0% density:    336.3 ms 25.0% density:    213.9 ms 10.0% density:    132.4 ms 1.0% density:     34.6 ms 0.1% density:      5.6 ms
```

让我们把这个放到表中并比较一下性能：

|  | 未优化（基准） | 跳过0s |
| --- | --- | --- |
| 密度 | 运行时间 | 速度 ^* | 运行时间 | 速度 ^* |
| 100.0% | 95.2 毫秒 | 1.0x | 95.4 毫秒 | 1.0x |
| 75.0% | 250.7 毫秒 | 1.0x | 245.5 毫秒 | 1.0x |
| 50.0% | 343.3 毫秒 | 1.0x | 336.3 毫秒 | 1.0x |
| 25.0% | 221.8 毫秒 | 1.0x | 213.9 毫秒 | 1.0x |
| 10.0% | 141.6 毫秒 | 1.0x | 132.4 毫秒 | 1.0x |
| 5.0% | 114.5 毫秒 | 1.0x | 95.9 毫秒 | 1.2x |
| 1.0% | 78.5 毫秒 | 1.0x | 34.6 毫秒 | 2.3x |
| 0.1% | 66.7 毫秒 | 1.0x | 5.6 毫秒 | 11.9x |

* 与基准比较的速度

对于超过10%的密度，我们观察到性能没有显著差异，但一旦密度降至≤5%，我们开始看到显著的性能提升：在1%密度时**速度提高超过2倍**，在0.1%密度时**速度提高超过10倍**。

### 跳过一半

我们可以通过跳过*每个单词*的一半（如果它全为0）来进一步优化这种方法。我们将为每个单词的一半创建位掩码：

```
export  const  WORD_FIRST_HALF_MASK  =  0x0000ffff;export  const  WORD_LATTER_HALF_MASK  =  0xffff0000;console.log(WORD_FIRST_HALF_MASK);console.log(WORD_LATTER_HALF_MASK);
```

使用它们，我们希望

+   只有在前半部分有任何设置位时，才会仅迭代1-16位，和

+   只有在后半部分有任何设置位时，才会仅迭代17-32位。

我们可以确定是否要像这样迭代一半：

```
const iterFirstHalf =  (word &  WORD_FIRST_HALF_MASK)  !==  0;const iterLatterHalf =  (word &  WORD_LATTER_HALF_MASK)  !==  0;
```

我们将用它来确定我们迭代位的范围：

```
const start = iterFirstHalf ?  0  :  WORD_LEN_HALF;const end = iterLatterHalf ?  WORD_LEN  :  WORD_LEN_HALF;for  (let i = start; i < end; i++)  {}
```

让我们看看这会带来多大的不同：

|  | 未优化（基准） | 跳过0s | 跳过0s和一半 |
| --- | --- | --- | --- |
| 密度 | 运行时间 | 速度 ^* | 运行时间 | 速度 ^* | 运行时间 | 速度 ^* |
| 100.0% | 95.2 毫秒 | 1.0x | 95.4 毫秒 | 1.0x | 99.2 毫秒 | 1.0x |
| 75.0% | 250.7 毫秒 | 1.0x | 245.5 毫秒 | 1.0x | 246.3 毫秒 | 1.0x |
| 50.0% | 343.3 毫秒 | 1.0x | 336.3 毫秒 | 1.0x | 346.3 毫秒 | 1.0x |
| 25.0% | 221.8 毫秒 | 1.0x | 213.9 毫秒 | 1.0x | 215.5 毫秒 | 1.0x |
| 10.0% | 141.6 毫秒 | 1.0x | 132.4 毫秒 | 1.0x | 133.6 毫秒 | 1.1x |
| 5.0% | 114.5 毫秒 | 1.0x | 95.9 毫秒 | 1.2x | 93.7 毫秒 | 1.2x |
| 1.0% | 78.5 毫秒 | 1.0x | 34.6 毫秒 | 2.3x | 28.5 毫秒 | 2.8x |
| 0.1% | 66.7 毫秒 | 1.0x | 5.6 毫秒 | 11.9x | 5.7 毫秒 | 11.7x |

* 与基准比较的速度

高密度集合会略微影响性能，但在大约1%的甜点密度处，我们看到了轻微的性能提升。

这种优化可能对您的平均集合密度是否值得并没有太大影响，但这并不会显著改变结果。

* * *

就在我的位集之旅中，我发现了一种不同的位迭代方法，它在所有集合密度上都有显著更好的结果。

## 补码

逐位迭代是昂贵的，并且需要在每次迭代时使用一个`if`语句来检查是否调用回调。这个`if`语句会创建一个[分支](https://johnnysswlab.com/how-branches-influence-the-performance-of-your-code-and-what-can-you-do-about-it/)，进一步降低性能。

如果我们能够以某种方式“跳转”到下一个设置位，我们将消除迭代0位并执行一堆`if`语句的需要。

事实证明，有一种非常便宜的方法可以找到最低有效位设置为1，看起来像这样：

当我第一次看到这个时，对我来说毫无意义。我想：

> *“使一个数变为负数不就是将符号位设为1吗？*
> 
> *“如果这样的话，`x & -x`就只产生`x`。”*

如果带符号整数是使用[符号-幅度](https://en.wikipedia.org/wiki/Signed_number_representations#Sign%E2%80%93magnitude)表示的，那么左边最多的位是符号位，其余位表示值（幅度）。

| 使用符号-幅度表示的数字 |
| --- |
| 正数 | 负数 |
| 位 | 值 | 位 | 值 |
| 00000000 | 0 | 10000000 | -0 |
| 00000001 | 1 | 10000001 | -1 |
| 00000010 | 2 | 10000010 | -2 |
| 00000011 | 3 | 10000011 | -3 |
| 00001001 | 9 | 10001001 | -9 |
| 01111111 | 127 | 11111111 | −127 |

但据我所学，带符号整数通常使用[二进制补码](https://en.wikipedia.org/wiki/Two%27s_complement)表示。

二进制补码不同于符号-幅度（以及[一补数](https://en.wikipedia.org/wiki/Signed_number_representations#Ones'_complement)）的是，它只有一个表示0的方式（没有-0）。

| 使用二进制补码表示的数字 |
| --- |
| 正数 | 负数 |
| 位 | 值 | 位 | 值 |
| 00000000 | 0 |  |  |
| 00000001 | 1 | 11111111 | -1 |
| 00000010 | 2 | 11111110 | -2 |
| 00000011 | 3 | 11111101 | -3 |
| 00001001 | 9 | 11110111 | -9 |
| 01111111 | 127 | 10000001 | −127 |
|  |  | 10000000 | −128 |

整数的二进制补码是通过以下计算得到的：

1.  反转位（包括符号位），然后

1.  向数字添加1。

```
00010011  1110110011101101 
```

注：这也适用于相反的方向（从负到正）

数字`19`的二进制表示中，最低有效位为1。反转使得最低有效位变为0，因此加1总是使第一位变为1。这使得`x & -x`对于任何最低有效位为1的数字都产生第1个设置位。

让我们看一个带有一些前导0的数字：

```
00110000  1100111111010000 
```

在这里，我们观察到在反转时所有最低有效位之前的位都变为1。当向数字加1时，这些1会传递，直到它们达到最低有效0位（这在反转前是最低有效1）。这使得`x & -x`对于具有前导0的任何数字都产生第1个设置位。

因此，在迭代一个字的位时，我们总是可以通过`word & -word`找到最低有效位。有趣的是，我们可以使用位异或来取消位：

```
while  (word !==  0)  {  const lsb = word &  -word; word ^= lsb;  }
```

现在我们可以遍历一个字中的置位位，但是我们遇到了一个小问题。我们希望调用回调函数时使用置位位的*索引*，而不是置位位本身。

我们将通过使用[汉明重量](https://en.wikipedia.org/wiki/Hamming_weight)来找到置位位的索引。

### 汉明重量

给定具有一个置位位的整数，我们希望能够快速找到该位的索引：

```
indexOfFirstSetBit(0b00000100)indexOfFirstSetBit(0b00000001)indexOfFirstSetBit(0b00100000)
```

暴力方法将逐位遍历位，但这样我们又回到了逐位遍历。这是行不通的。

一个观察结果是置位位的索引等于前导 0 的数量。

考虑当我们减去 1 时会发生什么。前导的 0 变成 1，并且置位位变成未设置。

这将问题从在 `x` 中找到置位位的索引转换为计算 `x - 1` 中置位位的数量。非零位的数量被称为[汉明重量](https://en.wikipedia.org/wiki/Hamming_weight)，事实证明我们可以非常便宜地[计算整数的汉明重量](https://en.wikipedia.org/wiki/Hamming_weight#Efficient_implementation)：

```
function  hammingWeight(n:  number):  number  { n -=  (n >>  1)  &  0x55555555; n =  (n &  0x33333333)  +  ((n >>>  2)  &  0x33333333);  return  (((n +  (n >>>  4))  &  0xf0f0f0f)  *  0x1010101)  >>  24;}
```

精确地说，这是如何工作的我们不深入讨论。我们只是相信这是有效的。

现在我们已经拥有了所有需要的元素。

### 使 BitSet.forEach 快速运行

如前所述，我们遍历每个字：

```
class  BitSet  {  forEach(callback:  (index:  number)  =>  void)  {  const words =  this.words;  for  (let wordIndex =  0; wordIndex < words.length; wordIndex++)  {  let word = words[wordIndex];  }  }}
```

当 `word` 非零时，我们找到最不显著的位 `lsb`：

```
while  (word !==  0)  {  const lsb = word &  -word;}
```

使用 `lsb`，我们可以使用 `lsb - 1` 的汉明重量计算索引：

```
const index =  (wordIndex <<  WORD_LOG)  +  hammingWeight(lsb -  1);callback(index);
```

在下一次迭代之前，我们通过 `word XOR lsb` 取消最低有效位，使 `word` 准备好进行下一次迭代：

完整的实现看起来像这样：

```
forEach(callback:  (index:  number)  =>  void)  {  const words =  this.words;  for  (let wordIndex =  0; wordIndex < words.length; wordIndex++)  {  let word = words[wordIndex];  while  (word !==  0)  {  const lsb = word &  -word;  const index =  (wordIndex <<  WORD_LOG)  +  hammingWeight(lsb -  1);  callback(index); word ^= lsb;  }  }}
```

但是这个优化版本有多快？让我们运行我们的基准测试并比较。

|  | 未优化（基准） | 跳过 0s | 优化 |
| --- | --- | --- | --- |
| 密度 | 运行时 | 速度 ^* | 运行时 | 速度 ^* | 运行时 | 速度 ^* |
| 100.0% | 95.2 ms | 1.0x | 95.4 ms | 1.0x | 53.4 ms | 1.8x |
| 75.0% | 250.7 ms | 1.0x | 245.5 ms | 1.0x | 91.9 ms | 2.7x |
| 50.0% | 343.3 ms | 1.0x | 336.3 ms | 1.0x | 68.4 ms | 5.0x |
| 25.0% | 221.8 ms | 1.0x | 213.9 ms | 1.0x | 44.4 ms | 5.0x |
| 10.0% | 141.6 ms | 1.0x | 132.4 ms | 1.0x | 30.1 ms | 4.7x |
| 5.0% | 114.5 ms | 1.0x | 95.9 ms | 1.2x | 24.8 ms | 4.6x |
| 1.0% | 78.5 ms | 1.0x | 34.6 ms | 2.3x | 10.0 ms | 7.8x |
| 0.1% | 66.7 ms | 1.0x | 5.6 ms | 11.9x | 4.0 ms | 16.7x |

* 与基准相比的速度

对于密度为 5-50%，我们获得了**性能提升约 5 倍**。75%及以上的较高密度获得了显著的速度提升超过 2 倍，而低于 5%的密度则看到了**性能提升 5-17 倍**。

## 完整的 BitSet 实现

我最近发布了一个性能优越且功能齐全的 `BitSet` 包 [on npm](https://www.npmjs.com/package/bitset-mut)。

如果您想探索完整的实现，请查看 [GitHub 仓库](https://github.com/alexharri/bitset-mut)。

## 进一步阅读

[Daniel Lemire](https://lemire.me/blog/) 在位集方面写了很多篇文章。他的[许多](https://lemire.me/blog/2018/02/21/iterating-over-set-bits-quickly/) [文](https://lemire.me/blog/2012/11/13/fast-sets-of-integers/) [章](https://lemire.me/blog/2019/05/03/really-fast-bitset-decoding-for-average-densities/) 都涉及到位集。他的[`FastBitSet` 实现](https://github.com/lemire/FastBitSet.js) 是我发现用于优化集合位的这一技巧的地方。如果你关注软件性能，他在这个主题上写得*很多*。

## 结语

任何给定的代码片段都可以优化，但采用不同的方法通常会超越那些局部优化。

在本文中，我们看到不同的算法通常会偏好某些输入，比如低密度集合与高密度集合。在考虑选择路径时，记住这些权衡是很有用的。如果可能的话，进行基准测试！

* * *

无论如何，感谢阅读这篇关于位集的短系列！如果你还没读过第一部分，你可以在这里找到它：[位集：位操作简介](/blog/bit-sets)。

或许我会写第三部分，深入探讨位集在布尔操作（与、或、异或、非等）中的性能，但目前为止，我对位集的思考已经足够多了。

— 亚历克斯·哈里

</main>
