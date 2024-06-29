<!--yml

category: 未分类

date: 2024-05-27 14:56:04

-->

# My Notes on GitLab Postgres Schema Design – Shekhar Gulati

> 来源：[https://shekhargulati.com/2022/07/08/my-notes-on-gitlabs-postgres-schema-design/](https://shekhargulati.com/2022/07/08/my-notes-on-gitlabs-postgres-schema-design/)

我花了一些时间研究Gitlab的Postgres模式。GitLab是Github的一个替代品。你可以自己托管GitLab，因为它是一个开源的DevOps平台。

我研究一个像Gitlab这样的大项目的模式的动机是为了将其与我设计的模式进行比较，并从他们的模式定义中学习一些最佳实践。我可以肯定地说我学到了很多。

> 我知道最佳实践有时是依赖于上下文的，所以不要盲目应用它们。

Gitlab的模式文件 `structure.sql` [1] 超过34000行代码。Gitlab是一个单片式Ruby on Rails应用程序。管理模式迁移的流行方式是使用 `schema.rb` 文件。Gitlab团队决定采用 `structure.sql` 的原因在他们的问题追踪器中的一个问题 [2] 中有提到。

> 现在阻止我们使用这些功能的是使用 `schema.rb`。这只能包含标准迁移（使用Rails DSL），旨在使模式文件与数据库系统中立并从特定SQL中抽象出来。这反过来意味着我们不能使用反映在模式中的扩展PostgreSQL功能。一些例子包括触发器、Postgres分区、物化视图和许多其他优秀功能。
> 
> 为了利用这些功能，我们应考虑使用纯SQL模式文件 (`structure.sql`) 而不是ruby/rails标准模式 `schema.rb`。
> 
> 更改将涉及切换 `config.active_record.schema_format = :sql` 并在SQL中重新生成模式。可能还需要调整一些构建步骤。

现在，让我们来看看我从Gitlab Postgres模式中学到的东西。

以下是一些人们在这篇文章上的推文。如果你觉得这篇文章有用，请分享并标记我 [@shekhargulati](https://twitter.com/shekhargulati)

## 1\. 为表选择合适的主键类型

在我的工作中，我犯过标准化主键类型的错误。这意味着无论表的结构、访问模式和增长率如何，都要标准化为 `bigint` 或 `uuid`，所以所有表都将具有相同的类型。

当你的数据库很小时，这对性能没有明显影响，但随着增长，主键对存储空间、写入速度和读取速度有明显影响。因此，我们应该在选择表的合适主键类型时进行深思熟虑。

正如我在早期文章[3]中讨论的，当你使用Postgres本机UUID v4类型而不是bigserial时，表大小增长了25%，插入速率降低了25%。这是一个很大的差异。我还与ULID进行了比较，但它的性能也很差。其中一个原因可能是ULID的实现。

Given this context I was interested to learn how Gitlab chooses primary key types.

Out of the 573 tables, 380 tables have bigserial primary key type, 170 have serial4 primary key type, and remaining 23 had composite primary keys.They had no table that used uuid v4 primary key or any other esoteric key type like ULID.

| Name | Description | Range | Text |
| --- | --- | --- | --- |
| `serial` | 4 bytes | 1 to 2147483647 | ~2.1 billion |
| `bigserial` | 8 bytes | 1 to 9223372036854775807 | ~9.2 quintillion |

> 1 quintillion is equal to 1000000000 billions

The decision to choose serial or bigserial is dependent on the number of records in that table.

Tables like `application_settings`, `badges`, `chat_teams`, `notification_settings`, `project_settings` use serial type. For some tables like `issues`, `web_hooks`, `merge_requests`, `projects` I was surprised to see that they had used the `serial` type.

The serial type might work for self-hosted community or enterprise versions but for Gitlab.com SaaS service this can cause issues. For example, Github had 128 million public repositories in 2020\. Even with 20 issues per repository it will cross the serial range. Also changing the type of the table is expensive. The table has to be rewritten, and you will have to wait. This will also be a problem if you have to shard the table.

I performed a quick experiment that showed that for my table with two columns and 10million records it takes 11 seconds to change the data type from integer to bigint.

```
create table exp_bs(id serial primary key, n bigint not null);

```

Insert 10million records

```
insert into exp_bs(n) select g.n from generate_series(1,10000000) as g(n);

```

Change column type from integer to bigint.

```
alter table exp_bs alter column id TYPE bigint;

```

```
ALTER TABLE
Time: 10845.062 ms (00:10.845)

```

You will also have to alter the sequence to change its type as well. This operation is quick.

```
alter sequence exp_bs_id_seq as bigint;

```

This finished in 4ms

```
ALTER SEQUENCE
Time: 4.505 ms

```

All the bigserial sequences start from 1 and go till the max value of bigint.

```
CREATE SEQUENCE audit_events_id_seq
    START WITH 1
    INCREMENT BY 1
    NO MINVALUE
    NO MAXVALUE
    CACHE 1;

```

## 2\. Use of internal and external ids

It is generally a good practice to not expose your primary keys to the external world. This is especially important when you use sequential auto-incrementing identifiers with type integer or bigint since they are guessable.

So, I was curious to know what happens when you create a Gitlab issue. Do we expose the primary key id to the external user or do we use some other id? If you expose the `issues` table primary key id then when you create an issue in your project it will not start with 1 and you can easily guess how many issues exist in the GitLab. This is both unsafe and poor user experience.

To avoid exposing your primary keys to the end user the common solution is use two ids. The first is your primary key id which remains internal to the system and never exposed to any public context. The second id is what we share with the external world. In my past experience I have used UUID v4 as the external id. As we discussed in the previous point there is a storage cost involved with using UUID.

GitLab 在需要与外部世界共享 ID 的表格中还使用了内部和外部 ID。像 `issues`、`ci_pipelines`、`deployments`、`epics` 等表格中有两个 ID — `id` 和 `iid`。以下是问题模式的一部分。如下所示，`iid` 具有整数数据类型。

```
CREATE TABLE issues (
    id integer NOT NULL,
    title character varying,
      project_id integer,
    iid integer,
    // rest of the columns removed
)

```

如您所见，存在 `id` 和 `iid` 列。`iid` 列的值与最终用户共享。使用 `project_id` 和 `iid` 唯一标识问题。这是因为可能存在具有相同 `iid` 的多个问题。为了更清楚地说明，如果您创建两个项目并在每个存储库中创建一个问题，则它们都需要如下示例所示具有可见的 id 1。`sg` 和 `sg2` 项目都以问题 id 1 开始。这是通过使用 `iid` 实现的。

```
https://gitlab.com/shekhargulati123/sg/-/issues/1
https://gitlab.com/shekhargulati123/sg2/-/issues/1

```

他们在 `project_id` 和 `iid` 上有一个唯一索引，以快速高效地获取问题。

```
CREATE UNIQUE INDEX index_issues_on_project_id_and_iid ON public.issues USING btree (project_id, iid);

```

## 3\. 使用带有检查约束的 `text` 字符类型

Postgres 在其文档[5]中描述了三种字符类型。

| 名称 | 描述 |
| --- | --- |
| `character varying(n)`, `varchar(n)` | 可变长度带限制 |
| `character(n)`, `char(n)` | 固定长度，空格填充 |
| `text` | 可变长度，无限制 |

我主要使用 `character varying(n)` 或 `varchar(n)` 来存储字符串值。Gitlab 的模式同时使用 `character varying(n)` 和 `text`，但更多地使用 `text` 类型。下面是一个示例表格。

```
CREATE TABLE audit_events (
    id bigint NOT NULL,
    author_id integer NOT NULL,
    entity_id integer NOT NULL,
    entity_type character varying NOT NULL,
    details text,
    ip_address inet,
    author_name text,
    entity_path text,
    target_details text,
    created_at timestamp without time zone NOT NULL,
    target_type text,
    target_id bigint,
    CONSTRAINT check_492aaa021d CHECK ((char_length(entity_path) <= 5500)),
    CONSTRAINT check_83ff8406e2 CHECK ((char_length(author_name) <= 255)),
    CONSTRAINT check_97a8c868e7 CHECK ((char_length(target_type) <= 255)),
    CONSTRAINT check_d493ec90b5 CHECK ((char_length(target_details) <= 5500))
)
PARTITION BY RANGE (created_at);

```

您可以看到，除了 `entity_type` 以外，所有其他列都是 `text` 类型。他们使用 `CHECK` 来定义长度约束。

如多个网络帖子[6,7]中所述，两种类型之间的性能差异不大。它们在底层都使用 `varlena` 类型。

`varchar(n)` 的问题在于，如果 n 变得更为严格，则将需要独占锁定。这可能会因表的大小而导致性能问题。

另一方面，带有 `CHECK` 约束的 `text` 列则不具有此问题。但在写入时会稍微耗时。

让我们做一个快速实验来证明这一点。我们将从创建一个简单的表格开始

```
create table cv_exp (id bigint primary key, s varchar(200) default gen_random_uuid() not null);
create index sidx on cv_exp (s);

```

插入 1000 万条记录

```
insert into cv_exp(id) select g.n from generate_series(1,10000000) as g(n);

```

如果我们将 s 的长度从 200 增加到 300，则是瞬间的。

```
alter table cv_exp alter column s type varchar(300);

```

```
ALTER TABLE
Time: 37.460 ms

```

但是，如果我们将 `s` 的长度从 300 减少到 100，则确实需要相当长的时间。

```
alter table cv_exp alter column s type varchar(100);

```

```
ALTER TABLE
Time: 35886.638 ms (00:35.887)

```

如您所见，这花了 36 秒。

让我们对文本列做同样的操作。

```
create table text_exp (id bigint primary key, 
s text default gen_random_uuid() not null,
CONSTRAINT check_15e644d856 CHECK ((char_length(s) <= 200)));

```

插入 1000 万条记录

```
insert into text_exp(id) select g.n from generate_series(1,10000000) as g(n);

```

在 Postgres 中没有 alter 约束。您必须先删除约束，然后再添加新约束。

```
 alter table text_exp drop constraint check_15e644d856;

```

现在，再次添加

```
alter table text_exp add constraint check_15e644d856 CHECK ((char_length(s) <= 100));

```

```
ALTER TABLE
Time: 1870.250 ms (00:01.870)

```

因此，如您所见，与 `character varying` 或 `varchar(n)` 相比，带有 `CHECK` 约束的 `text` 类型使您在需要长度检查时更容易演变模式。

我还注意到，它们在不需要长度检查的地方使用了 `character varying`，如下所示。

```
CREATE TABLE project_custom_attributes (
    id integer NOT NULL,
    created_at timestamp with time zone NOT NULL,
    updated_at timestamp with time zone NOT NULL,
    project_id integer NOT NULL,
    key character varying NOT NULL,
    value character varying NOT NULL
);

```

## 4\. 命名约定

命名遵循以下约定。

+   所有表都使用复数形式。例如 `issues`、`projects`、`audit_events`、`abuse_reports`、`approvers` 等。

+   表使用模块名称前缀来提供命名空间。例如，所有属于合并请求功能的表都以`merge_request`前缀开头，如下面的列表所示。

    +   merge_request_assignees

    +   merge_request_blocks

    +   merge_request_cleanup_schedules

    +   merge_request_context_commit_diff_files

    +   merge_request_context_commits

    +   等等..

+   表和列的名称遵循蛇形命名约定。下划线用于组合两个或多个单词。例如，`title`，`created_at`，`is_active`。

+   表达布尔值的列根据其目的遵循以下三种命名约定之一

    +   功能开关。例如 `create_issue`，`send_email`，`packages_enabled`，`merge_requests_rebase_enabled` 等。

    +   实体状态：例如 `deployed`，`onboarding_complete`，`archived`，`hidden` 等。

    +   限定词 - 它们以 `is_xxx` 或 `has_xxx` 开头。例如，`is_active`，`is_sample`，`has_confluence` 等。我认为这些可以用上述两种方式来表达。

+   索引遵循约定 `index_#{table_name}_on_#{column_1}_and_#{column_2}_#{condition}`。例如，`index_services_on_type_and_id_and_template_when_active`，`index_projects_on_id_service_desk_enabled`。

## 5\. 带时区和无时区的时间戳

GitLab 同时使用 `timestamp with timezone` 和 `timestamp without timezone`。

我的理解是，当系统执行操作时使用数据类型 `timestamp without timezone`，而用户操作时使用数据类型 `timestamp with time zone`。例如，在下面显示的 SQL 中，`created_at` 和 `updated_at` 使用的是 `timestamp without timezone`，而 `closed_at` 使用的是 `timestamp with timezone`。

```
CREATE TABLE issues (
    id integer NOT NULL,
    title character varying,

    created_at timestamp without time zone,
    updated_at timestamp without time zone,

    closed_at timestamp with time zone,
    closed_by_id integer,
);

```

另一个例子是 `merge_request_metrics`，其中 `latest_closed_at`，`first_comment_at`，`first_commit_at` 和 `last_commit_at` 使用 `timestamp with time zone`，而 `latest_build_started_at`，`latest_build_finished_at` 和 `merge_at` 则使用 `timestamp without timezone`。你可能会想为什么 `merge_at` 不使用时区。我认为这是因为系统可以根据某些条件或检查合并请求。

```
CREATE TABLE merge_request_metrics (
    id integer NOT NULL,
    latest_build_started_at timestamp without time zone,
    latest_build_finished_at timestamp without time zone,
    merged_at timestamp without time zone,
    created_at timestamp without time zone NOT NULL,
    updated_at timestamp without time zone NOT NULL,

    latest_closed_at timestamp with time zone,
    first_comment_at timestamp with time zone,
    first_commit_at timestamp with time zone,
    last_commit_at timestamp with time zone,
    first_approved_at timestamp with time zone,
    first_reassigned_at timestamp with time zone
);

```

## 6\. 外键约束

外键约束是两个表之间行的逻辑关联。通常使用外键在查询中连接表。

> **FOREIGN KEY** 约束是一个数据库构造，一个强制外键关系完整性（参考完整性）的实现。也就是说，它确保子表只能引用父表中存在的适当行。约束还通过不同方法防止了“孤立行”的存在。– [链接](https://docs.planetscale.com/learn/operating-without-foreign-key-constraints)

在过去几年中，我参与了多个项目，团队/架构师决定不使用外键约束。他们主要引用性能原因不使用外键约束。

当你创建外键时，性能可能会下降的一个原因是使用`ON DELETE CASCADE`操作。`ON DELETE CASCADE`操作的工作方式是，如果你删除父表中的一行，则在同一事务中也会删除子表中的任何引用行。你可能期望只删除一行，但最终可能会删除数百或数千乃至更多的子表行。但是，只有当一个父行与大量子表行关联时，才会出现这个问题。

团队不使用外键约束的另外两个原因是：

+   它们在特别是在MySQL的在线DDL模式迁移操作中表现不佳。

+   一旦将数据分片到多个数据库服务器中，维护外键约束就变得困难。

> 像PlanetScale（基于开源Vitess数据库）这样的MySQL兼容无服务器数据库不支持外键。

因此，我很想知道GitLab是否使用外键约束。

GitLab在大多数表中使用外键约束，但在少数表中如`audit_events`、`abuse_reports`、`web_hooks_logs`、`spam_logs`中不使用。我认为他们在`audit_events`、`abuse_reports`、`web_hooks_logs`、`spam_logs`表中不使用外键约束的两个主要原因是：

+   这些表的特性是不可变的。一旦条目写入，你就不希望对其进行更改。

+   这些表可能会增长到数百万甚至更多的行，因此即使有轻微的性能损失也会产生巨大影响。

GitLab在其余使用外键的表中同时使用`ON DELETE CASCADE`、`ON DELETE RESTRICT`和`ON DELETE SET NULL`操作。每种操作的示例如下所示。

```
ALTER TABLE ONLY todos
    ADD CONSTRAINT fk_rails_a27c483435 FOREIGN KEY (group_id) REFERENCES namespaces(id) ON DELETE CASCADE;

ALTER TABLE ONLY projects
    ADD CONSTRAINT fk_projects_namespace_id FOREIGN KEY (namespace_id) REFERENCES namespaces(id) ON DELETE RESTRICT;

ALTER TABLE ONLY authentication_events
    ADD CONSTRAINT fk_rails_b204656a54 FOREIGN KEY (user_id) REFERENCES users(id) ON DELETE SET NULL;

```

+   `ON DELETE SET NULL`将子表中匹配行的引用列设置为null。这会导致孤立行，但由于为NULL，你可以轻松识别它们。在这种操作中，单行删除可能导致子表中多行更新。这可能导致大事务、过多锁定和复制延迟。

+   `ON DELETE RESTRICT`阻止删除引用的子行。这不会导致孤立的子行，因为如果有引用它的子行，就无法删除父行。你会收到如下所示的异常。

```
  ERROR:  update or delete on table "a" violates foreign key constraint "fk_a_id" on table "b"
  DETAIL:  Key (id)=(1) is still referenced from table "b".

```

## 7\. 分区大表

GitLab使用分区来对可能会增长到巨大大小的表进行分区。这样做是为了提高查询性能。

+   按范围分区：这种分区策略根据选择的范围对表数据进行分区。当需要对时间序列数据进行分区时，通常会使用这种策略。`audit_events`和`web_hook_logs`表使用了这种策略。

+   按列表分区：这种分区策略根据列的离散值对表数据进行分区。`loose_foreign_keys_deleted_records`表使用了这种策略。

+   PARTITION BY HASH：指定每个分区的模数和余数来对表进行分区。每个分区将包含使得分区键的哈希值除以指定的模数产生指定余数的行。表 `product_analytics_events_experimental` 使用了这种策略。

您可以在 [Postgres 文档](https://www.postgresql.org/docs/current/ddl-partitioning.html) 中了解更多关于 Postgres 分区的信息。

## 8\. 支持使用 Trigrams 和 `gin_trgm_ops` 的 LIKE 搜索用例

GitLab 使用 GIN（Generalized Inverted Index）索引来执行高效搜索。

> GIN 索引类型旨在处理可分割的数据类型，并且您希望搜索各个组件值（例如数组元素、文本文档中的词元等）– [Tom Lane](https://www.postgresql.org/message-id/flat/26038.1559516834%40sss.pgh.pa.us#ccb004aefc151d913e7a274a9b30c631)

由于 LIKE 操作的性质，它支持任意通配符表达式，这在本质上很难索引。一个例子是 issues 表，您可能希望在标题和描述字段上进行搜索。因此，我们使用 pg_trgm 扩展来创建一个可以在三字母组上工作的索引。

```
CREATE INDEX index_issues_on_title_trigram ON issues USING gin (title gin_trgm_ops);
CREATE INDEX index_issues_on_description_trigram ON issues USING gin (description gin_trgm_ops);

```

GIN 索引使得搜索高效。让我们看看它的实际效果。

我们将创建一个如下所示的简单表。

```
create table words(id serial primary key, word text not null);

```

让我们插入一些数据。我从这个 [链接](https://www.bragitoff.com/2016/03/english-dictionary-in-csv-format/) 下载了一个英语单词列表的 CSV 格式。

```
\copy words(word) from '/Users/xxx/Aword.csv' CSV;

```

```
select count(*) from words;

```

```
 count
-------
 11616
(1 row)

```

我们将在 word 列上创建一个 btree 索引，然后我们将使用 gin 索引展示其效率。

```
create index id1 on words using btree (word);

```

让我们运行解释计划查询。

```
EXPLAIN select * from words where word like '%bul%';

```

```
                        QUERY PLAN
-----------------------------------------------------------
 Seq Scan on words  (cost=0.00..211.20 rows=1 width=14)
   Filter: (word ~~ '%bul%'::text)
(2 rows)

```

现在，让我们删除 btree 索引；

安装 `pg_trm` 扩展

```
CREATE EXTENSION pg_trgm;

```

创建索引。

```
create index index_words_on_word_trigram ON words USING gin (word gin_trgm_ops);

```

现在，让我们运行解释

```
EXPLAIN select count(*) from words where word like '%bul%';

```

```
                                             QUERY PLAN
----------------------------------------------------------------------------------------------------
 Aggregate  (cost=16.02..16.03 rows=1 width=8)
   ->  Bitmap Heap Scan on words  (cost=12.01..16.02 rows=1 width=0)
         Recheck Cond: (word ~~ '%bul%'::text)
         ->  Bitmap Index Scan on index_words_on_word_trigram  (cost=0.00..12.01 rows=1 width=0)
               Index Cond: (word ~~ '%bul%'::text)
(5 rows)

```

GitLab 还利用 `tsvector` 来支持完整的全文搜索。

在您的主数据存储中执行文本搜索的优点是：

+   实时索引。创建索引无需延迟

+   访问完整数据

+   在您的架构中减少复杂性

## 9\. 使用 `jsonb`

如我在早些时候的 [文章](https://shekhargulati.com/2022/01/08/when-to-use-json-data-type-in-database-schema-design/) 中讨论的，我在数据库模式设计中使用 json 数据类型来支持以下用例：

1.  转储请求数据，稍后进行处理

1.  支持额外字段

1.  一对多关系，其中多方不需要拥有自己的标识

1.  键值对使用情况

1.  更简单的实体-属性-值（EAV）设计

GitLab 的模式设计还在多个表中使用 `jsonb` 数据类型。他们主要用于我上述列表中的第1和第2个用例。与存储为纯文本相比，使用 jsonb 数据类型的优势在于 Postgres 对 `jsonb` 数据类型支持的高效查询。

表 `error_tracking_error_events` 使用 jsonb 数据类型存储有效载荷。这是一个将在稍后处理的转储请求数据的示例用例。我在我的博客文章中涵盖了一个类似的用例，因此请阅读了解更多信息。

```
CREATE TABLE error_tracking_error_events (
    id bigint NOT NULL,
    payload jsonb DEFAULT '{}'::jsonb NOT NULL,
  // rest removed
);

```

您可以使用 JSON 模式验证 JSON 文档的结构。

另一个示例是下面显示的`operations_strategies`表。您不知道可能收到多少参数，因此需要像`jsonb`这样的灵活数据类型。

```
CREATE TABLE operations_strategies (
    id bigint NOT NULL,
    feature_flag_id bigint NOT NULL,
    name character varying(255) NOT NULL,
    parameters jsonb DEFAULT '{}'::jsonb NOT NULL
);

```

下面显示了支持额外字段用例的示例。

```
CREATE TABLE packages_debian_file_metadata (
    created_at timestamp with time zone NOT NULL,
    updated_at timestamp with time zone NOT NULL,
    package_file_id bigint NOT NULL,
    file_type smallint NOT NULL,
    component text,
    architecture text,
    fields jsonb,
);

```

他们还使用jsonb来存储已经以JSON格式存在的数据。例如，在`vulnerability_finding_evidences`表中，报告数据已经是JSON格式，因此他们将其保存为`jsonb`数据类型。

```
CREATE TABLE vulnerability_finding_evidences (
    id bigint NOT NULL,
    created_at timestamp with time zone NOT NULL,
    updated_at timestamp with time zone NOT NULL,
    vulnerability_occurrence_id bigint NOT NULL,
    data jsonb DEFAULT '{}'::jsonb NOT NULL
);

```

## 10\. 其他小提示

+   像`updated_at`这样的审计字段仅在可以修改记录的表中使用。例如，`issues`有一个`updated_at`列。对于只追加不可变日志表，如代码片段中所示的`audit_events`，不具有`updated_at`列。

```
  CREATE TABLE issues (
      id integer NOT NULL,
      title character varying,
      author_id integer,
      project_id integer,
      created_at timestamp without time zone,
      updated_at timestamp without time zone,
      // removed remaining columns and constraints
  );

```

`audit_events`表中没有`updated_at`列。

```
  CREATE TABLE audit_events (
      id bigint NOT NULL,
      author_id integer NOT NULL,
      entity_id integer NOT NULL,
      created_at timestamp without time zone NOT NULL,
      // removed remaining columns and constraints
  )

```

+   枚举类型存储为smallint而不是`character varying`。它节省了空间。唯一的问题是您无法更改枚举值的顺序。在下面显示的示例中，`reason`和`severity_level`是枚举类型。

```
  CREATE TABLE merge_requests_compliance_violations (
      id bigint NOT NULL,
      violating_user_id bigint NOT NULL,
      merge_request_id bigint NOT NULL,
      reason smallint NOT NULL,
      severity_level smallint DEFAULT 0 NOT NULL
  );

```

+   乐观锁定在少数（8）个表中使用，例如`issues`和`ci_builds`，以防止多方对数据的编辑。乐观锁定假定数据冲突最小化，如果确实发生冲突，则应用程序会引发异常并忽略更新。如果存在`lock_version`字段，Active Record支持乐观锁定。对记录的每次更新都会增加`lock_version`列，锁定设施确保实例化两次的记录会使最后一个保存的记录抛出`StaleObjectError`，如果第一个记录也已更新，则不会。下面显示的`ci_builds`表使用了‘lock_version`列。

```
  CREATE TABLE ci_builds (
      status character varying,
      finished_at timestamp without time zone,
      trace text,

        lock_version integer DEFAULT 0,

        // removed columns
      CONSTRAINT check_1e2fbd1b39 CHECK ((lock_version IS NOT NULL))
  );

```

+   使用inet存储IP地址。我以前不知道`inet`类型。他们在`audit_events`和`authentication_events`表中使用了inet

```
  CREATE TABLE audit_events (
      id bigint NOT NULL,
      ip_address inet,
    // other columns removed for clarity
  );

```

GitLab并没有在存储`ip_address`的所有表中使用inet。例如，在`ci_runners`和`user_agent_details`表中，他们将其存储为`character varying`。我不确定为什么他们没有在存储IP地址的所有表中使用相同的类型。

您应该优先考虑`inet`，而不是将IP地址存储为纯文本类型，因为这些类型提供输入错误处理和专门的函数。

让我们快速看看它的操作。我们将从创建一个包含两个字段（id和`ip_addr`）的表开始

```
   create table e (id serial primary key, ip_addr inet not null);

```

我们可以像下面显示的那样插入一个有效记录。

```
  insert into e(ip_addr) values ('192.168.1.255');

```

我们还可以像下面显示的那样使用掩码插入记录。

```
  insert into e(ip_addr) values ('192.168.1.5/24');

```

这两条记录都将被插入

```
  select id, abbrev(ip_addr) from e;

```

```
   id |     abbrev
  ----+----------------
    1 | 192.168.1.255
    2 | 8.8.8.8
    3 | 192.168.1.5/24
  (3 rows)

```

如果我们尝试保存无效数据，则插入将失败。

```
  insert into e(ip_addr) values ('192.168.1');

```

```
  ERROR:  invalid input syntax for type inet: "192.168.1"
  LINE 1: insert into e(ip_addr) values ('192.168.1');

```

您可以使用Postgres支持的inet运算符来检查IP地址是否包含在子网中，如下所示。

```
  select * from e where ip_addr << inet '192.168.1.1/24';

```

```
   id |    ip_addr
  ----+---------------
    1 | 192.168.1.255
  (1 row)

```

如果我们要检查子网是否包含或等于，则执行以下操作

```
  select * from e where ip_addr <<= inet '192.168.1.1/24';

```

```
   id |    ip_addr
  ----+----------------
    1 | 192.168.1.255
    3 | 192.168.1.5/24
  (2 rows)

```

有许多其他由Postgres支持的运算符和函数。您可以在Postgres [文档](https://www.postgresql.org/docs/current/functions-net.html)中阅读它们。

+   Postgres的‘bytea`数据类型用于存储`SHA`、加密令牌、加密密钥、加密密码、指纹等。

+   Postgres数组类型用于存储具有多个值的列，如下所示。

> 当您确定不需要将数组中的项目与任何其他表创建关系时，请使用数组。它应该用于紧密耦合的一对多关系。 – [链接](https://stackoverflow.com/a/56298555)

例如，在下面显示的表中，我们将`*_ids`存储为数组，而不是以扁平方式存储它们，并与其他表定义关系。您不知道会提到多少用户和项目，因此使用类似`mentioned_user_id1`、`mentioned_user_id2`等列将是浪费的。

```
  CREATE TABLE alert_management_alert_user_mentions (
      id bigint NOT NULL,
      alert_management_alert_id bigint NOT NULL,
      note_id bigint,
      mentioned_users_ids integer[],
      mentioned_projects_ids integer[],
      mentioned_groups_ids integer[]
  );

```

Postgres数组的另一个常见用例是存储像主机、标签、URL等字段。

```
  CREATE TABLE dast_site_profiles (
      id bigint NOT NULL,
      excluded_urls text[] DEFAULT '{}'::text[] NOT NULL,
  );
  CREATE TABLE alert_management_alerts (
    id bigint NOT NULL,
    hosts text[] DEFAULT '{}'::text[] NOT NULL,
  );
  CREATE TABLE ci_pending_builds (
      id bigint NOT NULL,
      tag_ids integer[] DEFAULT '{}'::integer[],
  );

```

## 结论

我从GitLab架构中学到了很多。他们并不会盲目地将相同的实践应用于所有表设计。每个表都根据其目的、存储的数据类型和增长速率做出最佳决策。

## 参考资料

1.  GitLab架构`structure.sql` – [链接](https://gitlab.com/gitlab-org/gitlab/-/blob/master/db/structure.sql)

1.  问题29465：使用structure.sql而不是schema.rb – [链接](https://gitlab.com/gitlab-org/gitlab/-/issues/29465)

1.  在Postgres中选择主键类型 – [链接](https://shekhargulati.com/2022/06/23/choosing-a-primary-key-type-in-postgres/)

1.  GitHub通向128M公共存储库的路径 – [链接](https://towardsdatascience.com/githubs-path-to-128m-public-repositories-f6f656ab56b1#:~:text=There%20are%20over%20128%20million%20public%20repositories%20on%20GitHub.)

1.  [Postgres字符类型文档](https://www.postgresql.org/docs/current/datatype-character.html)

1.  文本与varchar（字符变长）之间的区别 – [链接](https://stackoverflow.com/questions/4848964/difference-between-text-and-varchar-character-varying)

1.  CHAR(x) vs. VARCHAR(x) vs. VARCHAR vs. TEXT – [链接](https://www.depesz.com/2010/03/02/charx-vs-varcharx-vs-varchar-vs-text/)
