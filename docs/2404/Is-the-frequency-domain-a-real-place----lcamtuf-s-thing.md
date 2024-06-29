<!--yml

category: 未分类

日期：2024-05-27 13:00:21

-->

# 频率域是一个真实的地方吗？- lcamtuf 的事情

> 来源：[https://lcamtuf.substack.com/p/is-the-frequency-domain-a-real-place](https://lcamtuf.substack.com/p/is-the-frequency-domain-a-real-place)

在[关于傅里叶变换的早期文章](https://lcamtuf.substack.com/p/not-so-fast-mr-fourier)中，我谈到了 *频率域* —— 一个奇妙的数学空间，复杂信号在其中被转化为正弦波的振幅和相位。频率域使我们能够执行各种信号处理技巧，这些技巧在直接观察数据的时间域中看起来几乎是不可能的。

*《蓝衣女孩》的波形图（上）和频率视图（下）。*

在那次深入探讨的结尾，我留下了一个问题没有回答：这个频率空间到底有多 *真实*？离散傅里叶变换在通信和信号处理中扮演着重要角色——但它是否揭示了宇宙的一些更深层次、看不见的真理？例如，方波到底存在吗？毕竟，变换会将它们转化为[一系列奇数谐波的正弦波](https://lcamtuf.substack.com/p/square-waves-or-non-elephant-biology)——而这种模型却能预测电子电路在现实生活中的行为。

今天，我想把傅里叶变换从神坛上推下来。可以肯定的是，正弦波在自然界中无处不在，因此这个分析工具非常适合执行许多任务。尽管如此，构造其他行为良好的频率域也是完全可能的，这些频率域遵循不同的规则——包括只有方波是真实的，其他都只是谐波。

要开始，让我们回顾一下离散余弦变换——这是离散傅里叶变换的一个简化、仅限于实数的版本。从之前的文章中，你可能还记得以下的 DCT-II 公式：

\(F_k = \sum\limits_{n=0}^{N-1} s_n \cdot cos ( \pi k { n + \frac12 \over N} )\)

操作归结为获取一系列输入值（*s[n]* —— 比如音频样本），将每个值乘以特定余弦表达式的值，然后将结果求和以获取特定频率二进制数 (*F[k]*) 的幅度读数。

这个算法的核心构造是 *cos()* 表达式，它生成一个与当前 DCT 二进制数对应频率的正弦波。这被称为 *基函数*；我们可以将其抽象并重新编写公式如下：

\(F_k = \sum\limits_{n=0}^{N-1} s_n \cdot B(k,n)\)

在这种对频率域变换的一般化表示中，*B(k, n)* 根据 *k* 和 *n* 的值返回一种 *乘法器*。软件工程师可能会觉得将 *B(k, n)* 理解为查找数组很直观。实际上，让我们计算那个数组——在数学术语中是一个 *矩阵* —— 对于 *N=16*：

*N=16 时的 DCT-II 基函数图。*

如果你记得 DCT 的操作，这个视觉应该很容易理解。第一行 *(k=0)* 对应于直流分量 — 即，一个在 0 Hz 处运行的余弦波，永远固定在 +1.00。下一行包含一个完成半个周期的余弦波，在 *k=2* 处是一个完整的周期，在 *k=3* 处是一个半周期，依此类推。

那么，如何构建一个新的基函数，将信号分成方波而不是正弦频率？稍微预示一下，答案似乎非常简单：

*N=16* 的 Walsh 矩阵。

这就是 Walsh 矩阵。它本质上由以不同速度运行的方波组成，尽管还有一些复杂的对称性。是的：每个乘数只是 +1 或 -1，因此计算归结为在输入数据中翻转一些符号，然后求和。

矩阵看起来相当微不足道，但其设计很复杂。为了捕捉所有频率和相位信息，行具有递增的 *序列性* — 每个下一行比前一行多一个符号翻转。此外，模式被精心设计以确保 *正交性* — 这种脆弱的输入-输出对称性允许在频率表示和原始时间序列数据之间无缝转换。

由于其复杂的结构，构造 Walsh 矩阵的最实用方法是从生成所谓的 Hadamard 矩阵开始。对于 *N=16*，这个中间矩阵看起来如下所示：

*N=16* 的 Hadamard 矩阵。嗯？

乍一看，绘图看起来混乱，但它只是 Walsh 布局的重新排序。例如，第 15 行移动到第 1 行，而原始的第 1 行现在位于第 8 行。但是重要的是，与 Walsh 不同，这种类似分形的模式实际上相当容易从头开始创建。

要开始 Hadamard 的运算，我们从以下微不足道的 1×1 数组开始：

\(H_0 = \begin{bmatrix} +1 \end{bmatrix}\)

然后，通过取前一步骤生成的数组 (*H[n-1]*) 并在原始尺寸的两倍网格上铺设四个副本来迭代地扩展它。前三个副本完全相同，最后一个 — 右下角 — 所有符号都翻转。这种扩展的数学表示如下：

\(H_{n} = H_{n-1} \otimes \begin{bmatrix} +1 & +1 \\ +1 & -1 \end{bmatrix}\)

这个花式运算符 (⊗) 被称为 *Kronecker 乘积*，但实际上只是一个炫耀的复制粘贴。第一个扩展 — 由四个 *H[0]* 的副本组成 — 看起来如下所示：

\(H_1 = \begin{bmatrix} \begin{array}{c | c } +1 & +1 \\ \hline +1 & -1 \end{array}\end{bmatrix} \)

另一个迭代产生四个 *H[1]* 的副本，生成这样的布局：

\(H_2 = \begin{bmatrix} \begin{array}{c c | c c} +1 & +1 & +1 & +1 \\ +1 & -1 & +1 & -1 \\ \hline +1 & +1 & -1 & -1 \\ +1 & -1 & -1 & +1 \end{array} \end{bmatrix}\)

…等等。生成的Hadamard矩阵的大小始终是*2^n×2^n*，其中*n*是我们经历的构造步骤数量。

在计算机上，可以通过以下平铺算法计算矩阵，但是我们可以使用一种巧妙的位运算技巧：事实证明，通过计算*x & y*，然后检查结果中设置的位数是偶数还是奇数，可以推断特定单元格中Hadamard函数的值。以下C代码正是如此，并在屏幕上显示了一个8×8的Hadamard矩阵：

> ```
> #include <stdint.h>
> #include <stdio.h>
> 
> #define HD_LEN_POW2 3
> #define HD_LEN      (1 << HD_LEN_POW2)
> 
> int8_t hadamard(uint32_t x, uint32_t y) {  
>   return (__builtin_popcount(x & y) % 2) ? -1 : 1;
> }
> 
> int main() {
>   for (uint32_t y = 0; y < HD_LEN; y++) {
>     for (uint32_t x = 0; x < HD_LEN; x++) printf("%+d ", hadamard(x, y));
>     putchar('\n');
>   }
> }
> ```

原则上，此矩阵足以构建频域变换。尽管如此，其频率箱的排序不直观，所以在离开之前让我们试着修复它。

要将之前展示的有序Hadamard矩阵转换为漂亮的形式，我们需要根据它们的序列性对行进行排序。

最简单的方式，在几乎每个参考实现中都可以找到，只需计算每行中符号更改的数量。尽管如此，[一位读者](https://icosahedron.website/@gaditb/112228725360149921)指出另一个巧妙的位操作技巧：对于一个具有*2^n*行的数组，Hadamard行映射可以通过将Walsh行号与其自身向右移动一个位并反转最后*n*位的顺序来计算。

这里有一个执行此操作并在屏幕上打印排序后的8×8数组的实现：

> ```
> `#include <stdint.h>
> #include <stdio.h>
> 
> #define HD_LEN_POW2 3
> #define HD_LEN      (1 << HD_LEN_POW2)
> 
> int8_t hadamard(uint32_t x, uint32_t y) {
>   return (__builtin_popcount(x & y) % 2) ? -1 : 1;
> }
> 
> uint32_t to_graycode(uint32_t val) {
>   return val ^ (val >> 1);
> }
> 
> uint32_t reverse_bits(uint32_t val, uint8_t bit_cnt) {
>   uint32_t res = 0;
>   while (bit_cnt--) { res <<= 1; res |= (val & 1); val >>= 1; }
>   return res;
> }
> 
> int8_t walsh_array[HD_LEN][HD_LEN];
> 
> void precompute_walsh() {
>   for (uint32_t y = 0; y < HD_LEN; y++) {
>     uint32_t hd_y = reverse_bits(to_graycode(y), HD_LEN_POW2);
>     for (int x = 0; x < HD_LEN; x++) walsh_array[y][x] = hadamard(x, hd_y);
>   }
> }
> 
> int main() {
>   precompute_walsh();
>   for (uint32_t y = 0; y < HD_LEN; y++) {
>     for (uint32_t x = 0; x < HD_LEN; x++) printf("%+d ", walsh_array[y][x]);
>     putchar('\n');
>   }
> }`
> ```

带有此技术，我们可以深入DCT实现并制作“离散方形变换”及其逆变换：

> ```
> ...previous code here, sans main()...
> 
> void sqft(double* out_buf, double* in_buf, uint32_t len) {
> 
>   for (uint32_t bin_no = 0; bin_no < len; bin_no++) {
>     double sum = 0;
>     for (uint32_t s_no = 0; s_no < len; s_no++)
>       sum += in_buf[s_no] * walsh_array[s_no][bin_no];
>     out_buf[bin_no] = sum;
>   }
> 
> }
> 
> void isqft(double* out_buf, double* in_buf, uint32_t len) {
> 
>   for (int s_no = 0; s_no < len; s_no++) {
>     double sum = 0;
>     for (int bin_no = 0; bin_no < len; bin_no++)
>       sum += in_buf[bin_no] * walsh_array[bin_no][s_no];
>     out_buf[s_no] = sum / len;
>   }
> 
> }
> ```

技术上，这被称为*华尔什-哈达玛变换*（WHT），但不要紧。让我们确认它是否有效：

> ```
> ...continuing from the previous snippet...
> 
> void print_buf(const char* prefix, double* buf, uint32_t len) {
>   printf("%s", prefix);
>   for (uint32_t i = 0; i < len; i++) printf(" %+.2f", buf[i]);
>   putchar('\n');
> }
> 
> int main() {
> 
>   double in[HD_LEN] = { 1, 1, 1, 1, 5, 5, 5, 5 };
>   double sq_freq[HD_LEN], out[HD_LEN];
> 
>   precompute_walsh();
>   sqft(sq_freq, in, HD_LEN);
>   isqft(out, sq_freq, HD_LEN);
> 
>   print_buf("Input :", in, HD_LEN);
>   print_buf("SQFT  :", sq_freq, HD_LEN);
>   print_buf("ISQFT :", out, HD_LEN);
> 
>   return 0;
> }
> ```

输入是一串类似方波的数字序列：1 1 1 1 5 5 5 5。从DFT或DCT的输出将会是跨越多个频率箱的多个谐波：

> ```
> `DCT   : +24.00 -10.25 -0.00 +3.60 +0.00 -2.41 -0.00 +2.04`
> ```

相比之下，由程序生成的频域表示仅显示在*F[0]*（DC）和*F[1]*中的非零分量，证实我们有一种可以将信号分解为纯方波的算法：

> ```
> `SQFT  : +24.00 -16.00 +0.00 +0.00 +0.00 +0.00 +0.00 +0.00`
> ```

最后，我们可以验证逆函数——*isqft()——*将频域转换回我们起始的内容：

> ```
> ISQFT : +1.00 +1.00 +1.00 +1.00 +5.00 +5.00 +5.00 +5.00
> ```

我找不到算法输出的可视化比较，所以我准备了自己的。这是“DARE” by Gorillaz的[🔈 11秒片段](https://lcamtuf.coredump.cx/dare.mp3)的准传统DCT频谱图：

*你老式、家庭友好的正弦波频谱图*。

…这是华尔什-哈达玛的酷炫等效图：

*方波频谱图的世界首秀*。

华尔什-哈达玛变换因其计算效率高且适用于某些数据类型而在某些领域中得到应用。重点不在于它需要更多使用；只是离散傅里叶变换并非唯一的真理。

*更多有关电子学的文章，请点击[这里](https://lcamtuf.coredump.cx/offsite.shtml)*。
