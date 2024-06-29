<!--yml

类别：未分类

日期：2024-05-29 13:22:36

-->

# Python 中的整洁并行输出 | Max Bernstein

> 来源：[https://bernsteinbear.com/blog/python-parallel-output/](https://bernsteinbear.com/blog/python-parallel-output/)

假设你有一个程序对列表进行一些处理（基于一个正在进行中的项目）：

```
#!/usr/bin/env python3 
def log(repo_name, *args):
    print(f"{repo_name}:", *args)

def randsleep():
    import random
    import time
    time.sleep(random.randint(1, 5))

def func(repo_name):
    log(repo_name, "Starting")
    randsleep()  # Can be substituted for actual work
    log(repo_name, "Installing")
    randsleep()
    log(repo_name, "Building")
    randsleep()
    log(repo_name, "Instrumenting")
    randsleep()
    log(repo_name, "Running tests")
    randsleep()
    log(repo_name, f"Result in {repo_name}.json")

repos = ["repoA", "repoB", "repoC", "repoD"]
for repo in repos:
    func(repo) 
```

这很好。它可以工作。有点吵闹，但它可以工作。但是然后你发现了一些伟大的东西：你的问题是数据并行的。也就是说，你可以在系统允许的范围内并行处理尽可能多的存储库。万岁！你使用 `multiprocessing` 重写了代码：

```
import multiprocessing

# ... 
with multiprocessing.Pool() as pool:
    pool.map(func, repos, chunksize=1) 
```

不幸的是，输出有点笨拙。虽然每行仍然很好地归属于一个存储库，但它左右吐出行并且这些行交错在一起。难道你不想念像 Buck 和 Bazel 以及 Cargo 这样的工具的美丽并行输出吗？

幸运的是，StackOverflow 用户 [Leedehai](https://stackoverflow.com/questions/6840420/rewrite-multiple-lines-in-the-console/59147732#59147732) 是一个终端专家，知道如何在控制台一次性重写多行。我们可以根据我们的需求调整那个答案：

```
def fill_output():
    to_fill = num_lines - len(last_output_per_process)
    for _ in range(to_fill):
        print()

def clean_up():
    for _ in range(num_lines):
        print("\x1b[1A\x1b[2K", end="")  # move up cursor and delete whole line 
def log(repo_name, *args):
    with terminal_lock:
        last_output_per_process[repo_name] = " ".join(str(arg) for arg in args)
        clean_up()
        sorted_lines = last_output_per_process.items()
        for repo_name, last_line in sorted_lines:
            print(f"{repo_name}: {last_line}")
        fill_output()

def func(repo_name):
    # ...
    with terminal_lock:
        del last_output_per_process[repo_name]

# ... 
repos = ["repoA", "repoB", "repoC", "repoD"]
num_procs = multiprocessing.cpu_count()
num_lines = min(len(repos), num_procs)
with multiprocessing.Manager() as manager:
    last_output_per_process = manager.dict()
    terminal_lock = manager.Lock()
    fill_output()
    with multiprocessing.Pool() as pool:
        pool.map(func, repos, chunksize=1)
    clean_up() 
```

这将每个项目的状态逐行打印到终端。它将按照添加到 `last_output_per_process` 的项目的顺序打印，但你可以通过（例如）按字母数字顺序排序来改变这一点：`sorted(last_output_per_process.items())`。

注意，我们必须锁定数据结构和终端输出，以避免混乱；它们在进程之间共享（通过 `Manager` 进行序列化）。

我不确定如果日志输出有多行长或者如果有人在搞乱 `stdout`/`stderr`（可能是一个零星的 `print`）。如果你发现或者有了整洁的解决方案，请写信告诉我们。

这种技术可能相对于任何有线程和锁的编程语言都是可移植的。关键区别在于那些实现应该使用线程而不是进程；我使用进程是因为这是 Python。

查看这个扩展版本的 [Gist](https://gist.github.com/tekknolagi/4bee494a6e4483e4d849559ba53d067b)。

## 这是给你的一个演示

既然你读到这里，这里有一个程序的演示：

现在享受你新发现的有趣输出吧！
