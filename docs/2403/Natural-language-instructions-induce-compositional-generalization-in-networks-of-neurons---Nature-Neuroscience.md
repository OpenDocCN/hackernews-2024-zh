<!--yml

category: 未分类

date: 2024-05-29 12:29:31

-->

# 自然语言指令诱导神经网络中的组合泛化 | 自然神经科学

> 来源：[https://www.nature.com/articles/s41593-024-01607-5](https://www.nature.com/articles/s41593-024-01607-5)

### 模型架构

#### 感觉运动-RNN

本文中使用的基础模型架构和任务结构遵循^([18](/articles/s41593-024-01607-5#ref-CR18 "Yang, G. R., Joglekar, M. R., Song, H. F., Newsome, W. T. & Wang, X.-J. Task representations in neural networks trained to perform many cognitive tasks. Nat. Neurosci. 22, 297–306 (2019)."))。所有传感运动单元网络均为门控循环单元（GRU）^([49](/articles/s41593-024-01607-5#ref-CR49 "Chung, J., Gulcehre, C., Cho, K. & Bengio, Y. Empirical evaluation of gated recurrent neural networks on sequence modeling. Preprint at                    https://arxiv.org/abs/1412.3555                                     (2014)."))，使用具有256个隐藏单元的修正线性单元（ReLU）非线性。网络的输入包括（1）感觉输入*X*[t]和（2）任务标识信息*I*[t]。我们将GRU中的隐藏活动初始化为\({h}^{0}\in {{\mathbb{R}}}^{256}\)，其值设置为0.1。所有传感运动单元网络使用相同的隐藏状态初始化，因此我们在网络方程中省略*h*⁰。在每个时间步，一个读出层Linear[out]根据循环隐藏单元*h*[*t*]的活动解码运动活动\(\hat{{y}_{t}}\)如下：

$$\begin{array}{ll}{h}_{t}={{{\rm{SensorimotorRNN}}}}\Big({X}_{t},{I}_{t};{h}_{t-1}\Big)\qquad\qquad{h}_{t}\in {{\mathbb{R}}}^{256}\\ {\hat{y}}_{t}=\sigma \Big({{{{\rm{Linear}}}}}_{{{{\rm{out}}}}}({h}_{t})\Big)\qquad\qquad\qquad\qquad\qquad\quad{\hat{y}}_{t}\in {{\mathbb{R}}}^{33}\end{array}$$

其中*σ*表示sigmoid函数。感觉输入*X*[t]由三个通道组成，两个感觉模态\({x}_{{{\mathrm{mod}}}\,1,t}\)和\({x}_{{{\mathrm{mod}}}\,2,t}\)，以及一个固定通道*x*[fix,*t*]。这两个模态\({x}_{{{\mathrm{mod}}}\,1,t}\)和\({x}_{{{\mathrm{mod}}}\,2,t}\in {{\mathbb{R}}}^{32}\)，这些模态的刺激被表示为在一个一维圆形变量周围的单位偏好方向的活动山峰。对于方向*θ*的输入，给定输入单元*u*[i]的活动与其偏好方向*θ*[i]为

$${u}_{i}=str \times 0.8\exp \left[-0.5 \times {\left(\frac{8| \theta -{\theta }_{i}| }{\pi }\right)}^{2}\right]$$

其中 *s**t**r* 是描述刺激强度的系数。定位通道 \({x}_{{{{\rm{fix}}}},t}\in {{\mathbb{R}}}^{1}\) 是一个单一单元，模拟网络的注视提示。总体而言，感觉输入 \({X}_{t}=({x}_{mod1,t},{x}_{mod2,t},{x}_{fix,t})\in {{\mathbb{R}}}^{65}\)。运动输出 \({\hat{{y}}_{t}}\) 包括一个32维环，表示对输入刺激的方向响应，以及一个单元表示模型的注视，因此 \({\hat{{y}}_{t}}\in {{\mathbb{R}}}^{33}\)。

对于所有模型，任务识别信息\({I}_{t}\in {{\mathbb{R}}}^{64}\)。任务识别信息在试验持续时间内一直存在，并保持不变，即\({I}_{t}={I}_{t{\prime} }\forall t,t{\prime}\)。对于所有模型，任务识别信息 *I*[*t*] 和感觉输入 *X*[*t*] 被连接作为传感运动-RNN的输入。

#### 非语言模型

对于SIMPLENET，我们通过使用Python包scipy.stats.ortho_group构造正交矩阵生成一组64维正交任务规则，并将该矩阵的行分配给每种任务类型。对于STRUCTURENET，我们以相同方式生成十个正交的64维向量，每个向量代表任务集的一个维度（即，对输入最弱与最强方向的响应，以相同或相反方向响应，在第一模态中仅关注刺激等）。任务的规则向量然后是这十个基向量的简单组合。有关结构规则向量的完整描述，请参阅附注[3](/articles/s41593-024-01607-5#MOESM1)。

我们还测试了SIMPLENETPLUS和STRUCTURENETPLUS，它们使用具有128个单元和ReLU非线性的额外隐藏层来处理正交任务规则 *I*[*t*]，生成一个向量 \(\bar{{I}_{t}}\)，该向量作为传感运动-RNN的任务识别信息。

$$\begin{array}{ll}{\bar{{I}_{t}}}^{{\prime} }=\rm{ReLU}({{{{\rm{Linear}}}}}_{{{{\rm{RuleEmb}}}}1}({I}_{t}))&{\bar{{I}_{t}}}^{{\prime} }\in {{\mathbb{R}}}^{128}\\ {\bar{{I}_{t}}}^{{\prime} }=\rm{ReLU}({{{{\rm{Linear}}}}}_{{{{\rm{RuleEmb}}}}2}({I}_{t}^{{\prime} }))&{\bar{{I}_{t}}}^{{\prime} }\in {{\mathbb{R}}}^{128}\\ \bar{{I}_{t}}=\rm{ReLU}({{{{\rm{Linear}}}}}_{{{{\rm{RuleEmb}}}}3}({\bar{{I}_{t}}}^{{\prime} }))&\bar{{I}_{t}}\in {{\mathbb{R}}}^{64}\end{array}$$

这些模型的完整结果包含在附图[4](/articles/s41593-024-01607-5#MOESM1)中。

#### 预训练变换器

我们测试的主要语言模型使用预训练的变压器架构来生成*I*。变压器的预训练目标类型不同，用于调整模型参数。GPT被训练为根据单词上下文预测下一个单词^([9](/articles/s41593-024-01607-5#ref-CR9 "Radford, A. et al. 语言模型是无监督的多任务学习者。OpenAI 1, 9 (2019)."))。GPT (XL)遵循相同的目标，但在更大的数据集上进行更长时间的训练^([50](/articles/s41593-024-01607-5#ref-CR50 "Radford, A. et al. 更好的语言模型及其影响。https://openai.com/blog/better-language-models/ (2019)."))。这两种模型都是完全自回归的。相比之下，BERT接受双向语言输入，并任务是预测出现在输入短语中间的屏蔽单词。此外，BERT还在一个简单的句子预测任务上进行训练，其中模型必须确定训练语料库中输入句子1是否跟在输入句子2后面。基于这个原理，SBERT明确地训练以生成整个句子的固定长度嵌入^([21](/articles/s41593-024-01607-5#ref-CR21 "Reimers, N. & Gurevych, I. Sentence-bert: 使用siamese bert-networks生成句子嵌入。预印本https://arxiv.org/abs/1908.10084 (2019)."))。它采用预训练的BERT网络，并在一个孪生结构中使用它们^([51](/articles/s41593-024-01607-5#ref-CR51 "Bromley, J. et al. 使用“孪生”时延神经网络进行签名验证。Int. J. Pattern Recognit. Artif. Intell. 7, 669–688 (1993)."))，这允许模型的权重按照斯坦福自然语言推理数据集的监督方式进行调整。自然语言推理是一个三分类任务，网络必须推断句子之间的逻辑关系：前提句是否暗示、与假设句矛盾或无关。最后，CLIP被训练以联合嵌入图像和语言^([23](/articles/s41593-024-01607-5#ref-CR23 "Radford, A. et al. 从自然语言监督中学习可传递的视觉模型。在Proc.第38届国际机器学习会议上（eds Marina, M. & Tong, Z.）8748–8763（PMLR，2021年）。"))。它使用来自带标题的图像数据，并要求通过对比损失适当地分类数据集中哪些文本和图像配对是匹配或不匹配的。

重要的是，变压器的自然输出是一个大小为\({\dim }_{{{{\rm{trans}}}}.}\times {{{\mathcal{T}}}}\)的矩阵，这反映了输入序列长度对变压器的固有维度。为了为句子创建嵌入空间，通常会对变压器输出应用池化方法，从而为每个指令生成固定长度的表示。

对于GPT、GPT (XL)、BERT和SBERT，我们使用平均池化方法。假设我们有一个输入指令\({w}_{1}\ldots {w}_{{{{\mathcal{T}}}}}\)。按照预训练语言模型的标准做法，我们的变压器输入在序列的开头和结尾处使用特殊的‘cls’和‘eos’标记进行标记。然后我们计算 *I* 如下：

$$\begin{array}{ll}{h}^{\rm{tran.}}={{{\rm{transformer}}}}\Big({{\mbox{[cls]}}}\,,{w}_{1}\ldots {w}_{{{{\mathcal{T}}}}},\,{{\mbox{[eos]}}}\Big),\qquad{h}^{\rm{tran.}}\in {{\mathbb{R}}}^{{\dim }_{{{{\rm{trans}}}}.}\times {{{\mathcal{T}}}}+2}\\ {h}^{I}={{{\rm{mean}}}}({h}^{\rm{tran.}}),\qquad\qquad\qquad\qquad\qquad\qquad\qquad\qquad{h}^{I}\in {{\mathbb{R}}}^{{\dim }_{{{{{\rm{trans}}}}}.}}\\ I={{{{\rm{Linear}}}}}_{{{{\rm{embed}}}}}({h}^{I})\qquad\qquad\qquad\qquad\qquad\qquad\qquad\qquad\quad I\in {{\mathbb{R}}}^{64}\end{array}$$

我们选择这种平均池化方法主要是因为一项先前的研究^([21](/articles/s41593-024-01607-5#ref-CR21 "Reimers, N. & Gurevych, I. Sentence-bert: Sentence embeddings using siamese bert-networks. Preprint at                    https://arxiv.org/abs/1908.10084                                     (2019).")) 发现这种方法产生了性能最佳的SBERT嵌入。另一种选择是简单地使用‘cls’标记的最终隐藏表示作为整个序列信息的摘要（考虑到BERT架构是双向的，该标记将能够访问整个序列）。

$$\begin{array}{ll}{h}^{\rm{tran.}}={{{\rm{transformer}}}}\Big(\,{{\mbox{[cls]}}}\,,{w}_{1}\ldots {w}_{{{{\mathcal{T}}}}},\,{{\mbox{[eos]}}}\,\Big),\qquad{h}^{\rm{tran.}}\in {{\mathbb{R}}}^{{\dim }_{{{{{\rm{trans}}}}}.}\times {{\,{\mathcal{T}}}}+2}\\ {h}^{I}=({h}_{{{{\rm{cls}}}}}^{\rm{tran.}})\qquad\qquad\qquad\qquad\qquad\qquad\quad\qquad\quad\;\;{h}^{I}\in {{\mathbb{R}}}^{{\dim }_{{{{{\rm{trans}}}}}.}}\end{array}$$

其中 \({h}_{{{{\rm{cls}}}}}^{\rm{tran.}}\) 表示 'cls' 标记的最后隐藏表示。参考 ^([21](/articles/s41593-024-01607-5#ref-CR21 "Reimers, N. & Gurevych, I. Sentence-bert: Sentence embeddings using siamese bert-networks. Preprint at                    https://arxiv.org/abs/1908.10084                                     (2019).")) 的研究发现，这种汇聚方法的表现低于平均汇聚，因此我们在结果中不包括这些替代方案。对于 GPT 和 GPT (XL)，我们还测试了一种汇聚方法，其中序列的固定长度表示取自 'eos' 标记的变压器输出。在这种情况下：

$$\begin{array}{ll}{h}^{\rm{tran.}}={{{\rm{transformer}}}}\Big(\,{{\mbox{[cls]}}}\,,{w}_{1}\ldots {w}_{{{{\mathcal{T}}}}},\,{{\mbox{[eos]}}}\,\Big),\qquad{h}^{\rm{tran.}}\in {{\mathbb{R}}}^{{\dim }_{{{{\rm{trans}}}}.}\times {{\;{\mathcal{T}}}}+2}\\ {h}^{I}=({h}_{{{{\rm{eos}}}}}^{\rm{tran.}}),\qquad\qquad\qquad\qquad\qquad\qquad\qquad\quad\quad{h}^{I}\in {{\mathbb{R}}}^{{\dim }_{{{{\rm{trans}}}}.}}\\ I={{{{\rm{Linear}}}}}_{{{{\rm{embed}}}}}({h}^{I}),\qquad\qquad\qquad\qquad\qquad\qquad\quad I\in {{\mathbb{R}}}^{64}\end{array}$$

我们发现，使用此汇聚方法，GPT 未能达到85%的宽松性能标准，而 GPT (XL) 的表现甚至低于平均汇聚，因此我们在主要结果中省略了这些模型（见附图 [11](/articles/s41593-024-01607-5#MOESM1)）。对于 CLIP 模型，我们使用与原始多模态训练过程相同的汇聚方法，即使用 [cls] 标记的输出，如上所述。

对于所有上述模型，我们还测试了一个版本，其中来自预训练变压器的信息通过具有256个隐藏单元和ReLU非线性的单隐藏层多层感知机传递。我们发现这种操作降低了所有模型的性能，验证了简单的线性嵌入对泛化性能的益处。

对于 GPT、BERT 和 SBERT，\({\dim }_{{{{\rm{trans}}}}.}=768\)，每个模型使用约1亿个参数；对于 SBERT (L)，\({\dim }_{{{{\rm{trans}}}}.}=1,024\)，模型使用约3亿个参数；对于 GPT (XL)，\({\dim }_{{{{\rm{trans}}}}.}=1,600\)，模型使用约15亿个参数；对于 CLIP，\({\dim }_{{{{\rm{trans}}}}.}=512\)，模型使用约6千万个参数。可以在 Huggingface 库中访问完整的 PyTorch 实现，包括所有预训练权重和模型超参数信息（[https://huggingface.co/docs/transformers/](https://huggingface.co/docs/transformers/)）^([52](/articles/s41593-024-01607-5#ref-CR52 "Wolf, T. et al. Transformers: state-of-the-art natural language processing. In Proc. 2020 Conference on Empirical Methods in Natural Language Processing: System Demonstrations (eds Liu, Q. & Schlangen, D.) 38–45 (Association for Computational Linguistics, 2020).")).

#### BoW 模型

对于我们的 Bag-of-Words 模型，指令被表示为二进制激活向量，其大小与指令词汇表相同，其中每个单元表示当前指令中相应单词的包含或排除。对于我们的指令集，∣vocab∣ = 181。然后将此向量通过一个线性层投影到64维空间。

$$\begin{array}{ll}{h}_{i}^{{{{\rm{BoW}}}}}=\left\{\begin{array}{ll}1\quad\,{{\mbox{如果}}}\,\,{w}_{i}\in ({w}_{1}\ldots {w}_{{{{\mathcal{T}}}}})\\ 0\quad\,{{\mbox{否则}}}\,\end{array}\right.\qquad\qquad{h}^{{{{\rm{BoW}}}}}\in {{\mathbb{R}}}^{| \rm{vocab}| }\\ I={{{{\rm{Linear}}}}}_{{{{\rm{embed}}}}}({h}^{{{{\rm{BoW}}}}}),\qquad\qquad\qquad\qquad\qquad\quad I\in {{\mathbb{R}}}^{64}\end{array}$$

#### 空白版语言模型

调整语言模型的最后几层结果表明性能有所提高（图 [2e](/articles/s41593-024-01607-5#Fig2)）。我们测试了另外两个模型，以确定仅训练从感觉运动任务中损失的空白版语言模型是否会改善性能。这些模型包括通过多层感知器传递 BoW 表示和通过随机初始化的 BERT 编码器的一层传递预训练的 BERT 词嵌入。与预训练模型相比，这两个模型的表现较差（附图 [4.5](/articles/s41593-024-01607-5#MOESM1)），证实语言预训练对泛化至关重要。

### 任务集

任务被分为五个相关子组：‘go’、‘decision-making’、‘matching’、‘comparison’和‘duration’。根据任务不同，刺激时段内可能出现多个刺激物。此外，根据任务不同，模型可能需要朝特定方向响应或完全抑制响应。除非另有说明，每个时间步骤独立地向每个输入单元添加均值为零的高斯噪声，并且此噪声的方差从 \({\mathbb{U}}[0.1,0.15]\) 中随机抽取。刺激的时间安排在任务类型之间有所不同。然而，对于所有任务，试验可以分为准备、刺激和响应时段。刺激时段可以细分为三个部分—stim1、delay和stim23—尽管并非所有任务都使用这些明确的部分。每次试验总共持续*T* = 150个时间步骤。设*d**u**r*[epoch]表示给定时期模拟时间步长的持续时间。那么

$$\begin{array}{rcl}&&du{r}_{{{{\rm{response}}}}} \sim\Big\{i| 20 < i\le 25;i\in {\mathbb{N}}\Big\}\\ &&du{r}_{{{{\rm{stim}}}}1},du{r}_{{{{\rm{stim}}}}2} \sim\Big\{i| 37 < i\le 50;i\in {\mathbb{N}}\Big\}\\ &&du{r}_{{{{\rm{delay}}}}} \sim\Big\{i| 15 < i\le 25;i\in {\mathbb{N}}\Big\}\\ &&du{r}_{{{{\rm{prep}}}}.}=150-\Big(du{r}_{{{{\rm{response}}}}}+du{r}_{{{{\rm{stim}}}}1}+du{r}_{{{{\rm{stim}}}}2}+du{r}_{{{{\rm{delay}}}}}\Big)\end{array}$$

对于不使用延迟结构的任务，stim1、stim2 和 delay 时期被组合在一个单一的刺激时期内，其中 \(du{r}_{{{{\rm{stimulus}}}}}=du{r}_{{{{\rm{stim}}}}1}+du{r}_{{{{\rm{stim}}}}2}+du{r}_{{{{\rm{delay}}}}}\)。除非另有说明，在准备和刺激时期内会激活具有恒定强度 *s**t**r*[fix] = 1 的注视提示。例如每个任务的示例试验，请参阅补充图 [13](/articles/s41593-024-01607-5#MOESM1)。

#### ‘Go’ 任务

‘Go’ 任务族包括 ‘Go’、‘RTGo’、‘AntiGo’、‘AntiRTGo’ 和每个任务的特定模态版本，用 ‘Mod1’ 和 ‘Mod2’ 表示。在 ‘Go’ 和 ‘AntiGo’ 任务中，刺激在刺激时期开始时呈现。所呈现刺激的方向是从均匀分布 \([0, 2*π]\) 中随机抽取的，即 \({\theta }_{{{{\rm{stim}}}}} \sim {\mathbb{U}}[0,2\pi ]\)。刺激将以等概率出现在模态 1 或模态 2 中。刺激的强度由 \(st{r}_{{{{\rm{stim}}}}} \sim {\mathbb{U}}[1.0,1.2]\) 给出。在 ‘Go’ 任务中，目标反应与呈现刺激的方向相同，即 \({\theta }_{{{{\rm{stim}}}}}={\theta }_{{{{\rm{target}}}}}\)，而在 ‘AntiGo’ 任务中，响应方向应与刺激方向相反，即 \({\theta }_{{{{\rm{stim}}}}}+\pi ={\theta }_{{{{\rm{target}}}}}\)。对于每个任务的特定模态版本，分别在每个模态中抽取一个刺激方向 \({\theta }_{{{{\rm{stim}}}},{{{\rm{mod}}}}1} \sim {\mathbb{U}}[0,2\pi ]\) 和 \({\theta }_{{{{\rm{stim}}}},{{{\rm{mod}}}}2} \sim {\mathbb{U}}[0,2\pi ]\)，以及特定模态的 Go 类型任务

$${\theta }_{{{{\rm{target}}}}}=\left\{\begin{array}{ll}{\theta }_{{{{\rm{stim}}}},{{{\rm{mod}}}}1} &{{\mbox{如果}}}\,\,\,{{\mbox{Mod1 任务}}}\\ {\theta }_{{{{\rm{stim}}}},{{{\rm{mod}}}}2} &{{\mbox{如果}}}\,\,\,{{\mbox{Mod2 任务}}}\end{array}\right.$$

而对于特定模态的 AntiGo 类型任务

$${\theta }_{{{{\rm{target}}}}}=\left\{\begin{array}{ll}{\theta }_{{{{\rm{stim}}}},{{{\rm{mod}}}}1}+\pi &{{\mbox{如果}}}\,\,\,{{\mbox{Mod1 任务}}}\,\\ {\theta }_{{{{\rm{stim}}}},{{{\rm{mod}}}}2}+\pi &{{\mbox{如果}}}\,\,\,{{\mbox{Mod2 任务}}}\end{array}\right.$$

对于 ‘Go’ 任务的 ‘RT’ 版本，刺激仅在响应时期呈现，注视提示从未熄灭。因此，刺激本身的存在即作为响应提示，模型必须尽快做出响应。否则，刺激会持续整个刺激时期。

#### ‘决策制定’任务

‘决策’类任务包括‘DM’（决策制定）、‘AntiDM’、‘MultiDM’（多感官决策制定）、‘AntiMultiDM’、每个任务的特定模态版本，以及‘DM’和‘AntiDM’的基于置信度的版本。对于这组所有任务，同时呈现两个刺激，并在刺激期内持续存在。它们根据 \({\theta }_{{{{\rm{stim}}}}1} \sim {\mathbb{U}}[0,2\pi ]\) 和 \({\theta }_{{{{\rm{stim}}}}2} \sim {\mathbb{U}}\)\([({\theta }_{{{{\rm{stim}}}}1}-0.2\pi ,{\theta }_{{{{\rm{stim}}}}1}-0.6\pi )\cup ({\theta }_{{{{\rm{stim}}}}1}+0.2\pi ,{\theta }_{{{{\rm{stim}}}}1}+0.6\pi )]\) 进行抽取。应用于两个刺激的基础强度由 \(st{r}_{\rm{base}} \sim {\mathbb{U}}[1.0,1.2]\) 抽取。对比度从离散分布中抽取，使得每个试验中每个方向的刺激强度为 \(st{r}_{{{{\rm{stim}}}}1}=st{r}_{\rm{base}}+c\) 和 \(st{r}_{{{{\rm{stim}}}}2}={str}_{\rm{base}}-c\)。

对于‘DM’任务，

$${\theta }_{{{{\rm{target}}}}}=\left\{\begin{array}{ll}{\theta }_{{{{\rm{stim}}}}1}\quad &\,{{\mbox{若}}}\,\,st{r}_{{{{\rm{stim}}}}1} > st{r}_{{{{\rm{stim}}}}2}\\ {\theta }_{{{{\rm{stim}}}}2}\quad &\,{{\mbox{否则}}}\,\end{array}\right.$$

以及对于‘AntiDM’任务，

$${\theta }_{{{{\rm{target}}}}}=\left\{\begin{array}{ll}{\theta }_{{{{\rm{stim}}}}1}\quad &\,{{\mbox{若}}}\,\,st{r}_{{{{\rm{stim}}}}1} < st{r}_{{{{\rm{stim}}}}2}\\ {\theta }_{{{{\rm{stim}}}}2}\quad &\,{{\mbox{否则}}}\,\end{array}\right.$$

对于这些任务版本，刺激以相等概率呈现在模态 1 或模态 2 中。对于每个任务的多感官版本，刺激方向以相同方式抽取并在两个模态中呈现，使得 \({\theta }_{{{{\rm{stim}}}}1,{{{\rm{mod}}}}1}={\theta }_{{{{\rm{stim}}}}1,{{{\rm{mod}}}}2}\) 和 \({\theta }_{{{{\rm{stim}}}}2,{{{\rm{mod}}}}1}={\theta }_{{{{\rm{stim}}}}2,{{{\rm{mod}}}}2}\)。每个模态的基础强度独立抽取。两个模态的对比度从离散分布中抽取，使得 \({c}_{{{\mathrm{mod}}}\,1},{c}_{{{\mathrm{mod}}}\,2} \sim \left\{0.2,0.175,0.15,0.125,-0.125,-0.15,-0.175,-0.2\right\}\)。若 \(| {c}_{{{\mathrm{mod}}}\,1}| -| {c}_{{{\mathrm{mod}}}\,2}| =0\)，则重新抽取对比度以避免零对比度的试验。若 \({c}_{{{\mathrm{mod}}}\,1}\) 和 \({c}_{{{\mathrm{mod}}}\,2}\) 同号，则重新抽取对比度以确保试验需要在两个模态上进行整合，而不仅仅在单一模态中执行‘DM’任务。目标响应的标准被测量为在两个模态上给定方向的强度之和。因此，对于‘MultiDM’任务，

$${\theta }_{{{{\rm{target}}}}}=\left\{\begin{array}{ll}{\theta }_{{{{\rm{stim}}}}1,{{\mathrm{mod}}}\,1}\quad &\,{{\mbox{if}}}\,\,st{r}_{{{{\rm{stim}}}}1,{{{\rm{mod}}}}1}+st{r}_{{{{\rm{stim}}}}1,{{{\rm{mod}}}}2} > st{r}_{{{{\rm{stim}}}}2,{{{\rm{mod}}}}1}\\&+st{r}_{{{{\rm{stim}}}}2,{{{\rm{mod}}}}2}\\ {\theta }_{{{{\rm{stim}}}}2,{{{\rm{mod}}}}1}\quad &\,{{\mbox{otherwise}}}\,\end{array}\right.$$

对于‘AntiMultiDM’

$${\theta }_{{{{\rm{target}}}}}=\left\{\begin{array}{ll}{\theta }_{{{{\rm{stim}}}}1,{{\mathrm{mod}}}\,1}\quad &\,{{\mbox{if}}}\,\,st{r}_{{{{\rm{stim}}}}1,{{{\rm{mod}}}}1}+st{r}_{{{{\rm{stim}}}}1,{{{\rm{mod}}}}2} < st{r}_{{{{\rm{stim}}}}2,{{{\rm{mod}}}}1}\\&+st{r}_{{{{\rm{stim}}}}2,{{{\rm{mod}}}}2}\\ {\theta }_{{{{\rm{stim}}}}2,{{{\rm{mod}}}}1}\quad &\,{{\mbox{otherwise}}}\,\end{array}\right.$$

任务的模态特定版本的刺激生成方式与任务的多感官版本相同。目标响应的标准与相关模态中的标准‘DM’和‘AntiDM’任务相同。

在基于信心的决策任务（‘ConDM’和‘ConAntiDM’）中，刺激的方向与上述方式相同。刺激以相等的概率显示在模态1或模态2中。每次试验中，*s**t**r*[base] = 1\. 每个试验的对比度和噪声基于在除‘ConDM’和‘ConAntiDM’之外所有任务上训练的SIMPLENET模型的阈值性能。一旦这个模型被训练好，我们会在噪声和对比度水平上建立一个阈值，以便模型在95%正确率下执行‘DM’或‘AntiDM’任务。在训练期间，我们以相等的概率从超过和低于该阈值的对比度和噪声水平中抽取试验。在噪声和对比度水平低于95%正确阈值的试验中，模型必须抑制响应，否则执行决策任务（‘DM’或‘AntiDM’）。

#### ‘比较’任务

我们的比较任务组包括‘COMP1’，‘COMP2’，‘MultiCOMP1’，‘MultiCOMP2’，每个任务的‘Anti’版本，以及‘COMP1’和‘COMP2’任务的模态特定版本。这组任务旨在将基本决策框架扩展到具有更复杂控制需求的环境中。这些任务利用刺激时期中的延迟结构，使得stim1仅在stim1时期出现，随后是一个延迟，最后是stim2\. 这为刺激提供了时间顺序。在‘COMP1’中，模型必须仅在第一个刺激的强度大于第二个刺激时才响应，否则抑制响应

$${\theta }_{{{{\rm{target}}}}}=\left\{\begin{array}{ll}{\theta }_{{{{\rm{stim}}}}1}\quad &\,{{\mbox{if}}}\,\,st{r}_{{{{\rm{stim}}}}1} > st{r}_{{{{\rm{stim}}}}2}\\ {{{\rm{repress}}}}\quad &\,{{\mbox{otherwise}}}\,\end{array}\right.$$

同样，在 ‘COMP2’ 中，如果第二个方向的强度大于第一个方向，模型必须响应第二个方向，否则抑制响应，即

$${\theta }_{{{{\rm{target}}}}}=\left\{\begin{array}{ll}{\theta }_{{{{\rm{stim}}}}2}\quad &\,{{\mbox{如果}}}\,\,st{r}_{{{{\rm{stim}}}}2} > {{{{\rm{str}}}}}_{{{{\rm{stim}}}}1}\\ {{{\rm{抑制}}}}\quad &\,{{\mbox{否则}}}\,\end{array}\right.$$

在任务的 ‘Anti’ 版本中，排序标准相同，除了具有最小强度的刺激，即对于 ‘AntiCOMP1’

$${\theta }_{{{{\rm{target}}}}}=\left\{\begin{array}{ll}{\theta }_{{{{\rm{stim}}}}1}\quad &\,{{\mbox{如果}}}\,\,{{{{\rm{str}}}}}_{{{{\rm{stim}}}}1} < {{{{\rm{str}}}}}_{{{{\rm{stim}}}}2}\\ {{{\rm{抑制}}}}\quad &\,{{\mbox{否则}}}\,\end{array}\right.$$

以及对于‘AntiCOMP2’

$${\theta }_{{{{\rm{target}}}}}=\left\{\begin{array}{ll}{\theta }_{{{{\rm{stim}}}}2}\quad &\,{{\mbox{如果}}}\,\,{{{{\rm{str}}}}}_{{{{\rm{stim}}}}2} < {{{{\rm{str}}}}}_{{{{\rm{stim}}}}1}\\ {{{\rm{抑制}}}}\quad &\,{{\mbox{否则}}}\,\end{array}\right.$$

在多感官设置中，目标方向的标准类似于多感官决策任务，其中强度跨越各个模态。同样，在特定模态版本中，标准仅适用于相关模态中的刺激。这些任务的刺激方向和强度与“决策”系列中类似任务的分布相同。然而，在训练期间，我们确保平衡需要响应的试验和模型必须抑制响应的试验。

#### 'Duration' 任务

‘duration’ 任务系列包括 ‘Dur1’、‘Dur2’、‘MultiDur1’、‘MultiDur2’，每个任务的 ‘Anti’ 版本和 ‘Dur1’ 和 ‘Dur2’ 任务的特定模态版本。这些任务要求模型执行具有额外需求或刺激排序决定响应相关性的时间估计任务。与 ‘comparison’ 任务类似，首先呈现 stim1，然后是延迟，然后是 stim2。对于 ‘Dur1’ 试验

$${\theta }_{{{{\rm{target}}}}}=\left\{\begin{array}{ll}{\theta }_{{{{\rm{stim}}}}1}\quad &\,{{\mbox{如果}}}\,\,du{r}_{{{{\rm{stim}}}}1} > du{r}_{{{{\rm{stim}}}}2}\\ {{{\rm{抑制}}}}\quad &\,{{\mbox{否则}}}\,\end{array}\right.$$

同样，在 ‘Dur2’ 中

$${\theta }_{{{{\rm{target}}}}}=\left\{\begin{array}{ll}{\theta }_{{{{\rm{stim}}}}2}\quad &\,{{\mbox{如果}}}\,\,du{r}_{{{{\rm{stim}}}}2} > du{r}_{{{{\rm{stim}}}}1}\\ {{{\rm{抑制}}}}\quad &\,{{\mbox{否则}}}\,\end{array}\right.$$

在这些任务的 ‘Anti’ 版本中，正确的响应是在满足排序标准的情况下以刺激最短持续时间的方向。因此，对于 ‘AntiDur1’

$${\theta }_{{{{\rm{target}}}}}=\left\{\begin{array}{ll}{\theta }_{{{{\rm{stim}}}}1}\quad &\,{{\mbox{如果}}}\,\,du{r}_{{{{\rm{stim}}}}1} < du{r}_{{{{\rm{stim}}}}2}\\ {{{\rm{抑制}}}}\quad &\,{{\mbox{否则}}}\,\end{array}\right.$$

以及对于 ‘AntiDur2’

$${\theta }_{{{{\rm{target}}}}}=\left\{\begin{array}{ll}{\theta }_{{{{\rm{stim}}}}2}\quad &\,{{\mbox{如果}}}\,\,du{r}_{{{{\rm{stim}}}}2} < du{r}_{{{{\rm{stim}}}}1}\\ {{{\rm{抑制}}}}\quad &\,{{\mbox{否则}}}\,\end{array}\right.$$

跨越这些任务方向是根据 \({\theta }_{{{{\rm{stim}}}}1} \sim {\mathbb{U}}[0,2\pi ]\) 和 \({\theta }_{{{{\rm{stim}}}}2} \sim {\mathbb{U}}[({\theta }_{{{{\rm{stim}}}}1}-0.2\pi ,{\theta }_{{{{\rm{stim}}}}1}-0.6\pi )\cup ({\theta }_{{{{\rm{stim}}}}1}+0.2\pi ,{\theta }_{{{{\rm{stim}}}}1}+0.6\pi )]\) 绘制的。刺激强度根据 \(st{r}_{{{{\rm{stim}}}}1},st{r}_{{{{\rm{stim}}}}2} \sim {\mathbb{U}}[0.8,1.2]\) 绘制。为了设置每个刺激的持续时间，我们首先绘制 \(du{r}_{{{{\rm{long}}}}} \sim\) \(\{i| 35 < i\le 50,i\in {\mathbb{N}}\}\) 和 \(du{r}_{{{{\rm{short}}}}} \sim \{i| 25 < i\le (du{r}_{{{{\rm{long}}}}}-8),i\in {\mathbb{N}}\}\)。在训练过程中，我们确定对于给定任务，哪些试验需要响应，哪些不需要，以平衡抑制和响应试验。然后，我们将 *d**u**r*[long] 和 *d**u**r*[short] 分配给 stim1 或 stim2，以便根据特定的任务类型需要适当的响应。

再次，多感官和特定模态版本的每个任务的正确响应标准遵循类似于“决策制定”和“比较”组中的类似任务，其中多感官版本的任务要求整合每种模态的总持续时间，而特定模态的任务仅需要考虑给定任务模态中的持续时间。对于多感官任务，我们绘制持续时间值 \(du{r}_{{{{\rm{long}}}}} \sim \{i| 75 < i\le 100,i\in {\mathbb{N}}\}\)，然后将这个值分成 *d**u**r*[long0] = *d**u**r*[long ]× 0.55 和 *d**u**r*[long1] = *d**u**r*[long ]× 0.45。我们还绘制一个值 *d**u**r*[short] = *d**u**r*[long] − Δ*d**u**r*，其中 \(\Delta dur \sim \{i| 15 < i\le 25,i\in {\mathbb{N}}\}\)。然后将此值进一步细分为 *d**u**r*[short0] = *d**u**r*[long1] + Δ*d**u**r*[short]，其中 \(\Delta du{r}_{{{{\rm{short}}}}} \sim\) \(\{i| 19 < i\le 15,i\in {\mathbb{N}}\}\)，以及 *d**u**r*[short1] = *d**u**r*[Short] − *d**u**r*[short0]。然后根据任务类型将短时和长时持续时间分配给有序刺激。通过这种方式绘制持续时间，确保像“决策制定”和“比较”组中一样，正确的答案确实要求模型整合两种模态的持续时间，而不仅仅是在给定模态中执行任务以实现正确的响应。

#### ‘匹配’任务

‘matching’ 任务系列包括 ‘DMS’（延迟匹配到刺激物）、‘DNMS’（延迟不匹配到刺激物）、‘DMC’（延迟匹配到类别）和 ‘DMNC’（延迟不匹配到类别）任务。对于所有任务，stim1 在刺激时期开始时呈现，随后是一个延迟，然后是 stim2 的呈现。刺激强度根据 \(st{r}_{{{{\rm{stim}}}}1},st{r}_{{{{\rm{stim}}}}2} \sim {\mathbb{U}}[0.8,1.2]\) 进行抽取。任意给定试验的输入模态以相等概率随机选择。在 ‘DMS’ 和 ‘DNMS’ 任务中，试验分为 ‘matching stim’ 试验或 ‘mismatching stim’ 试验，概率相等。在 ‘matching stim’ 试验中，\({\theta }_{{{{\rm{stim}}}}1} \sim {\mathbb{U}}[0,2\pi ]\) 和 \({\theta }_{{{{\rm{stim}}}}2}={\theta }_{{{{\rm{stim}}}}1}\)。在 ‘mismatch stim’ 试验中，\({\theta }_{{{{\rm{stim}}}}1} \sim {\mathbb{U}}[0,2\pi ]\) 和

$${\theta }_{{{{\rm{stim}}}}2} \sim {\mathbb{U}}[({\theta }_{{{{\rm{stim}}}}1}-0.2\pi ,{\theta }_{{{{\rm{stim}}}}1}-0.6\pi )\cup ({\theta }_{{{{\rm{stim}}}}1}+0.2\pi ,{\theta }_{{{{\rm{stim}}}}1}+0.6\pi )]。$$

对于 ‘DMS’，如果刺激物匹配，模型必须在显示的方向作出响应，否则抑制响应，

$${\theta }_{{{{\rm{target}}}}}=\left\{\begin{array}{ll}{\theta }_{{{{\rm{stim}}}}1}\quad &\,{{\mbox{if}}}\,\,{\theta }_{{{{\rm{stim}}}}1}={\theta }_{{{{\rm{stim}}}}2}\\ {{{\rm{repress}}}}\quad &\,{{\mbox{otherwise}}}\,\end{array}\right.$$

对于 ‘DNMS’，如果两个方向不匹配，模型必须响应于第二个方向，

$${\theta }_{{{{\rm{target}}}}}=\left\{\begin{array}{ll}{\theta }_{{{{\rm{stim}}}}2}\quad &\,{{\mbox{if}}}\,\,{\theta }_{{{{\rm{stim}}}}1}\ne {\theta }_{{{{\rm{stim}}}}2}\\ {{{\rm{repress}}}}\quad &\,{{\mbox{otherwise}}}\,\end{array}\right.$$

‘DMC’ 和 ‘DNMC’ 任务以类似方式组织。刺激输入空间均匀分为两类，即 cat1 = {*θ*: 0 < *θ*≤*π*} 和 cat2 = {*θ*: *π* < *θ*≤2*π*}。对于 ‘DMC’ 和 ‘DNMC’ 任务，试验分为 ‘matching cat.’ 试验或 ‘mismatching cat.’ 试验，概率相等。在 ‘matching cat.’ 试验中，\({\theta }_{{{{\rm{stim}}}}1} \sim {\mathbb{U}}[0,2\pi ]\) 和 \({\theta }_{{{{\rm{stim}}}}2} \sim {\mathbb{U}}({{{\mbox{cat}}}}_{{{{\rm{stim}}}}1})\)，其中 \({\mathbb{U}}({{{\mbox{cat}}}}_{{{{\rm{stim}}}}1})\) 是从 stim1 的类别中均匀抽取的。在 ‘mismatch stim’ 试验中，\({\theta }_{{{{\rm{stim}}}}1} \sim {\mathbb{U}}[0,2\pi ]\) 和 \({\theta }_{{{{\rm{stim}}}}2} \sim {\mathbb{U}}(-{{{\mbox{cat}}}}_{{{{\rm{stim}}}}1})\)，其中 \(-{{{\mbox{cat}}}}_{{{{\rm{stim}}}}1}\) 是与 stim1 相反的类别。对于 ‘DMC’，如果两个刺激物呈现在相同的类别中，模型必须在第一个方向作出响应，否则抑制响应，

$${\theta }_{{{{\rm{target}}}}}=\left\{\begin{array}{ll}{\theta }_{{{{\rm{stim}}}}1}\quad &\,{{\mbox{如果}}}\,\,{{{\mbox{cat}}}}_{{{{\rm{stim}}}}1}={{{\mbox{cat}}}}_{{{{\rm{stim}}}}2}\\ {{{\rm{抑制}}}}\quad &\,{{\mbox{否则}}}\,\end{array}\right.$$

-   对于'DNMC'，如果两个刺激出现在相反的类别中，则模型应对第二个方向作出响应，否则抑制响应。

$${\theta }_{{{{\rm{target}}}}}=\left\{\begin{array}{ll}{\theta }_{{{{\rm{stim}}}}2}\quad &\,{{\mbox{如果}}}\,\,{{{\mbox{cat}}}}_{{{{\rm{stim}}}}1}\ne {{{\mbox{cat}}}}_{{{{\rm{stim}}}}2}\\ {{{\rm{抑制}}}}\quad &\,{{\mbox{否则}}}\,\end{array}\right.$$

### -   目标输出和正确标准

-   试验的目标输出\(y\in {{\mathbb{R}}}^{33\times T}\)要求在刺激时期保持注视在*y*[1] = *y*[fix]，然后在响应时期要么朝着正确方向响应，要么在其余的目标响应单位*y*[2…33]中抑制活动。由于模型应保持注视直至响应，因此注视的目标设置为*y*[fix] = 0.85在准备和刺激时期和*y*[fix] = 0.05在响应时期。当不需要响应时，如在准备和刺激时期以及响应时期的抑制活动中，单位*i*的目标活动为*y*[*i*] = 0.05。或者，当存在响应方向目标时，

$${y}_{i}=0.8\exp \left[-0.5 \times {\left(\frac{8| {\theta }_{{{{\rm{target}}}}}-{\theta }_{i}| }{\pi }\right)}^{2}\right]+0.05$$

-   其中*θ*[*i*]是单位*i*的首选方向。类似于感觉刺激，目标单位的首选方向是从[0, 2*π*]分配给32个响应单元的均匀分布值。

-   模型的正确响应必须在准备和刺激时期维持凝视，即\({\hat{y}}_{{{{\rm{fix}}}}} > 0.5\)。当不需要响应时\({\hat{y}}_{i} < 0.15\)。需要响应时，使用种群矢量方法解码响应活动，并且\({\theta }_{{{{\rm{resp}}}}.}\in ({\theta }_{{{{\rm{target}}}}}-\frac{\pi }{10},{\theta }_{{{{\rm{target}}}}}+\frac{\pi }{10})\)。如果模型未达到任何这些标准，则试验响应为错误。

### -   模型训练

-   再次根据参考文献^([18](/articles/s41593-024-01607-5#ref-CR18 "Yang, G. R., Joglekar, M. R., Song, H. F., Newsome, W. T. & Wang, X.-J. Task representations in neural networks trained to perform many cognitive tasks. Nat. Neurosci. 22, 297–306 (2019)."))^，模型参数按照掩码均方误差损失（mMSE）进行监督式更新，计算模型的运动响应\({\hat{y}}_{1\ldots T}=\hat{y}\)和目标*y*[1…*T*] = *y*每个试验。

$$L={{{\rm{mMSE}}}}(\,y,\hat{y})={\rm{mask}} \times {\Big\langle {\left({\,y}_{t}-{{\hat{y}_{t}}}\right)}^{2}\Big\rangle }_{t}$$

在这里，乘法符号表示逐元素乘法。掩码权重衡量不同试验时期的重要性。在准备和刺激时期，掩码权重设置为 1；在响应时期的前五个时间步中，掩码值设为 0；在响应时期的其余时间步中，掩码权重设为 5。在所有时间步中，固定的掩码值是其他值的两倍。

对于所有模型，在我们的任务集上训练时，我们更新 Θ = {sensorimotor-RNN, Linear[out]}。对于指导模型，在正常训练过程中，我们还会更新 Linear[embed]。我们使用标准的PyTorch工具和Adam优化器进行模型训练。一个时代包括 2,400 个小批次，每个小批次包含 64 个试验。对于所有模型，我们使用与参考文献 ^([18](/articles/s41593-024-01607-5#ref-CR18 "Yang, G. R., Joglekar, M. R., Song, H. F., Newsome, W. T. & Wang, X.-J. Task representations in neural networks trained to perform many cognitive tasks. Nat. Neurosci. 22, 297–306 (2019).")) 相同的初始学习率，*l**r* = 0.001。我们发现，在训练的后期阶段，模型的表现会根据训练过程中呈现的最新任务而振荡，因此我们通过 *γ* = 0.95 的因子将每个时代的学习率衰减，这使得性能能够平稳收敛。根据参考文献 ^([18](/articles/s41593-024-01607-5#ref-CR18 "Yang, G. R., Joglekar, M. R., Song, H. F., Newsome, W. T. & Wang, X.-J. Task representations in neural networks trained to perform many cognitive tasks. Nat. Neurosci. 22, 297–306 (2019)."))，模型训练直到达到所有任务的 95% 阈值（并且至少训练 35 个时代）。我们发现，对于 GPTNET 的训练往往在多感官版本比较任务的性能阈值以下趋于渐近。这一点在各种训练超参数和学习率调度器制度下都成立。因此，我们将 GPTNET 的性能阈值放宽到 85%。对于每种模型类型，我们从五种不同的随机初始化开始训练五个模型。在适用的情况下，结果是这些初始化的平均值。

#### 语言模型微调

在微调模型时，我们允许来自传感器运动训练期间体验到的电机损失梯度，微调变压器语言模型的最终层的权重。在正常训练期间，我们在训练30个周期后对我们指导的模型进行检查点处理。然后，我们将最后三个变压器层添加到可训练参数集中，并将学习率重置为 *l**r* = 1 × 10^−⁴，对于 Θ = {sensorimotor-RNN, Linear[out]} 和 *l**r*^(lang) = 3 × 10^(−4)，对于 Θ^(lang) = {Linear[embed], transformer[−3,−2,−1]} 其中 transformer[−3,−2,−1] 表示相关变压器体系结构的最后三层的参数。我们使用这些降低的学习率来避免完全擦除现有的语言知识。同样对于 RNN 参数，我们发现上述学习率避免了对传感器运动知识的灾难性遗忘，同时也允许 RNN 适应所有模型中更新的语言嵌入。自回归模型对这一过程更为敏感，通常在微调开始时就崩溃。因此，对于 GPTNETXL 和 GPTNET，我们使用 *l**r*^(lang) = 5 × 10^(−5)，这导致了稳健的学习。模型训练直到达到 95% 的训练任务阈值或对于 GPTNET，正确率为 85%。

### 留出测试

在留出测试期间，我们向模型展示了一个被留出训练的任务的 100 批次。对于指导模型，在这个阶段允许更新的唯一权重是 Θ = {sensorimotor-RNN, Linear[out], Linear[embed]}。在此上下文中，SIMPLENET 和 STRUCTURENET 的所有权重都是可训练的。在这种留出设置中，我们发现对于一些表现较差的模型中更困难的任务，我们在训练期间使用的标准超参数导致了新任务的不稳定学习曲线。为了稳定性能并因此在模型之间进行公平比较，我们增加了批量大小至 256。然后，我们从标准学习率 0.001 开始，逐步减少 0.0005，直到所有模型显示出稳健的学习曲线。这导致了一个学习率为 8 × 10^(−4)。所有额外的结果在 [补充信息](/articles/s41593-024-01607-5#Sec36) 第 4 节中都遵循这一过程。

### CCGP 计算

要计算CCGP，我们在一对任务上训练了一个线性解码器，然后在具有类似关系的替代任务对上测试了该解码器。我们将任务分成了八个二分法：‘Go’ versus ‘Anti’，‘Standard’ versus ‘RT’，‘Weakest’ versus ‘Strongest’，‘Longest’ versus ‘Shortest’，‘First Stim.’ versus ‘Second Stim’，‘Stim Match’ versus ‘Category Match’，‘Matching’ versus ‘Non-Matching’ 和 ‘Mod1’ versus ‘Mod2’。例如，‘Go’ versus ‘Anti’ 二分法包括（‘Go’，‘AntiGo’），（‘GoMod1’，‘AntiGoMod1’），（‘GoMod2’，‘AntiGoMod2’），（‘RTGo’，‘AntiRTGo’），（‘RTGoMod1’，‘AntiRTGoMod1’）和（‘RTGoMod2’，‘AntiRTGoMod2’）任务对。对于‘RNN’任务表示，我们提取了250个示例试验的刺激开始时的活动。对于语言表示，我们向我们的语言模型输入相关任务的指令集，并直接分析‘嵌入’层中的活动或在每个变压器层中进行序列平均活动。对于非语言模型，我们简单地分析规则向量空间。解码器的训练和测试条件由跨任务集识别的二分法确定（补充说明 [1](/articles/s41593-024-01607-5#MOESM1)）。为了训练和测试解码器，我们使用了sklearn.svm.LinearSVC Python包。给定任务的CCGP分数是在任务被训练集或测试集中的二分法中获得的平均解码分数。对于主文中报告的模型分数，我们仅计算了对于任务已被从训练中排除的模型的CCGP分数。在补充图 [9](/articles/s41593-024-01607-5#MOESM1) 中，我们报告了在模型已对所有任务进行训练并且对保留任务进行了指令切换的任务上的分数。

对于图 [3e](/articles/s41593-024-01607-5#Fig3)，我们计算了持有任务表现与每个任务的CCGP分数之间的Pearson's *r*相关系数，以及一个*P*-值，检验这些指标是否不相关且正态分布（使用scipy.stats.pearsonr函数）。关于图 [3f](/articles/s41593-024-01607-5#Fig3) 中RNN和嵌入层的完整统计测试可以在补充图 [9](/articles/s41593-024-01607-5#MOESM1) 中找到。请注意，变压器语言模型在随机初始化Sensorimotor-RNNs时使用相同的预训练权重集，因此对于语言模型层，图 [3f](/articles/s41593-024-01607-5#Fig3) 中显示了这些语言模型的绝对分数。

### 条件从句/推断任务分析

我们首先将我们的任务集分成两组（如下所列）：包括条件子句和简单演绎推理组成部分的任务（30个任务）以及那些指令包括简单祈使句的任务（20个任务）。我们计算了每组在每个模型的随机初始化下的泛化性能均值之间的性能差异（图[2f](/articles/s41593-024-01607-5#Fig2)）。我们将这些差异与一个由对任务集进行50次随机洗牌并以相同方式计算差异的空分布进行比较，再次使用双侧异方差*t*检验。因为STRUCUTRENET是一个非语言模型，我们随后比较了STRUCUTRENET与我们指导模型的性能，以分离具有演绎推理成分的任务执行效果与处理具有更复杂条件子句结构指令的效果。所有统计检验结果见补充图[6](/articles/s41593-024-01607-5#MOESM1)。

简单的祈使句任务包括：‘Go’, ‘AntiGo’, ‘RTGo’, ‘AntiRTGo’, ‘GoMod1’, ‘GoMod2’, ‘AntiGoMod1’, ‘AntiGoMod2’, ‘RTGoMod1’, ‘AntiRTGoMod2’, ‘RTGoMod2’, ‘AntiRTGoMod2’, ‘DM’, ‘AntiDM’, ‘MultiDM’, ‘AntiMultiDM’, ‘DMMod1’, ‘DMMod2’, ‘AntiDMMod1’和‘AntiDMMod2’。

条件子句/演绎任务包括：‘ConDM’, ‘ConAntiDM’, ‘Dur1’, ‘Dur2’, ‘MultiDur1’, ‘MultiDur2’, ‘AntiDur1’, ‘AntiDur2’, ‘AntiMultiDur1’, ‘AntiMultiDur2’, ‘Dur1Mod1’, ‘Dur1Mod2’, ‘Dur2Mod1’, ‘Dur2Mod2’, ‘COMP1’, ‘COMP2’, ‘MultiCOMP1’, ‘MultiCOMP2’, ‘AntiCOMP1’, ‘AntiCOMP2’, ‘AntiMultiCOMP1’, ‘AntiMultiCOMP2’, ‘COMP1Mod1’, ‘COMP1Mod2’, ‘COMP2Mod1’, ‘COMP2Mod2’, ‘DMS’, ‘DNMS’, ‘DMC’和‘DMNC’。

### 语言生成训练

#### 自监督语言生成网络训练

我们的语言生成框架灵感来自经典的序列到序列建模，使用具有256隐藏单元的GRU及ReLU非线性函数。在序列的每一步中，一组解码器权重 Linear[words] 尝试从递归单元的隐藏状态中解码下一个标记 *w*[*τ*+1]。Production-RNN 的隐藏状态通过将SBERTNET (L)的时间平均和最大感觉运动活动串联，然后通过权重 Linear[sm] 进行初始化。驱动初始化感觉运动活动的语言指令，反过来成为Production-RNN输出的目标标记集。Production-RNN 的第一个输入始终是一个特殊的句子开头标记，解码器运行直到解码到句子结束标记或输入达到30个标记长度。假设 \({w}_{1,k}\ldots {w}_{{{{\mathcal{T}}}},k}\in {\rm{Instruc{t}}}_{k}^{i}\) 是指令 *k* 中的标记序列，其中 *k* 属于任务 *i* 的指令集，而 *X*^(*i*) 是任务 *i* 的试验感觉输入。为简洁起见，我们用Embed()表示语言模型嵌入指令的过程（见“预训练转换器”）。在第 *τ*^(th) 位置的解码标记 \({\hat{w}}_{\tau ,k}\) 如下确定

$$\begin{array}{ll}{h}_{T}^{sm}={{{\rm{SensorimotorRNN}}}}\left({X}^{i},Embed\left({w}_{1,k}\ldots {w}_{{{{\mathcal{T}}}},k}\right)\right)\quad\quad{h}_{T}^{sm}\in {{\mathbb{R}}}^{T\times 256}\\ sm\_out=\left.\right({{{{\rm{mean}}}}}_{T}\left({h}_{T}^{sm}\right),\mathop{\max }\limits_{T}\left({h}_{T}^{sm}\right)\quad\quad{sm}\_{out}\in {{\mathbb{R}}}^{512}\\ \overline{{h}_{0}^{{{{\rm{decoder}}}}}}={{{\rm{relu}}}}\left({{{{\rm{Linear}}}}}_{{{{\rm{sm}}}}}(sm\_out)\right)\quad\quad\overline{{h}_{0}^{{{{\rm{decoder}}}}}}\in {{\mathbb{R}}}^{256}\\ {h}_{0}^{{{{\rm{decoder}}}}}={{{\rm{Dropout}}}}\left(\overline{{h}_{0}^{{{{\rm{decoder}}}}}}\right)\;\;\;\quad\quad\quad\quad\quad\quad{h}_{0}^{{{{\rm{decoder}}}}}\in {{\mathbb{R}}}^{256}\\ {h}_{\tau }^{{{{\rm{decoder}}}}}={{{\rm{ProductionRNN}}}}\left({\hat{w}}_{1,k}\ldots {\hat{w}}_{\tau -1,k};{h}_{0}^{{{{\rm{decoder}}}}}\right),\quad\quad{h}_{\tau }^{{{{\rm{decoder}}}}}\in {{\mathbb{R}}}^{256}\\ {p}_{{\hat{w}}_{\tau ,k}}={{{\rm{softmax}}}}\left({{{{\rm{Linear}}}}}_{{{{\rm{words}}}}}\left({h}_{\tau ,k}^{{{{\rm{decoder}}}}}\right)\right)\quad\quad{p}_{{\hat{w}}_{\tau ,k}}\in {{\mathbb{R}}}^{| vocab| },\\ {\hat{w}}_{\tau ,k}={{{\rm{argmax}}}}\left({p}_{{\hat{w}}_{\tau ,k}}\right)\end{array}$$

模型参数 Θ^(production) = {Linear[sm], Linear[words], Production-RNN} 使用了传感运动-RNN 的输入 *w*[*τ*,*k*] 和 \({p}_{{\hat{w}}_{\tau ,i}}\) 之间的交叉熵损失进行训练。我们训练了 80 个 epoch，每个 epoch 包含 2,400 个批次，每个批次包含 64 个试验，并且任务类型被随机交错。我们发现，使用初始学习率 0.001 有时会导致模型在训练早期发散，因此我们选择了学习率为 1× 10^(−4)，这样可以保持稳定的早期训练。为了减轻在传感运动训练中检测到的类似振荡问题，我们还按每个 epoch 将学习率衰减 *γ* = 0.99。此外，使用了一个 dropout 层，dropout 率为 0.05，改善了性能。我们还使用了一种教师强迫课程，其中在某些比例的训练批次中，我们在每个时间步输入地面真实的指令标记 *w*[*τ*,*k*]，而不是模型解码的词 \({\hat{w}}_{\tau ,k}\)。每个 epoch，教师强迫比例 \({\rm{teacher}}\,{{\mbox{\_}}}{\rm{forcing}}{{\mbox{\_}}}\) \({\rm{ratio}}=0.5 \times \frac{80-{{{\rm{epoch}}}}}{80}\)。

#### 使用运动反馈获取嵌入层活动

对于任务*i*，我们寻求优化一组嵌入活动向量\({E}^{i}\in {{\mathbb{R}}}^{64}\)，使得当它们作为任务识别信息输入时，模型能够执行所需的任务。关键在于，我们冻结所有模型权重 Θ = {sensorimotor-RNN, Linear[out], Linear[embedding]}，只根据运动输出上的标准监督损失更新 *E*^(*i*)。为了概念上的清晰，GRU 对先前隐藏状态 *h*[*t*−1] 的依赖在下面的方程中被隐含地表示。

$$\begin{array}{rcl}{\hat{y}}{\,}^{i}&=&\sigma \Big({{{{\rm{Linear}}}}}_{{{{\rm{out}}}}}\left({{{\rm{SensorimotorRNN}}}}({X}^{\,i},{E}^{i})\right)\Big)\\ L&=&{\rm{mMSE}}(\;y,\hat{y})\end{array}$$

我们为每个任务优化了一组包含 25 个嵌入向量，同样使用了 Adam 优化器。在这里，优化空间中有许多对应于相关任务嵌入的次优局部最小值。因此，我们使用了初始学习率 *l**r* = 0.05，并且每个 epoch 时将其衰减 *γ* = 0.8。这比较高的学习率导致了比较健壮的学习。一个 epoch 包含 800 个批次，每个批次包含 64 个数据点，我们最少训练 1 个 epoch 或者直到在‘DMC’和‘DNMC’任务上达到 90% 或 85% 的阈值性能为止。

#### 生成任务指令

要生成任务说明，我们只需在传感运动-RNN的输入中使用集合 *E*^(*i*) 作为任务识别信息，并使用Production-RNN基于由 *E*^(*i*) 驱动的感觉运动活动输出说明。对于每个任务，我们使用嵌入向量集合为每个任务生成50条说明。我们对传感运动-RNN的5个初始化重复此过程，从而产生5个不同的语言生成网络和5个不同的学习嵌入向量集。每个任务的报告结果是在这5个网络上进行平均。对于混淆矩阵（图[5d](/articles/s41593-024-01607-5#Fig5)），我们报告了解码说明在给定任务或新说明的训练说明集中的平均百分比。合作模型性能（图[5e](/articles/s41593-024-01607-5#Fig5)）针对每个网络初始化通过测试每个可能的4个合作网络并对这些结果进行平均来计算。

### 样本大小/随机化

没有使用统计方法来预设样本大小，但根据参考文献 ^([18](/articles/s41593-024-01607-5#ref-CR18 "Yang, G. R., Joglekar, M. R., Song, H. F., Newsome, W. T. & Wang, X.-J. Task representations in neural networks trained to perform many cognitive tasks. Nat. Neurosci. 22, 297–306 (2019)."))，我们对每个测试的语言模型使用了五种不同的随机权重初始化。权重的随机化是在Python和PyTorch软件包中自动进行的。鉴于这种自动化的权重随机化，我们在研究中没有使用任何蒙眼程序。没有从分析中排除任何数据。

### 软件

所有模拟和数据分析均在Python 3.7.11中完成。使用PyTorch 1.10实现和训练模型（包括Adam优化器实现）。使用Transformers 4.16.2实现语言模型，所有预训练语言模型权重均来自Huggingface库（[https://huggingface.co/docs/transformers/](https://huggingface.co/docs/transformers/)）。我们还使用了scikit-learn 0.24.1和scipy 1.7.3执行分析。

### 汇报摘要

关于研究设计的进一步信息，请参阅与本文相关的[自然出版物报告摘要](/articles/s41593-024-01607-5#MOESM2)。
