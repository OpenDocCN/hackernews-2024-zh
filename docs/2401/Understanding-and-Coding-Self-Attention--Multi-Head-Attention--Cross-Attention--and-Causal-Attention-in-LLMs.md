<!--yml

类别：未分类

日期：2024-05-27 14:46:43

-->

# 理解和编写LLMs中的自注意力、多头注意力、交叉注意力和因果注意力

> 来源：[https://magazine.sebastianraschka.com/p/understanding-and-coding-self-attention](https://magazine.sebastianraschka.com/p/understanding-and-coding-self-attention)

本文将教你关于变压器架构和大型语言模型（LLMs）中使用的自注意力机制及其相关机制。自注意力和相关机制是LLMs的核心组件，当与这些模型一起工作时，了解它们将非常有用。

然而，我们不仅仅讨论自注意力机制，还将从头开始用 Python 和 PyTorch 编写它。在我看来，从零开始编写算法、模型和技术是学习的绝佳方式！

* * *

顺便提一下，本文是我在几乎一年前在我的旧博客上发表的现代化和扩展版的 “[从零开始理解和编写大型语言模型的自注意力机制](https://sebastianraschka.com/blog/2023/self-attention-from-scratch.html)” 。由于我非常喜欢写（和阅读）“从零开始”的文章，我想将这篇文章现代化为*Ahead of AI*。

此外，这篇文章激发了我写书*[从零开始构建大型语言模型](http://mng.bz/amjo)*的灵感，该书目前正在进行中。下面是一种总结该书并说明自注意力机制如何融入整体思路的思维模型。

为了使本文长度合理，我假设你已经了解了LLMs，也对基本水平的注意力机制有所了解。本文的目标和重点是通过Python和PyTorch代码演示了解注意力机制的工作原理。

自注意力机制通过原始的变压器论文（[Attention Is All You Need](https://arxiv.org/abs/1706.03762)）的介绍而得到引入，已经成为许多最先进的深度学习模型的基石，尤其是在自然语言处理（NLP）领域。由于自注意力现在无处不在，了解它的工作原理就显得尤为重要。

在深度学习中，“注意力”这个概念 [源于改进递归神经网络（RNNs）的努力](https://arxiv.org/abs/1409.0473) 来处理更长的序列或句子。例如，考虑将一种语言的句子翻译成另一种语言。逐字翻译一句话通常不可行，因为它忽略了每种语言独特的复杂语法结构和习语表达，导致不准确或荒谬的翻译。

错误的逐字逐句翻译（上）与正确翻译（下）的对比

为了克服这个问题，引入了注意力机制，使每个时间步骤都能访问所有序列元素。关键是要选择性地确定在特定上下文中哪些单词最重要。[2017年，变压器架构](https://arxiv.org/abs/1706.03762)引入了一个独立的自注意力机制，完全消除了对循环神经网络的需求。

（为了简洁起见，并且让文章集中讨论技术性自注意力的细节，我将这个背景动机部分保持简短，以便我们可以专注于代码实现。）

来自“注意力机制就是一切”的论文（[https://arxiv.org/abs/1706.03762](https://arxiv.org/abs/1706.03762)）的可视化，显示了单词“making”在输入中通过注意力权重依赖或关注其他单词的程度（颜色强度与注意力权重值成比例）。

我们可以将自注意力看作是一种通过包含有关输入上下文的信息来增强输入嵌入信息内容的机制。换句话说，自注意力机制使模型能够衡量输入序列中不同元素的重要性并动态调整它们对输出的影响。这对于语言处理任务尤其重要，因为单词的含义可以根据其在句子或文档中的上下文而改变。

请注意，自注意力有许多变体。特别关注的是使自注意力更高效。然而，大多数论文仍然实现了[Attention Is All You Need论文](https://arxiv.org/abs/1706.03762)中介绍的原始缩放点积注意力机制，因为自注意力对于大多数公司训练大规模变压器来说很少成为计算瓶颈。

因此，在本文中，我们将专注于原始的缩放点积注意力机制（称为自注意力），它仍然是实践中最流行和最广泛使用的注意力机制。但是，如果您对其他类型的注意力机制感兴趣，请查看[2020年](https://arxiv.org/abs/2009.06732) *[高效变压器：一项调查](https://arxiv.org/abs/2009.06732)*，[2023年](https://arxiv.org/abs/2302.01107) *[关于变压器高效训练的调查](https://arxiv.org/abs/2302.01107)* 综述以及最近的[FlashAttention](https://arxiv.org/abs/2205.14135)和[FlashAttention-v2](https://arxiv.org/abs/2307.08691) 论文。

在开始之前，让我们考虑一个输入句子*"Life is short, eat dessert first"*，我们想要通过自注意力机制处理。与处理文本的其他类型的建模方法类似（例如，使用循环神经网络或卷积神经网络），我们首先创建一个句子嵌入。

为简单起见，在这里我们的字典 `dc` 限定为出现在输入句子中的单词。在实际应用中，我们考虑训练数据集中的所有单词（典型的词汇量介于 3 万到 5 万个条目之间）。

**输入：**

```
sentence = 'Life is short, eat dessert first'
​
dc = {s:i for i,s 
      in enumerate(sorted(sentence.replace(',', '').split()))}

print(dc)
```

**输出：**

```
{'Life': 0, 'dessert': 1, 'eat': 2, 'first': 3, 'is': 4, 'short': 5}
```

接下来，我们使用这个字典给每个单词分配一个整数索引：

**输入：**

```
import torch
​
sentence_int = torch.tensor(
    [dc[s] for s in sentence.replace(',', '').split()]
)
print(sentence_int)
```

**输出：**

```
tensor([0, 4, 5, 2, 1, 3])
```

现在，利用输入句子的整数向量表示，我们可以使用嵌入层将输入编码为实向量嵌入。在这里，我们将使用一个小的 3 维嵌入，这样每个输入单词都用一个 3 维向量表示。

注意，嵌入大小通常范围从数百到数千维。例如，Llama 2 使用 4,096 维的嵌入。这里使用 3 维嵌入纯粹是为了说明的目的。这样我们就可以检查单个向量，而不会填满整个页面。

由于该句子由 6 个单词组成，这将导致一个 6×3 维的嵌入：

**输入：**

```
vocab_size = 50_000
​
torch.manual_seed(123)
embed = torch.nn.Embedding(vocab_size, 3)
embedded_sentence = embed(sentence_int).detach()
​
print(embedded_sentence)
print(embedded_sentence.shape)
```

**输出：**

```
tensor([[ 0.3374, -0.1778, -0.3035],
        [ 0.1794,  1.8951,  0.4954],
        [ 0.2692, -0.0770, -1.0205],
        [-0.2196, -0.3792,  0.7671],
        [-0.5880,  0.3486,  0.6603],
        [-1.1925,  0.6984, -1.4097]])
torch.Size([6, 3])
```

现在，让我们讨论广泛使用的自注意机制，即称为缩放点积注意力，它是变压器结构的一个组成部分。

自注意力利用三个权重矩阵，称为 ***W[q]***、***W[k]*** 和 ***W[v]***，这些矩阵在训练过程中作为模型参数进行调整。这些矩阵用于将输入投影为序列的 *查询*、*键* 和 *值* 组成部分。

通过权重矩阵 ***W*** 和嵌入输入 ***x*** 之间的矩阵乘法来获取相应的查询、键和值序列：

+   查询序列: ***q^((i))*** = ***x^((i))W[q]*** 对于序列 *1 … T*

+   键序列: ***k^((i))*** = ***x^((i))W[k]*** 对于序列 *1 … T*

+   值序列: ***v^((i))*** = ***x^((i))W[v]*** 对于序列 *1 … T*

索引 *i* 指的是输入序列中的标记索引位置，其长度为 *T*。

通过输入 ***x*** 和权重 ***W*** 计算查询、键和值向量。

在这里，***q^((i))*** 和 ***k^((i))*** 都是 ***d[k]*** 维向量。投影矩阵 ***W[q]*** 和 ***W[k]*** 的形状为 ***d*** × ***d[k]*** ，而 ***W[v]*** 的形状为 ***d*** × ***d[v]***。

（重要的是要注意，***d*** 表示每个单词向量的大小，***x***。）

因为我们计算查询和键向量的点积，所以这两个向量必须包含相同数量的元素（***d[q] = d[k]***）。在许多 LLMs 中，我们为值向量使用相同的大小，使得 ***d[q] = d[k] = d[v]***。然而，决定生成的上下文向量的大小的值向量 ***v^((i))*** 的元素数量可以是任意的。

因此，在下面的代码演示中，我们将设置 ***d[q] = d[k] = 2***，并使用 ***d[v] = 4***，初始化投影矩阵如下：

**输入：**

```
torch.manual_seed(123)
​
d = embedded_sentence.shape[1]
​
d_q, d_k, d_v = 2, 2, 4
​
W_query = torch.nn.Parameter(torch.rand(d, d_q))
W_key = torch.nn.Parameter(torch.rand(d, d_k))
W_value = torch.nn.Parameter(torch.rand(d, d_v))
```

（与之前的词嵌入向量类似，维度 ***d[q]***、***d[k]***、***d[v]*** 通常要大得多，但这里我们使用小数以进行说明。）

现在，假设我们有兴趣计算第二个输入元素的注意力向量 -- 第二个输入元素在这里充当查询：

对于下面的各节，我们重点关注第二个输入，***x***(2)。

在代码中，这看起来如下所示：

**In:**

```
x_2 = embedded_sentence[1]
query_2 = x_2 @ W_query
key_2 = x_2 @ W_key
value_2 = x_2 @ W_value
​
print(query_2.shape)
print(key_2.shape)
print(value_2.shape)
```

**Out:**

```
torch.Size([2])
torch.Size([2])
torch.Size([4])
```

我们随后可以将此推广为计算所有输入的其余键和值元素，因为我们稍后在计算未标准化注意力权重时会需要它们的帮助：

**In:**

```
keys = embedded_sentence @ W_key
values = embedded_sentence @ W_value
​
print("keys.shape:", keys.shape)
print("values.shape:", values.shape)
```

**Out:**

```
keys.shape: torch.Size([6, 2])
values.shape: torch.Size([6, 4])
```

现在我们已经拥有所有所需的键和值，我们可以继续下一步并计算未标准化的注意力权重 ***ω***（omega），如下图所示：

计算未标准化的注意力权重 ***ω***（omega）

如上图所示，我们将 ***ω[i,j]*** 计算为查询和键序列之间的点积，***ω[i,j]** =*  ***q^((i))***  ***k^((j))***。

例如，我们可以计算查询和第 5 个输入元素（对应索引位置 4）的未标准化注意力权重如下：

**In:**

```
omega_24 = query_2.dot(keys[4])
print(omega_24)
```

（请注意，***ω*** 是希腊字母“omega”的符号，因此上面的代码变量与之同名。）

**Out:**

```
tensor(1.2903)
```

由于我们稍后需要这些未标准化的注意力权重 ***ω*** 来计算实际的注意力权重，让我们计算所有输入令牌的 ***ω*** 值，如上图所示：

**In:**

```
omega_2 = query_2 @ keys.T
print(omega_2)
```

**Out:**

```
tensor([-0.6004,  3.4707, -1.5023,  0.4991,  1.2903, -1.3374])
```

自注意力中的后续步骤是将未标准化的注意力权重 ***ω*** 规范化为通过 softmax 函数获得的标准化的注意力权重 ***α***（alpha）。此外，在通过 softmax 函数规范化之前，使用 1/√{***d[k]***} 对 ***ω*** 进行缩放，如下所示：

计算标准化注意力权重 ***α***

通过 ***d[k]*** 的缩放确保权重向量的欧氏长度大致相同。这有助于防止注意力权重变得太小或太大，这可能导致数值不稳定或影响模型在训练过程中的收敛能力。

在代码中，我们可以实现注意力权重的计算如下：

**In:**

```
import torch.nn.functional as F
​
attention_weights_2 = F.softmax(omega_2 / d_k**0.5, dim=0)
print(attention_weights_2)
```

**Out:**

```
tensor([0.0386, 0.6870, 0.0204, 0.0840, 0.1470, 0.0229])
```

最后，最后一步是计算上下文向量 ***z^((2))***，它是我们原始查询输入 ***x^((2))*** 的一个加权版本，通过注意力权重包括所有其他输入元素作为其上下文：

注意力权重是特定于某个输入元素的。这里，我们选择了输入元素 ***x**(2)***。

在代码中，这看起来如下所示：

**In:**

```
context_vector_2 = attention_weights_2 @ values
​
print(context_vector_2.shape)
print(context_vector_2)
```

**Out:**

```
torch.Size([4])
tensor([0.5313, 1.3607, 0.7891, 1.3110])
```

请注意，这个输出向量的维度比原始输入向量更多（***d[v] = 4***），因为我们之前指定了 ***d[v] > d***；然而，嵌入大小选择 ***d[v]*** 是任意的。

现在，为了结束前面章节中自注意力机制的代码实现，我们可以将前面的代码总结为一个紧凑的`SelfAttention`类：

**In:**

```
import torch.nn as nn
​
class SelfAttention(nn.Module):
​
    def __init__(self, d_in, d_out_kq, d_out_v):
        super().__init__()
        self.d_out_kq = d_out_kq
        self.W_query = nn.Parameter(torch.rand(d_in, d_out_kq))
        self.W_key   = nn.Parameter(torch.rand(d_in, d_out_kq))
        self.W_value = nn.Parameter(torch.rand(d_in, d_out_v))
​
    def forward(self, x):
        keys = x @ self.W_key
        queries = x @ self.W_query
        values = x @ self.W_value

        attn_scores = queries @ keys.T  # unnormalized attention weights    
        attn_weights = torch.softmax(
            attn_scores / self.d_out_kq**0.5, dim=-1
        )

        context_vec = attn_weights @ values
        return context_vec
```

遵循PyTorch的约定，上面的`SelfAttention`类在`__init__`方法中初始化自注意力参数，并通过`forward`方法为所有输入计算注意力权重和上下文向量。我们可以如下使用这个类：

**In:**

```
torch.manual_seed(123)
​
# reduce d_out_v from 4 to 1, because we have 4 heads
d_in, d_out_kq, d_out_v = 3, 2, 4
​
sa = SelfAttention(d_in, d_out_kq, d_out_v)
print(sa(embedded_sentence))
```

**Out:**

```
tensor([[-0.1564,  0.1028, -0.0763, -0.0764],
        [ 0.5313,  1.3607,  0.7891,  1.3110],
        [-0.3542, -0.1234, -0.2627, -0.3706],
        [ 0.0071,  0.3345,  0.0969,  0.1998],
        [ 0.1008,  0.4780,  0.2021,  0.3674],
        [-0.5296, -0.2799, -0.4107, -0.6006]], grad_fn=<MmBackward0>)
```

如果你看第二行，你会发现它与前一节中`context_vector_2`中的值完全匹配：`tensor([0.5313, 1.3607, 0.7891, 1.3110])`。

在本文顶部的第一幅图中（下面再次显示以方便查看），我们看到transformer使用了一个名为*multi-head attention*的模块。

这个"多头"注意力模块如何与我们上面介绍的自注意力机制（缩放点积注意力）相关？

在缩放点积注意力中，输入序列使用表示查询、键和值的三个矩阵进行转换。在多头注意力的上下文中，这三个矩阵可以被认为是一个单独的注意力头。下图总结了我们之前涵盖并实现的这个单个注意力头：

总结之前实现的自注意力机制

正如其名称所示，多头注意力涉及多个这样的头，每个头包含查询、键和值矩阵。这个概念类似于卷积神经网络中使用多个核生成具有多个输出通道的特征图。

多头注意力：带有多个头的自注意力

为了在代码中说明这一点，我们可以为之前的`SelfAttention`类编写一个`MultiHeadAttentionWrapper`类：

```
class MultiHeadAttentionWrapper(nn.Module):
​
    def __init__(self, d_in, d_out_kq, d_out_v, num_heads):
        super().__init__()
        self.heads = nn.ModuleList(
            [SelfAttention(d_in, d_out_kq, d_out_v) 
             for _ in range(num_heads)]
        )
​
    def forward(self, x):
        return torch.cat([head(x) for head in self.heads], dim=-1)
```

`d_*`参数与`SelfAttention`类中以前的相同--这里唯一的新输入参数是注意力头的数量：

+   `d_in`: 输入特征向量的维度。

+   `d_out_kq`: 查询和键的输出维度。

+   `d_out_v`: 值的输出维度。

+   `num_heads`: 注意力头的数量。

我们使用这些输入参数初始化`SelfAttention`类`num_heads`次。我们使用PyTorch的`nn.ModuleList`来存储这些多个`SelfAttention`实例。

接下来，`forward`传递涉及独立地将每个`SelfAttention`头（存储在`self.heads`中）应用于输入`x`。然后，每个头的结果沿着最后一个维度（`dim=-1`）进行连接。让我们在下面看看它的实际操作！

首先，让我们假设我们有一个单一的自注意力头，输出维度为1，以便简单地进行说明：

**In:**

```
torch.manual_seed(123)
​
d_in, d_out_kq, d_out_v = 3, 2, 1
​
sa = SelfAttention(d_in, d_out_kq, d_out_v)
print(sa(embedded_sentence))
```

**Out:**

```
tensor([[-0.0185],
        [ 0.4003],
        [-0.1103],
        [ 0.0668],
        [ 0.1180],
        [-0.1827]], grad_fn=<MmBackward0>)
```

现在，让我们扩展到4个注意力头：

**In:**

```
torch.manual_seed(123)
​
block_size = embedded_sentence.shape[1]
mha = MultiHeadAttentionWrapper(
    d_in, d_out_kq, d_out_v, num_heads=4
)
​
context_vecs = mha(embedded_sentence)
​
print(context_vecs)
print("context_vecs.shape:", context_vecs.shape)
```

**Out:**

```
tensor([[-0.0185,  0.0170,  0.1999, -0.0860],
        [ 0.4003,  1.7137,  1.3981,  1.0497],
        [-0.1103, -0.1609,  0.0079, -0.2416],
        [ 0.0668,  0.3534,  0.2322,  0.1008],
        [ 0.1180,  0.6949,  0.3157,  0.2807],
        [-0.1827, -0.2060, -0.2393, -0.3167]], grad_fn=<CatBackward0>)
context_vecs.shape: torch.Size([6, 4])
```

基于上面的输出，你可以看到之前创建的单个自注意力头现在代表了上面输出张量的第一列。

请注意，多头注意力的结果是一个 6×4 维的张量：我们有 6 个输入令牌和 4 个自注意力头，每个自注意力头返回一个 1 维输出。在之前的自注意力部分中，我们也生成了一个 6×4 维的张量。这是因为我们将输出维度设置为 4 而不是 1。实际上，如果我们可以在 `SelfAttention` 类本身中调节输出嵌入大小，为什么我们需要多个注意力头呢？

单个自注意力头增加输出维度和使用多个注意力头的区别在于模型处理和从数据中学习的方式。虽然这两种方法都增加了模型表示数据不同特征或方面的能力，但它们的方式根本不同。

例如，多头注意力中的每个注意力头都有可能学会聚焦于输入序列的不同部分，捕捉数据中的各种方面或关系。这种表示上的多样性是多头注意力成功的关键。

多头注意力也可以更加高效，特别是在并行计算方面。每个头可以独立处理，使其非常适合现代硬件加速器，如 GPU 或 TPU，这些加速器擅长并行处理。

简而言之，使用多个注意力头不仅仅是增加模型的容量，而是增强其学习数据中各种特征和关系的能力。例如，7B Llama 2 模型使用了 32 个注意力头。

在上述代码演示中，我们设定 ***d_q = d_k = 2*** 以及 ***d_v = 4***。换句话说，我们使用相同的维度来处理查询和键序列。尽管值矩阵 ***W_v*** 通常选择与查询和键矩阵具有相同的维度（比如在 PyTorch 的 [MultiHeadAttention](https://pytorch.org/docs/stable/generated/torch.nn.MultiheadAttention.html) 类中），我们可以为值维度选择任意数量的大小。

由于维度有时候比较难以跟踪，请看下面的图总结到目前为止我们所涵盖的所有内容，图中描述了单个注意力头的各种张量大小。

先前实施的自注意力机制的另一种观点，重点是矩阵维度

现在，上面的插图对应于变压器中使用的*自我*-注意力机制。我们尚未讨论的此注意力机制的一个特定类型是*交叉*-注意力。

什么是跨注意力，它与自注意力有何不同？

在自注意力中，我们使用相同的输入序列。在跨注意力中，我们混合或结合两个*不同*的输入序列。在上面的原始变压器架构中，这是左侧编码器模块返回的序列和右侧解码器部分正在处理的输入序列。

请注意，在交叉注意力中，两个输入序列 `x_1` 和 `x_2` 可能具有不同数量的元素。但是，它们的嵌入维度必须匹配。

下图说明了交叉注意力的概念。如果我们设置 `x_1` *=* `x_2`，这相当于自注意力。

（注意，查询通常来自解码器，而键和值通常来自编码器。）

在代码中如何实现呢？我们将采用并修改我们之前在自注意力部分中实现的 `SelfAttention` 类，并仅进行一些微小的修改：

**输入：**

```
class CrossAttention(nn.Module):
​
    def __init__(self, d_in, d_out_kq, d_out_v):
        super().__init__()
        self.d_out_kq = d_out_kq
        self.W_query = nn.Parameter(torch.rand(d_in, d_out_kq))
        self.W_key   = nn.Parameter(torch.rand(d_in, d_out_kq))
        self.W_value = nn.Parameter(torch.rand(d_in, d_out_v))
​
    def forward(self, x_1, x_2):           # x_2 is new
        queries_1 = x_1 @ self.W_query

        keys_2 = x_2 @ self.W_key          # new
        values_2 = x_2 @ self.W_value      # new

        attn_scores = queries_1 @ keys_2.T # new 
        attn_weights = torch.softmax(
            attn_scores / self.d_out_kq**0.5, dim=-1)

        context_vec = attn_weights @ values_2
        return context_vec
```

`CrossAttention` 类与先前的 `SelfAttention` 类的区别如下：

+   `forward` 方法接受两个不同的输入，`x_1` 和 `x_2`。查询来自 `x_1`，而键和值来自 `x_2`。这意味着注意力机制正在评估两个不同输入之间的交互。

+   注意力分数是通过计算查询（来自 `x_1`）和键（来自 `x_2`）的点积来计算的。

+   与 `SelfAttention` 类似，每个上下文向量都是值的加权和。但是，在 `CrossAttention` 中，这些值来自第二个输入 (`x_2`)，而权重基于 `x_1` 和 `x_2` 之间的交互。

让我们看看它的运行情况：

**输入：**

```
torch.manual_seed(123)
​
d_in, d_out_kq, d_out_v = 3, 2, 4
​
crossattn = CrossAttention(d_in, d_out_kq, d_out_v)
​
first_input = embedded_sentence
second_input = torch.rand(8, d_in)
​
print("First input shape:", first_input.shape)
print("Second input shape:", second_input.shape)
```

**输出：**

```
First input shape: torch.Size([6, 3])
Second input shape: torch.Size([8, 3])
```

注意，计算交叉注意力时，第一个和第二个输入的标记数量（这里是行）不必相同：

**输入：**

```
context_vectors = crossattn(first_input, second_input)
​
print(context_vectors)
print("Output shape:", context_vectors.shape)
```

**输出：**

```
tensor([[0.4231, 0.8665, 0.6503, 1.0042],
        [0.4874, 0.9718, 0.7359, 1.1353],
        [0.4054, 0.8359, 0.6258, 0.9667],
        [0.4357, 0.8886, 0.6678, 1.0311],
        [0.4429, 0.9006, 0.6775, 1.0460],
        [0.3860, 0.8021, 0.5985, 0.9250]], grad_fn=<MmBackward0>)
Output shape: torch.Size([6, 4])
```

我们在上面谈了很多关于语言转换器的内容。在原始 transformer 架构中，当我们在语言翻译的上下文中从输入句子到输出句子时，交叉注意力非常有用。输入句子代表一个输入序列，而翻译代表第二个输入序列（两个句子的单词数量可以不同）。

另一个使用交叉注意力的热门模型是 Stable Diffusion。 Stable Diffusion 在 U-Net 模型中生成的图像和用于条件的文本提示之间使用交叉注意力，如 [具有潜在扩散模型的高分辨率图像合成](https://arxiv.org/abs/2112.10752) 中描述的——这是描述 Stable Diffusion 模型的原始论文，后来由 Stability AI 采用以实现流行的 Stable Diffusion 模型。

在本节中，我们将先前讨论的自注意力机制调整为因果自注意力机制，专门用于生成文本的类似 GPT（解码器风格）的 LLM。这种因果自注意力机制通常也被称为“掩码自注意力”。在原始 transformer 架构中，它对应于“掩码多头注意力”模块——为简单起见，我们将在本节中查看一个注意力头，但相同的概念可以推广到多个头。

因果自注意力确保序列中某个位置的输出仅基于先前位置的已知输出，而不是未来位置。简单来说，它确保了每个下一个单词的预测仅取决于前面的单词。为了在类似 GPT 的 LLMs 中实现这一点，对于每个处理的标记，我们都会屏蔽未来的标记，这些标记位于输入文本中当前标记之后。

在输入中对隐藏未来输入标记的注意力权重应用因果掩码如下图所示。

为了说明和实现因果自注意力，让我们使用前一节的未加权注意力得分和注意力权重。首先，我们快速回顾一下前一节*自注意力*部分中的注意力得分的计算：

**输入：**

```
torch.manual_seed(123)
​
d_in, d_out_kq, d_out_v = 3, 2, 4
​
W_query = nn.Parameter(torch.rand(d_in, d_out_kq))
W_key   = nn.Parameter(torch.rand(d_in, d_out_kq))
W_value = nn.Parameter(torch.rand(d_in, d_out_v))
​
x = embedded_sentence
​
keys = x @ W_key
queries = x @ W_query
values = x @ W_value
​
# attn_scores are the "omegas", 
# the unnormalized attention weights
attn_scores = queries @ keys.T 
​
print(attn_scores)
print(attn_scores.shape)
```

**输出：**

```
tensor([[ 0.0613, -0.3491,  0.1443, -0.0437, -0.1303,  0.1076],
        [-0.6004,  3.4707, -1.5023,  0.4991,  1.2903, -1.3374],
        [ 0.2432, -1.3934,  0.5869, -0.1851, -0.5191,  0.4730],
        [-0.0794,  0.4487, -0.1807,  0.0518,  0.1677, -0.1197],
        [-0.1510,  0.8626, -0.3597,  0.1112,  0.3216, -0.2787],
        [ 0.4344, -2.5037,  1.0740, -0.3509, -0.9315,  0.9265]],
       grad_fn=<MmBackward0>)
torch.Size([6, 6])
```

与之前的*自注意力*部分类似，上面的输出是一个包含这些成对未归一化的注意力权重（也称为注意力得分）的 6×6 张量，用于 6 个输入标记。

之前，我们通过 softmax 函数计算了缩放点积注意力，如下所示：

**输入：**

```
attn_weights = torch.softmax(attn_scores / d_out_kq**0.5, dim=1)
print(attn_weights)
```

**输出：**

```
tensor([[0.1772, 0.1326, 0.1879, 0.1645, 0.1547, 0.1831],
        [0.0386, 0.6870, 0.0204, 0.0840, 0.1470, 0.0229],
        [0.1965, 0.0618, 0.2506, 0.1452, 0.1146, 0.2312],
        [0.1505, 0.2187, 0.1401, 0.1651, 0.1793, 0.1463],
        [0.1347, 0.2758, 0.1162, 0.1621, 0.1881, 0.1231],
        [0.1973, 0.0247, 0.3102, 0.1132, 0.0751, 0.2794]],
       grad_fn=<SoftmaxBackward0>)
```

上面的 6×6 输出表示了注意力权重，我们也在*自注意力*部分之前计算过。

现在，在类似 GPT 的 LLMs 中，我们训练模型逐个标记（或单词）从左到右读取和生成。如果我们有一个训练文本样本如“Life is short eat desert first”，我们有以下设置，其中箭头右侧的词的上下文向量应仅包含其自身和前面的词：

上述设置的最简单方法是通过将注意力权重矩阵在对角线以上应用掩码来屏蔽所有未来的标记，如下图所示。这样，“未来”的单词在创建上下文向量时将不被包括在内，这些向量是对输入的注意力加权和。

应屏蔽掉对角线以上的注意力权重

在代码中，我们可以通过 PyTorch 的 [tril](https://pytorch.org/docs/stable/generated/torch.tril.html#) 函数实现这一点，首先我们使用它来创建一个由 1 和 0 组成的掩码：

**输入：**

```
block_size = attn_scores.shape[0]
mask_simple = torch.tril(torch.ones(block_size, block_size))
print(mask_simple)
```

**输出：**

```
tensor([[1., 0., 0., 0., 0., 0.],
        [1., 1., 0., 0., 0., 0.],
        [1., 1., 1., 0., 0., 0.],
        [1., 1., 1., 1., 0., 0.],
        [1., 1., 1., 1., 1., 0.],
        [1., 1., 1., 1., 1., 1.]])
```

接下来，我们将注意力权重与这个掩码相乘，将对角线以上的所有注意力权重归零：

**输入：**

```
masked_simple = attn_weights*mask_simple
print(masked_simple)
```

**输出：**

```
tensor([[0.1772, 0.0000, 0.0000, 0.0000, 0.0000, 0.0000],
        [0.0386, 0.6870, 0.0000, 0.0000, 0.0000, 0.0000],
        [0.1965, 0.0618, 0.2506, 0.0000, 0.0000, 0.0000],
        [0.1505, 0.2187, 0.1401, 0.1651, 0.0000, 0.0000],
        [0.1347, 0.2758, 0.1162, 0.1621, 0.1881, 0.0000],
        [0.1973, 0.0247, 0.3102, 0.1132, 0.0751, 0.2794]],
       grad_fn=<MulBackward0>)
```

虽然上述是屏蔽未来单词的一种方法，但请注意，每行中的注意力权重不再总和为一。为了缓解这一点，我们可以规范化行，使它们再次总和为 1，这是注意力权重的标准约定：

**输入：**

```
row_sums = masked_simple.sum(dim=1, keepdim=True)
masked_simple_norm = masked_simple / row_sums
print(masked_simple_norm)
```

**输出：**

```
tensor([[1.0000, 0.0000, 0.0000, 0.0000, 0.0000, 0.0000],
        [0.0532, 0.9468, 0.0000, 0.0000, 0.0000, 0.0000],
        [0.3862, 0.1214, 0.4924, 0.0000, 0.0000, 0.0000],
        [0.2232, 0.3242, 0.2078, 0.2449, 0.0000, 0.0000],
        [0.1536, 0.3145, 0.1325, 0.1849, 0.2145, 0.0000],
        [0.1973, 0.0247, 0.3102, 0.1132, 0.0751, 0.2794]],
       grad_fn=<DivBackward0>)
```

正如我们所看到的，每一行中的注意力权重现在总和为 1。

在神经网络中对注意权重进行归一化，例如在变换模型中，比未归一化的权重有两个主要优势。首先，归一化的注意权重之和为1类似于概率分布。这使得更容易以比例的形式解释模型对输入的各个部分的关注。其次，通过约束注意权重之和为1，这种归一化有助于控制权重和梯度的规模，以改善训练动态。

**没有重新归一化的更有效的屏蔽**

在我们上面编写的因果自注意力过程中，我们首先计算注意分数，然后计算注意权重，屏蔽掉对角线上方的注意权重，最后重新归一化注意权重。下图总结了这一点：

先前实施的因果自注意力过程

或者，有一种更有效的方法来实现相同的结果。在这种方法中，我们取注视分数，并在输入softmax函数计算注视权重之前用负无穷替换对角线上的值。在下图中总结了这一点：

实施因果自注意力的另一种更有效的方法

我们可以在PyTorch中编写这个过程，首先是对注视分数进行上三角部分的mask：

**输入:**

```
mask = torch.triu(torch.ones(block_size, block_size), diagonal=1)
masked = attn_scores.masked_fill(mask.bool(), -torch.inf)
print(masked)
```

上面的代码首先创建一个带有对角线以下为0，对角线以上为1的`mask`。在这里，`torch.triu`（**u**pper **tri**angle）保留矩阵的主对角线及其以上的元素，将其以下的元素归零，从而保留上三角部分。相反，`torch.tril`（**l**ower **t**riangle）保留主对角线及其以下的元素。

然后，通过正面mask值（1s）将对角线以上的所有元素替换为`-torch.inf`，结果如下所示。

**输出:**

```
tensor([[ 0.0613,    -inf,    -inf,    -inf,    -inf,    -inf],
        [-0.6004,  3.4707,    -inf,    -inf,    -inf,    -inf],
        [ 0.2432, -1.3934,  0.5869,    -inf,    -inf,    -inf],
        [-0.0794,  0.4487, -0.1807,  0.0518,    -inf,    -inf],
        [-0.1510,  0.8626, -0.3597,  0.1112,  0.3216,    -inf],
        [ 0.4344, -2.5037,  1.0740, -0.3509, -0.9315,  0.9265]],
       grad_fn=<MaskedFillBackward0>)
```

然后，我们只需像往常一样应用softmax函数，就可以得到归一化和屏蔽的注意权重：

**输入:**

```
attn_weights = torch.softmax(masked / d_out_kq**0.5, dim=1)
print(attn_weights)
```

**输出:**

```
tensor([[1.0000, 0.0000, 0.0000, 0.0000, 0.0000, 0.0000],
        [0.0532, 0.9468, 0.0000, 0.0000, 0.0000, 0.0000],
        [0.3862, 0.1214, 0.4924, 0.0000, 0.0000, 0.0000],
        [0.2232, 0.3242, 0.2078, 0.2449, 0.0000, 0.0000],
        [0.1536, 0.3145, 0.1325, 0.1849, 0.2145, 0.0000],
        [0.1973, 0.0247, 0.3102, 0.1132, 0.0751, 0.2794]],
       grad_fn=<SoftmaxBackward0>) 
```

为什么这样做？在最后一步应用的softmax函数将输入值转换为概率分布。当输入中存在`-inf`时，softmax有效地将其视为零概率。这是因为`e^(-inf)`接近于0，因此这些位置对输出概率没有贡献。

在本文中，我们通过逐步编码的方式探索了自注意力的内部工作。基于此，我们进一步研究了多头注意力，这是大型语言变换器的基本组件。

然后，我们还编写了交叉注意力，这是自注意力的一种变体，当应用于两个不同序列之间时特别有效。最后，我们编写了因果自注意力，这是在诸如GPT和Llama等解码器风格LLM中生成连贯和语境适当序列的关键概念。

通过从零开始编写这些复杂的机制，希望您对transformers和LLMs中使用的自注意力机制的内部工作原理有了很好的理解。

（请注意，本文中提供的代码仅用于说明目的。如果您计划为训练LLMs实现自注意力，我建议考虑优化的实现，如[Flash Attention](https://arxiv.org/abs/2307.08691)，它们可以减少内存占用和计算负载。）

如果你喜欢这篇文章，我的[从零开始构建大型语言模型](http://mng.bz/amjo)书籍通过类似（但更详细）的从零开始的方法来解释LLMs的工作原理。这包括编码数据处理步骤、LLM架构、预训练、微调和对齐阶段。

该书目前是Manning早期获取计划的一部分，新章节将定期发布。（通过Manning购买目前打折的早期获取版本的购买者也将在发布时收到最终的书籍。）相应的代码可在[GitHub上获得](https://github.com/rasbt/LLMs-from-scratch)。

* * *

*这本杂志是一个个人的热情项目，不提供直接的报酬。然而，对于那些希望支持我的人，请考虑购买[我的其中一本书](https://sebastianraschka.com/books)的副本。如果你觉得它们富有见地并且有益，请随时向你的朋友和同事推荐。*

**您的支持意义重大！谢谢！**
