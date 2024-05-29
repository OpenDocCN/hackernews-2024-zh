<!--yml
category: 未分类
date: 2024-05-27 14:45:26
-->

# Go can only read 1GiB per Read call

> 来源：[https://kgrz.io/go-file-read-max-size-buffer.html](https://kgrz.io/go-file-read-max-size-buffer.html)

<main class="post">

UPDATE: I don’t mean to say that this is a bad choice, or that it’s a bug, or even a performance implication. It’s just a choice that was made which seemed a bit opaque without doing all the history spelunking I did here, and it’s interesting to see the reasoning behind it.

There’s a 1GiB limit for a single `Read` call for an `os.File` entity (object? struct?) in Go, even though native `read` syscall can fill a 2GiB buffer (as tested in my arm macos and Intel Linux machine). I ran into this when looking at a pprof profile of a sample word count program I was writing, which showed the program was spending way too much time in the `syscall` module. That in this context can only mean one thing: way too many `read` syscalls were getting called. Something like this would show this behaviour:

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

That, on a 2.5G file would output something like:

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

Even though the initialised buffer size is 2GiB, only 1GiB is read into the buffer per iteration. Upon digging into the source code, it looks like this is a deliberate choice. The main change logs from the history point to the following:

1.  [https://codereview.appspot.com/89900044](https://codereview.appspot.com/89900044) as a fix for [golang/go#7812](https://github.com/golang/go/issues/7812). This had a fix for failing reads on file sizes greater than or equal to 2GiB on macos and freebsd by capping each `read` syscall to only read a 2GiB-1 bytes. For the rest of operating systems, at this point, there was no cap.
2.  [https://codereview.appspot.com/94070044](https://codereview.appspot.com/94070044) as a followup of 1, where the limit was decreased without any OS checks to 1GiB, with an explanation that at least it would allow for aligned reads from disk, as opposed to an odd number that might miss page caches (my understanding).

Note that a lot has changed since that changeset, and the current file reference for that `_unix.go` file in the changeset is [src/internal/poll/fd_unix.go](https://github.com/golang/go/blob/release-branch.go1.22/src/internal/poll/fd_unix.go#L132-L137).

### Aside: System limits

As per the linux [`read` syscall documentation](https://www.man7.org/linux/man-pages/man2/read.2.html#NOTES), the maximum bytes that can be transferred is 2GiB. And I tested this out with rudimentary scripts in Rust and C. The Rust program is taken verbatim from the example for [`read_to_end()`](https://doc.rust-lang.org/std/io/trait.Read.html#method.read_to_end). Running that under `strace` has the following output (truncated here):

```
read(3, ..., 6594816000) = 2147479552
read(3, ..., 4447336448) = 2147479552
read(3, ..., 2299856896) = 2147479552
read(3, ..., 152377344) = 152377344
read(3, "", 32)         = 0 
```

And a similar, simple C program results in similar output, when using the `read` syscall in a loop until the file is read:

```
SSIZE_MAX: 9223372036854775807 # outputting the limits.h constant
bytes read: 2147479552
bytes read: 2147479552
bytes read: 2147479552
bytes read: 152377344 
```

Although that’s neither here nor there, it’s still interesting that Go’s choice has been to pick 2GiB-1 and then 1GiB justifying the odd buffer size in the former.

</main>