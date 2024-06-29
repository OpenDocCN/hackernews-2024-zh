<!--yml

category: 未分类

date: 2024-05-27 14:38:08

-->

# Sensors | Free Full-Text | Text-Guided Image Editing Based on Post Score for Gaining Attention on Social Media

> 来源：[https://www.mdpi.com/1424-8220/24/3/921](https://www.mdpi.com/1424-8220/24/3/921)

## 1\. Introduction

随着互联网技术的显著进步，社交媒体已成为我们日常生活中的重要组成部分。社交媒体提供了各种互动，如与其他用户交流，寻找和提供新闻，买卖产品和广告。未来预计社交媒体用户数量还会进一步增加 [

[1](#B1-sensors-24-00921)

], 以及社交媒体内的互动潜力对社会可能产生重大影响。Instagram 近年来增长迅速，截至2023年，月活跃用户超过13亿人 [

[2](#B2-sensors-24-00921)

]. Instagram 是一个专门发布图片和视频的平台，允许用户“喜欢”并评论帖子。Instagram 的非凡成功证实了皮尤研究中心最近的一份报告 (

[https://www.pewresearch.org/](https://www.pewresearch.org/)

(访问时间：2023年11月20日)) 指出，照片和视频已成为在线的主要社交货币 [

[3](#B3-sensors-24-00921)

]. 换句话说，发布在Instagram上的图片具有重要的价值。为增强这种价值，有更多编辑图像的机会，自动图像编辑的研究变得愈发重要 [

[4](#B4-sensors-24-00921)

,

[5](#B5-sensors-24-00921)

,

[6](#B6-sensors-24-00921)

].

代表性的自动图像编辑方法包括图像着色 [

[7](#B7-sensors-24-00921)

], 图像修补 [

[8](#B8-sensors-24-00921)

], 以及风格转换 [

[9](#B9-sensors-24-00921)

], 有望消除图像编辑中的繁琐人工工作。这些方法通常用于单一模式转换。例如，风格转换方法可以自动将输入图像中的风格转换为特定艺术家的风格，但使用单一模型转换不同艺术家的风格及反映用户请求到编辑图像中存在限制。为解决这些问题，提出了几种用户友好的方法，称为文本引导的图片编辑 [

[10](#B10-sensors-24-00921)

,

[11](#B11-sensors-24-00921)

,

[12](#B12-sensors-24-00921)

,

[13](#B13-sensors-24-00921)

,

[14](#B14-sensors-24-00921)

,

[15](#B15-sensors-24-00921)

]. 随着StyleGAN等生成模型的巨大成功 [

[16](#B16-sensors-24-00921)

] 和扩散模型 [

[17](#B17-sensors-24-00921)

] 近年来，文本引导的图片编辑已成为热门话题。

尽管基于扩散模型的文本引导图像编辑达到了特别高的性能，用户必须花费精力设计如何提供文本提示以获得期望的结果。具体来说，如下所示

[图1](#sensors-24-00921-f001)

，即使文本提示具有相同的含义，文本引导图像编辑的结果也会因文本提示的表达方式不同而不同。用户需决定哪个结果最符合编辑后图像的预期用途。当编辑后图像的预期用途是发布到社交媒体时，对于没有社交媒体营销知识的用户来说，决定哪个结果将吸引更多观众的注意力是困难的。如下所示

[图2](#sensors-24-00921-f002)

，目前尚不清楚之前的文本引导图像编辑方法的结果在发布到社交媒体时会引起多少关注。此外，如下所示

[图2](#sensors-24-00921-f002)

，通过在考虑社交媒体上的关注的同时进行文本引导图像编辑，可以使编辑后的图像展示给更广泛的观众。综上所述，需要构建一种文本引导图像编辑方法，可以生成能够在社交媒体上引起关注的编辑后图像。

这里有很多关于社交媒体营销中帖子和影响者分类的研究 [

[18](#B18-sensors-24-00921)

，

[19](#B19-sensors-24-00921)

，

[20](#B20-sensors-24-00921)

，

[21](#B21-sensors-24-00921)

]。在帖子分类中，有几项工作 [

[18](#B18-sensors-24-00921)

，

[19](#B19-sensors-24-00921)

] 关注分类（例如时尚、旅行和食品）和病毒性（即某篇帖子在Twitter上获得比其他帖子更多的互动（目前为X），从而获得相对较多用户的关注 [

[19](#B19-sensors-24-00921)

）在社交媒体上的帖子。特别是分类帖子病毒性的研究对于从文本引导图像编辑的多个结果中选择发布到社交媒体的编辑图像很有用。然而，由于传统方法 [

[19](#B19-sensors-24-00921)

] 仅从发布的文本预测病毒性，无法预测编辑后图像在社交媒体上的关注。因此，需要构建一个新模型，该模型可以预测基于发布的图像以及发布的文本在社交媒体上的观众关注。

鉴于上述情况，本文提出了一种考虑社交媒体反应的新型文本引导图像编辑方法。所提方法的目标是为用户提供根据文本提示编辑过的图像，并且能够在社交媒体上吸引更多观众的注意。这里，观众的关注程度定义为根据帖子上的点赞和评论数量计算的参与率。所提方法的关键思想是引入一种新模型，用于从发布的图像和文本中预测代表社交媒体参与率的帖子分数，从而生成能够吸引更多观众关注的编辑过的图像。为了构建这一新模型，基于先前研究的工作[

[22](#B22-sensors-24-00921)

,

[23](#B23-sensors-24-00921)

]，从计算机视觉的角度分析了发布图像的内容或美学与参与率之间的关系，所提方法除了关注发布图像的美学和类别外，还关注发布图像和文本的特征，并计算帖子分数。然后，所提方法基于近年来备受关注的大语言模型，获得用户给定的几个类似于文本提示的其他表达方式。我们应用预训练的文本引导图像编辑方法，从每个文本提示生成编辑过的图像。使用多个文本提示可以增加获得在社交媒体上吸引更多观众注意的编辑过图像的可能性，同时执行用户期望的图像编辑。在这些方法中，通过利用预测代表参与率的帖子分数的新模型，我们最终选择了在社交媒体上能够获得更多观众关注的编辑过的图像。据我们所知，这是首个考虑社交媒体观众反应的文本引导图像编辑方法。所提方法可以为用户提供具有潜力获得最高参与率的编辑过的图像，并有望减轻对社交媒体营销知识有限的用户创建帖子的负担。

本文的其余部分组织如下。我们介绍了社交媒体营销、图像文本匹配和文本引导图像编辑相关工作

[第2节](#sec2-sensors-24-00921)

。在

[第3节](#sec3-sensors-24-00921)

，然后解释了考虑社交媒体反应进行文本引导图像编辑的提出方法。在

[第4节](#sec4-sensors-24-00921)

，作为初步验证，我们验证了预测帖子分数的提出模型的准确性。

[第5节](#sec5-sensors-24-00921)

展示了验证所提方法有效性的广泛实验结果。最后，在

[第6节](#sec6-sensors-24-00921)

。

## 3\. 基于帖子分数的文本引导图像编辑以获取社交媒体关注度

我们展示了提出方法的架构在

[图 3](#sensors-24-00921-f003)

。在提出的方法中，我们首先获得文本提示的四个释义表达

P

。然后，提出的方法应用预训练的文本引导图像编辑方法（即Instruct-Pix2Pix）并生成四张编辑后的图像。最后，我们利用提出的模型预测帖子得分并选择编辑后的图像

<math display="inline"><semantics><msup><mi>I</mi> <mo>′</mo></msup></semantics></math>

从所有文本引导的图像编辑方法的结果中选择最高帖子得分。

在提出的模型中，我们从发布的图像和文本计算反映社交媒体上参与率的帖子得分，如图所示

[图 4](#sensors-24-00921-f004)

。具体来说，基于先前的工作[

[22](#B22-sensors-24-00921)

。

[23](#B23-sensors-24-00921)

]分析了发布图像的内容或美学与参与率之间的关系，除了发布图像的特征和文本外，我们还关注发布图像的美学和类别，并计算帖子得分。我们详细描述了预测帖子得分的提出模型。

[第 3.1 节](#sec3dot1-sensors-24-00921)

。然后，应用该模型的提出方法的整体流程在

[第 3.2 节](#sec3dot2-sensors-24-00921)

。

#### 3.1\. 在社交媒体上计算反映参与率的帖子得分

本节解释了接收图像的提出模型

我

和一段文本

T

作为输入，并在其发布到社交媒体上时预测反映参与率的帖子得分。如图所示

[图 4](#sensors-24-00921-f004)

，为了预测帖子得分，提出的模型使用三个特征（1）从图像中提取的多模态特征

我

和文本

T

，（2）图像的美学特征

我

，和（3）一个类别特征（例如时尚、旅行和食物）。

**图 4.** 预测帖子得分的提出模型细节。为了计算帖子得分，提出的模型使用三个集成特征<math display="inline"><semantics><mi mathvariant="bold-italic">h</mi></semantics></math>预测参与率的类概率。

**图 4.** 预测帖子得分的提出模型细节。为了计算帖子得分，提出的模型使用三个集成特征<math display="inline"><semantics><mi mathvariant="bold-italic">h</mi></semantics></math>预测参与率的类概率。

#### 3.1.1\. 使用多种特征计算帖子得分

为了获得多模态特征，我们利用对比语言-图像预训练（CLIP）[

[37](#B37-sensors-24-00921)

]。CLIP包括图像编码器的两个神经网络

<math display="inline"><semantics><mrow><msub><mi>CLIP</mi> <mi>image</mi></msub> <mrow><mo>(</mo> <mo>·</mo> <mo>)</mo></mrow></mrow></semantics></math>

和一个文本编码器

<math display="inline"><semantics><mrow><msub><mi>CLIP</mi> <mi>text</mi></msub> <mrow><mo>(</mo> <mo>·</mo> <mo>)</mo></mrow></mrow></semantics></math>

。CLIP在非常大的图像-文本对数据集上进行训练，从而在高度表达的空间中提取图像和文本特征。提出的方法获得特征

<math display="inline"><semantics><mrow><msup><mrow><mi mathvariant="bold-italic">f</mi></mrow> <mi>I</mi></msup> <mo>∈</mo> <msup><mi mathvariant="double-struck">R</mi> <mi>D</mi></msup></mrow></semantics></math>

和

<math display="inline"><semantics><mrow><msup><mrow><mi mathvariant="bold-italic">f</mi></mrow> <mi>T</mi></msup> <mo>∈</mo> <msup><mi mathvariant="double-struck">R</mi> <mi>D</mi></msup></mrow></semantics></math>

从图像中提取

我

和文本

T

在CLIP空间中如下：

<math display="block"><semantics><mtable displaystyle="true"><mtr><mtd columnalign="right"><mrow><msup><mrow><mi mathvariant="bold-italic">f</mi></mrow> <mi>I</mi></msup> <mo>=</mo> <msub><mi>CLIP</mi> <mi>image</mi></msub> <mrow><mo>(</mo> <mi>I</mi> <mo>)</mo></mrow> <mo>,</mo></mrow></mtd></mtr></mtable></semantics></math>

<math display="block"><semantics><mtable displaystyle="true"><mtr><mtd columnalign="right"><mrow><msup><mrow><mi mathvariant="bold-italic">f</mi></mrow> <mi>T</mi></msup> <mo>=</mo> <msub><mi>CLIP</mi> <mi>text</mi></msub> <mrow><mo>(</mo> <mi>T</mi> <mo>)</mo></mrow> <mo>,</mo></mrow></mtd></mtr></mtable></semantics></math>

推导出图像和文本特征后，我们连接这些特征并计算最终的多模态特征

<math display="inline"><semantics><mrow><mi mathvariant="bold-italic">m</mi> <mo>∈</mo> <msup><mi mathvariant="double-struck">R</mi> <mrow><mn>2</mn> <mi>D</mi></mrow></msup></mrow></semantics></math>

如下：

这种多模态特征

<math display="inline"><semantics><mi mathvariant="bold-italic">m</mi></semantics></math>

是预测帖子分数的重要因素，因为它提供了图像

我

和文本

T

包含在帖子中。

以前的工作[

[23](#B23-sensors-24-00921)

]清除发布图像美学与参与率之间的关系，我们计算图像的美学特征

我

预测帖子分数。为了计算美学特征，我们应用了神经图像评估（NIMA）模型 [

[54](#B54-sensors-24-00921)

]。NIMA可以使用卷积神经网络预测图像美学的人类意见分布。NIMA模型以图像作为输入，并输出代表质量评分的10维softmax概率分布。我们使用这个分布，它是通过输入图像输出的

我

进入NIMA模型，作为审美特征

<math display="inline"><semantics><mrow><mi mathvariant="bold-italic">n</mi> <mo>∈</mo> <msup><mi mathvariant="double-struck">R</mi> <mn>10</mn></msup></mrow></semantics></math>

在提出的模型中预测帖子分数。这个特征

<math display="inline"><semantics><mi mathvariant="bold-italic">n</mi></semantics></math>

可以代表图像的美学

我

，这可能对帖子的整体质量有重要影响。

考虑到帖子类别与参与率之间的关系 [

[22](#B22-sensors-24-00921)

]，我们还专注于一个类别特征。遵循[

[18](#B18-sensors-24-00921)

]，我们区分八类帖子：美容、家庭、时尚、健身、食品、内饰、宠物和旅行。然而，由于手动将帖子标记为这八类需要大量的工作量，所以提出的模型利用了CLIP的能力。具体来说，我们首先计算这八个文本的文本特征

<math display="inline"><semantics><msubsup><mrow><mo>{</mo> <msub><mi>C</mi> <mi>i</mi></msub> <mo>}</mo></mrow> <mrow><mi>i</mi> <mo>=</mo> <mn>1</mn></mrow> <mn>8</mn></msubsup></semantics></math>

表示CLIP空间中类别名称如下：

<math display="block"><semantics><mtable displaystyle="true"><mtr><mtd columnalign="right"><mrow><msubsup><mrow><mo>{</mo> <msup><mrow><mi mathvariant="bold-italic">f</mi></mrow> <msub><mi>C</mi> <mi>i</mi></msub></msup> <mo>}</mo></mrow> <mrow><mi>i</mi> <mo>=</mo> <mn>1</mn></mrow> <mn>8</mn></msubsup> <mo>=</mo> <msubsup><mrow><mo>{</mo> <msub><mi>CLIP</mi> <mi>text</mi></msub> <mrow><mo>(</mo> <msub><mi>C</mi> <mi>i</mi></msub> <mo>)</mo></mrow> <mo>}</mo></mrow> <mrow><mi>i</mi> <mo>=</mo> <mn>1</mn></mrow> <mn>8</mn></msubsup> <mo>.</mo></mrow></mtd></mtr></mtable></semantics></math>

提出的模型计算余弦相似性

<math display="inline"><semantics><msub><mi>s</mi> <mi>i</mi></msub></semantics></math>

在文本特征之间

<math display="inline"><semantics><msup><mrow><mi mathvariant="bold-italic">f</mi></mrow> <msub><mi>C</mi> <mi>i</mi></msub></msup></semantics></math>

和图像特征

<math display="inline"><semantics><msup><mrow><mi mathvariant="bold-italic">f</mi></mrow> <mi>I</mi></msup></semantics></math>

在方程（

[1](#FD1-sensors-24-00921)

）如下：

<math display="block"><semantics><mtable displaystyle="true"><mtr><mtd columnalign="right"><mrow><msub><mi>s</mi> <mi>i</mi></msub> <mo>=</mo> <mfrac><mrow><msup><mrow><mi mathvariant="bold-italic">f</mi></mrow> <mi>I</mi></msup> <mo>·</mo> <msup><mrow><mi mathvariant="bold-italic">f</mi></mrow> <msub><mi>C</mi> <mi>i</mi></msub></msup></mrow> <mrow><mrow><mo>|</mo> <mo>|</mo></mrow> <msup><mrow><mi mathvariant="bold-italic">f</mi></mrow> <mi>I</mi></msup> <msub><mrow><mo>|</mo> <mo>|</mo></mrow> <mn>2</mn></msub> <mrow><mo>|</mo> <mo>|</mo></mrow> <msup><mrow><mi mathvariant="bold-italic">f</mi></mrow> <msub><mi>C</mi> <mi>i</mi></msub></msup> <msub><mrow><mo>|</mo> <mo>|</mo></mrow> <mn>2</mn></msub></mrow></mfrac> <mo>.</mo></mrow></mtd></mtr></mtable></semantics></math>

我们获得一个向量

<math display="inline"><semantics><mi mathvariant="bold-italic">s</mi></semantics></math>

代表图像的类别

我

通过连接计算的余弦相似性

<math display="inline"><semantics><msubsup><mrow><mo>{</mo> <msub><mi>s</mi> <mi>i</mi></msub> <mo>}</mo></mrow> <mrow><mi>i</mi> <mo>=</mo> <mn>1</mn></mrow> <mn>8</mn></msubsup></semantics></math>

如下：

<math display="block"><semantics><mtable displaystyle="true"><mtr><mtd columnalign="right"><mrow><mi mathvariant="bold-italic">s</mi> <mo>=</mo> <mo>[</mo> <msub><mi>s</mi> <mn>1</mn></msub> <mo>;</mo> <msub><mi>s</mi> <mn>2</mn></msub> <mo>;</mo> <mo>…</mo> <mo>;</mo> <msub><mi>s</mi> <mn>8</mn></msub> <mo>]</mo> <mo>.</mo></mrow></mtd></mtr></mtable></semantics></math>

提出的模型最终获得了类别特征

<math display="inline"><semantics><mrow><mi mathvariant="bold-italic">c</mi> <mo>∈</mo> <msup><mi mathvariant="double-struck">R</mi> <mn>8</mn></msup></mrow></semantics></math>

通过对向量应用softmax函数

<math display="inline"><semantics><mi mathvariant="bold-italic">s</mi></semantics></math>

。这个特征

<math display="inline"><semantics><mi mathvariant="bold-italic">c</mi></semantics></math>

可以代表图像所属的类别

我

属于的全面表达，无需手动标记。

提出的模型基于三个计算特征预测参与率的类

<math display="inline"><semantics><mi mathvariant="bold-italic">m</mi></semantics></math>

。

<math display="inline"><semantics><mi mathvariant="bold-italic">n</mi></semantics></math>

，和

<math display="inline"><semantics><mi mathvariant="bold-italic">c</mi></semantics></math>

。具体来说，我们使用三个特征首先计算连接特征

<math display="inline"><semantics><mrow><mi mathvariant="bold-italic">h</mi> <mo>∈</mo> <msup><mi mathvariant="double-struck">R</mi> <mrow><mn>2</mn> <mi>D</mi> <mo>+</mo> <mn>18</mn></mrow></msup></mrow></semantics></math>

如下：

这个连接特征

<math display="inline"><semantics><mi mathvariant="bold-italic">h</mi></semantics></math>

可以考虑图像的美学和类别

我

同时全面代表图像

我

和文本

T

包含在帖子中。这个特征

<math display="inline"><semantics><mi mathvariant="bold-italic">h</mi></semantics></math>

通过全连接层传递以获得类分布

<math display="inline"><semantics><mover accent="true"><mi mathvariant="bold-italic">y</mi> <mo stretchy="false">^</mo></mover></semantics></math>

参与率。通过使用

<math display="inline"><semantics><mover accent="true"><mi mathvariant="bold-italic">y</mi> <mo stretchy="false">^</mo></mover></semantics></math>

，提出的模型计算帖子分数如下：

<math display="block"><semantics><mtable displaystyle="true"><mtr><mtd columnalign="right"><mrow><mi>Post</mi> <mi>score</mi> <mo>=</mo> <munderover><mo>∑</mo> <mrow><mi>k</mi> <mo>=</mo> <mn>1</mn></mrow> <mi>K</mi></munderover> <mrow><mo>(</mo> <mi>k</mi> <mo>−</mo> <mn>1</mn> <mo>)</mo></mrow> <mo>×</mo> <msub><mover accent="true"><mi>y</mi> <mo stretchy="false">^</mo></mover> <mi>k</mi></msub> <mo>,</mo></mrow></mtd></mtr></mtable></semantics></math>

在哪里

<math display="inline"><semantics><msub><mover accent="true"><mi>y</mi> <mo stretchy="false">^</mo></mover> <mi>k</mi></msub></semantics></math>

是

k

预测的类分布的th元素

<math display="inline"><semantics><mover accent="true"><mi mathvariant="bold-italic">y</mi> <mo stretchy="false">^</mo></mover></semantics></math>

，和

K

是参与率类别的数量。分布朝向具有高参与率类别的偏斜程度越大，帖子分数越高。

#### 3.1.2。损失函数

为了预测参与率的类别分布，我们应用交叉熵 [

[55](#B55-sensors-24-00921)

作为损失函数，并将其视为参与率类别的分类任务。损失函数

<math display="inline"><semantics><mi mathvariant="script">L</mi></semantics></math>

描述如下：

<math display="block"><semantics><mtable displaystyle="true"><mtr><mtd columnalign="right"><mrow><mi mathvariant="script">L</mi> <mo>=</mo> <mo>−</mo> <munderover><mo>∑</mo> <mrow><mi>k</mi> <mo>=</mo> <mn>1</mn></mrow> <mi>K</mi></munderover> <msub><mi>y</mi> <mi>k</mi></msub> <mi>log</mi> <msub><mover accent="true"><mi>y</mi> <mo stretchy="false">^</mo></mover> <mi>k</mi></msub> <mo>,</mo></mrow></mtd></mtr></mtable></semantics></math>

其中

<math display="inline"><semantics><msub><mi>y</mi> <mi>k</mi></msub></semantics></math>

是

k

地元素的地面真实类别分布

<math display="inline"><semantics><mi mathvariant="bold-italic">y</mi></semantics></math>

。通过最小化从该函数计算得到的损失

<math display="inline"><semantics><mi mathvariant="script">L</mi></semantics></math>

，可以预测给定图像的参与率类别分布

我

和文本中构造的帖子分数

T

作为输入。

#### 3.2。基于帖子分数的帖子图像编辑

为了获得多个文本引导的图像编辑结果，提出的方法首先利用最近提出的大型语言模型（即GPT-3 [

[53](#B53-sensors-24-00921)

）。我们创建了句子“生成四个不同的释义

<math display="inline"><semantics><mrow><mo>{</mo> <mi>P</mi> <mo>}</mo></mrow></semantics></math>

”。使用文本提示

P

用于图像编辑。提出的方法可以获得与文本提示类似的四个表示

P

通过将创建的句子提供给大型语言模型。使用多个文本提示增加了获取在进行用户期望的图像编辑的同时吸引更多社交媒体观众关注的编辑图像的可能性。提出的方法使用与文本提示相似的四个表示

P

并基于文本引导的图像编辑方法生成四张编辑后的图像 [

[15](#B15-sensors-24-00921)

]。

为了选择在社交媒体上吸引更多观众关注的编辑图像，我们应用提出的模型预测在

[第3.1节](#sec3dot1-sensors-24-00921)

。提出的方法将要发布的文本和四张编辑后的图像分别输入该模型，并获得四个帖子分数。最后，提出的方法选择编辑后的图像

<math display="inline"><semantics><msup><mi>I</mi> <mo>′</mo></msup></semantics></math>

从所有编辑后的图像中选择具有最高帖子分数的图像。由此可见，可以生成图像

<math display="inline"><semantics><msup><mi>I</mi> <mo>′</mo></msup></semantics></math>

进行了文本引导的图像编辑，并将在社交媒体上获得关注。通过使用提出的方法，可以减少对社交媒体营销知识不足的用户使用文本引导的图像编辑的负担。

## 6。结论

本文提出了一种基于后分数的新型文本引导图像编辑方法，旨在在社交媒体上引起关注。所提出的方法新引入了一种模型，用于从发布的图像和文本预测社交媒体上的发布分数，从而生成受观众欢迎的编辑图像。在所提出的方法中，我们应用了预训练的文本引导图像编辑方法，并从大型语言模型生成的多个文本提示中获取多个编辑图像。在这些图像中，利用预测表示参与率的新型模型，所提出的方法选择了在社交媒体上获得最多观众关注的编辑图像。主观实验的结果表明，从所提出的方法生成的编辑图像准确反映了文本提示的内容，并在社交媒体上给观众留下了积极的印象。这些结果得到了主观评估的支持，研究对象发现他们在社交媒体上找到使用所提出的方法生成的编辑图像的帖子时，最愿意点“赞”或评论。

未来，我们将通过设计一种新的图像和文本特征集成技术来改进所提出的模型，以预测帖子分数。此外，通过基于文本引导图像编辑评估指标的评分选择编辑图像，可以确保图像编辑的准确性，从而提高所提出方法的整体性能。
