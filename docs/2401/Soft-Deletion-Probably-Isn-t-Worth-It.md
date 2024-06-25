<!--yml

category: 未分类

date: 2024-05-27 14:39:18

-->

# 软删除可能并不值得

> 来源：[https://brandur.org/soft-deletion](https://brandur.org/soft-deletion)

任何看过几个不同的生产数据库环境的人都可能熟悉“软删除”模式——表格不直接通过 `DELETE` 语句删除数据，而是增加了额外的 `deleted_at` 时间戳，并通过更新语句执行删除：

```
UPDATE foo SET deleted_at = now() WHERE id = $1; 
```

软删除背后的概念是使删除更安全和可逆。一旦记录被硬 `DELETE`，它可能从技术上来说仍然可以通过深入存储层来恢复，但可以说这非常难以实现。理论上，通过软删除，您只需将 `deleted_at` 设置为 `NULL`，然后完成：

```
-- and like magic, it's back!!
UPDATE foo SET deleted_at = NULL WHERE id = $1; 
```

但是这种技术有一些主要缺点。首先是软删除逻辑渗透到代码的各个部分。我们所有的选择都看起来像这样：

```
SELECT *
FROM customer
WHERE id = @id
    AND deleted_at IS NULL; 
```

忘记 `deleted_at` 上的额外谓词可能会带来危险后果，因为它意外返回了不再应该看到的数据。

一些 ORM 或 ORM 插件通过将额外的 `deleted_at` 条件自动链接到每个查询上，使这个过程变得更加容易（例如看看 [`acts_as_paranoid`](https://github.com/ActsAsParanoid/acts_as_paranoid)），但仅仅因为它被隐藏了并不一定会使事情变得更好。如果操作员直接查询数据库，他们更有可能忘记 `deleted_at`，因为通常 ORM 会为他们完成这项工作。

软删除的另一个后果是外键实际上会丢失。

外键的主要优势在于它们保证引用完整性。例如，假设您有一个客户在一个表中，可能会引用另一个表中的多个发票。如果没有外键，您可以删除一个客户，但可能会忘记删除其发票，从而留下一堆与已删除的客户相关的孤立发票。

在有外键的情况下，尝试删除该客户而不先删除发票会导致错误：

```
ERROR:  update or delete on table "customer" violates
    foreign key constraint "invoice_customer_id_fkey" on table "invoice"

DETAIL:  Key (id)=(64977e2b-40cc-4261-8879-1c1e6243699b) is still
    referenced from table "invoice". 
```

与其他关系型数据库功能（如预定义模式、类型和检查约束）一样，数据库帮助保持数据有效。

但是使用软删除，情况就不同了。一个客户可能会被软删除，并将其 `deleted_at` 标志设置，但我们现在又回到了可以忘记对其发票执行相同操作的情况。他们的外键仍然有效，因为客户记录在技术上仍然存在，但没有等效的检查以确保发票也被软删除，因此您可能会发现客户被“删除”，但其发票仍然存在。

过去几年在消费者数据保护方面取得了重大进展，例如在欧洲推出了 [GDPR](https://en.wikipedia.org/wiki/General_Data_Protection_Regulation)。因此，通常情况下不建议数据无限期保留，而软删除行默认情况下会出现这种情况。

因此，你最终可能会编写一个硬删除过程，该过程查看超出某个时间范围内的软删除记录，并从数据库中永久删除它们。

但是软删除使得这项工作更加困难的是，由于记录不能被删除而不确保其所有依赖项也被删除（`ON DELETE CASCADE` 可以自动完成此操作，但是级联的使用相当危险，不建议用于更高的数据保真度）。

幸运的是，在支持 CTEs（通用表表达式）的系统中仍然可以执行此操作，但是您将得到一些非常复杂的查询。以下是我最近编写的一个片段，它通过一次操作将所有外键满足并删除所有内容：

```
WITH team_deleted AS (
    DELETE FROM team
    WHERE (
        team.archived_at IS NOT NULL
        AND team.archived_at < @archived_at_horizon::timestamptz
    )
    RETURNING *
),

--
-- team resources
--
cluster_deleted AS (
    DELETE FROM cluster
    WHERE team_id IN (
        SELECT id FROM team_deleted
    )
    OR (
        archived_at IS NOT NULL
        AND archived_at < @archived_at_horizon::timestamptz
    )
    RETURNING *
),
invoice_deleted AS (
    DELETE FROM invoice
    WHERE team_id IN (
        SELECT id FROM team_deleted
    )
    OR (
        archived_at IS NOT NULL
        AND archived_at < @archived_at_horizon::timestamptz
    )
    RETURNING *
),

--
-- cluster + team resources
--
subscription_deleted AS (
    DELETE FROM subscription
    WHERE cluster_id IN (
        SELECT id FROM cluster_deleted
    ) OR team_id IN (
        SELECT id FROM team_deleted
    )
    RETURNING *
)

SELECT 'cluster', array_agg(id) FROM cluster_deleted
UNION ALL
SELECT 'invoice', array_agg(id) FROM invoice_deleted
UNION ALL
SELECT 'subscription', array_agg(id) FROM subscription_deleted
UNION ALL
SELECT 'team', array_agg(id) FROM team_deleted; 
```

完整版本的长度是原文的五倍，包括 30 张单独的表。这个工作虽然有效，但是过于复杂，成为一个负担。

即使进行了大量测试，这种查询仍可能成为可靠性问题，因为如果未来添加了新的依赖项，但忘记了更新查询，那么在一年后（或者硬删除的时间跨度是多长）突然就会开始失败。

再次强调，软删除理论上是对意外数据丢失的一种防范。作为反对的最后一个论点，我想请你考虑现实中是否真的有人做过恢复删除的事情。

当我在 Heroku 工作时，我们使用了软删除。

当我在 Stripe 工作时，我们使用了软删除。

现在我工作的这个岗位上，我们使用软删除。

据我所知，在过去的十多年里，任何一个地方都从未真正使用软删除来恢复已删除的内容。

最大的原因是几乎总是**数据删除还会产生非数据副作用**。可能已经调用外部系统来存档那里的记录，对象可能已经在 blob 存储中被移除，或者服务器已经关闭。该过程不能简单地通过在 `deleted_at` 上设置 `NULL` 来撤消 —— 所有这些其他操作也需要等价的撤消，而这些撤消很少存在。

在 Heroku 有几个案例，其中一个重要的用户不小心删除了一个应用并希望恢复它。我们有软删除，理论上其他删除副作用可以被撤消，但我们还是决定不去尝试，因为以前从未有人这样做过，而在紧急情况下尝试这样做是完全错误的时间 —— 我们几乎肯定会出错，让用户处于糟糕的状态。相反，我们通过创建一个新应用并帮助他们将已删除应用的环境和数据复制到新应用中来前进。因此，即使在理论上软删除最有用的地方，我们仍然没有使用它。

尽管我从未见过实际中的未删除操作，但软删除并不是完全无用的，因为我们偶尔会使用它来引用已删除的数据 - 通常是一个手动过程，有人想要查看一个已删除的对象，以帮助解决支持票证或尝试消除错误。

虽然我会因为上述列出的缺点而反对传统的软删除模式，但幸运的是，有一个折衷方案。

不是将删除的数据保留在删除的表中，可以有一个专门用于存储所有删除的数据的新关系，并且具有灵活的`jsonb`列，以便可以捕获任何其他表的属性：

```
CREATE TABLE deleted_record (
    id uuid PRIMARY KEY DEFAULT gen_ulid(),
    deleted_at timestamptz NOT NULL default now(),
    original_table varchar(200) NOT NULL,
    original_id uuid NOT NULL,
    data jsonb NOT NULL
); 
```

然后，删除变成了这样：

```
WITH deleted AS (
    DELETE FROM customer
    WHERE id = @id
    RETURNING *
)
INSERT INTO deleted_record
		(original_table, original_id, data)
SELECT 'foo', id, to_jsonb(deleted.*)
FROM deleted
RETURNING *; 
```

与`deleted_at`相比，这确实有一个缺点 - 选择列到`jsonb`的过程并不容易逆转。虽然这是可能的，但可能涉及构建一次性查询和手动干预。但再次，这可能没关系 - 考虑一下你真正要尝试恢复数据的频率。

这种技术解决了上面概述的所有问题：

+   对于正常的、非删除的数据的查询不再需要在各处包含`deleted_at IS NULL`。

+   外键仍然有效。尝试删除记录而不获取其依赖项是一个错误。

+   为了符合监管要求，删除旧记录变得非常非常容易：`DELETE FROM deleted_record WHERE deleted_at < now() - '1 year'::interval`。

删除的数据稍微难以获取，但并不多，并且仍然保留在以防有人需要查看它的情况下。
