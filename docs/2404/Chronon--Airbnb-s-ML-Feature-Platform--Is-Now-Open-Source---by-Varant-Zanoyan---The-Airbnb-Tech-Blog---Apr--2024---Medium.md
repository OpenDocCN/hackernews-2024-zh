<!--yml

category: 未分类

日期：2024-05-27 13:03:39

-->

# Chronon，Airbnb 的 ML 特征平台，现已开源 | 作者：Varant Zanoyan | Airbnb 技术博客 | 2024 年 4 月 | Medium

> 来源：[https://medium.com/airbnb-engineering/chronon-airbnbs-ml-feature-platform-is-now-open-source-d9c4dba859e8](https://medium.com/airbnb-engineering/chronon-airbnbs-ml-feature-platform-is-now-open-source-d9c4dba859e8)

# **Chronon，Airbnb 的 ML 特征平台，现已开源**

提供观测性和管理工具的功能平台，允许 ML 从业者使用多种数据源，同时处理数据工程的复杂性，并提供低延迟的流处理。

作者：[Varant Zanoyan](https://www.linkedin.com/in/vzanoyan/)，[Nikhil Simha Raprolu](https://www.linkedin.com/in/nikhilsimha/)

*Chronon 允许 ML 从业者使用多种数据源作为特征转换的输入。它处理数据管道的复杂性，如批处理和流处理计算，提供低延迟的服务，并提供一系列的观测性和管理工具。*

**Airbnb 欣然宣布** [**Chronon**](https://www.chronon.ai)**，我们的 ML 特征平台，现已开源。加入我们的** [**社区 Discord 频道**](https://discord.gg/GbmGATNqqP)** 与我们交流。**

**我们很高兴与我们的合作伙伴 Stripe 一同宣布这一消息，他们是项目的早期采用者和共同维护者。**

本博文涵盖了 Chronon 的主要动机和功能。有关 Chronon 核心概念的概述，请参阅[此前的帖子](/airbnb-engineering/chronon-a-declarative-feature-engineering-framework-b7b8ce796e04)。

# 背景

我们建立 Chronon 是为了缓解 ML 从业者的一个常见痛点：他们花费大部分时间管理支持其模型的数据，而不是进行建模本身。

在 Chronon 之前，从业者通常使用以下两种方法之一：

1.  **离线-在线复制：** ML 从业者使用数据仓库中的数据训练模型，然后找出在在线环境中复制这些特征的方法。这种方法的好处是允许从业者利用完整的数据仓库，包括数据源和大规模数据转换的强大工具。缺点是没有明确的方法来为在线推理提供模型特征，导致一致性和标签泄漏，严重影响模型性能。

1.  **记录和等待：** ML 从业者从在线服务环境中获取可用数据，用于模型推理。他们将相关特征记录到数据仓库中。一旦积累了足够的数据，他们就会在日志上训练模型，并用相同的数据进行服务。这种方法的好处是保证一致性，减少数据泄漏的可能性。然而，主要缺点是可能导致长时间的等待，影响快速响应用户行为变化的能力。

Chronon 方法允许同时拥有最佳的两种情况。Chronon 要求机器学习从业者仅定义他们的特性一次，既能为模型训练的离线流程提供动力，也能为模型推断的在线流程提供动力。此外，Chronon 还提供了强大的工具，用于特性链接、可观察性和数据质量、特性共享和管理。

# 工作原理

下面我们通过从[快速入门指南](https://chronon.ai/getting_started/Tutorial.html)中衍生的一个简单示例来探索支持 Chronon 大部分功能的主要组件。您可以按照该指南运行此示例。

假设我们是一家大型在线零售商，我们已经检测到一种基于用户购买后退货的欺诈风险。我们希望训练一个模型来预测某个交易是否可能导致欺诈性退货。每当用户开始结账流程时，我们都会调用这个模型。

## 定义特性

**购买数据：** 我们可以将购买日志数据按用户级别聚合，以查看用户在我们平台上的先前活动。具体来说，我们可以计算他们在各种时间窗口内的购买金额的 SUM、COUNT 和 AVERAGE。

```
source = Source(
    events=EventSource(
        table="data.purchases", # This points to the log table in the warehouse with historical purchase events, updated in batch daily
        topic="events/purchases", # The streaming source topic
        query=Query(
            selects=select("user_id","purchase_price"), # Select the fields we care about
            time_column="ts") # The event time
    ))

window_sizes = [Window(length=day, timeUnit=TimeUnit.DAYS) for day in [3, 14, 30]] # Define some window sizes to use below

v1 = GroupBy(
    sources=[source],
    keys=["user_id"], # We are aggregating by user
    online=True,
    aggregations=[Aggregation(
            input_column="purchase_price",
            operation=Operation.SUM,
            windows=window_sizes
        ), # The sum of purchases prices in various windows
        Aggregation(
            input_column="purchase_price",
            operation=Operation.COUNT,
            windows=window_sizes
        ), # The count of purchases in various windows
        Aggregation(
            input_column="purchase_price",
            operation=Operation.AVERAGE,
            windows=window_sizes
        ), # The average purchases by user in various windows
        Aggregation(
            input_column="purchase_price",
            operation=Operation.LAST_K(10),
        ), # The last 10 purchase prices aggregated as a list
    ],
)
```

*这会创建一个 `GroupBy`，通过在各种时间窗口内对各种字段进行聚合，以 `user_id` 作为主键，将 `purchases` 事件数据转换为有用特性。*

这将原始购买日志数据转换为用户级别的有用特性。

**用户数据：** 将用户数据转换为特性稍微简单些，主要因为我们不必担心执行聚合操作。在这种情况下，源数据的主键与特性的主键相同，因此我们只需提取列值，而不必在行上执行聚合操作：

```
source = Source(
    entities=EntitySource(
        snapshotTable="data.users", # This points to a table that contains daily snapshots of all users
        query=Query(
            selects=select("user_id","account_created_ds","email_verified"), # Select the fields we care about
        )
    ))

v1 = GroupBy(
    sources=[source],
    keys=["user_id"], # Primary key is the same as the primary key for the source table
    aggregations=None, # In this case, there are no aggregations or windows to define
    online=True,
) 
```

*这会创建一个 `GroupBy`，从 `data.users` 表中提取维度作为特性，以 `user_id` 作为主键。*

**将这些特性组合在一起：** 接下来，我们需要将先前定义的特性组合成一个单一视图，既可以用于模型训练时的数据回填，也可以作为模型推断时的完整向量服务在线提供。我们可以通过使用 Join API 来实现这一点。

对于我们的用例，特别重要的是要在正确的时间戳下计算特性。因为我们的模型在结账流程开始时运行，我们希望在回填中使用相应的时间戳，以便模型训练时的特性值逻辑上匹配在线推断中模型将看到的内容。

这里是定义的样子。请注意，它将我们之前在 API 的 right_parts 部分定义的特性组合在一起（还有另一个称为 returns 的特性集）。

```
 source = Source(
    events=EventSource(
        table="data.checkouts", 
        query=Query(
            selects=select("user_id"), # The primary key used to join various GroupBys together
            time_column="ts",
            ) # The event time used to compute feature values as-of
    ))

v1 = Join(  
    left=source,
    right_parts=[JoinPart(group_by=group_by) for group_by in [purchases_v1, returns_v1, users]] # Include the three GroupBys
)
```

# 数据回填/离线计算

用户可能会首先使用上述 Join 定义运行回填，以生成用于模型训练的历史特性值。Chronon 在执行此回填时具有一些关键优势：

1.  **时间点准确性：** 注意作为上述连接“左”侧使用的源。它建立在“data.checkouts”源之上，每行都包括一个“ts”时间戳，对应于该特定结账的逻辑时间。每个特性计算都保证与该时间戳的窗口准确对应。因此，对于前用户购买总额的一个月求和，每一行将根据左侧源提供的时间戳进行计算。

1.  **偏斜处理：** Chronon 的回填算法经过优化，用于处理高度偏斜的数据集，避免令人沮丧的内存溢出和挂起作业。

1.  **计算效率优化：** Chronon 能够直接在后端内置多种优化，减少计算时间和成本。

# 在线计算

Chronon 在在线特性计算中抽象了大量复杂性。在上述示例中，它将根据特性是批处理特性还是流处理特性进行计算。

**批处理特性（例如上述的用户特性）**

由于用户特性是建立在批处理表之上，Chronon 将简单地运行每日批处理作业，计算新的特性数值，随着新数据进入批处理数据存储，并将其上传到在线 KV 存储以提供服务。

**流处理特性（例如上述的购买特性）**

购买特性建立在包括流组件的源上，源中包含“topic”，表明Chronon将除了实时更新的流作业外，仍然运行批处理上传作业。批处理作业负责：

1.  **种子值设定：** 对于长时间窗口，回放所有原始事件并不实际。

1.  **压缩“窗口中部”并提供尾部准确性：** 为了精确窗口准确性，我们需要在窗口的头部和尾部都有原始事件。

然后，流作业将更新写入 KV 存储，以保持在获取时特性值的最新状态。

# 在线服务 / 抓取 API

Chronon 提供了一个 API 来以低延迟获取特性。我们可以为单个 GroupBy（例如上述的用户或购买特性）或进行连接获取值。以下是一个这样的连接请求和响应的示例：

```
// Fetching all features for user=123
Map<String, String> keyMap = new HashMap<>();
keyMap.put("user", "123")
Fetcher.fetch_join(new Request("quickstart_training_set_v1", keyMap));
// Sample response (map of feature name to value)
'{"purchase_price_avg_3d":14.2341, "purchase_price_avg_14d":11.89352, ...}'
```

*Java 代码，用于获取用户123的所有特性。返回类型是特性名称到特性值的映射。*

上述示例使用了 Java 客户端。还有 Scala 客户端和 Python CLI 工具，用于简便测试和调试：

```
run.py --mode=fetch -k '{"user_id":123}' -n quickstart/training_set -t join

> {"purchase_price_avg_3d":14.2341, "purchase_price_avg_14d":11.89352, ...}
```

*利用 run.py CLI 工具来执行与上述 Java 代码相同的获取请求。run.py 是快速测试 Chronon 工作流（如获取）的便捷方式。*

另一种选择是将这些 API 封装成一个服务，并通过 REST 端点进行请求。这种方法在 Airbnb 中用于在非 Java 环境（如 Ruby）中获取特性。

# 在线-离线一致性

Chronon 不仅有助于在线-离线准确性，还提供了一种衡量方法。衡量流水线从在线获取请求的日志开始。这些日志包括请求的主键和时间戳，以及获取的特征值。然后，Chronon 将键和时间戳传递给 Join 回填作为左侧，要求计算引擎回填特征值。然后，它将回填的值与实际获取的值进行比较，以衡量一致性。

# 下一步是什么？

开源只是我们与 Stripe 及更广泛社区合作中令人兴奋旅程的第一步。

我们的愿景是创建一个平台，使机器学习实践者能够在如何利用其数据方面做出最佳决策，并尽可能地简化实施这些决策的过程。以下是我们目前用来制定路线图的一些问题：

**我们能进一步降低迭代和计算的成本吗？**

Chronon 已经为像 Airbnb 和 Stripe 这样的大公司处理的数据规模而建。然而，我们可以进一步优化我们的计算引擎，既可以减少计算成本，也可以减少创建和尝试新特征的“时间成本”。

**我们能让创建新特征变得多么简单？**

特征工程是人类表达其领域知识以创建模型可以利用的信号的过程。Chronon 可以集成自然语言处理技术，使机器学习实践者能够用自然语言表达这些特征点子，并生成工作中的特征定义代码作为迭代起点。

降低创建特征的技术门槛将进一步打开机器学习实践者与具有宝贵领域专业知识的合作伙伴之间新形式的协作之门。

**我们能改进模型维护的方式吗？**

用户行为的改变可能会导致模型性能的变化，因为模型训练所使用的数据不再适用于当前情况。我们设想一个能够检测这些变化并及早主动采取应对策略的平台，可以通过重新训练、添加新特征、修改现有特征或上述方式的组合来解决这些问题。

**平台本身是否能成为一个智能代理，帮助机器学习实践者构建和部署最佳模型？**

我们收集到平台层的元数据越多，它作为通用机器学习助手的能力就会变得越强大。

我们提到了创建一个平台的目标，该平台可以自动运行实验，使用新数据来识别改进模型的方法。这样的平台还可以通过允许机器学习从业者提出问题来帮助数据管理，比如“在建模这个用例时，哪些特征类型通常最有用？”或“哪些数据源可以帮助我创建捕捉与此目标有关的信号的特征？”一个能够回答这些问题的平台代表了智能自动化的下一个层次。

# 入门指南

这里有一些资源，可以帮助您开始或评估Chronon是否适合您的团队。

对这种类型的工作感兴趣吗？请查看我们的空缺职位[这里](https://careers.airbnb.com/) — 我们正在招聘。

# 致谢

**赞助商：** [Henry Saputra](mailto:henry.saputra@airbnb.com) [李毅](mailto:yi.li@airbnb.com) [宋杰](mailto:jack.song@airbnb.com)

**贡献者：** [侯鹏宇](mailto:pengyu.hou@airbnb.com) [Cristian Figueroa](mailto:cristian.figueroa@airbnb.com) [丁浩臻](mailto:haozhen.ding@airbnb.com) [王若男](mailto:sophie.wang@airbnb.com) [Vamsee Yarlagadda](mailto:vamsee.y@airbnb.com) [陈海春](mailto:haichun.chen@airbnb.com) [张东涵](mailto:donghan.zhang@airbnb.com) [岑昊](mailto:hao.cen@airbnb.com) [韩雨莉](mailto:yuli.han@airbnb.com) [Evgenii Shapiro](mailto:evgeny.shapiro@airbnb.com) [Atul Kale](http://atul.kale@airbnb.com) Patrick Yoon
