<!--yml

category: 未分类

date: 2024-05-27 14:41:05

-->

# 在Postgres中实施系统版本表

> 来源：[https://hypirion.com/musings/implementing-system-versioned-tables-in-postgres](https://hypirion.com/musings/implementing-system-versioned-tables-in-postgres)

我喜欢Postgres，但也有些功能我真希望它们能实现。例如，[SQL:2011](https://en.wikipedia.org/wiki/SQL:2011) 规范添加了对系统版本表的支持。不幸的是，Postgres 和 SQLite 基本上是唯一还不支持它的 SQL 数据库。尽管在 [postgreqsl-hackers 邮件列表](https://www.postgresql.org/message-id/flat/CALAY4q-cXCD0r4OybD%3Dw7Hr7F026ZUY6%3DLMsVPUe6yw_PJpTKQ%40mail.gmail.com) 上已经广泛讨论了这个问题，但似乎在整个 2023 年里，讨论和实施工作都没有进展。

现在，市面上有一些实现版本控制的扩展，[`temporal_tables`](https://clarkdave.net/2015/02/historical-records-with-postgresql-and-temporal-tables-and-sql-2011/) 可能是最流行的，但我认为没有一个在例如 Azure 或 AWS 上的托管 Postgres 实例上得到支持。这意味着，如果我们想要系统版本表，我们只能自己动手实现。

在我们开始实际实施之前，我不打算花费段落来解释为什么我想要临时表，但我想指出，我主要想使用这些表来存储用户数据，这些数据由用户在网页上点击/输入时生成。它们也可以用于其他用途，但如果您有时间序列、想要进行事件溯源或在线分析处理，您应该选择一些适合该任务的技术，而不是随意使用临时表。

如果你只关心最终结果，请随意访问我的GitHub仓库 [time-travelling-todo-lists-in-postgres](https://github.com/hypirion/time-travelling-todo-lists-in-postgres)。这是一个具有时间旅行功能的待办事项应用程序，使用了这里描述的实现方法。在那里，你将获得如何自行使用它的信息，如何查询过去以及一些需要注意的问题和常见陷阱。

## 实施

我决定的实施方案包括为我想要版本控制的每个表创建两个表：一个快照表用于当前状态，一个历史表。

<svg xmlns="http://www.w3.org/2000/svg" viewBox="0 0 116.798 35.169"><text xml:space="preserve" x="30.997" y="45.291" transform="translate(-19.358 -27.08)"><tspan x="30.997" y="45.291">快照</tspan><tspan x="30.997" y="50.582">表</tspan></text><text xml:space="preserve" x="103.227" y="45.729" transform="translate(-19.358 -27.08)"><tspan x="103.227" y="45.729">历史</tspan><tspan x="103.227" y="51.021">表</tspan></text><text xml:space="preserve" x="69.26" y="33.634" transform="translate(-19.358 -27.08)"><tspan x="69.26" y="33.634">插入</tspan><tspan x="69.26" y="37.603">更新</tspan><tspan x="69.26" y="41.572">删除</tspan></text></svg>

快照表是用于当前世界状态的表，通常是您将要使用的表格：它与您熟悉的任何其他可变的就地表格完全相同。 历史表是在您想要通过历史查询时将要使用的表格。

首先，我们定义表格：

```
CREATE TABLE mytable (
  mytable_id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  data TEXT NOT NULL
);

CREATE TABLE mytable_history (
  -- copy these fields, always keep them first
  history_id UUID PRIMARY KEY,
  systime TSTZRANGE NOT NULL CHECK (NOT ISEMPTY(systime)),

  -- table fields, in the exact same order as in mytable
  mytable_id UUID NOT NULL,
  data TEXT NOT NULL
);

ALTER TABLE mytable_history
  ADD CONSTRAINT mytable_history_overlapping_excl
  EXCLUDE USING GIST (mytable_id WITH =, systime WITH &&); 
```

历史表包含快照表中完全相同的字段，但表中的前两列是`history_id`字段和记录活动的系统时间（`systime`）。 `lower(systime)`是记录此版本的记录最初存储的时间，`upper(systime)`是此记录不再有效的时间。 当`upper(systime)`是无穷大时，记录尚未删除。 显然，时间间隔不能为空，因为这将不匹配任何时间间隔。

接下来，我们有一个GiST索引，它防止具有相同主键的时间间隔重叠，并且还加速了对历史的查询。 根据您希望在其上执行的查询类型，您可能需要在历史表上添加更多索引。

然后我们为插入、更新和删除设置触发器：

```
--
-- insert:
--

CREATE OR REPLACE FUNCTION copy_mytable_inserts_into_history()
          RETURNS TRIGGER AS $$
  INSERT INTO mytable_history
    SELECT gen_random_uuid(), tstzrange(NOW(), NULL), NEW.*;
  RETURN NEW;
$$ LANGUAGE plpgsql;

CREATE TRIGGER mytable_history_insert_trigger
AFTER INSERT ON mytable
    FOR EACH ROW
    EXECUTE PROCEDURE copy_mytable_inserts_into_history();

--
-- update:
--

CREATE OR REPLACE FUNCTION copy_mytable_updates_into_history()
          RETURNS TRIGGER AS $$
  -- ignore changes inside the same tx
  DELETE FROM mytable_history
    WHERE mytable_id = NEW.mytable_id
      AND lower(systime) = NOW()
      AND upper_inf(systime);
  -- close current row
  -- (if any, may be deleted by previous line)
  UPDATE mytable_history
    SET systime = tstzrange(lower(systime), NOW())
    WHERE mytable_id = NEW.mytable_id
      AND systime @> NOW();
  -- insert new row
  INSERT INTO mytable_history
    SELECT gen_random_uuid(), tstzrange(NOW(), NULL), NEW.*;
  RETURN NEW;
$$ LANGUAGE plpgsql;

CREATE TRIGGER mytable_history_update_trigger
AFTER UPDATE ON mytable
    FOR EACH ROW
    WHEN (OLD.* IS DISTINCT FROM NEW.*)
    -- ^ to avoid updates on "noop calls", as best as possible
    EXECUTE PROCEDURE copy_mytable_updates_into_history();

--
-- delete:
--

CREATE OR REPLACE FUNCTION copy_mytable_deletes_into_history()
          RETURNS TRIGGER AS $$
  -- close current row
  -- note: updates and then deletes for same id
  -- in same tx will fail
  UPDATE mytable_history
    SET systime = tstzrange(lower(systime), NOW())
    WHERE mytable_id = OLD.mytable_id
      AND systime @> NOW();
  RETURN OLD;
$$ LANGUAGE plpgsql;

CREATE TRIGGER mytable_history_delete_trigger
AFTER DELETE ON mytable
    FOR EACH ROW
    EXECUTE PROCEDURE copy_mytable_deletes_into_history(); 
```

这是…… 长话短说，让我们来看看这些触发器中所有三种常见技巧的一些共同的把戏，然后再过一下我为更新和删除做出的设计决策。 最后我们来缩短它。

### 列顺序很重要

每当我们将新数据插入历史表时，它将始终以此形式：

```
INSERT INTO mytable_history
  SELECT gen_random_uuid(), tstzrange(NOW(), NULL), NEW.*; 
```

Postgres中我们在这里滥用了两部分。 如果我们查看[INSERT的文档](https://www.postgresql.org/docs/16/sql-insert.html)，我们将看到以下段落：

> 目标列名可以按任何顺序列出。 如果根本没有列名列表，则默认为表格中所有列按其声明的顺序； 或者如果`VALUES`子句或`query`提供的列仅有***N***列，则与显式或隐式的列列表从左到右关联。

另一个是`NEW.*`以完全相同的顺序扩展。 在我们的情况下，`NEW.*`将扩展为`NEW.mytable_id, NEW.data`，这意味着它与`mytable_history`表的行完美匹配：

```
INSERT INTO mytable_history
  SELECT gen_random_uuid(), tstzrange(NOW(), NULL),
         NEW.mytable_id, NEW.data; 
```

这非常聪明，但也很粗糙。 要使其工作，您必须做到两点：

+   在`history_id`和`systime`列后，列顺序必须完全相同。

+   每当您添加新列时，必须以相同的顺序将其添加到两个表中。

如果您不这样做，您将最终在历史表中得到损坏的插入。 在最好的情况下，它们会给您一个错误，但它们也可能悄无声息地破坏您的数据。

我不喜欢那样，但我不这样做有一个原因

```
INSERT INTO mytable_history
  (history_id, systime, mytable_id, data)
SELECT gen_random_uuid(), tstzrange(NOW(), NULL),
       NEW.mytable_id, NEW.data; 
```

这是因为当您想要从表中删除、重命名或添加新列时，它会**极大地增加工作量**。 通过省略目标列名的列表并扩展`NEW.*`，只要我决定同时对两个表进行操作，查询就会自动工作。

### 在系统时间上

如果你之前没有使用过 PostgreSQL 的范围类型，范围查询可能看起来有点神秘。PostgreSQL 文档对[范围类型](https://www.postgresql.org/docs/16/rangetypes.html)和[范围函数](https://www.postgresql.org/docs/16/functions-range.html)有出色的描述，这应该涵盖与它们相关的一切。虽然我没怎么用它，所以我猜你也不想详细阅读，我只是简单列出了不同部分的内容。

这个表达式

创建了 timestamptz 范围 `[a, b)`：从 *a* 开始，直到但不包括 *b*。如果 *a* 或 *b* 是 `NULL`，那么它们分别位于过去或未来的无限远处。

`lower(x)` 返回范围的下限，而 `upper_inf(x)` 如果上限是无限的则返回真。

最后，`systime @> NOW()` 检查 `systime` 是否包含 `NOW()` – 你可以把它看作是 `a <= NOW() AND NOW() < b`。

使用范围的原因并非主要是因为我想使用这些函数，而是因为 GiST 索引确保没有重叠，并且让我不那么担心我的查询会因为偶然而变得超级缓慢。

### 插入

插入操作可以说已经描述得比较完整了：我们将行插入到历史表中，表明它从事务开始时间到永远都是有效的。

### 更新

更新触发器如下所示：

```
-- ignore changes inside the same tx
DELETE FROM mytable_history
  WHERE mytable_id = NEW.mytable_id
    AND lower(systime) = NOW()
    AND upper_inf(systime);
-- close current row
UPDATE mytable_history
  SET systime = tstzrange(lower(systime), NOW())
  WHERE mytable_id = NEW.mytable_id
    AND systime @> NOW();
-- insert new row
INSERT INTO mytable_history
  SELECT gen_random_uuid(), tstzrange(NOW(), null), NEW.*; 
```

设计成在同一个事务中进行多次更新只会生成一行历史记录。这是有意的，因为否则会更难实现排序。

如果你需要在同一事务中有多个版本，你需要用闭合间隔(`[]`)替换半开间隔（默认为`[)`）。否则，你会得到空间隔，它们不与任何时间戳相关联。但现在你必须处理同一时间戳上相同状态的情况，因为多条记录可以得到`[NOW(), NOW()]`间隔。如果在这种情况下顺序很重要，你必须添加另一个排序 ID，以便按插入顺序获取行。

我不需要这些，它太复杂了，而且我不认为在同一时间点有多个状态是有意义的。如果你需要这样做，你可能更感兴趣的是事件表或其他东西。

### 删除

```
-- close current row
-- note: updates and then deletes for same id
-- in same tx will fail
UPDATE mytable_history
  SET systime = tstzrange(lower(systime), NOW())
  WHERE mytable_id = OLD.mytable_id
    AND systime @> NOW(); 
```

当涉及原子性时，删除操作有点不同寻常。如果你在同一个事务中先更新再删除某些内容，你应该怎么做？而且，为什么首先要这样做呢？

“更新和删除”在你想要记录谁删除了对象时似乎非常方便，此时删除操作将如下所示：

```
UPDATE mytable
  SET deleted_by = ${deleted_by}
WHERE mytable_id = ${mytable_id};

DELETE FROM mytable
WHERE mytable_id = ${mytable_id}; 
```

… 但会留下一个让你很烦恼的记录：那个“删除”记录，并不是历史的一部分，只包含有关删除本身的一些信息。

我认为即使你没有立即需要更改日志/审计日志，也最好为此做出一点努力：将删除信息存储在一个表中。你可以为所有表创建一个表，或者根据你的懒惰程度仅创建一个表：

```
CREATE TABLE delete_log (
  delete_log_id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  table_name TEXT NOT NULL,
  resource_id UUID NOT NULL,
  deleted_at TIMESTAMPTZ NOT NULL DEFAULT NOW(),
  deleted_by UUID NOT NULL REFERENCES users(user_id)
);

-- then delete as follows

INSERT INTO delete_log (table_name, resource_id, deleted_by)
VALUES ('mytable', ${mytable_id}, ${deleted_by});

DELETE FROM mytable
WHERE mytable_id = ${mytable_id}; 
```

我很难（或者可能根本做不到）轻松地制作一个触发器，因为`deleted_by`是我们必须以某种方式传递到数据库中的信息。

## 令人讨厌的冲突

虽然触发器本身不会影响你可以或不能做的事情，但GiST索引将使大多数同时更新相同项目的尝试失败。

```
ALTER TABLE mytable_history
  ADD CONSTRAINT mytable_history_overlapping_excl
  EXCLUDE USING GIST (mytable_id WITH =, systime WITH &&); 
```

即使你使用`SELECT .. FOR UPDATE`也会失败。当两个事务A和B同时到达时，如果拥有最近的`NOW()`时间戳的事务首先获得锁定，第二个事务将失败，因为`systime`最终为空时间间隔。

对我来说，这很烦人，但至少不是什么大问题。我通常需要历史记录的表通常不需要在相同的ID上进行写并发。如果确实需要，添加一个重试循环以处理写入就不难……当然，这会增加你需要注意的陷阱。

尝试使用`CLOCK_TIMESTAMP()`而不是`NOW()`来修复这个问题只会让情况变得更糟，在我看来。如果你希望同时插入多个项目，通常它们的时间戳都不会相同。这会导致一些对历史表的查询在技术上是错误的：

想象一下，你有一个列表和列表元素表，用户创建了一个有10个元素的新列表。使用`CLOCK_TIMESTAMP()`后，你现在有了11个不同的时间戳：一个用于列表根，每个列表元素有10个。如果你想知道列表在最初创建时的样子，你首先需要获取列表根。然后，你必须获取所有在列表中的元素，比如说，在创建后1秒钟内，以确保所有列表元素也被返回。但是，任何试图在过去某个任意时间重建状态的尝试可能会导致部分/损坏的结果。因此，我强烈建议避免这样做，而是在确实需要时使用其他方法来持久化历史记录。

## 替代方案

如果必须使用Postgres，基于触发器的模型对我来说似乎是最好的选择，但你可能想考虑一些替代方案。

### 你需要历史记录吗？

这里增加了复杂性，同时由于重叠的时间戳增加了事务失败的可能性，这是你无法真正防止的。这引出了一个问题：你真的需要历史记录吗？

我“有争议的”观点是，在今天和时代，我觉得真正的问题实际上是你是否可以在性能上承受得起，而不是它是否过于复杂。这种持久化历史的具体实现可能并不快速，但概念上很简单：每当你对表执行操作时，该操作都会存储在历史表中。原始表故意与你所熟悉的完全相同。如果你真的不喜欢潜在的GiST冲突，你可以放宽约束条件，以获得与你所熟悉的完全相同的行为。在这种情况下，唯一的区别是插入/删除/更新的性能。

是的，有许多情况下明显保留历史是荒唐的，但我也认为许多人认为存储历史的成本比实际要高得多^(如果你不确定，试一试，看看效果如何。因为你的原始表不会受影响，所以回滚并不困难。)

对我来说，有两个原因我认为你应该考虑它，即使你没有立即计划将其暴露给你的用户：

首先，将会有一个时间点，特定客户需要将他们的数据回滚到较早的版本。这可能是由于意外删除、恶意行为，甚至是损坏了某些客户数据的糟糕部署。虽然这远远不能替代备份，但它更快速、更容易执行。

```
BEGIN;

DELETE FROM mytable
WHERE company_id = ${company_id};

INSERT INTO mytable (your, fields, here)
SELECT h.your, h.field, h.here
FROM mytable_history h
WHERE h.company_id = ${company_id}
  AND h.systime @> ${backup_time};

COMMIT; 
```

当然，在实践中，这可能会比那更加复杂，这取决于数据模型和你想要多细粒度的操作。但这比设置备份服务器、与外部数据封装器进行桥接，最后以完全相同的方式执行插入要少努力，之后清理备份服务器。

还有其他情况，比如调试用户之一遇到的瞬时问题。我可以继续，但感觉像是在重复为[Datomic](https://www.datomic.com/)做销售演示。

第二个原因与说“我们不需要它”的直觉反应有关：如果你没有即时可用的时间旅行能力，你就无法探索和玩耍。这反过来意味着在为客户开发新功能时，你不会真正考虑它们。冒险听起来像是一个技术极客：我认为这抑制了创新。你*真的*认为将来你的系统中没有它的价值吗？

是的，是的，这听起来很像未来的证明。明确地说，YAGNI并没有错，但我只认为它在以下情况下才适用：

+   有显著的开销（无论是在实施复杂性还是系统性能方面）

+   可以事后实施的事情（即没有数据丢失）

并且系统版本化的表也不是。

…嗯，呃，让我在这里澄清一下。我展示给你的当前实现在你创建新表时需要大量工作。这是关于我的下一个观点的良好过渡：

### 使用适合任务的数据库。

因为Postgres不支持而自己编写所有这些长触发器有点愚蠢。如果你想要一个开源替代方案，也许MariaDB并不算太糟？它支持[SQL:2011](https://mariadb.com/kb/en/system-versioned-tables/)，甚至支持如果你想进一步的bitemporality。

当然，如果我不提到[Datomic](https://www.datomic.com/)，那就不太合适了。这是一个很棒的数据库，尽管不是开源的，语言支持相当有限（官方只支持Java/Clojure）。对于那些在微软领域的人，[SQL Server](https://learn.microsoft.com/en-us/sql/relational-databases/tables/temporal-tables?view=sql-server-ver16)据我所知对SQL:2011有很好的支持。

正如前面提到的，这是针对由某人在网页上点击或输入生成的用户数据。但对于物联网数据，你可能希望使用比时间表更好的工具，如果规模足够大，你可能希望使用OLAP数据库进行OLAP工作。

## 粗制滥造或离奇恐怖

我现在被Postgres卡住了，一方面是因为这是我已经有的东西，另一方面是因为我大量使用PostGIS。迁移根本不是一个选择，所以我必须利用我所拥有的。

我喜欢触发器解决方案的原因在于，除了列扩展黑客之外，理解起来并不是非常困难。问题在于它非常冗长，在过去由于几次复制粘贴错误，我宁愿每个表只有一个触发器。

这是可能的……尽管看起来并不美观。在我们将历史表和ID字段转换为输入参数后，更新触发器的样子如下：

```
CREATE FUNCTION copy_updates_into_history() RETURNS TRIGGER AS $$
DECLARE
  history_table TEXT := quote_ident(tg_argv[0]);
  id_field TEXT := quote_ident(tg_argv[1]);
BEGIN
  -- ignore changes inside the same tx
  EXECUTE 'DELETE FROM ' || history_table ||
    ' WHERE ' || id_field || ' = $1.' || id_field ||
    ' AND lower(systime) = NOW()' ||
    ' AND upper_inf(systime)' USING NEW;
  -- close current row
  -- (if any, may be deleted by previous line)
  EXECUTE 'UPDATE ' || history_table ||
    ' SET systime = tstzrange(lower(systime), NOW())'
    ' WHERE ' || id_field || ' = $1.' || id_field ||
    ' AND systime @> NOW()' USING NEW;
  -- insert new row
  EXECUTE 'INSERT INTO ' || history_table ||
    ' SELECT gen_random_uuid(), tstzrange(NOW(), null), $1.*'
    USING NEW;
  RETURN NEW;
END;
$$ LANGUAGE plpgsql; 
```

（插入和更新触发器相似，所以我这里不再赘述。）

为了分离历史表和ID字段，我们必须使用[`EXECUTE`](https://www.postgresql.org/docs/16/plpgsql-statements.html#PLPGSQL-STATEMENTS-EXECUTING-DYN)，这使得整个过程看起来更加黑客。它看起来很丑陋，如果你不熟悉`EXECUTE`或前面提到的列扩展魔法，可能会感觉完全离奇。不过，我认为这比第一次尝试的触发器实现要好得多。而这最好通过实际使用来展示：

```
CREATE TRIGGER mytable_history_update_trigger
AFTER UPDATE ON mytable
    FOR EACH ROW
    WHEN (OLD.* IS DISTINCT FROM NEW.*)
    -- ^ to avoid updates on "noop calls", as best as possible
    EXECUTE PROCEDURE
      copy_updates_into_history('mytable_history', 'mytable_id'); 
```

如果你想保留多个表的历史记录，使用这种方式要少得多！

虽然查询无法提前进行类型检查，但原始触发器也不会。*我们必须运行它们来验证它们是否引用了错误的表或ID*。如果是这样，那么我们应该选择减少粗制滥造搜索替换机会的替代方案。

而这个新实现正是如此！这个触发器只在两个位置有历史表和ID字段 - 输入参数中，而原始的触发器实现则在7个不同的位置散布着它们。另外，面对现实吧：对每个我们想要系统版本管理的表格复制粘贴一些大触发器真的不太合适。

## 概要

完整的实现在[Postgres中的时间旅行待办事项列表](https://github.com/hypirion/time-travelling-todo-lists-in-postgres)仓库中，而触发器则在迁移文件[`migrations/001_history_triggers.up.sql`](https://github.com/hypirion/time-travelling-todo-lists-in-postgres/blob/main/migrations/001_history_triggers.up.sql)中。

正如我所提到的，这并不完美：你会遇到一些与表格修改相关的大坑，它增加了并发更新中断的风险，而其他数据库应该有比我的三个触发器更好的实现。它可能也不适合大型数据库。

但是如果你被迫使用Postgres，我认为这也可以。然而，我认为你应该详细了解实现的工作原理，因为有几种方法会让你自讨苦吃。而且如果你已经走到了这一步，希望你真的会注意！
