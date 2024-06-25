<!--yml

分类: 未分类

日期: 2024-05-27 15:19:55

-->

# SQLite 存档文件

> 来源：[`www.sqlite.org/sqlar.html`](https://www.sqlite.org/sqlar.html)

# 1\. 简介

一个 "SQLite 存档" 是一个类似于 [ZIP 存档](https://en.wikipedia.org/wiki/Zip_(file_format)) 或 [Tarball](https://en.wikipedia.org/wiki/Tar_(computing)) 的文件容器，但基于一个 SQLite 数据库。

一个 SQLite 存档是一个普通的 SQLite 数据库文件，其中包含以下表作为其模式的一部分：

```
CREATE TABLE sqlar(
  name TEXT PRIMARY KEY,  -- name of the file
  mode INT,               -- access permissions
  mtime INT,              -- last modification time
  sz INT,                 -- original file size
  data BLOB               -- compressed content
);

```

SQLAR 表的每一行保存单个文件的内容。文件名（相对于存档根目录的完整路径名）位于 "name" 字段中。"mode" 字段是一个整数，表示文件的 Unix 风格的访问权限。"mtime" 是文件的修改时间，以自 1970 年以来的秒数表示。"sz" 是文件的原始未压缩大小。"data" 字段包含文件内容。内容通常使用 [Deflate](http://zlib.net/) 进行压缩，尽管不一定总是这样。如果 "sz" 字段的大小等于 "data" 字段的大小，则内容以未压缩形式存储。

## 1.1\. 数据库作为容器对象

一个 SQLite 存档是 SQLite 数据库可以作为包含许多较小数据组件的容器对象的更一般概念的一个例子。

对于像 PostgreSQL 或 Oracle 这样的客户端/服务器数据库，用户和开发人员倾向于将数据库视为服务或 "节点"，而不是作为对象。这是因为数据库内容分布在服务器上的多个文件中，或者可能分布在服务集群中的多个服务器上。不能指向单个文件甚至单个目录并说 "这就是数据库"。

相比之下，SQLite 将所有内容存储在单个磁盘文件中。这个单个文件是你可以指向并说 "这就是数据库" 的东西。它的行为就像一个对象。SQLite 数据库文件可以被复制、重命名、作为电子邮件附件发送、作为 POST HTTP 请求的参数传递，或者以其他数据对象的方式处理，如图像、文档或媒体文件。

研究表明，许多应用程序已经将 SQLite 作为容器对象使用。例如，[肯尼迪](https://odin.cse.buffalo.edu/papers/2015/TPCTC-sqlite-final.pdf)（与 SQLite 开发者无关）报告称，14% 的安卓应用程序从未向其 SQLite 数据库写入。据信，这些应用程序是从云端下载整个数据库，然后根据需要在本地使用信息。换句话说，这些应用程序使用 SQLite 的方式不是作为数据库，而是作为可查询的传输格式。

## 1.2\. 使用 SQLite 存档的应用程序

[Fossil 分布式版本控制](https://fossil-scm.org/)系统为用户提供了将检入作为 Tarballs、ZIP 存档或 SQLite 存档下载的选项。

# 2\. SQLite 存档的优势

1.  SQLite Archive 是灵活的。ZIP Archive 和 Tarballs 仅限于存储文件。SQLite Archive 存储文件以及似乎对应用程序有用的其他表格和/或关系数据。

1.  SQLite Archive 是事务性的。即使在更新过程中发生崩溃或断电，更新也是原子且持久的。读者看到的内容版本是一致且不变的，即使其他过程正在同时更新存档。

1.  SQLite Archive 可以增量更新。可以添加、删除或替换单个文件，而无需重写整个存档。

1.  使用高级查询语言（SQL）可以查询 SQLite Archive。一些示例：

    +   存档中所有以“.h”或“.cpp”结尾的文件的总大小是多少？

    +   文件中有多少百分比的文件压缩率低于 25%？

    +   存档中有多少可执行文件？这些问题（以及无数其他问题）可以在不解压缩或提取任何内容的情况下得到回答。

1.  已经为其他目的使用 SQLite 的应用程序可以轻松添加对 SQLite Archives 的支持，使用小扩展（[`sqlite.org/src/file/ext/misc/sqlar.c`](https://sqlite.org/src/file/ext/misc/sqlar.c)）来处理内容的压缩和解压缩。即使在存档中的文件是未压缩的，甚至可以省略这个微小的扩展。相比之下，支持 ZIP Archives 和/或 Tarballs 需要单独的库或大量额外的自定义代码，有时两者兼而有之。

1.  SQLite Archive 可以绕过防火墙强加的审查。例如，某些被视为“危险”的文件类型（例如：DLLs）会被 Gmail（例如：[Gmail](https://support.google.com/mail/answer/6590)）以及可能许多其他电子邮件服务和防火墙阻止，即使文件被包装在 ZIP Archive 或 Tarball 中。但是这些防火墙通常（尚未）不了解 SQLite Archives，因此可以将内容放入 SQLite Archive 以规避审查。

# SQLite Archive 的缺点

1.  SQLite Archive 是相对较新的格式。它首次在 2014 年描述。 ZIP Archives 和 Tarballs 相反，已存在几十年并且已成为标准格式。大多数程序员知道 ZIP Archive 或 Tarball 是什么，但是如果说“SQLite Archive”，您更有可能得到“什么？”的回复。用于处理 ZIP Archives 和 Tarballs 的工具更可能安装在标准计算机上。

1.  由于 SQLite 数据库是一种更通用的格式（它设计用于执行更多操作而不仅仅是存储一堆文件），因此它不像 ZIP Archive 或 Tarball 格式那样紧凑。SQLite Archive 通常比等效的 ZIP Archive 大约大 1%。Tarballs 以单个单位压缩而不是像 SQLite 和 ZIP Archives 那样单独压缩每个文件。因此，Tarballs 倾向于比 ZIP 或 SQLite Archives 小。

    例如，以下表格显示了 SQLite 3.22.0 源树中 1,743 个文件的 SQLite 存档、ZIP 存档和 Tarball 的相对大小：

    | SQLite 存档 | 10,754,048 |
    | --- | --- |
    | ZIP 存档（使用 Info-ZIP 3.0） | 10,662,365 |
    | ZIP 存档（使用 zipfile） | 10,390,215 |
    | Tarball |  9,781,109 |

1.  SQLite 存档仅支持 [Deflate](https://zlib.net/) 压缩方法。Tarballs 和 ZIP 存档支持更广泛的压缩方法。

# 4\. 从命令行管理 SQLite 存档

推荐的创建、更新、列出和提取 SQLite 存档的方式是使用 sqlite3.exe 命令行 shell 适用于 SQLite 版本 3.23.0（2018-04-02）或更高版本。此 CLI 支持 -A 命令行选项，可轻松管理 SQLite 存档。SQLite 版本 3.22.0（2018-01-22）的 CLI 具有用于管理 SQLite 存档的 .archive 命令，但需要与 shell 交互。

要使用以下命令之一列出名为 "example.sqlar" 的 SQLite 存档中的所有文件：

```
sqlite3 example.sqlar -At
sqlite3 example.sqlar -Atv

```

要从名为 "example.sqlar" 的 SQLite 存档中提取所有文件：

```
sqlite3 example.sqlar -Ax

```

要在当前目录中包含所有 *.txt 文件的新 SQLite 存档命名为 "alltxt.sqlar"：

```
sqlite3 alltxt.sqlar -Ac *.txt

```

要向现有的 SQLite 存档中添加或更新文件：

```
sqlite3 example.sqlar -Au *.md

```

要获取使用提示和所有选项的摘要，只需向 CLI 提供 -A 选项而无需其他参数：

如果文件名参数是 ZIP 存档而不是 SQLite 数据库，则所有这些命令都以相同的方式工作。

就像有“zip”程序管理 ZIP 存档，有“tar”程序管理 Tarballs 一样，存在 ["sqlar" 程序](https://sqlite.org/sqlar) 来管理 SQL 存档。"sqlar" 程序能够创建一个新的 SQLite 存档，列出现有存档的内容，添加或删除存档中的文件，和/或提取存档中的文件。另外，一个单独的 "sqlarfs" 程序能够将 SQLite 存档挂载为 [Fuse 文件系统](https://github.com/libfuse/libfuse)。

# 5\. 从应用程序代码管理 SQLite 存档

应用程序可以通过链接到 SQLite 并包含 [ext/misc/sqlar.c](https://sqlite.org/src/file/ext/misc/sqlar.c) 扩展来轻松读取或写入 SQLite 存档，以处理压缩和解压缩。sqlar.c 扩展创建了两个新的 SQL 函数。

**sqlar_compress(X)**

sqlar_compress(X) 函数尝试使用 [默认](https://zlib.net/) 算法压缩 blob X 的副本，并将结果作为 blob 返回。如果输入 X 不是可压缩的 blob，则返回 X 的副本。在将内容插入 SQLite 存档时使用此例程。

**sqlar_uncompress(Y,SZ)**

sqlar_uncompress(Y,SZ) 函数将撤销由 sqlar_compress(X) 完成的压缩。Y 参数是压缩内容（从先前调用 sqlar_compress() 的输出），而 SZ 是生成 Y 的输入 X 的原始未压缩大小。如果 SZ 小于或等于 Y 的大小，则表示未发生压缩，因此 sqlar_uncompress(Y,SZ) 返回 Y 的副本。否则，sqlar_uncompress(Y,SZ) 对 Y 运行 Inflate 算法以解压缩它并将其恢复为其原始形式，并返回未压缩内容。在从 SQLite 存档中提取内容时使用此例程。

使用上述两个例程，应用程序可以简单地向 SQLite 存档中插入新记录或提取现有记录。使用以下代码向 SQLite 存档中插入新记录：

```
INSERT INTO sqlar(name,mode,mtime,sz,data)
 VALUES ($name,$mode,strftime('%s',$mtime),
         length($content),sqlar_compress($content));

```

使用以下代码从 SQLite 存档中提取条目：

```
SELECT name, mode, datetime(mtime,'unixepoch'), sqlar_uncompress(data,sz)
  FROM sqlar
 WHERE ...;

```

上述代码适用于一般情况。对于只存储未压缩或不可压缩内容的 SQLite 存档的特殊情况（例如，存储仅 JPEG、GIF 和/或 PNG 图像的 SQLite 存档），则可以将内容插入和提取出数据库，而不使用 sqlar_compress() 和 sqlar_uncompress() 函数，并且不需要 sqlar.c 扩展。

*本页上次修改于 [2023-01-12 11:08:35](https://sqlite.org/docsrc/honeypot) UTC*
