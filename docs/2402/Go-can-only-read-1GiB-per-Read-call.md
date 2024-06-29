<!--yml

category: 未分类

date: 2024-05-27 14:45:26

-->

# Go 每次 `Read` 调用只能读取 1GiB

> 来源：[https://kgrz.io/go-file-read-max-size-buffer.html](https://kgrz.io/go-file-read-max-size-buffer.html)

<main class="post">

更新：我并不是说这是一个不好的选择，或者说这是一个 bug，或者是性能问题。这只是一个已经做出的选择，看起来有点不透明，如果不进行所有的历史挖掘的话，这个选择还是挺有趣的。

对于 Go 中的 `os.File` 对象（或结构体），单个 `Read` 调用的限制为 1GiB，尽管本地的 `read` 系统调用可以填充 2GiB 缓冲区（在我的 ARM macOS 和 Intel Linux 机器上测试过）。当我查看正在编写的示例单词计数程序的 pprof 概要时，我发现程序在 `syscall` 模块中花费了过多时间。在这种情况下，只能意味着 `read` 系统调用被调用了太多次。类似下面这样的情况将表现出这种行为：

```
f, err := os.Open("superlargefile.txt")
if err != nil {
    log.Fatal("error opening input file: ", err)
}
defer f.Close()

buf := make([]byte, 1024*1024*1024*2) // 2GiB buffer
fmt.Println("buffer size", len(buf))

for iter := 1; ; iter += 1 {
    n, err := f.Read(buf)

    if err != nil {
        if err == io.EOF {
            fmt.Println("done")
            break
        }

        log.Fatal("error reading input file: ", err)
    }

    fmt.Println("bytes read: ", n)
    fmt.Println("iter: ", iter)
} 
```

在一个 2.5G 文件上的输出可能如下：

```
buffer size 2147483648
bytes read:  1073741824
iter:  1
bytes read:  1073741824
iter:  2
bytes read:  490442752
iter:  3
done 
```

尽管初始化的缓冲区大小为 2GiB，但每次迭代只读取了 1GiB 到缓冲区中。深入研究源代码后发现，这是一个有意为之的选择。主要变更日志如下：

1.  [https://codereview.appspot.com/89900044](https://codereview.appspot.com/89900044) 作为对[golang/go#7812](https://github.com/golang/go/issues/7812)的修复。这个修复解决了在 macOS 和 FreeBSD 上读取大于或等于 2GiB 文件大小时读取失败的问题，通过限制每个`read`系统调用仅读取 2GiB-1 字节。对于其他操作系统，在这一点上并没有限制。

1.  [https://codereview.appspot.com/94070044](https://codereview.appspot.com/94070044) 作为对 1 的后续，其中限制被减少到 1GiB，没有进行任何操作系统检查，解释为至少可以允许从磁盘中读取对齐的数据，而不是可能会错过页缓存的奇数字节（我的理解）。

注意，自那次更改以来已经发生了很多变化，而该 `_unix.go` 文件在更改集中的当前文件引用为[src/internal/poll/fd_unix.go](https://github.com/golang/go/blob/release-branch.go1.22/src/internal/poll/fd_unix.go#L132-L137)。

### 旁注：系统限制

根据 Linux 的[`read`系统调用文档](https://www.man7.org/linux/man-pages/man2/read.2.html#NOTES)，可以传输的最大字节数为 2GiB。我用 Rust 和 C 中的基本脚本进行了测试。Rust 程序直接从 [`read_to_end()`的示例](https://doc.rust-lang.org/std/io/trait.Read.html#method.read_to_end) 中取得。在 `strace` 下运行的输出如下（这里进行了截断）：

```
read(3, ..., 6594816000) = 2147479552
read(3, ..., 4447336448) = 2147479552
read(3, ..., 2299856896) = 2147479552
read(3, ..., 152377344) = 152377344
read(3, "", 32)         = 0 
```

另外，一个类似的简单 C 程序在使用 `read` 系统调用在循环中读取文件时也会产生类似的输出：

```
SSIZE_MAX: 9223372036854775807 # outputting the limits.h constant
bytes read: 2147479552
bytes read: 2147479552
bytes read: 2147479552
bytes read: 152377344 
```

尽管这不是重点，但有趣的是 Go 的选择是选择了 2GiB-1，然后是 1GiB，以此来解释之前奇怪的缓冲区大小。

`</main>`
