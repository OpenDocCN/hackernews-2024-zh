<!--yml

category: 未分类

date: 2024-05-27 14:42:32

-->

# 非人类智力 - 克莱米埃收集

> 来源：[https://www.cremieux.xyz/p/nonhuman-intelligence](https://www.cremieux.xyz/p/nonhuman-intelligence)

*这是一系列定时写作帖子中的第二篇。如果花费超过一小时来完成它们，我的文本编辑器会关闭，到目前为止写的内容也会被删除。前一个定时写作可以在[这里](https://www.cremieux.xyz/p/what-happens-when-you-use-a-biased)找到。*

* * *

还没有人开发出一种能够让我们比较人类和动物智力的测试。有许多认知测试可以用来比较不同的人类，但没有一种可以让我们估计人类与例如猿类乃至更低智力水平的动物之间的智力差距。

猩猩可能是我们能够在人类与非人类物种之间进行比较的最接近的物种。这种可能性的线索可在卡夫曼、雷诺兹和卡夫曼2019年的论文中观察到，他们检验了为非人灵长类动物开发的智商测试的因子结构，即灵长类认知测试电池（PCTB）。

目前主导的智力测试模型是卡特尔-霍恩-卡罗尔（Cattell-Horn-Carroll），即CHC模型，源自约翰·B·卡罗尔在1993年的《人类认知能力：因子分析研究概述》。这张由蒂莫西·贝茨制作的图表是一个很好的可视化工具：

事实证明，这个人类模型大致上可以适应卡夫曼、雷诺兹和卡夫曼的PCTB数据，如下所示：

这个模型大体符合，但是如果你把这个测试电池给成年人，他们会在所有测试上都得满分，因为对于人类来说这些测试太简单了；我们远远超过了猿类。因此，解决方案是把这个测试电池给非常年幼的人类——学龄前儿童——因为他们的能力尚未超出猿类智能的范围。

尽管这样做似乎毫无意义，因为如果我们这样做，我们不是在比较人类和猿类的*智力水平*，那有什么意义呢？关键在于评估跨物种之间智力*结构*的等效性。这就是为什么使用诸如记忆和速度这样的基础认知任务不起作用的原因，因为这些测试之间的结构关系可能因关注点、遵守程序、理解测试规则能力等而在猿类和人类之间不同，而人类在这些方面可能是可以比较的，但我们不能确保猿类和人类是可以的。因此，这种比较不仅仅是关于智力本身，正如我们所希望的那样。

结构上的可比性将在许多领域产生巨大的影响。如果确保了这一点，猿类将立即成为神经增强剂的可行测试对象；不仅如此，它们还将成为发现神经增强剂候选物的工具，因为我们可以为提高智力而培育猿类，并努力发现其遗传基础，仅需几代人。跨物种的结构比较也将对智力的不同理论提出或反对意见，例如，可能抵制那些强调文化在人类中起作用的理论。毫无疑问，这一发现将为我们提供更多的可能性，但不用说，这种分析上的杠杆作用将是巨大的。

* * *

那么有没有可能有些东西比人类更聪明呢？

[Maxim Lott有一篇新文章，按照他们在图形矩阵测试中的表现排名不同的LLM](https://www.maximumtruth.org/p/ais-ranked-by-iq-ai-passes-100-iq)。这个想法是什么？因为现在AI已经达到了人类水平的智商，并且它们正在迅速进步，我们即将在口袋里拥有人类水平（甚至更高）的机器智能，这意味着“世界很可能很快会发生很大变化。”

让我们来测试一下。我得到了据说是从[这项研究](https://www.frontiersin.org/journals/psychology/articles/10.3389/fpsyg.2021.619440/full)中获取的26个问题，并以Lott风格口头输入的问题形式，给了Claude 3 Opus，使用了50次。然后我将Claude 3 Opus的回答与[Krautter等人的样本](https://osf.io/nc3us/)进行了比较。这些问题分别使用了以下四条规则：

我使用了8个不同条目（错误/正确或0/1分）分数总和的包裹来评估[测量不变性](https://www.cremieux.xyz/i/141076378/measurement-invariance)，这些包裹像这样自动计算：

```
Responses <- select(df, starts_with("item_"))
Matrix <- cor(Responses)
NItem <- ncol(Matrix)
Names <- colnames(Matrix)
Parcels <- list()
AvgCor <- apply(Matrix, 1, mean)
Sorted_Items <- names(sort(AvgCor, decreasing = T))

Rem <- Sorted_Items
for(i in 1:6) {
   Parcels[[length(Parcels) + 1]] <- head(Rem, 3)
   Rem <- setdiff(Rem, head(Rem, 3))
}

for(i in 1:2) {
   Parcels[[length(Parcels) + 1]] <- head(Rem, 4)
   Rem <- setdiff(Rem, head(Rem, 4))
}

fpr (i in seq_along(Parcels)) {
    Parcel_Names <- paste("Parcel", i)
    Responses[[Parcel_Names]] <- rowSums(select(Responses, all_of(Parcels[[i]]))), na.rm = T)
}

ParcSums <- data.frame(matrix(nrow = nrow(df), ncol = length(Parcels)))
names(ParcSums) <- paste0("Parcel", seq_along(Parcels))

for (i in seq_along(Parcels)) { 
     Parcel_Items <- Parcels[[i]]
     ParcSums[,i] <- rowSums(select(df, all_of(Parcel_Items)), na.rm = T)
}
```

在原始数据中，包裹包括{item_13, item_12, item_20}，{16, 14, 22}，{15, 10, 6}，{19, 21, 24}，{25, 9, 23}，{26, 17, 3}，{18, 11, 7, 2}，{5, 8, 4, 1}。这些包裹的单因素模型，使用*lavaan*中的WLSMV估计器进行了很好的拟合，具有稳健的CFI为0.998和稳健的RMSEA为0.036。这些包裹的*g*加载在0.771到0.901之间。

所以我去应用模型对Claude的数据进行评估，以在测试多组模型之前评估初始模型适配性。

不可能的。Claude的反应几乎完全是同质的，因此模型无法拟合，因为没有任何方差需要解释。这可能是因为每个Claude实例都应被视为重新测试同一个人，但不管怎样，这对评估机器和人类智能可比性构成了方法论上的障碍，尽管这并不证明这两者必然是不可比较的。

所以为了暗示人类和LLM表现的可比性，我将原始研究的四个不同群体中每个项目的正确百分比与克劳德评估的正确百分比进行了相关性分析，如下所示：

```
GroupMeans <- df_ParcSums %>% 
     select(training_or_bot, starts_with("item_")) %>%
     pivot_longer(cols = starts_with("item_"), names_to = "item", values_to = "score",
     names_transform = list(item = ~factor(., levels = paste0("item_", 1:26)))) %>%
    group_by(item, training_or_bot) %>%
    summarize(mean_score = mean(score, na.rm = T), .groups = 'drop') %>%
    pivot_wider(names_from = training_or_bot, values_from = mean_score) %>%
    arrange(match(item, paste0("item_", 1:26)))

GroupMat <- cor(GroupMeans[, -1])
```

相关性结果如下：

当不同培训水平的人群在图形矩阵规则上得到正确答案时，其他群体也是如此。但克劳德的表现基本上是随机的，与人类群体无关。不同个体的表现也与其他个体的表现相关，与个体之间使用的策略相一致，导致表现的共同模式，或更简单地说，问题的难度不同，因此一些问题更容易被更多人回答正确，而这些问题不是LLM（Large Language Models，大型语言模型）容易回答正确的问题。

更聪明的人类倾向于使用策略来回答问题，这些策略与成功回答和在问题之间使用相关联。没有证据表明克劳德使用人类使用的策略，因此至少我们没有*先验*理由相信克劳德的推理受到驱动推理的构造与人类相同。

* * *

假设我们有两个群体，一个是正常参加考试的人群，一个是在展示答案键后参加考试的人群。结果将是一个有偏见的比较：设置排除了测量不变性，因为测试将在正常参加考试的人群中测量的东西（智力）与在展示答案键的群体中测量的东西（记忆）不同。

如果记忆和智力不预测相同的事物，我们不应该期望在我们的两个不同群体中，考试结果预测相同的事物。如果LLM因为无关紧要的原因获得了“高智商”，我们也应该推断他们的表现不会像人类那样预测这些分数可能预测的事物。

[Lievens, Reeve和Heggestad](https://psycnet.apa.org/doiLanding?doi=10.1037%2F0021-9010.92.6.1672)提供了一个很好的例子来澄清这一点。他们给学生们医学院入学考试，并发现分数最初与智力相关，而不是与记忆相关；在重新测试时，分数与智力的相关性下降，而与记忆的相关性大大增加。此外，只有最初的分数预测了三年的平均成绩。记忆和智力在不同场合上的贡献是不同的，它们预测了不同的事物。

LLM的表现与人类认知显然有所不同，为什么要以不同的方式看待它呢？

我怀疑，随着语言模型在智商测试中的表现提升，它们的进步将继续表现为特定表现能力，而不是真正的推理能力，这涉及所有的心理测量意义和非意义。当语言模型的“智能”能够被证明类似于人类智能时，我会改变看法。但是，目前甚至如何测试这种可能性都不太清楚。
