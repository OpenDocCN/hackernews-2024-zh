<!--yml

category: 未分类

date: 2024-05-27 13:33:09

-->

# Postgres Bloat Minimization

> 来源：[https://supabase.com/blog/postgres-bloat](https://supabase.com/blog/postgres-bloat)

### Postgres 表数据如何存储[#](#how-data-of-postgres-tables-stored)

默认情况下，Postgres 中所有表数据都是使用“堆”方法物理存储的。因此，每个数据库都是一组1Gb文件（“段”），每个文件逻辑上分成8Kb页面。实际表行放置在具有足够自由空间的任何页面中。

当行数据更新时，构造并写入一个整个行的新版本（到任何空闲空间）。旧版本仍然存在，因为在更新时，事务尚未完成，未来可能会回滚。事务完成时，我们将在表中拥有两个或多个相同行的版本。清理旧版本由称为清理（和自动清理）的异步过程完成。

清理通过已完成事务更新或删除的所有表文件中的行版本，并在页面上释放空间。

然后它更新表的自由空间映射，以反映某些页面具有用于行插入的特定数量的自由空间。

它还更新页面的可见性图。它标记所有剩余行都是可见的。因此，索引扫描可以跳过可见性检查，这对于修改前的页面不适用。这显著提高了使用索引的查询速度。

在许多情况下，自动清理会自动运行，清理一切，需要很少的关注。但在某些情况下，我们需要深入了解并调整自动清理参数或手动运行清理。

清理将关系文件中的空间标记为未来行插入或更新可用。除非我们一次插入许多行，一次删除许多行，然后不进行任何插入或更新，否则这不是问题。文件中的空间仍保留，但我们不使用它。

在这种情况下，我们可以通过运行更积极的模式来释放实际文件系统空间：

它将从活动行重新构建表，并将其紧凑放置，以便释放文件系统空间。缺点是它需要一个排他锁，并且在`VACUUM FULL`执行时无法修改表。建议在数据库最少访问时执行此过程，例如晚上。

一个不需要完全锁定的替代方法是使用 pg_repack 扩展：

`_10

`pg_repack --table mytable test`

pg_repack 类似于对数据库 `test` 和表 `mytable` 进行 `VACUUM FULL` 操作。

要通过[Supabase CLI](https://supabase.com/docs/guides/cli/getting-started)查看数据库膨胀情况，请运行：

`_10

$ supabase inspect db bloat`

或：

`_13

-- 未使用行数

`_13

-- 活动行数`

如果数字相差超过2倍，很可能是自动清理未启动或未完成表。可能有几个合法的原因。

您可以通过运行以下命令查看最后一次成功的自动清理信息：

`_10

$ supabase inspect db vacuum-stats`

或：

让我们打开autovacuum日志，以便所有autovacuum事件都记录在日志中：

`_10

ALTER TABLE mytable SET log_autovacuum_min_duration to 0;`

我们可能会遇到这种情况的两种方式：

+   autovacuum尚未启动

+   autovacuum已启动，（可能多次），但从未成功过

Autovacuum根据几个配置参数如超时和对特定表的访问模式启动。也许它甚至合理地没有启动。

+   `autovacuum_vacuum_threshold` - 表中更新或删除的行数以调用autovacuum

+   `autovacuum_vacuum_insert_threshold` - 插入到表中的行数以调用autovacuum

+   `autovacuum_vacuum_scale_factor` - 由更新或删除修改的表的分数以调用autovacuum

+   `autovacuum_vacuum_insert_scale_factor` - 由插入修改的表的分数以调用autovacuum

当所有这些参数设置完成后，如果更新或删除的行数超过以下数值，自动清理将开始：

`_10

autovacuum_vacuum_scale_factor * size_of_table + autovacuum_vacuum_threshold`

相同的逻辑适用于插入操作。默认比例因子为表的20%，这对大表可能太高。如果我们希望对大表更频繁地进行自动清理，并且每次运行所需时间较短，则可以减少这些表的默认值，例如：

`_10

ALTER TABLE mytable SET autovacuum_vacuum_scale_factor to 0.05;`

+   `autovacuum_naptime`（默认为1分钟） - 每1分钟，autovacuum守护进程将检查数据库中所有表的状态，并决定是否为表启动autovacuum。大多数情况下，这个参数不需要修改。

要查看集群的全局清理设置，让我们运行：

`_10

SELECT * from pg_settings where category like 'Autovacuum';`

要查看表的当前设置（覆盖全局设置），运行：

`_10

SELECT relname, reloptions FROM pg_class WHERE relname='mytable';`

autovacuum无法成功的最常见原因是长时间运行的访问旧行版本的未关闭事务。在这种情况下，Postgres会认识到仍然需要这些行版本，因此在此之后创建的任何行版本都无法标记为死行。导致此问题的常见原因之一是意外保持打开的交互会话。当无法标记行为死行时，数据库会开始膨胀。

要查看所有打开的事务运行：

`_10

SELECT xact_start, state FROM pg_stat_activity;`

为了关闭发现处于空闲状态的事务：

`_10

SELECT pg_cancel_backend(pid) FROM pg_stat_activity WHERE xact_start = '<previous query中的值>';`

用于在会话中自动关闭空闲事务的参数：

`_10

SET idle_in_transaction_session_timeout TO '10000s';`

同一参数可以根据需要设置为角色或数据库。

另一个不太可能的可能性是由于锁定导致自动真空无法成功。如果您的某些进程获取了`SHARE UPDATE EXCLUSIVE`锁，例如`ALTER TABLE`子句，此锁将阻止真空处理表。在您的普通事务中的锁冲突可能导致长时间获取`SHARE UPDATE EXCLUSIVE`。当发生这种情况时的一个好方法是取消所有打开的事务并手动为表运行真空（或等待下一个自动真空来处理它）。

_10

SELECT pg_cancel_backend(pid) FROM pg_stat_activity WHERE state = 'active';`

可能是自动真空工作者过少或每个工作者由于`autovacuum_work_mem`设置低而运行缓慢。

+   `autovacuum_max_workers`（默认为3） - 执行表自动真空的并行工作者数量。当您有足够的核心时，增加默认值是值得的。请注意，这将减少同时运行的可能后端或常规并行工作者的数量。

+   `autovacuum_work_mem`（默认等于`maintenance_work_mem`或64Mb） - 每个自动真空工作者使用的工作内存。如果在日志中看到某些表的自动真空开始但花费了很长时间，可能可以通过增加此值减少时间。此参数只能在配置文件中修改。

用真空和自动真空是维护表而不膨胀的有效方式。它们有几个参数允许有效调整。对数据库做一些了解可以帮助防止自动真空成为问题，导致膨胀增加，例如：

+   长时间开放的事务

+   卡住的锁

+   分配给真空不足的资源

+   文件系统级别在大规模表修改后未释放的空间

有关膨胀和真空的更多信息，请参考以下资源：

[真空的完整官方文档](https://www.postgresql.org/docs/current/routine-vacuuming.html#VACUUM-BASICS)

[每个表的真空参数](https://www.postgresql.org/docs/current/sql-createtable.html#SQL-CREATETABLE-STORAGE-PARAMETERS)

[全局自动真空参数（大多数需要修改 postgresql.conf，但可以由每个表参数覆盖）](https://www.postgresql.org/docs/current/runtime-config-autovacuum.html)

[Supabase CLI 参考](https://supabase.com/docs/reference/cli/global-flags)
