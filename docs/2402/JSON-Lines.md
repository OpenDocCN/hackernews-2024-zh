<!--yml
category: 未分类
date: 2024-05-29 13:22:41
-->

# JSON Lines

> 来源：[https://jsonlines.org/](https://jsonlines.org/)

This page describes the JSON Lines text format, also called newline-delimited JSON. JSON Lines is a convenient format for storing structured data that may be processed one record at a time. It works well with unix-style text processing tools and shell pipelines. It's a great format for log files. It's also a flexible format for passing messages between cooperating processes.

The JSON Lines format has three requirements:

### 1\. UTF-8 Encoding

JSON allows encoding Unicode strings with only ASCII escape sequences, however those escapes will be hard to read when viewed in a text editor. The author of the JSON Lines file may choose to escape characters to work with plain ASCII files.

Encodings other than UTF-8 are very unlikely to be valid when decoded as UTF-8 so the chance of [accidentally misinterpreting characters](https://en.wikipedia.org/wiki/Mojibake) in JSON Lines files is low.

### 2\. Each Line is a Valid JSON Value

The most common values will be objects or arrays, but any JSON value is permitted.

See [json.org](https://json.org/) for more information about JSON values.

### 3\. Line Separator is `'\n'`

This means `'\r\n'` is also supported because surrounding white space is implicitly ignored when parsing JSON values.

The last character in the file *may* be a line separator, and it will be treated the same as if there was no line separator present.

### 4\. Suggested Conventions

JSON Lines files may be saved with the file extension `.jsonl`.

Stream compressors like `gzip` or `bzip2` are recommended for saving space, resulting in `.jsonl.gz` or `.jsonl.bz2` files.

MIME type may be `application/jsonl`, but this is not yet standardized; any help writing the RFC would be greatly appreciated (see [issue](https://github.com/wardi/jsonlines/issues/19)).

Text editing programs call the first line of a text file "line 1". The first value in a JSON Lines file should also be called "value 1".