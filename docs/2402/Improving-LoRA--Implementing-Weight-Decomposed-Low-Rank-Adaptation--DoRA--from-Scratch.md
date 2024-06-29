<!--yml

category: 未分类

date: 2024-05-27 14:58:06

-->

# 改进 LoRA：从头开始实现权重分解低秩适应（DoRA）

> 来源：[https://magazine.sebastianraschka.com/p/lora-and-dora-from-scratch](https://magazine.sebastianraschka.com/p/lora-and-dora-from-scratch)

低秩适应（LoRA）是一种机器学习技术，通过调整模型参数的一个小而低秩的子集（例如，预训练的 LLM 或视觉变换器），从而更好地适应特定的、通常是更小的数据集。

这种方法很重要，因为它允许在特定任务数据上高效微调大型模型，显著降低微调所需的计算成本和时间。

上周，研究人员提出了 [DoRA: 权重分解低秩适应](https://arxiv.org/abs/2402.09353)，这是 LoRA 的一个新的替代方法，可能在很大程度上优于 LoRA。

为了理解这些方法的工作原理，我们将在本文中从头开始用 PyTorch 实现 LoRA 和 DoRA ！

在深入讨论 DoRA 之前，让我们简要回顾一下 [LoRA](https://arxiv.org/abs/2106.09685) 的工作原理。

由于 LLM 很大，在训练过程中更新所有模型权重可能会因 GPU 内存限制而变得昂贵。假设我们有一个给定层的大权重矩阵 ***W***。在反向传播过程中，我们学习一个 ***ΔW*** 矩阵，其中包含了我们在训练过程中希望更新原始权重以最小化损失函数的信息。

在常规训练和微调中，权重更新定义如下：

***W[更新] = W + ΔW***

由 [Hu](https://arxiv.org/abs/2106.09685) *[等人](https://arxiv.org/abs/2106.09685)* 提出的 LoRA 方法提供了一个更有效的替代方案，用于计算权重更新 ***ΔW*** 的近似值 ***ΔW ≈ AB***。换句话说，在 LoRA 中，我们有以下形式，其中 ***A*** 和 ***B*** 是两个小的权重矩阵：

***W[更新] = W + A.B***

（在 "***A.B***" 中的 "***.***" 表示矩阵乘法。）

下图展示了全面微调和 LoRA 两者并列的公式。

*图：全面微调（左）和 LoRA 微调（右）的示意图。*

LoRA 如何节省 GPU 内存？如果预训练的权重矩阵 ***W*** 是一个 1000×1000 的矩阵，那么在常规微调中，权重更新矩阵 ***ΔW*** 也是一个 1000×1000 的矩阵。在这种情况下，***ΔW*** 有 1000000 个参数。如果我们考虑 LoRA 排名为 2，则 ***A*** 是一个 1000×2 的矩阵，***B*** 是一个 2×1000 的矩阵，当使用 LoRA 时，我们只需要更新 2×2×1000 = 4000 个参数。在前面的例子中，排名为 2 时，这是少了 250 倍的参数。

当然，***A*** 和 ***B*** 不能捕捉到 ***ΔW*** 能捕捉到的所有信息，但这是有意设计的。在使用LoRA时，我们假设模型需要一个完整排名的大矩阵 *W* 来捕获预训练数据集中的所有知识。然而，当我们对LLM进行微调时，我们不需要更新所有权重，只需在比 ***ΔW*** 更少的权重中捕捉适应的核心信息；因此，我们通过 ***AB*** 进行低秩更新。

如果您仔细观察，上述图中的完整微调和LoRA描述与我之前展示的公式略有不同。这是由于矩阵乘法的分配法则：我们不必将权重与更新后的权重相加，而可以保持它们分开。例如，如果 ***x*** 是输入数据，则对于常规微调，我们可以写如下内容：

***x.(W+ΔW) = x.W + x.ΔW***

类似地，我们可以为LoRA编写以下内容：

***x.(W+A.B)** = **x.W + x.A.B***

我们可以保持LoRA权重矩阵分开的事实使LoRA特别吸引人。在实践中，这意味着我们根本不需要修改预训练模型的权重，因为我们可以在运行时应用LoRA矩阵。如果考虑为多个客户托管模型，这将特别有用。与必须为每个客户保存大型更新模型不同，您只需保存一小组LoRA权重以及原始预训练模型。

为了使这个过程更加具体并提供额外的直觉，我们将在下一节从头开始用代码实现LoRA。

我们首先通过初始化一个 `LoRALayer` 来创建矩阵A和B，以及alpha缩放超参数和排名超参数。这一层可以接受输入并计算相应的输出，如下图所示。

展示了LoRA矩阵A和B的排名 *r*。

在代码中，上述在图中描述的LoRA层如下所示：

```
import torch.nn as nn

class LoRALayer(nn.Module):
    def __init__(self, in_dim, out_dim, rank, alpha):
        super().__init__()
        std_dev = 1 / torch.sqrt(torch.tensor(rank).float())
        self.A = nn.Parameter(torch.randn(in_dim, rank) * std_dev)
        self.B = nn.Parameter(torch.zeros(rank, out_dim))
        self.alpha = alpha

    def forward(self, x):
        x = self.alpha * (x @ self.A @ self.B)
        return x
```

在上述代码中，`rank` 是控制矩阵 *A* 和 *B* 内部维度的超参数。换句话说，此参数控制LoRA引入的额外参数数量，并且是决定模型适应性与参数效率平衡的关键因素。

第二个超参数 `alpha` 是应用于低秩适应输出的缩放超参数。它实质上控制适应层输出对被适应层原始输出的影响程度。这可以看作是调节低秩适应对层输出影响的一种方式。

到目前为止，我们上面实现的 `LoRALayer` 类允许我们转换层输入 `x`。然而，在LoRA中，我们通常有兴趣替换现有的线性层，以便将权重更新应用于现有的预训练权重，如下图所示：

*应用于现有线性层的LoRA*

为了像上图所示那样融入原始的 Linear 层权重，我们将实现一个 `LinearWithLoRA` 层，它使用先前实现的 `LoRALayer`，并可以用来替换神经网络中现有的 `Linea`r 层，例如，LLM 中的自注意模块或前馈模块：

```
class LinearWithLoRA(nn.Module):

    def __init__(self, linear, rank, alpha):
        super().__init__()
        self.linear = linear
        self.lora = LoRALayer(
            linear.in_features, linear.out_features, rank, alpha
        )

    def forward(self, x):
        return self.linear(x) + self.lora(x)
```

请注意，由于在 LoRA 层中用零值初始化权重矩阵 B（`LoraLayer` 中的 `self.B`），矩阵 *A* 和 *B* 的乘积结果是一个由 0 组成的矩阵，并且不影响原始权重（因为将 0 加到原始权重上不会修改它们）。

让我们在一个由单个 `Linear` 层表示的小神经网络层上尝试 LoRA：

**输入：**

```
torch.manual_seed(123)
layer = nn.Linear(10, 2)
x = torch.randn((1, 10))

print("Original output:", layer(x))
```

**输出：**

```
Original output: tensor([[0.6639, 0.4487]], grad_fn=<AddmmBackward0>)
```

现在，将 LoRA 应用于 `Linea`r 层，我们看到结果与之前相同，因为我们还没有训练 LoRA 权重。换句话说，一切都如预期地工作：

**输入：**

```
layer_lora_1 = LinearWithLoRA(layer, rank=2, alpha=4)
print("LoRA output:", layer_lora_1(x))
```

**输出：**

```
LoRA output: tensor([[0.6639, 0.4487]], grad_fn=<AddmmBackward0>)
```

之前，我提到了矩阵乘法的分配律：

***x.(W+A.B)** = **x.W + x.A.B***。

这意味着我们可以将 LoRA 矩阵和原始权重合并或合并，这应该会得到一个等效的实现。在代码中，这种对 `LinearWithLoRA` 层的替代实现如下所示：

```
class LinearWithLoRAMerged(nn.Module):
    def __init__(self, linear, rank, alpha):
        super().__init__()
        self.linear = linear
        self.lora = LoRALayer(
            linear.in_features, linear.out_features, rank, alpha
        )

    def forward(self, x):
        lora = self.lora.A @ self.lora.B # Combine LoRA matrices
        # Then combine LoRA with orig. weights
        combined_weight = self.linear.weight + self.lora.alpha*lora.T 
        return F.linear(x, combined_weight, self.linear.bias)
```

简而言之，`LinearWithLoRAMerged` 计算方程左侧 ***x.(W+A.B)** = **x.W + x.A.B***，而 `LinearWithLoRA` 计算右侧 -- 两者等价。

我们可以通过以下代码验证这与之前的输出相同：

**输入：**

```
layer_lora_2 = LinearWithLoRAMerged(layer, rank=2, alpha=4)
print("LoRA output:", layer_lora_2(x))
```

**输出：**

```
LoRA output: tensor([[0.6639, 0.4487]], grad_fn=<AddmmBackward0>)
```

现在我们有了一个可行的 LoRA 实现，让我们看看如何在下一节将其应用于神经网络。

为什么我们要以上述方式使用 PyTorch 模块实现 LoRA？这种方法使我们能够轻松地用我们的新 `LinearWithLoRA`（或 `LinearWithLoRAMerged`）层替换现有神经网络中的 `Linear` 层（例如，大语言模型的前馈或注意模块）。

为简单起见，我们现在专注于一个小的3层多层感知器，而不是LLM，如下图所示：

一个简单的3层多层感知器

在代码中，我们可以将上述显示的多层感知器实现如下：

**输入：**

```
class MultilayerPerceptron(nn.Module):
    def __init__(self, num_features, 
        num_hidden_1, num_hidden_2, num_classes):
        super().__init__()
        self.layers = nn.Sequential(
            nn.Linear(num_features, num_hidden_1),
            nn.ReLU(),
            nn.Linear(num_hidden_1, num_hidden_2),
            nn.ReLU(),

            nn.Linear(num_hidden_2, num_classes)
        )

    def forward(self, x):
        x = self.layers(x)
        return x

model = MultilayerPerceptron(
    num_features=num_features,
    num_hidden_1=num_hidden_1,
    num_hidden_2=num_hidden_2, 
    num_classes=num_classes
)

print(model)
```

**输出：**

```
MultilayerPerceptron(
  (layers): Sequential(
    (0): Linear(in_features=784, out_features=128, bias=True)
    (1): ReLU()
    (2): Linear(in_features=128, out_features=256, bias=True)
    (3): ReLU()
    (4): Linear(in_features=256, out_features=10, bias=True)
  )
)
```

使用 `LinearWithLora`，我们可以通过在多层感知器模型中替换原始的 `Linear` 层来添加 LoRA 层：

**输入：**

```
model.layers[0] = LinearWithLoRA(model.layers[0], rank=4, alpha=8)
model.layers[2] = LinearWithLoRA(model.layers[2], rank=4, alpha=8)
model.layers[4] = LinearWithLoRA(model.layers[4], rank=4, alpha=8)

print(model)
```

**输出：**

```
MultilayerPerceptron(
  (layers): Sequential(
    (0): LinearWithLoRA(
      (linear): Linear(in_features=784, out_features=128, bias=True)
      (lora): LoRALayer()
    )
    (1): ReLU()
    (2): LinearWithLoRA(
      (linear): Linear(in_features=128, out_features=256, bias=True)
      (lora): LoRALayer()
    )
    (3): ReLU()
    (4): LinearWithLoRA(
      (linear): Linear(in_features=256, out_features=10, bias=True)
      (lora): LoRALayer()
    )
  )
)
```

然后，我们可以冻结原始的 `Linear` 层，并且只让 `LoRALaye`r 层可训练，如下所示：

**输入：**

```
def freeze_linear_layers(model):
    for child in model.children():
        if isinstance(child, nn.Linear):
            for param in child.parameters():
                param.requires_grad = False
        else:
            # Recursively freeze linear layers in children modules
            freeze_linear_layers(child)

freeze_linear_layers(model)
for name, param in model.named_parameters():
    print(f"{name}: {param.requires_grad}")
```

**输出：**

```
layers.0.linear.weight: False
layers.0.linear.bias: False
layers.0.lora.A: True
layers.0.lora.B: True
layers.2.linear.weight: False
layers.2.linear.bias: False
layers.2.lora.A: True
layers.2.lora.B: True
layers.4.linear.weight: False
layers.4.linear.bias: False
layers.4.lora.A: True
layers.4.lora.B: True
```

基于上述 `True` 和 `False` 的值，我们可以直观地确认现在只有 LoRA 层是可训练的（`True` 表示可训练，`False` 表示冻结）。在实践中，我们将使用这种 LoRA 配置在新的数据集或任务上训练网络。

为了避免使本文变得过长，我跳过了用于训练此模型的样板代码。但如果您对完整的代码感兴趣，可以在这里找到一个独立的代码笔记本：[https://github.com/rasbt/dora-from-scratch](https://github.com/rasbt/dora-from-scratch)。

此外，如果您对从头开始的LoRA解释和应用于LLM的LoRA感兴趣，请还查看我Lightning Studio的[《LoRA从零开始 - 在PyTorch中实现LLM的低秩适应》](https://lightning.ai/lightning-ai/studios/code-lora-from-scratch)。

您可能已经注意到，我们花了很多时间实施和讨论LoRA。这是因为DoRA（[Weight-Decomposed Low-Rank Adaptation](https://arxiv.org/abs/2402.09353)）可以被看作是在LoRA基础上构建的改进或扩展，我们现在可以轻松地调整之前的一些代码来实现DoRA。

DoRA可以用两步来描述，其中第一步是将预训练的权重矩阵分解为幅度向量(***m***)和方向矩阵(***V***）。第二步是将LoRA应用于方向矩阵***V***并分别训练幅度向量***m***。

将向量分解为幅度和方向两个部分的方法灵感来自数学原理，即任何向量都可以表示为其幅度（表示其长度的标量值）和其方向（表示其在空间中方向的单位向量）的乘积。

单个向量的方向和幅度的示意图。例如，如果有一个二维向量[1, 2]，我们可以将其分解为幅度2.24和方向向量[0.447, 0.894]。然后，2.24 * [0.447, 0.894] = [1, 2]。

在DoRA中，我们将预训练的整个权重矩阵***W***分解为幅度和方向两个部分，其中权重矩阵的每列（向量）对应于连接所有输入到特定输出神经元的权重。

因此，分解***W***的结果是一个表示权重矩阵中每个列向量的规模或长度的幅度向量***m***，如下图所示。

DoRA中权重矩阵分解的示意图

然后，DoRA使用方向矩阵***V***并应用标准LoRA，例如：

***W' = m (V + ΔV)/norm = m (W + AB)/norm***

归一化，我在这个概述中简称为"norm"，是基于Saliman和Kingma在2016年提出的《[Weight Normalization: A Simple Reparameterization to Accelerate Training of Deep Neural Networks](https://arxiv.org/abs/1602.07868)》论文中提出的权重归一化方法。

DoRA的两步过程（将预训练的权重矩阵分解并将LoRA应用于方向矩阵）在下面的DoRA论文中的图表中进一步说明。

开发 DoRA 的动机基于分析和比较 LoRA 和完全微调学习模式。DoRA 的作者发现，LoRA 可能会等比例增加或减少大小和方向更新，但似乎缺乏在完全微调中发现的仅进行细微方向更改的能力。因此，研究人员提出了大小和方向分量的解耦。

换句话说，他们的 DoRA 方法旨在仅对方向分量 ***V*** 应用 LoRA，同时允许大小分量 ***m*** 分别进行训练。

引入大小向量 ***m*** 如果与 LoRA 相比，DoRA 将增加 0.01% 的参数。然而，在 LLM 和视觉变换器基准测试中，他们发现，即使将 DoRA 的秩减半，例如当 DoRA 仅使用正常 LoRA 参数的一半时，DoRA 的性能仍然优于 LoRA，如下面的性能比较所示。

正如我几个月前在另一篇文章中所写的，LoRA 需要精心调整秩以优化性能：[使用 LoRA（低秩适应）进行微调LLM的实用技巧](https://magazine.sebastianraschka.com/p/practical-tips-for-finetuning-llms-using-lora)。然而，DoRA 在秩的变化上似乎更加稳健，如下面的比较所示。

成功使用较小秩的 DoRA 的可能性使得这种方法比 LoRA 更加参数高效。

总的来说，我对结果印象深刻，并且将 LoRA 实现升级为 DoRA 应该不会太困难，我们将在下一节中完成。

在这一节中，我们将看到 DoRA 在代码中的实现。之前我们说过，我们可以使用大小 ***m*** 和方向分量 ***V*** 初始化预训练权重 ***W[0]***。例如，我们有以下方程：

\(W_0 = m \frac{V}{||V||_c} \)

其中 ||***V***||[c] 是向量 ***V*** 的向量范数。然后我们可以像下面这样编写包括 LoRA 权重更新 ***BA*** 的 DoRA：

\(W^{\prime}=m \frac{V+BA}{\|V+BA\|_c}\)

现在，在 DoRA 论文中，作者将 DoRA 阐述如下，他们在训练期间使用初始预训练权重 *W[0]* 作为方向分量，并学习大小向量 ***m***：

\(W^{\prime}={m} \frac{V+\Delta V}{\|V+\Delta V\|_c}={m} \frac{W_0+{B A}}{\left\|W_0+{B A}\right\|_c}\)

这里，***ΔV*** 是方向分量矩阵 ***V*** 的更新。

虽然原始作者尚未发布官方实现，但你可以在[这里](https://github.com/catid/dora/blob/main/dora.py)找到一个独立的实现，这个实现粗略地启发了我下面的实现。

利用我们之前的 `LinearWithLoRAMerged` 实现，我们可以将其升级为 DoRA，如下所示：

```
class LinearWithDoRAMerged(nn.Module):

    def __init__(self, linear, rank, alpha):
        super().__init__()
        self.linear = linear
        self.lora = LoRALayer(
            linear.in_features, linear.out_features, rank, alpha
        )
        self.m = nn.Parameter(
            self.linear.weight.norm(p=2, dim=0, keepdim=True))

  # Code loosely inspired by    
  # https://github.com/catid/dora/blob/main/dora.py

    def forward(self, x):
        lora = self.lora.A @ self.lora.B
        numerator = self.linear.weight + self.lora.alpha*lora.T
        denominator = numerator.norm(p=2, dim=0, keepdim=True)
        directional_component = numerator / denominator
        new_weight = self.m * directional_component
        return F.linear(x, new_weight, self.linear.bias)
```

`LinearWithDoRAMerged` 类在几个关键方面与我们之前的 `LinearWithLoRAMerged` 类不同，主要体现在如何修改和应用线性层权重上。然而，两个类都整合了 `LoRALayer` 来增强原始线性层的权重，但是DoRA添加了权重归一化和调整。

下图展示了两个类的文件差异并排显示：

`LinearWithLoRAMerged` 和 `LinearWithDoRAMerged` 的文件差异

如上图所示，`LinearWithDoRAMerged` 引入了一个额外步骤，涉及对增强权重进行动态归一化。

在将原始权重与LoRA调整后的权重（`self.linear.weight + self.lora.alpha*lora.T`）合并后，计算这些组合权重在列方向上的范数（`column_norm`）。然后，通过将它们除以它们的范数来对组合权重进行归一化（`V = combined_weight / column_norm`）。这一步确保了组合权重矩阵的每列具有单位范数，从而通过保持权重更新的规模来稳定学习过程。

DoRA还引入了一个可学习的向量 `self.m`，表示归一化权重矩阵每列的大小。该参数允许模型在训练过程中动态调整组合权重矩阵中每个权重向量的比例。这种额外的灵活性有助于模型更好地捕捉不同特征的重要性。

总之，`LinearWithDoRAMerged` 通过引入动态权重归一化和调整来扩展了 `LinearWithLoRAMerged` 的概念，从而提高了训练性能。

在实践中，考虑到之前的多层感知机，我们可以简单地将现有的线性层替换为我们的 `LinearWithDoRAMerged` 层，如下所示：

**输入:**

```
model.layers[0] = LinearWithDoRAMerged(model.layers[0], rank=4, alpha=8)
model.layers[2] = LinearWithDoRAMerged(model.layers[2], rank=4, alpha=8)
model.layers[4] = LinearWithDoRAMerged(model.layers[4], rank=4, alpha=8)

print(model)
```

**输出:**

```
MultilayerPerceptron(
  (layers): Sequential(
    (0): LinearWithDoRAMerged(
      (linear): Linear(in_features=784, out_features=128, bias=True)
      (lora): LoRALayer()
    )
    (1): ReLU()
    (2): LinearWithDoRAMerged(
      (linear): Linear(in_features=128, out_features=256, bias=True)
      (lora): LoRALayer()
    )
    (3): ReLU()
    (4): LinearWithDoRAMerged(
      (linear): Linear(in_features=256, out_features=10, bias=True)
      (lora): LoRALayer()
    )
  )
)
```

在微调模型之前，我们可以重用之前实现的 `freeze_linear_layers` 函数，仅使LoRA权重和幅度向量可训练：

**输入:**

```
freeze_linear_layers(model)
for name, param in model.named_parameters():
    print(f"{name}: {param.requires_grad}")
```

**输出:**

```
layers.0.m: True
layers.0.linear.weight: False
layers.0.linear.bias: False
layers.0.lora.A: True
layers.0.lora.B: True
layers.2.m: True
layers.2.linear.weight: False
layers.2.linear.bias: False
layers.2.lora.A: True
layers.2.lora.B: True
layers.4.m: True
layers.4.linear.weight: False
layers.4.linear.bias: False
layers.4.lora.A: True
layers.4.lora.B: True
```

**包括模型训练的完整代码示例可在我的GitHub仓库中找到：[https://github.com/rasbt/dora-from-scratch](https://github.com/rasbt/dora-from-scratch).**

在我看来，DoRA似乎是LoRA的一个逻辑、有效和有前景的扩展，我很兴奋地想在真实的LLM微调环境中尝试它。

与此同时，我还将上述的DoRA实现添加到[LoRA从零开始 - 在PyTorch中为LLM实现低秩适应](https://lightning.ai/lightning-ai/studios/code-lora-from-scratch) Lightning Studio，用于微调DistilBERT语言模型（参见 `bonus_02_finetune-with-dora.ipynb`）。即使没有超参数调整，我已经看到比LoRA更高超过1%的预测准确率提升。

* * *

*这本杂志是我个人的热情项目，不提供直接的报酬。但是，对于那些希望支持我的人，请考虑购买一本[我的书籍之一](https://sebastianraschka.com/books)。如果你觉得它们有见地且有益，请随时向你的朋友和同事推荐。*

**你的支持对我意义重大！谢谢！**
