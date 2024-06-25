<!--yml

类别：未分类

日期：2024-05-27 14:54:02

-->

# 操作备忘单 - PostgreSQL wiki

> 来源：[`wiki.postgresql.org/wiki/Operations_cheat_sheet`](https://wiki.postgresql.org/wiki/Operations_cheat_sheet)

## 介绍

本页面旨在从 PostgreSQL 社区的智慧中学习。

参与 PostgreSQL 社区的人员，无论是组织还是个人，都在博客、维基和网站上发布了大量有用的信息。然而，它们分散在各处，找到您正在寻找的信息将不容易。

因此，本页面已经编译了一系列链接到编辑者认为信息丰富的文章，这些文章来自注册在 Planet PostgreSQL 上的数百个博客站点，还有 PostgreSQL wiki 和网站。这些文章中选取了一些值得注意的主题，在这里进行组织和总结，以供介绍之用。请随意添加您认为对他人有帮助的文章链接。除链接之外，还很感谢您添加对它们的摘要。

**备注**

+   不要期望从这些文章中提取出所有内容！强烈建议直接深入原始文章。

+   某些配置参数、函数、统计视图和系统表/视图可能仅在某些主要版本之后才可用。请参阅 PostgreSQL 文档以获取可用性信息。

## 架构

### 客户端-服务器架构

+   服务器

    +   数据库服务器：postgres

    +   服务器应用程序：例如 initdb、pg_ctl、pg_upgrade

+   客户端

    +   客户端接口：例如 libpq、ECPG、pgJDBC、psqlODBC、Npgsql

    +   客户端应用程序：例如 psql、pgbench、pg_dump、pg_restore

+   前端/后端协议

    +   前端=客户端，后端=服务器

    +   基于消息的协议，用于在 TCP/IP 和 Unix 域套接字上前端和后端之间的通信

    +   PostgreSQL 7.4 自 2003 年以来的当前版本为 3.0

+   客户端和服务器之间的兼容性

    +   psql 最适用于与相同或更旧主要版本的服务器。它也可以与新主要版本的服务器一起使用，尽管反斜杠命令可能会失败。

    +   pg_dump 可以从比其自身旧的服务器中备份（支持 9.2 版本之前的服务器）。它无法从新主要版本的服务器中备份。

    +   驱动程序与服务器版本无关：始终使用最新的驱动程序版本

    +   例如，pgJDBC 支持 8.2+ 服务器，Npgsql 经过支持的 5 个服务器主要版本测试。

### 逻辑数据库结构

+   数据库集群是数据库、角色和表空间的集合

+   初始化后，数据库集群最初包含一些数据库：

    +   template1：除非指定了另一个模板数据库，否则新数据库将从此克隆

    +   template0：模板 1 的原始内容的原始副本

    +   postgres：实用程序和用户使用的默认数据库

+   每个数据库都包含自己的系统目录，其中存储了本地数据库对象的元数据。

+   数据库集群包含一些共享的系统目录，其中存储了集群范围全局对象的元数据。

    +   共享系统目录可以从每个数据库内部访问

    +   获取共享系统目录的查询：`SELECT relname FROM pg_class WHERE relisshared AND relkind = 'r';`

    +   例如 pg_authid、pg_database、pg_tablespace

+   表空间

    +   pg_global ($PGDATA/global/)：存储共享系统目录

    +   pg_default ($PGDATA/base/)：存储 template0、template1、postgres。其他数据库的默认表空间。

    +   用户表空间：使用 `CREATE TABLESPACE name LOCATION 'dir_path';` 创建

### 数据库对象层次结构

+   数据库

    +   存取方法

    +   Cast

    +   事件触发器

    +   扩展

    +   外部数据包装器

    +   外部服务器

    +   过程语言

    +   发布

    +   行级安全策略（名称必须与表的任何其他策略的名称不同）

    +   规则（名称必须与同一表的任何其他规则的名称不同）

    +   模式

        +   聚合函数

        +   整理

        +   转换

        +   数据类型

        +   域

        +   扩展统计信息

        +   外部表

        +   函数

        +   索引

        +   材料化视图

        +   操作符

        +   操作符类

        +   操作符族

        +   程序

        +   序列

        +   表

        +   文本搜索配置

        +   文本搜索字典

        +   文本搜索解析器

        +   文本搜索模板

        +   触发器（继承其表的模式）

        +   视图

    +   订阅

    +   转换

    +   用户映射

+   角色

+   表空间

### 对象标识符（OID）

+   OID 在 PostgreSQL 内部用作各种系统表的主键。

    +   例如 `SELECT oid, relname FROM pg_class WHERE relname = 'mytable';`

+   类型 OID 表示一个 OID。

+   oid 是一个无符号的四字节整数。

+   一个 OID 是从单个集群范围的计数器分配的，因此它不足以提供数据库范围的唯一性。

    +   pg_depend 和 pg_shdepend 中的两个 OID（classid 和 objid）标识特定对象。

### 物理数据库结构

目录

+   数据目录（$PGDATA）

    +   base/: 包含每个数据库子目录的子目录

    +   global/: 包含集群范围表的子目录，如 pg_database

    +   pg_multixact/: 包含多事务状态数据的子目录（用于共享行锁）

    +   pg_subtrans/: 包含子事务状态数据的子目录

    +   pg_tblspc/: 包含指向表空间的符号链接的子目录

    +   pg_wal/: 包含 WAL（预写式日志）文件的子目录

    +   pg_xact/: 包含事务提交状态数据的子目录

+   配置文件目录（可选）

+   表空间目录（可选）

+   WAL 目录（可选）

数据目录中的文件

+   配置文件（postgresql.conf、pg_hba.conf、pg_ident.conf）：可以存储在其他目录中

+   控制文件（global/pg_control）：存储控制信息，如集群状态、检查点日志位置、下一个 OID、下一个 XID

+   常规关系数据文件

    +   关系是一组元组：表、索引、序列、材料化视图等。

    +   每个关系都有自己的一组文件。

    +   每个文件由 8 KB 块组成。

    +   惰性分配：新的堆表文件包含 0 个块，而新的 B 树索引文件包含 1 个块。

    +   有一些类型的数据文件（分叉）：主要、FSM、VM、初始化

    +   主分支（`base/<database_OID>/<relation_filenode_no>`）

        +   例如，`"SELECT pg_relation_filepath('mytable');"` 返回 `base/17354/32185`，其中 17354 是数据库的 OID，32185 是 mytable 的文件节点编号。

        +   存储元组数据。

    +   FSM（空闲空间图）分叉（`base/<database_OID>/<relation_filenode_no>_fsm`）

        +   跟踪关系中的空闲空间。

        +   条目组织成一棵树，每个叶节点条目存储一个关系块中的空闲空间。

        +   [pg_freespacemap](https://www.postgresql.org/docs/current/pgfreespacemap.html) 和 [pageinspect](https://www.postgresql.org/docs/current/pageinspect.html) 可用于检查其内容。

    +   VM（可见性图）分叉（`base/<database_OID>/<relation_filenode_no>_vm`）

        +   跟踪：

            +   页面仅包含已知对所有活动事务可见的元组。

            +   页面仅包含已冻结元组的页面

        +   每个堆关系都有一个可见性图；索引没有一个。

        +   每个堆页存储两位：

            +   所有可见位：如果设置，则页面不包含需要清理的元组。还可由仅索引扫描使用，以仅使用索引元组来回答查询。

            +   所有冻结位：如果设置，则页面上的所有元组都已冻结，因此清理程序可以跳过页面。

        +   [pg_visibility](https://www.postgresql.org/docs/current/pgvisibility.html) 可用于检查其内容。

    +   初始化分叉（base/<database_OID>/<relation_filenode_no>_init）

        +   每个未记录的表和索引都有一个初始化分叉。

        +   内容为空：表是 0 块，索引是 1 块。

        +   未记录关系在恢复期间被重置：初始化分叉被复制到主分叉，其他分叉被擦除。

+   临时关系数据文件

    +   `base/<database_OID>/tBBB_FFF`

        +   BBB 是创建文件的后端的后端 ID，FFF 是文件节点编号。

        +   例如 `base/5/t3_16450`

    +   有主、FSM 和 VM 分叉，但没有初始化分叉。

+   大关系被划分为 1 GB 段文件。

    +   例如，`12345, 12345.1, 12345.2, ...`

页面（= 块）

+   每页为 8 KB。在构建 PostgreSQL 时可配置。

+   关系具有相同的格式。

+   内容在内存和存储中相同。

+   每页存储多个称为项的数据值。在表中，一个项是一行；在索引中，一个项是一个索引条目。

+   [pageinspect](https://www.postgresql.org/docs/current/pageinspect.html) 可用于检查内容。

+   页面布局：

    1.  页面头：24 字节

    1.  指向实际项的项目标识符数组：每个条目是一个 (偏移量，长度) 对。每项 4 字节。

    1.  空闲空间

    1.  项

    1.  特殊空间：表为 0 字节，索引类型（btree、GIN、GiST 等）的不同字节。

表行

+   [pageinspect](https://www.postgresql.org/docs/current/pageinspect.html) 可用于检查内容。

+   行布局

    1.  头部：23 字节

    1.  空位图（可选）：每列 1 位

    1.  用户数据：行的列

### 实例

实例是一组服务器端进程，它们的本地内存和共享内存。

进程

+   单线程：postmaster 为每个客户端连接启动单个后端进程。因此，每个 SQL 执行仅使用一个 CPU 核心。并行查询、索引构建、VACUUM 等可以通过运行多个服务器进程利用多个 CPU 核心。

+   postmaster：所有服务器进程的父进程。控制服务器的启动和关闭。创建共享内存和信号量。启动其他服务器进程并清理死掉的进程。打开并侦听 TCP 端口和/或 Unix 域套接字，接受连接请求，并生成客户端后端以将连接请求传递给它们。

+   （客户端）后端：代表客户端会话并处理其请求，即执行 SQL 命令。

+   后台进程

    +   记录器：通过管道捕获来自其他进程的所有 stderr 输出，并将其写入日志文件。

    +   检查点处理程序：处理所有检查点。

    +   后台写入者：定期唤醒并将脏共享缓冲区写出，以便其他进程在需要释放共享缓冲区以读入另一页时不必写入它们。

    +   启动：执行崩溃和时间点恢复。一旦恢复完成，就会结束。

    +   统计收集器：通过 UDP 套接字从其他进程接收消息，累积关于服务器活动的统计信息，并将其写入文件。可以使用 pg_stat... 视图查看统计信息。此进程在 PostgreSQL 15 中已消失。

    +   WAL 缓冲区写入者：定期唤醒并将 WAL 缓冲区写入，以减少其他进程必须写入的 WAL 量。还确保来自异步提交事务的提交 WAL 记录的写入。

    +   存档程序：存档 WAL 文件。

    +   自动清理启动器：当自动清理启用时始终运行。安排自动清理工作者运行。

    +   自动清理工作者：根据启动器确定连接到数据库，检查系统目录以选择表，并对其进行清理。

    +   并行工作者：执行并行查询计划的一部分。

    +   WAL 接收者：在备用服务器上运行。从 walsender 接收 WAL，将其存储在磁盘上，并告诉启动过程基于其继续恢复。

    +   walsender：在主服务器上运行。将 WAL 发送到单个 walreceiver。

    +   逻辑复制启动器：在订阅者上运行。协调逻辑复制工作者的运行。

    +   逻辑复制工作者：在订阅者上运行。每个订阅的应用程序工作者从发布者的 walsender 接收逻辑更改并应用它们。一个或多个表同步工作者为每个表执行初始表复制。

+   后台工作者：运行系统提供的或用户提供的代码。例如，用于并行查询和逻辑复制。

内存

+   共享内存

    +   共享缓冲区：存储数据文件（主要、FSM 和 VM 分叉）的缓存副本。

    +   WAL 缓冲区：事务在将其写入磁盘之前将 WAL 记录放在这里。

    +   其他各种区域：一个大的共享内存段被划分为用于特定用途的区域。

    +   可以使用[pg_shmem_allocations](https://www.postgresql.org/docs/current/view-pg-shmem-allocations.html)查看区域的分配情况。

+   本地内存

    +   工作内存：为排序和哈希等查询操作分配。通过 work_mem 和 hash_mem_multiplier 参数配置。

    +   维护工作内存：为维护操作分配，例如 VACUUM、CREATE INDEX 和 ALTER TABLE。通过 maintenance_work_mem 参数配置。

    +   临时缓冲区：用于缓存临时表块。通过 temp_buffers 参数配置。

    +   其他各种区域：为特定用途（例如来自客户端的消息、事务、查询计划、执行状态）分配内存上下文。每个会话可能有数百个内存上下文。

    +   可以使用[pg_backend_memory_contexts](https://www.postgresql.org/docs/current/view-pg-backend-memory-contexts.html)视图查看当前会话的内存上下文的分配和使用情况，并使用函数`pg_log_backend_memory_contexts(backend_pid)`查看其他会话的情况。

### 读取和写入数据库数据

+   读：

    +   首先，在共享缓冲区中搜索包含目标块的缓冲区。如果找到，它将返回给请求者。

    +   否则，从空闲缓冲区列表中分配一个缓冲区，并将目标块从数据文件读入缓冲区中。

    +   如果没有空闲缓冲区，则驱逐并使用已用缓冲区。如果是脏的，则将缓冲区写出到磁盘。

+   写

    +   查找目标共享缓冲区，修改其内容，并将更改写入 WAL 缓冲区。

    +   修改事务将其 WAL 记录从 WAL 缓冲区写入磁盘，包括提交 WAL 记录。

    +   修改的脏共享缓冲区由后台写入程序、检查点程序或任何其他进程刷新到磁盘。这与事务完成是异步的。

+   任何后端都可以读取和写入共享缓冲区、WAL 缓冲区、数据和 WAL 文件。与其他一些 DBMS 不同，写操作不是由特定的后台进程执行的。

+   数据库数据文件一次读取和写入一个块。没有多块 I/O。

+   一些操作绕过共享缓冲区：在索引创建期间写入索引、CREATE DATABASE、ALTER TABLE ... SET TABLESPACE

### 查询处理

1.  客户端连接到数据库，向服务器发送查询（SQL 命令），然后接收结果。

1.  解析器首先检查查询的正确语法。然后，解释查询的语义，以了解引用了哪些表、视图、函数、数据类型等。

1.  重写系统（重写器）根据系统目录 pg_rewrite 中存储的规则转换查询。一个例子是视图：访问视图的查询被重写以使用基表。

1.  计划器/优化器创建查询计划。

1.  执行器执行查询计划，并将结果集返回给客户端。

注释

+   每个会话都在与单个数据库的连接上运行。因此，它无法访问不同数据库中的表。但是，一个会话可以连接到另一个数据库并通过像 postgres_fdw 这样的外部数据包装器创建另一个会话，并在那里访问表。例如，一个 SQL 命令可以在本地数据库上连接一个表，而在远程数据库上连接另一个表。

+   每个 SQL 命令基本上只使用一个 CPU 核心。并行查询和一些实用命令（如 CREATE INDEX 和 VACUUM）可以通过运行后台工作进程使用多个 CPU 核心。

### 参考资料

PostgreSQL 文档

其他资源

## 可靠性和可用性

### 连接

故障排除连接时要检查的内容

+   数据库服务器是否可达？

    +   `telnet <host> <port>`，或

    +   `nc -zv <host> <port>`

    +   使用 `traceroute`（Unix/Linux）或 `tracert`（Windows），指定主机和中间路由器允许的协议。

+   主机和端口是否正确？

+   服务器是否正在运行？

+   服务器端防火墙是否允许通过该端口通信？

+   客户端防火墙是否允许与服务器端口通信？

+   pg_hba.conf 文件中是否有允许 SSL/非 SSL、客户端主机、数据库和用户组合的条目？

+   是否配置了 listen_addresses 参数以允许通过所需的 IP 地址（包括 IPv4 和/或 IPv6）进行连接？

+   数据库、用户名和密码是否正确？

+   用户是否有连接到数据库的权限？

    +   使用 psql 的 `\l` 或 pg_database.datacl 来检查权限

+   数据库服务器是否具有足够的 CPU 和内存资源？

+   是否达到了最大连接限制？

    +   max_connections 和 superuser_reserved_connections 参数（在实例级别）

    +   CREATE/ALTER DATABASE CONNECTION LIMIT（在数据库级别）

    +   CREATE/ALTER ROLE CONNECTION LIMIT（在用户级别）

连接终止和查询取消

+   当连接关闭时，任何未完成的事务都将被回滚。

+   终止连接（`pg_terminate_backend()`）和取消查询（`pg_cancel_backend()`）并不总是有效。例如，在后端进程运行于不可中断的部分时，它们就不起作用，比如等待获取轻量级锁、针对网络存储设备的读/写系统调用以及没有取消点的循环。

+   在适当的级别设置 statement_timeout（语句、用户、数据库、实例）。不建议在广泛级别设置短超时，因为它会取消意图长时间运行的查询。

+   适当设置客户端超时。

+   适当设置服务器端超时。

    +   tcp_keepalives_idle、tcp_keepalives_interval、tcp_keepalives_count

        +   TCP keep-alive 在 TCP 连接空闲时工作。当套接字连接正在建立或已发送某些数据并等待其 ACK 时，它不起作用。

        +   有效超时时间是 `tcp_keepalives_idle + tcp_keepalives_interval * tcp_keepalives_count`。

    +   tcp_user_timeout

        +   设置 TCP 重传超时。在 Linux 上相对较新的功能。

        +   当 TCP keep-alive 无效时，提供帮助。即，当套接字连接正在建立或已发送某些数据并等待其 ACK 时。

        +   与 TCP keep-alive 一起使用时会引起混淆，因为此时 TCP keep-alive 超时。将 tcp_user_timeout 设置为 tcp_keepalives_idle + tcp_keepalives_interval * tcp_keepalives_count 可以确保安全。

    +   authentication_timeout

    +   idle_in_transaction_session_timeout

    +   idle_session_timeout

    +   client_connection_check_interval

连接故障转移

+   客户端驱动程序允许在连接字符串中指定多个主机。

    +   连接超时应用于连接字符串中的每个主机。因此，在列表中运行的主机之前有许多失败的主机，如果存在，则建立连接可能需要意外长的时间。

+   故障转移后恢复会话状态：会话变量、预处理语句、临时表、可保持游标（使用 DECLARE CURSOR WITH HOLD 创建）、咨询锁、会话用户（使用 SET SESSION AUTHORIZATION 设置）、当前用户（使用 SET ROLE 设置）。

+   谨慎处理事务重试：当事务提交期间发生数据库服务器故障转移时，无法确定事务是已提交还是已回滚。

    +   `pg_xact_status( xid8 )` 可用于确定事务结果。

### WAL

WAL 的作用是什么

+   崩溃恢复、存档恢复（时间点恢复：PITR）和复制

+   更新无论事务是否已提交都会重新执行。

+   与其他流行的 DBMS 不同，没有撤消日志（前置图像）或操作。

    +   因此，事务回滚和崩溃恢复很快。

    +   中止事务的更改保留在内存和磁盘上，但由于 MVCC，它们对其他事务是不可见的。

WAL 结构

+   一系列 8 KB 块。

+   每个块可以包含多个 WAL 记录。此外，每个 WAL 记录可以跨越多个块。

+   内容在内存和存储上都相同。

+   WAL 缓冲区是内存中的一系列连续块。它以循环方式使用。

+   存储上的 WAL 被分割为 WAL 段文件。每个 WAL 段文件默认为 16 MB，可使用 `initdb` 的 `--wal-segsize=size` 进行配置。

写入 WAL

+   在内存中，修改共享缓冲区中的数据页，然后将更改写入 WAL 缓冲区。

+   WAL 缓冲区始终按顺序写入 WAL 文件（无随机写）。

+   在将共享缓冲区中的脏数据页写出到磁盘之前，首先写入到最新的影响数据页的所有 WAL 记录。这条规则就是 WAL（预写日志）。

    +   每个数据页在其页头中都有一个 WAL 记录的位置（LSN），表示对其的最新更新。这就是页 LSN。

    +   LSN（日志序列号）：一个无符号的 8 字节整数。它表示 WAL 段、块和块中的偏移量。

+   如果写入 WAL 文件失败，则实例将崩溃并显示 PANIC 消息。

+   WAL 体积可能会因为以下原因而超过 max_wal_size：

    +   像使用 COPY 加载数据这样的大量写入

    +   无法存档 WAL 文件

    +   wal_keep_size 的大值

    +   一个未使用的复制槽

+   当：SELECT 可以修改数据页并写入 WAL 时：

    +   获取行锁，例如，SELECT FOR UPDATE。它们在元组头中设置 xmax，并且可能更新 MultiXact 数据结构。

    +   修剪行指针并整理页面。

    +   设置提示位到元组头。启用页面校验和时，会发出 WAL。

### 事务

ACID：它们归因于什么

+   原子性：事务回滚和数据库恢复

+   一致性：完整性约束和触发器，例如非 NULL、检查、主键/唯一/外键约束

+   隔离：MVCC 和锁

+   持久性：WAL

事务 ID（XID）

+   当事务首次修改数据时，例如在 INSERT、UPDATE、DELETE 和 SELET FOR SHARE/UPDATE 中，会为其分配 XID。

    +   XID 分配与 XidGen LWLock 串行化。

    +   XID 分配通常非常快，但有时可能会出现故障。它每 32K 个事务分配并清零一个新的提交日志（clog）页通过 SLRU 缓存。该 clog 页分配可能会刷新一个脏页以进行页面替换。

    +   这可能导致响应时间的不可预测的波动。

+   只读事务不分配 XID。它们不受 LWLock 分配新 XID 的竞争的限制。

+   XIDs 存储在元组头中，并作为 xmin 和 xmax 系统列可见（`SELECT xmin, xmax, * FROM mytable`）。

    +   xmin 是创建元组（INSERT、UPDATE、COPY）的事务的 XID。

    +   xmax 是一个事务的 XID，它可能：

        +   删除了元组（DELETE、UPDATE）。

        +   锁定了元组（例如，SELECT FOR SHARE/UPDATE）

+   特殊 XID 值

    +   0：无效 XID

    +   1：引导 XID。在 initdb 期间的引导处理中由引导使用。

    +   2：冻结 XID：最近的 PostgreSQL 版本仅对序列元组使用此值，而不用于表。

MVCC：多版本并发控制

+   MVCC 的主要优势是“读不阻塞写，写不阻塞读”。即，对同一行的 UPDATE/DELETE 和 SELECT 不会互相阻塞。

    +   写入相同行的阻塞彼此。

    +   在传统的基于锁的并发控制中，对同一行的读写会发生冲突。

+   整体运作方式：

    +   对行进行插入和更新会创建行的新版本。更新会为其他正在运行的事务保留旧的行版本。（多版本）

    +   创建事务的 XID 设置为新行版本中的 xmin 字段。

    +   新行版本只对其创建事务可见，直到其提交。

    +   一旦创建事务提交，所有后续的新事务都将能够看到新的行版本。其他现有事务继续看到旧的行版本。旧的行版本现在是一个“死元组”。

    +   删除行不会移除行版本。它会将删除事务的 XID 设置为行版本中的 xmax 字段。

    +   被删除的行版本在其删除事务提交之前对其删除事务是不可见的。

    +   一旦删除事务提交，所有新的后续事务将无法看到行版本。其他现有事务继续看到行版本。现在行版本是一个“死元组”。

    +   最后，当没有剩余的事务能够看到死元组时，vacuum 会将其移除。

+   元组可见性的工作原理：

    +   每个事务使用自己的快照、提交日志（clog）、以及目标元组头中的 xmin 和/或 xmax，以确定它是否可以看到给定的行版本。

    +   快照是什么：

        +   一个显示某一时间点运行的事务的图像。

        +   你可以运行"`SELECT pg_current_snapshot();`"来查看当前事务的快照。

        +   快照的文本表示形式是 xmin:xmax:xip_list。例如，10:20:10,14,15。

        +   xmin：仍处于活动状态的最低事务 ID。所有小于 xmin 的事务 ID 要么已提交且可见，要么已回滚且已死。

        +   xmax：最高已完成的事务 ID 的后一个。所有大于或等于 xmax 的事务 ID 在快照生成时尚未完成，因此是不可见的。

        +   xip_list：在快照生成时正在进行的事务。一个事务 ID，满足 xmin <= X < xmax 且不在此列表中，已在快照生成时完成，因此根据其提交状态，它可能是可见的或死的。

        +   在读提交事务中，每个 SQL 语句的开始都会获取一个快照。

        +   在可重复读或串行化事务中，快照在第一个 SQL 语句开始时获取，并在整个事务期间使用。

    +   提交日志（clog）是什么：

        +   一个表示事务状态的位数组。

        +   两个位用于指示事务的结果：进行中、已提交、已中止、子已提交。

        +   存储在$PGDATA/pg_xact/目录下的一组文件中。

        +   缓存在 128 个 8 KB 页面的内存缓冲区中。

    +   当快照显示目标事务已完成时，会查询 clog。

    +   基于快照和 clog，已提交事务的更改是可见的，而已中止或正在运行的事务的更改是不可见的。

    +   实际的元组可见性要复杂得多...

**提示位**

+   提示位是元组头的 infomask 字段中帮助确定元组可见性的位。

+   它们用于性能优化。对数据正确性不是必需的。

+   它们表示由 xmin 或 xmax 指示的事务是已提交还是已中止。有四个标志位：

    1.  `HEAP_XMIN_COMMITTED`: xmin 事务已提交

    1.  `HEAP_XMIN_INVALID`: xmin 事务已中止

    1.  `HEAP_XMAX_COMMITTED`: xmax 事务已提交

    1.  `HEAP_XMAX_INVALID`: xmax 事务已中止

+   提示位的使用方式：

    1.  事务检查提示位以查看 xmin 和/或 xmax 事务是否已提交或已中止。

    1.  如果设置了提示位，完成。

    1.  否则，检查提交日志（$PGDATA/pg_xact/），可能还要检查子事务层次结构（$PGDATA/pg_subtrans/）以确定事务结果。这是一项昂贵的操作。

    1.  设置提示位。稍后将它们持久化到磁盘上。

+   设置提示位会写入一个数据页面，并且如果启用了页面校验和，则也会写入 WAL。

### 锁

锁定请求可以等待，即使请求的模式与持有的锁兼容。

+   Q：你认为事务 3 是否会继续运行查询？

    1.  事务 1：针对 mytable 运行的长时间`SELECT`查询仍在运行。

    1.  事务 2：运行 "`ALTER TABLE mytable ADD COLUMN new_col int;`"。被阻塞，因为 ALTER TABLE 的 Access Exclusive 锁请求与事务 1 的`SELECT`持有的 Access Share 锁冲突。

    1.  事务 3：针对 mytable 运行一个简短的`SELECT`查询。

+   A：事务 3 将等待，直到事务 2 完成，因为事务 2 较早，并且正在等待。

+   后续请求者尊重等待队列中的早期请求者，并且不会超越它们。否则，早期请求者可能会等待过长时间。

+   因此，即使是预计会快速运行的 DDL 也要执行：

    +   在非高峰时段，和/或者

    +   带有锁定超时。例如，在 DDL 之前运行 "`SET lock_timeout = '5s';`"。如果超时，请重试 DDL。

+   对于轻量级锁来说，这并不正确。在极端情况下，对 LWLock 的独占模式请求可能会等待几十秒，因为后续的共享模式请求者一个接一个地到来。

准备事务潜伏持有锁定

+   准备事务继续持有锁定，但它不会出现在 [pg_stat_activity](https://www.postgresql.org/docs/current/monitoring-stats.html#MONITORING-PG-STAT-ACTIVITY-VIEW) 中，因为它没有关联的会话。

+   [pg_locks](https://www.postgresql.org/docs/current/view-pg-locks.html)显示准备事务为具有 NULL pid 的条目。检查 pg_prepared_xacts。

### 数据完整性验证

数据校验和

+   目的和用途

    +   每个关系的数据页面，包括所有分支，都在其页面头中具有一个 16 位的校验和。

    +   旨在通过 I/O 系统（例如，卷管理器，文件系统，磁盘驱动器，存储固件，存储设备等）检测损坏。早期检测可防止损坏的传播。

    +   不设计用于检测内存错误。

    +   在整个集群级别启用，要么使用 `initdb` 的 -k/--data-checksums，要么在数据库服务器关闭时使用 `pg_checksums`。由于其性能开销，默认情况下是禁用的。

    +   运行 `"SHOW data_checksums"` 以了解数据校验和是否已启用。它返回 on 或 off。

+   工作原理

    +   当数据页面即将写入磁盘时，从页面内容计算校验和并设置校验和。

    +   刚刚从磁盘读取页面后，通过比较页面头中设置的值和新计算的值来验证校验和。

    +   如果验证失败，则发出警告和错误消息，导致查询失败。

    +   如果在执行校验和验证之前，页面头未通过基本的合理性检查，则查询将以相同的错误消息失败，而不会出现指示校验和失败的警告。

WAL CRC

+   WAL 在每个 WAL 记录头中使用 32 位 CRC。

+   当 WAL 记录放入 WAL 缓冲区时设置 CRC，并在读取 WAL 记录时验证。

检测、绕过或修复数据损坏的实用工具（有些可能很危险！）

+   附加模块

    +   [amcheck](https://www.postgresql.org/docs/current/amcheck.html)：检测堆（表、序列、物化视图）和 B-tree 索引的逻辑损坏。

    +   [pg_surgery](https://www.postgresql.org/docs/current/pgsurgery.html)：`heap_force_kill()`和`heap_force_freeze()`分别强制删除和冻结堆元组。

    +   [pg_visibility_map](https://www.postgresql.org/docs/current/pgvisibility.html)：`pg_check_frozen()`和`pg_check_visible()`检测可见性映射损坏。

+   配置参数

    +   ignore_checksum_failure

    +   zero_damaged_pages

    +   ignore_system_indexes

### 备份和恢复

备份和恢复方法

1.  文件系统级备份（二进制格式）

1.  SQL dump 与 pg_dump/pg_dumpall（文本格式）

1.  连续归档（二进制格式）

备份和恢复方法的特征

+   SQL dump 和连续归档可以在线执行。文件系统级备份需要关闭数据库服务器。

+   SQL dump 可以选择性地备份和还原单个表。其他方法无法备份或还原仅某些单个表或表空间。

+   SQL dump 通常会更小，因为 SQL 脚本只需包含索引创建命令，而不是索引数据。

+   SQL dump 可以加载到较新主要版本的数据库中。

+   SQL dump 可以将数据库传输到不同的机器架构，例如从 32 位服务器到 64 位服务器。

+   连续归档可以执行 PITR。数据库集群可以恢复到最新状态或到某个特定时间点。

+   由 pg_dump 创建的转储文件是一致的；每个数据库的转储是 pg_dump 启动时数据库的快照。pg_dumpall 为每个数据库依次调用 pg_dump，因此不保证数据库集群的一致性。

+   pg_dump 在单个事务中为数据库中的所有数据进行转储，发出许多 SELECT 命令。该长时间运行的事务可能：

    +   阻止其他需要强锁定模式的操作，例如 ALTER TABLE、TRUNCATE、CLUSTER、REINDEX。

    +   导致表和索引膨胀，因为 vacuum 无法删除死元组。

+   连续归档的`pg_backup_start()`和`pg_basebackup`在开始时执行检查点。用户可以在“快速”和“扩散”之间选择检查点速度。

+   存档恢复以及崩溃恢复会清空未记录关系的内容。SQL dump 会输出未记录表的内容。

+   pg_dumpall 的--no-role-passwords 选项使用 pg_roles 而不是 pg_authid 来转储数据库角色。这允许在像 DBaaS 这样的受限环境中使用 pg_dumpall，其中用户不被允许读取 pg_authid 以保护密码。恢复的角色将具有 NULL 密码。

### 流复制

架构

+   拓扑结构

    +   只有整个数据库集群会被复制。无法部分复制。

    +   一个主服务器复制到一个或多个备用服务器。

    +   每个备用服务器从一个主服务器复制。

    +   备用服务器可以级联更改到其他备用服务器。

    +   主服务器不知道备用服务器的位置。备用服务器连接到由 primary_conninfo 参数指定的主服务器。

        +   例如。`primary_conninfo = 'host=192.168.1.50 port=5432 user=foo password=foopass'`

+   主服务器和备用服务器版本

    +   不同的主要版本不兼容。

    +   不同的次要版本会工作，因为磁盘格式相同，但不提供正式支持。建议将主服务器和备用服务器保持在相同的次要版本。

    +   在次要版本升级期间，最安全的做法是首先更新备用服务器。

+   进程和数据流

    +   在服务器启动时，备用服务器首先从归档中读取并应用 WAL，然后从$PGDATA/pg_wal/读取，并启动一个 walreceiver，该 walreceiver 连接到主服务器并从主服务器流式传输 WAL。如果复制连接终止，它会以 5 秒的间隔重复此周期，该间隔可以由 wal_retrieve_retry_interval 配置。

    +   主服务器在接受来自 walreceiver 的连接请求时生成 walsender。

    +   walsender 读取并发送 WAL 到 walreceiver。

    +   walreceiver 将流式 WAL 写入并刷新到$PGDATA/pg_wal/，并通知启动进程。

    +   单个启动进程读取并应用 WAL。

    +   walreceiver 定期通知 walsender 有关复制进度的信息--它已经写入，刷新和应用了多远的 WAL。

    +   级联备用服务器具有运行 walsenders 和 walreceiver 的功能。

+   复制用户

    +   复制用户需要 REPLICATION 角色属性。

    +   REPLICATION 使用户能够读取用于复制的所有数据，但不能用于 SELECT 查询的消耗。

一般管理

+   备用服务器是只读的。包括角色在内的任何对象都不能仅在备用服务器上创建。

+   max_wal_senders 应该略高于备用服务器的数量，这样在临时意外断开连接后，备用服务器仍然保留断开的 walsender 时，备用服务器可以接受连接。

+   备份可以在备用服务器上进行。

+   archive_timeout 不需要缩短数据丢失窗口。

+   级联复制减少了主服务器的负载。

+   主服务器上的 WAL

    +   没有任何措施，主服务器不关心备用服务器并删除/回收备用服务器仍然需要的旧 WAL 文件。

    +   如果备用服务器请求已删除的 WAL，则主服务器会发出如下消息：`"ERROR: requested WAL segment 000000020000000300000041 has already been removed"`。

    +   要使主服务器保留 WAL 文件，可以使用复制插槽（首选）或设置 keep_wal_size（旧方法）。

    +   max_slot_wal_keep_size 限制了复制插槽保留的 WAL 卷。

+   同步复制

    +   如果没有同步备用服务器可用，事务在提交期间会挂起。

    +   要恢复挂起的事务，请移除 synchronous_standby_names 设置并重新加载配置。这会使复制变为异步。

复制延迟的原因

+   硬件配置：服务器、存储和网络

+   主服务器负载过重：主服务器生成的 WAL 量过大，以至于独立启动进程无法跟上。

    +   将 wal_compression 设置为 on 以减少 WAL 的数量。

+   从缓慢的归档检索 WAL：备机无法从主服务器获取 WAL，因此必须从 WAL 归档中获取。

+   恢复冲突：某些操作的回放可能会被在备机上运行的查询阻塞。

    +   当使用热备时，这是相关的。

    +   将 max_standby_archive_delay 和 max_standby_streaming_delay 减少到取消冲突查询并尽早恢复 WAL 回放。

热备

+   在服务器处于归档恢复或备机模式时运行只读查询的能力。

+   要确定服务器是否处于热备状态，请使用 PostgreSQL 14+的`"SHOW in_hot_standby"`或否则使用`"SELECT pg_is_in_recovery()"`

+   恢复冲突

    +   WAL 回放与备机上的查询之间的冲突。

    +   延迟 WAL 回放或取消查询。

    +   导致恢复冲突的主服务器操作包括：

        +   需要独占访问锁的操作：DDL、LOCK、通过 vacuum 进行文件截断（包括自动 vacuum）

            +   独占访问锁请求通过 WAL 记录并由备机重放。

        +   在备机上放置临时文件的表空间被删除

        +   删除备机上连接的客户端的数据库

        +   对备机事务仍然可见的死元组进行 vacuum 清理，根据它们的快照

        +   对备机事务有缓冲引用的页面进行 vacuum 清理（例如，光标位于该页面上。）

    +   在恢复冲突发生时会发生什么

        +   WAL 应用程序最多等待由 max_standby_archive_delay 和 max_standby_streaming_delay 指定的时间段（除了回放 DROP DATABASE 和 ALTER DATABASE SET TABLESPACE 之外。）

        +   然后，在回放 DROP DATABASE 时终止冲突会话，或在其他情况下取消冲突查询。

        +   如果空闲会话持有锁，则该会话也将被终止。

    +   监控恢复冲突

        +   [pg_stat_database_conflicts](https://www.postgresql.org/docs/current/monitoring-stats.html#MONITORING-PG-STAT-DATABASE-CONFLICTS-VIEW) 在备机上显示由于每种恢复冲突类型而取消的查询数量。

        +   "log_recovery_conflict_waits = on" 记录 WAL 应用程序等待时间超过 deadlock_timeout 并且等待结束的消息。

            +   `LOG: recovery still waiting after 1.023 ms: recovery conflict on snapshot`

            +   `DETAIL: Conflicting processes: 1234, 1235`

            +   `LOG: recovery finished waiting after 3.234 ms: recovery conflict on snapshot`

    +   最小化由于恢复冲突取消的查询数量

        +   避免需要独占访问锁的操作。例如，ALTER TABLE、VACUUM FULL、CLUSTER、REINDEX、TRUNCATE

        +   通过在主服务器上设置 vacuum_truncate 存储参数来禁用通过 vacuum 进行的文件截断。

            +   ex. `ALTER TABLE some_table SET (vacuum_truncate = off);`

        +   在备机上设置 hot_standby_feedback = on。

            +   将最老的 XID 发送给主服务器，在 pg_stat_replication.backend_xmin 中反映，当清理死元组时会考虑这一点。

            +   可能因为延迟清理死元组而导致表膨胀。

            +   无法阻止所有冲突。

        +   在备机上调整 max_standby_streaming_delay/max_standby_archive_delay。

        +   在主服务器上调整 vacuum_defer_cleanup_age。

    +   最好有单独的备机，一些用于高可用性，另一些用于容忍陈旧数据的读工作负载。

监控复制延迟

+   [pg_stat_replication](https://www.postgresql.org/docs/current/monitoring-stats.html#MONITORING-PG-STAT-REPLICATION-VIEW)

    +   不仅在主服务器上可用，也在级联备机上可用。

    +   pg_current_wal_lsn 和视图的 sent_lsn 字段之间存在很大差异，可能表明主服务器负载过重。

    +   在备机上，sent_lsn 和 pg_last_wal_receive_lsn 之间的差异可能表示网络延迟，或者备机负载过重。

+   [pg_stat_wal_receiver](https://www.postgresql.org/docs/current/monitoring-stats.html#MONITORING-PG-STAT-WAL-RECEIVER-VIEW)

    +   pg_last_wal_replay_lsn()和视图的 flushed_lsn 之间存在很大差异，表明 WAL 接收速度快于回放速度。

    +   例如。`SELECT pg_wal_lsn_diff(pg_last_wal_replay_lsn(), flushed_lsn) FROM pg_stat_wal_receiver;`

+   [pg_stat_wal](https://www.postgresql.org/docs/current/monitoring-stats.html#MONITORING-PG-STAT-WAL-VIEW)

    +   检查为重写工作负载生成的 WAL 量。

+   检查存储写入延迟、IOPs 和吞吐量，以检查是否存在大量写入活动。

+   [pg_stat_database_conflicts](https://www.postgresql.org/docs/current/monitoring-stats.html#MONITORING-PG-STAT-DATABASE-CONFLICTS-VIEW)

### 逻辑复制

架构

+   使用发布和订阅模型

    +   发布物是要复制其更改的表的集合。

    +   订阅表示连接到发布者及其要订阅的发布物的连接。

    +   一个发布者可以发布一个或多个发布物。

    +   一个订阅者可以有一个或多个订阅。

    +   一个发布物可以有多个订阅者。

    +   一个订阅可以订阅多个发布物。

    +   发布物可以选择限制它们产生的更改到任何组合的 INSERT、UPDATE、DELETE 和 TRUNCATE。

    +   发布物可以限制要复制的行和列。

+   进程和数据流

    +   参与的进程：发布者上的 walsender，订阅者上的订阅工作者（应用工作者、表同步工作者）。

    +   walreceiver 不会出现，即使使用了一些与 walreceiver 相关的参数。

    1.  在订阅者的服务器启动时，逻辑复制启动器会启动，除非 max_logical_replication_workers 为 0。

    1.  逻辑复制启动器为每个启用的订阅启动一个应用工作者。

    1.  应用工作者连接到发布者。

    1.  应用工作者为尚未完成初始同步的表启动 tablesync 工作者。这些 tablesync 工作者各自连接到发布者。

    1.  发布者为每个来自订阅工作者的连接请求生成一个 walsender。

    1.  tablesync 工作者的发布者为 tablesync 工作者发送表的初始副本。（初始数据同步/复制）

    1.  walsender 读取 WAL，将更改解码为逻辑复制协议格式，并将它们存储在逻辑解码工作内存和可能的文件中。当事务提交时，walsender 将其解码的更改发送给订阅工作者。

    1.  订阅工作者应用接收到的更改。

常规管理

+   主要限制

    +   发布只能包含表。

    +   DDL 不会被复制。

        +   在订阅者上首先添加表列，然后在发布者上添加。删除表列时反转顺序。

    +   序列数据不会被复制。

+   复制标识

    +   发布的表必须具有复制标识以复制 UPDATE 和 DELETE 操作。

    +   用作在订阅者上更新或删除行的标识键。

    +   如果发布表没有复制标识，那么在发布者上 UPDATE 和 DELETE 会失败。INSERT 会成功。

    +   可以是主键（默认）、唯一索引或整行。

    +   可以通过`ALTER TABLE REPLICA IDENTITY`进行配置。

    +   复制标识列的旧值被记录在 WAL 中。

+   调整性能

    +   max_sync_workers_per_subscription

        +   基于 max_sync_workers_per_subscription 配置，多个 tablesync 工作者（每个表一个）将并行运行。

        +   当一个订阅中有许多表时，这可能会有效地加快初始表同步的速度。

    +   logical_decoding_work_mem

        +   检查[pg_stat_replication_slots](https://www.postgresql.org/docs/current/monitoring-stats.html#MONITORING-PG-STAT-REPLICATION-SLOTS-VIEW)以查看溢出到磁盘的事务。如果 spill_txns、spill_count 和 spill_bytes 很高，请考虑增加此参数值。

复制冲突

+   由于约束违反或缺乏权限，订阅者上传入更改的应用可能会失败。这就是冲突。

+   解决冲突：

    1.  如果尚未，请通过运行`"ALTER SUBSCRIPTION name DISABLE;"`来禁用订阅。订阅可以被配置为在应用工作者检测到任何错误时自动禁用。运行`"ALTER SUBSCRIPTION ... WITH (disable_on_error = on);"`

    1.  在服务器日志中查找冲突事务的复制起点名称和结束 LSN。

    1.  执行以下操作之一：

        +   通过运行`"ALTER SUBSCRIPTION ... SKIP (end LSN of a conflicting transaction)"`或`"SELECT pg_replication_origin_advance(rep_origin_name, next LSN of the end of a conflicting transaction)"`跳过发布者的事务。

        +   在订阅者上修复表数据。

    1.  通过运行`"ALTER SUBSCRIPTION name ENABLE;"`来启用订阅。

监控

+   发布者

    +   [pg_stat_replication_slots](https://www.postgresql.org/docs/current/monitoring-stats.html#MONITORING-PG-STAT-REPLICATION-SLOTS-VIEW)

        +   每个逻辑复制插槽一行。总事务数和解码数据量，以及溢出到磁盘的事务数和解码数据量。

    +   用于流复制的其他系统和统计视图。

+   订阅者

    +   [pg_stat_subscription_stats](https://www.postgresql.org/docs/current/monitoring-stats.html#MONITORING-PG-STAT-SUBSCRIPTION-STATS)

        +   每个订阅的一行。在初始表同步期间和应用更改时出现的错误数。

    +   [pg_stat_subscription](https://www.postgresql.org/docs/current/monitoring-stats.html#MONITORING-PG-STAT-SUBSCRIPTION) 监控统计信息

        +   每个订阅工作进程（应用程序工作进程，表同步工作进程）的一行。正在复制的表，上次 WAL 位置发送/报告等。

    +   [pg_subscription_rel](https://www.postgresql.org/docs/current/catalog-pg-subscription-rel.html)

        +   每个订阅表的一行。表的状态，以了解初始同步是否正在进行。

### 参考文献

PostgreSQL 文档

连接

WAL

事务

锁

数据完整性验证

备份和恢复

流复制

逻辑复制

## 安全性

### 认证

在更改密码时加密密码

+   `CREATE/ROLE ... PASSWORD 'some_password'` 将指定的密码原样发送和记录，因此指定未加密的密码是危险的。

+   这些语句接受加密密码（使用 MD5 或 SCRAM 哈希）。

+   psql 的 \password 命令很方便

    +   psql 运行 `"SHOW password_encryption"` 命令来确定密码哈希方案（MD5 或 SCRAM），对提供的密码进行哈希处理，然后发出 ALTER 命令。

    +   哈希密码仍然可能出现在服务器日志中。临时将 log_min_error_statement 设置为 'PANIC' 可以防止这种情况。

数据库内的认证配置文件功能非常有限

+   PostgreSQL 仅通过 `CREATE/ROLE VALID UNTIL 'some_timestamp'` 提供密码过期功能。

+   不提供如下功能：

    +   强制密码复杂性

    +   当某用户在特定时间段内的失败登录尝试次数超过阈值时，锁定该用户账户

    +   在一定天数之前限制重新使用相同的密码

实施密码复杂性：使用以下之一：

跟踪失败的登录尝试：执行以下操作之一：

+   搜索服务器日志以查找包含 "password authentication failed" 或 SQLSTATE 28P01 (invalid_password) 的消息

    +   使用 SQLSTATE 比消息文本更好，因为消息可能根据服务器版本和 lc_message 设置而变化。（将 %e 添加到 log_line_prefix 以发出 SQLSTATE。）

+   [PostgreSQL 的 Trusted Language Extensions (pg_tle)](https://github.com/aws/pg_tle)

    +   用户使用客户端认证钩子并创建记录和检查登录失败的扩展。

### 授权

默认情况下角色权限会继承

+   在 SQL 标准和其他 DBMS 中，需要使用 `SET ROLE` 命令以获取另一个角色的权限。

+   在 PostgreSQL 中，角色会自动继承其所属的其他角色的权限。这可能会让人感到惊讶。

+   为了逼近 SQL 标准，对于用户和角色使用 NOINHERIT。

预定义角色

+   提供了一些角色，以将部分管理权限授予非超级用户。

+   可以由 GRANT 给予。

+   代表角色是：

    +   pg_monitor：可以读取各种有用的配置设置、统计信息和其他系统信息。

    +   pg_signal_backend：可以向其他后端发送信号以取消查询或终止会话。

    +   pg_read_server_files、pg_write_server_files 和 pg_execute_server_program：以数据库运行的用户身份访问文件并在数据库服务器上运行程序。例如，这些使 COPY 可以在服务器上的文件和另一个程序之间复制数据，如 gzip 和 curl。

默认权限

+   `ALTER DEFAULT PRIVILEGES`可以设置将自动授予未来创建的数据库对象的默认权限。

    +   例如。`ALTER DEFAULT PRIVILEGES IN SCHEMA app_schema GRANT INSERT, UPDATE, DELETE, SELECT ON TABLES TO app_user;`

+   目标数据库对象是模式、表、视图、序列、函数和类型。

+   不会改变现有数据库对象的权限。

### 参考资料

PostgreSQL 文档

+   [PostgreSQL 用户账户](https://www.postgresql.org/docs/current/postgres-user.html)

+   [数据库角色](https://www.postgresql.org/docs/current/user-manag.html)

+   [客户端身份验证](https://www.postgresql.org/docs/current/client-authentication.html)

+   [权限](https://www.postgresql.org/docs/current/ddl-priv.html)

+   [行安全策略](https://www.postgresql.org/docs/current/ddl-rowsecurity.html)

+   [加密选项](https://www.postgresql.org/docs/current/encryption-options.html)

+   [SSL 安全 TCP/IP 连接](https://www.postgresql.org/docs/current/ssl-tcp.html)

+   [使用 GSSAPI 加密的安全 TCP/IP 连接](https://www.postgresql.org/docs/current/gssapi-enc.html)

身份验证

授权

隐藏数据

## 可管理性

### 内存

大型结果集导致客户端内存耗尽

+   运行 SELECT 时，psql 检索整个结果集并将所有行存储在客户端内存中。

+   使用 FETCH_COUNT 变量，例如`"psql -v FETCH_COUNT=100 ..."`，psql 使用游标并发出 DECLARE、FETCH 和 CLOSE 以逐步检索结果集。

+   客户端驱动程序具有类似的功能，例如 psqlODBC 的 UseDeclareFetch 和 PgJDBC 的 defaultRowFetchSize 连接参数。

服务器端内存不足（OOM）问题的常见原因

+   高数量的连接

    +   即使空闲连接也可能继续占用大量内存。PostgreSQL 在会话期间将数据库对象元数据保留在内存中。这是为了性能。您可以通过庞大的 CacheMemoryContext 来注意到这一点。

    +   如果使用连接池，许多连接随着时间的推移可能会消耗大量内存。这是因为连接是随机从池中选取的，用于访问某些关系，然后释放回池中，这导致许多会话积累了许多关系的元数据。

+   高的 work_mem 值

    +   建议不要在实例级别（postgresql.conf）或数据库级别（ALTER DATABASE）设置高值的 work_mem。许多会话可能同时分配那么多内存。更糟糕的是，每个 SQL 语句可能会并行运行这样的排序和/或哈希操作，其中每个操作可以分配与 work_mem 一样多的内存。

    +   对于基于哈希的操作，最多将分配 work_mem * hash_mem_multiplier 字节的工作内存。

+   低的 max_locks_per_transaction 值

    +   每个可锁定对象（例如，表、索引、序列、XID，但不是行）在锁定时都会在锁表中分配一个条目。该条目表示可锁定对象、授予者、等待者和授予/请求的锁模式。

    +   锁表分配在共享内存中。其大小在服务器启动时固定。

    +   max_locks_per_transaction 的默认值为 64。这意味着每个事务预计将锁定 64 个或更少的对象。

    +   锁表中的条目数为（max_connections + max_prepared_transactions + alpha）* max_locks_per_transaction。

    +   如果可用的话，一个事务可以使用超过 max_locks_per_transaction 条目。

    +   如果许多并发事务中的每个事务可能会访问更多对象，例如，触及数百或数千个分区，请增加 max_locks_per_transaction。

无法检索大的 bytea 值

+   例如，成功插入 550 MB 的 bytea 列值后，抓取失败，并显示类似于`"invalid memory aloc request size 1277232195"`的错误消息。

+   为什么？

    +   当 PostgreSQL 服务器将查询结果发送给客户端时，它会将数据转换为文本格式或以二进制格式返回。

    +   psql 和客户端驱动程序指示服务器使用文本格式。

    +   [当将 bytea 数据转换为文本格式时，PostgreSQL 使用十六进制或转义格式](https://www.postgresql.org/docs/current/datatype-binary.html)。默认格式为十六进制。十六进制和转义格式分别使用 2 和 4 个字节来表示文本格式中的每个原始字节。

        +   例如。`SELECT 'abc'::bytea;` 返回`\x616263`

    +   PostgreSQL 服务器为将每个列值转换为文本格式分配一个连续的内存区域。此分配大小限制为 1 GB - 1。这个限制与 TOAST 中的可变长度数据类型的处理有关。

    +   由于这个限制，PostgreSQL 无法以文本格式返回超过 500 MB 的 bytea 数据。

### 存储

全存储的常见原因

+   表膨胀是因为清理程序无法移除死元组：死元组保留的原因另行描述。

+   WAL 积累：WAL 容量持续增长的原因另行描述。

+   服务器日志文件

    +   因为 pgAudit、auto_explain 和其他日志参数（如 log_statement 和 log_min_duration_statement）而导致过多的日志记录

    +   日志轮换和清除未正确配置：log_rotation_age、log_rotation_size、log_truncate_on_rotation

+   创建临时文件

    +   work_mem 较小和/或查询计划不佳。

    +   保持可保持的游标打开。

        +   例如 `DECLARE CURSOR cur WITH HOLD FOR SELECT * FROM mytable; COMMIT;`

        +   在提交期间，可保持游标的结果集存储在大小为 work_mem 的工作内存区域中，并将超出 work_mem 的内容溢出到临时文件中。

    +   使用以下命令检查临时文件的使用情况：

        +   pg_stat_database 的 temp_files 和 temp_bytes

        +   log_temp_files = on，当删除文件时记录文件路径和大小

        +   通过 EXPLAIN ANALYZE 或 auto_explain 获得的查询计划

存储配额

+   PostgreSQL 除了临时文件外无法限制存储使用。

+   temp_file_limit 可以限制每个会话在任何时刻使用的临时文件的总大小。超过此限制的事务将被中止。

+   如果要限制数据库、表或 WAL（$PGDATA/pg_wal/）的大小，请将其放入具有有限大小的文件系统上的表空间中。

    +   可以通过`CREATE/ALTER DATABASE/TABLE ... TABLESPACE`显式指定数据库/表的表空间，也可以通过 default_tablespace 参数隐式指定。

    +   temp_tablespaces 可用于指定为临时表/索引和排序/散列操作创建临时文件的位置。

    +   WAL 目录可以通过 initdb 的--waldir 选项指定。另外，在创建数据库集群后，可以将其移动到数据目录外并使用符号链接进行链接。

### 记录和调试

`FATAL: database is starting up`

+   在旧的主要版本中，此消息可能在服务器启动期间每秒输出一次。

+   这可能看起来令人震惊，但这并不是实际问题。

+   “为什么？pg_ctl start”在后台启动 postmaster，并尝试在 1 秒间隔内连接到数据库。如果连接成功，pg_ctl 将返回成功。否则，如果服务器仍在执行恢复且无法接受连接，则报告以上消息。

+   在较新的主要版本中，您将不再看到此消息。 pg_ctl 不会尝试连接。相反，当 postmaster 可以接受连接时，postmaster 在 postmaster.pid 中写入“ready”，并且 pg_ctl 对其进行检查。

避免通过限制目标来避免过多的记录。

+   不仅可以为每个用户、数据库或它们的组合配置日志记录，还可以配置许多其他参数。例如：

    +   `ALTER USER oltp_user SET log_min_duration_statement = '3s';`

    +   `ALTER DATABASE analytics_db SET log_min_duration_statement = '60s';`

    +   `ALTER USER batch_user IN DATABASE oltp_db SET log_min_duration_statement = '30s';`

可以为会话启用调试日志记录，而不会使服务器日志混乱

+   可能不可接受将 log_min_messages 全局设置为 DEBUG1 - DEBUG5，因为那样会输出大量日志。

+   您可以像这样在客户端仅获取特定操作的调试消息：

    +   `export PGOPTIONS="-c client_min_messages=DEBUG5"`

`psql -d postgres -c "select 1"`

弄清楚 psql 的反斜杠命令的作用

+   使用 psql 的-E/--echo-hidden 选项。它会显示后台发出的查询。

删除重复行

+   以下查询删除重复行，仅保留最小 ctid 的行并显示已删除的行内容。

+   ctid 是一个系统列，表示行版本在其表内的物理位置：（块编号，项 ID）。由于 UPDATE 和 VACUUM FULL，ctid 可能会改变，因此在此操作期间锁定表可能是安全的。

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

### 清理

清理的目的

+   为了回收或重用由更新或删除行占用的磁盘空间。

+   为了更新 PostgreSQL 查询规划器使用的数据统计信息。

+   为了更新可见性图，从而加速仅索引扫描。

+   为防止由于事务 ID 环绕或多事务 ID 环绕而导致的非常旧数据的丢失。

清理类型

+   并发（惰性或常规）清理

    +   在目标关系上获取共享更新独占锁。不会阻止 SELECT 和 DML 命令。

    +   保留原始数据文件并对其进行修改。TIDs 不会改变。

    +   仅当末尾有一定数量的连续空块时，数据文件才会收缩。文件中间未使用的空间留作重用。

    +   在 pg_stat_progress_vacuum 视图中报告其进度。

+   全量清理

    +   在目标关系上获取独占访问锁。阻止 SELECT 和 DML 命令。

    +   将活跃元组从旧数据文件复制到新数据文件，并删除旧数据文件。重建索引。TIDs 会改变。

    +   数据文件将被完全和最小地打包。

    +   可能会使用两倍的磁盘空间：一个用于现有关系，另一个用于新关系。

    +   总是积极地冻结元组。

    +   实际处理与 CLUSTER 相同。

    +   在 pg_stat_progress_cluster 视图中报告其进度。

+   自动清理永远不会运行全量清理。

清理的主要步骤

1.  开始一个事务。

    +   当存在多个目标关系时，清理会为每个关系启动并提交一个事务，以尽快释放锁。

1.  获取堆的共享更新独占锁并打开它。如果关系无法获取锁，不可逆转的清理会放弃对关系的清理，发出以下消息。

    +   `LOG: 跳过了 vacuum "rel_name" --- 锁不可用`

1.  获取索引的行独占锁并打开它们。

1.  分配工作内存以累积死元组的 TIDs。

1.  直到整个堆被处理完为止，重复执行以下步骤：

    +   扫描堆：在工作内存中累积死元组 TIDs，直到工作内存满或达到堆的末尾。保留死元组的项 ID。此外，如果需要，修剪和重组每个页面，并可能冻结活元组。

    +   清理索引：删除包含死元组 TID 的索引条目。

    +   对堆进行清理：回收死元组的项目 ID。这是在这里完成的，而不是在扫描堆时完成的，因为直到指向它的索引条目被删除之后，项目 ID 才能被释放。

    +   在上述处理过程中更新 FSM 和 VM。

1.  清理索引。

    +   更新 pg_class 的 relpages 和 reltuples 中每个索引的统计信息。

    +   关闭索引，但保留它们的锁直到事务结束。

1.  截断堆，以便将关系末尾的空页面返回给操作系统。

    +   如果堆至少具有 1,000 个块和（relation_size / 16）个连续空块，那么数据文件将被截断。

    +   在堆上获取访问独占锁。如果另一个事务持有冲突的锁，则最多等待 5 秒。如果无法获得锁，则放弃截断。

    +   向后扫描堆以验证末尾页面仍然为空。定期检查是否有另一个事务正在等待冲突的锁。如果有其他人在等待，释放访问独占锁并放弃截断。

1.  更新关系统计信息。

    +   更新 pg_class 的 relpages、reltuples、relallvisible、relhasindex、relhasrules、relhastriggers、relfrozenxid 和 relminmxid。

1.  关闭关系。

1.  提交事务。

1.  对关系的 TOAST 表进行清理。

1.  为每个关系重复上述处理过程。

1.  更新数据库统计信息。

    +   将 pg_database.datfrozenxid 更新为 pg_class.relfrozenxid 值的最小值，并在 pg_xact/中截断提交日志。

    +   将 pg_database.datminmxid 更新为 pg_class.relminmxid 值的最小值，并在 pg_multixact/中截断 MultiXact 数据。

Autovacuum 设计为非侵入式。

+   Autovacuum 每完成一定量的工作就休息（睡眠）一次。因此，它不会持续消耗资源。

    +   “一定量的工作”和休眠时间可以通过 autovacuum_vacuum_cost_limit 和 autovacuum_vacuum_cost_delay 进行配置。autovacuum_vacuum_cost_delay 默认为 2 毫秒。

+   如果由于某些冲突的锁而无法锁定关系，则 Autovacuum 跳过该关系。回绕防止自动真空不会执行此操作。

+   如果并发事务在等待冲突的关系锁并且发现该锁被 Autovacuum 持有，则取消非侵入式 autovacuum。可以看到这些消息：

    +   `ERROR: canceling autovacuum task`

    +   `DETAIL: automatic vacuum of table "mytable"`

+   如果 VM 显示数据页仅具有对所有事务可见的元组（VM 中设置了 all-visible 位），则 VACUUM 跳过读取数据页。即使是这样的页面，Aggressive vacuum 也会读取以冻结元组。

+   如果 VM 显示数据页仅具有冻结元组（VM 中设置了 all-frozen 位），则 VACUUM 跳过读取数据页。

+   当无法在数据页上获得独占 LWLock 时，Autovacuum 对数据页执行减少的工作。对于回绕，Autovacuum 不会执行此操作。

+   如果另一个事务持有或等待对目标关系的锁，则 VACUUM 放弃截断关系。

自动清理未针对一个关系运行

+   检查以下情况是否存在。

    +   pg_stat_all_tables 的 last_autovacuum 和 autovacuum_count 列

    +   将 log_autovacuum_min_duration 设置为 0 后的服务器日志

+   常见原因

    +   关系必须符合自动清理的条件。

        +   UPDATE/DELETE 多的关系：更新/删除的元组 >= autovacuum_vacuum_threshold + autovacuum_vacuum_scale_factor * pg_class.reltuples

        +   INSERT 多的关系：插入的元组 >= autovacuum_vacuum_insert_threshold + autovacuum_vacuum_insert_scale_factor * pg_class.reltuples

    +   自动清理工作者忙于其他许多和/或大型关系。

    +   一些事务持续请求或长时间持有冲突的关系锁。不带环绕预防的清理操作会放弃这样的关系。

    +   自动清理无法清理临时表。需要手动运行清理。这可能导致 XID 环绕和数据库关闭。

    +   存储在 pg_stat/ 中的统计数据由于崩溃或存档恢复（包括故障转移）而丢失。这些统计数据在恢复期间总是被重置。自动清理依赖于统计数据，可以通过 pg_stat_all_tables 看到，以确定是否需要清理。

为什么清理操作不会移除死元组

+   自动清理缓慢

+   运行时间长的事务

+   具有 hot_standby_feedback = on 的物理备用

+   未使用的复制槽

+   孤立的预备事务

减少 XID 环绕的风险

+   减少 XID 的消耗。

    +   每个子事务都分配了自己的 XID。子事务由 SAVEPOINT 和 PL/pgSQL 的异常块（BEGIN ... EXCEPTION）启动。

    +   一些客户端驱动程序提供了语句级别的回滚。它会将每个 SQL 语句用 SAVEPOINT 和 RELEASE SAVEPOINT 包围起来。

+   让自动清理顺利运行（见上文）。

+   降低 autovacuum_vacuum_insert_scale_factor（PostgreSQL 13+）或 autovacuum_freeze_max_age，以便自动清理更频繁地处理表。

+   定期安排 VACUUM FREEZE 运行。

加速自动清理的配置

+   为大型表降低 autovacuum_vacuum_threshold、autovacuum_vacuum_scale_factor、autovacuum_vacuum_insert_threshold、autovacuum_vacuum_insert_scale_factor。

+   减少 autovacuum_naptime

    +   即使写入工作负载繁重，主机具有许多 CPU 核心，1 秒也是实用的。

+   增加 autovacuum_max_workers

    +   在有许多关系的情况下效果更好。每个关系只由一个自动清理工作者处理。

    +   同样要增加 autovacuum_vacuum_cost_limit。否则，每个自动清理工作者都会更频繁地休眠，因为成本限制是所有活动自动清理工作者共享的。

+   增加 maintenance_work_mem/autovacuum_work_mem

    +   工作内存存储着一组死元组的 TID 数组。TID 是 (块编号, 项编号)，大小为 6 个字节。

    +   设置大的值可以减少索引扫描次数。

    +   最大分配大小为 1 GB - 1，无论参数值有多大。

    +   不总是分配指定大小。实际大小足以容纳所有可能的 TID，因此对于小表来说，它会很小。

    +   对于没有索引的表，只分配少于 2 KB。Vacuum 仅累积一个表块的 TID，因为它不需要扫描索引。

+   增加 vacuum_buffer_usage_limit。

    +   默认情况下，Vacuum 使用 256 KB 的环形缓冲区来缓存数据页面，以便不驱逐应用程序可能使用的页面。

    +   Vacuum 也受益于缓存页面：堆页面被读取两次，索引页面可能被多次读取。

    +   将此设置为 0 允许 vacuum 无限制地使用共享缓冲区。

+   减少 autovacuum_vacuum_cost_delay，增加 autovacuum_vacuum_cost_limit。

    +   将 autovacuum_vacuum_cost_delay 设置为 0，这样可以使 autovacuum 运行类似于手动 vacuum。

+   对一个大表进行分区，以便多个 autovacuum 工作者可以同时处理其分区。

+   删除不必要的索引。

    +   Autovacuum 一次处理一个索引。（手动 vacuum 可以使用其 PARALLEL 选项并行处理它们。）

### 升级。

版本的特征。

+   主要版本。

    +   包含新功能和不兼容性。

    +   每年发布一次。

    +   敏感的错误修复仅合并到最新的主要版本中。“敏感”包括可能导致不兼容性、不稳定性和安全性问题或需要大量代码更改但价值不大的修复。

    +   升级可以跳过中间的主要版本。例如，版本 11 可以直接升级到 16，而不需要经过 12 到 15。

    +   始终需要仔细规划和测试来处理不兼容的更改。

+   次要版本。

    +   仅包含经常遇到的错误、安全问题和数据损坏问题，以减少升级风险。

    +   始终建议运行最新的次要版本。社区认为不升级比升级更加危险。

    +   每三个月发布至少一次，2 月、5 月、8 月、11 月的第二个星期四。可能会发布额外的次要版本以解决紧急问题。

    +   升级可以跳过中间的次要版本。

    +   通常不需要转储和恢复；您可以停止数据库服务器，安装更新的二进制文件，然后重新启动服务器。

    +   对于某些次要版本，可能需要额外的手动步骤来修复已修复错误的不良影响，例如重建受影响的索引。请参阅发行说明中的“迁移到版本 <major>.<minor>”部分。

主要升级方法。

1.  pg_dumpall/pg_dump 和 psql/pg_restore：易于使用，停机时间长。

1.  pg_upgrade：相对容易，停机时间较短。

1.  逻辑复制：复杂的设置和操作，最小的停机时间。

pg_upgrade 概述。

+   将数据库集群升级到稍后的主要版本，而无需转储/恢复用户数据。

+   不是就地升级：将数据从旧的数据库集群迁移到使用 initdb 新创建的新数据库集群。

+   基本思想是因为关系数据存储格式很少更改，而只有系统目录的布局更改，所以 pg_upgrade 只是转储并恢复数据库架构，并使用关系数据文件原样。

+   支持从 9.2 版本及更高版本升级。

+   不支持降级。

+   不迁移 pg_statistic 中的数据库统计信息。用户需要在每个数据库完成 pg_upgrade 后运行 ANALYZE。

pg_upgrade 的主要步骤

1.  创建日志和中间文件的输出目录。

1.  检查目标集群的主要版本是否较新，并且通过比较 pg_control 中的信息来检查旧集群和新集群是否二进制兼容。

1.  获取旧集群的数据库和关系（表、索引、TOAST 表、matview）列表。

1.  获取包含 C 语言函数的库名称列表。

1.  执行各种检查以查找升级的阻碍者，例如无法连接到数据库和存在准备事务。

1.  通过运行 `pg_dumpall --globals-only` 创建全局对象的转储。

1.  通过运行 `pg_dump --schema-only` 为每个数据库创建转储。当指定 --jobs 时，这将并行化，为每个数据库生成一个进程或线程。

1.  通过运行 `LOAD` 检查之前提取的带有 C 语言函数的加载库是否存在于新集群中。

1.  从旧集群复制提交日志文件和 pg_xact/ 中的 MultiXact 文件到新集群。

1.  为新集群设置下一个 XID 和 MultiXact ID 以接管旧集群。

1.  通过运行 `psql` 在新集群中恢复全局对象。

1.  通过运行 `pg_restore` 在新集群中恢复数据库模式。当指定 --jobs 时，这将并行化，为每个数据库生成一个进程或线程。

1.  获取新集群的数据库和关系（表、索引、TOAST 表、matview）列表。

1.  从旧集群链接或复制用户关系文件到新集群。当指定 --jobs 时，这将并行化，为每个表空间生成一个进程或线程。

1.  为新集群设置下一个 OID。

1.  创建一个删除旧集群的脚本（delete_old_cluster.sh）。该脚本删除数据目录和表空间版本目录。

1.  报告应更新的扩展并创建 update_extensions.sql。此脚本包含一系列 ALTER EXTENSION ... UPDATE 命令。

用于故障排除 pg_upgrade 的日志文件

+   存储在 $NEWPGDATA/pg_upgrade_output.d/<时间戳>/

+   成功完成 pg_upgrade 后移除。

+   文件：

    +   pg_upgrade_server.log：Postgres 服务器日志。作为 pg_ctl 的 -l 指定。

    +   pg_upgrade_dump_<DB-OID>.log：pg_dump 和 pg_restore 的日志。

    +   pg_upgrade_utility.log：pg_upgrade 运行的各种命令的日志，如 psql、pg_resetwal。这包括用于转储和恢复全局对象的 pg_dumpall/psql。

    +   pg_upgrade_internal.log：其他 pg_upgrade 日志。

    +   loadable_libraries.txt：在旧集群中存在但在新集群中找不到的 C 语言函数库列表。

### 参考文献

PostgreSQL 文档

内存

存储

国际化和本地化

记录

真空

分区

升级

杂项提示

## 应用程序开发

### 数据类型

数字

+   对于精确计算和/或数字位数很多的情况，请选择 numeric 类型。

+   对于小的存储空间和更快的计算，请选择整数类型（smallint、int、bigint）和浮点类型（real、double precision、float）。

+   decimal 类型是 numeric 的别名。 psql 的\d 和 pg_dump 输出 decimal 列为 numeric 而不是 decimal。

时间戳

+   timestamp without time zone 忽略 TimeZone 参数。 值被存储并原样返回。

+   timestamp with time zone 尊重输入值中的显式时区或者 TimeZone 参数。输入值转换为 UTC，输出值根据生效的时区从存储值转换。

二进制

+   存储二进制数据的可用方法

    +   bytea 数据类型

    +   大对象：使用类似文件系统的 open/close/read/write 接口，数据存储在 pg_largeobject 中，用户表列包含指向 pg_largeobject 中行的 OID 值。

    +   外部文件：应用程序在文件系统或对象存储中管理数据，并将文件路径存储在表字符列中。

+   如何选择：

    +   需要事务（ACID）属性吗？->bytea、大对象

    +   处理 1 GB 或更大的列值？->大对象、外部文件

    +   需要随机和/或分段访问吗？->大对象、外部文件

    +   想要在 100 MB 或更大的列值中获得最佳性能？->外部文件

使用大对象的提示

+   **不要使用大对象。** 它们可能会有问题。 使用 bytea 列或外部文件存储，如操作系统文件系统和对象存储。

+   删除大量的 LOB

    +   尝试在单个事务中删除多个大对象，例如，`"SELECT lo_unlink(lo_oid) FROM mytable;"`可能会失败，并显示以下消息：

        +   `错误：共享内存已用完`

        +   `提示：您可能需要增加 max_locks_per_transaction。`

    +   原因：当删除大对象时，它会以独占访问模式锁定。因此，需要在锁表中有与删除的 LOB 相同数量的条目。

    +   解决方案：执行以下任一项或两项：

        +   增加 max_locks_per_transaction。必须重新启动数据库服务器。

        +   按块删除 LOB，例如，每个事务删除 100 个 LOB。

+   处理孤立的 LOB

    +   孤立的 LOB 是一个大对象，其 OID 未出现在数据库的任何 oid 或 lo 数据列中。

    +   如果应用程序在删除关联表行时未调用`lo_unlink()`，则会导致这样一个孤立的 LOB 存在。

        +   解决方案：执行以下任一项或两项：

        +   使用[vacuumlo](https://www.postgresql.org/docs/current/vacuumlo.html)来删除孤立的 LOB。

        +   使用[扩展](https://www.postgresql.org/docs/current/lo.htmllo)，并在 LOB 列上设置触发器。当包含 LOB OID 的表行被更新或删除时，它会自动调用 lo_unlink()。

### 序列

没有无间隙序列

+   当：

    +   事务回滚：因为 nextval()和 setval()调用永远不会回滚，分配的序列值不会被回收。

    +   缓存的值未使用：如果为序列启用了缓存，则 nextval()会预先分配指定数量的值，并将它们缓存在会话的本地内存中。随后的 nextval()调用会从缓存中获取值，直到缓存为空，然后再次预分配一些值。因此，如果会话结束而未使用所有缓存的值，则会出现间隙。

    +   服务器崩溃：即使使用 NO CACHE 序列，您也可以在这些步骤中看到间隙：nextval() -> 崩溃恢复 -> nextval()。出于性能考虑，PostgreSQL 不会为每次从序列获取值都进行 WAL 日志记录。nextval()会 WAL 日志记录当前值之后 32 个数字的值，接下来的 32 次调用 nextval()不会 WAL 日志记录任何内容。因此，某些数字似乎被跳过了。

### 参考文献

PostgreSQL 文档

数据类型

大对象

分页

序列

## 可扩展性和性能

### 许多连接

想处理许多并发客户端？那么就要做：

+   在每个应用程序服务器以及中央服务器上设置连接池。

+   将 max_connections 和实际连接数限制为 CPU 核心数量的几倍，或者最多几百个。

+   性能往往会在此限制以上下降，主要原因是：

    +   客户端后端的内存使用率高，可能导致交换。

    +   CPU 上下文切换

    +   CPU 缓存行争用

    +   锁，特别是自旋锁：如果一个进程持有自旋锁，并且其他进程进入相同的受保护区域，那些后来者将等待自旋锁并继续消耗 CPU。

    +   处理 PostgreSQL 内部数据结构：某些数据结构及其处理取决于连接数；在这里，创建快照突出显示

+   即使是空闲连接也不是无辜的。它们会导致高资源使用率。

### 检测问题

为 ORM（对象关系映射器）增加 track_activity_query_size

+   一些视图，如 pg_stat_activity 和 pg_stat_statements，显示查询字符串。

+   因为这些查询字符串存储在固定大小的共享内存中，每个这样的查询字符串的长度都是固定的，即 track_activity_query_size。较长的查询会被截断到此限制。

+   Hibernate 或其他 ORM 产生非常长的查询。将 track_activity_query_size 设置为 32 KB 左右可能会有用。

利用[plprofiler](https://github.com/bigsql/plprofiler)诊断 PL/pgSQL 函数和过程的瓶颈

+   这是一个创建 HTML 报告的扩展，显示函数和过程每个步骤的运行时间，从那里调用的例程的总运行时间，以及每个例程的执行时间。

### 日志记录

服务器日志可能会阻塞但不显示等待

+   日志收集器（记录器）是唯一将日志写入服务器日志文件的进程。

+   每个后端进程将日志写入其标准错误，该标准错误通过 Unix 管道连接到记录器的读端点。记录器从管道中读取消息并将其写入文件。

+   如果管道被填满，后端在写入时可能会被阻塞。当日志记录量过大时（例如，当使用了一些 pgAudit、auto_explain 和 log_min_duration_statement 的组合并且有许多并发会话运行短查询时），就会发生这种情况。

+   管道上的阻塞不会被视为等待事件（可能是错误），因此后端似乎正在消耗 CPU。

### 导入和导出

`ALTER TABLE SET UNLOGGED/LOGGED` 非常耗费资源

+   这会将整个表重写到新的数据文件中，并将这些写操作写入 WAL 日志。

+   因此，您不能用它来进行有效的数据加载 - 将表切换为 UNLOGGED，加载数据到表中，然后将其设置回 LOGGED。

在使用 COPY 加载数据后，查询或第一个 VACUUM 会变慢

+   这是因为这些命令必须为它们想要查看的行设置提示位。设置提示位会修改共享缓冲区，并在启用数据校验和的情况下写入 WAL 日志。这可能会产生大量的写入。

+   使用`COPY (FREEZE)`来解决。FREEZE 选项冻结加载的行并设置它们的提示位。

+   表必须在当前子事务中已经创建或截断。这是为了防止其他事务在 COPY 事务提交之前看到冻结的行。

### 外部数据包装器（FDW）

通过 [postgres_fdw](https://www.postgresql.org/docs/current/postgres-fdw.html) 加快查询速度

+   在外部表上手动运行 ANALYZE

    +   autovacuum 不会在外部表上执行 ANALYZE。因此，本地统计信息可能会变得过时，并导致查询计划不佳。

+   为长时间运行的查询启用 use_remote_estimate

    +   即，`ALTER FOREIGN SERVER/TABLE ... OPTIONS (use_remote_estimate 'true');`

    +   这使得 postgres_fdw 发出 EXPLAIN 以在远程服务器上执行成本估算。

    +   由于 EXPLAIN 的往返，查询计划时间会变长。因此，对于短查询来说，这可能不值得成本。您可以针对 OLTP、批处理和分析工作负载使用具有不同设置的不同外部服务器/表。

+   增加 fetch_size

    +   即，`ALTER FOREIGN SERVER/TABLE ... OPTIONS (fetch_size '1000');`

    +   postgres_fdw 使用游标从外部表中获取行。fetch_size 确定一次获取的行数。默认值为 100。

    +   如果网络延迟较高，通过增加此设置来减少往返次数可能会有所帮助。请注意，较高的值需要更多的内存来存储获取的行。

+   增加 batch_size

    +   即，`ALTER FOREIGN SERVER/TABLE ... OPTIONS (batch_size '1000');`

    +   默认情况下，postgres_fdw 在多行插入（`INSERT ... SELECT, INSERT ... VALUES (row1), (row2),..., COPY FROM`）期间逐行插入外部表中的一行。

    +   提高此设置将大幅提高吞吐量，特别是在网络延迟高的情况下。

+   在扩展参数中列出在本地和远程服务器上具有兼容行为的扩展名称

    +   即，`ALTER FOREIGN SERVER ... OPTIONS (extensions 'extension1,extension2');`

    +   这些扩展中的不可变函数和运算符被认为会在本地服务器和远程服务器上产生相同的结果。因此，它们的执行将被发送到远程服务器。

    +   这在 WHERE 子句中使用这些函数和运算符时特别有益。这些过滤器将在远程服务器上执行，因此传输的行数较少。

### 全文搜索

插入许多新文档后，全文搜索查询变得慢得多

+   插入数据到启用了 fastupdate 的 GIN 索引时，新的索引条目不会放入主索引结构中。相反，它们被放在由 gin_pending_list_limit 设置的索引的挂起列表中。稍后，当挂起列表区域变满时，那些挂起列表条目将被移动到主索引结构中。

+   这对于良好的性能很重要，因为插入一个文档会涉及对主索引的许多插入，具体取决于文档中的单词数量。

+   全文搜索查询在扫描主索引结构之前会扫描挂起列表。因此，如果挂起列表包含许多挂起条目，查询就会变慢。

+   执行 vacuum（包括自动 vacuum）也会将 pending-list 项移到主索引中。因此，在 vacuum 后，全文搜索查询将变得更快。

+   建议调整 autovacuum，以便在插入或更新文档后合理频繁地运行。

+   可以使用以下查询来查看挂起列表页面和元组的数量（pgstatginindex 在 pgstattuple 扩展中）：

    +   `SELECT * FROM pgstatginindex('some_gin_index');`

### 实用性

快速随机抽样表行

+   传统方法速度慢，因为它扫描并对整个表进行排序。

    +   `SELECT * FROM mytable ORDER BY random() LIMIT 1;`

+   使用 TABLESAMPLE 子句返回行非常快，几乎与表的大小无关。

    +   TABLESAMPLE 获取表的抽样部分。提供了一些内置的抽样方法。

    +   此外，可以通过添加扩展来定制抽样方法。例如，[tsm_system_rows](https://www.postgresql.org/docs/current/tsm-system-rows.html)检索指定数量的随机行：

        +   `CREATE EXTENSION tsm_system_rows;`

        +   `SELECT * FROM mytable TABLESAMPLE SYSTEM_ROWS(1);`

    +   SYSTEM_ROWS 在表的数据文件中随机选取一个块，然后在其中顺序提取行。如果需要更多行，将选择其他块。

### 内存

使用 huge pages

+   设置 huge_pages = on。

    +   这将大大减少内存使用，因为页面表变得更小。

    +   同样，由于 CPU 的 TLB 缓存未命中减少，可以期待性能得到改善。

+   在稳定运行的一部分中，应优先选择“on”而不是“try”来使用 huge_pages，考虑到内存使用量的减少和性能的改善。

    +   当 huge_pages 设置为“on”且操作系统无法分配足够的 huge 页面时，PostgreSQL 拒绝启动并发出以下消息：

        +   `FATAL: could not map anonymous shared memory: Cannot allocate memory`

        +   `提示：这个错误通常意味着 PostgreSQL 对共享内存段的请求超出了可用的内存、交换空间或大页面。要减少请求大小（当前为 1234567890 字节），可以通过减少 PostgreSQL 的共享内存使用量来减少，例如减少 shared_buffers 或 max_connections。`

    +   在这种情况下，重新启动操作系统或执行故障转移。

共享缓冲区的提示

+   避免客户端后端进行磁盘写入。

    +   如果服务器进程想要一个新页面时没有可用的免费共享缓冲区，它必须驱逐一个已使用的缓冲区。如果被驱逐的页面是脏的，则服务器进程需要将页面写入磁盘。这会增加响应时间。

    +   可以通过检查 [pg_stat_bgwriter](https://www.postgresql.org/docs/current/monitoring-stats.html#MONITORING-PG-STAT-BGWRITER-VIEW) 中的 buffers_backend 是否高来检测到这种不良情况。如果 buffers_backend_fsync 也很高，则情况更糟。

    +   缓解方法：

        +   增加更多的空闲缓冲区：增加 shared_buffers。

        +   增加更多的清理缓冲区：增加 bgwriter_lru_multiplier，使后台写进程更积极地写入脏缓冲区。如果 pg_stat_bgwriter 的 maxwritten_clean 经常上升，请尝试增加 bgwriter_lru_maxpages。

+   大型共享缓冲区可能会适得其反。

    +   在具有高性能存储的主机上，共享缓冲区的好处减少了。

    +   这是因为 PostgreSQL 使用操作系统的文件系统缓存：数据被缓存在文件系统缓存中，以及在共享缓冲区中（双缓冲）。

    +   因此，首先将 RAM 的 25% 用于共享缓冲区，然后将其增加到大约 40%，只要您能看到一些改善。

    +   然而，一些基准测试表明，64 GB 或更多可能会有害。

+   利用[pg_prewarm](https://www.postgresql.org/docs/current/pgprewarm.html)在故障转移后快速恢复性能。

    +   数据库服务器重启或故障转移后，共享缓冲区的内容为空或与故障转移前相比可能会有很大不同。因此，应用程序响应时间会恶化，直到共享缓冲区被预热。

    +   在 shared_preload_libraries 中添加 pg_prewarm，并将 pg_prewarm.autoprewarm 设置为 on。

    +   这将启动自动预热工作进程，该进程周期性地将在共享缓冲区中缓存的关系和块编号列表保存在文件中。在服务器启动时，pg_prewarm 工作进程会读取文件以重新填充共享缓冲区。

+   `"SELECT * FROM some_table;"` 不一定会缓存整个表。

    +   你可能希望为性能测试或应用程序预热而这样做，但这是行不通的。而且，它根本不缓存索引。

    +   如果关系的大小大于共享缓冲区的四分之一，则其顺序扫描仅使用 256 KB 的共享缓冲区。

    +   这背后的想法是，只有这样一个扫描所触及的页面不太可能很快再次需要，因此 PostgreSQL 尝试防止这样的大型顺序扫描将许多有用的页面从共享缓冲区中驱逐出去。

    +   同样，批量写入，例如，COPY FROM 和 CREATE TABLE AS SELECT，仅使用 16 MB 的共享缓冲区。

    +   要缓存整个关系，请运行`SELECT pg_prewarm('relation_name')`。这对索引也适用。

本地内存的技巧

+   设置足够的 work_mem 需要尝试和错误。

    +   不幸的是，没有一种简单的方法来估算 work_mem 设置以避免磁盘溢出。

    +   log_temp_files 显示的临时文件不足以说明问题。必须包括在内存中缓冲临时数据的额外开销。

    +   估算 work_mem 的一种方法是将在查询计划中找到的已排序或已散列的计划行的宽度和数量相乘。为了考虑额外的开销，比如再乘以 1.1 或者更多。

    +   如果使用了并行查询，请将结果除以（使用的并行工作者数+1）。“+1”是指并行领导进程。

    +   运行 EXPLAIN ANALYZE 以查看是否使用了外部文件。尝试增加 work_mem 直到外部文件的使用消失为止。

+   effective_cache_size 不会分配任何内存。

    +   这仅用于估算索引扫描的成本。规划器假设此数量的内存可用于缓存查询数据。

    +   较高的值使索引扫描的使用更有可能，较低的值使顺序扫描的使用更有可能。

### 网络

在运行大量短 SQL 命令时要注意网络延迟

+   当您将批处理应用程序（在连续发出许多小 SQL 语句的情况下）迁移到不同的环境时，它是否变得慢了很多倍？

+   这可能是因为网络延迟较高。检查一下网络通信是否缓慢。

    +   测量简单 SQL 的往返时间，例如，

    +   检查等待事件 ClientRead 和 ClientWrite 是否增加。

### 游标

+   DECLARE CURSOR 很快。它创建了一个查询计划，但不计算结果集。FETCH 开始计算。

+   游标查询与非游标查询的计划方式不同。您可以看到相同 SELECT 语句的不同查询计划。

    +   非游标查询针对总运行时间进行了优化。优化器假设客户端将消耗整个结果集。

        +   由于索引扫描被认为是昂贵的，因此更有可能选择顺序扫描和排序。

    +   游标查询针对启动时间和初始数据检索的运行时间进行了优化。优化器假设客户端只会获取结果集的一部分。

        +   优化器选择索引扫描以加快数据的前 10%的创建速度。

        +   “10%”可以通过 cursor_tuple_fraction 参数进行配置。

### 锁

利用高性能的快速路径锁

+   如果大量并发的短事务每个都触及许多关系，则保护锁表的 lwlocks 可能成为争用瓶颈。这种争用可视为 LWLock:LockManager 等待事件。

+   尽管锁表被分成了 16 个分区，并且它们被不同的 lwlocks 覆盖，但数百个并发事务可能导致在这些 lwlocks 上等待。

+   快速路径锁来拯救：

    +   弱锁（访问共享、行共享和行排它模式）使用快速路径锁机制获取。它不使用锁表。相反，这些锁记录在共享内存中的每个后端区域中。

    +   SELECT 和 DMLs 获取这些弱锁，因此它们不会遭受锁管理器 lwlock 竞争的影响。

+   然而，如果以下任一条件为真，则无法使用快速路径锁：

    +   事务已经具有 16 个快速路径关系锁。每个后端记录区域限制为 16 个条目。访问具有许多分区和索引的表或连接许多表的查询将丢失。

    +   某些事务尝试获取强锁（共享、共享行排它、排它和访问排它锁模式）。

        +   同一关系上的现有快速路径锁将转移到锁表中。

        +   如果有人拥有或请求强锁，则随后的事务可能无法使用快速路径锁获取不同关系上的锁。这是因为强锁的存在是使用 1024 个整数计数器的数组管理的，这些计数器实际上是对锁空间的 1024 分区。如果请求的弱锁要在与现有强锁相同的分区中管理，则不能使用快速路径。

+   快速路径锁会显示在 [pg_locks](https://www.postgresql.org/docs/current/view-pg-locks.html) 中，其 fastpath 列为 true。

### HOT

利用 HOT（仅堆元组）

+   HOT 加快了 UPDATE。

+   如果没有使用 HOT，那么出了什么问题？

    +   索引会变得更大，因为每个行版本在每个索引中都有一个索引条目。使用这些索引进行索引扫描也会变慢。

    +   WAL 卷会变大，更新会变慢，因为任何列的更新都会在所有索引中插入新条目。

+   要使 HOT 起作用，必须同时满足以下两个条件：

    +   包含更新行的块具有足够的空闲空间来容纳新的行版本。

    +   更新不修改任何索引列。

+   那么，我该怎么办呢？

    +   在表上设置填充因子以为新行版本腾出空间。

        +   例如，`CREATE TABLE mytable ... WITH (fillfactor = 90);`，`ALTER TABLE mytable SET (fillfactor = 90);`

        +   较低的填充因子会使表变大，从而导致共享缓冲区未命中和较长的顺序扫描。

        +   也许你应该从 fillfactor = 90 开始，并在 HOT 的效果不佳时降低设置。

    +   删除不必要的索引。

+   我怎么知道 HOT 是否在起作用？

### 表布局

为了获得最佳的存储效率和性能，应从最大的固定长度类型（例如 bigint、timestamp）到最小的固定长度类型（例如 smallint、bool），然后到可变长度类型（例如 numeric、text、bytea）声明表列。

+   存储效率来自数据对齐要求。

    +   例如，bigint 在 8 字节边界上对齐，而 bool 在 1 字节边界上对齐。

    +   在以下示例中，前者返回 48，后者返回 39。

        +   `SELECT pg_column_size(ROW('true'::bool, '1'::bigint, '1'::smallint, '1'::int));`

        +   `SELECT pg_column_size(ROW('1'::bigint, '1'::int, '1'::smallint, 'true'::bool));`

    +   可以通过 `SELECT typalign, typname FROM pg_type ORDER BY 1, 2;` 查看对齐要求。

+   更小的数据大小和直接列访问带来更好的性能。

    +   如果固定长度列位于行的前面，PostgreSQL 可以计算并缓存行中固定长度列的位置。因此，可以使用其偏移量直接访问任何行的请求的固定长度列数据。

    +   一旦出现可变长度列，需要为每行计算后续列的位置，方法是加上其列的实际长度。因此，对行尾的列的访问将变慢。

在 FROM 子句中使用返回复合类型的函数，而不是 SELECT 列列表

+   假设 sample_func() 的返回类型是复合类型 `(a int, b int, c int)`。

+   不好：`SELECT (sample_func()).*;`

+   好的：`SELECT * FROM sample_func();`

+   在不好的情况下，`"(sample_func()).*"` 被扩展为 `"(sample_func()).a"`、`"(sample_func()).b"`、`"(sample_func()).c"`。因此，该函数被调用三次。

TOAST（超大属性存储技术）

+   这是一种存储高达 1 GB - 1 的大值的机制。

+   元组不能跨多个页面。那么，如果列值大于页面大小（通常为 8 KB），该如何存储？

+   可 TOAST 的数据类型的大列值被压缩和/或分成块。每个块作为表的相关 TOAST 表中的单独行存储。块大小被选择为四个块行将适合于一个页面。这对于 8 KB 页面大小约为 2,000 字节。

+   可 TOAST 的数据类型是具有可变长度（varlena）表示的数据类型。也就是说，一个 1 或 4 字节的 varlena 头后跟列值。char(n) 看起来像是固定长度的，但它有 varlena 格式。

+   TOAST 表

    +   每个表有 0 或 1 个 TOAST 表和 TOAST 索引。

    +   如果需要，TOAST 表及其索引将在 CREATE/ALTER TABLE 中创建。

    +   TOAST 表是 pg_toast.pg_toast_<main_table_OID>。

    +   TOAST 索引是 pg_toast.pg_toast_<main_table_OID>_index。

    +   TOAST 表的 OID 存储在表的 pg_class.reltoastrelid 中。

    +   TOAST 索引的 OID 存储在 TOAST 表的 pg_class.reltoastidxid 中。

    +   每个 TOAST 表都有这些列：

        +   chunk_id OID：标识特定的 TOASTed 值

        +   chunk_seq int：值内块的序列号

        +   chunk_data bytea：块的实际数据

        +   主键（chunk_id，chunk_seq）

+   TOAST 如何工作

    +   仅当要存储在表中的行值宽于 2 KB（当页面大小为 8 KB 时）时才会触发。

    +   压缩和/或将列值移动到 TOAST 表，直到行值短于 2 KB（当页面大小为 8 KB 时）或无法获得更多收益为止。可以使用 CREATE/ALTER TABLE 中的存储参数 toast_tuple_target 来调整每个表的此 2 KB 阈值。

    +   将 TOAST 值的 chunk_id 存储在主表的列中。这称为 TOAST 指针。

+   可以使用`ALTER TABLE ALTER COLUMN column_name SET STORAGE { PLAIN | EXTERNAL | EXTENDED | MAIN }`从四个选项中选择值存储策略——是否应该压缩或移动到 TOAST 表中。

+   压缩方法可以在 pglz 和 lz4 之间选择。它可以通过在 CREATE/ALTER TABLE 中使用 COMPRESSION 列选项为每个列设置，否则使用 default_toast_compression 参数。

+   TOAST 值的插入可能会变得出奇的慢。

    +   这种情况通常在目标表已经拥有数百万个 TOAST 值后出现，特别是在连续插入大量行之后。

    +   为什么？

        +   每个 TOAST 值都由一个 OID 标识。

        +   OID 是一个无符号 4 字节值，它是从每 40 亿个值都会环绕的群集范围计数器生成的。因此，单个表不能有超过 2³²（40 亿）个 TOAST 值。

        +   当插入一个 TOAST 值时，PostgreSQL 为其生成一个新的 OID，检查目标表中是否已经使用了相同的 OID 的现有 TOAST 值。如果已使用，PostgreSQL 将生成下一个 OID 并再次执行检查。直到找到一个空闲的 OID。

        +   如果目标表中使用连续的 OID，则此重试会花费很长时间。

+   解决方法是对表进行分区。每个分区都有自己的 TOAST 表。因此，每个分区中重复 OID 的可能性降低了。

### 事务

被杀死（死掉）的索引元组可能会使查询速度奇怪地加快

+   如果在相同查询的相同执行计划中遇到不同的执行时间，可能是由于被杀死的索引元组。

+   每当索引扫描获取一个堆元组，只发现它已经死掉时，它就会将索引元组标记为被杀死的（死掉的）。然后，将来的索引扫描将会忽略它。这样可以避免其索引键比较以及其堆元组获取。

子事务可能会有害

+   子事务是事务的一部分，可以回滚而不回滚主（顶级）事务。

+   子事务可以由 SAVEPOINT 命令显式启动，也可以在 PL/pgSQL 中进入带有 EXCEPTION 子句的块时隐式启动。

+   一些客户端驱动程序提供了一个选项，可以为每个 SQL 语句启动和结束一个子事务，例如 PgJDBC 的连接参数“autosave”。要注意它们的默认值。

+   每个子事务在执行需要 XID 的操作时（例如修改数据或锁定行）都会分配自己的 XID。

+   元组头的 xmin 和 xmax 字段记录了更新它的子事务的 XID。为了检查元组的可见性，看到 xmin/xmax 的事务需要知道主事务是否已经结束，而不是子事务。

+   如何知道子事务的主事务：

    +   当子事务分配其 XID 时，它会在$PGDATA/pg_subtrans/中记录其直接父项的 XID。

    +   pg_subtrans 的结构是 XIDS 数组。 例如，XID 100 的父 XID 存储在数组的第 101 个元素中。 数组分为 8 KB 页面。

    +   pg_subtrans 数据被缓存在 32 页的内存区域中。 该区域由 SLRU（简单最近未使用）缓冲区管理。 因此，缓存可以包含 32 页 * 8 KB / 4 = 65,536 个事务。

    +   因此，为了获取主事务的 XID，需要遍历与子事务嵌套深度相同的条目数。

+   pg_subtrans 是否总是用于元组可见性的检查？

    +   不。 快照不仅存储主事务的 XID，还存储子事务的 XID。 如果检查者的快照包含所有子事务，则可以在不查阅 pg_subtrans 的情况下完成工作。

    +   然而，情况并非总是如此。 每个后端最多可以在共享内存中的 ProcArray 条目中具有 64 个子事务 XID。 如果主事务有超过 64 个子事务，则其 ProcArray 条目将被标记为溢出。

    +   在创建快照时，扫描所有运行中事务的 ProcArray 条目，以收集主事务和子事务的 XID。 如果任何条目被标记为溢出，则将快照标记为子溢出。

    +   子溢出的快照不包含确定可见性所需的所有数据，因此必须使用 pg_subtrans 将元组的 xmin/xmax 追溯到它们的顶层事务 XID。

+   所以，问题是什么？

    +   pg_subtrans 的读取器与写入器竞争 lwlocks 以保护 SLRU 缓冲区，读取器和写入器分别以共享模式和独占模式锁定其父级的 XID。

    +   pg_subtrans 缓存不是很大。 在许多并发子事务中，会出现磁盘 I/O。

+   我怎样才能知道这种情况发生的可能性？

    +   等待事件 LWLock:SubtransBuffer，LWLock:SubtransSLRU，IO:SLRURead 和 IO:SLRUWrite 不断增长。

    +   [pg_stat_slru](https://www.postgresql.org/docs/current/monitoring-stats.html#MONITORING-PG-STAT-SLRU-VIEW)显示其 Subtrans 行中的 blks_read 和 blks_hit 持续增加。（PostgreSQL 13+）

    +   pg_stat_get_backend_subxact(backend_id)返回 subxact_count 和 subxact_overflow。（PostgreSQL 16+）

在内部，MultiXact 可能会影响性能

+   MultiXact 是什么？

    +   一种记录元组上多个锁定者 XID 的机制。（多事务）

    +   元组头中的 xmax 字段记录了锁定该元组的 XID。

    +   那么，当多个事务在同一元组上获取锁时会发生什么？

        +   例如，具有 XID 100 的第一个事务运行了`SELECT ... FOR SHARE`。 xmax 变成了 100。

        +   接下来，具有 XID 101 的第二个事务在相同的元组上运行相同的`SELECT ... FOR SHARE`。 然后，分配一个新的 MultiXact ID，例如 1，并设置为 xmax 字段。

        +   将 MultiXact ID 1 到实际锁定者的 XIDS（100、101）的映射添加到$PGDATA/pg_multixact/中。

+   外键约束实现为执行`"SELECT ... FOR KEY SHARE"`的约束触发器。 因此，可能在您不知情的情况下使用 MultiXact。

+   问题可能是什么？

    +   像 pg_subtrans 一样，pg_multixact 也通过 SLRU 进行缓存。因此，它可能会受到 lwlock 争用和磁盘 I/O 的影响。

    +   当一个 XID 被添加为现有 MultiXact ID 的新成员时，将分配一个新的 MultiXact ID，并将现有成员 XID 复制到新位置。在上面的例子中，当 XID 102 加入具有成员（100、101）的 MultiXact ID 1 时，将新分配 MultiXact 2，将（100、101）复制到那里，并添加 102。如果许多事务同时锁定同一行，则此复制变得更加重要。

+   我怎么知道这种情况发生的可能性？

    +   等待事件 LWLock:MultiXact*、IO:SLRURead 和 IO:SLRUWrite 不断增加。

    +   [pg_stat_slru](https://www.postgresql.org/docs/current/monitoring-stats.html#MONITORING-PG-STAT-SLRU-VIEW) 在其行中显示了 MultiXactOffset 和 MultiXactMember 的 blks_read 和 blks_hit 的增加。 (PostgreSQL 13+)

    +   pg_get_multixact_members('<MultiXact ID>') 返回一个成员 XID 及其锁模式的集合。

### WAL 和检查点

检查点概述

+   通过刷新未写入（=脏）缓存数据，同步内存和存储中的数据的处理。

+   什么时候执行？

    +   自上次检查点以来已经经过了由 checkpoint_timeout 指定的时间。

    +   根据 max_wal_size 累积了一定数量的 WAL。

    +   在基本备份开始时（pg_basebackup，pg_backup_start()）。

    +   关闭数据库实例。

    +   完成任何形式的恢复。

    +   其他杂项所需的时间，例如 CREATE DATABASE，以便数据文件可以在不经过共享缓冲区的情况下被复制/移动。

+   由 checkpoint_timeout 引起的检查点称为计划检查点，而其他检查点称为请求检查点。

+   完成检查点时，旧的 WAL 段文件将被删除或回收为新的 WAL 段文件以供将来重用，基于 min_wal_size。

    +   在这里，“旧的”意味着“不再需要用于崩溃恢复，因为这些 WAL 段中的所有更改都已经持久化到数据文件中”。

    +   然而，旧的 WAL 段文件会保留，直到它们被归档并且不再被 wal_keep_size 或任何复制插槽所需。

+   检查点是侵入式的，因为：

    +   存储 I/O 争用，对于数据和 WAL 都是如此。

    +   缓冲区内容锁 lwlock 争用：当检查点正在以共享模式持有其缓冲区内容锁并刷新共享缓冲区时，修改相同缓冲区的事务需要独占锁，并且需要等待 lwlock 被释放。

    +   由于完整页写入，WAL 体积增加。

+   什么是完整页写入？

    +   在检查点之后对每个数据页进行首次修改时，整个页面内容都会被 WAL 记录，而不仅仅是更改部分。这对于在恢复期间恢复破损的页面是必要的。

    +   如果在 PostgreSQL 写入页面时主机崩溃，则可能会导致页面破损。因为 I/O 的原子单位通常比 PostgreSQL 页面大小小（例如，512 字节的磁盘扇区），所以可能会出现页面的一部分是新的而另一部分是旧的。

减少检查点的影响

+   监控检查点的频率。

    +   调度和请求的检查点数量分别可以在[pg_stat_bgwriter](https://www.postgresql.org/docs/current/monitoring-stats.html#MONITORING-PG-STAT-BGWRITER-VIEW)中的 checkpoints_timed 和 checkpoints_req 中查看。

        +   绝大部分检查点应该是计划的，而不是请求的。计划的检查点允许负载均匀分布在系统的正常操作中。频繁的请求检查点可能导致性能变化。

    +   如果两个连续检查点之间的经过时间小于 checkpoint_warning，并且新的检查点是由 WAL 积累请求的，则服务器日志会显示以下消息。

        +   `LOG:检查点发生得太频繁（相隔 8 秒）`

        +   `提示：考虑增加配置参数“max_wal_size”。`

+   通过增加 max_wal_size 和/或 checkpoint_timeout 来降低检查点的频率。

    +   注意，这可能会增加崩溃恢复所需的时间。

+   将 wal_compression 设置为 on。这会减少对全页写入 WAL 的需求。

+   增加 min_wal_size。这减少了事务创建新 WAL 段文件的需求。

### 索引

索引的缺点

+   索引会消耗磁盘空间。

+   更大的磁盘空间会增加物理备份的大小和持续时间。

+   索引会减慢 INSERT/DELETE/COPY 语句，因为它们总是要修改所有索引。

+   索引会阻止 HOT 更新。HOT 仅适用于对非索引列的修改。

你可能没有注意到的索引好处

+   B-tree 索引可以加速 max()和 min()聚合。它们只需在索引的末尾读取索引条目。

+   表达式上的索引也会收集计算值的统计信息。

    +   例如：`CREATE INDEX myindex1 ON mytable ((col1 + col2 * 3));`

    +   你可以查看索引表达式的统计信息。例如，上述情况中，统计信息出现在 pg_stats 中，tablename=myindex1，attname=expr。

    +   对索引表达式可以设置统计目标。例如，`ALTER INDEX index_name ALTER COLUMN expr SET STATISTICS 1000;`

+   外键上的索引可以加快约束处理的速度。

    +   例如：`CREATE TABLE orders (...，product_id int REFERENCES products ON CASCADE DELETE);`

    +   你可以使用 EXPLAIN ANALYZE 和 auto_explain 来查看约束级联处理所用的时间。外键约束在内部使用触发器实现。

        +   例如：`EXPLAIN ANALYZE DELETE FROM products WHERE product_id = 2;`

        +   ...`约束 orders_product_id_fkey 的触发器：时间=0.322 调用=1`

使索引只扫描起作用

+   使用 EXPLAIN ANALYZE 查看索引只扫描需要读取堆的次数。例如，它显示类似于“堆抓取：0”。0 是最好的。

+   将 autovacuum 设置得更积极一些，或者运行 VACUUM 来更新可见性地图。这会减少堆抓取。

### 查询计划

ANALYZE 的风险

+   Autovacuum 不会在临时表或外部表上运行 ANALYZE。需要手动运行 ANALYZE。

+   即使表内容没有更改，ANALYZE 之后也可能会改变查询计划。

    +   ANALYZE 获取表内容的随机样本（300 x 默认统计目标行）。因此，收集的统计信息可能会因读取的行而异。

    +   要避免或减少此查询计划变化，请执行以下操作之一：

        +   使用像 pg_hint_plan 这样的第三方软件修复查询计划。

        +   提高 ANALYZE 收集的统计信息量，即 `ALTER TABLE ... ALTER COLUMN ... SET STATISTICS`。使用的行数越多，统计数据波动就越小。但是，这会使 ANALYZE 和查询规划变慢，因为写入或读取的统计数据更多。

        +   将表的存储参数 autovacuum_analyze_threshold 和 autovacuum_analyze_scale_factor 设置为较大的值，以便 autovacuum 几乎不会 ANALYZE 它。然后，如果需要，进行手动 ANALYZE。

使用返回集函数可能导致查询计划不佳

+   当函数用于 WHERE 子句或连接中过滤行时，可能会观察到这种情况。

+   这是因为规划器没有关于选择性的合理准确信息。因此，其成本估算将不准确。

+   CREATE/ALTER FUNCTION 可以设置固定的成本和返回的行数。SUPPORT 子句提供的规划器支持函数，需要用 C 编写，可以动态更改成本和行数。

    +   `CREATE FUNCTION ... RETURNS {SETOF ... | TABLE(...)} COST execution_cost ROWS result_rows SUPPORT support_function`

自定义计划和通用计划

+   PREPARE 执行解析、分析和重写以生成准备好的语句。

    +   例如 `PREPARE stmt(int) AS SELECT * FROM mytable WHERE col = $1;`

+   EXECUTE 制定查询计划并执行它。

+   考虑特定参数值的查询计划是最好的。这种计划称为自定义计划。另一方面，不考虑参数值的查询计划称为通用计划。

+   通过占位符的存在，你可以区分自定义计划和通用计划。例如，

    +   自定义计划：`Filter: (col = 123)`

    +   通用计划：`Filter: (col = $1)`

+   但是规划是昂贵的。如果通用计划足够好，PostgreSQL 会使用它来避免制定自定义计划。

    +   PostgreSQL 在准备语句的前五次执行中使用自定义计划。

    +   在第六次执行时，将生成一个通用计划，并将其成本与过去五次执行的平均成本进行比较。

    +   如果通用计划的成本更低，则会继续采用它。不会考虑自定义计划。

    +   否则，将创建并使用新的自定义计划。在后续的执行中，将通用计划的成本与所有过去自定义计划执行的平均成本进行比较，并选择较低者。

+   你可以通过将 plan_cache_mode 设置为 force_generic_plan 或 force_custom_plan 来强制使用通用计划或自定义计划。如果通用计划的成本估计过低，则可能需要强制使用自定义计划。

### 参考资料

PostgreSQL 文档

多连接

问题检测

存储器

存储

网络

表格布局

SQL 技巧

事务

热度优化（HOT）

WAL 和检查点

锁定

索引

查询规划

连接

日志记录

并行查询

导入和导出

外部数据包装器（FDW）

触发器

全文搜索

实用性
