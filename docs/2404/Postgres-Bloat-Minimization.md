<!--yml
category: 未分类
date: 2024-05-27 13:33:09
-->

# Postgres Bloat Minimization

> 来源：[https://supabase.com/blog/postgres-bloat](https://supabase.com/blog/postgres-bloat)

### How data of Postgres tables stored[#](#how-data-of-postgres-tables-stored)

By default, all table data in Postgres are physically stored using the “heap” method. So every database is a set of 1Gb files (”segments”) and each file is logically split into 8Kb pages. Actual table rows are put into any page with enough free space.

When the row data is updated, a new version of a whole row is constructed and written (to any free space). The old one remains because, at the time of the update, the transaction is not completed and can be rolled back in the future. When the transaction is completed we’ll have two or several versions of the same row in the table. Cleaning old ones is by an asynchronous process called vacuum (and autovacuum).

Vacuum goes through all table files looking for row versions that were updated or deleted by already completed transactions and frees the space on the pages.

Then it updates the table’s free-space-map to reflect that some page has a certain amount of free space for row inserts.

It also updates the visibility map for a page. It marks that all remaining rows are visible. So index scans can skip visibility checks, which is not so for the modified page before vacuuming. This significantly increases the speed of queries using indexes.

In many cases vacuum runs automatically, cleans everything, and requires little care. But in some scenarios, we need to go deeper and tune the autovacuum parameters or run the vacuum manually.

Vacuum marks space in a relation file as free to use for future row inserts or updates. And it’s not a problem unless we insert many rows, delete many rows at once, and then don’t make any inserts or updates. Space remains reserved in a file but we don’t use it.

In this case, we could free actual filesystem space by running more aggressive mode:

It will rebuild the table from live rows and they will be placed compactly so that filesystem space will be freed. The downside is that it needs an exclusive lock and you won’t be able to modify the table while `VACUUM FULL` does it's work. It’s wise to execute that process when the database is least accessed e.g. at night.

An alternative way that doesn’t need full locks is using pg_repack extension:

 `_10

pg_repack --table mytable test` 

pg_repack operates similarly to VACUUM FULL for database `test` and table `mytable`.

To see database bloat via the [Supabase CLI](https://supabase.com/docs/guides/cli/getting-started) run:

 `_10

$ supabase inspect db bloat` 

or:

 `_13

-- number of dead rows

_13

-- number of live rows` 

If the numbers differ by more than 2x, chances are that the autovacuum didn’t start or hasn’t completed for a table. There could be several legitimate reasons for this.

You can see information for the last successful autovacuum by running:

 `_10

$ supabase inspect db vacuum-stats` 

or:

Let’s turn on autovacuum logging so that all autovacuum events land in the log:

 `_10

ALTER TABLE mytable SET log_autovacuum_min_duration to 0;` 

There are two ways we can encounter this situation:

*   autovacuum hasn’t started
*   autovacuum did start, (possibly multiple times) but never succeeded

Autovacuum starts based on several configuration parameters like timeout and pattern of access to a particular table. Maybe it’s even legitimate that it hasn’t started.

*   `autovacuum_vacuum_threshold` - number of rows updated or deleted in a table to invoke autovacuum
*   `autovacuum_vacuum_insert_threshold` - number of rows inserted into a table to invoke autovacuum
*   `autovacuum_vacuum_scale_factor` - a fraction of table modified by updates or deletes to invoke autovacuum
*   `autovacuum_vacuum_insert_scale_factor` - a fraction of table modified by inserts to invoke autovacuum

With all these parameters set, the autovacuum will start if the number of rows updated or deleted exceeds:

 `_10

autovacuum_vacuum_scale_factor * size_of_table + autovacuum_vacuum_threshold` 

The same logic applies for inserts. Default scale factors are 20% of a table, which could be too high for big tables. If we want autovacuum to occur on large tables more frequently and take less time each run, decrease the default values for these tables e.g.:

 `_10

ALTER TABLE mytable SET autovacuum_vacuum_scale_factor to 0.05;` 

*   `autovacuum_naptime` (default 1 min) - Each 1-minute autovacuum daemon will see the state of all tables in the database and decide whether to start autovacuum for a table. Most often this parameter does not need to be modified.

To see global vacuum settings for your cluster let’s run:

 `_10

SELECT * from pg_settings where category like 'Autovacuum';` 

To see current settings for a table (that overrides global settings) run:

 `_10

SELECT relname, reloptions FROM pg_class WHERE relname='mytable';` 

The most common reason autovacuum doesn’t succeed is long-running open transactions that access old row versions. In that case, Postgres recognizes that the row versions are still needed so any row versions created after that point can’t be marked as dead. One common cause for this problem is interactive sessions that were left open on accident. When tuples can’t be marked as dead, the database begins to bloat.

To see all open transactions run:

 `_10

SELECT xact_start, state FROM pg_stat_activity;` 

To close transactions found to be idling:

 `_10

SELECT pg_cancel_backend(pid) FROM pg_stat_activity WHERE xact_start = '<value from previous query>';` 

For automatically closing idle transactions in a session:

 `_10

SET idle_in_transaction_session_timeout TO '10000s';` 

The same parameter could be set per role or database as needed.

Another, less likely, possibility is that autovacuum can’t succeed due to locks. If some of your processes take the `SHARE UPDATE EXCLUSIVE` lock e.g. `ALTER TABLE` clause, this lock will prevent vacuum from processing a table. Lock conflicts in your ordinary transactions could cause `SHARE UPDATE EXCLUSIVE` to be taken for a long time. A good recipe when this happens is to cancel all the open transactions and run VACUUM for a table manually (or wait until the next autovacuum comes for it).

 `_10

SELECT pg_cancel_backend(pid) FROM pg_stat_activity WHERE state = 'active';` 

There could be too few autovacuum workers or each worker could operate slowly due to the low setting of `autovacuum_work_mem`.

*   `autovacuum_max_workers` (default 3) - Number of parallel workers doing autovacuum for tables. When you have enough cores, increasing the default value is worthwhile. Note that this will decrease the number of possible backends or regular parallel workers running at a time.
*   `autovacuum_work_mem` (default equal to `maintenance_work_mem` or 64Mb) - work memory that is used per autovacuum worker. If you see in your logs that autovacuum for some tables starts but takes a long time, that time may be decreased by increasing this value. This parameter can only be modified in the config file.

Vacuum and autovacuum are efficient ways to maintain the tables without bloat. They have several parameters that allow efficient tuning. Some insight into what database does can help prevent the cases where autovacuum becomes problematic and bloat increases i.e.:

*   Long open transactions
*   Stuck locks
*   Insufficient resources allocated to vacuuming
*   Space not freed on a filesystem level after massive table modifications

For more information about bloat and vacuuming, here are some direct references to resources:

[Full official documentation for vacuum](https://www.postgresql.org/docs/current/routine-vacuuming.html#VACUUM-BASICS)

[Per-table vacuum parameters](https://www.postgresql.org/docs/current/sql-createtable.html#SQL-CREATETABLE-STORAGE-PARAMETERS)

[Global autovacuum parameters (most of them need modifying postgresql.conf, but could be overridden by per-table parameters)](https://www.postgresql.org/docs/current/runtime-config-autovacuum.html)

[Supabase CLI reference](https://supabase.com/docs/reference/cli/global-flags)