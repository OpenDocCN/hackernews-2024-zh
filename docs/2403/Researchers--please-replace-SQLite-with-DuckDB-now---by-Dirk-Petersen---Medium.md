<!--yml

category: 未分类

date: 2024-05-27 15:01:17

-->

# 研究人员，请立即使用 DuckDB 替代 SQLite | 作者 Dirk Petersen | Medium

> 来源：[https://dirk-petersen.medium.com/researchers-please-replace-sqlite-with-duckdb-now-f038044a2702](https://dirk-petersen.medium.com/researchers-please-replace-sqlite-with-duckdb-now-f038044a2702)

# 研究人员，请立即使用 DuckDB 替代 SQLite

## 如果您是一个有计算工作负载的研究人员，您可能会在某些任务中使用 SQLite。请立即放弃它，转而使用 DuckDB，它更快速且更易于使用。

您是一名研究人员，可能从事生命或社会科学，您的组织为您提供了访问高性能机器的权限，该机器拥有多个 CPU 和大量内存，可能作为 CPU 群集的一部分。您熟悉 Linux，并使用 Python/Pandas 和/或 R 来处理和分析数据。有时，您会使用 SQLite，例如对大型数据集进行预过滤，也作为长期存储数据集的选项。

一些同事和 IT 专家告诉您，SQLite 是一个仅用于测试的玩具数据库引擎（这是不正确的），并建议您切换到速度更快、更成熟的 PostgreSQL 或其他数据库。但您喜欢 SQLite 不需要自己的服务器，您可以在项目文件夹中创建一个 SQL 数据库，并与您正在使用的所有其他文件一起使用。

不要改变这种方法，即使现代计算机上的现代 PostgreSQL 速度更快。相反，您应该从 SQLite 切换到 DuckDB，因为它更快且更易于使用；这些是其主要优点：

+   DuckDB 从设计之初就考虑了在机器的所有 CPU 核心上进行利用。

+   DuckDB 优化了复杂查询，而 SQLite 和大多数其他 SQL 数据库更倾向于同时写入多个数据集。您可以在[这里](https://medium.com/scalecapacity/columnar-database-and-row-database-how-are-they-different-2efa8e4c38a3)详细了解。

+   DuckDB 支持多种快速数据格式，并可以并行读取多个数据库文件。

我们使用一台中等配置的 32 核服务器（Intel Gold 6326 CPU @ 2.90GHz），内存为 768 GB，并配备快速本地闪存存储。该系统连接到一个快速的共享 Posix 文件系统，读写吞吐量超过 4GB/s，如[scratch-dna benchmark](https://www.delltechnologies.com/asset/en-au/products/storage/industry-market/white-paper-esg-technical-review-performance-testing-onefs-google-cloud.pdf)所示。

```
[dp@node]$ scratch-dna -v -p 4 1024 104857600 3 ./gscratch

Building random DNA sequence of 100.0 MB...
Writing 1024 files with filesizes between 100.0 MB and 300.0 MB...
Files Completed:      21, Data Written:   3.3GiB, Files Remaining:    1005, Cur FPS:    21, Throughput: 3400 MiB/s
Files Completed:      44, Data Written:   7.8GiB, Files Remaining:     982, Cur FPS:    22, Throughput: 4000 MiB/s
Files Completed:      68, Data Written:  12.7GiB, Files Remaining:     958, Cur FPS:    22, Throughput: 4333 MiB/s
Files Completed:      89, Data Written:  16.7GiB, Files Remaining:     937, Cur FPS:    22, Throughput: 4275 MiB/s
```

以及至少 1.2 GB/s 读写吞吐量的本地闪存磁盘

```
[dp@node]$ scratch-dna -v -p 4 1024 104857600 3 $TMPDIR

Building random DNA sequence of 100.0 MB...
Writing 1024 files with filesizes between 100.0 MB and 300.0 MB...
Files Completed:       4, Data Written:   1.1GiB, Files Remaining:    1022, Cur FPS:     4, Throughput: 1100 MiB/s
Files Completed:      12, Data Written:   2.6GiB, Files Remaining:    1014, Cur FPS:     6, Throughput: 1350 MiB/s
Files Completed:      19, Data Written:   3.6GiB, Files Remaining:    1007, Cur FPS:     6, Throughput: 1233 MiB/s
```

现在，让我们使用一个合理大小的 CSV 文件来测试 SQLite 和 DuckDB — 不要太大，但也足够大以展示差异。如果您从事研究工作，可能会访问连接到 Linux 机器的快速 POSIX 文件系统。此文件系统可能包含许多百万或十亿个文件，我们希望利用这些信息来分析文件的元数据，如大小、文件类型等。您可以使用 [pwalk](https://github.com/fizwit/filesystem-reporting-tools) 生成一个包含您文件夹元数据的 CSV 文件，然后使用 'iconv' 确保它具有正确的格式以便与数据库引擎一起使用。

```
pwalk --NoSnap --header "/your/dept-folder" > dept-folder-tmp.csv
iconv -f ISO-8859-1 -t UTF-8 dept-folder-tmp.csv > dept-folder.csv
```

## 测试 SQLite

让我们将这个 CSV 文件导入 SQLite。我们将使用独立的 SQLite 版本 3.38 和 Python 版本 3.10，它带有 SQLite 版本 3.39。此外，我们将使用 Pandas 作为辅助工具，以检测 CSV 文件的列是数字、文本还是日期 — 这是 SQLite 缺乏的功能。Pandas 在内存中操作，并可以直接将其表插入 SQLite 数据库。但是，Pandas 不随 Python 一起提供，需要单独安装。

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

然后我们在共享文件系统中执行脚本，这需要超过43分钟。

```
time ./sqlite-import.py

real    43m25.215s
user    28m23.660s
sys     15m2.266s 
```

那真是太多了……但所有列似乎都已正确导入：

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

现在，让我们在我们的共享 POSIX 文件系统上运行一个简单的查询，检查是否已插入所有 2.83 亿条记录。我们不需要 Python 来执行此操作；它足够简单，我们可以在 Bash shell 中执行它。我们只需计算表中记录的数量：

```
time sqlite3 ./db/x-dept.db "select count(*) from meta_table;
283587401
real 20m30.382s
user 0m14.746s
sys 1m41.663s 
```

超过20分钟相当长，我们的快速共享文件系统可能出了问题？让我们将数据库复制到本地磁盘，然后再试一次：

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

好吧，既然我们复制了文件，它可能已被缓存，但这本身不能解释差异。即使在缓存的情况下，从共享文件系统重新运行数据也更快，但仍然很慢，需要将近11分钟：

```
time sqlite3 ./db/x-dept.db "select count(*) from meta_table;
283587401
real    10m44.268s
user    0m12.983s
sys     1m40.303s
```

让我们回到本地文件并运行另一个简单但稍微复杂的查询。我们想知道所有文件的总磁盘消耗量（以字节为单位），并生成 'st_size' 列的 sum()：

```
time sqlite3 $TMPDIR/x-dept.db "select sum(st_size) from meta_table;"
591263908311685

real    1m40.343s
user    0m37.363s
sys     1m2.965s
```

稍微复杂的查询所花费的时间比简单查询长了超过50%，这是合理的，并返回约 538 TiB（591263908311685 字节）的磁盘消耗量

## 测试 DuckDB

我们注意到的第一件事是，DuckDB 可以直接使用 CSV 文件而无需先导入它们。我们可以使用 'read_csv_auto' 函数自动确定每列中的数据类型：

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

在一个 CSV 文件上直接运行 SQL 查询花了3分钟，并且我们可以看到 CPU 利用率超过 800%，这意味着 8-9 个 CPU 核心在忙碌。我们还看到 DuckDB 有一个漂亮的进度条。

现在，直接使用CSV文件的工作速度较慢，因为在使用这种数据交换格式时无法优化查询，但我们可以通过原始计算能力来抵消这一点。然而，让我们探索一下其他选择。我们可以将CSV文件导入到可能非常快速的本地DuckDB格式...但是DuckDB相当新，其他工具将无法读取DuckDB格式，因此让我们检查是否支持其他格式。我们注意到Apache Arrow正在获得很多人气，但我们也看到了受欢迎的旧Parquet格式，可以被许多工具读取。让我们试试这个：

```
time duckdb -s "COPY (SELECT * FROM \
     read_csv_auto('./db/x-dept.csv') TO './db/x-dept.parquet';"

real    5m9.249s
user    31m56.600s
sys     5m43.471s
```

好吧，这个转换花了超过5分钟，这还算不错。现在让我们看看我们可以如何处理‘x-dept.parquet’。如果我们通过共享的POSIX文件系统读取它，让我们运行我们之前的查询：

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

哎呀，0.6秒而不是20多分钟？在共享文件系统上快了2000倍，在本地闪存/SSD上与SQLite比较快了100多倍！而且我们甚至还没有使用DuckDB的内部数据库格式。让我们通过我们的第二个查询确认这些结果：

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

同样令人印象深刻。现在，让我们尝试一些更复杂的内容。我们想知道每种文件类型的百分比份额（由文件扩展名识别），以及每种文件类型占用的总磁盘空间。在这种情况下，我们将SQL语句放在一个名为‘extension-summary.sql’的文本文件中：

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

然后我们让DuckDB对我们的Parquet文件运行它：

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

这是在不同的服务器上执行的，以确保缓存是冷的。3.3秒是令人印象深刻的，但显然，我们需要找到一个更复杂的查询来测试DuckDB的真正能力。让我们尝试识别所有可能重复的文件，因为它们具有相同的名称、文件大小和修改日期，但存储在不同的目录中。我们对SQL并不是很了解，所以让我们向ChatGPT寻求帮助。鉴于DuckDB兼容PostgreSQL语法，一旦我们描述了表结构并解释了我们想要实现的目标，它就提供了一个合适的解决方案，然后我们将其复制到‘dedup.sql’文本文件中。

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

在这种情况下，13个CPU核心运行了22秒来执行一个相当复杂的查询。将parquet文件复制到本地磁盘后重新运行此查询，结果如下：

```
time duckdb < dedup-local.sql
real    0m21.838s
user    2m30.352s
sys     1m34.448s
```

换句话说，共享文件系统和本地闪存磁盘之间没有性能差异。

现在让我们再次与SQLite和我们的文件‘extentions-summary.sql’进行比较：

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

好吧，6分钟15秒对比3.3秒意味着DuckDB比SQLite至少快113倍，至少在这个例子中是这样。

我们复杂的‘dedup.sql’查询的表现如何？在再次询问ChatGPT将兼容PostgreSQL的查询转换为SQLite的查询后，我们可以执行它：

```
time cat dedup.sql | sqlite3 $TMPDIR/x-dept.db
real    5m20.571s
user    2m39.911s
sys     1m23.271s
```

在这种情况下，DuckDB只比SQLite快15倍，性能差异可能主要归因于DuckDB使用的多个CPU核心。

## 最后，谈一下通配符

研究数据集通常分散在多个文件中。这有时是因为数据集过大而无法存储在单个文件中；另一些情况下，是因为全球和分布式工作的研究人员管理自己的数据集，数据仅需偶尔合并以进行分析。在许多数据库系统中，这意味着您可能有多个需要使用“union”查询合并的表，但使用DuckDB，这个过程非常简单：假设您有三个文件：‘./data/prj-asia.csv’、‘./data/prj-africa.csv’和‘./data/prj-europe.csv’，所有文件具有相同的模式（列结构）。您可以简单地使用通配符将所有三个文件读取为单个表：

```
duckdb -c "select count(*) from read_csv_auto('./data/prj-*.csv');"
```

## DuckDB vs SQLite在分析中的优势总结

+   DuckDB在某些情况下比SQLite快得多

+   DuckDB具有更多内置的数据导入功能，无需外部Python包

+   DuckDB在共享的Posix文件系统中没有任何性能瓶颈，这在大多数研究环境中是常见的

+   DuckDB使用PostgreSQL语法，这是数据科学家和新的开源数据库项目中最普遍的SQL行话

+   DuckDB内置支持写入和读取Parquet和Apache Arrow数据格式

+   Python和R包是DuckDB项目的一个重要组成部分

+   通配符的支持允许研究人员并行处理多个文件

## 在应继续使用SQLite的情况下

SQLite是地球上使用最多的数据库之一，也是最稳定的之一，数百万智能手机应用程序在内部使用它。有许多使用SQLite的好理由，这些理由在[SQLite网站](https://www.sqlite.org/whentouse.html)上已经详细说明。研究人员喜欢SQLite因为其灵活性和无需数据库服务器的特性。然而，由于DuckDB在支持这种用例方面做得更好，研究人员应考虑立即转换。

## 其他资源

+   我希望很快写一篇关于DuckDB的第二篇文章，现在更加关注DuckDB在高性能计算（HPC）系统中的使用。这篇文章将讨论通配符功能。

+   有一篇类似的[数据科学家文章在这里](https://towardsdatascience.com/forget-about-sqlite-use-duckdb-instead-and-thank-me-later-df76ee9bb777)，但那篇文章使用了更多技术性语言，我觉得有必要写一些更为包容所有研究人员的内容。

+   如果您有兴趣使用更专业和功能丰富的工具分析您的文件系统元数据，您应考虑[https://starfishstorage.com/](https://starfishstorage.com/)
