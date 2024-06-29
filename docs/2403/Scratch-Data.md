<!--yml

类别：未分类

日期：2024-05-27 14:32:29

-->

# Scratch Data

> 来源：[https://scratchdata.com/blog/building-a-ledger/](https://scratchdata.com/blog/building-a-ledger/)

# 构建一个可扩展的会计分类帐

有许多[博客](https://beancount.github.io/docs/the_double_entry_counting_method.html#double-entry-bookkeeping) [文章](https://www.moderntreasury.com/journal/accounting-for-developers-part-i)解释开发人员的复式记账基础知识。我打算分享一个简单 - 我认为是优雅的 - 数据库架构，用于记录和汇总分类帐条目。

工程师倾向于在会计术语上做手势，放弃像“借方”和“贷方”这样的术语 - 毕竟，为什么不只使用正数和负数？我认为这会导致混淆的结果。看看ledger-cli的[文档](https://www.ledger-cli.org/3.0/doc/ledger3.html#Stating-where-money-goes)中的例子：

> “当您查看账户总额时，您可能会惊讶地发现支出是正数，而收入是负数。这可能需要一些时间来适应，但是…”

我理解这个论点：因为收入的正常余额是贷方，而ledger-cli将贷方表示为负数，所以收入会显示为负数。但这与财务报表的准备方式完全不一致。

所以让我们设计一个可以在数据库中轻松建模并与实际会计一致的系统。

## 数据库选择的重要性

如果您正在构建一个拥有数百万交易的应用程序，您肯定会发现在这些列上调用SUM()是非常慢的。解决这个问题的一种方法是预先聚合数据，也许是按天，并将其存储在单独的表中。这可以在应用程序中完成，也可以是物化视图或触发器。

另一个选择是使用像Clickhouse这样的列式数据库。这就是我们在这里选择的方式：我们更喜欢保持数据模型简单，并使用技术快速处理数据，而不是复杂化数据插入过程。

## 科目表

我们首先要定义的是我们的账户列表。我们的账户表有3列：

+   **名称**。账户的名称（资产、负债等等）

+   **编号**。通常，账户被分配一个数字层次结构。例如：100 资产，101 现金，106 应收账款，等等。这里有用的是我们可以通过位置值来汇总子账户的价值。稍后我们会举个例子。

+   **正常余额**。在我们的模式中，我们定义`1`表示贷方，`-1`表示借方。用户看不到这一点！但在算术上很方便。

这是我们的表格，使用SQLite：

```
CREATE TABLE "accounts" (
    "name"      TEXT,
    "number"    INTEGER,
    "normal"    INTEGER
)
```

然后我们将填充一些账户：

| 名称 | 编号 | 正常 |
| --- | --- | --- |
| 资产 | 100 | 1 |
| 现金 | 110 | 1 |
| 商品 | 120 | 1 |
| 负债 | 200 | -1 |
| 递延收入 | 210 | -1 |
| 收入 | 300 | -1 |
| 费用 | 400 | 1 |
| 商品成本 | 410 | 1 |
| 股本 | 500 | -1 |
| 资本 | 510 | -1 |

注意现金和商品合并成资产（其他子账户也是如此）。所有资产账户都在“100”范围内。这是企业设置会计科目表的[典型方式](https://www.accountingtools.com/articles/chart-of-accounts-numbering.html)。

这个模式已经很有用了！仅凭借我们了解的账户及其正常余额，我们可以推导出会计等式：

```
SELECT
  group_concat(name , ' + ') AS expression
FROM accounts
GROUP BY normal;
```

| 表达式 |
| --- |
| 负债 + 收入 + 权益 + 预收收入 + 资本 |
| 资产 + 费用 + 现金 + 商品 + 销售成本 |

每行代表等式的一边。这是等式的一个相当详细的表达。我们可以通过选择以100结尾的账户来获取高级账户。这种算术方法非常巧妙，可以根据需要将数据汇总到任何细分。

```
SELECT group_concat(name, ' + ') AS expression
FROM accounts
WHERE number % 100 = 0
GROUP BY normal;
```

| 表达式 |
| --- |
| 负债 + 收入 + 权益 |
| 资产 + 费用 |

更好了！再加上一些SQL，我们可以输出等式本身：

```
select 
  max(left_side) || ' = ' || max(right_side) as equation 
from 
  (
    select 
      group_concat(
        case when normal == 1 then name end, ' + '
      ) as left_side, 
      group_concat(
        case when normal == -1 then name end, ' + '
      ) as right_side 
    from 
      accounts 
    where 
      number % 100 == 0 
    group by 
      normal
  );
```

## 交易

现在我们有一个可用的会计科目表，让我们添加交易。我们的交易表格非常简单明了。

```
CREATE TABLE "transactions"
  (
     "id"        INTEGER, 
     "date"      TEXT,
     "amount"    REAL,
     "account"   INTEGER,
     "direction" INTEGER
  ) 
```

+   **交易ID**。这将标识组成单笔交易的借方和贷方项目。

+   **日期**。交易日期。

+   **金额**。交易金额。通常是正数 - 我们不使用负数表示贷方，这有专门的列来表示。）

+   **账户**。这是此交易行项目的账户编号（例如，现金的编号是110）。

+   **方向**。我们选择`1`表示借方，`-1`表示贷方，与之前一样。这是算术操作的一个便利约定。

### 示例交易

作为示例，我们将记录多笔分户账的条目，展示开户余额、购买库存，然后将库存销售给客户。这篇文章不会详细说明每笔交易的会计解释（敬请关注！），但展示了如何利用这些数据进行基本查询。

在我们的数据库中，我们添加了以下行：

| id | 日期 | 金额 | 账户 | 方向 |
| --- | --- | --- | --- | --- |
| 0 | 2022-01-01 | 500.0 | 110 | 1 |
| 0 | 2022-01-01 | 500.0 | 510 | -1 |
| 1 | 2022-01-01 | 100.0 | 120 | 1 |
| 1 | 2022-01-01 | 100.0 | 110 | -1 |
| 2 | 2022-02-01 | 15.0 | 110 | 1 |
| 2 | 2022-02-01 | 15.0 | 210 | -1 |
| 3 | 2022-02-05 | 15.0 | 210 | 1 |
| 3 | 2022-02-05 | 15.0 | 300 | -1 |
| 4 | 2022-02-05 | 3.0 | 410 | 1 |
| 4 | 2022-02-05 | 3.0 | 120 | -1 |

注意，同一ID有多行。这是因为这些行都属于同一笔交易 - 该交易的借方金额等于贷方金额。

分解交易 `0`：

+   金额是$500。

+   第一行是借方，标记为`direction=1`。账户是现金，因为账号110与我们的账户表相符。由于现金与“资产”共享相同的前缀，因此此交易汇总到“资产”账户。

+   第二行是贷方，表示为 `direction=-1`。同样，账户号码 `510` 是资本，属于权益账户。

## 查询交易

现在我们有了完整的账簿条目，让我们运行一些 SQL 查询吧！这些都出奇地易懂 - 敢我说优雅。此模式保留了会计的规范，数据库操作廉价，输出与任何标准会计报表一致。

### 使用交易和账户详细信息进行连接

这是显示交易和账户信息的基本查询。

```
select
  *
from
  transactions
  left join accounts on transactions.account = accounts.number;
```

| id | 日期 | 金额 | 账户 | 方向 | 名称 | 编号 | 正常 |
| --- | --- | --- | --- | --- | --- | --- | --- |
| 2 | 2022-02-01 | 15.0 | 110 | 1 | 现金 | 110 | 1 |
| 2 | 2022-02-01 | 15.0 | 210 | -1 | 递延收入 | 210 | -1 |
| 3 | 2022-02-05 | 15.0 | 210 | 1 | 递延收入 | 210 | -1 |
| 3 | 2022-02-05 | 15.0 | 300 | -1 | 收入 | 300 | -1 |
| 1 | 2022-01-01 | 100.0 | 110 | -1 | 现金 | 110 | 1 |
| 1 | 2022-01-01 | 100.0 | 120 | 1 | 商品 | 120 | 1 |
| 4 | 2022-02-05 | 3.0 | 120 | -1 | 商品 | 120 | 1 |
| 4 | 2022-02-05 | 3.0 | 410 | 1 | 已售出商品成本 | 410 | 1 |
| 0 | 2022-01-01 | 500.0 | 510 | -1 | 资本 | 510 | -1 |
| 0 | 2022-01-01 | 500.0 | 110 | 1 | 现金 | 110 | 1 |

### 验证借方 = 贷方

此查询帮助我们验证总体上借方和贷方是否匹配。

```
select
  sum(case when direction == 1 then amount end) as DR,
  sum(case when direction == -1 then amount end) as CR
from
  transactions;
```

借方和贷方应该加起来为 0\. 我们可以像这样验证：

```
select
  sum(direction * amount)
from
  transactions;
```

| sum(direction * amount) |
| --- |
| 0.0 |

如果我们想要找到借贷不平衡的交易怎么办？

```
select
  id,
  sum(direction * amount) as s
from
  transactions
group by
  id
having
  s != 0;
```

### 余额

组装资产负债表很容易：

```
select
  (account) as a,
  name,
  sum(amount * direction * normal) as balance
from
  transactions
  left join accounts on a = accounts.number
group by
  name
order by
  a,
  name;
```

| a | 名称 | 余额 |
| --- | --- | --- |
| 110 | 现金 | 415.0 |
| 120 | 商品 | 97.0 |
| 210 | 递延收入 | 0.0 |
| 300 | 收入 | 15.0 |
| 410 | 已售出商品成本 | 3.0 |
| 510 | 资本 | 500.0 |

此查询最重要的部分是 `SUM(amount * direction * normal)`。这确保我们正确增加和减少余额，并确保余额为正。

如果我们需要一个报告，将子账户汇总到主账户中，我们可以使用算术来找到父账户号码。

```
select
  ((account / 100) * 100) as a,
  name,
  sum(amount * direction * normal) as balance
from
  transactions
  left join accounts on a = accounts.number
group by
  name
order by
  a,
  name;
```

| a | 名称 | 余额 |
| --- | --- | --- |
| 100 | 资产 | 512.0 |
| 200 | 负债 | 0.0 |
| 300 | 收入 | 15.0 |
| 400 | 费用 | 3.0 |
| 500 | 股本 | 500.0 |

在这里，我们将现金和商品合并为资产。

最后，这是我们如何以人类可读的方式显示所有交易：

```
select
  id,
  date,
  name,
  case when direction == 1 then amount end as DR,
  case when direction == -1 then amount end as CR
from
  transactions
  left join accounts on account = accounts.number
order by
  id,
  date,
  CR,
  DR;
```

| id | 日期 | 名称 | 借方 | 贷方 |
| --- | --- | --- | --- | --- |
| 0 | 2022-01-01 | 现金 | 500.0 |  |
| 0 | 2022-01-01 | 资本 |  | 500.0 |
| 1 | 2022-01-01 | 商品 | 100.0 |  |
| 1 | 2022-01-01 | 现金 |  | 100.0 |
| 2 | 2022-02-01 | 现金 | 15.0 |  |
| 2 | 2022-02-01 | 递延收入 |  | 15.0 |
| 3 | 2022-02-05 | 递延收入 | 15.0 |  |
| 3 | 2022-02-05 | 收入 |  | 15.0 |
| 4 | 2022-02-05 | 已售出商品成本 | 3.0 |  |
| 4 | 2022-02-05 | 商品 |  | 3.0 |

## 与临时数据一起进行流式处理

最后，有人可能会问：我们如何将所有这些数据导入数据库？如果使用数据仓库（Clickhouse、Snowflake 等），则每次交易发生时进行单独的 INSERT 语句是不可能的。最终会设置一个每晚批量加载过程。

如果您能够在交易发生时实时流传日志条目，那会怎么样？您可以得到最新的资产负债表。幸运的是，Scratch Data 让这变得非常简单。

您可以将数据流到 Scratch，我们将自动收集它，创建数据库模式，并安全地批量插入。

### 流式传输 Stripe 和 Shopify 数据

Stripe 和 Shopify 都有 Webhooks 用于跟踪每一笔交易。通过我们的 API 端点，您可以将 Scratch Data 设置为 Webhook 的目标，每笔交易将实时流入数据库。查看我们的博客文章，例如 [Stripe](/blog/stripe-data-ingest/) 和 [Shopify](/blog/shopify-data-ingest/)。

### 从代码流式传输

如果您想从代码中流传数据 - 也许您有自己的 Webhook 或应用程序代码 - 这也非常简单！以下是 JSON 的样式：

```
{
    "date": "2022-01-01",
    "amount": 500.00,
    "account": 110,
    "direction": 1
}
```

然后将其 POST 出去：

```
$ curl -X POST "https://api.scratchdata.com/data?table=transactions" \
    --json '{"amount": 500.00 ...}'
```

从这里，数据可以流向您的应用程序（如果您正在构建用户界面的仪表板）或作为 Excel 文件。

## 结论

这希望是一个关于如何设计具有高概率生成可由您的财务团队使用的数据的分户账系统的起点，使用正确的术语。

如果你想了解更多关于我们如何帮助您构建这样的系统，请[联系我们](https://q29ksuefpvm.typeform.com/to/baKR3j0p#source=building_a_ledger)！
