<!--yml

category: 未分类

date: 2024-05-29 13:21:12

-->

# PostgreSQL 数据变更跟踪的终极指南 | Medium

> 来源：[https://exaspark.medium.com/the-ultimate-guide-to-postgresql-data-change-tracking-c3fa88779572](https://exaspark.medium.com/the-ultimate-guide-to-postgresql-data-change-tracking-c3fa88779572)

# **PostgreSQL 数据变更跟踪的终极指南**

PostgreSQL，作为最受欢迎的数据库之一，根据 [DB-Engines Ranking](https://db-engines.com/en/blog_post/106) 被评为2023年的 DBMS 之一，并根据 [HN Hiring Trends](https://www.hntrends.com/2024/january.html?compare=SQL+Server) 在初创公司中使用比其他任何数据库都多。

PostgreSQL 是初创公司中最受欢迎的数据库

自2011年起，SQL 标准已包含与 [时间数据库](https://en.wikipedia.org/wiki/Temporal_database) 相关的特性，允许存储随时间变化的数据，而不仅仅是当前数据状态。然而，关系数据库并未完全遵循这些标准。在 PostgreSQL 的情况下，尽管已提交了一些讨论和补丁，但它并不支持这些特性。

PostgreSQL 有诸如 [periods](https://github.com/xocolatl/periods) 和 [temporal_tables](https://github.com/arkhipov/temporal_tables) 的扩展，支持时间表。不幸的是，像 AWS、Azure 和 GCP 这样的云提供商不允许在托管数据库中运行自定义 C 扩展。

让我们探索在2024年对我们可用的 PostgreSQL 数据变更跟踪的五种替代方法。

# 触发器和审计表

创建带有审计表的 PostgreSQL 触发器

PostgreSQL 允许添加触发器，使用自定义的过程化 SQL 代码执行对行更改的`INSERT`、`UPDATE`和`DELETE`查询。官方的 PostgreSQL wiki 描述了一个通用的 [审计触发器函数](https://wiki.postgresql.org/wiki/Audit_trigger)。让我们快速看一个简化的例子。

首先，在名为`audit`的独立模式中创建一个名为`logged_actions`的表：

```
CREATE schema audit;

CREATE TABLE audit.logged_actions (
  schema_name TEXT NOT NULL,
  table_name TEXT NOT NULL,
  user_name TEXT,
  action_tstamp TIMESTAMP WITH TIME ZONE NOT NULL DEFAULT current_timestamp,
  action TEXT NOT NULL CHECK (action IN ('I','D','U')),
  original_data TEXT,
  new_data TEXT,
  query TEXT
);
```

接下来，创建一个函数来插入审计记录，并在希望跟踪的表上建立触发器，例如`my_table`：

```
CREATE OR REPLACE FUNCTION audit.if_modified_func() RETURNS TRIGGER AS $body$
BEGIN
  IF (TG_OP = 'UPDATE') THEN
    INSERT INTO audit.logged_actions (schema_name,table_name,user_name,action,original_data,new_data,query)
    VALUES (TG_TABLE_SCHEMA::TEXT,TG_TABLE_NAME::TEXT,session_user::TEXT,substring(TG_OP,1,1),ROW(OLD.*),ROW(NEW.*),current_query());
    RETURN NEW;
  elsif (TG_OP = 'DELETE') THEN
    INSERT INTO audit.logged_actions (schema_name,table_name,user_name,action,original_data,query)
    VALUES (TG_TABLE_SCHEMA::TEXT,TG_TABLE_NAME::TEXT,session_user::TEXT,substring(TG_OP,1,1),ROW(OLD.*),current_query());
    RETURN OLD;
  elsif (TG_OP = 'INSERT') THEN
    INSERT INTO audit.logged_actions (schema_name,table_name,user_name,action,new_data,query)
    VALUES (TG_TABLE_SCHEMA::TEXT,TG_TABLE_NAME::TEXT,session_user::TEXT,substring(TG_OP,1,1),ROW(NEW.*),current_query());
    RETURN NEW;
  END IF;
END;
$body$
LANGUAGE plpgsql;

CREATE TRIGGER my_table_if_modified_trigger
AFTER INSERT OR UPDATE OR DELETE ON my_table
FOR EACH ROW EXECUTE PROCEDURE if_modified_func();
```

完成后，对于`my_table`中的行更改将在`audit.logged_actions`中创建记录：

```
INSERT INTO my_table(x,y) VALUES (1, 2);
SELECT * FROM audit.logged_actions;
```

如果您想通过使用 JSONB 列而不是 TEXT 进一步改进此解决方案，忽略某些列的更改，暂停对表的审计等，请查看此 [audit-trigger](https://github.com/2ndQuadrant/audit-trigger) 仓库及其分支中的 SQL 示例。

另一种选择是使用触发器编写的 [temporal_tables](https://github.com/nearform/temporal_tables) 实现。其主要区别在于它在一个记录有效的时间范围内存储记录，而不仅仅是在记录更改时存储一个初始时间戳。这样可以通过选择在特定时间点有效的记录来执行时间旅行查询更加容易。

## 缺点

+   性能。触发器通过在每次`INSERT`、`UPDATE`和`DELETE`操作上同步插入额外的记录增加了性能开销。

+   安全性。任何具有超级用户访问权限的人都可以修改触发器并进行不被注意的数据更改。还建议确保审核表中的记录不能被修改或删除。

+   维护。管理跨多个不断变化的表的复杂触发器可能会变得繁琐。在 SQL 脚本中犯一个小错误可能会破坏查询或数据变更跟踪功能。

# 触发器和 Notify/Listen

具有 PostgreSQL 触发器的 Notify

此方法类似于前一个方法，但不是直接将数据更改写入审核表，而是通过触发器将它们通过发布/订阅机制传递到另一个专门用于读取和存储这些数据更改的系统中：

```
CREATE OR REPLACE FUNCTION if_modified_func() RETURNS TRIGGER AS $body$
BEGIN
  IF (TG_OP = 'UPDATE') THEN
    PEFORM pg_notify('data_changes', json_build_object(
      'schema_name', TG_TABLE_SCHEMA::TEXT,
      'table_name', TG_TABLE_NAME::TEXT,
      'user_name', session_user::TEXT,
      'action', substring(TG_OP,1,1),
      'original_data', jsonb_build(OLD),
      'new_data', jsonb_build(NEW)
    )::TEXT);
    RETURN NEW;
  elsif (TG_OP = 'DELETE') THEN
    PEFORM pg_notify('data_changes', json_build_object(
      'schema_name', TG_TABLE_SCHEMA::TEXT,
      'table_name', TG_TABLE_NAME::TEXT,
      'user_name', session_user::TEXT,
      'action', substring(TG_OP,1,1),
      'original_data', jsonb_build(OLD)
    )::TEXT);
    RETURN OLD;
  elsif (TG_OP = 'INSERT') THEN
    PEFORM pg_notify('data_changes', json_build_object(
      'schema_name', TG_TABLE_SCHEMA::TEXT,
      'table_name', TG_TABLE_NAME::TEXT,
      'user_name', session_user::TEXT,
      'action', substring(TG_OP,1,1),
      'new_data', jsonb_build(NEW)
    )::TEXT);
    RETURN NEW;
  END IF;
END;
$body$
LANGUAGE plpgsql;

CREATE TRIGGER my_table_if_modified_trigger
AFTER INSERT OR UPDATE OR DELETE ON my_table
FOR EACH ROW EXECUTE PROCEDURE if_modified_func();
```

现在可以运行一个作为工作者的单独进程，监听包含数据更改的消息并将其单独存储：

```
LISTEN data_changes;
```

## 缺点

+   “至多一次”传递**。** `listen/notify` 通知没有持久化，这意味着如果监听器断开连接，则可能会错过在其重新连接之前发生的更新。

+   负载大小限制。`listen/notify` 消息默认具有最大 8000 字节的负载大小限制。对于更大的负载，建议将它们存储在数据库审核表中，并仅发送记录的引用。

+   在生产环境中调试与触发器以及`listen/notify`相关的故障排除问题可能会很具有挑战性，因为它们具有异步和分布式的特性。

# 应用程序级跟踪

使用 PostgreSQL 审核表进行应用程序级跟踪

如果您可以控制连接并在 PostgreSQL 数据库中进行数据变更的代码库，则还可以选择以下选项之一：

+   在发出`INSERT`、`UPDATE`和`DELETE`查询时手动记录所有数据变更

+   使用现有的开源库，这些库与流行的 ORM 集成

例如，Ruby 中有 [paper_trail](https://github.com/paper-trail-gem/paper_trail) 与 ActiveRecord 配合使用，Django 中有 [django-simple-history](https://github.com/jazzband/django-simple-history)。它们在高级别上使用回调或中间件向审核表中插入额外的记录。以下是一个在 Ruby 中简化的示例：

```
class User < ApplicationRecord
  after_commit :track_data_changes

  private

  def track_data_changes
    AuditRecord.create!(auditable: self, changes: changes)
  end
end
```

在应用程序级别，还可以实现 [事件溯源](https://martinfowler.com/eaaDev/EventSourcing.html)，将追加日志作为真相的来源。但这是一个单独的、大型和令人兴奋的主题，值得撰写一篇独立的博客文章。

## 缺点

+   可靠性。应用程序级别的数据更改跟踪不如数据库级别的更改跟踪准确。例如，应用程序外部进行的数据更改不会被跟踪，开发人员可能会意外地跳过回调，或者如果更改数据的查询成功但插入审核记录的查询失败，则可能存在数据不一致性。

+   性能。通过回调手动捕获更改并将其插入数据库会导致运行时应用程序和数据库的开销。

+   可扩展性。这些审计表通常存储在同一个数据库中，并且可能会很快变得难以管理，这可能需要分离存储、实施声明性分区和持续归档。

# 变更数据捕获

[变更数据捕获](https://zh.wikipedia.org/wiki/%E5%8F%98%E6%9B%B4%E6%95%B0%E6%8D%95%E8%8E%B7)（CDC）是一种识别和捕获数据库中数据更改的模式，并将这些更改发送到下游系统。最常用于[ETL](https://zh.wikipedia.org/wiki/%E6%8F%90%E5%8F%96%E6%95%B0%E6%8D%AE%E5%BA%93%E8%BD%AC%E6%8D%A2%E5%8A%A0%E8%BD%BD)，将数据发送到数据仓库进行分析。

实施CDC有多种方法。其中一种与我们已经讨论过的不重叠的方法是基于日志的CDC。使用PostgreSQL，可以连接到用于数据耐用性、恢复和复制到其他实例的[Write-Ahead Log](https://www.postgresql.org/docs/current/wal-intro.html)（WAL）。

使用PostgreSQL逻辑复制实现基于日志的CDC

PostgreSQL支持两种类型的复制：物理复制和逻辑复制。后者允许在行级别解码WAL更改并过滤它们，例如通过表名。这正是我们需要使用CDC实现数据更改跟踪的内容。

这里是使用逻辑复制检索数据更改所需的基本步骤：

1\. 在`postgresql.conf`中将`wal_level`设置为`logical`，然后重新启动数据库。

2\. 创建一个类似于“发布/订阅通道”的发布物，用于接收数据更改：

```
CREATE PUBLICATION my_publication FOR ALL TABLES;
```

3\. 创建一个逻辑复制插槽，类似于WAL中的“游标位置”：

```
SELECT * FROM pg_create_logical_replication_slot('my_replication_slot', 'wal2json')
```

4\. 获取最新的未读更改：

```
SELECT * FROM pg_logical_slot_get_changes('my_replication_slot', NULL, NULL)
```

要在PostgreSQL中实现基于日志的CDC，我建议使用现有的开源解决方案。最流行的一个是[Debezium](https://github.com/debezium/debezium)。

## 缺点

+   有限的上下文。PostgreSQL WAL仅包含有关行更改的低级信息，不包括触发更改的SQL查询信息、用户信息或任何特定于应用程序的上下文信息。

+   复杂性。实施CDC会增加大量系统复杂性。这涉及运行一个连接到PostgreSQL作为副本的服务器，消耗数据更改并将其存储在某个地方。

+   调整。在生产环境中运行可能需要更深入地了解PostgreSQL内部，并正确配置系统。例如，定期刷新复制槽的位置以回收WAL磁盘空间。

# 集成变更数据捕获

集成CDC与应用程序上下文

为了解决WAL中存储的数据变更有限信息的挑战，我们可以使用通过直接传递附加上下文到WAL的巧妙方法。

这里是在行更改时传递附加上下文的简单示例：

```
CREATE OR REPLACE FUNCTION if_modified_func() RETURNS TRIGGER AS $body$
BEGIN
  PERFORM pg_logical_emit_message(true, 'my_message', 'ADDITIONAL_CONTEXT');

  IF (TG_OP = 'DELETE') THEN
    RETURN OLD;
  ELSE
    RETURN NEW;
  END IF;
END;
$body$
LANGUAGE plpgsql;

CREATE TRIGGER my_table_if_modified_trigger
AFTER INSERT OR UPDATE OR DELETE ON my_table
FOR EACH ROW EXECUTE PROCEDURE if_modified_func();
```

请注意，`pg_logical_emit_message`函数已添加到PostgreSQL作为插件的内部函数。它允许命名空间和发出消息，这些消息将存储在WAL中。自PostgreSQL v14以来，可以使用标准的逻辑解码插件`pgoutput`来读取这些消息。

有一个名为[Bemi](https://github.com/BemiHQ/bemi)的开源项目，它不仅允许跟踪低级别的数据变更，还可以读取任何自定义上下文与CDC并将所有内容粘合在一起。完整的声明，我是核心贡献者之一。

例如，它可以与流行的ORM和适配器集成，以传递特定于应用程序的上下文和所有数据更改：

```
import { setContext } from "@bemi-db/prisma";
import express, { Request } from "express";

const app = express();

app.use(
  // Customizable context
  setContext((req: Request) => ({
    userId: req.user?.id,
    endpoint: req.url,
    params: req.body,
  }))
);
```

## 缺点

+   与实施CDC相关的复杂性和调整。

如果您需要一个即用型的云解决方案，可以在几分钟内集成并连接到PostgreSQL，请查看[bemi.io](https://bemi.io/)。

# 结论

PostgreSQL数据变更跟踪方法比较

1.  如果您需要基本的数据变更跟踪，**带有审计表的触发器**是一个很好的初始解决方案。

1.  **带有监听/通知的触发器**是在开发环境中进行简单测试的一个不错选择。

1.  如果您更重视特定于应用程序的上下文（关于用户信息、API端点等）而不是可靠性，可以使用**应用程序级跟踪**。

1.  **变更数据捕获（Change Data Capture）**是一个不错的选择，如果您优先考虑作为统一解决方案的可靠性和可扩展性，例如，可以在许多数据库中重复使用。

1.  最后，**集成变更数据捕获（Integrated Change Data Capture）**是您需要的最佳选择，如果您需要一个强大的数据变更跟踪系统，还可以集成到您的应用程序中。如果您需要云管理解决方案，请选择[bemi.io](https://bemi.io/)。
