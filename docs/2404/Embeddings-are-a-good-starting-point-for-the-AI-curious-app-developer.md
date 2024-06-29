<!--yml

category: 未分类

date: 2024-05-27 13:16:22

-->

# 对于对AI感兴趣的应用程序开发者来说，嵌入是一个很好的起点

> 来源：[https://bawolf.substack.com/p/embeddings-are-a-good-starting-point](https://bawolf.substack.com/p/embeddings-are-a-good-starting-point)

向量嵌入对我来说是一个Overton窗口转移的体验，不是因为它们是足够先进的技术，无法与魔法区分开来，而是相反。一旦开始使用它们，就觉得显而易见，这才是搜索体验一直应该有的：不再是“你是怎么做到的？”而是更平凡的，“为什么这不普及到每个地方呢？”

如果您是一个寻找借口涉足这个新AI世界的应用程序开发者，这感觉就是一个合适的起点。嵌入只是一组数字数组，但它们包含了大量人类知识的压缩形式，并将过去需要大量专门项目才能完成的功能缩减为个体产品工程师可以承担的项目。

使用嵌入的工具选项非常多。我将突出我们的选择，并指出您可能希望根据自己的情况做出不同选择的地方。以下是我希望您能带走的一些要点：

+   向量嵌入适用于搜索和推荐，因为它们擅长衡量与任意输入的相似性。这甚至适用于不同的口头语言，如法语或日语。

+   [Pgvector](https://github.com/pgvector/pgvector/) 是一个在不添加新服务的情况下存储和查询嵌入的Postgres扩展。它之所以强大，是因为可以将标准SQL逻辑与嵌入操作结合起来。

+   与LLM不同，使用嵌入感觉像是常规的确定性代码。

我和我的朋友Charlie Yuan建立了这个小型[图标应用](https://www.v0.app)，帮助人们发现图标。非常简短而甜美。我们有您可以查询、收藏并添加到项目中的图标集。

有许多[专业的向量数据库](https://lakefs.io/blog/12-vector-databases-2023/)可供选择。但我们选择了与业务逻辑（如过滤和评分）融合的Postgres和[Pgvector](https://github.com/pgvector/pgvector/)。虽然它不是最快的向量数据库，但我们不希望在多个数据源之间比较结果。Pgvector可能已经为您喜欢的数据库客户端[提供了一个库](https://github.com/pgvector/pgvector)。我们的项目完全使用Typescript，并且使用了[drizzle-orm](https://github.com/pgvector/pgvector-node?tab=readme-ov-file#drizzle-orm)。文档将是安装文档的更为全面的地方，所以我略过了这部分，专注于您可以构建的功能。

使用pgvector设置后，我们创建了一种策略，将我们的图标数据编码为矢量嵌入。嵌入是多维空间中的点，高达数千个维度。不幸的是，该网格的轴并非像“大小”或“亮度”这样人类可理解的概念。它们有点像黑匣子。幸运的是，像任何良好的抽象一样，它们是带有良好API的黑匣子。

最佳实践似乎是找到最能代表人们想搜索的细节，并创建一个输出该细节的字符串的函数。我们的图标可以作为许多基于用例的‘类别’的一部分，并且有许多描述性的‘标签’与之相关联。我们将这些信息与图标名称一起编码，因为这最能代表图标的内容。而图标所属的图标集名称或其尺寸则不相关。我们生成的字符串基本上看起来像这样：

```
const createIconEmbeddingsString = (icon) => `icon: "${icon.name}", categories: [${categories}] tags: [${tags}]`;
```

接下来，我们选择了一个嵌入模型。[OpenAI的嵌入模型](https://platform.openai.com/docs/guides/embeddings)可能会很好地工作。我们正在使用他们的`text-embedding-3-small`。如果您想深入了解，请查看[排行榜](https://huggingface.co/spaces/mteb/leaderboard)，并选择最符合您需求的模型。无论您使用嵌入API还是自托管的开源选项，接口都应该是文本输入和嵌入输出。

许多网站都实施了搜索，但大多数图标网站是通过[文本](https://icon-sets.iconify.design/?query=woof) [匹配](https://react-icons.github.io/react-icons/search/#q=woof) 或全文搜索来实现搜索。如果您正在寻找狗图标，它们会在图标元数据中搜索包含‘dog’的图标。如果它们想要更[巧妙](https://fontawesome.com/search?q=woof&o=r)，它们会想出与‘dog’相关的词袋，比如‘k9’、‘puppy’和‘woof’，以捕捉近似匹配。这相当脆弱。必须为每个图标选择标签；如果漏掉了一个重要的标签，用户就找不到他们想要的内容。

当我们搜索‘[狗](https://www.v0.app/search?g=popularity&q=dog&qid=nmezx8efde3pvydym2f2sk9n)’时，我们的应用程序会获得相关的结果。我们还可以通过测量您搜索查询的嵌入与每个图标的余弦相似性来获得‘[小狗](https://www.v0.app/search?g=popularity&q=puppy&qid=v27kjdebk8ex7n30qe4b5tfs)’的可靠结果，而不需要使用词袋。有多种方法来衡量嵌入之间的相似度，但OpenAI的嵌入设计得非常适合余弦距离。余弦相似性只是余弦距离的相反。无论何时可以通过余弦距离排序来利用[索引](https://github.com/pgvector/pgvector?tab=readme-ov-file#why-isnt-a-query-using-an-index)。

```
cosine_similarity(x,y) = 1 - cosine distance(x,y)
```

你甚至可以尝试像“[猎犬](https://www.v0.app/search?q=hound)”，“[贵宾犬](https://www.v0.app/search?q=poodle)”或者我的最爱“[萨摩耶](https://www.v0.app/search?q=samoyed)”这样的狗品种。它几乎就像魔法一样运行。但这还不是全部；它也适用于其他语言。尝试“[chien](https://www.v0.app/search?g=popularity&q=chien&qid=mcbp6rk9z3y8lb1emt1zblev)”甚至“[犬](https://www.v0.app/search?g=popularity&q=%E7%8A%AC&qid=hd8whx733f3rljhadasgfzb9)”！使用pgvector，我们可以通过简单的SQL查询获得这些结果。

```
SELECT
  1 - cosine_distance (search_query.embedding,
    icon.embedding) as similarity,
  *
FROM
  icon
  join search_query on search_query.text = 'dog'
ORDER BY
  cosine_distance (search_query.embedding, icon.embedding) ASC
LIMIT 50;
```

由于我们按距离排序，每次搜索都会按接近程度返回表中的每个结果。我们通过固定数量截断结果，以使其可管理，将查询限制在前50个结果。像查询余弦相似度小于0.8的结果那样使用任意距离截断是很诱人的。不幸的是，一个查询要产生正确感觉的绝对距离可能与另一个查询大不相同。尽可能地通过结果数量而不是最小值来限制。

如果我们只想要相似性搜索，任何向量数据库都可以，但Postgres允许我们在其上叠加更多功能。初始搜索跨越所有图标集中的所有图标，但我们用户的风格可能只匹配某些图标集。我们可以像在任何其他Postgres查询中那样按值进行过滤。

```
SELECT
  1 - cosine_distance (search_query.embedding,
    icon.embedding) AS similarity,
  *
FROM
  icon
  JOIN search_query ON search_query.text = 'dog'
  JOIN icon_set ON icon_set.slug = icon.icon_set_slug
WHERE
  icon_set.slug in('lucide', 'mdi')
ORDER BY
  cosine_distance (search_query.embedding,
    icon.embedding) ASC
LIMIT 50;
```

这是确定性的；搜索“dog”的每个人将获得相同的结果。然而，输入仍然是无界的，因此嵌入式搜索并不保证为每个输入产生最佳结果。我们可以尝试不同的嵌入模型或编码图标到嵌入中以改善系统。

嵌入式搜索通常会将正确的图标放在页面上，但正确的图标并不总是第一个结果。我们可以制作一个简单的算法，根据用户反馈调整并随时间改进。为此，我们将统计用户每次点击特定搜索查询的图标的次数。在排名搜索结果时，我们将为每个图标创建一个结合嵌入式搜索和点击数据的得分。

这是一个简单的排名算法。细节看起来有些凌乱，因为有些空值检查和单位转换，但基本原理如下。

1.  获取搜索查询的图标的余弦相似度。结果将是一个介于0和1之间的数字。乘以0.5。

1.  对每个图标的点击次数除以该查询中点击次数最多的图标的点击次数。这将最常点击的图标归一化为1，最少点击的为0。乘以0.5。

1.  最终得分是这两个值相加，范围在0-1之间。

```
SELECT
  (
    0.5 * COALESCE(      -- so nulls are turned into 0
     1 - cosine_distance (search_query.embedding, icon.embedding),
     0
    ) + 0.5 *     -- so clicks matter less than embeddings.
    COALESCE( 
      search_query_selection. "count"::decimal / 
        max(search_query_selection. "count") OVER (),
    0)
  ) AS score,
  icon.*
FROM
  icon
  LEFT JOIN search_query_selection
    ON icon.id = search_query_selection.icon_id
  LEFT JOIN search_query
    ON search_query.text = 'dog'
      AND search_query.id = search_query_selection.search_query_id
  ORDER BY score DESC
  LIMIT 50;
```

应该等权重考虑向量嵌入距离和点击次数吗？点击次数的数量级是否比原始数字更重要？这个算法可能需要调整，但这只是数据库处理计算的一个示例。

对于一个独立的向量数据库，我们可能需要从两个数据库中获取所有图标的值，然后在应用代码中比较它们，或者做出类似从向量数据库中拉取100个结果并根据Postgres查询的点击分数对这些结果进行筛选等权衡。相反，我们只需查询结果并显示它们。

此外，我们在每个[图标页面](https://www.v0.app/icon/lucide/arrow-up)的内容前向部分都包含了同一图标集和类别中的其他图标。这样，您在查看`arrow-up`时可以看到其他“导航”图标。不幸的是，我们的图标集并非所有都有类别。在这些情况下，我们使用嵌入来创建类似的图标部分。我们可以不需要从用户输入获取嵌入，而是可以将选择的图标的嵌入与相同的余弦相似度测量进行比较。

```
WITH current_icon AS (
    SELECT
      embedding,
      slug,
      icon_set_slug
    FROM
      icon
    WHERE
      icon_set_slug = 'lucide'
      AND slug = 'activity'
)
SELECT
    *
FROM
    icon
INNER JOIN current_icon ON
icon.icon_set_slug = current_icon.icon_set_slug
AND icon.slug != current_icon.slug
ORDER BY
    1 - cosine_distance (
    icon.embedding,
    current_icon.embedding
)
LIMIT 50;
```

我希望您在使用我们的应用和向量嵌入时有一个很好的体验！我们要特别感谢所有推动技术前沿的工程师们，正是他们让我们能够站在他们的肩膀上。希望这份谦逊的贡献能帮助您更轻松地为您的应用添加嵌入特性！

这里是我们实现决策的摘要以及一些其他选择的信息来源。

我们选择了[pgvector](https://github.com/pgvector/pgvector)/Postgres，但也有很多[其他选择](https://lakefs.io/blog/12-vector-databases-2023/)，包括一些适用于其他标准数据库如MongoDB的选择。

我们使用Typescript并选择了[drizzle-orm](https://github.com/pgvector/pgvector-node?tab=readme-ov-file#drizzle-orm)。我还在Phoenix elixir生态系统中使用了[elixir库](https://github.com/pgvector/pgvector-elixir)与Ecto。您可以在[这里](https://github.com/pgvector/pgvector/?tab=readme-ov-file#languages)找到适合您客户端的库。

我们的应用托管在[Neon](https://neon.tech/)上。此外，我曾经使用过[fly.io](https://fly.io/)，尽管当时他们的选择并不是*真正*的托管实例。现在，他们有了一个[托管解决方案](https://fly.io/docs/reference/supabase/)，与Supabase合作。如果您想查找其他服务商，Pgvector在[此git问题](https://github.com/pgvector/pgvector/issues/54)中列出了一份笨拙的支持列表。

我们选择了OpenAI的`text-embedding-3-small`。如果您想尝试其他内容，请查看Huggingface的[排行榜](https://huggingface.co/spaces/mteb/leaderboard)。

我们嵌入了键值对的字符串，这些键值对最能描述我们的图标。看起来主要参与者并没有做出什么非常疯狂的事情；重要的是是否要嵌入键，或者仅仅是值，以及你的记录的哪些属性是相关的。OpenAI的[菜谱](https://cookbook.openai.com/examples/vector_databases/readme)中的其他示例展示了[其他的](https://github.com/vercel/examples/blob/main/storage/postgres-pgvector/prisma/seed.ts#L35) [选择](https://github.com/neondatabase/yc-idea-matcher/blob/18eb9dd6ddd14eeeb2167d78088f092ab6882f42/generate-embeddings.ts#L51)。

我们使用余弦相似度作为我们的距离函数，因为这是OpenAI在他们的嵌入文档中[推荐的](https://platform.openai.com/docs/guides/embeddings/which-distance-function-should-i-use)方法。其他嵌入可能针对不同的策略进行了优化。Pgvector支持l2距离、内积和余弦距离，详见[此处](https://github.com/pgvector/pgvector?tab=readme-ov-file#distances)。

在这些示例中，我们将查询限制在前50个结果之内。你也可以将搜索限制在某个距离阈值之上或之下。这似乎并不是非常可靠。相对距离似乎比离散数值更有意义。如果你坚持使用阈值，我建议将其设置得相当宽松，如0.1或0.05，并且结合使用限制条件。你可以通过使用广范围来避免返回无关紧要的长尾结果。
