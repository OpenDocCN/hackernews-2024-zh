<!--yml
category: 未分类
date: 2024-05-29 12:34:30
-->

# Pack

> 来源：[https://pack.ac/note/pack](https://pack.ac/note/pack)

## Introduction

Pack is a container format that can support files and raw data, and it is made to be safe, fast, and reliable for years to come.

It is designed to be the most efficient way to pack any amount of data and improve the user experience to fit the future. The author proposes a fresh solution based on well-formed standards to significantly improve performance while improving features like random access, update, security, and user experience.

It is free to use, and the source code is available with a permissive license, ensuring freedom from patent concerns.

You can get the latest version from

[https://pack.ac/](https://pack.ac/)

.

## Reason

It's been decades since any new proper work was done for a container format, and the author felt that there was a need for a new design considering new generations of hardware and advancements in algorithms.

Most popular solutions like Zip, gzip, tar, RAR, or 7-Zip are near or more than three decades old. While holding value for such a long time (in the computer world) is testimony to their greatness, the work is far from done.

To demonstrate that, here are some numbers comparing the output size and speed as CompressedSpeed.

```
Packing a copy of a folder containing more than 81K files and around 1.25 GB on a test machine:ZIP:     253 MB,  146 s      = 1
tar.gz:  214 MB,  28.5 s     ====== 6
tar:    1345 MB,  4.7 s      ====== 6
RAR:     235 MB,  27.5 s     ====== 6
7z:      135 MB,  54.2 s     ===== 5
Pack:    194 MB,  1.3 s      ================================================================================================================================================= 145
```

Unpack speed, random access, memory use, and other factors are also improved similarly.

Pack is smart and configures itself as needed; there are not many dials to play with.

Beside numbers, the Pack format is based on the universal and arguably one of the most safe and stable file formats, SQLite3, and compression is based on Zstandard, the leading standard algorithm in the field. These design choices promise reliable storage, transactional updates, exceptional speed, low resource usage, smartness, and simplicity.

## Future

In the near future, stabilizing will be the most important task. Locking and encryption, graphical interface, OS integration, and development tools and libraries are planned and under development. Builds for many more platforms will be available over time.

For the time being, you are encouraged to try Pack for yourself and share results and notes with others, including author by emailing, o at pack.ac.

If you want to know more about design, plans, and other notes, you can check out

[Notes](/notes)

.

You can find the source code for a deeper study at

[Source](/source)

.

## Conclusion

The field of data compression has been well-lit with great work in the past decades, and Pack tries to continue this path as the next step and proposes the next universal choice in the field.

The author is aware of unfathomable speedups and design decisions. Readers are welcome to test, read the code, and engage in discussions.

Everyone is encouraged to use Pack to have a more efficient solution and build on it for an improved future.

* * *

*A gift to anyone passionate about data, especially Phil Katz, D. Richard Hipp, Yann Collet, and me.*

## Footnotes