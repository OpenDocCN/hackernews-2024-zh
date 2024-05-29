<!--yml
category: 未分类
date: 2024-05-27 14:58:50
-->

# wddbfs – Mount a sqlite database as a filesystem

> 来源：[https://adamobeng.com/wddbfs-mount-a-sqlite-database-as-a-filesystem/](https://adamobeng.com/wddbfs-mount-a-sqlite-database-as-a-filesystem/)

# wddbfs – Mount a sqlite database as a filesystem

17 Feb 2024 | Categories: hacks

Often when I’m prototyping a project, I hesitate to use a sqlite database despite their [many adavantages](https://sqlite.org/appfileformat.html). It seems much easier to just dump a bunch of files in a directory and to rely on the universal support for the filesystem API to read/delete/update records. Part of this is avoiding the overhead of figuring out a relational schema, but an equal amount of friction comes from the fact that .sqlite files are just slightly more difficult to inspect: the SQL syntax for selecting a few records is much more verbose than `head -n` or `tail -n`, there are special commands (which don’t work in some environments/versions) for listing tables, and neither my text editor nor my shell has autocompletion for database queries.

To try to get the best of both worlds, I have put together a little utility called *wddbfs*, which exposes a sqlite database as a (WebDAV^() filesystem, accessible to anything which can work with a filesystem, including terminals, file managers, and text editors.)

Here’s how it works. If you install it with:

`pip install git+https://github.com/adamobeng/wddbfs`

You can mount a database with:

```
wddbfs --anonymous --db-path=/path/to/an/example/database/like/Chinook_Sqlite.sqlite 
```

Which will be available at localhost:8080 with no username or password required.

Once you’ve [mounted](https://support.apple.com/guide/mac-help/connect-disconnect-a-webdav-server-mac-mchlp1546/mac) this WebDAV filesystem at, for example `/Volumes/127.0.0.1/`, you can see all the databases you specified with `--db-path`.

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

By default, all the tables can be read as CSV, TSV, json and line-delimited json (“.jsonl”)

These files can be manipulated with tools that work with a standard filesystem:

```
$ tail -n 3 Chinook_Sqlite.sqlite/Album.tsv
345     Monteverdi: L'Orfeo     273
346     Mozart: Chamber Music   274
347     Koyaanisqatsi (Soundtrack from the Motion Picture)      275
$ grep "Mahler" Chinook_Sqlite.sqlite/Artist.jsonl 
{"ArtistId": 240, "Name": "Gustav Mahler"} 
```

Although for now, the whole table gets read into memory for every read so this won’t work well for very large database files. There’s also no write support… yet.