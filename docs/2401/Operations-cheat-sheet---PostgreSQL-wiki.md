<!--yml
category: 未分类
date: 2024-05-27 14:54:02
-->

# Operations cheat sheet - PostgreSQL wiki

> 来源：[https://wiki.postgresql.org/wiki/Operations_cheat_sheet](https://wiki.postgresql.org/wiki/Operations_cheat_sheet)

## Introduction

This page is aimed at learning from the wisdom of the PostgreSQL community.

People involved in the PostgreSQL community, as organizations or individuals, are posting vast amounts of useful information on blogs, wikis, and websites. However, they are scattered, and it will not be easy to find the information you are looking for.

Therefore, this page has compiled a collection of links to articles that the editor(s) informative, from hundreds of blog sites registered on Planet PostgreSQL, plus the PostgreSQL wiki and websites. Some notable topics from these articles are picked up here, trying to organize and summarize them for introduction purposes. Please feel free to add links to the articles you find helpful to others. Adding the summaries of them is also appreciated in addition to the links.

**Notes**

*   do not expect everything to be extracted from those articles! Diving directly into the original articles is strongly recommended.
*   Some configuration parameters, functions, statistics views, and system tables/views may only be available after some major versions. Consult PostgreSQL documentation for availability.

## Architecture

### Client-Server architecture

*   Server
    *   Database server: postgres
    *   Server application: ex. initdb, pg_ctl, pg_upgrade
*   Client
    *   Client interface: ex. libpq, ECPG, pgJDBC, psqlODBC, Npgsql
    *   Client application: ex. psql, pgbench, pg_dump, pg_restore
*   Frontend/Backend protocol
    *   Frontend=client, Backend=server
    *   message-based protocol for communication between frontends and backends over TCP/IP and Unix-domain sockets
    *   Current version has been 3.0 since PostgreSQL 7.4 in 2003
*   Compatibility between client and server
    *   psql works best with servers of the same or an older major version. It also works with servers of a newer major version, although backslash commands are likely to fail.
    *   pg_dump can dump from servers older than itself (servers back to 9.2\. are supported). It cannot dump from servers newer major versions.
    *   Driver is server-version agnostic: Always use the latest driver version
    *   ex. pgJDBC supports 8.2+ server, Npgsql is tested with supported 5 server major versions.

### Logical database structures

*   Database cluster is a collection of databases, roles, and tablespaces
*   The database cluster initially contains some databases after initdb:
    *   template1: a new database is cloned from this, unless another template database is specified
    *   template0: a pristine copy of the original contents of template1
    *   postgres: default database used by utilities and users
*   Each database contains its own system catalogs that store the metadata of its local database objects
*   The database cluster contains some shared system catalogs that store the metadata of the cluster-wide global objects
    *   Shared system catalogs can be accessed from inside each database
    *   Query to get shared system catalogs: `SELECT relname FROM pg_class WHERE relisshared AND relkind = 'r';`
    *   ex. pg_authid, pg_database, pg_tablespace
*   Tablespace
    *   pg_global ($PGDATA/global/): store shared system catalogs
    *   pg_default ($PGDATA/base/): store template0, template1, postgres. The default tablespace for other databases.
    *   User tablespaces: created with `CREATE TABLESPACE name LOCATION 'dir_path';`

### Database object hierarchy

*   Database
    *   Access method
    *   Cast
    *   Event trigger
    *   Extension
    *   Foreign-data wrapper
    *   Foreign server
    *   Procedural language
    *   Publication
    *   Row-level security policy (the name must be distinct from that of any other policy for the table)
    *   Rule (the name must be distinct from that of any other rule for the same table)
    *   Schema
        *   Aggregate function
        *   Collation
        *   Conversion
        *   Data type
        *   Domain
        *   Extended statistics
        *   Foreign table
        *   Function
        *   Index
        *   Materialized view
        *   Operator
        *   Operator class
        *   Operator family
        *   Procedure
        *   Sequence
        *   Table
        *   Text search configuration
        *   Text search dictionary
        *   Text search parser
        *   Text search template
        *   Trigger (inherits the schema of its table)
        *   View
    *   Subscription
    *   Transform
    *   User mapping
*   Role
*   Tablespace

### Object identifier (OID)

*   OIDs are used internally by PostgreSQL as primary keys for various system tables.
    *   ex. `SELECT oid, relname FROM pg_class WHERE relname = 'mytable';`
*   Type oid represents an OID.
*   oid is an unsigned four-byte integer.
*   An OID is allocated from a single cluster-wide counter, so it is not large enough to provide database-wide uniqueness.
    *   A specific object is identified by two OIDs (classid and objid) in pg_depend and pg_shdepend.

### Physical database structures

Directories

*   Data directory ($PGDATA)
    *   base/: Subdirectory containing per-database subdirectories
    *   global/: Subdirectory containing cluster-wide tables, such as pg_database
    *   pg_multixact/: Subdirectory containing multitransaction status data (used for shared row locks)
    *   pg_subtrans/: Subdirectory containing subtransaction status data
    *   pg_tblspc/: Subdirectory containing symbolic links to tablespaces
    *   pg_wal/: Subdirectory containing WAL (Write-Ahead Log) files
    *   pg_xact/: Subdirectory containing transaction commit status data
*   Configuration file directories (optional)
*   Tablespace directories (optional)
*   WAL directory (optional)

Files in data directory

*   Configuration files (postgresql.conf, pg_hba.conf, pg_ident.conf): Can be stored in other directories
*   Control file (global/pg_control): Stores control info such as the cluster state, checkpoint log location, next OID, next XID
*   Regular relation data file
    *   A relation is a set of tuples: table, index, sequence, materialized view, etc.
    *   Each relation has its own set of files.
    *   Each file consists of 8 KB blocks.
    *   Lazy allocation: A new heap table file contains 0 blocks, while a new B-tree index file contains 1 block.
    *   There are some types of data files (forks): main, FSM, VM, initialization
    *   Main fork (`base/<database_OID>/<relation_filenode_no>`)
        *   ex. `"SELECT pg_relation_filepath('mytable');"` returns `base/17354/32185`, where 17354 is the database's OID and 32185 is the mytable's filenode number
        *   Stores tuple data.
    *   FSM (free space map) fork (`base/<database_OID>/<relation_filenode_no>_fsm`)
        *   Keeps track of free space in the relation.
        *   Entries are organized as a tree, where each leaf node entry stores free space in one relation block.
        *   [pg_freespacemap](https://www.postgresql.org/docs/current/pgfreespacemap.html) and [pageinspect](https://www.postgresql.org/docs/current/pageinspect.html) can be used to examine its contents.
    *   VM (visibility map) fork (`base/<database_OID>/<relation_filenode_no>_vm`)
        *   Keeps track of:
            *   which pages contain only tuples that are known to be visible to all active transactions
            *   which pages contain only frozen tuples
        *   Each heap relation has a Visibility Map; an index does not have one.
        *   Stores two bits per heap page:
            *   All-visible bit: if set, the page does not contain any tuples that need to be vacuumed. Also used by index-only scans to answer queries using only the index tuple.
            *   All-frozen bit: if set, all tuples on the page have been frozen, therefore vacuum can skip the page.
        *   [pg_visibility](https://www.postgresql.org/docs/current/pgvisibility.html) can be used to examine its contents.
    *   Initialization fork (base/<database_OID>/<relation_filenode_no>_init)
        *   Each unlogged table and index has an initialization fork.
        *   The content is empty: table is 0 block, index is 1 block.
        *   Unlogged relations are reset during recovery: the initialization fork is copied over the main fork, and other forks are erased.
*   Temporary relation data file
    *   `base/<database_OID>/tBBB_FFF`
        *   BBB is the backend ID of the backend which created the file, and FFF is the filenode number
        *   ex. `base/5/t3_16450`
    *   Has main, FSM, and VM forks, but not the initialization fork.
*   A large relation is divided into 1 GB segment files.
    *   e.g., `12345, 12345.1, 12345.2, ...`

Page (= block)

*   Each page is 8 KB. Configurable when building PostgreSQL.
*   Relations have the same format.
*   The content is the same in memory and on storage.
*   Each page stores multiple data values called items. In a table, an item is a row; in an index, an item is an index entry.
*   [pageinspect](https://www.postgresql.org/docs/current/pageinspect.html) can be used to examine the content.
*   Layout of a page:
    1.  Page header: 24 bytes
    2.  An array of item identifiers pointing to the actual items: Each entry is an (offset,length) pair. 4 bytes per item.
    3.  Free space
    4.  Items
    5.  Special space: 0 byte for tables, different bytes for index types (btree, GIN, GiST, etc.)

Table row

*   [[1]](https://www.postgresql.org/docs/current/pageinspect.html) pageinspect] can be used to examine the content.
*   Layout of a row
    1.  Header: 23 bytes
    2.  Null bitmap (optional): 1 bit for each column
    3.  User data: columns of the row

### Instance

The instance is a group of server-side processes, their local memory, and the shared memory.

Processes

*   Single-threaded: postmaster launches a single backend process for each client connection. Thus, each SQL execution only uses a single CPU core. Parallel query, index build, VACUUM etc. can utilize multiple CPU cores by running multiple server processes.
*   postmaster: The parent of all server processes. Controls the server startup and shutdown. Create shared memory and semaphores. Launches other server processes and reaps dead ones. Opens and listens on TCP ports and/or Unix domain sockets, accepts connection requests, and spawns client backends to pass the connection requests to.
*   (Client) backend: Acts on behalf of a client session and handles its requests, i.e., executes SQL commands.
*   Background processes
    *   logger: Catches all stderr output from other processes through pipes, and writes them to log files.
    *   checkpointer: Handles all checkpoints.
    *   background writer: Periodically wakes up and writes out dirty shared buffers so that other processes don't have to write them when they need to free a shared buffer to read in another page.
    *   startup: Performs crash and point-in-time recovery. Ends as soon as the recovery is complete.
    *   stats collector: Receives messages from other processes through UDP sockets, accumulates statistics about server activity, and writes them to files. The statistics can be viewed with pg_stat... views. This process is gone as of PostgreSQL 15.
    *   walwriter: Periodically wakes up and writes out WAL buffers to reduce the amount of WAL that other processes have to write. Also ensures the writes of commit WAL records from asynchronously committed transactions.
    *   archiver: Archives WAL files.
    *   autovacuum launcher: Always running when autovacuum is enabled. Schedules autovacuum workers to run.
    *   autovacuum worker: Connect to a database as determined in the launcher, examine system catalogs to select the tables, and vacuum them.
    *   parallel worker: Executes part of a parallel query plan.
    *   walreceiver: Runs on the standby server. Receives WAL from the walsender, stores it on disk, and tells the startup process to continue recovery based on it.
    *   walsender: Runs on the primary server. Sends WAL to a single walreceiver.
    *   logical replication launcher: Run on the subscriber. Coordinates logical replication workers to run.
    *   logical replication worker: Runs on the subscriber. An apply worker per subscription receives logical changes from walsender on the publisher and applies them. One or more tablesync workers perform initial table copy for each table.
*   Background worker: Runs system-supplied or user-supplied code. e.g., used for parallel query and logical replication.

Memory

*   Shared memory
    *   Shared buffers: Stores the cached copy of data files (main, FSM, and VM forks).
    *   WAL buffers: Transactions put WAL records here before writing them out to disk.
    *   Other various areas: One large shared memory segment is divided into areas for specific uses.
    *   The allocations of areas can be examined with [pg_shmem_allocations](https://www.postgresql.org/docs/current/view-pg-shmem-allocations.html).
*   Local memory
    *   Work memory: Allocated for a query operation such as sort and hash. Configured with work_mem and hash_mem_multiplier parameters.
    *   Maintenance work memory: Allocated for maintenance operations, such as VACUUM, CREATE INDEX, and ALTER TABLE. Configured with maintenance_work_mem parameter.
    *   Temporary buffers: Allocated for caching temporary table blocks. Configured with temp_buffers parameter.
    *   Other various areas: A memory context is allocated for a specific usage (e.g., a message from client, transaction, query plan, execution state). Hundreds of memory contexts per session are possible.
    *   The allocation and usage of memory contexts can be examined with [pg_backend_memory_contexts](https://www.postgresql.org/docs/current/view-pg-backend-memory-contexts.html) view for the current session, and with the function `pg_log_backend_memory_contexts(backend_pid)` for other sessions.

### Reading and writing database data

*   Read:
    *   First, search the shared buffers for a buffer containing the target block. If found, it's returned to the requester.
    *   Otherwise, allocate a buffer from a free buffer list, and read the target block from the data file into the buffer.
    *   If there's no free buffer, evict and use a used buffer. If it's dirty, writes out the buffer to disk.
*   Write
    *   Find the target shared buffer, modify its contents, and write the changes to the WAL buffers.
    *   The modifying transaction writes out its WAL records from the WAL buffers to disk, including the commit WAL record.
    *   The modified dirty shared buffers are flushed to disk by background writer, checkpointer, or any other process. This is asynchronous with the transaction completion.
*   Any backend can read and write shared buffers, WAL buffers, data and WAL files. Unlike some other DBMSs, writes are not exclusively performed by a particular background process.
*   The database data file is read and written one block at a time. There's no multiblock I/O.
*   Some operations bypass shared buffers: the write of an index during index creation, CREATE DATABASE, ALTER TABLE ... SET TABLESPACE

### Query processing

1.  A client connects to a database, sends a query (SQL command) to the server, and receives the result.
2.  The parser first checks the query for correct syntax. Then, it interprets the semantics of the query to understand which tables, views, functions, data types, etc. are referenced.
3.  The rewrite system (rewriter) transforms the query based on the rules stored in the system catalog pg_rewrite. One example is the view: a query that accesses a view is rewritten to use the base table.
4.  The planner/optimizer creates a query plan.
5.  The executor executes the query plan and returns the result set to the client.

Notes

*   Each session runs on a connection to a single database. Therefore, it cannot access tables on a different database. However, one session can connect to another database and create another session via a foreign data wrapper like postgres_fdw, and access tables there. For example, an SQL command can join one table on the local database and another table on a remote database.
*   Each SQL command basically uses only one CPU core. A parallel query and some utility commands such as CREATE INDEX and VACUUM can use multiple CPU cores by running background workers.

### References

PostgreSQL Documentation

Other resources

## Reliability and availability

### Connection

What to check when troubleshooting connectivity

*   Is the database server reachable?
    *   `telnet <host> <port>`, or
    *   `nc -zv <host> <port>`
    *   Use `traceroute` (Unix/Linux) or `tracert` (Windows), specifying the protocol allowed by the host and intermediary routers.
*   Are the host and port correct?
*   Is the server running?
*   Does the server-side firewall allow communication through the port?
*   Does the client-side firewall allow communication to the server port?
*   Does pg_hba.conf have any entry that allow the combination of SSL/non-SSL, client host, database and user?
*   Is the listen_addresses parameter configured to allow connection through the desired IP addresses, including IPv4 and/or IPv6?
*   Are the database, user name, and password correct?
*   Does the user have permission to connect to the database?
    *   Check privileges with psql's `\l` or pg_database.datacl
*   Does the database server have enough CPU and memory resources?
*   Isn't the maximum connection limit reached?
    *   max_connections and superuser_reserved_connections parameters (at instance level)
    *   CREATE/ALTER DATABASE CONNECTION LIMIT (at database level)
    *   CREATE/ALTER ROLE CONNECTION LIMIT (at user level)

Connection termination and query cancellation

*   When the connection is closed, any incomplete transaction is rolled back.
*   Terminating a connection (`pg_terminate_backend()`) and canceling a query (`pg_cancel_backend()`) does not always work. For example, they don't work while the backend process is running in an uninterruptible section, such as waiting to acquire a lightweight lock, a read/write system call against a network storage device, and a loop without a cancellation point.
*   Set statement_timeout at appropriate levels (statement, user, database, instance). A short timeout is not recommended at a wide level because it cancels intentional long-running queries.
*   Set client-side timeouts appropriately.
*   Set server-side timeouts appropriately.
    *   tcp_keepalives_idle, tcp_keepalives_interval, tcp_keepalives_count
        *   TCP keep-alive works while the TCP connection is idle. It does not work when the socket connection is being established, or some data has been sent and waiting for its ACK.
        *   The effective timeout is tcp_keepalives_idle + tcp_keepalives_interval * tcp_keepalives_count.
    *   tcp_user_timeout
        *   Sets the TCP retransmission timeout. Relatively newly available on Linux.
        *   Comes to the rescue when TCP keep-alive doesn't help. i.e., when the socket connection is being established, or some data has been sent and waiting for its ACK.
        *   Confusing when used together with TCP keep-alive because this changes when TCP keep-alive times out. It would be safe to set tcp_user_timeout to tcp_keepalives_idle + tcp_keepalives_interval * tcp_keepalives_count.
    *   authentication_timeout
    *   idle_in_transaction_session_timeout
    *   idle_session_timeout
    *   client_connection_check_interval

Connection failover

*   Client drivers allow multiple hosts to be specified in the connection string.
    *   Connection timeout is applied to each host in the connection string. Therefore, making a connection may take unexpectedly long if there are many failed hosts before the running one in the list.
*   Restore session state after failover: session variables, prepared statements, temporary tables, holdable cursors (created with DECLARE CURSOR WITH HOLD), advisory locks, session user (set with SET SESSION AUTHORIZATION), current user (set with SET ROLE).
*   Be careful about transaction retry: It's unknown whether the transaction was committed or rolled back when a database server failover happens during the transaction commit.
    *   `pg_xact_status( xid8 )` can be used to determine the transaction outcome.

### WAL

What WAL is for

*   Crash recovery, archive recovery (Point-In-Time-Recovery: PITR), and replication
*   Updates are redone regardless of whether the transaction was committed.
*   There is no undo log (before image) or operation, unlike other popular DBMSs.
    *   Therefore, transaction rollback and crash recovery is fast.
    *   Changes by an aborted transaction are left in memory and on disk, but they are invisible to other transactions thanks to MVCC.

WAL structure

*   A sequence of 8 KB blocks.
*   Each block can contain multiple WAL records. Also, each WAL record can span multiple blocks.
*   The content is the same both in memory and on storage.
*   The WAL buffer is a contiguous array of blocks in memory. It's used in a circular fashion.
*   The WAL on storage is divided into WAL segment files. Each WAL segment file is 16 MB by default, which is configurable with `initdb`'s `--wal-segsize=size`.

Writing WAL

*   In memory, modify the data pages in shared buffers, and then write the change into the WAL buffer.
*   WAL buffer is always written to WAL files sequentially (no random write).
*   Before writing a dirty data page in a shared buffer out to disk, first all WAL records up to the latest one that affected the data page. This rule is the WAL (Write Ahead Log).
    *   Each data page has, in its page header, the location (LSN) of the WAL record that represents the latest update to it. This is the page LSN.
    *   LSN (Log Sequence Number): an unsigned 8-byte integer. It represents the WAL segment, block, and an offset in that block.
*   If writing to the WAL file fails, the instance will crash with a PANIC message.
*   WAL volume can grow beyond max_wal_size due to the reasons such as:
    *   Heavy writes like loading data with COPY
    *   failure to archive WAL files
    *   the large value of wal_keep_size
    *   an unused replication slot
*   SELECT can modify data pages and write WAL when:
    *   acquiring row locks, e.g., SELECT FOR UPDATE. They set xmax in the tuple header, and could possibly update MultiXact data structures.
    *   pruning line pointers and defragmenting the page.
    *   setting hint bits to tuple headers. WAL is emitted when page checksums are enabled.

### Transaction

ACID: what they are attributed to

*   Atomicity: transaction rollback and database recovery
*   Consistency: integrity constraints and triggers, such as non-NULL, check, primary key/unique/foreign key constraints
*   Isolation: MVCC and locks
*   Durability: WAL

Transaction ID (XID)

*   A transaction is assigned an XID when it first modifies data, such as in INSERT, UPDATE, DELETE, and SELET FOR SHARE/UPDATE.
    *   XID assignments are serialized with XidGen LWLock.
    *   XID assignment is usually very fast, but it might sometimes experience hiccups. It allocates and zeros a new commit log (clog) page through SLRU cache every 32K transactions. That clog page allocation could possibly flush a dirty page for page replacement.
    *   This could cause an unpredictable spike of response time.
*   Read-only transactions do not assign an XID. They are free from the LWLock contention for assigning a new XID.
*   XIDs are stored in tuple headers and visible as xmin and xmax system columns (`SELECT xmin, xmax, * FROM mytable`).
    *   xmin is the XID of a transaction that created the tuple (INSERT, UPDATE, COPY).
    *   xmax is the XID of a transaction that either:
        *   deleted the tuple (DELETE, UPDATE).
        *   locked the tuple (e.g., SELECT FOR SHARE/UPDATE)
*   Special XID values
    *   0: invalid XID
    *   1: bootstrap XID. Used by bootstrap processing during initdb.
    *   2: Frozen XID: Recent versions of PostgreSQL only use this for sequence tuples, not for tables.

MVCC: Multi-Version Concurrency Control

*   The major advantage of MVCC is "reading never blocks writing and writing never blocks reading." i.e., UPDATE/DELETE and SELECT on the same row do not block each other.
    *   Writes to the same row block each other.
    *   In the traditional lock-based concurrency control, read and write on the same row conflict.
*   How it works overall:
    *   Insert and update to a row create a new version of the row. Update leaves the old row version for other running transactions. (Multi-version)
    *   XID of the creating transaction is set to xmin field in the new row version.
    *   The new row version is only visible to its creating transaction until it commits.
    *   Once the creating transaction commits, all new subsequent transactions will be able to see the new row version. Other existing transactions continue to see the old row version. The old row version is a "dead tuple" now.
    *   Delete to a row does not remove the row version. It sets the XID of the deleting transaction to xmax field in the row version.
    *   The deleted row version is only invisible to its deleting transaction until it commits.
    *   Once the deleting transaction commits, all new subsequent transactions won't be able to see the row version. Other existing transactions continue to see the row version. The row version is a "dead tuple" now.
    *   Finally, when there are no transactions remaining that can see the dead tuple, vacuum removes it.
*   How tuple visibility works:
    *   Each transaction uses its own snapshot, commit log (clog), and the xmin and/or xmax in the target tuple header, to determine whether it can see a given row version.
    *   What snapshot is:
        *   A picture of what transactions are running at a certain point of time.
        *   You can run "`SELECT pg_current_snapshot();`" to see the snapshot of the current transaction.
        *   The snapshot's textual representation is xmin:xmax:xip_list. e.g., 10:20:10,14,15.
        *   xmin: Lowest transaction ID that was still active. All transaction IDs less than xmin are either committed and visible, or rolled back and dead.
        *   xmax: One past the highest completed transaction ID. All transaction IDs greater than or equal to xmax had not yet completed as of the time of the snapshot, and thus are invisible.
        *   xip_list: Transactions in progress at the time of the snapshot. A transaction ID that is xmin <= X < xmax and not in this list was already completed at the time of the snapshot, and thus is either visible or dead according to its commit status.
        *   In a READ COMMITTED transaction, a snapshot is taken at the beginning of every SQL statement.
        *   In a REPEATABLE READ or SERIALIZABLE transaction, a snapshot is obtained at the start of the first SQL statement and used throughout the transaction.
    *   What commit log (clog) is:
        *   An array of bits representing the transaction status.
        *   Two bits are used to indicate a transaction's outcome: IN PROGRESS, COMMITTED, ABORTED, SUBCOMMITTED.
        *   Stored in a set of files in $PGDATA/pg_xact/.
        *   Cached in memory buffers of 128 8 KB pages.
    *   Clog is consulted when the snapshot shows that the target transaction has been completed.
    *   Based on the snapshot and clog, the change by a committed transaction is visible, and that by an aborted or running transaction is invisible.
    *   The actual tuple visibility is much more complex...

Hint bit

*   Hint bits are the bits in the tuple header's infomask field that help determine tuple visibility.
*   They are for performance optimization. Not essential to data correctness.
*   They represent whether the transaction indicated by xmin or xmax was committed or aborted. There are four flag bits:
    1.  `HEAP_XMIN_COMMITTED`: xmin transaction was committed
    2.  `HEAP_XMIN_INVALID`: xmin transaction was aborted
    3.  `HEAP_XMAX_COMMITTED`: xmax transaction was committed
    4.  `HEAP_XMAX_INVALID`: xmax transaction was aborted
*   How hint bits are used:
    1.  A transaction checks the hint bits to see if the xmin and/or xmax transaction was committed or aborted.
    2.  If the hint bits are set, done.
    3.  Otherwise, examine the commit log ($PGDATA/pg_xact/), and possibly the subtransaction hierarchy ($PGDATA/pg_subtrans/) to determine the transaction outcome. This is an expensive operation.
    4.  Set the hint bits. They will be persisted to disk later.
*   Setting hint bits writes a data page, and can also write WAL if page checksums are enabled.

### Lock

A lock request can wait, even when the requested mode is compatible with held locks.

*   Q: Do you think Transaction 3 goes on to run the query?
    1.  Transaction 1: A long-running `SELECT` is still running against mytable.
    2.  Transaction 2: Run "`ALTER TABLE mytable ADD COLUMN new_col int;`". Get blocked because ALTER TABLE's Access Exclusive lock request conflicts with the Access Share lock held by Transaction 1's `SELECT`.
    3.  Transaction 3: Run a short `SELECT` query against mytable.
*   A: Transaction 3 waits until Transaction 2 completes, because Transaction 2 came earlier and is waiting.
*   Later requestors respect earlier waiters in the wait queue and do not overtake them. Otherwise, earlier requestors might wait for an unduly long time.
*   Therefore, execute even the DDL that is expected to run fast:
    *   during off-peak hours, and/or
    *   with a lock timeout. e.g., run "`SET lock_timeout = '5s';`" before the DDL. Retry the DDL if it times out.
*   This is not true for lightweight locks. In extreme cases, an Exclusive mode request on a LWLock could wait for dozens of seconds due to later Share mode requestors coming one after another.

Prepared transactions lurk holding locks

*   A prepared transaction continues to hold locks, but it does not appear in [pg_stat_activity](https://www.postgresql.org/docs/current/monitoring-stats.html#MONITORING-PG-STAT-ACTIVITY-VIEW) because it has no associated session.
*   [pg_locks](https://www.postgresql.org/docs/current/view-pg-locks.html) shows the prepared transaction as an entry having NULL pid. Check pg_prepared_xacts.

### Data integrity validation

Data checksums

*   Purpose and usage
    *   The data page of every relation, including all forks, has a 16-bit checksum in its page header.
    *   Designed to detect corruption by the I/O system (e.g., volume manager, file system, disk driver, storage firmware, storage device, etc.) Early detection prevents the propagation of corruption.
    *   Not designed to detect memory errors.
    *   Enabled at the full cluster level, either with `initdb`'s -k/--data-checksums or with `pg_checksums` while the database server is shut down. Disabled by default due to its performance overhead.
    *   Run `"SHOW data_checksums"` to know if data checksums are enabled. It returns on or off.
*   How it works
    *   The checksum is calculated from the page content and set when the data page is about to be written out to disk.
    *   Just after reading the page from disk, the checksum is verified by comparing the value set in the page header and the newly calculated value.
    *   If the verification fails, a WARNING and ERROR messages are emitted, resulting in a query failure.
    *   If the page header fails a basic sanity check before performing checksum verification, the query will fail with the same ERROR message, without the WARNING that indicates the checksum failure.

WAL CRC

*   WAL uses a 32-bit CRC in each WAL record header.
*   The CRC is set when the WAL record is put in the WAL buffer, and verified when the WAL record is read.

Utilities to detect, bypass, or repair data corruption (some could be dangerous!)

*   Additional modules
    *   [amcheck](https://www.postgresql.org/docs/current/amcheck.html): Detects logical corruption of heaps (table, sequence, materialized view) and B-tree indexes.
    *   [pg_surgery](https://www.postgresql.org/docs/current/pgsurgery.html): `heap_force_kill()` and `heap_force_freeze()` forcibly removes and freezes heap tuples respectively.
    *   [pg_visibility_map](https://www.postgresql.org/docs/current/pgvisibility.html): `pg_check_frozen()` and `pg_check_visible()` detect visibility map corruption.
*   Configuration parameters
    *   ignore_checksum_failure
    *   zero_damaged_pages
    *   ignore_system_indexes

### Backup and recovery

Backup and recovery methods

1.  File system level backup (binary format)
2.  SQL Dump with pg_dump/pg_dumpall (text format)
3.  Continuous archiving (binary format)

Characteristics of backup and recovery methods

*   SQL dump and continuous archiving can be performed online. The file system level backup requires the database server to be shut down.
*   SQL dump can selectively back up and restore individual tables. The other methods cannot back up or restore only certain individual tables or tablespaces.
*   The SQL dump will typically be smaller, because the SQL script needs to contain just the index creation command, not the index data.
*   The SQL dump can be loaded into a database of a newer major version.
*   The SQL dump can transfer a database to a different machine architecture, such as going from a 32-bit to a 64-bit server.
*   Continuous archiving can perform PITR. The database cluster can be recovered up-to-date or to a certain point of time.
*   Dumps created by pg_dump are consistent; the dump of each database is a snapshot of the database when pg_dump started. pg_dumpall calls pg_dump for each database in turn, so the database cluster-wide consistency is not guaranteed.
*   pg_dump dumps all data in a database within a single transaction, issuing many SELECT commands. That long-running transaction could:
    *   block other operations that require strong lock modes, such as ALTER TABLE, TRUNCATE, CLUSTER, REINDEX.
    *   cause table and index bloat, because vacuum cannot remove dead tuples.
*   `pg_backup_start()` and `pg_basebackup` of continuous archiving performs a checkpoint at the beginning. The user can choose the checkpoint speed between "fast" and "spread".
*   Archive recovery, as well as crash recovery, empties the content of unlogged relations. SQL dump outputs the contents of unlogged tables.
*   pg_dumpall's --no-role-passwords option uses pg_roles instead of pg_authid to dump database roles. This allows the use of pg_dumpall in restricted environments like DBaaS where users are not permitted to read pg_authid to protect passwords. The restored roles will have NULL passwords.

### Streaming replication

Architecture

*   Topology
    *   Only the entire database cluster is replicated. Partial replication is not possible.
    *   One primary server replicates to one or more standby servers.
    *   Each standby replicates from one primary.
    *   The standby can cascade changes to other standbys.
    *   The primary is unaware of the locations of standbys. The standby connects to the primary specified by primary_conninfo parameter.
        *   ex. `primary_conninfo = 'host=192.168.1.50 port=5432 user=foo password=foopass'`
*   Primary and standby versions
    *   Different major versions don't work.
    *   Different minor versions will work because the disk format is the same, but no formal support is offered. It's advised to keep primary and standby servers at the same minor version.
    *   It's safest to update the standby servers first during minor version upgrade.
*   Processes and data flow
    *   At server startup, the standby first reads and applies WAL from archive, next from $PGDATA/pg_wal/, and then launches one walreceiver, which connects to the primary and streams WAL from there. If the replication connection is terminated, it repeats this cycle at 5 second intervals, which can be configured by wal_retrieve_retry_interval.
    *   The primary spawns the walsender when it accepts the connection request from walreceiver.
    *   walsender reads and sends WAL to the walreceiver.
    *   walreceiver writes and flushes the streamed WAL to $PGDATA/pg_wal/, and notifies the startup process.
    *   A single startup process reads and applies the WAL.
    *   walreceiver periodically notifies the walsender of replication progress -- how far it has written, flushed, and applied the WAL.
    *   A cascading standby has walsenders as well as a walreceiver running.
*   Replication user
    *   The replication user needs REPLICATION role attribute.
    *   REPLICATION enables the user to read all data for replication, but not for consumption by SELECT queries.

General administration

*   The standby is read-only. Any object, including roles, cannot be created only on the standby.
*   max_wal_senders should be slightly higher than the number of standby servers, so that the standby can accept connections after a temporary unexpected disconnection while the disconnected walsender still remains.
*   Backups can be taken on the standby.
*   archive_timeout is not required to reduce the data loss window.
*   Cascading replication reduces the load on the primary.
*   WAL on the primary
    *   Without any measure, the primary does not care about the standby and remove/recycle old WAL files that the standby still need.
    *   If the standby requests WAL that has already been removed, the primary emits a message like `"ERROR: requested WAL segment 000000020000000300000041 has already been removed"`.
    *   To make the primary preserve WAL files, either use a replication slot (preferred) or set keep_wal_size (old method).
    *   max_slot_wal_keep_size caps the WAL volume preserved by the replication slot.
*   Synchronous replication
    *   The transaction hangs during its commit if no synchronous standby is available.
    *   To resume the hanging transaction, remove synchronous_standby_names setting and reload the configuration. That makes the replication asynchronous.

Causes of replication lag

*   Hardware configuration: Server, storage, and network
*   Heavy workload on the primary: The amount of WAL generated on the primary is so large that the solo startup process cannot keep up.
    *   Set wal_compression = on to reduce the amount of WAL.
*   Retrieving WAL from slow archive: The standby could not get WAL from the primary, so it has to fetch it from the WAL archive.
*   Recovery conflicts: The replay of some operations can be blocked by queries running on the standby.
    *   This is relevant when hot standby is used.
    *   Reduce max_standby_archive_delay and max_standby_streaming_delay to cancel conflicting queries and resume WAL replay early.

Hot standby

*   The ability to run read-only queries while the server is in archive recovery or standby mode.
*   To determine if the server is in hot standby, use `"SHOW in_hot_standby"` for PostgreSQL 14+, or `"SELECT pg_is_in_recovery()"` otherwise.
*   Recovery conflicts
    *   Conflicts between the WAL replay and queries on the standby.
    *   Either delay WAL replay or cancel queries.
    *   Actions on the primary that cause recovery conflicts include:
        *   Operations that take Access Exclusive locks: DDL, LOCK, file truncation by vacuum (including autovacuum)
            *   Access Exclusive lock requests are WAL-logged and replayed by the standby.
        *   Dropping a tablespace where queries put temporary files on the standby
        *   Dropping a database to which clients are connected on the standby
        *   Vacuum cleanup of dead tuples that standby transaction still can see according to their snapshots
        *   Vacuum cleanup of a page on which standby transactions have a buffer pin (e.g., the cursor is positioned on the page.)
    *   What happens upon recovery conflicts
        *   WAL application waits for at most the period specified by max_standby_archive_delay and max_standby_streaming_delay (except for the replay of DROP DATABASE and ALTER DATABASE SET TABLESPACE.)
        *   Then, conflicting sessions are terminated in the case of replaying DROP DATABASE, or conflicting queries are canceled in other cases.
        *   If an idle session holds a lock, the session is also terminated.
    *   Monitoring recovery conflicts
        *   [pg_stat_database_conflicts](https://www.postgresql.org/docs/current/monitoring-stats.html#MONITORING-PG-STAT-DATABASE-CONFLICTS-VIEW) on the standby shows the number of canceled queries due to each type of recovery conflict.
        *   "log_recovery_conflict_waits = on" logs messages that the WAL application has waited longer than deadlock_timeout and the wait finished.
            *   `LOG: recovery still waiting after 1.023 ms: recovery conflict on snapshot`
            *   `DETAIL: Conflicting processes: 1234, 1235`
            *   `LOG: recovery finished waiting after 3.234 ms: recovery conflict on snapshot`
    *   Minimizing the number of queries cancelled due to recovery conflict
        *   Avoid operations that require Access Exclusive locks. e.g., ALTER TABLE, VACUUM FULL, CLUSTER, REINDEX, TRUNCATE
        *   Disable file truncation by vacuum by setting vacuum_truncate storage parameter on the primary.
            *   ex. `ALTER TABLE some_table SET (vacuum_truncate = off);`
        *   Set hot_standby_feedback = on the standby.
            *   sends the oldest XID to the primary, reflected in pg_stat_replication.backend_xmin, which is taken into account when vacuum decides to remove a dead tuple.
            *   can incur table bloat because dead tuple removal is delayed.
            *   cannot prevent all conflicts.
        *   Adjust max_standby_streaming_delay/max_standby_archive_delay on the standby.
        *   Adjust vacuum_defer_cleanup_age on the primary.
    *   It's ideal to have separate standbys, some for high availability and others for read workloads that tolerate stale data.

Monitoring replication lag

*   [pg_stat_replication](https://www.postgresql.org/docs/current/monitoring-stats.html#MONITORING-PG-STAT-REPLICATION-VIEW)
    *   Available not only on the primary but also on the cascading standby.
    *   Large differences between pg_current_wal_lsn and the view's sent_lsn field might indicate that the primary server is under heavy load.
    *   Differences between sent_lsn and pg_last_wal_receive_lsn on the standby might indicate network delay, or that the standby is under heavy load.
*   [pg_stat_wal_receiver](https://www.postgresql.org/docs/current/monitoring-stats.html#MONITORING-PG-STAT-WAL-RECEIVER-VIEW)
    *   A large difference between pg_last_wal_replay_lsn() and the view's flushed_lsn indicates that WAL is being received faster than it can be replayed.
    *   ex. `SELECT pg_wal_lsn_diff(pg_last_wal_replay_lsn(), flushed_lsn) FROM pg_stat_wal_receiver;`
*   [pg_stat_wal](https://www.postgresql.org/docs/current/monitoring-stats.html#MONITORING-PG-STAT-WAL-VIEW)
    *   Check the amount of WAL generated for heavy write workload.
*   Storage write latency, IOPs, and throughput to check for heavy write activity.
*   [pg_stat_database_conflicts](https://www.postgresql.org/docs/current/monitoring-stats.html#MONITORING-PG-STAT-DATABASE-CONFLICTS-VIEW)

### Logical replication

Architecture

*   Uses a publish and subscribe model
    *   A publication is a collection of tables whose changes are to be replicated.
    *   A subscription represents a connection to the publisher and its publications to subscribe to.
    *   One publisher publishes one or more publications.
    *   One subscriber has one or more subscriptions.
    *   A publication can have multiple subscribers.
    *   A subscription can subscribe to multiple publications.
    *   Publications can choose to limit the changes they produce to any combination of INSERT, UPDATE, DELETE, and TRUNCATE.
    *   Publications can restrict the rows and columns to be replicated.
*   Processes and data flow
    *   Processes involved: walsender on the publisher, subscription workers (apply worker, tablesync worker) on the subscriber.
    *   walreceiver does not appear, even though some walreceiver-related parameters are used.
    1.  At the server startup on the subscriber, logical replication launcher is started unless max_logical_replication_workers is 0.
    2.  logical replication launcher starts an apply worker for each enabled subscription.
    3.  The apply worker connects to the publisher.
    4.  The apply worker launches tablesync workers for tables that have not completed initial synchronization. Those tablesync workers each connect to the publisher.
    5.  The publisher spawns a walsender for each connection request from the subscription workers.
    6.  The walsender for a tablesync worker sends the initial copy of a table to the tablesync worker. (Initial data synchronization/copy)
    7.  walsender reads WAL, decodes changes into the logical replication protocol format, and store them in the logical decoding work memory and possibly file. When a transaction commits, walsender sends its decoded changes to the subscription workers.
    8.  The subscription workers apply the received changes.

General administration

*   Major restrictions
    *   Publications can only contain tables.
    *   DDL are not replicated.
        *   Add table columns on the subscriber first, then on the publisher. Reverse the order when dropping table columns.
    *   Sequence data is not replicated.
*   Replication identity
    *   A published table must have a replica identity to replicate UPDATE and DELETE operations.
    *   Used as a key to identify rows to update or delete on the subscriber.
    *   UPDATE and DELETE fail on the publisher if the published table has no replica identity. INSERT succeeds.
    *   Can be either of the primary key (by default), unique index, or the full row.
    *   Can be configured by `ALTER TABLE REPLICA IDENTITY`.
    *   The old values of replica identity columns are WAL-logged.
*   Tuning performance
    *   max_sync_workers_per_subscription
        *   Multiple tablesync workers (one for each table) will run in parallel based on the max_sync_workers_per_subscription configuration.
        *   This may be effective to speed up initial table synchronization when there are many tables in a subscription.
    *   logical_decoding_work_mem
        *   Check [pg_stat_replication_slots](https://www.postgresql.org/docs/current/monitoring-stats.html#MONITORING-PG-STAT-REPLICATION-SLOTS-VIEW) for spilled transactions to disk. If spill_txns, spill_count, and spill_bytes are high, consider increasing this parameter value.

Replication conflicts

*   The application of incoming changes on the subscriber may fail due to constraint violation or lack of permission. This is the conflict.
*   Resolving conflicts:
    1.  Disable the subscription if it's not yet by running `"ALTER SUBSCRIPTION name DISABLE;"`. The subscription can be configured to be automatically disabled when any errors are detected by the apply worker. Run `"ALTER SUBSCRIPTION ... WITH (disable_on_error = on);"`
    2.  Look up the replication origin name and the end LSN of a conflicting transaction in the server log.
    3.  Do either of:
        *   skip the transaction from publisher by running `"ALTER SUBSCRIPTION ... SKIP (end LSN of a conflicting transaction)"` or `"SELECT pg_replication_origin_advance(rep_origin_name, next LSN of the end of a conflicting transaction)"`
        *   fix the table data on the subscriber.
    4.  Enable the subscription by running `"ALTER SUBSCRIPTION name ENABLE;"`.

Monitoring

*   Publisher
    *   [pg_stat_replication_slots](https://www.postgresql.org/docs/current/monitoring-stats.html#MONITORING-PG-STAT-REPLICATION-SLOTS-VIEW)
        *   One row per logical replication slot. Total number of transactions and amount of decoded data, and number of transactions and amount of decoded data that were spilled to disk.
    *   Other system and statistics views used for streaming replication.
*   Subscriber
    *   [pg_stat_subscription_stats](https://www.postgresql.org/docs/current/monitoring-stats.html#MONITORING-PG-STAT-SUBSCRIPTION-STATS)
        *   One row per subscription. Numbers of errors during the initial table synchronization and while applying changes.
    *   [pg_stat_subscription](https://www.postgresql.org/docs/current/monitoring-stats.html#MONITORING-PG-STAT-SUBSCRIPTION)
        *   One row per subscription worker (apply worker, tablesync worker). Table being copied, last WAL location sent/reported etc.
    *   [pg_subscription_rel](https://www.postgresql.org/docs/current/catalog-pg-subscription-rel.html)
        *   One row for each subscribed table. Status of the table to know whether the initial synchronization is in progress.

### References

PostgreSQL Documentation

Connection

WAL

Transaction

Lock

Data integrity validation

Backup and recovery

Streaming replication

Logical replication

## Security

### Authentication

Encrypt password when changing it

*   `CREATE/ROLE ... PASSWORD 'some_password'` sends and logs the specified password as is. Thus, specifying an unencrypted password is dangerous.
*   Those statements accept an encrypted password (hashed with MD5 or SCRAM).
*   psql's \password is convenient
    *   psql runs `"SHOW password_encryption"` to determine the password hash scheme (MD5 or SCRAM), hashes the supplied password, and then issues an ALTER command.
    *   The hashed password still can appear in the server log. Temporarily setting log_min_error_statement to 'PANIC' prevents that.

In-database authentication profile is very limited

*   PostgreSQL offers only password expiration by `CREATE/ROLE VALID UNTIL 'some_timestamp'`.
*   Does not provide functionality such as:
    *   Enforcing password complexity
    *   Locking out a user account when the number of failed login attempts exceeds a threshold within a certain period of time
    *   Restricting reuse of the same password before a certain number of days pass

Implementing password complexity: use either of:

Tracking failed login attempts: Do either of:

*   Search the server log for messages that include "password authentication failed" or the SQLSTATE 28P01 (invalid_password)
    *   Using SQLSTATE is better than the message text, because the message can vary depending on the server version and lc_message setting. (Add %e to log_line_prefix to emit the SQLSTATE.)
*   [Trusted Language Extensions for PostgreSQL (pg_tle)](https://github.com/aws/pg_tle)
    *   The user uses the client authentication hook and creates an extension that records and checks login failures.

### Authorization

Role privileges are inherited by default

*   In the SQL standard and other DBMSs, `SET ROLE` needs to be used to gain privileges of another role.
*   In PostgreSQL, a role automatically inherits the privileges of other roles that it is a member of. This might be surprising.
*   To approximate the SQL standard, use NOINHERIT for users and NOINHERIT for roles.

Predefined roles

*   Some roles are provided to give part of administrative privileges to non-superusers.
*   They can be given by GRANT.
*   The representative roles are:
    *   pg_monitor: can read various useful configuration settings, statistics and other system information.
    *   pg_signal_backend: can send signals to other backends to cancel a query or terminate a session.
    *   pg_read_server_files, pg_write_server_files and pg_execute_server_program: access files and run programs on the database server as the user the database runs as. e.g., these enable COPY data to/from files on the server or another program like gzip and curl.

Default privileges

*   `ALTER DEFAULT PRIVILEGES` can set the default privileges that will be automatically given to database objects created in the future.
    *   ex. `ALTER DEFAULT PRIVILEGES IN SCHEMA app_schema GRANT INSERT, UPDATE, DELETE, SELECT ON TABLES TO app_user;`
*   The target database objects are schema, table, view, sequence, function, and type.
*   Does not change the privileges of existing database objects.

### References

PostgreSQL Documentation

*   [ The PostgreSQL User Accounthttps://www.postgresql.org/docs/current/postgres-user.html]
*   [ Database Roleshttps://www.postgresql.org/docs/current/user-manag.html]
*   [ Client Authenticationhttps://www.postgresql.org/docs/current/client-authentication.html]
*   [ Privilegeshttps://www.postgresql.org/docs/current/ddl-priv.html]
*   [ Row Security Policieshttps://www.postgresql.org/docs/current/ddl-rowsecurity.html]
*   [ Encryption Optionshttps://www.postgresql.org/docs/current/encryption-options.html]
*   [ Secure TCP/IP Connections with SSLhttps://www.postgresql.org/docs/current/ssl-tcp.html]
*   [ Secure TCP/IP Connections with GSSAPI Encryptionhttps://www.postgresql.org/docs/current/gssapi-enc.html]

Authentication

Authorization

Hiding data

## Manageability

### Memory

A large result set causes out of memory on the client

*   When running SELECT, psql retrieves the entire result set and stores all rows in the client memory.
*   With the FETCH_COUNT variable like `"psql -v FETCH_COUNT=100 ..."`, psql uses a cursor and issues DECLARE, FETCH, and CLOSE to retrieve the result set piecemeal.
*   Client drivers have similar facility, such as psqlODBC's UseDeclareFetch and PgJDBC's defaultRowFetchSize connection parameter.

Common causes of server-side out of memory (OOM) issues

*   A high number of connections
    *   Even idle connections could continue to hold much memory. PostgreSQL keeps database object metadata in memory during the session. This is for performance. You can notice this by bloated CacheMemoryContext.
    *   If connection pooling is used, many connections may consume large amounts of memory over time. This is because connections are picked up randomly from the pool, used to access some relations, and released back to the pool, which results in many sessions accumulating the meta data of many relations.
*   A high value of work_mem
    *   It's advised not to set a high value to work_mem at an instance level (postgresql.conf) or a database level (ALTER DATABASE). Many sessions could allocate that amount of memory simultaneously. What's worse, each SQL statement could run such sort and/or hash operations in parallel, each of which can allocate as much memory as work_mem.
    *   For a hash-based operation, work_mem * hash_mem_multiplier bytes of work memory will be allocated at maximum.
*   Low max_locks_per_transaction
    *   Each lockable object (e.g., table, index, sequence, XID, but not row) is allocated an entry in the lock table when it's locked. The entry represents the lockable object, grantees, waiters, and granted/requested lock modes.
    *   The lock table is allocated in shared memory. Its size is fixed at server startup.
    *   The default value of max_locks_per_transaction is 64\. This means that each transaction is expected to lock 64 or fewer objects.
    *   The number of entries in the lock table is (max_connections + max_prepared_transactions + alpha) * max_locks_per_transaction.
    *   One transaction can use more than max_locks_per_transaction entries, if those are available.
    *   If each of many concurrent transactions may access more objects, say, touch hundreds or thousands of partitions, increase max_locks_per_transaction.

Cannot retrieve a large bytea value

*   For example, after successfully inserting 550 MB of bytea column value, fetching it fails with an error message like `"invalid memory aloc request size 1277232195"`.
*   Why?
    *   When the PostgreSQL server sends query results to the client, it either converts the data to text format or returns it in binary format.
    *   psql and client drivers instruct the server to use text format.
    *   [PostgreSQL uses either hex or escape format](https://www.postgresql.org/docs/current/datatype-binary.html) when it converts bytea data to text format. The default format is hex. hex and escape formats use 2 and 4 bytes respectively to represent each original byte in text format.
        *   ex. `SELECT 'abc'::bytea;` returns `\x616263`
    *   The PostgreSQL server allocates one contiguous memory area to convert each column value to text format. This allocation size is limited to 1 GB - 1\. That restriction has something to do with the handling of variable-length data types in TOAST.
    *   Due to this limitation, PostgreSQL cannot return bytea data over 500 MB in text format.

### Storage

Common causes of full storage

*   Table bloat because vacuum cannot remove dead tuples: the reasons dead tuples remain are described separately.
*   WAL accumulation: the reasons WAL volume continues to grow are described separately.
*   Server log files
    *   Excessive logging because of pgAudit, auto_explain, and other logging parameters such as log_statement and log_min_duration_statement
    *   Log rotation and purge are not configured properly: log_rotation_age, log_rotation_size, log_truncate_on_rotation
*   Temporary files are created
    *   The work_mem is small and/or the query plan is bad.
    *   A holdable cursor is kept open.
        *   e.g. `DECLARE CURSOR cur WITH HOLD FOR SELECT * FROM mytable; COMMIT;`
        *   During the commit, the result set of the holdable cursor is stored in a work memory area of size work_mem, and the content beyond work_mem is spilled to a temporary file.
    *   Check temporary file usage with:
        *   temp_files and temp_bytes of pg_stat_database
        *   log_temp_files = on, which logs the file path and size when the file is deleted
        *   query plans obtained by EXPLAIN ANALYZE or auto_explain

Storage quota

*   PostgreSQL cannot constrain storage usage except for temporary files.
*   temp_file_limit can limit the total size of temporary files used by each session at any instant. A transaction exceeding this limit will be aborted.
*   If you want to limit the size of a database, table, or WAL ($PGDATA/pg_wal/), put it in a tablespace on a file system with limited size.
    *   The tablespace of a database/table can be specified explicitly by `CREATE/ALTER DATABASE/TABLE ... TABLESPACE`, or implicitly by the default_tablespace parameter.
    *   temp_tablespaces can be used to specify where temporary files are created for temporary tables/indexes and sort/hash operations.
    *   WAL directory can be specified by initdb's --waldir option. Also, after the database cluster has been created, it can be moved outside the data directory and linked with a symbolic link.

### Logging and debugging

`FATAL: database is starting up`

*   In old major versions, this message can be output at 1 second intervals during the server startup.
*   This may look startling, but it's not an actual problem.
*   "Why? pg_ctl start" launches postmaster in the background, and tries to connect to the database at 1 second intervals. If the connection is successful, pg_ctl returns success. Otherwise, if the server is still performing recovery and unable to accept connections, the above message is reported.
*   In newer major versions, you won't see the message any more. pg_ctl does not attempt connection. Instead, postmaster writes "ready" in postmaster.pid when it can accept connections, and pg_ctl checks it.

Avoid excessive logging by restricting targets

*   Not only logging but many parameters can be configured for each user, database, or the combination of them. For example:
    *   `ALTER USER oltp_user SET log_min_duration_statement = '3s';`
    *   `ALTER DATABASE analytics_db SET log_min_duration_statement = '60s';`
    *   `ALTER USER batch_user IN DATABASE oltp_db SET log_min_duration_statement = '30s';`

Debug-logging can be enabled for a session without cluttering the server log

*   It may not be acceptable to globally set log_min_messages to DEBUG1 - DEBUG5, because that would output voluminous logs.
*   You can obtain debug messages for a particular operation only on the client like this:
    *   `export PGOPTIONS="-c client_min_messages=DEBUG5"`

`psql -d postgres -c "select 1"`

Find out what psql's backslash commands do

*   Use psql's -E/--echo-hidden option. It reveals the query issued in the background.

Deleting duplicate rows

*   The following query deletes duplicate rows, leaving the one with the minimum ctid and displays the deleted row content.
*   ctid is a system column that represents the physical location of the row version within its table: (block number, item ID). ctid can change due to UPDATE and VACUUM FULL, so it'd be probably safe to lock the table in Share or stronger mode during this operation.

 ````
WITH x AS (SELECT some_table dup, min(ctid)
    FROM        some_table
    GROUP BY 1
    HAVING count(*) > 1
)
DELETE FROM    some_table
USING     x
WHERE     (some_table) = (dup)
    AND some_table.ctid <> x.min
RETURNING some_table.*;

```` 

### Vacuum

Purposes of vacuum

*   To recover or reuse disk space occupied by updated or deleted rows.
*   To update data statistics used by the PostgreSQL query planner.
*   To update the visibility map, which speeds up index-only scans.
*   To protect against loss of very old data due to transaction ID wraparound or multixact ID wraparound.

Vacuum types

*   Concurrent (lazy, or regular) vacuum
    *   Acquires a Share Update Exclusive lock on the target relation. Does not prevent SELECT and DML commands.
    *   Keeps the original data files and modifies them. TIDs do not change.
    *   The data file shrinks only when there are more than certain number of contiguous empty blocks at the end. Unused space in the middle of the file is left for reuse.
    *   Reports its progress in the pg_stat_progress_vacuum view.
*   FULL vacuum
    *   Acquires an Access Exclusive lock on the target relation. Prevents SELECT and DML commands.
    *   Copies live tuples from the old data files to new data files, and remove old data files. Rebuilds indexes. TIDs change.
    *   The data files will be packed fully and minimal.
    *   Could uses twice the disk space: one for existing relations, and another for new ones.
    *   Always aggressively freezes tuples.
    *   The actual processing is the same as CLUSTER.
    *   Reports its progress in the pg_stat_progress_cluster view.
*   Autovacuum never runs FULL vacuum.

Main steps of vacuum

1.  Starts a transaction.
    *   When there are multiple target relations, vacuum starts and commits a transaction for each relation to release locks as soon as possible.
2.  Gets an Share Update Exclusive lock for a heap and opens it. Non-wrap-around-prevention vacuum gives up vacuuming the relation if the relation cannot get the lock, emitting the following message.
    *   `LOG: skipping of vacuum "rel_name" --- lock not available`
3.  Gets Row Exclusive locks for the indexes and opens them.
4.  Allocates the work memory to accumulate the TIDs of dead tuples.
5.  Repeats the following steps until the entire heap has been processed:
    *   Scans the heap: Accumulates dead tuple TIDs in the work memory until it gets full or the end of the heap is reached. The item IDs for the dead tuples are retained. Also, prunes and defragments each page if required, and possibly freezes live tuples.
    *   Vacuums the indexes: Delete index entries that contain dead tuple TIDs.
    *   Vacuums the heap: Reclaims the item IDs for the dead tuples. This is done here, not while scanning the heap, because the item ID cannot be freed until the index entries pointing to it have been deleted.
    *   Updates the FSM and VM during the above processing.
6.  Cleans up the indexes.
    *   Updates every index's stats in pg_class's relpages and reltuples.
    *   Closes the indexes but retains their locks until transaction end.
7.  Truncate the heap so as to return empty pages at the end of the relation to the operating system.
    *   The data file is truncated if the heap has at least the lesser of 1,000 blocks and (relation_size / 16) contiguous empty blocks at its end.
    *   Takes an Access Exclusive lock on the heap. If another transaction holds a conflicting lock, wait for at most 5 seconds. If the lock cannot be obtained, gives up truncating.
    *   Scans backwards the heap to verify that the end pages are still empty. Periodically checks if another transaction is waiting for a conflicting lock. If someone else is waiting, releases the Access Exclusive lock and gives up truncating.
8.  Updates relation stats.
    *   Updates pg_class's relpages, reltuples, relallvisible, relhasindex, relhasrules, relhastriggers, relfrozenxid, and relminmxid.
9.  Close the relation.
10.  Commits a transaction.
11.  Vacuums the relation's TOAST table.
12.  Repeats the above processing for each relation.
13.  Updates database stats.
    *   Updates pg_database.datfrozenxid to be the minimum of pg_class.relfrozenxid values, and truncates commit log in pg_xact/.
    *   Updates pg_database.datminmxid to be the minimum of pg_class.relminmxid values, and truncates MultiXact data in pg_multixact/.

Autovacuum is designed to be non-intrusive

*   Autovacuum takes a rest (sleep) every time it has done a certain amount of work. Therefore, it does not continuously consume resources.
    *   "A certain amount of work" and the sleep time can be configured by autovacuum_vacuum_cost_limit and autovacuum_vacuum_cost_delay respectively. autovacuum_vacuum_cost_delay is 2 ms by default.
*   Autovacuum skips the relation if it cannot get a lock on it due to some conflicting lock. Wrap-around-prevention autovacuum does not do this.
*   A concurrent transaction cancels non-aggressive autovacuum, if it has waited on a conflicting relation lock for deadlock_timeout seconds and found out that the lock is held by autovacuum. These messages can be seen:
    *   `ERROR: canceling autovacuum task`
    *   `DETAIL: automatic vacuum of table "mytable"`
*   Vacuum skips reading data pages if VM shows that they have only tuples visible to all transactions (all-visible bit is set in VM). Aggressive vacuum reads even such pages to freeze tuples.
*   Vacuum skips reading data pages if VM shows that they have only frozen tuples (all-frozen bit is set in VM).
*   Autovacuum performs reduced work on a data page when it cannot get an exclusive LWLock on it. Autovacuum for wrap-around does not do this.
*   Vacuum gives up truncating the relation if another transaction holds or is waiting for a lock on the target relation.

Autovacuum is not run against a relation

*   Check the following to see if that is the case.
    *   last_autovacuum and autovacuum_count columns of pg_stat_all_tables
    *   Server logs after setting log_autovacuum_min_duration to 0
*   Common reasons
    *   The relation has to be eligible for autovacuum.
        *   UPDATE/DELETE-mostly relations: updated/deleted tuples >= autovacuum_vacuum_threshold + autovacuum_vacuum_scale_factor * pg_class.reltuples
        *   INSERT-mostly relations: inserted tuples >= autovacuum_vacuum_insert_threshold + autovacuum_vacuum_insert_scale_factor * pg_class.reltuples
    *   Autovacuum workers are busy with other many and/or large relations.
    *   Some transactions continuously request or hold a conflicting relation lock for long. Non-wrap-around-prevention vacuum gives up such a relation.
    *   Autovacuum cannot vacuum temporary tables. Manual vacuum needs to be run. This might lead to XID wrap-around and database shutdown.
    *   Statistics stored in pg_stat/ were lost due to crash or archive recovery, including failover. Those statistics are always reset during recovery. Autovacuum depends on the statistics, which can be seen via pg_stat_all_tables, to determine if vacuuming is needed.

Why vacuum does not remove dead tuples

*   Slow autovacuum
*   Long-running transactions
*   Physical standbys with hot_standby_feedback = on
*   Unused replication slots
*   Orphaned prepared transactions

Reducing the risk of XID wrap-around

*   Reduce XID consumption.
    *   Each subtransaction allocates its own XID. A subtransaction is started by SAVEPOINT and PL/pgSQL's exception block (BEGIN ... EXCEPTION).
    *   Some client drivers offer statement-level rollback. It encloses each SQL statement with SAVEPOINT and RELEASE SAVEPOINT.
*   Make autovacuum run smoothly (see above).
*   Lower autovacuum_vacuum_insert_scale_factor (PostgreSQL 13+) or autovacuum_freeze_max_age so that autovacuum processes the table more frequently.
*   Schedule regular VACUUM FREEZE runs.

Configuration to speed up autovacuum

*   Lower autovacuum_vacuum_threshold, autovacuum_vacuum_scale_factor, autovacuum_vacuum_insert_threshold, autovacuum_vacuum_insert_scale_factor for large tables.
*   Decrease autovacuum_naptime
    *   Even 1s is practical if the write workload is heavy and the host has many CPU cores.
*   Increase autovacuum_max_workers
    *   Effective when there are many relations. Each relation is handled by only one autovacuum worker.
    *   Increase autovacuum_vacuum_cost_limit as well. Otherwise, each autovacuum worker would sleep more frequently, because cost limits are shared among all active autovacuum workers.
*   Increase maintenance_work_mem/autovacuum_work_mem
    *   The work memory stores an array of dead tuple TIDs. A TID is (block no, item no), which is 6 bytes.
    *   Setting a large value reduces the number of index scans.
    *   The maximum allocated size is 1 GB - 1 no matter how large the parameter values are.
    *   Does not always allocate the specified size. The actual size is large enough to accommodate all possible TIDs, so it will be small for small tables.
    *   For a table with no index, only less than 2 KB is allocated. Vacuum only accumulates TIDs for one table block because it does not need to scan indexes.
*   Increase vacuum_buffer_usage_limit
    *   Vacuum uses 256 KB of ring buffers by default to cache data pages, so that it does not evict pages that are likely to be used by applications.
    *   Vacuum also benefits from caching pages: heap pages are read twice, and index pages may possibly be read more than once.
    *   Setting this to 0 allows vacuum to use shared buffers without limit.
*   Decrease autovacuum_vacuum_cost_delay, increase autovacuum_vacuum_cost_limit
    *   Setting autovacuum_vacuum_cost_delay to 0, which keeps autovacuum running like the manual vacuum.
*   Partition a large table so that multiple autovacuum workers can work on its partitions concurrently.
*   Delete unnecessary indexes.
    *   Autovacuum processes indexes one at a time. (Manual vacuum can process them in parallel with its PARALLEL option.)

### Upgrade

Characteristics of versions

*   Major version
    *   Contains new features and incompatibilities.
    *   Released once a year.
    *   Sensitive bug fixes are only incorporated into the latest major version. "Sensitive" includes the fixes that could lead to incompatibility, adverse effects such as unstability and security, or require lots of code changes not worth the benefit.
    *   The upgrade can skip intervening major versions. e.g., version 11 can be upgraded to 16 without going through 12 to 15.
    *   Always requires careful planning and testing to deal with incompatible changes.
*   Minor version
    *   Contains only frequently-encountered bugs, security issues, and data corruption problems to reduce the risk associated with upgrading.
    *   Running the latest minor version is always recommended. The community considers not upgrading to be riskier than upgrading.
    *   Released at least once every three months, the second Thursday of February, May, August, November. Additional minor versions may be released to address urgent issues.
    *   The upgrade can skip intervening minor versions.
    *   Does not normally require a dump and restore; you can stop the database server, install the updated binaries, and restart the server.
    *   Additional manual steps may be required for some minor versions to remedy the bad effects of fixed bugs, such as rebuilding affected indexes. See the section "Migration to Version <major>.<minor>" in the release note.

Major upgrade methods

1.  pg_dumpall/pg_dump and psql/pg_restore: easy, long downtime
2.  pg_upgrade: relatively easy, shorter downtime
3.  Logical replication: complex setup and operation, minimal downtime

Overview of pg_upgrade

*   Upgrades a database cluster to a later major version without dump/restore of user data.
*   Not an in-place upgrade: migrates data from an old database cluster to a new database cluster freshly created with initdb.
*   The basic idea is that because the relation data storage format rarely changes and only the layout of system catalogs change, pg_upgrade just dumps and restores the database schema and uses relation data files as-is.
*   Upgrade from 9.2 and later is supported.
*   Downgrade is not possible.
*   Does not migrate database statistics in pg_statistic. The user needs to run ANALYZE in every database after pg_upgrade completes.

Main steps of pg_upgrade

1.  Creates output directory for log and intermediate files.
2.  Checks that the major version of the target cluster is newer, and that the old and new clusters are binary-compatible by comparing information in pg_control.
3.  Gets the list of databases and relations (table, index, TOAST table, matview) of the old cluster.
4.  Gets the list of library names that contain C-language functions.
5.  Performs various checks to find blockers of upgrade, such as the inability to connect to databases and the presence of prepared transactions.
6.  Creates a dump of global objects by running `pg_dumpall --globals-only`.
7.  Creates dumps of each database by running `pg_dump --schema-only`. This is parallelized by spawning one process or thread for each database when --jobs is specified.
8.  Checks the previously extracted loadable libraries with C-language functions exist in the new cluster by running `LOAD`.
9.  Copies commit log files in pg_xact/ and MultiXact files in pg_multixact/ from the old cluster to the new cluster.
10.  Sets next XID and MultiXact ID for the new cluster to take over the old cluster.
11.  Restores global objects in the new cluster by running `psql`.
12.  Restores database schemas in the new cluster by running `pg_restore`. This is parallelized by spawning one process or thread for each database when --jobs is specified.
13.  Gets the list of databases and relations (table, index, TOAST table, matview) of the new cluster.
14.  Links or copies user relation files from the old cluster to the new cluster. This is parallelized by spawning one process or thread for each tablespace when --jobs is specified.
15.  Sets next OID for the new cluster.
16.  Creates a script to delete the old cluster (delete_old_cluster.sh). This script removes the data directory and tablespace version directories.
17.  Reports extensions that should be updated and creates update_extensions.sql. This script contains a list of ALTER EXTENSION ... UPDATE commands.

Log files for troubleshooting pg_upgrade

*   Stored in $NEWPGDATA/pg_upgrade_output.d/<timestamp>/
*   Removed when pg_upgrade completes successfully.
*   Files:
    *   pg_upgrade_server.log: The postgres server logs. Specified as pg_ctl's -l.
    *   pg_upgrade_dump_<DB-OID>.log: Logs of pg_dump and pg_restore.
    *   pg_upgrade_utility.log: Logs of miscellaneous commands run by pg_upgrade, such as psql, pg_resetwal. This includes pg_dumpall/psql to dump and restore global objects.
    *   pg_upgrade_internal.log: Other pg_upgrade logs.
    *   loadable_libraries.txt: List of C-language function libraries that exist in the old cluster but are not found in the new cluster.

### References

PostgreSQL Documentation

Memory

Storage

Internationalization and localization

Logging

Vacuum

Partitioning

Upgrade

Miscellaneous tips

## Application development

### Data type

Numeric

*   For exact calculation and/or numbers with many digits, choose numeric.
*   For small storage space and faster calculation, choose integer types (smallint, int, bigint) and floating-point types (real, double precision, float).
*   decimal type is an alias for numeric. psql's \d and pg_dump output decimal columns as numeric instead of decimal.

Timestamp

*   timestamp without time zone ignores the TimeZone parameter. The value is stored and returned as-is.
*   timestamp with time zone honors the explicit time zone in the input value or otherwise the TimeZone parameter. The input value is converted to UTC, and the output value is converted from the stored value, according to the time zone in effect.

Binary

*   Available methods to store binary data
    *   bytea data type
    *   Large object: Use filesystem-like open/close/read/write interface, data is stored in pg_largeobject, the user table column contains an OID value that points to a row in pg_largeobject.
    *   External file: The application manages data in filesystems, or object storage and stores the file path in a table character column.
*   How to choose:
    *   Need transactional (ACID) properties? -> bytea, large object
    *   Handle 1 GB or larger column value? -> large object, external file
    *   Need random and/or piecemeal access? -> large object, external file
    *   Want best performance with 100 MB or larger column values? -> external file

Tips for using large objects

*   **Do not use large objects.** They can be problematic. Use bytea columns or external file storage such as an OS file system and object storage.
*   Removing lots of LOBs
    *   Trying to remove many large objects within a single transaction, e.g., `"SELECT lo_unlink(lo_oid) FROM mytable;"` can fail with the following message:
        *   `ERROR: out of shared memory`
        *   `HINT: You might need to increase max_locks_per_transaction.`
    *   Cause: When a large object is deleted, it is locked with Access Exclusive mode. Therefore, as many entries as the deleted LOBs are required in the lock table.
    *   Solutions: Do either or both of:
        *   Increase max_locks_per_transaction. The database server has to be restarted.
        *   Delete LOBs in chunks, e.g., 100 LOBs per transaction.
*   Dealing with orphaned LOBs
    *   An orphaned LOB is a large object whose OID does not appear in any oid or lo data column of the database.
    *   Such an orphaned LOB would result if the application fails to delete it by calling lo_unlink() when it deletes the associated table row.
        *   Solutions: Do both of:
        *   Use [vacuumlo](https://www.postgresql.org/docs/current/vacuumlo.html) to delete orphaned LOBs.
        *   Use [extension](https://www.postgresql.org/docs/current/lo.htmllo) and set a trigger on the LOB column. It automatically calls lo_unlink() when the table row containing the LOB OID is updated or deleted.

### Sequence

There is no gapless sequence

*   A sequence produces gaps when:
    *   The transaction rolls back: Because nextval() and setval() calls are never rolled back, allocated sequence values are not reclaimed.
    *   Cached values are unused: If the caching is enabled for a sequence, nextval() preallocates the specified number of values and caches them in the session's local memory. Subsequent nextval() calls fetch values from the cache until the cache is empty, and then preallocate some values again. So, if the session ends without using all the cached values, those will be gaps.
    *   The server crashes: Even with a NO CACHE sequence, you can see a gap in these steps: nextval() -> crash recovery -> nextval(). For performance, PostgreSQL does not WAL-log every fetching of a value from a sequence. nextval() WAL-logs a value 32 numbers ahead of the current value, and the next 32 calls to nextval() don’t WAL-log anything. As a result, some numbers appear to be skipped.

### References

PostgreSQL Documentation

Data type

Large object

Pagination

Sequence

## Scalability and performance

### Many connections

Want to handle many concurrent clients? Then do:

*   Set up connection pooling on each application server as well as on a central server.
*   Limit max_connections and the actual number of connections to a few times the number of CPU cores, or at most a few hundreds.
*   Performance tends to drop above this limit, mainly because of:
    *   high memory usage by client backends, possibly leading to swapping.
    *   CPU context switches
    *   CPU cache line contention
    *   locks, particularly spinlocks: if one process holds a spinlock and other processes comes to the same protected section, those latecomers will wait spinning on the lock and continues to consume CPU.
    *   processing PostgreSQL internal data structures: some data structures and its processing depends on the number of connections; creating a snapshot stands out here
*   Even idle connections are not innocent. They contribute to high resource usage.

### Detecting problems

Increase track_activity_query_size for ORMs (Object Relational Mappers)

*   Some views such as pg_stat_activity and pg_stat_statements show query strings.
*   Because those query strings are stored in fixed-size shared memory, the length of each such query string is fixed, which is track_activity_query_size. Longer queries are cut off at this limit.
*   Hibernate or some other ORMs produce very long queries. It may be useful to set track_activity_query_size to 32 KB or so.

Utilize [plprofiler](https://github.com/bigsql/plprofiler) to diagnose the bottlenecks of PL/pgSQL functions and procedures

*   This is an extension that creates an HTML report showing the runtimes of each step of the function and procedure, total runtimes of routines called from there, as well as the execution time of each routine.

### Logging

Server logging could block but not show waits

*   The logging collector (logger) is the sole process to write logs to server log files.
*   Every backend process writes logs to its standard error, which is connected through a Unix pipe to the read endpoint of the logger. The logger reads messages from the pipe and writes them to files.
*   If the pipe gets full, the backend could block when writing to it. This happens when the logger is behind due to overwhelming amount of logging, for example, when some combination of pgAudit, auto_explain, and log_min_duration_statement is used and many concurrent sessions are running short queries.
*   The block on the pipe is not treated as a wait event (possibly by mistake), so the backend appears to be consuming CPU.

### Import and export

`ALTER TABLE SET UNLOGGED/LOGGED` is heavy

*   This rewrites the entire table into new data files and WAL-logs those writes.
*   Hence, you cannot use it for efficient data loading - switching the table to UNLOGGED, load data into the table, and setting it back to LOGGED.

Queries or the first vacuum are slow after loading data with COPY

*   This is because those commands have to set hint bits for rows they want to see. Setting hint bits modifies shared buffers and WAL-logs the changes if data checksums are enabled. These could generate massive writes.
*   `COPY (FREEZE)` comes to the rescue. FREEZE option freezes the loaded rows and set their hint bits.
*   The table must have been created or truncated in the current subtransaction. This is to prevent other transactions from seeing the frozen rows before the COPY transaction commits.

### Foreign Data Wrapper (FDW)

Speeding up queries via [postgres_fdw](https://www.postgresql.org/docs/current/postgres-fdw.html)

*   Run ANALYZE manually on foreign tables
    *   autovacuum does not execute ANALYZE on foreign tables. Hence, the local statistics may get stale and lead to poor query plans.
*   Enable use_remote_estimate for long-running queries
    *   i.e., `ALTER FOREIGN SERVER/TABLE ... OPTIONS (use_remote_estimate 'true');`
    *   This makes postgres_fdw issue EXPLAIN to perform the cost estimate on the remote server.
    *   The query planning time will get longer due to the round-trip for EXPLAIN. So, this may not be worth the cost for short queries. You can use different foreign servers/tables with different settings for OLTP, batch, and analytics workloads.
*   Increase fetch_size
    *   i.e., `ALTER FOREIGN SERVER/TABLE ... OPTIONS (fetch_size '1000');`
    *   postgres_fdw uses a cursor to fetch rows from a foreign table. fetch_size determines the number of rows to fetch at a time. The default is 100.
    *   If the network latency is high, reducing the round-trips by increasing this setting may help. Be aware that higher values require more memory to store fetched rows.
*   Increase batch_size
    *   i.e., `ALTER FOREIGN SERVER/TABLE ... OPTIONS (batch_size '1000');`
    *   By default, postgres_fdw inserts one row at a time into a foreign table during multi-row inserts (`INSERT ... SELECT, INSERT ... VALUES (row1), (row2),..., COPY FROM`).
    *   Raising this setting will dramatically increase the throughput, particularly where the network latency is high.
*   List the extension names in extensions parameter that have compatible behavior on both the local and remote servers
    *   i.e., `ALTER FOREIGN SERVER ... OPTIONS (extensions 'extension1,extension2');`
    *   The immutable functions and operators in those extensions are considered to bring the same result on the local and remote servers. As a result, execution of them will be shipped to the remote server.
    *   This is particularly beneficial when those functions and operators are used in the WHERE clause. Those filters will be executed on the remote server and thus fewer rows are transfered.

### Full text search

Full-text search queries got much slower after inserting many new documents

*   When inserting data into an GIN index that has fastupdate enabled, the new index entries are not put into the index main structure. Instead, they are placed in the index's pending-list whose size is set by gin_pending_list_limit. Later, when the pending-list area becomes full, those pending-list entries are moved to the main index structure.
*   This is for good performance, because inserting one document involves many insertions into the main index, depending on the number of words in the document.
*   Full-text search queries scan the pending-list before the main index structure. Therefore, they are slow if the pending-list contains many pending entries.
*   Vacuum, including autovacuum, also moves the pending-list entries into the main index. So, the full-text search query will be faster after vacuum.
*   It is advised to tune autovacuum so that it runs reasonably frequently after inserting or updating documents.
*   The number of pending-list pages and tuples can be seen with this query (the pgstatginindex is in pgstattuple extension):
    *   `SELECT * FROM pgstatginindex('some_gin_index');`

### Utility

Fast random sampling of table rows

*   The traditional method is slow, because it scans and sorts the entire table.
    *   `SELECT * FROM mytable ORDER BY random() LIMIT 1;`
*   Using TABLESAMPLE clause returns rows very quickly almost independently of the table size.
    *   TABLESAMPLE fetches a sample portion of a table. Some built-in sampling methods are provided.
    *   Also, The sampling method can be customized by adding an extension. For example, [tsm_system_rows](https://www.postgresql.org/docs/current/tsm-system-rows.html) retrieves a specified number of random rows:
        *   `CREATE EXTENSION tsm_system_rows;`
        *   `SELECT * FROM mytable TABLESAMPLE SYSTEM_ROWS(1);`
    *   SYSTEM_ROWS picks up a random block in the table's data file, and then fetchs rows sequentially in it. If more rows are necessary, additional blocks will be chosen.

### Memory

Use huge pages

*   Set huge_pages = on.
    *   This will reduce memory usage dramatically because the [page table](https://en.wikipedia.org/wiki/Page_table) gets smaller.
    *   Also, improved performance can be expected thanks to the reduction of CPU's TLB cache misses.
*   "on" should be preferred for huge_pages to "try", considering the reduced memory usage and improved performance as a part of stable operation.
    *   When huge_pages is set to "on", and the OS cannot allocate enough huge pages, PostgreSQL refuses to start emitting the following messages:
        *   `FATAL: could not map anonymous shared memory: Cannot allocate memory`
        *   `HINT: This error usually means that PostgreSQL's request for a shared memory segment exceeded available memory, swap space, or huge pages. To reduce the request size (currently 1234567890 bytes), reduce PostgreSQL's shared memory usage, perhaps by reducing shared_buffers or max_connections.`
    *   In this case, reboot the OS or perform failover.

Tips for shared buffers

*   Avoid disk writes by client backends.
    *   If there is no free shared buffer when the server process wants a new page, it has to evict a used buffer. If the evicted page is dirty, the server process needs to write the page to disk. This adds to the response time.
    *   This undesirable situation can be detected by checking that the buffers_backend in [pg_stat_bgwriter](https://www.postgresql.org/docs/current/monitoring-stats.html#MONITORING-PG-STAT-BGWRITER-VIEW) is high. If buffers_backend_fsync is also high, the situation is worse.
    *   To alleviate this:
        *   Make more free buffers: increase shared_buffers.
        *   Make more clean buffers: increase bgwriter_lru_multiplier so that the background writer writes dirty buffers more aggressively. If the maxwritten_clean of pg_stat_bgwriter rises frequently, try increasing bgwriter_lru_maxpages.
*   Large shared buffers may be counterproductive.
    *   The benefits of shared buffers is diminished on a host with high-performance storage.
    *   That's because PostgreSQL uses the OS's filesystem cache: the data is cached in the filesystem cache as well as in shared buffers (double buffering).
    *   Therefore, start with 25% of RAM for shared buffers, and then increase it up to around 40% as long as you can see some improvement.
    *   However, some benchmark demonstrated that 64 GB or more can do harm.
*   Leverage [pg_prewarm](https://www.postgresql.org/docs/current/pgprewarm.html) to quickly regain performance after failover.
    *   After the database server restart or failover, the contents of shared buffers is empty or can be quite different from what was before the failover. Thus, application response times get worse until the shared buffers are warmed up.
    *   Add pg_prewarm in shared_preload_libraries, and set pg_prewarm.autoprewarm to on.
    *   This launches the autoprewarm worker, which periodically saves in a file the list of relation and block numbers cached in shared buffers. At server startup, pg_prewarm worker reads the file to refill shared buffers.
*   `"SELECT * FROM some_table;"` does not necessarily cache the entire table.
    *   You may want to do this for performance test or application warmup, but it doesn't work. Also, it does not cache indexes at all.
    *   If the size of a relation is larger than a quarter of shared buffers, its sequential scan only uses 256 KB of shared buffers.
    *   The idea behind this is that a page that has been touched only by such a scan is unlikely to be needed again soon, so PostgreSQL tries to prevent such large sequential scans from evicting many useful pages out of the shared buffers.
    *   Likewise, bulk writes, e.g., COPY FROM and CREATE TABLE AS SELECT, use only 16 MB of shared buffers.
    *   To cache the entire relation, run `SELECT pg_prewarm('relation_name')`. This works for indexes as well.

Tips for local memory

*   Setting enough work_mem requires try and error.
    *   Unfortunately, there is no easy way to estimate a work_mem setting to avoid disk spilling.
    *   The temporary file that log_temp_files shows is not sufficient. Additional overhead for buffering temporary data in memory must be included.
    *   One way to estimate work_mem is to multiply the width and number of plan rows that are sorted or hashed, found in the query plan. Add some extra for overhead, say, further multiply it by 1.1 or so.
    *   If parallel query is used, divide the result by (number of parallel workers used + 1). "+1" is for the parallel leader process.
    *   Run EXPLAIN ANALYZE to see if external file is used. Try increasing work_mem until the use of external file disappears.
*   effective_cache_size does not allocate any memory.
    *   This is only used to estimate the costs of index scans. The planner assumes this amount of memory is available for caching query data.
    *   A higher value makes it more likely index scans will be used, a lower value makes it more likely sequential scans will be used.

### Network

Watch out for network latency when running lots of short SQL commands

*   Did your batch application, which issues a lot of small SQL statements in succession, get many times slower when you migrated it to a different environment?
*   That may be because the network latency is higher. Check to see if the network communication is slow.
    *   Measure the round-trip time of a simple SQL, e.g.,
    *   Check if the wait events ClientRead and ClientWrite are increasing.

### Cursor

*   DECLARE CURSOR is fast. It creates a query plan, but does not calculate the result set. FETCH starts the calculation.
*   A cursor query is planned differently from a non-cursor query. You can see different query plans for the same SELECT statement.
    *   Non-cursor queries are optimized for total runtime. The optimizer assumes that the client will consume the entire result set.
        *   A sequential scan and sort is more likely to be chosen because the index scan is considered expensive.
    *   Cursor queries are optimized for the runtime of startup and initial data retrieval. The optimizer assumes that the client will fetch only a fraction of the result set.
        *   The optimizer goes for an index scan to speed up the creation of the first 10% of the data.
        *   The "10%" can be configured with cursor_tuple_fraction parameter.

### Lock

Utilize fast-path locks for high performance

*   If lots of concurrent short transactions each touch many relations, the lwlocks to protect the lock table can become a contention bottleneck. That contention is visible as the LWLock:LockManager wait event.
*   Although the lock table is divided into 16 partitions and they are covered by different lwlocks, hundreds of concurrent transactions can lead to waits on those lwlocks.
*   Fast-path locks come to the rescue:
    *   Weak locks (Access Share, Row Share, and Row Exclusive modes) are taken using the fast-path lock mechanism. It doesn't use the lock table. Instead, those locks are recorded in the per-backend area in shared memory.
    *   SELECT and DMLs take those weak locks, so they don't suffer from the lock manager lwlock contention.
*   However, fast-path locks cannot be used if either of the following is true:
    *   The transaction already has 16 fast-path relation locks. The per-backend recording area is limited to 16 entries. Queries that access tables with many partitions and indexes, or join many tables will lose.
    *   Some transaction tries to acquire a strong lock (Share, ShareRowExclusive, Exclusive, and AccessExclusiveLock modes).
        *   Existing fast-path locks on the same relation are transferred to the lock table.
        *   If someone has or requesting a strong lock, subsequent transactions which acquire a lock on a different relation may not be able to use the fast-path lock. This is because the presence of strong locks are managed using an array of 1024 integer counters, which are in effect a 1024-way partitioning of the lock space. If the requested weak lock is to be managed in the same partition as an existing strong lock, it cannot be fast-path.
*   The fast-path lock shows up in [pg_locks](https://www.postgresql.org/docs/current/view-pg-locks.html) as the fastpath column being true.

### HOT

Take advantage of HOT (Heap-Only Tuple)

*   HOT speeds up UPDATEs.
*   What's wrong if HOT is not used?
    *   Indexes will be bigger, because each row version has an index entry in every index. Index scans using those indexes will be slower, too.
    *   WAL volume will be larger and update will be slower, because the update of any column inserts new entries into all indexes.
*   For HOT to work, both of the following conditions must be met:
    *   The block containing the updated row has enough free space to accommodate the new row version.
    *   The update does not modify any indexed column.
*   Then, what should I do?
    *   Set fillfactor on the table to make room for new row versions.
        *   e.g., `CREATE TABLE mytable ... WITH (fillfactor = 90);`, `ALTER TABLE mytable SET (fillfactor = 90);`
        *   Lower fillfactor makes the table bigger, which results in shared buffer misses and longer sequential scans.
        *   Maybe you should start with fillfactor = 90, and lower the setting if HOT is not working well.
    *   Drop unnecessary indexes.
*   How do I know if HOT is working?

### Table layout

For best storage efficiency and performance, declare table columns from largest fixed length types (e.g., bigint, timestamp) to smallest fixed length types (e.g., smallint, bool), then variable length types (e.g., numeric, text, bytea)

*   Storage efficiency comes from the data alignment requirements.
    *   For example, bigint is aligned on 8 byte boundary, while bool is aligned on 1 byte boundary.
    *   In the following example, the former of the following returns 48, and the latter returns 39.
        *   `SELECT pg_column_size(ROW('true'::bool, '1'::bigint, '1'::smallint, '1'::int));`
        *   `SELECT pg_column_size(ROW('1'::bigint, '1'::int, '1'::smallint, 'true'::bool));`
    *   The alignment requirements can be seen with: `SELECT typalign, typname FROM pg_type ORDER BY 1, 2;`
*   Better performance comes from the aforementioned smaller data size, and direct column access.
    *   If fixed-length columns are placed in front of the row, PostgreSQL can calculate and cache the positions of fixed-length columns in the row. So, a requested fixed-length column data of any row can be accessed directly using its offset.
    *   Once a variable-length column appears, the positions of subsequent columns need to be calculated for each row, by adding up its columns' actual lengths. As a result, access to columns at the end of the row will be slow.

Use functions returning a composite type in FROM clause instead of SELECT column list

*   Suppose sample_func()'s return type is a composite type `(a int, b int, c int)`.
*   Bad: `SELECT (sample_func()).*;`
*   Good: `SELECT * FROM sample_func();`
*   In the bad case, `"(sample_func()).*"` is expanded to `"(sample_func()).a"`, `"(sample_func()).b"`, `"(sample_func()).c"`. Thus, the function is called three times.

TOAST (The Oversized-Attribute Storage Technique)

*   This is a mechanism to store large values of up to 1 GB - 1.
*   A tuple cannot span multiple pages. Then, how is a column value stored that is larger than the page size (commonly 8 KB)?
*   Large column values of TOAST-able data types are compressed and/or broken up into chunks. Each chunk is stored as a separate row in the table's associated TOAST table. The chunk size is chosen so that four chunk rows will fit on a page. That is about 2,000 bytes for 8 KB page size.
*   A TOAST-able data type is the one which has a variable-length (varlena) representation. That is, a 1 or 4 byte varlena header followed by the column value. char(n) seems like fixed-length, but it has a varlena format.
*   TOAST table
    *   Each table has 0 or 1 TOAST table and TOAST index.
    *   The TOAST table and its index are created in CREATE/ALTER TABLE if needed.
    *   The TOAST table is pg_toast.pg_toast_<main_table_OID>.
    *   The TOAST index is pg_toast.pg_toast_<main_table_OID>_index.
    *   The TOAST table's OID is stored in the table's pg_class.reltoastrelid.
    *   The TOAST index's OID is stored in the TOAST table's pg_class.reltoastidxid.
    *   Every TOAST table has these columns:
        *   chunk_id OID: an OID identifying the particular TOASTed value
        *   chunk_seq int: a sequence number for the chunk within its value
        *   chunk_data bytea: the actual data of the chunk
        *   Primary key (chunk_id, chunk_seq)
*   How TOAST works
    *   It's triggered only when a row value to be stored in a table is wider than 2 KB (when the page size is 8 KB).
    *   Compresses and/or moves column values to the TOAST table, until the row value is shorter than 2 KB (when the page size is 8 KB) or no more gains can be had. This 2 KB threshold can be adjusted for each table using the storage parameter toast_tuple_target in CREATE/ALTER TABLE.
    *   Stores the TOASTed value's chunk_id in the main table's column. This is called a TOAST pointer.
*   The value storage strategy - whether it should be compressed or moved to the TOAST table - can be chosen from four options using `ALTER TABLE ALTER COLUMN column_name SET STORAGE { PLAIN | EXTERNAL | EXTENDED | MAIN }`
*   The compression method can be chosen between pglz and lz4\. It can be set for each column by using the COMPRESSION column option in CREATE/ALTER TABLE or otherwise the default_toast_compression parameter.
*   Insertion of a TOASTed value could become unsurprisingly slow.
    *   This tends to be seen when the target table already has millions of TOASTed values, particularly after inserting lots of rows in succession.
    *   Why?
        *   Each TOASTed value is identified by an OID.
        *   OID is an unsigned 4-byte value, which is generated from a cluster-wide counter that wraps around every 4 billion values. Therefore, a single table cannot have more than 2^32 (4 billion) TOASTed values.
        *   When inserting a TOASTed value, PostgreSQL generates a new OID for it, checks if an existing TOASTed value in the target table already uses the same OID. If it's used, PostgreSQL generates the next OID and perform the check again. This is repeated until a free OID is found.
        *   If successive OIDs are used in the target table, this retry takes a long time.
*   The remedy is to partition the table. Each partition has its own TOAST table. Thus, the likelihood of duplicate OID in each partition is reduced.

### Transaction

Killed (dead) index tuples can give mysterious query speedup

*   If you encounter varying execution times for the same execution plan of the same query, that may be thanks to killed index tuples.
*   Whenever an index scan fetches a heap tuple only to find that it is dead, it marks the index tuple as killed (dead). Then future index scans will ignore it. This avoids its index key comparison as well as its heap tuple fetch.

Subtransactions can be harmful

*   A subtransaction is a part of a transaction that can be rolled back without rolling back the main (top-level) transaction.
*   A subtransaction is started explicitly by a SAVEPOINT command, or implicitly when you enter a block with an EXCEPTION clause in PL/pgSQL.
*   Some client drivers provide an option to start and end a subtransaction for every SQL statement, such as PgJDBC's connection parameter "autosave". Watch out for their default values.
*   Each subtransaction allocates its own XID when it performs an operation that needs an XID, such as modifying data or locking a row.
*   The tuple header's xmin and xmax fields record the XID of the subtransaction that updated it. For checking tuple visibility, a transaction that sees the xmin/xmax needs to know whether the main transaction, not the subtransaction, has ended.
*   How to know the main transaction of a subtransaction:
    *   When a subtransaction assigns its XID, it records its direct parent's XID in $PGDATA/pg_subtrans/.
    *   The structure of pg_subtrans is an array of XIDS. For example, the parent XID of XID 100 is stored in the 101st element of the array. The array is divided into 8 KB pages.
    *   The pg_subtrans data is cached in a memory area of 32 pages. The area is managed by SLRU (simple least-recently used) buffers. So, The cache can contain 32 pages * 8 KB / 4 = 65,536 transactions.
    *   Therefore, to get the main transaction's XID, as many entries as the subtransaction nesting depth need to be traversed.
*   Is pg_subtrans always examined for tuple visibility?
    *   No. A snapshot stores not only main transactions' XIDs but also subtransactions' XIDs. If the checker's snapshot contains all subtransactions, it can get the job done without consulting pg_subtrans.
    *   However, that's not always the case. Each backend can have at most 64 subtransaction XIDs in its ProcArray entry in shared memory. If the main transaction has more than 64 subtransactions, its ProcArray entry is marked overflowed.
    *   When creating a snapshot, the ProcArray entries of all running transactions are scanned to collect the XIDs of main and sub transactions. If any entry is marked overflowed, the snapshot is marked suboverflowed.
    *   A suboverflowed snapshot does not contain all data required to determine visibility, so the tuple's xmin/xmax must be traced back to their top-level transaction XID using pg_subtrans.
*   So, what's the problem?
    *   The readers of pg_subtrans contend for lwlocks to protect the SLRU buffers with the writers, who register their parents' XID. The reader and writer takes Share and Exclusive mode locks respectively.
    *   The pg_subtrans cache is not so big. Under many concurrent subtransactions, disk I/O arise.
*   How can I know the possibility of this happening?
    *   The wait events LWLock:SubtransBuffer, LWLock:SubtransSLRU, IO:SLRURead, and IO:SLRUWrite keep growing.
    *   [pg_stat_slru](https://www.postgresql.org/docs/current/monitoring-stats.html#MONITORING-PG-STAT-SLRU-VIEW) shows increasing blks_read and blks_hit in its row for Subtrans. (PostgreSQL 13+)
    *   pg_stat_get_backend_subxact(backend_id) returns subxact_count and subxact_overflow. (PostgreSQL 16+)

MultiXact can harm performance under the hood

*   What is MultiXact?
    *   A mechanism to record the XIDs of multiple lockers on a tuple. (Multi-transaction)
    *   The xmax field in the tuple header records the XID that locks the tuple.
    *   Then, what happens when multiple transactions acquire locks on the same tuple?
        *   For example, the first transaction with XID 100 runs `SELECT ... FOR SHARE`. The xmax becomes 100.
        *   Next, the second transaction with XID 101 runs the same `SELECT ... FOR SHARE` on the same tuple. Then, a new MultiXact ID, say 1, is allocated and set to the xmax field.
        *   The mapping from MultiXact ID 1 to the actual lockers' XIDS (100, 101) is added in $PGDATA/pg_multixact/.
*   The foreign key constraint is implemented as a constraint trigger that executes `"SELECT ... FOR KEY SHARE"`. Therefore, MultiXact may be used without your knowledge.
*   What could be the problem?
    *   Like pg_subtrans, pg_multixact is cached through the SLRU. So, it can suffer from the lwlock contention and disk I/O.
    *   When an XID is added as a new member of an existing MultiXact ID, a new MultiXact ID is allocated and existing member XIDs are copied to a new location. In the above example, when XID 102 joins MultiXact ID 1 with members (100, 101), MultiXact 2 is newly allocated, (100, 101) are copied there, and 102 is added. If many transactions lock the same row concurrently, this copy gets heavier.
*   How can I know the possibility of this happening?
    *   The wait events LWLock:MultiXact*, IO:SLRURead, and IO:SLRUWrite keep growing.
    *   [pg_stat_slru](https://www.postgresql.org/docs/current/monitoring-stats.html#MONITORING-PG-STAT-SLRU-VIEW) shows increasing blks_read and blks_hit in its rows for MultiXactOffset and MultiXactMember. (PostgreSQL 13+)
    *   pg_get_multixact_members('<MultiXact ID>') returns a set of member XIDs and their lock modes.

### WAL and checkpoint

Overview of checkpoint

*   A processing to synchronize data both in memory and on storage by flushing unwritten (=dirty) cached data.
*   When is it performed?
    *   The time specified by checkpoint_timeout has passed since the last checkpoint.
    *   A certain amount of WAL has accumulateed, which is based on max_wal_size.
    *   At the start of a base backup (pg_basebackup, pg_backup_start()).
    *   Shutting down the database instance.
    *   Completing any form of recovery.
    *   Other miscellaneous required timings such as CREATE DATABASE, so that data files can be copied/moved without going through shared buffers.
*   The checkpoint caused by checkpoint_timeout is called a scheduled checkpoint, while others are called a requested checkpoint.
*   When finishing a checkpoint, old WAL segment files are removed or recycled as new WAL segment files for future reuse, based on min_wal_size.
    *   Here, "old" means "no longer necessary for crash recovery because all the changes in those WAL segments have been persisted to data files."
    *   However, old WAL segment files are kept until they are archived and no longer needed by wal_keep_size or any replication slots.
*   Checkpoint is intrusive because:
    *   Storage I/O contention, for both data and WAL.
    *   Buffer content lock lwlock contention: While the checkpointer is flushing a shared buffer with its buffer content lock held in Share mode, a transaction that modifies the same buffer, which requires an Exclusive lock, needs to wait for the lwlock to be released.
    *   WAL volume increase due to full page writes.
*   What is full page writes?
    *   During the first modification to each data page after a checkpoint, the entire page content is WAL-logged instead of just the change. This is necessary to recover a torn page during recovery.
    *   A torn page can result if the host crashes while PostgreSQL is writing a page. Because the atomic unit of I/O is usually smaller (say, 512 byte disk sector) than the PostgreSQL page size (commonly 8 KB), it could be possible that part of a page is new and the other is old.

Reducing the impact of checkpoints

*   Monitor the frequency of checkpoints
    *   The number of scheduled and requested checkpoints can be seen by checkpoints_timed and checkpoints_req respectively in [pg_stat_bgwriter](https://www.postgresql.org/docs/current/monitoring-stats.html#MONITORING-PG-STAT-BGWRITER-VIEW).
        *   The vast majority of checkpoints should be scheduled rather than requested. Scheduled checkpoints allow the load to be evenly spread throughout the normal operation of the system. Frequent requested checkpoints are likely to cause variations in performance.
    *   The server log shows the following messages, if the elapsed time between two successive checkpoints is shorter than checkpoint_warning and the newer one is requested by WAL accumulation.
        *   `LOG: checkpoints are occurring too frequently (8 seconds apart)`
        *   `HINT: Consider increasing the configuration parameter "max_wal_size".`
*   Lower the frequency of checkpoints by increasing max_wal_size and/or checkpoint_timeout.
    *   Note that this can increase the amount of time needed for crash recovery.
*   Set wal_compression to on. This reduces the WAL for full page writes.
*   Increase min_wal_size. This reduces the need for transactions to create new WAL segment files.

### Index

Disadvantages of indexes

*   Indexes consume disk space.
*   Larger disk space increases the size and duration of physical backups.
*   Indexes slow down INSERT/DELETE/COPY statements because they always have to modify all indexes.
*   Indexes prevent HOT updates. HOT works only for modifications to non-indexed columns.

Benefits of indexes you might not notice

*   B-tree indexes can speed up the max() and min() aggregates. They can just read the index entries at the end of the index.
*   Indexes on expressions also gather statistics on the calculated values of the expression.
    *   ex. `CREATE INDEX myindex1 ON mytable ((col1 + col2 * 3));`
    *   You can see the statistics of indexed expressions. For example, in the above case, the statistics appear in pg_stats as tablename=myindex1 and attname=expr.
    *   The statistics target can be set for indexed expressions. e.g., `ALTER INDEX index_name ALTER COLUMN expr SET STATISTICS 1000;`
*   Indexes on foreign keys speed up constraint processing.
    *   ex. `CREATE TABLE orders (..., product_id int REFERENCES products ON CASCADE DELETE);`
    *   You can see the time taken for the constraint cascade processing with EXPLAIN ANALYZE and auto_explain. Foreign key constraints are implemented using triggers internally.
        *   ex. `EXPLAIN ANALYZE DELETE FROM products WHERE product_id = 2;`
        *   ... `Trigger for constraint orders_product_id_fkey: time=0.322 calls=1`

Making an index-only scan work

*   Use EXPLAIN ANALYZE to see how many times the index-only scan had to read the heap. For example, it shows something like "Heap Fetches: 0". 0 is the best.
*   Make autovacuum more aggressive or run VACUUM to update the visibility map. That would reduce the heap fetches.

### Query planning

Pitfalls of ANALYZE

*   Autovacuum does not run ANALYZE on temporary tables or foreign tables. Manually ANALYZE them.
*   The query plan can change after ANALYZE even when the table content hasn't changed.
    *   ANALYZE takes a random sample of the table contents (300 x default_statistics_target rows). Hence, the collected statistics can vary depending on which rows are read.
    *   To avoid or reduce this query plan variance, do either of:
        *   Fix the query plan using third-party software like pg_hint_plan.
        *   Raise the amount of statistics collected by ANALYZE, i.e., `ALTER TABLE ... ALTER COLUMN ... SET STATISTICS`. The more rows are used, the less the statistics fluctuation would be. However, this will make the ANALYZE and query planning slower because more statistics are written or read.
        *   Set the table's storage parameters autovacuum_analyze_threshold and autovacuum_analyze_scale_factor to large values, so that autovacuum won't practically ANALYZE it. Then, do manual ANALYZE if needed.

Use of a set-returning function could lead to a poor query plan

*   This is likely to be observed when the function is used to filter rows in WHERE clause or join.
*   That's because the planner does not have reasonably accurate information about selectivity. Thus, its cost estimate would be inaccurate.
*   CREATE/ALTER FUNCTION can set a fixed cost and the number of rows it returns. The planner support function given by the SUPPORT clause, which needs to be written in C, can change the cost and rows dynamically.
    *   `CREATE FUNCTION ... RETURNS {SETOF ... | TABLE(...)} COST execution_cost ROWS result_rows SUPPORT support_function`

Custom plan and generic plan

*   PREPARE performs parse, analysis, and rewrite to generate a prepared statement.
    *   ex. `PREPARE stmt(int) AS SELECT * FROM mytable WHERE col = $1;`
*   EXECUTE makes a query plan and execute it.
*   A query plan that takes specific parameter values into account is the best. Such plans are called a custom plan. On the other hand, a query plan that doesn't consider parameter values is called a generic plan.
*   You can tell a custom plan from a generic plan by the presence of a placeholder. For instance,
    *   Custom plan: `Filter: (col = 123)`
    *   Generic plan: `Filter: (col = $1)`
*   But planning is costly. If the generic plan is good enough, PostgreSQL uses it to avoid making custom plans.
    *   PostgreSQL uses a custom plan for the first five executions of a prepared statement.
    *   On the sixth execution, a generic plan is generated, and its cost is compared with the average cost of the past five executions.
    *   If the cost of the generic plan is cheaper, it continues to adopt it. Custom plans won't be considered.
    *   Otherwise, a new custom plan is created and used. On subsequent executions, the cost of the generic plan is compared with the average cost of all past executions of custom plans, and whichever is cheaper is chosen.
*   You can force a generic or custom plan by setting plan_cache_mode to force_generic_plan or force_custom_plan respectively. This might be necessary to force custom plans, if the cost estimate of the generic plan is underestimated.

### References

PostgreSQL Documentation

Many connections

Detecting problems

Memory

Storage

Network

Table layout

SQL tricks

Transaction

HOT

WAL and checkpoint

Lock

Index

Query planning

Join

Logging

Parallel query

Import and export

Foreign Data Wrapper (FDW)

Trigger

Full text search

Utility