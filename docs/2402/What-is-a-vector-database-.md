<!--yml

类别：未分类

日期：2024-05-29 13:18:29

-->

# 什么是向量数据库？

> 来源：[https://blog.meilisearch.com/what-is-a-vector-database/](https://blog.meilisearch.com/what-is-a-vector-database/)

## 理解向量数据库

向量数据库是基于相似性进行搜索的首选，这在像推荐下一个喜爱的电影、识别照片中的人物或找到与搜索相关联的文本等**人工智能驱动应用**中发挥着关键作用。这些应用的核心是**向量嵌入**，这种复杂的数据形式超出了传统数据库的存储和检索能力。

### 向量嵌入的角色

[向量嵌入](https://blog.meilisearch.com/what-are-vector-embeddings/?utm_source=blog&utm_medium=vector-db-content&utm_campaign=vector-search)是一种将复杂的非数值数据（如单词、句子甚至图像）**转换为数值格式**，同时保留其语义**含义**和关系的方法。

嵌入向量是由机器学习模型生成的**多维对象**，其中每个维度代表数据的不同特征或方面。为了正确捕捉数据的复杂性，向量的维数可以从几十到数千不等，取决于数据的大小和性质。

### 向量数据库与传统数据库的对比

这种复杂性使得**传统数据库**（设计用于在表格中存储结构化数据）无法处理嵌入。这些向量的数量和复杂性，每个可能包含数千个维度，挑战了行列格式的匹配。这种不匹配要求为向量数据量身定制的替代存储和检索解决方案。

这就是**向量数据库**如[Meilisearch](https://www.meilisearch.com/?utm_source=blog&utm_medium=vector-db-content&utm_campaign=vector-search)的作用所在。它们旨在解决向量嵌入的独特需求，促进其所包含信息的高效存储和检索。特别是，它们支持执行**相似性搜索**，也称为语义搜索，这对有效利用嵌入至关重要。

换句话说，向量数据库使我们能够轻松高效地与向量嵌入交互，这对需要语义理解和相似性匹配的应用程序至关重要。

## 什么是相似性搜索？

如果我们把**向量嵌入**比作宇宙中巨大星座中的星星，那么相似性搜索或**向量搜索**就像是在空间中寻找距离你当前位置最近的星星。在实际中，这意味着根据你的搜索查询找到最相关的文档、图像或产品。

为了实现这一目标，您需要测量**查询向量**与数据库中其他向量之间的**距离**，通常使用如[余弦相似度](https://en.wikipedia.org/wiki/Cosine_similarity?ref=blog.meilisearch.com)或[欧几里得距离](https://en.wikipedia.org/wiki/Euclidean_distance?ref=blog.meilisearch.com)等方法。这些方法是用于确定其他数据点与您查询之间的接近程度，类似于测量夜空中星星的**接近度**。

### 机器学习模型的角色

然而，这种搜索的**成功**并不仅仅依赖于数学计算；它高度依赖于用于生成和查询向量的**机器学习模型**。每个向量的含义都与创建它的模型的语义空间密切相关。在这里，一致性至关重要，确保所有向量都能“说同一种语言”，并遵循相同的上下文规则，使得搜索**有意义且准确**。换句话说，为了实现相关的搜索结果，使用相同的模型来生成和查询嵌入向量是至关重要的。

**相似性搜索**是向量数据库如**Meilisearch**真正发光的地方，因为它们允许广泛的应用，如人脸识别、电影推荐和个性化内容发现。通过让用户将向量嵌入存储在其文档旁边，Meilisearch不仅促进了相似性搜索，还引入了**混合搜索**能力，扩展了其潜在应用。通过集成来自各种AI解决方案提供商的模型，Meilisearch使用户能够**优化向量嵌入**以更好地满足其特定需求。

总之，这些数据库分析和比较复杂数据模式的能力，允许在多个领域实现高度相关和准确的结果，增强用户体验和操作效率。

AI搜索即将登陆Meilisearch Cloud，加入等待列表：

* * *

[Meilisearch](https://meilisearch.com/?utm_source=blog&utm_medium=vector-db-content&utm_campaign=vector-search) 是一个开源搜索引擎，不仅为最终用户提供最先进的体验，还提供简单直观的开发者体验。

作为关键词搜索的长期参与者，Meilisearch使用户能够构建在基于AI驱动的解决方案之上的搜索用例，不仅支持向量搜索作为向量存储，还通过提供混合搜索，将全文搜索与语义搜索相结合，提升搜索结果的准确性和全面性。

想要了解更多关于 Meilisearch 的内容，你可以加入 [Discord](https://discord.meilisearch.com/?ref=blog.meilisearch.com) 社区或订阅 [newsletter](https://meilisearch.us2.list-manage.com/subscribe?u=27870f7b71c908a8b359599fb&id=79582d828e&utm_source=blog&utm_medium=vector-db-content&utm_campaign=vector-search)。你可以通过查看 [roadmap](https://roadmap.meilisearch.com/?utm_source=blog&utm_medium=vector-db-content&utm_campaign=vector-search) 和参与 [product discussions](https://github.com/orgs/meilisearch/discussions?utm_source=blog&utm_medium=vector-db-content&utm_campaign=vector-search) 了解更多关于该产品的信息。
