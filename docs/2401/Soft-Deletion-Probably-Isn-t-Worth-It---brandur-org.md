<!--yml

类别：未分类

日期：2024-05-28 17:48:20

-->

# 软删除可能不值得——brandur.org

> 来源：[https://brandur.org/soft-deletion](https://brandur.org/soft-deletion)

见过几个不同的生产数据库环境的人可能对“软删除”模式很熟悉——而不是通过`DELETE`语句直接删除数据，表会得到一个额外的`deleted_at`时间戳，并且删除是通过更新语句来执行的：

```
UPDATE foo SET deleted_at = now() WHERE id = $1; 
```

软删除背后的概念是使删除更安全、可逆。一旦记录被硬`DELETE`删除，从技术上讲，它可能仍然可以通过深入存储层来恢复，但可以说这确实很难。理论上，通过软删除，你只需将`deleted_at`设置为`NULL`，然后就完成了：

```
-- and like magic, it's back!!
UPDATE foo SET deleted_at = NULL WHERE id = $1; 
```

但这种技术也有一些重大缺点。首先，软删除逻辑会渗透到代码的各个部分。我们所有的选择看起来都像这样：

```
SELECT *
FROM customer
WHERE id = @id
    AND deleted_at IS NULL; 
```

忘记了在`deleted_at`上添加额外的谓词可能会带来危险后果，因为它会意外地返回不再应该看到的数据。

一些ORM或ORM插件通过自动将额外的`deleted_at`子句链式添加到每个查询中（例如，参见[`acts_as_paranoid`](https://github.com/ActsAsParanoid/acts_as_paranoid)），但只是因为它被隐藏了，并不一定使事情变得更好。如果操作员曾经直接查询过数据库，他们更有可能忘记`deleted_at`，因为通常ORM会替他们完成这项工作。

软删除的另一个后果是外键实际上丢失了作用。

外键的主要优势在于它们保证了引用完整性。例如，假设你有一个表中的客户可能参考另一个表中的多张发票。如果没有外键，你可以删除一个客户，但忘记删除其发票，从而留下一堆孤立的发票，这些发票引用了已经消失的客户。

对于外键，试图在不先删除发票的情况下删除该客户是一个错误：

```
ERROR:  update or delete on table "customer" violates
    foreign key constraint "invoice_customer_id_fkey" on table "invoice"

DETAIL:  Key (id)=(64977e2b-40cc-4261-8879-1c1e6243699b) is still
    referenced from table "invoice". 
```

与其他关系数据库功能（如预定义模式、类型和检查约束）一样，数据库有助于保持数据的有效性。

但是使用软删除，这一切都不存在了。一个客户可能被软删除，其`deleted_at`标志被设置，但我们现在又回到了可能忘记对其发票执行相同操作的状态。它们的外键仍然有效，因为客户记录从技术上讲仍然存在，但没有相应的检查来确认发票是否也被软删除，因此你可能会发现你的客户被“删除”了，但其发票仍然存在。

过去几年在消费者数据保护方面取得了重大进展，比如在欧洲推出了[GDPR](https://en.wikipedia.org/wiki/General_Data_Protection_Regulation)。因此，一般不建议无限期保留数据，这在软删除行的情况下通常是默认情况。

因此，您最终可能会发现自己编写一个硬删除过程，该过程查看软删除的记录是否超过了某个特定的时间跨度，并从数据库中永久删除它们。

但是，软删除所导致的相同外键现在使得这项工作更加困难，因为一个记录不能被移除而不确保所有依赖项也被移除（`ON DELETE CASCADE`可以自动执行此操作，但是级联的使用相当危险，不建议用于更高保真度的数据）。

幸运的是，在支持CTE（通用表达式）的系统中，您仍然可以执行此操作，例如Postgres，但是您最终会得到一些非常复杂的查询。以下是我最近编写的一个片段，它通过将所有外键满足作为单个操作的一部分来删除所有内容：

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

这个未删减版本长五倍，包含了整整30个单独的表格。这种方法很酷，但是太过复杂了，以至于成为了一个负担。

即使进行了大量测试，这种类型的查询仍然可能会成为可靠性问题，因为如果在将来添加了新的依赖项，但忘记更新查询，那么在一年（或者无论硬删除的地平线是多少）的延迟后，它将突然开始失败。

再一次，软删除在理论上是对意外数据丢失的一种对策。作为最后一个反对它的论点，我要问你，现实中是否有人真的会做未删除的事情。

当我在Heroku工作时，我们使用软删除。

当我在Stripe工作时，我们使用软删除。

在我现在的工作中，我们使用软删除。

据我所知，在这些地方的十多年里，从来没有一次真正使用软删除来恢复删除的东西。

这样做的最大原因是几乎总是，**数据删除也会产生非数据副作用**。可能已经调用了外部系统来存档记录，对象可能已经在blob存储中被移除，或者服务器已经关闭。这个过程不能简单地通过在`deleted_at`上设置`NULL`来撤销 - 所有这些其他操作也需要等效的撤消，而且它们很少存在。

在Heroku，我们遇到过几个情况，一个重要的用户不小心删除了一个应用程序，并希望恢复它。我们有软删除，理论上其他删除的副作用可以被撤销，但我们还是决定不尝试，因为以前从未有人这样做过，而且在紧急情况下尝试做这件事情完全是错误的时间 - 我们几乎肯定会做错事情，让用户处于糟糕的状态。相反，我们通过创建一个新应用程序来继续前进，并帮助他们将已删除的应用程序的环境和数据复制到其中。所以即使在理论上软删除最有用的情况下，我们仍然没有使用它。

尽管我从未见过实践中的撤销操作，软删除并不完全没有用，因为我们偶尔会使用它来引用已删除的数据——通常是一个手动过程，某人想要查看已删除的对象，以协助支持票证或试图消除错误。

虽然我会因为上面列出的缺点而反对传统的软删除模式，但幸运的是有一个折衷方案。

不再将已删除的数据保存在删除它们的同一张表中，可以有一个专门用于存储所有已删除数据的新关系，并且带有灵活的`jsonb`列，以便它可以捕获任何其他表的属性：

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

与`deleted_at`相比，这确实有一个缺点——将列选择到`jsonb`中的过程不容易逆转。虽然这是可能的，但可能涉及构建一次性查询和手动干预。但再次说一遍，也许这没关系——考虑一下你真的会多频繁地尝试撤销数据。

这种技术解决了上述所有问题：

+   查询正常的、非删除数据不再需要在每个地方包含`deleted_at IS NULL`。

+   外键仍然有效。试图删除记录而不获取其依赖项是一个错误。

+   为了符合监管要求，删除旧记录变得非常容易：`DELETE FROM deleted_record WHERE deleted_at < now() - '1 year'::interval`。

虽然删除的数据有点难以获取，但并不多，而且仍然保留在那里，以防有人需要查看它。
