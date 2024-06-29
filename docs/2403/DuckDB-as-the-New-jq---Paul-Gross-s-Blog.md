<!--yml

类别：未分类

日期：2024-05-29 12:32:42

-->

# DuckDB 作为新的 jq - Paul Gross 的博客

> 来源：[https://www.pgrs.net/2024/03/21/duckdb-as-the-new-jq/](https://www.pgrs.net/2024/03/21/duckdb-as-the-new-jq/)

最近，我对 [DuckDB](https://duckdb.org/) 项目很感兴趣（类似于面向数据应用的 [SQLite](https://www.sqlite.org/)）。其中一个令人惊讶的特性是，它包含许多数据导入工具，无需额外的依赖项。这意味着它可以原生地读取和解析 JSON 作为数据库表，还支持多种其他格式。

我每天都在大量使用 JSON，并且在探索文档时常常使用 [jq](https://jqlang.github.io/jq/)。我喜欢 `jq`，但发现它难以使用。语法非常强大，但除了选择字段之外，我想做其他操作时都必须研究文档。

一旦我了解到 DuckDB 可以直接将 JSON 文件读入内存，我意识到我可以用它来替代目前使用 `jq` 的许多工作。与复杂和定制的 `jq` 语法相比，我对 SQL 非常熟悉，几乎每天都在使用。

这里是一个例子：

首先，我们获取一些示例 JSON 进行玩耍。我使用 GitHub API 从 golang org 获取了存储库信息：

```
% curl 'https://api.github.com/orgs/golang/repos' > repos.json 
```

现在，作为一个示例问题来回答，让我们获取一些关于开源许可证类型的统计数据。

JSON 的结构如下：

```
[  {  "id":  1914329,  "name":  "gddo",  "license":  {  "key":  "bsd-3-clause",  "name":  "BSD 3-Clause \"New\" or \"Revised\" License",  ...  },  ...  },  {  "id":  11440704,  "name":  "glog",  "license":  {  "key":  "apache-2.0",  "name":  "Apache License 2.0",  ...  },  ...  },  ...  ] 
```

这可能不是最佳方式，但这是我在搜索和阅读一些文档后，使用 `jq` 拼凑起来的东西：

```
 % cat repos.json | jq \
  'group_by(.license.key)
  | map({license: .[0].license.key, count: length})
  | sort_by(.count)
  | reverse'
[
  {
    "license": "bsd-3-clause",
    "count": 23
  },
  {
    "license": "apache-2.0",
    "count": 5
  },
  {
    "license": null,
    "count": 2
  }
] 
```

这是在 DuckDB 中使用 SQL 的样子：

```
% duckdb -c \
  "select license->>'key' as license, count(*) as count \ from 'repos.json' \ group by 1 \ order by count desc"
┌──────────────┬───────┐
│   license    │ count │
│   varchar    │ int64 │
├──────────────┼───────┤
│ bsd-3-clause │    23 │
│ apache-2.0   │     5 │
│              │     2 │
└──────────────┴───────┘ 
```

对我来说，这个 SQL 更简单，我能够在不查看任何文档的情况下编写它。唯一棘手的部分是使用 `->>` 操作符查询嵌套 JSON。然而，其语法与 [PostgreSQL JSON 函数](https://www.postgresql.org/docs/current/functions-json.html) 相同，因此我对此很熟悉。

如果我们确实需要 JSON 格式的输出，DuckDB 也有相应的标志：

```
% duckdb -json -c \
  "select license->>'key' as license, count(*) as count \ from 'repos.json' \ group by 1 \ order by count desc"
[{"license":"bsd-3-clause","count":23},
{"license":"apache-2.0","count":5},
{"license":null,"count":2}] 
```

即使在使用 DuckDB 进行大量工作之后，我们仍然可以使用 `jq` 进行漂亮的打印：

```
% duckdb -json -c \
  "select license->>'key' as license, count(*) as count \ from 'repos.json' \ group by 1 \ order by count desc" \
  | jq
[
  {
    "license": "bsd-3-clause",
    "count": 23
  },
  {
    "license": "apache-2.0",
    "count": 5
  },
  {
    "license": null,
    "count": 2
  }
] 
```

JSON 只是将数据导入 DuckDB 的多种方法之一。这种方法也适用于 CSV、parquet、Excel 文件等。

而我可以选择创建表并将数据持久化存储，但通常只是查询数据，不需要持久性。

阅读更多关于 DuckDB 在此博客文章中出色的 JSON 支持：[深度嵌套 JSON 的解析，逐向量进行](https://duckdb.org/2023/03/03/json.html)

**更新：**

我还了解到 DuckDB 不仅可以从本地文件，而且可以直接从 URL 读取 JSON：

```
% duckdb -c \
  "select license->>'key' as license, count(*) as count \ from read_json('https://api.github.com/orgs/golang/repos') \ group by 1 \ order by count desc" 
```
