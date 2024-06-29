<!--yml

category: 未分类

日期：2024年5月27日 14:58:50

-->

# **wddbfs** – 将 SQLite 数据库挂载为文件系统

> 来源：[https://adamobeng.com/wddbfs-mount-a-sqlite-database-as-a-filesystem/](https://adamobeng.com/wddbfs-mount-a-sqlite-database-as-a-filesystem/)

# **wddbfs** – 将 SQLite 数据库挂载为文件系统

2024年2月17日 | 分类：黑客

在我进行项目原型设计时，经常会犹豫是否使用 SQLite 数据库，尽管它们有[许多优点](https://sqlite.org/appfileformat.html)。直接在目录中倾倒一堆文件并依赖于文件系统 API 的普遍支持来读取/删除/更新记录似乎更容易。部分原因是避免了理解关系模式的开销，但同样大部分摩擦源于 .sqlite 文件略微难以检查：选择几条记录的 SQL 语法要比 `head -n` 或 `tail -n` 冗长得多，列出表的特殊命令（在某些环境/版本中不起作用），以及我的文本编辑器和 shell 都没有数据库查询的自动补全功能。

为了尝试获得两全其美，我编写了一个名为 *wddbfs* 的小工具，它将 SQLite 数据库暴露为一个 (WebDAV^() 文件系统，可供任何可以处理文件系统的工具使用，包括终端、文件管理器和文本编辑器。

安装方法如下：

`pip install git+https://github.com/adamobeng/wddbfs`

您可以使用以下命令挂载数据库：

```
wddbfs --anonymous --db-path=/path/to/an/example/database/like/Chinook_Sqlite.sqlite 
```

将在 localhost:8080 上可用，无需用户名或密码。

一旦您在例如 `/Volumes/127.0.0.1/` 上[挂载](https://support.apple.com/guide/mac-help/connect-disconnect-a-webdav-server-mac-mchlp1546/mac)了这个 WebDAV 文件系统，您可以使用 `--db-path` 查看指定的所有数据库。

```
$ ls /Volumes/127.0.0.1/
Chinook_Sqlite.sqlite
$ ls /Volumes/127.0.0.1/Chinook_Sqlite.sqlite
Album.csv           Customer.tsv        Invoice.jsonl       Playlist.json
Album.json          Employee.csv        Invoice.tsv         Playlist.jsonl
Album.jsonl         Employee.json       InvoiceLine.csv     Playlist.tsv
Album.tsv           Employee.jsonl      InvoiceLine.json    PlaylistTrack.csv
Artist.csv          Employee.tsv        InvoiceLine.jsonl   PlaylistTrack.json
Artist.json         Genre.csv           InvoiceLine.tsv     PlaylistTrack.jsonl
Artist.jsonl        Genre.json          MediaType.csv       PlaylistTrack.tsv
Artist.tsv          Genre.jsonl         MediaType.json      Track.csv
Customer.csv        Genre.tsv           MediaType.jsonl     Track.json
Customer.json       Invoice.csv         MediaType.tsv       Track.jsonl
Customer.jsonl      Invoice.json        Playlist.csv        Track.tsv 
```

默认情况下，所有表都可以作为 CSV、TSV、json 和行分隔的 json (“.jsonl”) 读取。

这些文件可以通过使用标准文件系统的工具进行操作：

```
$ tail -n 3 Chinook_Sqlite.sqlite/Album.tsv
345     Monteverdi: L'Orfeo     273
346     Mozart: Chamber Music   274
347     Koyaanisqatsi (Soundtrack from the Motion Picture)      275
$ grep "Mahler" Chinook_Sqlite.sqlite/Artist.jsonl 
{"ArtistId": 240, "Name": "Gustav Mahler"} 
```

尽管目前，每次读取时整个表都会加载到内存中，因此对于非常大的数据库文件来说，这种方法效果不佳。目前也没有写入支持……但愿未来会有。
