<!--yml
category: 未分类
date: 2024-05-27 15:01:17
-->

# Researchers, please replace SQLite with DuckDB now | by Dirk Petersen | Medium

> 来源：[https://dirk-petersen.medium.com/researchers-please-replace-sqlite-with-duckdb-now-f038044a2702](https://dirk-petersen.medium.com/researchers-please-replace-sqlite-with-duckdb-now-f038044a2702)

# Researchers, please replace SQLite with DuckDB now

## If you are a researcher with computational workloads, you might be using SQLite for some tasks. Please drop it now and switch to DuckDB, it is much faster and easier to use

You are a researcher, potentially in the life or social sciences, and your organization offers you access to a fast machine with many CPUs and lots of memory, potentially as part of a CPU cluster. You are familiar with Linux and use Python/Pandas and/or R to manipulate and analyze your data. You sometimes use SQLite, for example, to pre-filter large datasets but also as an option for storing datasets longer term.

Some colleagues and IT folks have told you that SQLite is a toy database engine for testing (which isn’t true) and that you should switch to something faster and more mature, such as PostgreSQL or other databases. But you like the fact that SQLite does not require its own server, and you can just create a SQL database in your project folder along with all the other files you are using.

Don’t change that approach, even though a modern PostgreSQL on a modern computer will be faster. Instead, you should switch from SQLite to DuckDB because it is much faster and easier to use; these are the main benefits:

*   DuckDB is designed from day one to use all the CPU cores in your machine
*   DuckDB is optimized for complex queries, while SQLite and most other SQL databases are more optimized for writing multiple datasets at the same time. You can read up on the [details here](https://medium.com/scalecapacity/columnar-database-and-row-database-how-are-they-different-2efa8e4c38a3).
*   DuckDB supports multiple fast data formats out of the box and can read many database files in parallel.

As test system we use a middle of the road 32 core server (Intel Gold 6326 CPU @ 2.90GHz) with 768 GB Memory and fast local flash storage. The system is connected to a fast shared Posix file system with more than 4GB/s read/write throughput as shown by the [scratch-dna benchmark](https://www.delltechnologies.com/asset/en-au/products/storage/industry-market/white-paper-esg-technical-review-performance-testing-onefs-google-cloud.pdf).

```
[dp@node]$ scratch-dna -v -p 4 1024 104857600 3 ./gscratch

Building random DNA sequence of 100.0 MB...
Writing 1024 files with filesizes between 100.0 MB and 300.0 MB...
Files Completed:      21, Data Written:   3.3GiB, Files Remaining:    1005, Cur FPS:    21, Throughput: 3400 MiB/s
Files Completed:      44, Data Written:   7.8GiB, Files Remaining:     982, Cur FPS:    22, Throughput: 4000 MiB/s
Files Completed:      68, Data Written:  12.7GiB, Files Remaining:     958, Cur FPS:    22, Throughput: 4333 MiB/s
Files Completed:      89, Data Written:  16.7GiB, Files Remaining:     937, Cur FPS:    22, Throughput: 4275 MiB/s
```

and a local flash disk with at least 1.2 GB/s read/write throughput

```
[dp@node]$ scratch-dna -v -p 4 1024 104857600 3 $TMPDIR

Building random DNA sequence of 100.0 MB...
Writing 1024 files with filesizes between 100.0 MB and 300.0 MB...
Files Completed:       4, Data Written:   1.1GiB, Files Remaining:    1022, Cur FPS:     4, Throughput: 1100 MiB/s
Files Completed:      12, Data Written:   2.6GiB, Files Remaining:    1014, Cur FPS:     6, Throughput: 1350 MiB/s
Files Completed:      19, Data Written:   3.6GiB, Files Remaining:    1007, Cur FPS:     6, Throughput: 1233 MiB/s
```

Now, let’s test SQLite and DuckDB with a reasonably sized CSV file — nothing too huge, but also large enough to demonstrate the difference. If you work in research, you will likely have access to a fast POSIX filesystem connected to your Linux machine. This filesystem may contain many millions or billions of files, and we would like to harvest that information to do some analysis of your file metadata, such as sizes, file types, etc. You can use [pwalk](https://github.com/fizwit/filesystem-reporting-tools) to generate a CSV file of your folder metadata and then ensure with ‘iconv’ that it has the right format to be used with a database engine.

```
pwalk --NoSnap --header "/your/dept-folder" > dept-folder-tmp.csv
iconv -f ISO-8859-1 -t UTF-8 dept-folder-tmp.csv > dept-folder.csv
```

## Testing SQLite

Let’s import this CSV file into SQLite. We’ll use standalone SQLite version 3.38 and Python version 3.10, which comes with SQLite version 3.39\. Additionally, we’ll use Pandas as a helper to detect if the columns of the CSV files are numbers, text, or dates — a functionality SQLite lacks. Pandas operates in memory and can insert its table directly into a SQLite database. However, Pandas does not come with Python and needs to be installed separately.

```
import pandas, sqlite3

# File paths
csv_file_path = './db/x-dept.csv'
sqlite_db_path = './db/x-dept.db'

# Read CSV file into dataframe and copy the dataframe to SQLite
df = pandas.read_csv(csv_file_path)

# Connect to SQLite database (it will be created if it doesn’t exist)
conn = sqlite3.connect(sqlite_db_path)

# Insert data from Pandas Data Frame to a new 'meta_table' in sqlite db 
df.to_sql('meta_table', conn, index=False, if_exists='replace')

# Commit and close
conn.commit()
conn.close()
```

Then we execute the script in our shared file system and it takes more than 43 minutes.

```
time ./sqlite-import.py

real    43m25.215s
user    28m23.660s
sys     15m2.266s 
```

That is a lot ….. but all the columns seem to have been imported correctly:

```
sqlite3 ./db/x-dept.db ".schema"

CREATE TABLE IF NOT EXISTS "meta_table" (
  "inode" INTEGER,
  "parent-inode" INTEGER,
  "directory-depth" INTEGER,
  "filename" TEXT,
  "fileExtension" TEXT,
  "UID" INTEGER,
  "GID" INTEGER,
  "st_size" INTEGER,
  "st_dev" INTEGER,
  "st_blocks" INTEGER,
  "st_nlink" INTEGER,
  "st_mode" INTEGER,
  "st_atime" INTEGER,
  "st_mtime" INTEGER,
  "st_ctime" INTEGER,
  "pw_fcount" INTEGER,
  "pw_dirsum" INTEGER
);
```

Now, let’s run a simple query on our shared POSIX file system to check if all 283 million records were inserted. We don’t need Python for this; it’s simple enough that we can just execute it in our Bash shell. We just count the number of records in the table:

```
time sqlite3 ./db/x-dept.db "select count(*) from meta_table;
283587401
real 20m30.382s
user 0m14.746s
sys 1m41.663s 
```

More than 20 min is quite long, there may be something wrong with our fast shared filesystem? Let’s copy the database to local disk and try again:

```
time cp ./db/x-dept.db $TMPDIR/

real    1m19.890s
user    0m0.024s
sys     1m16.456s

time sqlite3 $TMPDIR/x-dept.db "select count(*) from meta_table;"
283587401

real    1m3.460s
user    0m6.061s
sys     0m57.392s
```

Well, since we’ve copied the file, it may have been cached, but that alone cannot explain the difference. Re-running the data from the shared file system, even when cached, is faster, but it’s still slow and takes almost 11 minutes:

```
time sqlite3 ./db/x-dept.db "select count(*) from meta_table;
283587401
real    10m44.268s
user    0m12.983s
sys     1m40.303s
```

Let’s return to our local file and run another simple yet slightly more complex query. We’d like to know the total disk consumption of all files in bytes and generate a sum() of the ‘st_size’ column:

```
time sqlite3 $TMPDIR/x-dept.db "select sum(st_size) from meta_table;"
591263908311685

real    1m40.343s
user    0m37.363s
sys     1m2.965s
```

The slightly more complex query takes more than 50% longer than the simple one which makes sense and returns about 538 TiB (591263908311685 bytes) of disk consumption

## Testing DuckDB

One of the first things we notice is that DuckDB can use CSV files directly without importing them first. We can use the ‘read_csv_auto’ function to automatically figure out the data types in each column.:

```
time duckdb -c "select count(*) from read_csv_auto('./db/x-dept.csv');"
100% ▕████████████████████████████████████████████████████████████▏
┌──────────────┐
│ count_star() │
│    int64     │
├──────────────┤
│    283587401 │
└──────────────┘

real    3m16.446s
user    18m11.720s
sys     2m53.538s
```

It took 3 minutes to run a sql query directly on a CSV file and we can see that the CPU utilization is more than 800% which means that 8–9 CPU cores were busy. We also see that DuckDB has a nice progress bar.

Now, working with CSV files directly is slow, as queries cannot be optimized when using this data interchange format, but we can counteract that with raw compute power. However, let’s explore other options we have. We can import the CSV file into a native DuckDB format that is probably very fast… but DuckDB is quite new, and no other tools will be able to read the DuckDB format, so let’s check if other formats are supported. We notice Apache Arrow, which is gaining a lot of popularity, but we also see the good old Parquet format, which can be read by many tools out there. Let’s try that:

```
time duckdb -s "COPY (SELECT * FROM \
     read_csv_auto('./db/x-dept.csv') TO './db/x-dept.parquet';"

real    5m9.249s
user    31m56.600s
sys     5m43.471s
```

OK, this conversion took just over 5 minutes, which is not bad. Now let’s see what we can do with ‘x-dept.parquet’. If we read it through our shared POSIX file system, let’s run our previous queries:

```
time duckdb -c "select count(*) from './db/x-dept.parquet';"

┌──────────────┐
│ count_star() │
│    int64     │
├──────────────┤
│    283587401 │
└──────────────┘

real    0m0.641s
user    0m0.578s
sys     0m0.155s
```

Oops, 0.6 seconds instead of more than 20 minutes ? That is 2000 times faster on a shared filesystem and more than 100 times faster when comparing to SQLite on local flash/SSD! And we did not even use DuckDB’s internal database format yet. Let’s confirm these results with our second query:

```
time duckdb -c "select sum(st_size) from './db/CEDAR.parquet';"
┌─────────────────┐
│  sum(st_size)   │
│     int128      │
├─────────────────┤
│ 591263908311685 │
└─────────────────┘

real    0m1.559s
user    0m6.254s
sys     0m1.877s
```

Equally impressive. Now, let’s try something a bit more complex. We would like to know the percentage share of each file type, as identified by the file extension, as well as the total disk space each file type consumes. In this case, we’ll put the SQL statement in a text file called ‘extension-summary.sql’ :

```
WITH TotalSize AS (
    SELECT SUM(st_size) AS totalSize
    FROM './db/x-dept.parquet'
)
SELECT
    fileExtension as fileExt,
    ROUND((SUM(st_size) * 100.0 / totalSize), 1) AS pct,
    ROUND((SUM(st_size) / (1024.0 * 1024.0 * 1024.0)), 3) AS GiB
FROM
    './db/x-dept.parquet',
    TotalSize
GROUP BY
    fileExtension, totalSize
HAVING
    (SUM(st_size) * 100.0 / totalSize) > 0.1
ORDER BY
    pct DESC
LIMIT 6
```

and then we ask DuckDB to run it against our Parquet file:

```
time duckdb < extension-summary.sql 

┌─────────┬────────┬────────────┐
│ fileExt │  pct   │    GiB     │
│ varchar │ double │   double   │
├─────────┼────────┼────────────┤
│ gz      │   23.4 │  128928.05 │
│ bam     │   20.9 │ 114925.732 │
│ czi     │   14.1 │  77548.979 │
│ fq      │    7.6 │  41954.356 │
│ sam     │    5.1 │  27812.136 │
│ tif     │    5.0 │   27457.25 │
├─────────┴────────┴────────────┤
│  6 rows             3 columns │
└───────────────────────────────┘

real    0m3.320s
user    0m24.991s
sys     0m4.127s
```

This was executed on a different server from the previous one to ensure that the cache was cold. 3.3 seconds is impressive, but obviously, we need to find an even more complex query to test the true capabilities of DuckDB. Let’s try to identify all files that may be duplicates, as they have the same name, file size, and modification date, but are stored in different directories. We don’t really have in-depth knowledge of SQL, so let’s ask ChatGPT for some help. Given that DuckDB is compatible with PostgreSQL syntax, once we describe the table structure and explain what we want to achieve, it provides a suitable solution that we then copy into the ‘dedup.sql’ text file.

```
SELECT
    -- Extract the filename without path
    SUBSTRING(
        filename FROM LENGTH(filename) - POSITION('/' IN REVERSE(filename)) + 2
        FOR
        LENGTH(filename) - POSITION('/' IN REVERSE(filename)) - POSITION('.' IN REVERSE(filename)) + 1
    ) AS plain_file_name,
    st_mtime,
    st_size,
    COUNT(*) as duplicates_count,
    ARRAY_AGG(filename) as duplicate_files  -- Collect the full paths of all duplicates
FROM
    './db/x-dept.parquet'
WHERE
    filename NOT LIKE '%/miniconda3/%' AND
    filename NOT LIKE '%/miniconda2/%' AND    -- let's ignore all that miniconda stuff
    st_size > 1024*1024                       -- we only look at files > 1 MB
GROUP BY
    plain_file_name,
    st_mtime,
    st_size
HAVING
    COUNT(*) > 1  -- Only groups with more than one file are duplicates
ORDER BY
    duplicates_count DESC;
```

```
time duckdb < dedup.sql
real    0m21.780s
user    2m40.679s
sys     1m34.142s
```

In this case 13 CPU cores ran for 22 seconds to execute a pretty complex query. Rerunning this query after copying the parquet file to a local disk gives us this:

```
time duckdb < dedup-local.sql
real    0m21.838s
user    2m30.352s
sys     1m34.448s
```

In other words, there is no performance difference between the shared file system and the local flash disk.

Now let’s compare this again with SQLite and our file ‘extentions-summary.sql’:

```
time cat extension_summary.sql | sqlite3 $TMPDIR/x-dept.db
gz|23.4|128928.05
bam|20.9|114925.732
czi|14.1|77548.979
fq|7.6|41954.356
sam|5.1|27812.136
tif|5.0|27457.25

real    6m15.232s
user    4m6.303s
sys     2m8.892s
```

OK, 6 minutes and 15 seconds versus 3.3 seconds means that DuckDB is 113 times faster than SQLite, at least in this example.

And how does our complex ‘dedup.sql’ query perform? After asking ChatGPT again to convert the PostgreSQL-compatible query to one that works with SQLite, we can execute it:

```
time cat dedup.sql | sqlite3 $TMPDIR/x-dept.db
real    5m20.571s
user    2m39.911s
sys     1m23.271s
```

In this case DuckDB is only 15 times faster than SQLite and the performance difference can perhaps be mostly attributed to the multiple cpu cores, that DuckDB uses.

## And finally, a word about wildcards

Research datasets are often dispersed across multiple files. This is sometimes because the datasets are too large to be stored in a single file; other times, it’s because researchers, working globally and distributed, manage their own datasets, and the data only needs to be merged occasionally for analytics purposes. In many database systems, this would mean you have multiple tables that would need to be combined with a ‘union’ query, but with DuckDB, this process is very straightforward: Imagine you had three files: ‘./data/prj-asia.csv’, ‘./data/prj-africa.csv’, and ‘./data/prj-europe.csv’, all with the same schema (column structure). You can simply use a wildcard to read all three files as a single table:

```
duckdb -c "select count(*) from read_csv_auto('./data/prj-*.csv');"
```

## Summary of Benefits of DuckDB vs SQLite for Analytics

*   DuckDB is much faster than SQLite, in some cases orders of magnitude
*   DuckDB has much more data import functionality built-in, no external python packages needed
*   DuckDB does not experience any performance bottlenecks with shared Posix filesystems which are common in most research environments
*   DuckDB uses PostgreSQL syntax, the most prevalent SQL slang among data scientists and new open source database projects
*   DuckDB has built-in support for writing and reading Parquet and Apache Arrow data formats
*   Python and R packages are integral part of the DuckDB project
*   Support of wildcards allows researchers to work with many files in parallel

## Where you should continue to use SQLite

SQLite is the most used database on planet Earth and is also one of the most stable, with millions of smartphone apps using it internally. There are many good reasons to use SQLite, and they are laid out on the [SQLite website](https://www.sqlite.org/whentouse.html). Researchers appreciate SQLite because of its flexibility and the fact that no database server is needed. However, as DuckDB does a better job of supporting this use case, researchers should consider switching today.

## Other resources

*   I hope to write a second article about DuckDB soon, now more geared towards use of DuckDB in High Performance Computing (HPC) systems. A discussion of the wildcard feature will be part of this.
*   There is a similar [article for data scientists here](https://towardsdatascience.com/forget-about-sqlite-use-duckdb-instead-and-thank-me-later-df76ee9bb777), but that one has more techie language and I felt the need to write something, that is more inclusive towards all researchers.
*   If you are interested in analyzing your filesystem metadata with a more professional and feature rich tool, you should consider [https://starfishstorage.com/](https://starfishstorage.com/)