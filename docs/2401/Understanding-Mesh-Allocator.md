<!--yml
category: 未分类
date: 2024-05-27 15:15:28
-->

# Understanding Mesh Allocator

> 来源：[https://veera.app/mesh_allocator.html](https://veera.app/mesh_allocator.html)

Applications written in manually memory managed languages might fail to allocate memory, even when the required amount of space is available in memory. This problem is known as memory [fragmentation](https://en.wikipedia.org/wiki/Fragmentation_(computing)) and can be solved by compaction, where all allocated objects are moved together to make a contiguous block of free space available.

Garbage collected languages can solve this by stopping program execution to compact allocated objects and update all pointers to point to the relocated objects. But, languages which expose raw pointers to the programmer don’t have this luxury as it is impossible to trace and update all the pointers.

Therefore, to reduce memory fragmentation for applications written in manually memory managed languages, compaction should occur without changing the pointers’ values and this is exactly where Mesh shines.