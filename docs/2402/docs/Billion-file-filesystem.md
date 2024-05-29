<!--yml
category: 未分类
date: 2024-05-27 14:42:28
-->

# Billion file filesystem

> 来源：[https://blog.liw.fi/posts/2024/billion/](https://blog.liw.fi/posts/2024/billion/)

For a lark I made an `ext4` file system with a billion empty files. [https://gitlab.com/larswirzenius/create-empty-files](https://gitlab.com/larswirzenius/create-empty-files) has the program I wrote for this. I’ve done this [before](/posts/2020/10/01/a_billion_files/), but this time I made it a little simpler for me to do it again: everything in one Rust program rather than clunky scripts.

The disk image starts out as a terabyte-size sparse file, taking no space, and a file system is put in that. Thus, the image is all zeroes except for what actually gets written to it by the file system. With a billion empty files, the image uses 276 GiB disk space. On my desktop machine it took about 26 hours to create.

Compressing makes it smaller:

| `gzip -1` | 20 | 20546385531 |
| `gzip -9` | 15 | 15948883594 |
| `xz -T0 -1` | 11 | 11389236780 |
| `xz -T0 -M 60GiB -9` | 10 | 10465717256 |

(I did not measure compression times, sorry. If that interests you, you’ll have to do the work yourself.)

If you have a use for such a filesystem, please get in touch. However, if you can spend a day, you can easily create one yourself and save me a bit of bandwidth.

If you’d like a different filesystem, it should be easy enough to adjust the program I wrote to use another filesystem type.

Of what use is a filesystem with many empty files? You could use it to benchmark things that do things to all the files in a directory tree. For example, what is the fastest way to list all those files? Delete them? Back them up? Create an archive file with all of them? (These might be an interesting project for someone in university or college, maybe?)