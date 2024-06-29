<!--yml

类别：未分类

日期：2024-05-29 12:46:28

-->

# 在Postgres C代码中查找内存泄漏

> 来源：[https://www.enterprisedb.com/blog/finding-memory-leaks-postgres-c-code](https://www.enterprisedb.com/blog/finding-memory-leaks-postgres-c-code)

我上周一直在寻找Postgres的WAL发送进程中的内存泄漏。我花了几天时间更熟悉Valgrind和gcc/clang的sanitizers，但最终对它们没有什么有用的进展。最后，我偶然发现了bcc工具集合中的memleak程序，它直接将我引向了问题的源头。

自从我第一次尝试弄清楚这一切以来，我想分享一下我经历的过程。与一些人为的内存泄漏问题共事。

尽管这发生在Postgres中，并且这篇文章介绍了在Postgres中引入泄漏以便我们进行调查的过程，但这些技术是广泛有用的。

具体来说，通过本文的最后部分，我们将能够获得一条很好的堆栈跟踪，指向运行中程序的泄漏，而无需对程序进行任何修改。

```
        135168 bytes in 1 allocations from stack
                0x0000ffff7fb7ede8      sysmalloc_mmap.isra.0+0x74 [libc.so.6]
                0x0000ffff7fb7fc40      sysmalloc+0x200 [libc.so.6]
                0x0000ffff7fb80fbc      _int_malloc+0xdfc [libc.so.6]
                0x0000ffff7fb81c50      __malloc+0x22c [libc.so.6]
                0x0000fffffffff000      [unknown] [[uprobes]]
                0x0000000000927714      palloc+0x40 [postgres]
                0x00000000007df2f4      PostgresMain+0x1ac [postgres]
                0x0000000000755f34      report_fork_failure_to_client+0x0 [postgres]
                0x0000000000757e94      BackendStartup+0x1b8 [postgres]
                0x000000000075805c      ServerLoop+0xfc [postgres]
                0x00000000007592f8      PostmasterMain+0xed4 [postgres]
                0x00000000006927d8      main+0x230 [postgres]
                0x0000ffff7fb109dc      __libc_start_call_main+0x7c [libc.so.6]
                0x0000ffff7fb10ab0      __libc_start_main@GLIBC_2.17+0x9c [libc.so.6]
                0x00000000004912f0      _start+0x30 [postgres]

```

## Valgrind

计算机科学学生通常通过Valgrind来学习调试程序中的内存泄漏问题。但Valgrind主要在程序结束时检查泄漏，而不是在执行过程中。

例如，在这个简单而有泄漏的`main.c`中：

```
#include "stdio.h"
#include "string.h"

int main() {
  char* x = strdup("flubber");
  printf("%s\n", x);
  return 1;
}

```

我们可以用Valgrind构建和运行，它会告诉我们有关泄漏的信息：

```
$ gcc main.c
$ valgrind --leak-check=full ./a.out
==203008== Memcheck, a memory error detector
==203008== Copyright (C) 2002-2022, and GNU GPL'd, by Julian Seward et al.
==203008== Using Valgrind-3.22.0 and LibVEX; rerun with -h for copyright info
==203008== Command: ./a.out
==203008==
flubber
==203008==
==203008== HEAP SUMMARY:
==203008==     in use at exit: 8 bytes in 1 blocks
==203008==   total heap usage: 2 allocs, 1 frees, 1,032 bytes allocated
==203008==
==203008== 8 bytes in 1 blocks are definitely lost in loss record 1 of 1
==203008==    at 0x48C0860: malloc (vg_replace_malloc.c:442)
==203008==    by 0x49A8FB3: strdup (strdup.c:42)
==203008==    by 0x4101FB: main (in /home/phil/tmp/a.out)
==203008==
==203008== LEAK SUMMARY:
==203008==    definitely lost: 8 bytes in 1 blocks
==203008==    indirectly lost: 0 bytes in 0 blocks
==203008==      possibly lost: 0 bytes in 0 blocks
==203008==    still reachable: 0 bytes in 0 blocks
==203008==         suppressed: 0 bytes in 0 blocks
==203008==
==203008== For lists of detected and suppressed errors, rerun with: -s
==203008== ERROR SUMMARY: 1 errors from 1 contexts (suppressed: 0 from 0)

```

我们可以添加正确的调用`free(x)`：

```
#include "stdio.h"
#include "string.h"
#include "stdlib.h"

int main() {
  char* x = strdup("flubber");
  printf("%s\n", x);
  free(x);
  return 1;
}

```

重新构建，并再次用Valgrind运行：

```
$ gcc main.c
$ valgrind --leak-check=full ./a.out
==203099== Memcheck, a memory error detector
==203099== Copyright (C) 2002-2022, and GNU GPL'd, by Julian Seward et al.
==203099== Using Valgrind-3.22.0 and LibVEX; rerun with -h for copyright info
==203099== Command: ./a.out
==203099==
flubber
==203099==
==203099== HEAP SUMMARY:
==203099==     in use at exit: 0 bytes in 0 blocks
==203099==   total heap usage: 2 allocs, 2 frees, 1,032 bytes allocated
==203099==
==203099== All heap blocks were freed -- no leaks are possible
==203099==
==203099== For lists of detected and suppressed errors, rerun with: -s
==203099== ERROR SUMMARY: 0 errors from 0 contexts (suppressed: 0 from 0)

```

而Valgrind告诉我们确实没有泄漏。

## 在Arenas中的泄漏

但是有各种方法来管理内存。Postgres代码通常不直接调用malloc。它调用`palloc`在`MemoryContext`中分配内存（在其他代码库中有时称为“[arenas](https://www.rfleury.com/p/untangling-lifetimes-the-arena-allocator)”）。`MemoryContexts`是一块从中`palloc`获取内存的空闲内存。然后在某个时刻一次性释放。

问题在于内存上下文的生命周期内可能会发生泄漏。而Valgrind很难告诉您这些泄漏的情况。

Postgres确实有一个有用的内置方法来转储内存上下文信息。例如，您可以将此行添加到任何函数中，或者可以从gdb中执行此操作：

```
MemoryContextStats(TopMemoryContext);
```

然后，您将获得关于内存上下文和所有子内存上下文中分配的信息，写入stderr。通过观察`MemoryContextStats`随时间增加的分配量，并通过添加额外的内存上下文，您可能能够发现泄漏的位置。

但是明确添加此调用或通过`gdb`添加是相当手动的。如果有一些系统可以自动告诉我们有关内存分配增长的信息，那将更加方便。

让我们在Postgres中引入一个内存泄漏，并看看我们能做些什么来发现它，重新演示我们不知道泄漏位置的情况。

## 设置

首先获取Postgres，检出版本16.2并构建它：

```
$ git clone https://github.com/postgres/postgres
$ cd postgres
$ git checkout REL_16_2
$ ./configure \
    CFLAGS="-ggdb -Og -g3 -fno-omit-frame-pointer -DUSE_VALGRIND" \
    --prefix=$(pwd)/build \
    --libdir=$(pwd)/build/lib \
    --enable-debug
$ make && make install

```

这些标志启用Valgrind，启用调试模式，并设置构建以本地安装而不是全局安装。

构建完后，您可以创建一个Postgres数据库：

```
$ ./build/bin/initdb test-db
The files belonging to this database system will be owned by user "phil".
This user must also own the server process.

The database cluster will be initialized with locale "en_US.UTF-8".
The default database encoding has accordingly been set to "UTF8".
The default text search configuration will be set to "english".

Data page checksums are disabled.

creating directory test-db ... ok
creating subdirectories ... ok
selecting dynamic shared memory implementation ... posix
selecting default max_connections ... 100
selecting default shared_buffers ... 128MB
selecting default time zone ... America/New_York
creating configuration files ... ok
running bootstrap script ... ok
performing post-bootstrap initialization ... ok
syncing data to disk ... ok

initdb: warning: enabling "trust" authentication for local connections
initdb: hint: You can change this by editing pg_hba.conf or using the option -A, or --auth-local and --auth-host, the next time you run initdb.

Success. You can now start the database server using:

    build/bin/pg_ctl -D test-db -l logfile start

```

然后在前台启动Postgres服务器：

```
$ ./build/bin/postgres --config-file=$(pwd)/test-db/postgresql.conf -D $(pwd)/test-db -k $(pwd)/test-db
2024-03-21 16:08:10.984 EDT [213083] LOG:  starting PostgreSQL 16.2 on aarch64-unknown-linux-gnu, compiled by gcc (GCC) 13.2.1 20231205 (Red Hat 13.2.1-6), 64-bit
2024-03-21 16:08:10.986 EDT [213083] LOG:  listening on IPv6 address "::1", port 5432
2024-03-21 16:08:10.986 EDT [213083] LOG:  listening on IPv4 address "127.0.0.1", port 5432
2024-03-21 16:08:10.987 EDT [213083] LOG:  listening on Unix socket "/home/phil/tmp/postgres/test-db/.s.PGSQL.5432"
2024-03-21 16:08:10.989 EDT [213086] LOG:  database system was shut down at 2024-03-21 16:07:20 EDT
2024-03-21 16:08:10.992 EDT [213083] LOG:  database system is ready to accept connections

```

在第二个终端中，用我们构建的`psql`连接到它：

```
./build/bin/psql -h localhost postgres
psql (16.2)
Type "help" for help.

postgres=# select 1;
 ?column?
----------
        1
(1 row)

```

太好了！现在让我们引入一个内存泄漏。

## 引入内存泄漏

`src/backend/tcop/postgres.c`中的`PostgresMain()`函数有一个看起来很诱人的巨大循环。

```
 4417     /*
 4418      * Non-error queries loop here.
 4419      */
 4420
 4421     for (;;)
 4422     {
 4423         int         firstchar;
 4424         StringInfoData input_message;
 4425
 4426         #define CHUNKSIZE 4096
 4427         char* waste = palloc(CHUNKSIZE);
 4428         printf("%d from %d", CHUNKSIZE, getpid());

```

让我们进行这个改变：

```
diff --git a/src/backend/tcop/postgres.c b/src/backend/tcop/postgres.c
index 36cc99ec9c..28d08f3b11 100644
--- a/src/backend/tcop/postgres.c
+++ b/src/backend/tcop/postgres.c
@@ -4423,6 +4423,10 @@ PostgresMain(const char *dbname, const char *username)
                int                     firstchar;
                StringInfoData input_message;

+               #define CHUNKSIZE 4096
+               char* waste = palloc(CHUNKSIZE);
+               printf("%d from %d\n", CHUNKSIZE, getpid());
+
                /*
                 * At top of loop, reset extended-query-message flag, so that any
                 * errors encountered in "idle" state don't provoke skip.

```

因为我们在主Postgres循环中分配内存，所以每当单个连接的客户端在同一连接上发送语句时，我们将分配一个新的4KB块。

因为我们使用`palloc`，所有内存将在进程结束时以Valgrind认为有效的方式释放。但在单个客户端的生命周期内，处理连接的Postgres后端进程将随着客户端发送语句而增加内存。

这是一个内存泄漏！在极端情况下，如果客户端发送1百万条语句，我们将在一个进程中分配4KB * 1,000,000 = 4GB，并且在进程退出之前不会释放。虽然这是一个有些牵强的例子，但这种情况确实会发生。

## 设置Valgrind包装器

让我们用我们的差异重建，记住Valgrind已经启用。

```
$ make && make install
```

我们仍然需要在Valgrind中包装Postgres。Tom Lane在Postgres邮件列表上[分享](https://www.postgresql.org/message-id/159904.1608307376%40sss.pgh.pa.us)了他用于此目的的包装器脚本，我已经进行了适应。将其添加到Postgres仓库根目录中的`postgres.valgrind`中：

```
#!/usr/bin/env bash

mkdir -p $HOME/valgrind-logs

SCRIPT_DIR=$( cd -- "$( dirname -- "${BASH_SOURCE[0]}" )" &> /dev/null && pwd )

exec valgrind \
     --quiet \
     --show-leak-kinds=all \
     --track-origins=yes \
     --verbose \
     --suppressions=$(realpath $SCRIPT_DIR/../../)/src/tools/valgrind.supp \
     --time-stamp=yes --error-exitcode=128 --trace-children=yes \
     --log-file=$HOME/valgrind-logs/%p.log \
     $SCRIPT_DIR/postgres.orig "$@"

```

旁注：我也尝试使用--leak-check=full标志运行此Valgrind脚本，但是当我使用此标志时，Postgres总是崩溃。我不知道为什么。

现在将当前的Postgres二进制文件重命名为`postgres.orig`，并将此脚本复制为新的Postgres二进制文件：

```
$ mv build/bin/postgres build/bin/postgres.orig
$ chmod +x postgres.valgrind
$ cp postgres.valgrind build/bin/postgres

```

启动服务器（无需重新`initdb`）：

```
$ ./build/bin/postgres --config-file=$(pwd)/test-db/postgresql.conf -D $(pwd)/test-db -k $(pwd)/test-db
2024-03-21 16:18:28.233 EDT [216336] LOG:  starting PostgreSQL 16.2 on aarch64-unknown-linux-gnu, compiled by gcc (GCC) 13.2.1 20231205 (Red Hat 13.2.1-6), 64-bit
2024-03-21 16:18:28.234 EDT [216336] LOG:  listening on IPv6 address "::1", port 5432
2024-03-21 16:18:28.234 EDT [216336] LOG:  listening on IPv4 address "127.0.0.1", port 5432
2024-03-21 16:18:28.234 EDT [216336] LOG:  listening on Unix socket "/home/phil/tmp/postgres/test-db/.s.PGSQL.5432"
2024-03-21 16:18:28.236 EDT [216336] LOG:  database system was shut down at 2024-03-21 16:18:00 EDT
2024-03-21 16:18:28.238 EDT [216336] LOG:  database system is ready to accept connections

```

并像以前一样连接`psql`：

```
$ ./build/bin/psql -h localhost postgres
psql (16.2)
Type "help" for help.

postgres=#

```

您将立即在Postgres服务器进程中看到一行日志：

```
4096 from 216344
```

这就是我们引入的内存泄漏。如果我们在同一个`psql`进程中多次运行`SELECT 1`：

```
$ ./build/bin/psql -h localhost postgres
psql (16.2)
Type "help" for help.

postgres=# select 1;
 ?column?
----------
        1
(1 row)

postgres=# select 1;
 ?column?
----------
        1
(1 row)

postgres=# select 1;
 ?column?
----------
        1
(1 row)

postgres=# select 1;
 ?column?
----------
        1
(1 row)

```

我们将在服务器日志中看到相同数量的打印语句：

```
... omitted ...
2024-03-21 16:18:28.238 EDT [216336] LOG:  database system is ready to accept connections
4096 from 216344
4096 from 216344
4096 from 216344
4096 from 216344
4096 from 216344

```

现在如果我们在服务器进程上按下Ctrl-c，我们可以查看Valgrind日志。

```
$ cat ~/valgrind-logs/*
==00:00:00:00.000 216336== Memcheck, a memory error detector
==00:00:00:00.000 216336== Copyright (C) 2002-2022, and GNU GPL'd, by Julian Seward et al.
==00:00:00:00.000 216336== Using Valgrind-3.22.0 and LibVEX; rerun with -h for copyright info
==00:00:00:00.000 216336== Command: /home/phil/tmp/postgres/build/bin/postgres.orig --config-file=/home/phil/tmp/postgres/test-db/postgresql.conf -D /home/phil/tmp/postgres/test-db -k /home/phil/tmp/postgres/test-db
==00:00:00:00.000 216336== Parent PID: 143066
==00:00:00:00.000 216336==
==00:00:00:14.925 216336==
==00:00:00:14.925 216336== HEAP SUMMARY:
==00:00:00:14.925 216336==     in use at exit: 218,493 bytes in 75 blocks
==00:00:00:14.925 216336==   total heap usage: 1,379 allocs, 353 frees, 1,325,653 bytes allocated
==00:00:00:14.925 216336==
==00:00:00:14.926 216336== LEAK SUMMARY:
==00:00:00:14.926 216336==    definitely lost: 410 bytes in 5 blocks
==00:00:00:14.926 216336==    indirectly lost: 3,331 bytes in 45 blocks
==00:00:00:14.926 216336==      possibly lost: 102,984 bytes in 8 blocks
==00:00:00:14.926 216336==    still reachable: 47,148 bytes in 182 blocks
==00:00:00:14.926 216336==         suppressed: 0 bytes in 0 blocks
==00:00:00:14.926 216336== Rerun with --leak-check=full to see details of leaked memory
==00:00:00:14.926 216336==
==00:00:00:14.926 216336== For lists of detected and suppressed errors, rerun with: -s
==00:00:00:14.926 216336== ERROR SUMMARY: 0 errors from 0 contexts (suppressed: 0 from 0)

```

它只找到了410字节明确丢失。无法解释每个客户端语句积累的4096 * 5 = 20,480字节的损失。

此外，Valgrind非常慢。Valgrind的[手册](https://valgrind.org/docs/manual/valgrind_manual.pdf)中说：

> 您的程序将比正常运行慢得多（例如，20到30倍），并且使用更多内存。

我们需要尝试其他方法。

## AddressSanitizer、LeakSanitizer

Valgrind自2002年以来一直存在。但大约在2013年，Google直接在`clang`和后来的`gcc`中贡献了类似的功能。这段历史对我来说有点难以理解。AddressSanitizer和LeakSanitizer可能曾经是独立的，但现在AddressSanitizer包含了LeakSanitizer。要获取此Sanitizer，只需使用`-fsanitize=address`编译代码。

让我们回到那个简单的、有内存泄漏的`main.c`程序：

```
$ cat main.c
#include "stdio.h"
#include "string.h"
#include "stdlib.h"

int main() {
  char* x = strdup("flubber");
  printf("%s\n", x);
  return 1;
}

```

并使用 Sanitizers 编译并运行：

```
$ gcc -fsanitize=address main.c
$ ./a.out
flubber
=================================================================
==223338==ERROR: LeakSanitizer: detected memory leaks

Direct leak of 8 byte(s) in 1 object(s) allocated from:
    #0 0xffff90c817d4 in __interceptor_strdup (/lib64/libasan.so.8+0x817d4) (BuildId: 3109905b64795755dad05d7bb29ad23633a06660)
    #1 0x410238 in main (/home/phil/tmp/a.out+0x410238) (BuildId: e98d3e4308c692d5fedef8494d1a925d8f40759a)
    #2 0xffff90a609d8 in __libc_start_call_main (/lib64/libc.so.6+0x309d8) (BuildId: 8adf04071ee0060cbf618c401e50c21b9070755f)
    #3 0xffff90a60aac in __libc_start_main@@GLIBC_2.34 (/lib64/libc.so.6+0x30aac) (BuildId: 8adf04071ee0060cbf618c401e50c21b9070755f)
    #4 0x41012c in _start (/home/phil/tmp/a.out+0x41012c) (BuildId: e98d3e4308c692d5fedef8494d1a925d8f40759a)

SUMMARY: AddressSanitizer: 8 byte(s) leaked in 1 allocation(s).

```

嘿，这太完美了。甚至比使用 Valgrind 和需要包装器更简单。泄漏检测就直接内建到二进制文件中了。

让我们试试在 Postgres 上。

## Postgres 和 AddressSanitizer

回到之前克隆的 Postgres git 仓库，记得我们应用了一个会在每个客户端语句中泄漏内存的差异。

```
diff --git a/src/backend/tcop/postgres.c b/src/backend/tcop/postgres.c
index 36cc99ec9c..28d08f3b11 100644
--- a/src/backend/tcop/postgres.c
+++ b/src/backend/tcop/postgres.c
@@ -4423,6 +4423,10 @@ PostgresMain(const char *dbname, const char *username)
                int                     firstchar;
                StringInfoData input_message;

+               #define CHUNKSIZE 4096
+               char* waste = palloc(CHUNKSIZE);
+               printf("%d from %d\n", CHUNKSIZE, getpid());
+
                /*
                 * At top of loop, reset extended-query-message flag, so that any
                 * errors encountered in "idle" state don't provoke skip.
```

运行 `make clean`，然后再次运行 autoconf，但这次在 `CFLAGS` 值中加入 `-fsanitize=address`。然后重新构建。

```
$ make clean
$ ./configure \
    CFLAGS="-ggdb -Og -g3 -fno-omit-frame-pointer -fsanitize=address" \
    --prefix=$(pwd)/build \
    --libdir=$(pwd)/build/lib \
    --enable-debug
$ make && make install

```

现在再次运行 Postgres 服务器：

```
./build/bin/postgres --config-file=$(pwd)/test-db/postgresql.conf -D $(pwd)/test-db -k $(pwd)/test-db
2024-03-22 10:31:54.724 EDT [233257] LOG:  starting PostgreSQL 16.2 on aarch64-unknown-linux-gnu, compiled by gcc (GCC) 13.2.1 20231205 (Red Hat 13.2.1-6), 64-bit
2024-03-22 10:31:54.725 EDT [233257] LOG:  listening on IPv6 address "::1", port 5432
2024-03-22 10:31:54.725 EDT [233257] LOG:  listening on IPv4 address "127.0.0.1", port 5432
2024-03-22 10:31:54.727 EDT [233257] LOG:  listening on Unix socket "/home/phil/tmp/postgres/test-db/.s.PGSQL.5432"
2024-03-22 10:31:54.731 EDT [233260] LOG:  database system was interrupted; last known up at 2024-03-22 10:28:56 EDT
2024-03-22 10:31:54.740 EDT [233260] LOG:  database system was not properly shut down; automatic recovery in progress
2024-03-22 10:31:54.741 EDT [233260] LOG:  invalid record length at 0/1531B08: expected at least 24, got 0
2024-03-22 10:31:54.741 EDT [233260] LOG:  redo is not required
2024-03-22 10:31:54.742 EDT [233258] LOG:  checkpoint starting: end-of-recovery immediate wait
2024-03-22 10:31:54.793 EDT [233258] LOG:  checkpoint complete: wrote 3 buffers (0.0%); 0 WAL file(s) added, 0 removed, 0 recycled; write=0.050 s, sync=0.001 s, total=0.052 s; sync files=2, longest=0.001 s, average=0.001 s; distance=0 kB, estimate=0 kB; lsn=0/1531B08, redo lsn=0/1531B08

=================================================================
==233260==ERROR: LeakSanitizer: detected memory leaks
Direct leak of 328 byte(s) in 1 object(s) allocated from:
    #0 0xffffaa8e57b0 in malloc (/lib64/libasan.so.8+0xc57b0) (BuildId: 3109905b64795755dad05d7bb29ad23633a06660)
    #1 0x10104ac in save_ps_display_args /home/phil/tmp/postgres/src/backend/utils/misc/ps_status.c:165
    #2 0x9aed7c in main /home/phil/tmp/postgres/src/backend/main/main.c:91
    #3 0xffffa9fb09d8 in __libc_start_call_main (/lib64/libc.so.6+0x309d8) (BuildId: 8adf04071ee0060cbf618c401e50c21b9070755f)
    #4 0xffffa9fb0aac in __libc_start_main@@GLIBC_2.34 (/lib64/libc.so.6+0x30aac) (BuildId: 8adf04071ee0060cbf618c401e50c21b9070755f)
    #5 0x4a15ac in _start (/home/phil/tmp/postgres/build/bin/postgres+0x4a15ac) (BuildId: 4cede14f9b35edbb8bf598d2efbc883c4ace6797)

SUMMARY: AddressSanitizer: 328 byte(s) leaked in 1 allocation(s).
2024-03-22 10:31:54.831 EDT [233257] LOG:  startup process (PID 233260) exited with exit code 1
2024-03-22 10:31:54.831 EDT [233257] LOG:  terminating any other active server processes
2024-03-22 10:31:54.831 EDT [233257] LOG:  shutting down due to startup process failure
2024-03-22 10:31:54.832 EDT [233257] LOG:  database system is shut down

=================================================================
==233257==ERROR: LeakSanitizer: detected memory leaks

Direct leak of 328 byte(s) in 1 object(s) allocated from:
    #0 0xffffaa8e57b0 in malloc (/lib64/libasan.so.8+0xc57b0) (BuildId: 3109905b64795755dad05d7bb29ad23633a06660)
    #1 0x10104ac in save_ps_display_args /home/phil/tmp/postgres/src/backend/utils/misc/ps_status.c:165
    #2 0x9aed7c in main /home/phil/tmp/postgres/src/backend/main/main.c:91
    #3 0xffffa9fb09d8 in __libc_start_call_main (/lib64/libc.so.6+0x309d8) (BuildId: 8adf04071ee0060cbf618c401e50c21b9070755f)
    #4 0xffffa9fb0aac in __libc_start_main@@GLIBC_2.34 (/lib64/libc.so.6+0x30aac) (BuildId: 8adf04071ee0060cbf618c401e50c21b9070755f)
    #5 0x4a15ac in _start (/home/phil/tmp/postgres/build/bin/postgres+0x4a15ac) (BuildId: 4cede14f9b35edbb8bf598d2efbc883c4ace6797)

SUMMARY: AddressSanitizer: 328 byte(s) leaked in 1 allocation(s).

```

好的，它立即崩溃了！这很有趣。

但仔细观察，这些消息与我们引入的 4096 字节泄漏无关，而且似乎不像是重要的泄漏，因为它们与程序初始化相关，而且不在循环中，所示的分配不会随时间而增长。所以这不是个大问题。

我们的泄漏在哪里呢？原来 AddressSanitizer/LeakSanitizer 的默认行为是在第一个错误时崩溃。在这种情况下，以及许多只在稍后引入 Sanitizers 的生产系统中，这并不实用。幸运的是，Sanitizers 有一些我们可以在环境变量中设置的运行时选项。

让我们设置环境变量 `ASAN_OPTIONS="halt_on_error=false"`，然后重新运行 Postgres：

```
$ ASAN_OPTIONS="halt_on_error=false" ./build/bin/postgres --config-file=$(pwd)/test-db/postgresql.conf -D $(pwd)/test-db -k $(pwd)/test-db
2024-03-22 10:38:07.588 EDT [233554] LOG:  starting PostgreSQL 16.2 on aarch64-unknown-linux-gnu, compiled by gcc (GCC) 13.2.1 20231205 (Red Hat 13.2.1-6), 64-bit
2024-03-22 10:38:07.589 EDT [233554] LOG:  listening on IPv6 address "::1", port 5432
2024-03-22 10:38:07.589 EDT [233554] LOG:  listening on IPv4 address "127.0.0.1", port 5432
2024-03-22 10:38:07.591 EDT [233554] LOG:  listening on Unix socket "/home/phil/tmp/postgres/test-db/.s.PGSQL.5432"
2024-03-22 10:38:07.595 EDT [233557] LOG:  database system was shut down at 2024-03-22 10:38:06 EDT

=================================================================
==233557==ERROR: LeakSanitizer: detected memory leaks

Direct leak of 336 byte(s) in 1 object(s) allocated from:
    #0 0xffffbc6957b0 in malloc (/lib64/libasan.so.8+0xc57b0) (BuildId: 3109905b64795755dad05d7bb29ad23633a06660)
    #1 0x10104ac in save_ps_display_args /home/phil/tmp/postgres/src/backend/utils/misc/ps_status.c:165
    #2 0x9aed7c in main /home/phil/tmp/postgres/src/backend/main/main.c:91
    #3 0xffffbbd609d8 in __libc_start_call_main (/lib64/libc.so.6+0x309d8) (BuildId: 8adf04071ee0060cbf618c401e50c21b9070755f)
    #4 0xffffbbd60aac in __libc_start_main@@GLIBC_2.34 (/lib64/libc.so.6+0x30aac) (BuildId: 8adf04071ee0060cbf618c401e50c21b9070755f)
    #5 0x4a15ac in _start (/home/phil/tmp/postgres/build/bin/postgres+0x4a15ac) (BuildId: 4cede14f9b35edbb8bf598d2efbc883c4ace6797)

SUMMARY: AddressSanitizer: 336 byte(s) leaked in 1 allocation(s).
2024-03-22 10:38:07.651 EDT [233554] LOG:  database system is ready to accept connections

```

好吧！它报告了一个泄漏，但继续运行。让我们像之前一样连接 `psql` 触发我们添加的泄漏。

```
$ ./build/bin/psql -h localhost postgres
psql (16.2)
Type "help" for help.

postgres=# select 1;
 ?column?
----------
        1
(1 row)

postgres=# select 1;
 ?column?
----------
        1
(1 row)

postgres=# select 1;
 ?column?
----------
        1
(1 row)

postgres=# select 1;
 ?column?
----------
        1
(1 row)

```

然后退出连接。你可能会看到 `psql` 进程本身的泄漏报告。但我们要找的泄漏在 Postgres 后端进程中。

```
2024-03-22 10:38:07.651 EDT [233554] LOG:  database system is ready to accept connections
4096 from 233705
4096 from 233705
4096 from 233705
4096 from 233705
4096 from 233705

=================================================================
==233710==ERROR: LeakSanitizer: detected memory leaks
Direct leak of 336 byte(s) in 1 object(s) allocated from:
    #0 0xffffbc6957b0 in malloc (/lib64/libasan.so.8+0xc57b0) (BuildId: 3109905b64795755dad05d7bb29ad23633a06660)
    #1 0x10104ac in save_ps_display_args /home/phil/tmp/postgres/src/backend/utils/misc/ps_status.c:165
    #2 0x9aed7c in main /home/phil/tmp/postgres/src/backend/main/main.c:91
    #3 0xffffbbd609d8 in __libc_start_call_main (/lib64/libc.so.6+0x309d8) (BuildId: 8adf04071ee0060cbf618c401e50c21b9070755f)
    #4 0xffffbbd60aac in __libc_start_main@@GLIBC_2.34 (/lib64/libc.so.6+0x30aac) (BuildId: 8adf04071ee0060cbf618c401e50c21b9070755f)
    #5 0x4a15ac in _start (/home/phil/tmp/postgres/build/bin/postgres+0x4a15ac) (BuildId: 4cede14f9b35edbb8bf598d2efbc883c4ace6797)

SUMMARY: AddressSanitizer: 336 byte(s) leaked in 1 allocation(s).

=================================================================
==233705==ERROR: LeakSanitizer: detected memory leaks

Direct leak of 336 byte(s) in 1 object(s) allocated from:
    #0 0xffffbc6957b0 in malloc (/lib64/libasan.so.8+0xc57b0) (BuildId: 3109905b64795755dad05d7bb29ad23633a06660)
    #1 0x10104ac in save_ps_display_args /home/phil/tmp/postgres/src/backend/utils/misc/ps_status.c:165
    #2 0x9aed7c in main /home/phil/tmp/postgres/src/backend/main/main.c:91
    #3 0xffffbbd609d8 in __libc_start_call_main (/lib64/libc.so.6+0x309d8) (BuildId: 8adf04071ee0060cbf618c401e50c21b9070755f)
    #4 0xffffbbd60aac in __libc_start_main@@GLIBC_2.34 (/lib64/libc.so.6+0x30aac) (BuildId: 8adf04071ee0060cbf618c401e50c21b9070755f)
    #5 0x4a15ac in _start (/home/phil/tmp/postgres/build/bin/postgres+0x4a15ac) (BuildId: 4cede14f9b35edbb8bf598d2efbc883c4ace6797)

SUMMARY: AddressSanitizer: 336 byte(s) leaked in 1 allocation(s).

```

但是关于我们的泄漏没有任何信息！我猜 Sanitizers 在这方面与 Valgrind 类似，因为内存在进程结束之前被释放了，而不是之前。

此时我陷入了困境。

## eBPF 和 bcc 工具

我一直在阅读 Brendan Gregg 的《[系统性能](https://www.brendangregg.com/blog/2020-07-15/systems-performance-2nd-edition.html)》一书，所以我开始想知道是否有办法使用 [perf](https://perf.wiki.kernel.org/index.php/Main_Page) 或 [bpf profile](https://github.com/iovisor/bcc/blob/master/tools/profile.py) 来记录运行程序的内存使用情况，并查看分配来源。我已经使用 `perf` 来分析 *函数*，但没有用于 *内存使用* 的分析。我尝试找到如何记录分配和堆栈跟踪的方法，但没能搞清楚。

因此，我查看了 `bpf profile` 并注意到 Brendan Gregg 在内存泄漏方面的[页面](https://www.brendangregg.com/FlameGraphs/memoryflamegraphs.html)。原来有一种工具可以直接识别内存泄漏，叫做 [memleak.py](https://github.com/iovisor/bcc/blob/master/tools/memleak.py)！

如果你还没有安装，可以[安装](https://github.com/iovisor/bcc/blob/master/INSTALL.md) bcc 工具套件。不幸的是，每个发行版都改变了程序的名称，所以可能最容易使用 `find`：

```
$ sudo find / -name '*memleak*'
/tmp/kheaders-6.5.6-300.fc39.aarch64/include/linux/kmemleak.h
/usr/share/man/man3/curs_memleaks.3x.gz
/usr/share/man/man8/bcc-memleak.8.gz
/usr/share/bcc/tools/doc/memleak_example.txt
/usr/share/bcc/tools/memleak
/usr/src/kernels/6.7.9-200.fc39.aarch64/include/linux/kmemleak.h
/usr/src/kernels/6.7.9-200.fc39.aarch64/samples/kmemleak

```

好的，它位于 `/usr/share/bcc/tools/memleak`。

让我们重新构建 Postgres，去掉 Sanitizers：

```
$ make clean
$ ./configure \
    CFLAGS="-ggdb -Og -g3 -fno-omit-frame-pointer" \
    --prefix=$(pwd)/build \
    --libdir=$(pwd)/build/lib \
    --enable-debug
$ make && make install

```

运行Postgres服务器：

```
$ ./build/bin/postgres --config-file=$(pwd)/test-db/postgresql.conf -D $(pwd)/test-db -k $(pwd)/test-db
2024-03-22 11:36:06.364 EDT [245850] LOG:  starting PostgreSQL 16.2 on aarch64-unknown-linux-gnu, compiled by gcc (GCC) 13.2.1 20231205 (Red Hat 13.2.1-6), 64-bit
2024-03-22 11:36:06.364 EDT [245850] LOG:  listening on IPv6 address "::1", port 5432
2024-03-22 11:36:06.364 EDT [245850] LOG:  listening on IPv4 address "127.0.0.1", port 5432
2024-03-22 11:36:06.365 EDT [245850] LOG:  listening on Unix socket "/home/phil/tmp/postgres/test-db/.s.PGSQL.5432"
2024-03-22 11:36:06.368 EDT [245853] LOG:  database system was shut down at 2024-03-22 11:36:05 EDT
2024-03-22 11:36:06.370 EDT [245850] LOG:  database system is ready to accept connections
```

并启动`psql`，获取处理服务器上我们请求的相关后端进程PID：

```
$ ./build/bin/psql -h localhost postgres
psql (16.2)
Type "help" for help.

postgres=# select pg_backend_pid();
 pg_backend_pid
----------------
         245858
(1 row)

```

打开一个新的终端会话，保持`psql`连接和Postgres服务器进程的活动状态。然后运行`memleak`程序：

```
$ sudo /usr/share/bcc/tools/memleak -p 245858
Attaching to pid 245858, Ctrl+C to quit.
[11:37:19] Top 10 stacks with outstanding allocations:
[11:37:24] Top 10 stacks with outstanding allocations:

```

它将每五秒钟开始报告内存泄漏。

让我们通过在`psql`会话中继续运行`SELECT 1`语句来更积极地触发内存泄漏：

```
$ ./build/bin/psql -h localhost postgres
psql (16.2)
Type "help" for help.

postgres=# select pg_backend_pid();
 pg_backend_pid
----------------
         245858
(1 row)

postgres=# select 1;
 ?column?
----------
        1
(1 row)

postgres=# select 1;
 ?column?
----------
        1
(1 row)

```

在运行这个`SELECT 1`命令大约20-30次之后，你应该会看到`memleak`程序报告：

```
        135168 bytes in 1 allocations from stack
                0x0000ffff7fb7ede8      sysmalloc_mmap.isra.0+0x74 [libc.so.6]
                0x0000ffff7fb7fc40      sysmalloc+0x200 [libc.so.6]
                0x0000ffff7fb80fbc      _int_malloc+0xdfc [libc.so.6]
                0x0000ffff7fb81c50      __malloc+0x22c [libc.so.6]
                0x0000fffffffff000      [unknown] [[uprobes]]
                0x0000000000927714      palloc+0x40 [postgres]
                0x00000000007df2f4      PostgresMain+0x1ac [postgres]
                0x0000000000755f34      report_fork_failure_to_client+0x0 [postgres]
                0x0000000000757e94      BackendStartup+0x1b8 [postgres]
                0x000000000075805c      ServerLoop+0xfc [postgres]
                0x00000000007592f8      PostmasterMain+0xed4 [postgres]
                0x00000000006927d8      main+0x230 [postgres]
                0x0000ffff7fb109dc      __libc_start_call_main+0x7c [libc.so.6]
                0x0000ffff7fb10ab0      __libc_start_main@GLIBC_2.17+0x9c [libc.so.6]
                0x00000000004912f0      _start+0x30 [postgres]

```

而*这*就是内存泄漏，还附带着漂亮的堆栈跟踪，帮助我们准确定位它的位置。
