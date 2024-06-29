<!--yml

category: 未分类

date: 2024-05-27 13:34:24

-->

# GQL的崛起：图查询语言的新ISO标准

> 来源：[https://www.tigergraph.com/blogs/gsql/the-rise-of-gql-a-new-iso-standard-in-graph-query-language/](https://www.tigergraph.com/blogs/gsql/the-rise-of-gql-a-new-iso-standard-in-graph-query-language/)

作者：[吴明熙](https://www.linkedin.com/in/mingxi-wu-a1704817/)

我们非常高兴地庆祝数据库技术领域的一个重要里程碑：ISO GQL标准第一版于2024年4月12日发布。您可以在此处访问标准[here](https://www.iso.org/standard/76120.html)。

作为GQL标准版本1的积极贡献者，我见证了从其起源到高质量最终出版的整个过程。我可以证明，这标志着数据库历史上的一个重要里程碑。感谢GQL编辑器[斯蒂芬·普兰提科](https://www.linkedin.com/in/stefan-plantikow-49896637/)和[斯蒂芬·卡南](https://www.linkedin.com/in/ACoAAAAtayUB9pzZK_nIKrdtC8hm6aoCRwnzxk8)，以及编辑工具支持者[吉姆·梅尔顿](https://www.linkedin.com/in/ACoAAADNnC4BsfclEJ6S5Ux61kbqFFtkzTn8dA4)，WG3召集人[基思·哈尔](https://www.linkedin.com/in/ACoAAAJRseUBq_q6BsMZ8gZhHW3T3StojwXC4X0)，以及GQL宣言的原始作者[阿拉斯泰尔·格林](https://www.linkedin.com/in/ACoAABa6wGYB8jtCFp2w2BWnBEWhFrKNYNm9dzE)。这一优雅的标准是由包括学术界和工业界查询语言专家在内的强大国际团队开发的。一个非详尽的贡献者名单可以在这两篇SIGMOD论文[1](https://dl.acm.org/doi/abs/10.1145/3514221.3526057)，[2](https://dl.acm.org/doi/10.1145/3183713.3190654)中找到。

在这篇博客中，我将分享我对GQL的个人热情，并邀请您开始学习它。

### **GQL是什么？**

GQL，或称为图查询语言，是为属性图数据库定义的ISO标准。它是自1986年SQL标准首次发布以来ISO标准委员会数据库领域出现的第一个同胞数据库语言。

GQL旨在成为属性图数据库的事实查询语言标准，以应对对此类数据库日益增长的需求。

让我们重新审视一些属性图数据库的背景知识。

首先，属性图数据库利用属性图数据建模，其中顶点和边作为表示数据元素的基本单位。相比之下，传统的关系数据模型依赖于表来表示数据。直观地说，属性图模型提供了更大的灵活性，并且更贴近对象之间的真实交互。

第二，有各种存储格式专门支持属性图数据模型。本地图数据库主要通过其存储格式与非本地同行区别开来。在本地图数据库中，顶点和边被视为存储层的一等公民，这导致了卓越的性能，特别是在路径遍历和迭代图算法中。这种性能提升源于本地图数据库作为巨大对象索引的固有结构。在图遍历过程中，通过它们连接的边，从一个活动顶点集导航到另一个，避免了需要昂贵的运行时连接（如关系数据库所做的）来重新连接数据元素。在我看来，本地图数据库的图存储格式是与其他数据库的关键差异。TigerGraph属于本地图数据库的范畴。

### **为什么选择GQL？为什么现在？**

随着图数据库行业的发展，它成为了继关系数据库和键值存储数据库之后的第三种经典数据库。在过去的十年中，蓬勃发展的图数据库行业见证了众多厂商推出了各自的图数据库产品，每个产品都伴随其专有的图查询语言。例如，Neo4j的Cypher、TigerGraph的GSQL、Oracle的PGQ、LDBC的G-core以及Gremlin等。关于图查询语言的全面概述，我建议参考Peter Boncz在CWI的见解深刻的[survey slide](https://homepages.cwi.nl/~boncz/gql-survey.pdf)。

在这个蓬勃发展的生态系统中，GQL应运而生，以应对对标准化图查询语言日益增长的需求。其发布奠定了坚实的基础，并推动了未来几年图数据库的繁荣，类似于SQL对关系数据库的作用。

### **浅谈GQL的表层**

在其核心，GQL使用模式匹配语法来声明性地针对图数据库进行查询，类似于关系数据库中的SQL。

模式的基本构件包括(1)顶点模式和(2)边模式，如下所示。

+   **MATCH** (x:Account **WHERE** x.isBlocked=’no’)

    +   x绑定到图中的一个顶点。()是一个节点模式。

+   **MATCH** −[e:Transfer **WHERE** e.amount>5M]−>

    +   e绑定到图中的一条边。-[]->表示一种边的模式。

借助以上基础构件，你可以通过交替节点和边模式来构建更长的模式。

+   **MATCH** (x:Account)-[:SignInWithIP]->(y:IP)  -[:Associated]->(p:Phone)

每个匹配的模式提供一张表。你可以像在SQL中那样对表进行过滤、分组、聚合和投影。

GQL有两种语法风格：一种是Cypher，另一种是SQL。然而，这两种风格共享相同的核心模式匹配部分。

如果你来自SQL世界，你会像这样使用。

+   **SELECT** x.age, y.ip**FROM** g **MATCH** (x:Account)-[:SignInWithIP]-(y:IP)  **WHERE** x.name == “John”

*注：TigerGraph 是 GQL 中 SQL 风格的主要贡献者和倡导者，因为我们坚信 SQL 用户将更容易学习带有 SQL 方言的 GQL。*

如果您熟悉 Cypher 的世界，您将会使用下面的例子。

+   **MATCH** (x:Account)-[:SignInWithIP]-(y:IP)  **WHERE** x.name == “John”  **RETURN** x.age, y.ip

此外，GQL 支持模式匹配语句的线性组合，意味着每个 MATCH 语句生成的结果表列可以驱动下一个 MATCH 语句。这种线性组合可以用两种风格来说明：

{

**SELECT** expression_list  **FROM** g **MATCH** graph_pattern1  **WHERE** xxx

**NEXT**

**SELECT** expression_list  **FROM** g **MATCH** graph_pattern2  **WHERE** yyy

**NEXT**

**SELECT** expression_list  **FROM** g **MATCH** graph_pattern3  **WHERE** zzz

}

{

**USE** g  **MATCH** graph_pattern1  **WHERE** xxx  **YIELD** expression_list

**NEXT**

**USE** g**MATCH** graph_pattern2  **WHERE** yyy  **YIELD** expression_list    **NEXT**

**USE** g  **MATCH** graph_pattern2  **WHERE** zzz  **RETURN** expression_list

}

除了查询语法，GQL 还支持类似 Linux 文件系统的目录层次结构，用于托管图模式及其目录对象。这种目录布局的物理设计是为了适应两级属性图建模的灵活性，其中一个图是逻辑容器，顶点和边是基本容器。这个设计最初是在 TigerGraph 和 Neo4j 之间非正式讨论的结果，因为我们最初受到我们的多图建模产品提供的启发——多个图可以共享一部分基本顶点和边的容器。经过多轮建设性讨论后，我们一致认为图模式将在这种目录布局下自由演变，特别是用于跨图共享和连接。

还有许多其他宝藏在 GQL 标准中；有兴趣的读者可以购买并学习这个标准。

### **如何学习呢？**

许多 TigerGraph 用户问我如何学习 GQL 或者提前看看。这里是我推荐的快速上手方法：

1.  研究核心模式匹配设计：通过阅读 [这篇](https://arxiv.org/abs/2112.06217) SIGMOD 论文来熟悉模式匹配语法。这为理解模式匹配的细微差别和设计理念奠定了基础。

1.  研究现有的 LDBC-SNB BI 基准查询，使用 [Cypher](https://github.com/ldbc/ldbc_snb_bi/tree/main/cypher/queries) 和 [GSQL](https://github.com/ldbc/ldbc_snb_bi/tree/main/tigergraph/queries_syntax_v3) 编写的查询。它们与 GQL 语法有很多相似之处。

1.  深入标准：阅读 628 页的 ISO GQL 标准（费用：217 瑞士法郎），全面理解该语言的规范。

最后也是最重要的一点，开始写作吧！

### **接下来 GQL 会怎么发展？**

GQL 1版本标志着持续旅程的开端。随着行业的发展，仍然需要解决一些挑战，包括图形视图支持、触发器支持、增强的图算法能力以及连接图和表等。在TigerGraph，我们正在积极实施GQL，以支持这一标准，保持我们对openCypher和GQL的坚定承诺。

回顾我们的旅程，Alin Deutsch、Yu Xu和我在2015年夏天在加州山景城开始了GSQL的创造，从SQL标准和模式匹配文献中汲取灵感。那个夏天，Alin和我实现了最初版本的GSQL，熬夜不少。在整个过程中，我们将最佳实践和现实世界的经验贡献给了ISO的GQL标准。我们与ISO标准委员会继续这个旅程，积极塑造GQL未来的版本。在这个过程中，我们在全球图形社区内结交了许多朋友，共同推动了这一美丽的标准。我们希望您能认识到这个社区为这一努力付出的奉献和热情，并花时间学习、欣赏设计，最重要的是开始使用它。

总之，GQL代表了标准化图查询语言的重大进步。无论您是经验丰富的数据库专业人员还是新手，现在是学习GQL并拥抱图数据管理未来的时候了。请继续关注GQL领域的激动人心的发展！

如果您想了解更多关于TigerGraph和GQL的信息，请联系我们：[[email protected]](/cdn-cgi/l/email-protection)。
