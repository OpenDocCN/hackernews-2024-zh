<!--yml

类别：未分类

日期：2024-05-27 14:54:49

-->

# Manifest AI - 线性转换器终究更快

> 来源：[https://manifestai.com/blogposts/faster-after-all/](https://manifestai.com/blogposts/faster-after-all/)

<main class="content page-columns page-full" id="quarto-document-content">

\[ \newcommand{\R}{\mathbb{R}} \newcommand{\Z}{\mathbb{Z}} \newcommand{\N}{\mathbb{N}} \newcommand{\sft}{\text{softmax}} \newcommand{\List}{\text{List}} \newcommand{\Seq}{\text{Seq}} \newcommand{\SeqT}{\text{SeqT}} \newcommand{\CSeqT}{\text{CSeqT}} \newcommand{\Dist}{\text{Dist}} \newcommand{\SM}{\text{SM}} \newcommand{\Fn}{\text{Fn}} \newcommand{\Tok}{\text{Tok}} \newcommand{\Aij}{ A_{[i,j]}} \]

众所周知，将转换器的注意力层中的指数项去掉可以实现一种递归的重新表述，其计算成本与上下文长度呈线性而非二次关系[[1]](#ref-katharopoulos2020transformers)。人们期望这样的架构在训练时速度会更快，尤其是当上下文规模较大时。然而，在使用硬件加速器（例如GPU）训练深度神经网络时，FLOPs的减少并不总能直接转化为实际加速。最初的实证实验证明，基于线性转换器的大型语言模型的训练速度比经典的转换器慢，这导致许多人将这种方法视为理论上不错但在实践中无益的方法[[2]](#ref-Tay2022Efficient)。

目前，该领域的传统智慧认为，具有高度优化的硬件利用率（例如FlashAttention）的二次成本算法能够实现最佳的训练吞吐量[[3]](#ref-dao2023flashattention)。但这是错误的。在这篇文章中，我们解释了实现线性转换器的几种不同方式，并且我们展示了事实上它们可以产生巨大的速度提升。

**实验设置。** 每次计时实验都在H100上以批量大小为1和词汇量为50k进行。我们从对GPT2的确切JAX复制开始（针对[nanoGPT](https://github.com/karpathy/nanoGPT)进行了数值测试），仅修改了自注意力层。我们的实现还包括现代的附加功能，如混合精度训练。

下面的实验展示了中心要点。我们比较了一个经过优化的转换器的速度，一个直接的线性转换器的递归实现（如[[1]](#ref-katharopoulos2020transformers)中所述），以及一个经过优化的线性转换器（在第3节中描述）。我们发现，尽管在上下文大小方面的增长更好，但线性转换器在挂钟时间上训练要比转换器慢得多。相比之下，我们的算法在所有上下文和模型大小下都严格快于甚至是高度优化的FlashAttention基线。在中等规模的上下文大小下，这种速度提升比FlashAttention本身更有影响。

随着上下文大小的增加，我们利用越来越多的GPU DRAM，直到最终遇到内存不足（OOM）错误，无法再运行训练步骤。不同的算法具有不同的内存需求（例如，FlashAttention的内存使用比传统注意力显着少），因此我们测试的各种算法在此图上有不同的范围。

但速度只是问题的一部分。我们还需要知道线性变压器是否实际上像标准变压器一样最小化损失。不幸的是，如下图所示，直接将GPT2 转换为使用线性变压器层会显著损害学习能力。

**实验设置。** 这些实验每个都在一个8xH100节点上运行了24小时。对于所有运行，我们使用了flash-attention。我们使用了分块算法来选择线性变压器运行的最佳块大小（参见第3节）。所使用的数据集是c4[[4]](#ref-c4)，由[tiktoken](https://github.com/openai/tiktoken)的GPT2编码器进行标记化。我们绘制了训练损失，因为数据集足够大，所有训练都是在单纯的epoch设置下完成的，因此训练和保留损失之间没有区别。

这些结果在第5节中详细讨论，但简而言之：随着序列长度的增长，线性变压器似乎变得越来越不稳定，这对学习产生了负面影响。这意味着我们的线性变压器有点无用，因为长上下文中看到的加速带来的积极影响被学习下降的负面影响所削弱。

已经提出了其他变体的线性变压器，声称解决了这些学习问题[[5]](#ref-choromanski2020rethinking)–[[11]](#ref-yang2023gated)，但我们今天不对它们进行调查。由于我们在这篇文章中的主要动机是分享我们对如何实现高效线性变压器的见解，我们坚持使用最自然的变体，这与标准变压器最接近。在未来的文章中，我们将解释如何改进线性变压器的学习，从而使我们能够高效地训练具有超快采样的前所未有的长上下文语言模型。

## 1\. 线性变压器

变压器层的输入是查询、键和值的序列\(Q_i, K_i, V_i \in \R^d\)，其中\(i\)的范围从\(1\)到序列长度\(t\)。输出是一个序列\(Y_i\in \R^d\)。变压器层的著名公式，最早由Vaswani等人[[12]](#ref-vaswani2017attention)推广，是：\[ Y_i^\text{Transformer} = \sum_{j=1}^i e^{Q^T_i K_j} V_j \] 这里我们简化了符号，省略了softmax的分母。尽管分母提供的归一化对于良好的学习性能很重要，但对计算成本或实现速度的影响不大，而这正是我们这里的主要关注点。

尽管我们在数学表达中省略了归一化，但在我们的所有实验中，归一化都已包含在内。

*线性变换器*（LT）层的公式非常相似：只需将项 \(e^{Q^T_i K_j} \to Q^T_i K_j\) 替换为 \[ Y_i^\text{LinearTransformer} = \sum_{j=1}^i Q^T_i K_j V_j \]

我们所有关于线性变换器的实验还包括一个归一化因子，我们经验性地发现这对于学习性能很重要。借鉴 [[1]](#ref-katharopoulos2020transformers)，我们在 `softplus` 中确保和为正后，通过将键和查询置于正象限，将每个 \(Y_i\) 除以 \(\sum_{j=1}^i Q^T_i K_j\)。

这一层“线性”是指输出 \(Y\) 与所有 \(Q\), \(K\) 和 \(V\) 呈线性关系。从现在开始，我们将省略 \(Y_i^\text{LinearTransformer}\) 的上标，并只写成 \(Y_i\)。为了开始探索线性变换器的计算成本，考虑以下实现。

```
def LT_attention(Q, K, V):
 """
 Shapes of inputs are
 Q: [t, d]  K: [t, d]  V: [t, d]
 Shapes of outputs are
 Y: [t, d]
 """
 t, d = Q.shape
 Y_list = []
 for i in range(t):           # loop cost: O(t^2 d)
 Y_i = zeros(d)
 Q_i = Q[i]
 for j in range(i):       # loop cost: O(id)
 A_ij = inner(K[j], Q_i)  # cost: O(d)
 Y_i += A_ij * V[j]   # cost: O(d)
 Y_list.append(Y_i)
 return stack(Y_list)
```

*任何熟悉自注意力的人都会意识到这是一种糟糕的实现，因为嵌套的 for 循环不适合 GPU。别担心：在下一节中，我们将讨论并行化计算的实现方式，因此在现代硬件上运行速度更快。目前，此伪代码的主要目的是突出第一种计算模式，我们称之为 *注意* 表达式，其 FLOP 成本为 \(O(t^2 d)\)。

我们在介绍中看到的大规模加速的关键来自于利用线性性来重组计算。考虑以下因式分解：\[ Y_i = \sum_{j=1}^i Q^T_i K_j V_j = \underbrace{ \left ( \sum_{j=1}^i V_j K_j^T\right )}_{S_i} \; \; Q_i \] 用这种形式写出来，我们注意到标记为 \(S_i \in \R^{d\times d}\) 的术语可以被认为是一个总结到时间 \(i\) 的所有相关信息的状态。我们可以很容易地重写成以下递归方程\[ Y_{i} = S_i Q_i \;\;\;\;\;\;\;\;\;\;\;\; S_i = S_{i-1} + V_i K_i^T \] 其中我们假设 \(S_{0} = 0\in \R^{d\times d}\)。用这种形式写出来，我们意识到线性变换器是一个 RNN。我们还可以为这种方法编写伪代码，我们称之为 *状态* 表达式，并分析成本：

```
def LT_state(Q, K, V):
 """
 Shapes of inputs are
 Q: [t, d]  K: [t, d]  V: [t, d]
 Shapes of outputs are
 Y: [t, d]
 """
 t, d = Q.shape
 S_i = zeros(d, d) # shape [d,d]
 Y_list = []
 for i in range(t):        # loop cost: O(t d^2)
 S_i += outer(K[i], V[i]) # cost: O(d^2)
 Y_i = S_i @ Q[i]      # cost: O(d^2)
 Y_list.append(Y_i)
 return stack(Y_list)
```

*我们看到这里的成本是 \(O(t d^2)\)。

因此，标准的变换器层总成本为 \(O(t^2 d)\)，而线性变换器有两种不同成本的表达式。通过从注意力表达式切换到状态表达式，我们可以将成本从 \(O(t^2 d)\) 更改为 \(O(t d^2)\)，以一个 \(t\) 项换取一个 \(d\) 项。**  **## 2\. 并行实现

一般来说，对于在 GPU 上运行的任何东西，使用 for 循环都是一种糟糕的实现方式。当使用像 PyTorch 或 JAX 这样的高级框架时，实现高 GPU 利用率的最简单方法是仅使用已经针对 GPU 进行高度优化的原语：矩阵乘法，逐元素操作等。我们可以将我们的注意力和状态算法重写为这种风格，使它们变得高效。

首先，让我们为注意力做这件事。我们的主要技术是使用一个重量级矩阵乘法计算包含了`LT_attention`中的for循环中出现的所有项 `outer(Q[i], K[j])` 的注意力矩阵 \(A\)。

```
def LT_attention_parallel_no_flash(Q, K, V):
 """
 Shapes of inputs are
 Q: [t, d]  K: [t, d]  V: [t, d]
 Shapes of outputs are
 Y: [t, d]
 """
 t = Q.shape[0]
 M = causal_mask(t)
 A_raw = Q @ K.T  # cost O(t^2 d)
 A = A_raw * M    # cost O(t^2)
 Y = A @ V        # cost O(t^2 d)
 return Y
```

*这个实现是过去五年里几乎每个变压器的核心，它驱动着最近爆炸式流行的所有AI模型。由于它由少量超并行操作组成，它获得了高GPU利用率。最近，专门的 *闪光注意力* 内核已被用于通过避免显式存储注意力矩阵 \(A\) 来获得更进一步的加速，从而节省了内存和内存传输上花费的时间。从算法上讲，闪光注意力是并行注意力的一个变种，具有相同的计算成本（以FLOP计算）。我们使用 `LT_attention_parallel` 来指代闪光注意力实现。

接下来，让我们并行化递归公式。这个优化不太为人所知。关键是并行计算所有项 \(V_i K^T_i\)，然后使用可以[并行化](https://en.wikipedia.org/wiki/Prefix_sum)的累积和来组合它们。

```
def LT_state_parallel(Q, K, V):
 """
 Shapes of inputs are
 Q: [t, d]  K: [t, d]  V: [t, d]
 Shapes of outputs are
 Y: [t, d]
 """
 P = V[:,:,None] @ K[:,None,:]  # cost: O(t d^2)
 S = cumsum(P, axis=0)          # cost: O(log_2(t) t d^2)
 Y = S @ Q[:,:,None]            # cost: O(t d^2)
 return Y[:,:0]
```

*该算法的FLOP成本为 \(O(\log_2(t) t d^2)\)。

现在我们有了四个数学上等价的线性变压器实现，让我们计时我们的GPT2模型的训练步骤，并看看它有多大的差异。对于我们的`LT_attention_parallel`实现，我们使用了我们在Triton中实现的自定义线性自注意闪光核，基于[OpenAI的FlashAttention2实现](https://triton-lang.org/main/getting-started/tutorials/06-fused-attention.html)。

这里有一些要点：

+   预料之中，所有的`attention`变种都具有二次渐近成本（在对数-对数图上的斜率为2）。所有的`state`变种都具有线性渐近成本（斜率为1）。

+   `LT_state_parallel`比`LT_state`快一个数量级。

+   `LT_attention_parallel_no_flash`比`LT_attention`快两个数量级。

+   `LT_attention_parallel`似乎渐近地稳定为比`LT_attention_parallel_no_flash`快一个数量级。

+   对于大多数设置，`LT_attention_parallel`是最快的。（这是标准变压器使用的算法的线性版本。）

+   并行注意力是小上下文尺寸的最快算法。然而，在约13k上下文尺寸处，`LT_state_parallel`克服了`LT_attention_parallel_no_flash`，并在约100k处克服了`LT_attention_parallel`。

总的来说，这些结果描绘了一个清晰的图景：对于小上下文使用注意力算法，对于大上下文使用状态算法。但我们真的面临二元选择吗？我们能否将状态和注意力的思想结合起来，得到两者的最佳效果？**  **## 3\. 分块表述

很明显，对于较小的上下文大小，计算 \(t\) 乘以 \(t\) 的注意力矩阵要比计算许多 \(d\) 乘以 \(d\) 的状态矩阵更有效率。但是当 \(t\) 增大时，注意力矩阵的二次成本开始占主导地位。注意到对于较小的 \(t\)，注意力是非常高效的，而对于较大的 \(t\)，状态是必要的，这促使我们对 LT 方程进行最后一次改写。

让 \(c \in \N\) 为一个正整数，我们称之为 *分块大小*。对于任何 \(i\in \N\)，找到唯一的 \(n\in \Z\) 使得 \(cn < i \le c(n+1)\)。我们可以轻松地看到以下方程与之前的方程等价。 \[ Y_{i} = S_{cn}Q_i + \sum_{j=cn+1}^i Q_i^T K_j V_j \;\;\;\;\;\;\;\;\;\;\;\; S_{c(n+1)} = S_{cn} + \sum_{j=cn+1}^{c(n+1)} V_j K_j^T \] 关键思想是我们只计算一部分所有状态：\(S_0, S_c, S_{2c}, \cdots\)。然后，为了计算每个输出 \(Y_i\)，我们只需要考虑最近状态 \(S_{cn}\) 的贡献，以及范围 \(cn < j \le i\) 中所有时刻 \(j\) 的贡献（通过注意力计算）。

作为伪代码，看起来是这样的：

```
def LT_attention_with_initial_state(S, Q, K, V):
 """
 Shapes of inputs are
 S: [d, d]  Q: [c, d]  K: [c, d]  V: [c, d]
 Shapes of outputs are
 Y: [c, d]
 """
 Y_state = Q @ S                               # cost O(c d^2)
 Y_attention = LT_attention_parallel(Q, K, V)  # cost O(c^2 d)
 Y = Y_state + Y_attention                     # cost O(cd)
 return Y

def LT_chunked(Q, K, V, c):
 """
 Shapes of inputs are
 Q: [t, d]  K: [t, d]  V: [t, d], c: int
 Shapes of outputs are
 Y: [t, d]
 """
 t, d = Q.shape
 assert t % c == 0
 Q_, K_, V_ = [arr.reshape(t//c, c, d)
 `               for arr in [Q,K,V]]
 P_ = K_.transpose([0,2,1]) @ V_  # cost O(t d^2)
 S_ = cumsum(P_, axis=0) - P_     # cost O(log_2(t/c)(t/c)d^2)
 Y_ = vmap(LT_attention_with_initial_state, axis=0)(
 S_, Q_, K_, V_)      # cost O(td^2 + tcd)
 return Y_.reshape(t, d)
```

*成本为 \(O\left(td^2 + tcd + \log_2(t/c)(t/c)d^2\right)\)，再次避免对 \(t\) 的二次依赖。此外，请注意，该算法内部调用了 `LT_attention_parallel`，因此我们可以使用快速注意力内核来执行计算。

这个算法有一个超参数，即分块大小，必须正确设置以获得最佳性能。幸运的是，通过经验法确定最佳分块大小并不昂贵，因为只需测量每个分块大小的几个训练步骤就足以确定哪个是最快的。在下面的图中，我们展示了在每种设置下最佳分块算法的速度。

我们看到 `LT_chunked` 提供了所期望的两全其美的行为，因为在所有设置中它都等同或优于其他所有方法。现在我们看到与标准自注意相比的大幅度（并且快速增长）的加速度，最终释放了线性变换器的真正力量。

当处理语言模型时，高效训练并非唯一值得考虑的性能。一旦模型训练完毕，利用它的成本有多高？当我们进行采样时，我们有一个标记序列 \(z_1 \cdots z_t\)，我们想要采样下一个标记 \(z_{t+1}\)。

从传统变压器中抽样的最有效算法称为*KV-cache*算法[[14]](#ref-pope2023efficiently)。该算法假定当我们生成令牌\(z_{t+1}\)时，我们已经计算并缓存了所有\(K_i, V_i\)对于所有\(0 \le i \le t\)的情况。为了在给定这些缓存信息的情况下计算时间\(t+1\)的注意力层的输出，我们可以使用\[ Y_{t+1}^\text{Transformer} = \sum_{j=1}^{t+1} e^{Q^T_i K_j} V_j \] 很容易看出这是一个\(O(td)\)操作。换句话说，随着序列长度的增加，抽样每个后续令牌变得更加计算昂贵。这是传统变压器架构的主要限制之一。

然而，使用线性变压器，我们可以访问循环公式。线性变压器是一个状态大小为\(O(d^2)\)的RNN。\[ Y_{i} = S_i Q_i \;\;\;\;\;\;\;\;\;\;\;\; S_i = S_{i-1} + V_i K_i^T \] 我们可以比较在抽样序列时生成任何特定令牌所需的时间：

正如预期的那样，我们看到线性变压器的抽样时间与上下文长度无关，而KV-cache变压器的抽样时间呈线性增长。这导致在大的上下文中存在很大的推理速度差距。

## 5\. 学习性能

到目前为止，我们的重点一直放在如何有效实现线性变压器上。但是，一个运行速度很快的架构如果不学习得好，那么它就是无用的。我们现在将比较GPT2基线与其线性变压器对应物在各种上下文大小下的学习曲线。

为了控制更长的上下文通过引入更多的令牌每次更新帮助学习的事实，我们通过减小批处理大小来保持跨上下文大小的令牌数每次更新恒定。在所有上下文长度中，每次更新看到\(2^{19}\)个令牌。重要的是，对于这组实验，我们使用的是数据集c4 [[4]](#ref-c4)，它没有太多的长期结构：它由平均长度约为500个令牌的短文章的随机收集组成。

首先，让我们来看看GPT2和我们修改后的Linear-GPT2的参数缩放规律。下图显示了在我们的8xH100服务器上训练24小时后的每个规模和每个上下文大小的最终性能。如果您想查看完整的学习曲线，请单击第二个选项卡。

两种架构在模型大小方面的扩展类似，但在性能上存在差距。随着上下文大小的增加，这种差距似乎会加大，同时出现更多的损失峰值和一般不稳定性。可能是由于线性变压器未能有效利用其上下文，导致这种差距。为了探究这是否为真，我们可以使用相同的实验，但将所有上下文长度一起可视化。

我们看到两种架构之间存在明显不同的趋势。对于GPT2，上下文长度的增加会减慢学习的初始速度，但最终所有上下文长度（至少超过1024）都会收敛到类似的最终损失。而对于线性-GPT2，增加上下文不仅会在开始时减慢学习速度，而且还会导致最终性能更差和不良的损失峰值。

对于GPT2的结果符合我们在给定情境下所期望的健康模型。请注意，由于我们的数据集由短文章组成，平均长度约为~500个标记，我们可以假设即使是1024的上下文窗口也将包含几乎所有相关内容。因此，我们不会预期增加上下文长度超过这一点将减少模型最终收敛到的损失。然而，长上下文模型确实需要学会*忽略*更多不相关的标记，从而解释了初始学习的减缓。

相比之下，线性-GPT2的结果似乎表明线性变压器架构存在某种潜在的不稳定性。它甚至似乎无法忽略不相关的信息，更不用说利用有用的长期结构了。

修复这些学习问题将成为我们未来在线性变压器领域的主要工作重点。我们目前的架构基本上是GPT2的一对一克隆，尽可能地保持与原始架构的一致性；诸如初始化、规范化和超参数等方面直接复制。众所周知，这些决策可能对扩展和稳定性产生重大影响，并且通常需要以架构特定的方式进行调整。在文献中，还提出了各种其他线性变体的自注意力机制，这些变体融合了诸如门控机制之类的技术，以改善稳定性和学习[[5]](#ref-choromanski2020rethinking)–[[11]](#ref-yang2023gated)。未来的文章将包括对所有这些选择的影响进行彻底研究。

最终，任何线性变压器都可能无法完全匹配经典变压器基线的上下文缩放规律。虽然去除指数似乎是一个较小的变化，但它代表了表达能力的有意义减少。但正如我们所展示的，线性变压器可以训练得多个数量级更有效。在等效计算的情况下，线性变压器可以训练更多步骤；或者，保持步骤不变，我们可以训练一个更大的模型。如果这种额外的缩放足以补偿降低的表达能力，线性变压器将成为更高效的架构。

## 致谢

我们要感谢：Jono Ridgway 协助准备发布；Eric Alcaide 介绍了关联扫描算法；Jannis Fengler 致力于工具的开发，以确认我们的 JAX GPT2 在数值上复制了 NanoGPT；Joel Einbinder 和 Aaron Mondal 协助建立我们的工程堆栈；Warfa Jibril、Tony Pezzullo 和 Desh Raj 对博客文章草稿提供了反馈；Sander Dieleman 指导我们阅读了一些早期的线性变换器文献。…

## 参考文献

[1]

A. Katharopoulos，A. Vyas，N. Pappas 和 F. Fleuret，“变换器即 RNN：具有线性注意力的快速自回归变换器”，在 *国际机器学习会议* 中，PMLR，2020 年，第 5156-5165 页。

[2]

Y. Tay，M. Dehghani，D. Bahri 和 D. Metzler，

“高效变换器：一项调查”，*ACM 计算调查*

，第 55 卷，第 6 期，第 1-28 页，2022 年，doi:

[10.1145/3530811](https://doi.org/10.1145/3530811)

。

[3]

T. Dao，“Flashattention-2：更好的并行性和工作分区的更快注意力”，*arXiv 预印本 arXiv:2307.08691*，2023 年。

[4]

C. Raffel *等*，“通过统一的文本到文本变换器探索迁移学习的极限”，*机器学习研究杂志*，第 21 卷，第 1 期，第 5485-5551 页，2020 年。

[5]

K. Choromanski *等*，“通过表演者重新思考注意力”，*arXiv 预印本 arXiv:2009.14794*，2020 年。

[6]

Z. Qin *等*，“Cosformer: 重新思考注意力机制中的 softmax”，*arXiv 预印本 arXiv:2202.08791*，2022。

[7]

H. Peng，N. Pappas，D. Yogatama，R. Schwartz，N. A. Smith 和 L. Kong，“随机特征注意力”，*arXiv 预印本 arXiv:2103.02143*，2021 年。

[8]

Z. Qin *等*，“线性变换器中的魔鬼”，*arXiv 预印本 arXiv:2210.10340*，2022 年。

[9]

W. Hua，Z. Dai，H. Liu 和 Q. Le，“线性时间中的变换器质量”，在 *国际机器学习会议* 中，PMLR，2022 年，第 9099-9117 页。

[10]

Y. Sun *等*，“保留网络：大型语言模型的变换器继任者”，*arXiv 预印本 arXiv:2307.08621*，2023 年。

[11]

S. Yang，B. Wang，Y. Shen，R. Panda 和 Y. Kim， “硬件高效训练的门控线性注意力变换器”，*arXiv 预印本 arXiv:2312.06635*，2023。

[12]

A. Vaswani *等*， “注意力就是一切”，在 *神经信息处理系统的进展* 中，第 30 卷，2017 年。

[13]

P. Tillet，H.-T. Kung 和 D. Cox，“Triton：用于瓦片式神经网络计算的中间语言和编译器”，在 *第三届 ACM SIGPLAN 国际机器学习和编程语言研讨会论文集* 中，2019 年，第 10-19 页。

[14]

R. Pope *等*， “高效扩展变换器推理”，在 *机器学习和系统的论文集* 中，第 5 卷，2023 年。

[15]

Z. Shen，M. Zhang，H. Zhao，S. Yi 和 H. Li，“高效注意力：具有线性复杂性的注意力”，在 *IEEE/CVF 冬季计算机视觉应用会议论文集* 中，2021 年，第 3531-3539 页。

[16]

A. de Brébisson 和 P. Vincent，"一个廉价的线性注意力机制，具有快速查找和固定大小表示"，*arXiv预印本 arXiv:1609.05866*，2016年。

## 引用

BibTeX 引用：

```
@misc{buckman2024,
  author = {Buckman, Jacob and Gelada, Carles},
  publisher = {Manifest AI},
  title = {Linear {Transformers} {Are} {Faster} {After} {All}},
  date = {2024-01-05},
  langid = {en}
} 
```

*为了归属，请将这项工作引用为：*

*J. Buckman 和 C. Gelada，"线性变换器终究更快。" Manifest AI，2024年1月05日。*
