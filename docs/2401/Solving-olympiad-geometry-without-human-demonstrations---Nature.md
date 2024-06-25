<!--yml

分类：未分类

日期：2024-05-27 14:51:43

-->

# 解决奥林匹克几何问题，无需人类演示 | 自然

> 来源：[https://www.nature.com/articles/s41586-023-06747-5](https://www.nature.com/articles/s41586-023-06747-5)

### 几何表示

通用形式语言，如Lean^([31](/articles/s41586-023-06747-5#ref-CR31 "de Moura, L. & Ullrich, S. in Proc. 28th International Conference on Automated Deduction, CADE 28 (eds Platzer, A. & Sutcliffe, G.) 625–635 (Springer, 2021)."))，目前仍需要大量的基础工作来描述大多数IMO几何问题。我们不直接解决这个挑战，因为它需要深厚的专业知识和超出定理证明方法论范围的实质性研究。为了规避这个障碍，我们转而采用了GEX^([10](/articles/s41586-023-06747-5#ref-CR10 "Chou, S. C., Gao, X. S. & Zhang, J. Z. A deductive database approach to automated geometry theorem proving and discovering. J. Autom. Reason. 25, 219–246 (2000)."))、JGEX^([17](/articles/s41586-023-06747-5#ref-CR17 "Ye, Z., Chou, S. C. & Gao, X. S. in Proc. Automated Deduction in Geometry: 7th International Workshop, ADG 2008 (eds Sturm, T. & Zengler, C.) 189–195 (Springer, 2011)."))、MMP/Geometer^([13](/articles/s41586-023-06747-5#ref-CR13 "Gao, X. S. & Lin, Q. in Proc. Automated Deduction in Geometry: 4th International Workshop, ADG 2002 (ed. Winkler, F.) 44–66 (Springer, 2004)."))和GeoLogic^([19](/articles/s41586-023-06747-5#ref-CR19 "Olšák, M. in Proc. 7th International Conference on Mathematical Software – ICMS 2020 (eds Bigatti, A., Carette, J., Davenport, J., Joswig, M. & de Wolff, T.) 263–271 (Springer, 2020)."))等更专业的语言，这些工作致力于提供一个逻辑和图形环境，用于具有人类非退化性和拓扑假设的合成几何定理。这种语言的示例显示在图[1d,f](/articles/s41586-023-06747-5#Fig1)中。由于其狭窄的表述，所有IMO几何问题的75%可以适应这种表示。在这种类型的几何环境中，每个证明步骤都经过逻辑和数值验证，也可以由人类读者评估，就好像它是由IMO参赛者编写的，这要归功于语言的高度自然语法。为了涵盖更具表现力的代数和算术推理，我们还将整数、分数和几何常量添加到了这种语言的词汇表中。我们不进一步推动几何表示的完整解决方案，因为它是一个单独且极具挑战性的研究课题，需要数学形式化社区的大量投入。

### 抽样一致的定理前提

我们开发了一种类似于JGEX所使用的建设性图建造语言^([17](/articles/s41586-023-06747-5#ref-CR17 "Ye, Z., Chou, S. C. & Gao, X. S. in Proc. Automated Deduction in Geometry: 7th International Workshop, ADG 2008 (eds Sturm, T. & Zengler, C.) 189–195 (Springer, 2011)."))，每次构造一个前提中的对象，而不是自由采样涉及多个对象的许多前提，因此避免生成自相矛盾的前提集。详尽的构造操作列表见扩展数据表[1](/articles/s41586-023-06747-5#Tab2)。这些操作包括构造以某种方式与其他点相关的新点，即共线点，内心/外心等，以及以数字为参数的构造，例如，“构造点X，使得给定的数字*α*，∠ABX = *α*”。人们可以通过更复杂的操作扩展此列表，以描述更具表现力的几何场景，从而改善合成数据多样性和测试集覆盖。更通用和富有表现力的图形生成器语言可以在参考中找到^([32](/articles/s41586-023-06747-5#ref-CR32 "Krueger, R., Han, J. M. & Selsam, D. in Proc. 28th International Conference on Automated Deduction, CADE 28 (eds Platzer, A. & Sutcliffe, G.) 577–588 (Springer, 2021)."))。我们使用了一种足以描述IMO-AG-30中的问题并且能够很好地与符号引擎DD配合工作的更简单的语言。

### 符号推导引擎

引擎的核心功能是推导出新的真实陈述，给定了定理的前提条件。推导可以通过几何规则来实现，比如‘如果X则Y’，其中X和Y是一组几何陈述，比如‘A，B，C共线’。我们使用了结构化DD^([10](/articles/s41586-023-06747-5#ref-CR10 "Chou, S. C., Gao, X. S. & Zhang, J. Z. A deductive database approach to automated geometry theorem proving and discovering. J. Autom. Reason. 25, 219–246 (2000)."),[17](/articles/s41586-023-06747-5#ref-CR17 "Ye, Z., Chou, S. C. & Gao, X. S. in Proc. Automated Deduction in Geometry: 7th International Workshop, ADG 2008 (eds Sturm, T. & Zengler, C.) 189–195 (Springer, 2011)."))，因为它可以在标准非加速硬件上在几秒钟内找到推导闭包。为了进一步增强推导，我们还在AlphaGeometry中内置了通过AR进行推导的能力。AR可以进行角度/比率/距离追踪的证明步骤。AR的详细示例显示在扩展数据表[2](/articles/s41586-023-06747-5#Tab3)中。这样的证明步骤在几何证明中是无处不在的，但并不被几何规则所覆盖。我们扩展了在GeoLogic^([19](/articles/s41586-023-06747-5#ref-CR19 "Olšák, M. in Proc. 7th International Conference on Mathematical Software – ICMS 2020 (eds Bigatti, A., Carette, J., Davenport, J., Joswig, M. & de Wolff, T.) 263–271 (Springer, 2020)."))中实现的高斯消元过程，以在几秒钟内找到所有可能的线性算子的推导闭包。我们的符号推导引擎是DD和AR的错综复杂的整合，我们交替应用它们来扩展已知真实陈述的联合闭包，直到扩展停止。这个过程通常在标准非加速硬件上在几秒钟内至多几分钟内完成。

### 代数推理

在几何定理证明的文献中，尚未对代数推导进行完整处理。例如，在 iGeoTutor^([12](/articles/s41586-023-06747-5#ref-CR12 "Wang, K. & Su, Z. in Proc. Twenty-Fourth International Joint Conference on Artificial Intelligence (IJCAI 2015) (ACM, 2015).")) 中，使用了 Z3 (ref. ^([33](/articles/s41586-023-06747-5#ref-CR33 "de Moura, L. & Bjørner, N. in Proc. 14th International Conference on Tools and Algorithms for the Construction and Analysis of Systems, TACAS 2008 (eds Ramakrishnan, C. R. & Rehof, J.) 337–340 (Springer, 2008)."))) 来处理算术推理，但未涵盖代数操作。DD (ref. ^([17](/articles/s41586-023-06747-5#ref-CR17 "Ye, Z., Chou, S. C. & Gao, X. S. in Proc. Automated Deduction in Geometry: 7th International Workshop, ADG 2008 (eds Sturm, T. & Zengler, C.) 189–195 (Springer, 2011)."))) 通过将代数推导表达为一些有限的推导规则来处理代数推导，因此，它无法表达更复杂的操作，未涵盖算术推理。到目前为止，最一般的处理方式类似于参考文献中的处理方式。^([34](/articles/s41586-023-06747-5#ref-CR34 "Todd, P. A method for the automated discovery of angle theorems. EPTCS 352, 148–155 (2021).")) 中的处理方式，用于仅角度定理的发现，并在 GeoLogic^([19](/articles/s41586-023-06747-5#ref-CR19 "Olšák, M. in Proc. 7th International Conference on Mathematical Software – ICMS 2020 (eds Bigatti, A., Carette, J., Davenport, J., Joswig, M. & de Wolff, T.) 263–271 (Springer, 2020).")) 中实现了角度和比例的处理方式。我们扩展了这个公式以涵盖关于角度、比例和点之间距离的所有推理，以及带有几何常数如‘pi’或‘1:2’的算术推理。代数推理的具体示例在扩展数据表 [2](/articles/s41586-023-06747-5#Tab3) 中给出。

在高层次上，我们首先将输入的线性方程转换为它们系数的矩阵。具体而言，我们创建一个系数矩阵 *A* ∈ *R*^(*M*×*N*)，其中 *N* 是变量的数量，*M* 是输入方程的数量。在几何学中，任何等式都是 *a* − *b* = *c* − *d* ⇔ *a* − *b* − *c* + *d* = 0 的形式。例如，角度相等 ∠ABC = ∠XYZ 被表示为 *s*(AB) − *s*(BC) = *s*(XY) − *s*(YZ)，其中 *s*(AB) 是 AB 与 x 方向的夹角，模 pi。类似地，比例 AB:CD = EF:GH 被表示为 log(AB) − log(CD) = log(EF) − log(GH)，其中 log(AB) 是段 AB 的长度的对数。对于距离，每个变量都是一个（点，线）对，表示特定线上的特定点。

由于所有等式的形式为‘*a* − *b* − *c* + *d* = 0’，我们在每个等式的列中使用值+1、-1、-1、+1来填充与变量*a*、*b*、*c*和*d*相对应的列。对A运行高斯消元会返回一个新的矩阵，在每个列上具有领先的1，本质上表示每个变量作为所有剩余变量的唯一线性组合。例如，假设我们有‘*a* − *b* = *b* − *c*’、‘*d* − *c* = *a* − *d*’和‘*b* − *c* = *c* − *e*’作为输入等式，运行高斯消元过程（在下面的方程中表示为GE）返回以下结果：

$$(\begin{array}{ccccc}a & b & c & d & e\\ 1 & -2 & 1 & 0 & 0\\ -1 & 0 & -1 & 2 & 0\\ 0 & 1 & -2 & 0 & 1\end{array})\,\mathop{\to }\limits^{GE}(\begin{array}{ccccc}a & b & c & d & e\\ 1 & 0 & 0 & -1.5 & 0.5\\ 0 & 1 & 0 & -1 & 0\\ 0 & 0 & 1 & -0.5 & -0.5\end{array})\Rightarrow \{\begin{array}{c}a=1.5d-0.5e\\ b=d\\ c=0.5d+0.5e\end{array}$$

从这个结果中，我们可以通过检查*x*[1] = *x*[2]或*x*[1] − *x*[2] = *x*[2] − *x*[3]或*x*[1] − *x*[2] = *x*[3] − *x*[4]来确定性地和全面地推导所有新的等式，其中{*x*[1], *x*[2], *x*[3], *x*[4]}是所有变量的任意4排列。在上述高斯消元中，例如，AR从三个输入等式中推导出*b* = *d*。为了处理几何常数，如‘0.5π’或‘5:12’，我们将‘π’和‘1’作为默认变量包含在所有系数矩阵中。

### 推理数据库实现

与DD的原始实现不同，我们使用图数据结构来捕获几何的对称性，而不是使用规范形式的字符串。使用图数据结构，我们不仅捕获了函数参数的对称排列，还捕获了等式、共线性和共圆性的传递性。这个图数据结构本身包含了几何规则列表中明确说明的一些推理规则。因此，原始列表中的这些推理规则在探索中没有被使用，但在最终证明序列化为文本时，会被隐式使用并显式列出。

#### 回溯以找到最小证明

每个推导步骤都需要与回溯算法配对，该算法返回推导该步骤的结论语句所需的最小祖先语句集合。这是提取证明图和主文本中描述的最小前提的核心构建块。需要一种最小前提提取算法来避免通过不必要的传递性贡献于证明的多余辅助构造。例如，如果可以通过其他推理链直接获得‘*a* = *c*’，则‘*a* = *b*’和‘*b* = *c*’可能就不是必要的。

### 用于几何规则推导的回溯

为此，我们记录等式传递图。例如，如果推导出 ‘*a* = *b*’, ‘*b* = *c*’, ‘*c* = *d*’ 和 ‘*a* = *d*’，导致节点 *a*, *b*, *c* 和 *d* 连接到相同的‘等式节点’ *e*，我们在 *e* 中维护一个图，其边为 [(*a*, *b*), (*b*, *c*), (*c*, *d*), (*a*, *d*)]。这使得回溯算法能够执行广度优先搜索，以找到 *a*, *b*, *c* 和 *d* 中任意一对变量之间等式传递的最短路径。然而，对于共线性和共圆性，表示更加复杂。在这些情况下，使用具有 3-边或 4-边的超图 *G*(*V*, *E*) 作为等式传递图。回溯现在等价于为节点的目标集合 *S*（三个共线节点或四个共圆节点）找到最小生成树（在以下等式中表示为 MST），其权重是其超边 *e*′ 的并集的基数：

$${\rm{MST}}(S)={\min }_{T\subset E}| {\bigcup }_{{e}^{{\prime} }\subset T}w({e}^{{\prime} })| \,{\rm{s.t.}}\,S\subset T$$

这样的优化是 NP 难的，因为它是从顶点覆盖的决策版本进行的简化。在这种情况下，我们简单地使用贪婪算法来找到最佳努力最小生成树。

### 代数推导的回溯

通过识别，高斯消元的回溯等价于混合整数线性规划问题。给定按照前文描述构造的输入方程的系数矩阵 *A*，以及带有系数向量 *b* ∈ *R*^(*N*) 的目标方程，我们通过定义非负整数决策向量 *x*, *y* ∈ *Z*^(*M*) 并解决以下混合整数线性规划问题，确定 *b* 的最小前提集合：

$$x,y={\min }_{x,y}{\sum }_{i}\left({x}_{i}+{y}_{i}\right)\,{\rm{s.t.}}\,{A}^{{\rm{T}}}\left(x-y\right)=b$$

由 *b* 表示的等式的最小即时父节点集合将是其对应决策值 (*x*[*i*] − *y*[*i*]) 非零的第 *i* 个方程（*A* 的第 *i* 行）。

### 整合 DD 和 AR

DD 和 AR 被交替应用以扩展它们的联合推导闭包。由 DD 产生的输出，由推理规则推导出的新语句组成，被馈送到 AR，反之亦然。例如，如果 DD 推导出 'AB 平行于 CD'，则 AB 和 CD 直线的斜率将更新为 AR 系数矩阵 *A* 中的等变量，在 '代数推理' 部分定义。也就是说，将在 *A* 中添加一个新行，该行在与变量斜率（AB）对应的列上有 '1'，在斜率（CD）的列上有 '−1'。随着 AR 的执行，再次运行高斯消元和混合整数线性规划，生成新的等式作为下一个 DD 迭代的输入。这个循环重复，直到联合推导闭包停止扩展。DD 和 AR 都是确定性过程，仅依赖于定理前提，因此在实现中不需要任何设计选择。

### 证明修剪

尽管对于任何节点来说其直接祖先集是最小的，但这并不能保证完全追溯的依赖子图 *G*(*N*) 和必要前提 *P* 是最小的。在这里，我们将最小性定义为 *G*(*N*) 和 *P* 不能进一步修剪而不失去结论可达性的属性。没有最小性，我们得到许多带有虚构辅助构造的合成证明，与实际证明关系浅薄且完全可以丢弃。为了解决这个问题，我们进行详尽的试错，丢弃每个辅助点的子集，并在较小的前提子集上重新运行 DD+AR 以验证目标可达性。最后，我们返回在所有试验中获得的最小证明。这个证明修剪过程在合成数据生成期间和测试时间的每次成功证明搜索后都进行。

### 并行化数据生成和去重

我们在大量并行的 CPU 工作线程上运行我们的合成数据生成过程，每个线程使用不同的随机种子以减少重复。在将该过程运行在 100,000 个 CPU 工作线程上 72 小时后，我们获得了大约 5 亿个合成证明示例。我们将证明陈述重新格式化为它们的规范形式（例如，对单个术语的参数进行排序以及对同一证明步骤中的术语进行排序等），以避免与自身和测试集之间的浅层重复。最终，我们获得了 1 亿个独特的定理 - 证明示例。其中 900 万个示例涉及至少一项辅助构造。我们在合成数据中找不到 IMO-AG-30 问题。在 JGEX 收集的几何问题集合中^([17](/articles/s41586-023-06747-5#ref-CR17 "Ye, Z., Chou, S. C. & Gao, X. S. in Proc. Automated Deduction in Geometry: 7th International Workshop, ADG 2008 (eds Sturm, T. & Zengler, C.) 189–195 (Springer, 2011)."))，其中主要包含中等难度和众所周知定理的问题，我们在合成数据中发现了近 20 个问题。这表明训练数据涵盖了几何学中相当多的常识，但更复杂定理的空间仍然要大得多。

### 语言模型架构和训练

我们使用 [Meliad](https://github.com/google-research/meliad) 库^([35](/articles/s41586-023-06747-5#ref-CR35 "Hutchins, D., Rabe, M., Wu, Y., Schlag, I. & Staats, C. Meliad. Github                    https://github.com/google-research/meliad                                     (2022).")) 进行转换器训练，采用其基本设置。转换器具有 12 层，嵌入维度为 1,024，八个注意力头和具有 ReLU 激活的交互注意力密集层，维度为 4,096。总体上，转换器具有 1.51 亿个参数，不包括其输入和输出头的嵌入层。我们的定制分词器采用 “word” 模式使用 SentencePiece^([36](/articles/s41586-023-06747-5#ref-CR36 "Kudo, T. & Richardson, J. SentencePiece: a simple and language independent subword tokenizer and detokenizer for neural text processing. Preprint at                    https://arxiv.org/abs/1808.06226                                     (2018).")) 进行训练，并具有 757 个词汇量。我们将最大上下文长度限制为 1,024 个标记，并使用 T5 风格的相对位置嵌入^([37](/articles/s41586-023-06747-5#ref-CR37 "Raffel, C. et al. Exploring the limits of transfer learning with a unified text-to-text transformer. J. Mach. Learn. Res. 21, 5485–5551 (2020)."))。我们还使用了序列打包^([38](/articles/s41586-023-06747-5#ref-CR38 "Kosec, M., Fu, S. & Krell, M. M. Packing: towards 2x NLP BERT acceleration. Preprint at                    https://openreview.net/forum?id=3_MUAtqR0aA                                     (2021)."),[39](/articles/s41586-023-06747-5#ref-CR39 "Krell, M. M., Kosec, M., Perez, S. P. & Iyer, M., Fitzgibbon A. W. Efficient sequence packing without cross-contamination: accelerating large language models without impacting performance. Preprint at                    https://arxiv.org/abs/2107.02027                                     (2022)."))，因为我们的序列中超过 90% 的序列长度都在 200 以下。在训练过程中，我们在注意力前后都应用了 5% 的 dropout^([40](/articles/s41586-023-06747-5#ref-CR40 "Srivastava, N., Hinton, G., Krizhevsky, A., Sutskever, I. & Salakhutdinov, R. Dropout: a simple way to prevent neural networks from overfitting. J. Mach. Learn. Res. 15, 1929–1958 (2014).")) 率。作为硬件加速器，我们使用了 4 × 4 片的 TPUv3（参考^([41](/articles/s41586-023-06747-5#ref-CR41 "Norrie, T. et al. The design process for Google’s training chips: TPUv2 and TPUv3\. IEEE Micro. 41, 56–63 (2021) Feb 9.")））。对于预训练，我们以每个核心 16 的批量大小和余弦学习率调度进行训练，该调度在 10,000,000 步内从 0.01 降至 0.001。对于微调，我们在另外的 1,000,000 步中保持最终的学习率为 0.001。对于没有预训练的设置，我们在 1,000,000 步内将学习率从 0.01 降至 0.001。我们不执行任何超参数调整。这些超参数值要么是选择为一个较大的圆整数（训练步骤），要么是在 Meliad 代码库中提供的默认值。

#### 并行化证明搜索

因为语言模型解码过程返回描述*k*个替代辅助构造的*k*个不同序列，我们对这*k*个选项进行波束搜索，使用每个波束的得分作为其值函数。这种设置在波束间高度可并行化，当有并行计算资源时，可以实现大幅加速。在我们的实验中，我们使用*k* = 512的波束大小，最大迭代次数为16，并且每个节点的分支因子，即解码批处理大小，为32。这是我们的transformer大小在GPU V100内存中可以容纳的最大推理时间批处理大小。扩大这些因素以检查更大的搜索空间的一部分，可能会进一步改善AlphaGeometry的结果。

对于每个问题，我们使用了四个GPU工作器的池，每个都托管了transformer语言模型的副本，以在备选波束之间分配工作，并使用了一组1万个CPU工作器来托管符号求解器，跨30个问题的所有波束共享。这样，早期终止的问题可以为运行时间较长的问题贡献其计算能力的一部分。我们记录了每个单独问题上符号求解器的运行时间，这在设计上在所有波束中保持大致恒定。我们利用这一点和语言模型解码速度来推断每个问题所需的独立并行性，以保持在扩展数据图[1](/articles/s41586-023-06747-5#Fig7)中IMO的不同时间限制之下。

### 数据和搜索的影响

我们对AlphaGeometry在原始训练数据的较小部分（20％，40％，60％和80％）进行了训练，并发现即使在20％的训练数据中，AlphaGeometry仍然解决了21个问题，多于最强基线（DD + AR + 人工设计的启发式）解决的18个问题，如扩展数据图[6a](/articles/s41586-023-06747-5#Fig12)所示。为了研究语言模型在证明搜索中的波束搜索的影响，我们分别在证明搜索过程中减少了波束大小和搜索深度，并在扩展数据图[6c，d](/articles/s41586-023-06747-5#Fig12)中报告了结果。我们发现，当波束大小为8时，即从原始波束大小512减小了64倍，AlphaGeometry仍然可以解决21个问题。通过将搜索深度从16减小到仅两个，同时保持波束大小恒定为512，也可以得到21个问题的类似结果。

### 在更大的测试集上的评估

我们在一个更大的测试集上评估了AlphaGeometry和其他基线，该测试集由231个几何问题组成，在ref ^([17](/articles/s41586-023-06747-5#ref-CR17 "Ye, Z., Chou, S. C. & Gao, X. S. in Proc. Automated Deduction in Geometry: 7th International Workshop, ADG 2008 (eds Sturm, T. & Zengler, C.) 189–195 (Springer, 2011)."))中策划。这一集合涵盖了IMO竞赛之外的更广泛的来源：教科书例子和练习，地区奥林匹克竞赛和著名几何定理；有些甚至比典型的IMO问题更复杂，例如五个圆定理，莫雷定理或萨瓦亚玛和泰博尔定理。结果在扩展数据图[6b](/articles/s41586-023-06747-5#Fig12)中报告。不同方法的总体排名与表[1](/articles/s41586-023-06747-5#Tab1)中保持一致，AlphaGeometry几乎解决了所有问题（98.7%）。最强大的基线DD + AR + 人工设计的启发式解决了92.2%，而以前的最先进技术解决了75%。

#### AlphaGeometry框架及其对其他领域的适用性

AlphaGeometry的神经符号设置的强大之处在于其能够生成辅助构造，这是许多数学领域的重要组成部分。在扩展数据表格[3](/articles/s41586-023-06747-5#Tab4)中，我们在另外四个数学领域中给出了例子，其中提出辅助构造对解决问题至关重要。在扩展数据表格[4](/articles/s41586-023-06747-5#Tab5)中，我们对几何证明和不等式证明进行了逐行比较，以突出它们如何都适用于相同的框架，进而强调了它们之间的相似之处。

我们的论文表明，语言模型可以从合成数据中学习生成辅助构造，其中问题陈述和辅助构造一起随机生成，然后使用回溯算法进行分离以识别依赖差异。具体来说，AlphaGeometry框架需要以下要素：

1.  (1)

    领域对象和定义的实现。

1.  (2)

    一个随机的前提采样器。

1.  (3)

    在实现内操作的符号引擎(s) (1)。

1.  (4)

    符号引擎的回溯过程。

使用这四个要素和主文中描述的算法，可以为任何目标领域生成合成数据。正如我们的论文所示，构建每个要素都存在非平凡的工程挑战。例如，目前的组合学形式化还处于起步阶段，对(1)和(2)构成挑战。此外，为不同领域构建强大的符号引擎需要深入的领域专业知识，对(3)和(4)构成挑战。我们考虑将这一框架应用于更广泛的范围，作为未来的工作，并期待进一步解决这些挑战的创新。

### 定理证明中的变压器

自动定理证明领域的研究可以追溯到上世纪五十年代（参见 ^([6](/articles/s41586-023-06747-5#ref-CR6 "Gelernter, H. L. in Proc. First International Conference on Information Processing (IFIP) 273–281 (UNESCO, 1959)."),[42](/articles/s41586-023-06747-5#ref-CR42 "Gilmore, P. C. A proof method for quantification theory: its justification and realization. IBM J. Res. Dev. 4, 28–35 (1960)."),[43](/articles/s41586-023-06747-5#ref-CR43 "Davis, M. & Putnam, H. A computing procedure for quantification theory. J. ACM. 7, 201–215 (1960)."))），产生了高度优化的一阶逻辑求解器，如 E（参见 ^([44](/articles/s41586-023-06747-5#ref-CR44 "Schulz, S. E – a brainiac theorem prover. AI Commun. 15, 111–126 (2002)."))) 或 Vampire^([45](/articles/s41586-023-06747-5#ref-CR45 "Riazanov, A. & Voronkov, A. in Proc. First International Joint Conference on Automated Reasoning, IJCAR 2001 (eds Goré, R., Leitsch, A. & Nipkow, T.) 376–380 (Springer, 2001)."))。二十一世纪二十年代，深度学习成熟为自动定理证明的新强大工具，展示了在前提选择和证明引导方面的巨大成功^([46](#ref-CR46 "Irving, G. et al. DeepMath - deep sequence models for premise selection. Adv. Neural Inf. Process. Syst.                    https://doi.org/10.48550/arXiv.1606.04442                                     (2016)."),[47](#ref-CR47 "Wang, M., Tang, Y., Wang, J. & Deng, J. Premise selection for theorem proving by deep graph embedding. Adv. Neural Inf. Process. Syst.                    https://doi.org/10.48550/arXiv.1709.09994                                     (2017)."),[48](#ref-CR48 "Loos, S., Irving, G., Szegedy, C. & Kaliszyk, C. Deep network guided proof search. Preprint at                    https://arxiv.org/abs/1701.06972                                     (2017)."),[49](/articles/s41586-023-06747-5#ref-CR49 "Bansal, K., Loos, S., Rabe, M., Szegedy, C. & Wilcox S. in Proc. 36th International Conference on Machine Learning 454–463 (PMLR, 2019)."))，以及 SAT 求解^([50](/articles/s41586-023-06747-5#ref-CR50 "Selsam, D. et al. Learning a SAT solver from single-bit supervision. Preprint at                    https://doi.org/10.48550/arXiv.1802.03685                                     (2019)."))。另一方面，变换器^([18](/articles/s41586-023-06747-5#ref-CR18 "Vaswani, A. et al. Attention is all you need. Adv. Neural Inf. Process. Syst. 30 (2017)."))在各种任务上表现出卓越的推理能力^([51](#ref-CR51 "Saxton, D., Grefenstette, E., Hill, F. & Kohli, P. Analysing mathematical reasoning abilities of neural models. Preprint at                    https://doi.org/10.48550/arXiv.1904.01557                                     (2019)."),[52](#ref-CR52 "Lample, G. & Charton F. Deep learning for symbolic mathematics. Preprint at                    https://doi.org/10.48550/arXiv.1912.01412                                     (2019)."),[53](/articles/s41586-023-06747-5#ref-CR53 "Charton, F., Hayat, A. & Lample, G. Learning advanced mathematical computations from examples. Preprint at                    https://doi.org/10.48550/arXiv.2006.06462                                     (2021)."))。将变换器语言模型应用于定理证明的第一个成功是 GPT-f（参见 ^([15](/articles/s41586-023-06747-5#ref-CR15 "Polu, S. & Sutskever, I. Generative language modeling for automated theorem proving. Preprint at                    https://arxiv.org/abs/2009.03393                                     (2020)."))）。其后的扩展^([2](/articles/s41586-023-06747-5#ref-CR2 "Polu, S. et al. Formal mathematics statement curriculum learning. Preprint at                    https://doi.org/10.48550/arXiv.2202.01344                                     (2023)."),[16](/articles/s41586-023-06747-5#ref-CR16 "Han, J. M., Rute, J., Wu, Y., Ayers, E. W., & Polu, S. Proof artifact co-training for theorem proving with language models. Preprint at                    https://doi.org/10.48550/arXiv.2102.06203                                     (2022)."))进一步发展了这个方向，使机器首次能够解决一些奥林匹克级别的问题。证明搜索算法和在线训练方面的创新^([3](/articles/s41586-023-06747-5#ref-CR3 "Lample, G. et al. Hypertree proof search for neural theorem proving. Adv. Neural Inf. Process. Syst. 35, 26337–26349 (2022)."))也改进了基于变换器的方法，解决了代数和数论中的十个（改编后的）IMO问题。然而，这些进展是建立在大量人类证明示例和由人类设计和策划的独立问题陈述的基础上的。

#### 几何定理证明

几何定理证明演变成一个完全独立的领域。其文献分为两个分支，一个是计算代数方法，另一个是搜索方法。前者自从引入吴氏方法^([21](/articles/s41586-023-06747-5#ref-CR21 "吴伟庭. 关于决策问题和初等几何定理证明的机械化. 《科学通报》, 1978, 21: 159–172."))后，在理论上被广泛认为已解决，该方法可以理论上决定任何等式类型的几何陈述的真值，基于早期作品介绍的专门的代数工具^([54](/articles/s41586-023-06747-5#ref-CR54 "Collins, G. E. in Proc. 2nd GI Conference on Automata Theory and Formal Languages (ed. Barkhage, H.) 134–183 (Springer, 1975)."),[55](/articles/s41586-023-06747-5#ref-CR55 "Ritt, J. F. Differential Algebra (Colloquium Publications, 1950)."))。即使计算代数有着强大的理论保证，但由于其巨大的时间和空间复杂性，实际上其性能可能受到限制^([56](/articles/s41586-023-06747-5#ref-CR56 "Chou, S. C. Proving Elementary Geometry Theorems Using Wu’s Algorithm. Doctoral dissertation, Univ. Texas at Austin (1985)."))。此外，计算代数的方法论对人工智能研究并不感兴趣，后者寻求使用搜索方法证明定理，这是一种更类似于人类且更具通用性的过程。

搜索方法早在上世纪50年代就开始了（参见^([6](/articles/s41586-023-06747-5#ref-CR6 "Gelernter, H. L. in Proc. First International Conference on Information Processing (IFIP) 273–281 (UNESCO, 1959)."),[7](/articles/s41586-023-06747-5#ref-CR7 "Gelernter, H., Hansen, J. R. & Loveland, D. W. in Papers presented at the May 3–5, 1960, western joint IRE-AIEE-ACM computer conference 143–149 (ACM, 1960).")))，并在20世纪持续发展^([57](#ref-CR57 "Nevins, A. J. Plane geometry theorem proving using forward chaining. Artif. Intell. 6, 1–23 (1975)."),[58](#ref-CR58 "Coelho, H. & Pereira, L. M. Automated reasoning in geometry theorem proving with Prolog. J. Autom. Reason. 2, 329–390 (1986)."),[59](#ref-CR59 "Quaife, A. Automated development of Tarski’s geometry. J. Autom. Reason. 5, 97–118 (1989)."),[60](/articles/s41586-023-06747-5#ref-CR60 "McCharen, J. D., Overbeek, R. A. & Lawrence, T. in The Collected Works of Larry Wos 166–196 (2000)."))。随着DD的引入^([10](/articles/s41586-023-06747-5#ref-CR10 "Chou, S. C., Gao, X. S. & Zhang, J. Z. A deductive database approach to automated geometry theorem proving and discovering. J. Autom. Reason. 25, 219–246 (2000)."),[17](/articles/s41586-023-06747-5#ref-CR17 "Ye, Z., Chou, S. C. & Gao, X. S. in Proc. Automated Deduction in Geometry: 7th International Workshop, ADG 2008 (eds Sturm, T. & Zengler, C.) 189–195 (Springer, 2011)."))、区域方法^([61](/articles/s41586-023-06747-5#ref-CR61 "Chou, S. C., Gao, X. S. & Zhang, J. Machine Proofs in Geometry: Automated Production of Readable Proofs for Geometry Theorems (World Scientific, 1994).")) 和全角方法^([30](/articles/s41586-023-06747-5#ref-CR30 "Chou, S. C., Gao, X. S. & Zhang, J. Z. Automated generation of readable proofs with geometric invariants: II. Theorem proving with full-angles. J. Autom. Reason. 17, 349–370 (1996)."))，几何求解器使用的推导规则比Tarski或Hilbert的公理更高级，并且能够证明比在形式语言中操作的定理更多、更复杂的定理。然而，当今的几何定理证明仍然依赖于人类设计的辅助构造的启发式方法^([10](#ref-CR10 "Chou, S. C., Gao, X. S. & Zhang, J. Z. A deductive database approach to automated geometry theorem proving and discovering. J. Autom. Reason. 25, 219–246 (2000)."),[11](#ref-CR11 "Matsuda, N. & Vanlehn, K. GRAMY: a geometry theorem prover capable of construction. J. Autom. Reason. 32, 3–33 (2004)."),[12](#ref-CR12 "Wang, K. & Su, Z. in Proc. Twenty-Fourth International Joint Conference on Artificial Intelligence (IJCAI 2015) (ACM, 2015)."),[13](#ref-CR13 "Gao, X. S. & Lin, Q. in Proc. Automated Deduction in Geometry: 4th International Workshop, ADG 2002 (ed. Winkler, F.) 44–66 (Springer, 2004)."),

#### 定理证明中的合成数据

合成数据长期以来被认为是定理证明中的重要组成部分^([63](#ref-CR63 "吴，Y.，姜，A. Q.，巴，J. & Grosse，R. INT：用于评估定理证明中泛化的不等式基准。预印本地址 https://doi.org/10.48550/arXiv.2007.02924 (2021)."),[64](#ref-CR64 "Zombori，Z.，Csiszárik，A.，Michalewski，H.，Kaliszyk，C. & Urban，J.在Proc. 30th International Conference on Automated Reasoning with Analytic Tableaux and Related Methods (eds Das，A. & Negri，S.) 167–186 (Springer，2021)."),[65](#ref-CR65 "Fawzi，A.，Malinowski，M.，Fawzi，H.，Fawzi，O.学习动态多项式证明。Adv. Neural Inf. Process. Syst. https://doi.org/10.48550/arXiv.1906.01681 (2019)."),[66](/articles/s41586-023-06747-5#ref-CR66 "王，M. & 邓，J.通过学习生成定理来学习证明定理。Adv. Neural Inf. Process. Syst. 33，18146–18157 (2020).")). 最先进的机器学习方法利用专家迭代生成合成证明的课程^([2](/articles/s41586-023-06747-5#ref-CR2 "Polu，S. et al.形式数学语句课程学习。预印本地址 https://doi.org/10.48550/arXiv.2202.01344 (2023)."),[3](/articles/s41586-023-06747-5#ref-CR3 "Lample，G. et al.超树证明搜索用于神经定理证明。Adv. Neural Inf. Process. Syst. 35，26337–26349 (2022)."),[15](/articles/s41586-023-06747-5#ref-CR15 "Polu，S. & Sutskever，I.用于自动定理证明的生成语言建模。预印本地址 https://arxiv.org/abs/2009.03393 (2020).")). 然而，他们的方法仅为人类设计和选择的一组固定的预定义问题生成合成证明。另一方面，我们的方法完全从头开始生成合成问题和证明。相似地，Aygün等^([67](/articles/s41586-023-06747-5#ref-CR67 "Aygün，E. et al.在Proc.第39届国际机器学习会议1198–1210 (PMLR，2022).")) 使用后见之明经验重播生成合成证明^([68](/articles/s41586-023-06747-5#ref-CR68 "Andrychowicz，M. et al.后见之明经验重播。Adv. Neural Inf. Process. Syst. https://doi.org/10.48550/arXiv.1707.01495 (2017).")), 提供一系列定理难度的平滑范围，以帮助学习，与我们的工作类似。然而，AlphaGeometry并未训练在由人类策划的现有猜想上，并且不会从目标定理的证明尝试中学习。他们的方法因此是正交的，可以用来进一步改进AlphaGeometry。与我们的工作最相似的是Firoiu等^([69](/articles/s41586-023-06747-5#ref-CR69 "Firoiu，V. et al.从合成数据训练一阶定理证明器。预印本地址 https://doi.org/10.48550/arXiv.2103.03798 (2021)."))，他们的方法使用前向建议者通过深度优先探索生成合成数据并纯粹基于这些合成数据训练神经网络。另一方面，我们的工作使用广度优先探索，这对获取最小证明和前提是必要的，并使用回溯算法来识别辅助构造，从而引入前向建议者无法提出的新符号和假设。
