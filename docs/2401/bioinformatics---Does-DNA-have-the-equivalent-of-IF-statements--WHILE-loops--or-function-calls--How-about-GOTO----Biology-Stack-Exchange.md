<!--yml

类别：未分类

日期：2024-05-27 14:40:01

-->

# 生物信息学 - DNA 是否具有 IF 语句、WHILE 循环或函数调用的等价物？GOTO 呢？ - 生物学堆栈交换

> 来源：[`biology.stackexchange.com/questions/30116/does-dna-have-the-equivalent-of-if-statements-while-loops-or-function-calls-h`](https://biology.stackexchange.com/questions/30116/does-dna-have-the-equivalent-of-if-statements-while-loops-or-function-calls-h)

补充前面的回答，但**转录干扰**（见例如[Shearwin et al., 2005](http://www.ncbi.nlm.nih.gov/pmc/articles/PMC2941638/)）可以看作是一种 IF 语句（或 WHILE）的形式，因为：

```
if(x transcribed){not y transcribed} 
```

干扰不一定是二进制的，更常见的是分级响应。转录干扰也可以发生在 RNA 阶段（见例如[Xue et al, 2014](http://www.nature.com/nature/journal/v514/n7524/full/nature13671.html)），使用[*反义 RNA*](http://en.wikipedia.org/wiki/Antisense_RNA) 并通常提供负反馈环路，但是干扰然后从 DNA 中移除，不代表直接在 DNA 阶段的 IF 语句类比。

对我来说，`GOTO` 主要适用于顺序代码执行，而这对 DNA 并非如此（大量转录一直在同时进行）。更普遍地说，DNA 的并行“执行”，以及 DNA、转录物和蛋白质（等等）之间的持续相互作用和反馈循环，也意味着细胞过程远没有计算机代码那么清晰和可追踪，这意味着计算机代码对于细胞过程和 DNA 功能来说是一个非常不恰当的比喻。
