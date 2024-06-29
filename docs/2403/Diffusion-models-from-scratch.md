<!--yml

类别：未分类

日期：2024-05-27 14:49:17

-->

# 从头实现扩散模型

> 来源：[https://www.chenyang.co/diffusion.html](https://www.chenyang.co/diffusion.html)

\[\newcommand{\Kset}{\mathcal{K}} \newcommand{\distK}{ {\rm dist}_{\Kset} } \newcommand{\projK}{ {\rm proj}_{\Kset} } \newcommand{\eps}{\epsilon} \newcommand{\Loss}{\mathcal{L}} \newcommand{\norm}[1]{\left\lVert #1 \right\lVert} \newcommand{\R}{\mathbb{R}} \DeclareMathOperator{\softmin}{softmin} \DeclareMathOperator{\distop}{dist}\]

[论文（ICML 2024）](https://arxiv.org/abs/2306.04848)   [代码（Github）](https://github.com/yuanchenyang/smalldiffusion)

[HackerNews 上的讨论](https://news.ycombinator.com/item?id=39672450)

最近，扩散模型在生成建模中取得了令人印象深刻的结果，特别是从多模态分布中抽样。扩散模型不仅在文本到图像生成工具（如[Stable Diffusion](https://github.com/Stability-AI/stablediffusion)）中广泛采用，它们还在其他应用领域表现出色，比如[音频](https://text-to-audio.github.io/) / [视频](https://openai.com/research/video-generation-models-as-world-simulators/) / [3D](https://zero123.cs.columbia.edu/) 生成，[蛋白质设计](https://www.nature.com/articles/s41586-023-06415-8)，[机器人路径规划](https://diffusion-policy.cs.columbia.edu/)，所有这些都需要从多模态分布中抽样。

本教程旨在从优化的角度介绍扩散模型，如我们在[我们的论文](https://arxiv.org/abs/2306.04848)中介绍的（与[Frank Permenter](https://www.mit.edu/~fperment/)合作）。它将介绍理论和代码，使用理论来解释如何从头实现扩散模型。在教程结束时，您将学会为玩具数据集实现训练和抽样代码，这也适用于更大的数据集和模型。

在本教程中，我们将主要参考来自[`smalldiffusion`](https://github.com/yuanchenyang/smalldiffusion)的代码。出于教学目的，这里呈现的代码将简化自[原始库代码](https://github.com/yuanchenyang/smalldiffusion/blob/main/src/smalldiffusion/)，原始库本身有详细的注释，易于阅读。

#### 训练扩散模型

扩散模型旨在从训练样例学到的集合中生成样本，我们将其表示为 \(\mathcal{K}\)。例如，如果我们想生成图像，\(\mathcal{K} \subset \mathbb{R}^{c\times h \times w}\) 是对应于真实图像的像素值集合。扩散模型还适用于除图像之外的 \(\mathcal{K}\)，例如音频、视频、机器人轨迹，甚至离散域如文本生成。

简而言之，扩散模型的训练方法为：

1.  抽样 \(x_0 \sim \mathcal{K}\)，噪声水平 \(\sigma \sim [\sigma_\min, \sigma_\max]\)，噪声 \(\epsilon \sim N(0, I)\)

1.  生成带噪数据 \(x_\sigma = x_0 + \sigma \epsilon\)

1.  通过最小化平方损失从 \(x_\sigma\) 预测 \(\epsilon\)（噪声的方向）

这相当于通过最小化损失函数训练一个参数化为 \(\theta\) 的神经网络 \(\epsilon_\theta(x, \sigma)\)。

\[\Loss(\theta) = \mathop{\mathbb{E}} \lVert\epsilon_\theta(x_0 + \sigma_t \epsilon, \sigma_t) - \epsilon \lVert^2\]

在实践中，这通过以下简单的 `training_loop` 完成：

```
def training_loop(loader  : DataLoader,
                  model   : nn.Module,
                  schedule: Schedule,
                  epochs  : int = 10000):
    optimizer = torch.optim.Adam(model.parameters())
    for _ in range(epochs):
        for x0 in loader:
            optimizer.zero_grad()
            sigma, eps = generate_train_sample(x0, schedule)
            eps_hat = model(x0 + sigma * eps, sigma)
            loss = nn.MSELoss()(eps_hat, eps)
            optimizer.backward(loss)
            optimizer.step() 
```

训练循环迭代处理 `x0` 的批次，然后使用 `generate_train_sample` 来采样噪声水平 `sigma` 和噪声向量 `eps`：

```
def generate_train_sample(x0: torch.FloatTensor, schedule: Schedule):
    sigma = schedule.sample_batch(x0)
    eps = torch.randn_like(x0)
    return sigma, eps 
```

##### 噪声时间表

在实践中，\(\sigma\) 并不是从区间 \([\sigma_\min, \sigma_\max]\) 均匀采样的，而是将此区间离散化为 \(N\) 个不同值的一个 *\(\sigma\) 时间表*：\(\{ \sigma_t \}_{t=1}^N\)，在训练期间从此列表中均匀采样。我们定义了封装了可能的 `sigmas` 列表的 `Schedule` 类，并在训练过程中从此列表中采样。

```
class Schedule:
    def __init__(self, sigmas: torch.FloatTensor):
        self.sigmas = sigmas
    def __getitem__(self, i) -> torch.FloatTensor:
        return self.sigmas[i]
    def __len__(self) -> int:
        return len(self.sigmas)
    def sample_batch(self, x0:torch.FloatTensor) -> torch.FloatTensor:
        return self[torch.randint(len(self), (x0.shape[0],))].to(x0) 
```

在本教程中，我们将使用下面定义的对数线性时间表：

```
class ScheduleLogLinear(Schedule):
    def __init__(self, N: int, sigma_min: float=0.02, sigma_max: float=10):
        super().__init__(torch.logspace(math.log10(sigma_min), math.log10(sigma_max), N)) 
```

其他常用的时间表包括像素空间扩散模型的 `ScheduleDDPM` 和 Latent Diffusion 模型如 Stable Diffusion 的 `ScheduleLDM`。下面的图表比较了这三种时间表的默认参数。

不同扩散时间表的比较图

##### 玩具示例

在本教程中，我们将从最早的扩散论文之一中使用的玩具数据集开始 [[Sohl-Dickstein et.al. 2015]](https://arxiv.org/abs/1503.03585)，其中 \(\Kset \subset \R^2\) 是从螺旋中抽样的点。我们首先构建并可视化该数据集：

```
dataset = Swissroll(np.pi/2, 5*np.pi, 100)
loader  = DataLoader(dataset, batch_size=2048) 
```

瑞士卷玩具数据集

对于这个简单的数据集，我们可以使用多层感知器（MLP）来实现去噪器：

```
def get_sigma_embeds(sigma):
    sigma = sigma.unsqueeze(1)
    return torch.cat([torch.sin(torch.log(sigma)/2),
                      torch.cos(torch.log(sigma)/2)], dim=1)

class TimeInputMLP(nn.Module):
    def __init__(self, dim, hidden_dims):
        super().__init__()
        layers = []
        for in_dim, out_dim in pairwise((dim + 2,) + hidden_dims):
            layers.extend([nn.Linear(in_dim, out_dim), nn.GELU()])
        layers.append(nn.Linear(hidden_dims[-1], dim))
        self.net = nn.Sequential(*layers)
        self.input_dims = (dim,)

    def rand_input(self, batchsize):
        return torch.randn((batchsize,) + self.input_dims)

    def forward(self, x, sigma):
        sigma_embeds = get_sigma_embeds(sigma)         # shape: b x 2
        nn_input = torch.cat([x, sigma_embeds], dim=1) # shape: b x (dim + 2)
        return self.net(nn_input)

model = TimeInputMLP(dim=2, hidden_dims=(16,128,128,128,128,16)) 
```

MLP 接受 \(x \in \R^2\) 的串联和噪声水平 \(\sigma\) 的嵌入，然后预测噪声 \(\epsilon \in \R^2\)。虽然许多扩散模型使用正弦位置嵌入来表示 \(\sigma\)，但简单的二维嵌入同样有效：

二维 \(\sigma_t\) 嵌入

现在我们具备了训练扩散模型的所有要素。

```
schedule = ScheduleLogLinear(N=200, sigma_min=0.005, sigma_max=10)
trainer  = training_loop(loader, model, schedule, epochs=15000)
losses   = [ns.loss.item() for ns in trainer] 
```

15000 个 epoch 的训练损失，用移动平均平滑

学得的去噪器 \(\eps_\theta(x, \sigma)\) 可以被视为由噪声水平 \(\sigma\) 参数化的向量场，通过为不同的 \(x\) 和 \(\sigma\) 绘制 \(x - \sigma \eps_\theta(x, \sigma)\) 来进行可视化。

对于不同的 \(x\) 和 \(\sigma\)，预测的 \(\hat{x}_0 = x - \sigma \eps_\theta(x, \sigma)\) 的绘图

在上述图中，箭头从每个带噪数据点 \(x\) 指向由噪声水平 \(\sigma\) 预测的“干净”数据点。在高噪声水平下，去噪器倾向于预测数据的均值，但在低噪声水平下，如果其输入 \(x\) 也接近数据，则去噪器预测实际数据点。

我们如何解释去噪器学到了什么，并且如何创建从扩散模型采样的过程？我们将建立扩散模型的理论，然后根据这一理论推导采样算法。

#### 作为近似投影的去噪器

扩散训练过程学习了一个去噪器 \(\eps_\theta(x, \sigma)\)。在[我们的论文](https://arxiv.org/abs/2306.04848)，我们将学习到的去噪器解释为数据流形 \(\Kset\) 的近似投影，并且扩散过程的目标是将到 \(\Kset\) 的距离最小化。这激励我们引入相对误差逼近模型来分析扩散采样算法的收敛性。首先，我们介绍一些距离和投影函数的基本性质。

##### 距离和投影函数

到集合 \(\Kset \subseteq \R^n\) 的*距离函数*定义为

\[\distK(x) := \min \{ \norm{x-x_0} : x_0 \in \Kset \}.\]

\(x \in \R^n\) 的*投影*，表示为 \(\projK(x)\)，是达到这个距离的点集合：

\[\projK(x) := \{ x_0 \in \Kset : \distK(x) = \norm{x-x_0} \}\]

如果 \(\projK(x)\) 是唯一的，则 \(\distK(x)\) 的梯度，即距离函数的最速下降方向，指向这个唯一的投影点：

**命题** 假设 \(\Kset\) 是闭集且 \(x \not \in \Kset\)。如果 \(\projK(x)\) 是唯一的，那么

\[\nabla \frac{1}{2} \distK(x)^2 = \distK(x) \nabla \distK(x) = x-\projK(x).\]

这告诉我们，如果我们可以学习每个 \(x\) 的 \(\nabla \distK(x)\)，我们可以简单地沿着这个方向移动，找到 \(x\) 在 \(\Kset\) 上的投影。学习这个梯度的一个问题是 \(\distK\) 并非在每处都可微，因此 \(\nabla \distK\) 不是一个连续函数。为了解决这个问题，我们引入一个由参数 \(\sigma\) 平滑的平方距离函数，使用 \(\softmin\) 操作符而不是 \(\min\)。

\[\distop^2_\Kset(x, \sigma) := \substack{\softmin_{\sigma^2} \\ x_0 \in \Kset} \norm{x_0 - x}^2 = {\textstyle -\sigma^2 \log\left(\sum_{x_0 \in \Kset} \exp\left(-\frac{\norm{x_0 - x}^2}{2\sigma^2}\right)\right)}\]

来自[[Madan and Levin 2022]](https://arxiv.org/pdf/2108.10480.pdf)的以下图片显示了距离函数及其平滑版本的等高线。

平滑距离函数具有连续梯度

从这张图片中，我们可以看到 \(\nabla \distK(x)\) 指向 \(\Kset\) 中离 \(x\) 最近的点，而 \(\nabla \distop^2(x, \sigma)\) 则指向由 \(x\) 决定的 \(\Kset\) 中点的加权平均。

##### 理想的去噪器

特定噪声水平 \(\sigma\) 的理想或最优去噪器 \(\epsilon^*\) 是训练损失函数的精确最小化器。当数据是有限集合 \(\Kset\) 上的离散均匀分布时，理想的去噪器有一个精确的闭式表达式，如下：

\[\eps^*(x_\sigma, \sigma) = \frac{\sum_{x_0 \in \Kset} (x_\sigma-x_0) \exp(-\norm{x_\sigma-x_0}^2/2\sigma^2)} {\sigma \sum_{x_0 \in \Kset} \exp(-\norm{x_\sigma-x_0}^2/2\sigma^2)}\]

从上述表达式中，我们看到理想去噪器指向 \(\Kset\) 中所有数据点的加权平均，其中每个 \(x_0 \in \Kset\) 的权重决定到 \(x_0\) 的距离。使用这个表达式，我们还可以实现计算上可行的小数据集理想去噪器：

```
def sq_norm(M, k):
    # M: b x n --(norm)--> b --(repeat)--> b x k
    return (torch.norm(M, dim=1)**2).unsqueeze(1).repeat(1,k)

class IdealDenoiser:
    def __init__(self, dataset: torch.utils.data.Dataset):
        self.data = torch.stack(list(dataset))

    def __call__(self, x, sigma):
        x = x.flatten(start_dim=1)
        d = self.data.flatten(start_dim=1)
        xb, db = x.shape[0], d.shape[0]
        sq_diffs = sq_norm(x, db) + sq_norm(d, xb).T - 2 * x @ d.T
        weights = torch.nn.functional.softmax(-sq_diffs/2/sigma**2, dim=1)
        return (x - torch.einsum('ij,j...->i...', weights, self.data))/sigma 
```

对于我们的玩具数据集，我们可以绘制由理想去噪器预测的 \(\epsilon^*\) 方向，以展示不同噪声水平 \(\sigma\) 下的情况：

\(\eps^*(x, \sigma)\) 的方向图，用于不同的 \(x\) 和 \(\sigma\) 值的绘图

从我们的图表中可以看出，对于较大的 \(\sigma\) 值，\(\epsilon^*\) 指向数据的均值，但对于较小的 \(\sigma\) 值，\(\epsilon^*\) 指向最近的数据点。

我们论文的一个见解是，对于固定的 \(\sigma\)，理想去噪器等价于平滑的平方距离函数的梯度：

\*\*定理\*\* 对于所有 \(\sigma > 0\) 和 \(x \in \R^n\)，我们有

\[\frac{1}{2} \nabla_x \distop^2_\Kset(x, \sigma) = \sigma \eps^*(x, \sigma).\]

这告诉我们，通过最小化扩散训练目标函数 \(\Loss(\theta)\) 找到的理想去噪器实际上是平滑的平方距离函数到底层数据流形 \(\Kset\) 的梯度。这种联系是激发我们解释去噪器是近似投影的关键。

##### 相对误差模型

为了分析扩散抽样算法的收敛性，我们引入了一个相对误差模型，该模型表明由去噪器预测的投影 \(x-\sigma \epsilon_{\theta}( x, \sigma)\) 在输入到去噪器的 \(\sigma\) 良好估计 \(\distK(x)/\sqrt{n}\) 时很好地逼近 \(\projK(x)\)。对于常数 \(1 > \eta \ge 0\) 和 \(\nu \ge 1\)，我们假设

\[\norm{ x-\sigma \epsilon_{\theta}( x, \sigma) - \projK(x)} \le \eta \distK(x)\]

当 \((x, \sigma)\) 满足 \(\frac{1}{\nu}\distK(x) \le \sqrt{n}\sigma \le \nu \distK(x)\)。除了上述关于理想去噪器的讨论外，这种误差模型的动机来自以下观察。

*低噪声* 当 \(\sigma\) 很小时，且流形假设成立时，去噪近似于投影，因为大部分添加的噪声与数据流形正交。

当添加的噪声较小时，大部分噪声与流形的切空间正交。在流形假设下，去噪近似于投影。

*高噪声* 当 \(\sigma\) 相对于 \(\Kset\) 的直径较大时，那么预测任何 \(\Kset\) 数据的加权平均的任何去噪器都具有较小的相对误差。

当添加的噪声相对于数据直径较大时，去噪和投影点在同一个方向上。

我们还对图像数据集上预训练的扩散模型的误差模型进行了实证测试。CIFAR-10 数据集足够小，可以计算理想去噪器。我们的实验表明，对于这个数据集，采样轨迹上的精确投影与理想去噪器输出之间的相对误差很小。

理想的去噪器很好地近似了在相对误差模型下对 CIFAR-10 数据集的投影

#### 从扩散模型中采样

给定学习得到的去噪器 \(\epsilon_\theta(x, \sigma)\)，我们如何从中采样以获得 \(\Kset\) 中的点 \(x_0\)？给定嘈杂的 \(x_t\) 和噪声水平 \(\sigma_t\)，去噪器 \(\eps_\theta(x_t, \sigma_t)\) 通过以下方式预测 \(x_0\)：

\[\hat x_0^t := x_t - \sigma_t \eps_\theta(x_t, \sigma_t)\]

从相对误差假设的直觉告诉我们，我们希望从 \((x_T, \sigma_T)\) 开始，其中 \(\distK(x_T)/\sqrt{n} \approx \sigma_T\)。这通过选择 \(\sigma_T\) 相对于 \(\Kset\) 的直径足够大，并且从方差为 \(\sigma_T\) 的 \(N(0, \sigma_T)\) 独立同分布采样的 \(x_T\) 来实现。这确保了 \(x_T\) 离 \(\Kset\) 很远。

尽管 \(\hat x_0^T = x_T - \sigma_T \eps_\theta(x_T, \sigma_T)\) 的相对误差很小，但绝对误差 \(\distK(\hat x_0^T)\) 可能仍然很大，因为 \(\distK(x_T)\) 很大。事实上，在高噪声水平下，理想去噪器的表达告诉我们 \(\hat x_0^T\) 应该接近数据的均值 \(\Kset\)。我们无法通过单次调用去噪器获得接近 \(\Kset\) 的样本。

采样过程根据 \(\sigma_t\) 的调度迭代地调用去噪器。

因此，我们希望通过预定的 \(\sigma_t\) 调度 *迭代地调用去噪器*，以获得序列 \(x_T, \ldots, x_t, \ldots, x_0\)，希望 \(\distK(x_t)\) 随着 \(\sigma_t\) 的变化而减少。

\[x_{t-1} = x_t - (\sigma_t - \sigma_{t-1}) \eps_\theta(x_t, \sigma_t)\]

这正是确定性的 [DDIM 采样算法](https://arxiv.org/abs/2010.02502)，尽管通过变量的改变在不同坐标系中呈现。详情请参见我们论文的 [附录 A](https://arxiv.org/abs/2306.04848)。

#### 作为距离最小化的扩散采样

我们可以将扩散采样迭代解释为在平方距离函数 \(f(x) = \frac{1}{2} \distK(x)^2\) 上的梯度下降。简言之，

**DDIM 是在 \(f(x)\) 上使用步长 \(1- \sigma_{t-1}/\sigma_t\) 近似梯度下降，其中 \(\nabla f(x_t)\) 由 \(\eps_\theta(x_t, \sigma_t)\) 估计得到。**

我们应该如何选择 \(\sigma_t\) 的调度？这决定了我们在采样过程中采取的梯度步数和大小。如果步数太少，\(\distK(x_t)\) 可能不会减少，算法可能不会收敛。另一方面，如果我们采取许多小步，我们需要多次评估去噪器，这是一个计算昂贵的操作。这促使我们定义了可接受的调度。

**定义** 一个*可接受的调度* \(\{ \sigma_t \}_{t=0}^T\) 确保在每次迭代中都满足 \(\frac{1}{\nu} \distK(x_t) \le \sqrt{n} \sigma_t \le \nu \distK(x_t)\)。特别地，几何递减（即对数线性）的 \(\sigma_t\) 序列是一个可接受的调度。

我们的主要定理表明，如果 \(\{\sigma_t\}_{t=0}^T\) 是一个可接受的调度，并且 \(\epsilon_\theta(x_t, \sigma_t)\) 满足我们的相对误差模型，那么可以控制相对误差，采样过程旨在最小化距离收敛。

**定理** 让 \(x_t\) 表示由DDIM生成的序列，并假设对所有 \(x_t\) 都存在 \(\nabla \distK(x)\)，并且 \(\distK(x_T) = \sqrt n \sigma_T\)。那么

1.  \(x_t\) 是通过梯度下降在带有步长 \(1 - \sigma_{t-1}/\sigma_{t}\) 的平方距离函数上生成的

1.  对所有 \(t\)，\(\distK(x_{t})/\sqrt{n} \approx \sigma_{t}\)

回到我们的玩具示例，我们可以通过从原始对数线性调度中进行子采样来找到一个可接受的调度，并按以下方式实现DDIM采样器：

```
class Schedule:
    ...
    def sample_sigmas(self, steps: int) -> torch.FloatTensor:
        indices = list((len(self) * (1 - np.arange(0, steps)/steps))
                       .round().astype(np.int64) - 1)
        return self[indices + [0]]

batchsize = 2000
sigmas = schedule.sample_sigmas(20)
xt = model.rand_input(batchsize) * sigmas[0]
for sig, sig_prev in pairwise(sigmas):
    eps = model(xt, sig.to(xt))
    xt -= (sig - sig_prev) * eps 
```

从20步DDIM中采样

我们看到大多数样本都接近原始数据，但还有改进的空间。一种方法是增加DDIM步数，但这会增加额外的计算成本。接下来，我们使用扩散模型的解释来推导一个更高效的采样器。

#### 使用梯度估计改进采样器

由于 \(\nabla \distK(x)\) 在 \(x\) 和 \(\projK(x)\) 之间是不变的，我们的目标是通过更新来最小化估计误差 \(\sqrt{n} \nabla \distK(x) - \epsilon_{\theta}(x_t, \sigma_t)\)：

\[\bar\eps_t = \gamma \eps_{\theta}(x_t, \sigma_t) + (1-\gamma) \eps_{\theta}(x_{t+1}, \sigma_{t+1})\]

直觉上，此更新使用当前估计来纠正前一步骤中的任何错误：

我们的梯度估计更新步骤

与DDIM采样器相比，这导致更快的收敛，因为我们玩具模型上的样本更接近原始数据。

从20步梯度估计采样器中采样

相对于默认的DDIM采样器，我们的采样器可以被解释为添加动量，导致轨迹有可能超过但更快地收敛。

采样轨迹变化的动量项 \(\gamma\)

实证上，在生成过程中添加噪声也能提高采样质量。为了在保持原始 \(\sigma_t\) 调度的同时实现这一点，我们需要将噪声去噪到较小的 \(\sigma_{t'}\)，然后再添加回噪声 \(w_t \sim N(0, I)\)。

\[x_{t-1} = x_t - (\sigma_t - \sigma_{t'}) \epsilon_\theta(x_t, \sigma_t) + \eta w_t\]

如果我们假设 \(\mathop{\mathbb{E}}\norm{w_t}^2 = \norm{\epsilon_\theta(x_t, \sigma_t)}^2\)，我们应该选择 \(\eta\)，使得更新的范数在期望中保持恒定。这导致了选择 \(\sigma_{t-1} = \sigma_t^\mu \sigma_{t'}^{1-\mu}\)，其中 \(0 \le \mu < 1\)（按定义，\(\sigma_{t'} \le \sigma_{t-1} \le \sigma_t\)），以及 \(\eta = \sqrt{\sigma_{t-1}^2 - \sigma_{t'}^2}\)。当 \(\mu = \frac{1}{2}\) 时，我们确切地恢复了[DDPM抽样器](https://arxiv.org/abs/2006.11239)（参见[我们论文的附录A](https://arxiv.org/abs/2306.04848)进行推导）。

采样轨迹在采样过程中添加不同数量的噪声

我们的梯度估计更新可以与采样过程中添加噪声相结合。总之，我们的完整更新步骤是

\[x_{t-1} = x_t - (\sigma_t - \sigma_{t'}) \bar\eps_t + \eta w_t\]

下面实现了完全泛化DDIM（`gam=1, mu=0`）、DDPM（`gam=1, mu=0.5`）和我们的梯度估计抽样器（`gam=2, mu=0`）。

```
@torch.no_grad()
def samples(model      : nn.Module,
            sigmas     : torch.FloatTensor, # Iterable with N+1 values for N sampling steps
            gam        : float = 1.,        # Suggested to use gam >= 1
            mu         : float = 0.,        # Requires mu in [0, 1)
            xt         : Optional[torch.FloatTensor] = None,
            batchsize  : int = 1):
    xt = model.rand_input(batchsize) * sigmas[0]
    eps = None
    for i, (sig, sig_prev) in enumerate(pairwise(sigmas)):
        eps, eps_prev = model(xt, sig.to(xt)), eps
        eps_av = eps * gam + eps_prev * (1-gam)  if i > 0 else eps
        sig_p = (sig_prev/sig**mu)**(1/(1-mu)) # sig_prev == sig**mu sig_p**(1-mu)
        eta = (sig_prev**2 - sig_p**2).sqrt()
        xt = xt - (sig - sig_p) * eps_av + eta * model.rand_input(batchsize).to(xt)
        yield xt 
```

#### 大规模示例

上述训练代码不仅适用于我们的玩具数据集，还可用于从头开始训练图像扩散模型。请参见[此示例](https://github.com/yuanchenyang/smalldiffusion/blob/main/examples/fashion_mnist.py)，了解在[FashionMNIST数据集上训练以在[此排行榜](https://paperswithcode.com/sota/image-generation-on-fashion-mnist)上获得第二名FID分数的示例：

从在FashionMNIST数据集上训练的扩散模型中采样样本

采样代码可以在不修改的情况下工作，从最先进的预训练潜在扩散模型中进行采样：

```
schedule = ScheduleLDM(1000)
model    = ModelLatentDiffusion('stabilityai/stable-diffusion-2-1-base')
model.set_text_condition('An astronaut riding a horse')
*xts, x0 = samples(model, schedule.sample_sigmas(50))
decoded  = model.decode_latents(x0) 
```

我们可以通过动量项 \(\gamma\) 可视化对高分辨率文本到图像生成的不同效果。

使用稳定扩散进行文本到图像样本生成

#### 其他资源

同样推荐以下有关扩散模型的博客文章：

1.  [什么是扩散模型](https://lilianweng.github.io/posts/2021-07-11-diffusion-models/)从反转马尔可夫过程的离散时间视角引入了扩散模型

1.  [通过估计数据分布梯度进行生成建模](https://yang-song.net/blog/2021/score/)介绍了从反向随机微分方程的连续时间视角引入扩散模型

1.  [注释扩散模型](https://huggingface.co/blog/annotated-diffusion)详细介绍了扩散模型的PyTorch实现

#### 引文

如果您发现本博客有用，请考虑引用我们的论文：

```
@article{permenter2023interpreting,
  title={Interpreting and improving diffusion models using the euclidean distance function},
  author={Permenter, Frank and Yuan, Chenyang},
  journal={arXiv preprint arXiv:2306.04848},
  year={2023}
} 
```
