<!--yml

category: 未分类

date: 2024-05-29 13:27:09

-->

# Electronics | Free Full-Text | MeMPA：一种用于应对内存墙问题的内存映射M-SIMD协处理器

> 来源：[https://www.mdpi.com/2079-9292/13/5/854](https://www.mdpi.com/2079-9292/13/5/854)

为了直接评估MeMPA实现的性能优劣，我们决定在架构上映射与混合SIMD评估中使用的相同基准测试

[2](#B2-electronics-13-00854)

，即K-最近邻（K-NN）、K均值、矩阵-向量乘法（MVM）、均值和方差（

<math display="inline"><semantics><mi>μ</mi></semantics></math>

&

<math display="inline"><semantics><msup><mi>σ</mi> <mn>2</mn></msup></semantics></math>

），以及离散傅里叶变换（DFT）。此外，由于MeMPA缺乏真正的编译器，我们排除了在

[Section 2](#sec2-electronics-13-00854)

其中手动映射将会非常困难。

[Table 1](#electronics-13-00854-t001)

总结了算法在数据处理数量、算法执行所需的时钟周期数以及相关的功耗方面的映射。

为了简洁起见，在接下来的内容中，仅详细描述了其中一个实现算法的映射。特别是，MVM映射出于两个重要原因。一方面，MVM允许我们轻松指出MeMPA如何突出显示作为减少树机制、M-SIMD计算范式和像战舰游戏一样的启用机制，可以有效地执行应用程序。另一方面，它代表了卷积神经网络的基本操作，属于数据密集型应用程序的集合，这些应用程序在时间和能耗方面都将从MeMPA的使用中大大受益。然而，可以通过与描述的MVM实现相同的方式推导出所有其他算法的实现。

#### MVM

任何算法在MeMPA上的映射由两个宏阶段确定：处理矩阵初始化阶段，在其中CPU加载要处理的所有数据到处理矩阵中，以及算法执行阶段，在其中实际执行算法。

关于在《[MeMPA: A Memory Mapped M-SIMD Co-Processor to Cope with the Memory Wall Issue](https://www.mdpi.com/2079-9292/13/5/854)》中描述的MVM，

[Table 1](#electronics-13-00854-t001)

，实现的乘法是在一个16×16矩阵和一个16×1向量之间执行的。因此，在处理矩阵初始化阶段，每个矩阵元素都存储在智能块的不同块字中，而所有向量项都加载到标准块的第一行中。整个处理矩阵初始化阶段共花费了272个时钟周期，即每个数据写入使用一个时钟周期。

然后，一个指令被实例化（一个时钟周期），用于将块字数据备份到寄存器文件的第一个位置，这一步骤对于任何算法都是必要的，以避免在算法结束时，MeMPA保存结果到块字中以便CPU看到时，丢失初始数据。

然后，真正的算法执行阶段开始，通过在每个矩阵元素之间执行 256 个乘积 (

<math display="inline"><semantics><msub><mi>x</mi> <mrow><mi>i</mi> <mo>,</mo> <mi>j</mi></mrow></msub></semantics></math>

) 和右向量项 (

<math display="inline"><semantics><msub><mi>y</mi> <mi>j</mi></msub></semantics></math>

) 需要

<math display="inline"><semantics><mrow><msub><mi>z</mi> <mi>i</mi></msub> <mo>=</mo> <msubsup><mo>∑</mo> <mrow><mi>j</mi> <mo>=</mo> <mn>0</mn></mrow> <mn>15</mn></msubsup> <msub><mi>x</mi> <mrow><mi>i</mi> <mo>,</mo> <mi>j</mi></mrow></msub> <msub><mi>y</mi> <mi>j</mi></msub></mrow></semantics></math>

. 总共，使用了 5 + 1 条指令（六个时钟周期）来完成这些乘法。特别是，对于前五条指令，所有的 ID 都是活动的，每次只驱动一个智能块行。第一种方案

`VLIW_Instruction`

报告

[图 4](#electronics-13-00854-f004)

.

通过此指令，驱动了第 1、6 和第 11 行智能块行以计算所有

<math display="inline"><semantics><mrow><msub><mi>x</mi> <mrow><mi>i</mi> <mo>,</mo> <mi>j</mi></mrow></msub> <mo>·</mo> <msub><mi>y</mi> <mi>j</mi></msub></mrow></semantics></math>

(i = 0, 5, 10 和 j = 0, …, 15) 并行项并将结果保存在相关的旁路存储器中。更详细地说，看看第一条指令的结构在

[图 4](#electronics-13-00854-f004)

, 可以看到对于 ID1 (即图中从顶部开始的第一组五行智能块行)，

[图 2](#electronics-13-00854-f002)

b)，激活模式仅看到第一行智能块启用，而其他行则未启用 (

`EN ROW = “10000”`

). 执行的操作是乘法 (

`OPCODE = Multiplier`

), 将来自列互联和块字的数据作为源操作符 (

`SOURCE OP = Col_Int_Block_Word`

). Column Interconnection 中的数据在

`ADDRESS S1`

字段，表示从组内第一个智能块开始的地址偏移值。在这种情况下，作为第一组智能块，偏移将等于 16，即处理矩阵的第十六行（对应标准块的第一行）。最后，目的地简单地指定在

`DEST OP`

字段作为旁路，希望保存在旁路存储器中。同样的推理可以应用于 ID2，它标识第二组智能块，也排列在五行中。再次，用于计算最终向量元素的

`EN ROW`

等于

`“10000”`

仅标识子组的第一行；操作始终是乘法 (

`OPCODE = Multiplier`

); 源操作数始终来自列互联和块字 (

`SOURCE OP = Col_Int_Block_Word`

); 目标始终为旁路存储器 (

`DEST OP = Bypass`

), 而这次的偏移量等于 11，总是指向标准块的第一行。

<math display="inline"><semantics><mrow><msub><mi>x</mi> <mrow><mn>15</mn> <mo>,</mo> <mi>j</mi></mrow></msub> <msub><mi>y</mi> <mi>j</mi></msub></mrow></semantics></math> 乘积。

一旦处理矩阵的旁路存储器中准备好所有的乘积，所有生成的和都在 Block Words 的第一个智能块列中。

<math display="inline"><semantics><msub><mi>z</mi> <mi>i</mi></msub></semantics></math>

在四个指令（四个时钟周期）中进行了值的计算。为了实现这一点，充分利用了由行互联实施的减少树机制，从而将计算16个并行求和所需的指令数从15个减少到4个。这四个指令中的第一个指令的方案如下所示：

[图 4](#electronics-13-00854-f004)

（指令8）。与之前一组指令不同的是，在智能块列的所有启用信号始终处于活动状态的情况下，这四个指令中的每一个激活了不同的智能块列集合，而所有的ID都驱动了当前时钟周期的所有智能块行，以进行相同的操作。对于第一个指令，所有奇数智能块列都被启用，以便计算所有后续的和并保存在旁路存储器中：

<math display="inline"><semantics><mrow><msub><mi>x</mi> <mrow><mi>i</mi> <mo>,</mo> <mn>0</mn></mrow></msub> <msub><mi>y</mi> <mn>0</mn></msub> <mo>+</mo> <msub><mi>x</mi> <mrow><mi>i</mi> <mo>,</mo> <mn>1</mn></mrow></msub> <msub><mi>y</mi> <mn>1</mn></msub></mrow></semantics></math>

。

<math display="inline"><semantics><mrow><msub><mi>x</mi> <mrow><mi>i</mi> <mo>,</mo> <mn>2</mn></mrow></msub> <msub><mi>y</mi> <mn>2</mn></msub> <mo>+</mo> <msub><mi>x</mi> <mrow><mi>i</mi> <mo>,</mo> <mn>3</mn></mrow></msub> <msub><mi>y</mi> <mn>3</mn></msub></mrow></semantics></math>

。

<math display="inline"><semantics><mrow><msub><mi>x</mi> <mrow><mi>i</mi> <mo>,</mo> <mn>4</mn></mrow></msub> <msub><mi>y</mi> <mn>4</mn></msub> <mo>+</mo> <msub><mi>x</mi> <mrow><mi>i</mi> <mo>,</mo> <mn>5</mn></mrow></msub> <msub><mi>y</mi> <mn>5</mn></msub></mrow></semantics></math>

，利用相同的推理，第二、第三、第四和第五个指令计算了分别对应于i等于（1,6,11）、（2,7,12）、（3,8,13）和（4,9,14）的项。最后，第六个指令通过第三个指令解码器仅启用了最后一个智能块行来执行

<math display="inline"><semantics><mrow><msub><mi>x</mi> <mrow><mi>i</mi> <mo>,</mo> <mn>6</mn></mrow></msub> <msub><mi>y</mi> <mn>6</mn></msub> <mo>+</mo> <msub><mi>x</mi> <mrow><mi>i</mi> <mo>,</mo> <mn>7</mn></mrow></msub> <msub><mi>y</mi> <mn>7</mn></msub></mrow></semantics></math>

。

<math display="inline"><semantics><mrow><msub><mi>x</mi> <mrow><mi>i</mi> <mo>,</mo> <mn>8</mn></mrow></msub> <msub><mi>y</mi> <mn>8</mn></msub> <mo>+</mo> <msub><mi>x</mi> <mrow><mi>i</mi> <mo>,</mo> <mn>9</mn></mrow></msub> <msub><mi>y</mi> <mn>9</mn></msub></mrow></semantics></math>

。

<math display="inline"><semantics><mrow><msub><mi>x</mi> <mrow><mi>i</mi> <mo>,</mo> <mn>10</mn></mrow></msub> <msub><mi>y</mi> <mn>10</mn></msub> <mo>+</mo> <msub><mi>x</mi> <mrow><mi>i</mi> <mo>,</mo> <mn>11</mn></mrow></msub> <msub><mi>y</mi> <mn>11</mn></msub></mrow></semantics></math>

。

<math display="inline"><semantics><mrow><msub><mi>x</mi> <mrow><mi>i</mi> <mo>,</mo> <mn>12</mn></mrow></msub> <msub><mi>y</mi> <mn>12</mn></msub> <mo>+</mo> <msub><mi>x</mi> <mrow><mi>i</mi> <mo>,</mo> <mn>13</mn></mrow></msub> <msub><mi>y</mi> <mn>3</mn></msub></mrow></semantics></math>

。

<math display="inline"><semantics><mrow><msub><mi>x</mi> <mrow><mi>i</mi> <mo>,</mo> <mn>14</mn></mrow></msub> <msub><mi>y</mi> <mn>14</mn></msub> <mo>+</mo> <msub><mi>x</mi> <mrow><mi>i</mi> <mo>,</mo> <mn>15</mn></mrow></msub> <msub><mi>y</mi> <mn>15</mn></msub></mrow></semantics></math>

，对于i = 0, …, 15。然后，对于第二个指令，仅启用列1、5、9和13来计算对应的项。类似地，第三和第四个指令实施了剩余的求和，以便在最后一个指令结束后，所有最终的

<math display="inline"><semantics><mrow><msub><mi>x</mi> <mrow><mi>i</mi> <mo>,</mo> <mn>0</mn></mrow></msub> <msub><mi>y</mi> <mn>0</mn></msub> <mo>+</mo> <msub><mi>x</mi> <mrow><mi>i</mi> <mo>,</mo> <mn>1</mn></mrow></msub> <msub><mi>y</mi> <mn>1</mn></msub> <mo>+</mo> <msub><mi>x</mi> <mrow><mi>i</mi> <mo>,</mo> <mn>2</mn></mrow></msub> <msub><mi>y</mi> <mn>2</mn></msub> <mo>+</mo> <msub><mi>x</mi> <mrow><mi>i</mi> <mo>,</mo> <mn>3</mn></mrow></msub> <msub><mi>y</mi> <mn>3</mn></msub></mrow></semantics></math>

。

<math display="inline"><semantics><mrow><msub><mi>x</mi> <mrow><mi>i</mi> <mo>,</mo> <mn>4</mn></mrow></msub> <msub><mi>y</mi> <mn>4</mn></msub> <mo>+</mo> <msub><mi>x</mi> <mrow><mi>i</mi> <mo>,</mo> <mn>5</mn></mrow></msub> <msub><mi>y</mi> <mn>5</mn></msub> <mo>+</mo> <msub><mi>x</mi> <mrow><mi>i</mi> <mo>,</mo> <mn>6</mn></mrow></msub> <msub><mi>y</mi> <mn>6</mn></msub> <mo>+</mo> <msub><mi>x</mi> <mrow><mi>i</mi> <mo>,</mo> <mn>7</mn></mrow></msub> <msub><mi>y</mi> <mn>7</mn></msub></mrow></semantics></math>

。

<math display="inline"><semantics><mrow><msub><mi>x</mi> <mrow><mi>i</mi> <mo>,</mo> <mn>8</mn></mrow></msub> <msub><mi>y</mi> <mn>8</mn></msub> <mo>+</mo> <msub><mi>x</mi> <mrow><mi>i</mi> <mo>,</mo> <mn>9</mn></mrow></msub> <msub><mi>y</mi> <mn>9</mn></msub> <mo>+</mo> <msub><mi>x</mi> <mrow><mi>i</mi> <mo>,</mo> <mn>10</mn></mrow></msub> <msub><mi>y</mi> <mn>10</mn></msub> <mo>+</mo> <msub><mi>x</mi> <mrow><mi>i</mi> <mo>,</mo> <mn>11</mn></mrow></msub> <msub><mi>y</mi> <mn>11</mn></msub></mrow></semantics></math>

。

<math display="inline"><semantics><mrow><msub><mi>x</mi> <mrow><mi>i</mi> <mo>,</mo> <mn>12</mn></mrow></msub> <msub><mi>y</mi> <mn>12</mn></msub> <mo>+</mo> <msub><mi>x</mi> <mrow><mi>i</mi> <mo>,</mo> <mn>13</mn></mrow></msub> <msub><mi>y</mi> <mn>3</mn></msub> <mo>+</mo> <msub><mi>x</mi> <mrow><mi>i</mi> <mo>,</mo> <mn>14</mn></mrow></msub> <msub><mi>y</mi> <mn>14</mn></msub> <mo>+</mo> <msub><mi>x</mi> <mrow><mi>i</mi> <mo>,</mo> <mn>15</mn></mrow></msub> <msub><mi>y</mi> <mn>15</mn></msub></mrow></semantics></math>

。

<math display="inline"><semantics><msub><mi>z</mi> <mi>i</mi></msub></semantics></math>

值在第一个智能块列的块字中可用。
