<!--yml
category: 未分类
date: 2024-05-29 12:46:28
-->

# Finding memory leaks in Postgres C code

> 来源：[https://www.enterprisedb.com/blog/finding-memory-leaks-postgres-c-code](https://www.enterprisedb.com/blog/finding-memory-leaks-postgres-c-code)

I spent the last week looking for a memory leak in Postgres’s WAL Sender process. I spent a few days getting more acquainted with Valgrind and gcc/clang sanitizers, but ultimately got nowhere useful with them. Finally, I stumbled on the memleak program from the bcc tools collection which led me right to the source.

Since I had a bit of trouble figuring this all out for the first time, I wanted to share the process I went through. Working with some contrived memory leaks.

Although this happened in Postgres, and this post introduces a leak into Postgres to set us up for an investigation, the techniques are broadly useful.

Specifically, by the end of this post, we’ll be able to get a nice stack trace leading to a leak for a running program without any modification to the program required.

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

Computer Science students typically learn about Valgrind for debugging memory leaks in programs. But Valgrind primarily checks for leaks at the end of the program, not during it.

For example in this simple, leaky `main.c`:

```
#include "stdio.h"
#include "string.h"

int main() {
  char* x = strdup("flubber");
  printf("%s\n", x);
  return 1;
}

```

We can build and run with Valgrind and it will tell us about the leak:

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

We can add in the correct call to `free(x)`:

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

Rebuild, and run with Valgrind again:

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

And Valgrind tells us there is indeed no leak.

## Leaks in Arenas

But there are various ways to manage memory. Postgres code doesn’t typically call malloc directly. It calls `palloc` to allocate memory in a `MemoryContext` (which in other codebases is sometimes called “[arenas](https://www.rfleury.com/p/untangling-lifetimes-the-arena-allocator)”). `MemoryContexts` are a chunk of free memory from which `palloc` takes memory. It is then freed all at once at some point.

The problem is that leaks can happen during the lifetime of the `MemoryContext`. And Valgrind can’t easily tell you about these leaks.

Postgres does have a useful builtin method to dump memory context info. For example you can add this line to any function, or you can execute this from gdb:

```
MemoryContextStats(TopMemoryContext);
```

And you will get information about allocations in the memory context and all child memory contexts, written to stderr. By observing the increase in allocations printed by `MemoryContextStats` over time, and by adding additional memory contexts, you may be able to discover the location of a leak.

But adding this call explicitly or via `gdb` is pretty manual. It would be much more convenient if there was some system that could automatically tell us about growing memory allocations.

Let’s introduce a leak into Postgres and see what we can do to discover it, reenacting a situation wherein we don’t know where this leak is.

## Setup

First grab Postgres, checkout release 16.2 and build it:

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

These flags enable Valgrind, enable debug mode, and set the build up to install locally rather than globally.

Once you’ve built it, you can create a Postgres database:

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

And then start the Postgres server in the foreground:

```
$ ./build/bin/postgres --config-file=$(pwd)/test-db/postgresql.conf -D $(pwd)/test-db -k $(pwd)/test-db
2024-03-21 16:08:10.984 EDT [213083] LOG:  starting PostgreSQL 16.2 on aarch64-unknown-linux-gnu, compiled by gcc (GCC) 13.2.1 20231205 (Red Hat 13.2.1-6), 64-bit
2024-03-21 16:08:10.986 EDT [213083] LOG:  listening on IPv6 address "::1", port 5432
2024-03-21 16:08:10.986 EDT [213083] LOG:  listening on IPv4 address "127.0.0.1", port 5432
2024-03-21 16:08:10.987 EDT [213083] LOG:  listening on Unix socket "/home/phil/tmp/postgres/test-db/.s.PGSQL.5432"
2024-03-21 16:08:10.989 EDT [213086] LOG:  database system was shut down at 2024-03-21 16:07:20 EDT
2024-03-21 16:08:10.992 EDT [213083] LOG:  database system is ready to accept connections

```

In a second terminal, connect to it with our built copy of `psql`:

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

Great! Now let’s introduce a leak.

## Introducing a leak

The function `PostgresMain()` in `src/backend/tcop/postgres.c` has a giant loop that looks tempting.

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

Let’s make this change:

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

Since we’re allocating in the main Postgres loop, every time a single connected client sends a statement on the same connection, we’ll allocate a new 4KB chunk.

Since we’re using `palloc`, all memory will be freed at the end of the process in a way Valgrind will think is valid. But during the lifetime of a single client, the Postgres backend process handling the connection will keep increasing in memory as the client sends statements.

This is a memory leak! In the extreme case, if a client sends 1 million statements, we’d allocate 4KB * 1,000,000 = 4GB in a single process and not free it until the process exits. This is a somewhat contrived case but this sort of situation does happen.

## Set up Valgrind wrapper

Let’s rebuild with our diff, remember that Valgrind has already been enabled.

```
$ make && make install
```

We still need to wrap Postgres in Valgrind. Tom Lane [shared](https://www.postgresql.org/message-id/159904.1608307376%40sss.pgh.pa.us) his wrapper script for this on the Postgres mailing list, which I’ve adapted. Add this to `postgres.valgrind` in the root directory of the Postgres repo:

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

Sidenote: I tried to also run this Valgrind script with the --leak-check=full flag, but Postgres would always crash when I had this flag. I don’t know why.

Now rename the current Postgres binary to `postgres.orig`, and copy this script as the new Postgres binary:

```
$ mv build/bin/postgres build/bin/postgres.orig
$ chmod +x postgres.valgrind
$ cp postgres.valgrind build/bin/postgres

```

Start the server (no need to re-`initdb`):

```
$ ./build/bin/postgres --config-file=$(pwd)/test-db/postgresql.conf -D $(pwd)/test-db -k $(pwd)/test-db
2024-03-21 16:18:28.233 EDT [216336] LOG:  starting PostgreSQL 16.2 on aarch64-unknown-linux-gnu, compiled by gcc (GCC) 13.2.1 20231205 (Red Hat 13.2.1-6), 64-bit
2024-03-21 16:18:28.234 EDT [216336] LOG:  listening on IPv6 address "::1", port 5432
2024-03-21 16:18:28.234 EDT [216336] LOG:  listening on IPv4 address "127.0.0.1", port 5432
2024-03-21 16:18:28.234 EDT [216336] LOG:  listening on Unix socket "/home/phil/tmp/postgres/test-db/.s.PGSQL.5432"
2024-03-21 16:18:28.236 EDT [216336] LOG:  database system was shut down at 2024-03-21 16:18:00 EDT
2024-03-21 16:18:28.238 EDT [216336] LOG:  database system is ready to accept connections

```

And connect with `psql` like before:

```
$ ./build/bin/psql -h localhost postgres
psql (16.2)
Type "help" for help.

postgres=#

```

You’ll immediately see a log line in the Postgres server process:

```
4096 from 216344
```

And this was the leak we introduced. If we run `SELECT 1` a number of times in that same `psql` process:

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

We’ll see the same number of print statements in the server logs:

```
... omitted ...
2024-03-21 16:18:28.238 EDT [216336] LOG:  database system is ready to accept connections
4096 from 216344
4096 from 216344
4096 from 216344
4096 from 216344
4096 from 216344

```

Now if we Ctrl-c on the server process, we can look at Valgrind logs.

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

It only found 410 bytes definitely lost. There is no way to account for the accumulation of 4096*5 = 20,480 bytes lost with every client statement.

Moreover, Valgrind is very slow. The Valgrind [manual](https://valgrind.org/docs/manual/valgrind_manual.pdf) says:

> Your program will run much slower (eg. 20 to 30 times) than normal, and use a lot more memory.

We’ll have to try something else.

## AddressSanitizer, LeakSanitizer

Valgrind has been around since 2002\. But around 2013, Google contributed somewhat similar functionality directly into `clang` and later `gcc`. The history is a little difficult for me to follow. AddressSanitizer and LeakSanitizer may have been independent at one point but now AddressSanitizer contains LeakSanitizer. To get the sanitizer, you simply compile your code with `-fsanitize=address`.

Let’s go back to that simple, leaky `main.c` program:

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

And compile and run with the sanitizers on:

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

Hey, that’s perfect. And even simpler than using Valgrind and requiring a wrapper. The leak detection is just built right into the binary.

Let’s try this with Postgres.

## Postgres and AddressSanitizer

Back in the Postgres git repo we cloned earlier, remember we have this diff applied that will leak memory with every client statement.

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

Run `make clean` and autoconf again but this time with `-fsanitize=address` in the `CFLAGS` value. Then rebuild.

```
$ make clean
$ ./configure \
    CFLAGS="-ggdb -Og -g3 -fno-omit-frame-pointer -fsanitize=address" \
    --prefix=$(pwd)/build \
    --libdir=$(pwd)/build/lib \
    --enable-debug
$ make && make install

```

Now run the Postgres server again:

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

Ok it immediately crashed! That’s interesting.

But looking closely, the messages are 1) not related to the 4096-byte leak we introduced and 2) don’t really seem like leaks that matter since these are related to program initialization. Since they’re not in a loop the allocations shown won’t keep growing over time. So not a huge deal.

Where was our leak? It turns out the default behavior of AddressSanitizer/LeakSanitizer is to crash on the first error. In this situation, and likely many production systems that only later introduce sanitizers, that isn’t useful. Thankfully the sanitizers have some runtime options we can set in environment variables.

Let’s set the environment variable `ASAN_OPTIONS="halt_on_error=false"` and re-run Postgres:

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

Ok! It reported a leak but it kept going. Let’s trigger the leak we added by connecting with `psql` like before.

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

And exit the connection. You may see a leak report for the `psql` process itself. But the leak we are looking for is in the Postgres backend process.

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

But nothing about our leak! I guess the sanitizers work similarly to Valgrind in this regard, since the memory was freed by the time the process ended. Just not before.

I got stumped for a while at this point.

## eBPF and bcc tools

I’ve been reading Brendan Gregg’s [Systems Performance](https://www.brendangregg.com/blog/2020-07-15/systems-performance-2nd-edition.html) book, so I started wondering if there was a way to use [perf](https://perf.wiki.kernel.org/index.php/Main_Page) or [bpf profile](https://github.com/iovisor/bcc/blob/master/tools/profile.py) to record the memory usage of a running program and see where the allocations were coming from. I have used `perf` for profiling *functions* but not for profiling *memory usage*. I looked around for how you might record allocations with a stack trace but couldn’t figure it out.

So I looked into `bpf profile` and noticed Brendan Gregg’s [page](https://www.brendangregg.com/FlameGraphs/memoryflamegraphs.html) on memory leaks. Turns out there’s a tool for identifying memory leaks directly, [memleak.py](https://github.com/iovisor/bcc/blob/master/tools/memleak.py)!

To use it, [install](https://github.com/iovisor/bcc/blob/master/INSTALL.md) the bcc tools suite if you don’t have it already. Unfortunately each distribution changes the names of the programs so it may be easiest to use `find`:

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

Ok, so it’s at `/usr/share/bcc/tools/memleak`.

Let’s rebuild Postgres without sanitizers on:

```
$ make clean
$ ./configure \
    CFLAGS="-ggdb -Og -g3 -fno-omit-frame-pointer" \
    --prefix=$(pwd)/build \
    --libdir=$(pwd)/build/lib \
    --enable-debug
$ make && make install

```

Run the Postgres server:

```
$ ./build/bin/postgres --config-file=$(pwd)/test-db/postgresql.conf -D $(pwd)/test-db -k $(pwd)/test-db
2024-03-22 11:36:06.364 EDT [245850] LOG:  starting PostgreSQL 16.2 on aarch64-unknown-linux-gnu, compiled by gcc (GCC) 13.2.1 20231205 (Red Hat 13.2.1-6), 64-bit
2024-03-22 11:36:06.364 EDT [245850] LOG:  listening on IPv6 address "::1", port 5432
2024-03-22 11:36:06.364 EDT [245850] LOG:  listening on IPv4 address "127.0.0.1", port 5432
2024-03-22 11:36:06.365 EDT [245850] LOG:  listening on Unix socket "/home/phil/tmp/postgres/test-db/.s.PGSQL.5432"
2024-03-22 11:36:06.368 EDT [245853] LOG:  database system was shut down at 2024-03-22 11:36:05 EDT
2024-03-22 11:36:06.370 EDT [245850] LOG:  database system is ready to accept connections
```

And start `psql` and grab the associated backend process PID that is handling our request on the server:

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

Open a new terminal session, keeping alive that `psql` connection and the Postgres server process. And run the `memleak` program:

```
$ sudo /usr/share/bcc/tools/memleak -p 245858
Attaching to pid 245858, Ctrl+C to quit.
[11:37:19] Top 10 stacks with outstanding allocations:
[11:37:24] Top 10 stacks with outstanding allocations:

```

It will start reporting leaks every five seconds.

Let’s trigger the leak more aggressively by continuing to run `SELECT 1` statements in the `psql` session:

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

After running this `SELECT 1` command 20-30 times or so, you should see the memleak program report:

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

And *that* is the memory leak, with a beautiful stack trace to help us find exactly where it is.