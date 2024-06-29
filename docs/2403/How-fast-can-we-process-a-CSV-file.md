<!--yml

类别：未分类

日期：2024-05-27 14:45:43

-->

# 我们能多快地处理一个 CSV 文件

> 来源：[https://datapythonista.me/blog/how-fast-can-we-process-a-csv-file](https://datapythonista.me/blog/how-fast-can-we-process-a-csv-file)

关注我获取更多内容或联系获取工作机会：

[推特](https://twitter.com/datapythonista) / [领英](https://www.linkedin.com/in/datapythonista/)

## 介绍

逗号分隔值（CSV）因其简单性和人类易读性而成为存储表格数据的极其流行的格式，相比更高效的二进制格式如 Parquet，它可以直接被人类阅读：

```
`name,age
Maryam,23
Mèng yáo,56
Ebunoluwa,31` 
```

我认为任何数据专业人士或爱好者在某个时刻都必须处理过 CSV 文件，无论是用于练习数据集（如泰坦尼克号或鸢尾花数据集），还是用于客户从电子表格中导出的数据或其他用途。

如果您关心性能，可能要避免使用 CSV 文件。但由于我们的数据源通常如同我们的家人，我们不能选择，我们将在这篇博文中看到如何尽可能快地处理一个 CSV 文件。

由于我在 pandas 核心开发团队已经超过 5 年，并且作为 pandas 用户的时间更长，我对于认为处理 CSV 文件的事实标准类似于这样有非常大的偏见：

```
`import pandas

df = pandas.read_csv("people.csv")
df["age"].mean()` 
```

虽然我认为今天这仍然是一个合理的选择，但在这篇博文中，我将展示许多其他选择，特别关注速度。

在我继续之前，我需要做两个**重要的声明**：

+   执行速度并不总是您关心的问题。拥有清晰且易维护的代码通常更为重要。市场时间可能也更为重要。记住唐纳德·克努斯（Donald Knuth）所说的：“过早优化是万恶之源”。

+   基准测试不具有很好的推广性。对于我正在解决的问题来说，最快的方法可能不是您问题的最快方法。一般而言，说“工具 X 是最快的”是没有意义的，除非您了解确切测量的任务内容、数据量、其分布以及所使用的硬件……

## 问题

对于这篇博文，我定义了一个简单的 CSV 文件和一个单一的任务。CSV 文件有一百万行，第一列是一个我们将忽略的 ID，还有 8 列整数随机数，范围是 -2e16 到 2e16。

这些数字代表一个 8-D 空间中点的坐标，任务是找出有多少个点距离原点不超过 1e16 单位。在实践中，这就像计算：

```
`math.sqrt(col1 ** 2 +
          col2 ** 2 +
          col3 ** 2 +
          col4 ** 2 +
          col5 ** 2 +
          col6 ** 2 +
          col7 ** 2 +
          col8 ** 2)` 
```

这个任务的理念是进行一些 CPU 计算，但不要太多，因此阅读和解析文件仍然非常重要。

正如前面提到的，这个任务可能与你执行的任务非常不同。例如，添加一个我们不需要的非常长的文本列会增加计算机从磁盘读取数据的时间，与 CPU 使用时间相比，结果将有所不同。虽然这个任务可能有些随意，但我认为它可以教会我们很多关于性能的知识。

## 回到基础知识

我之前说过，在我的观点中，使用 pandas 是一种事实上的标准。这是当用 pandas 处理此特定任务时的示例：

```
`values = (pandas.read_csv(fname,
                          header=None,
                          index_col=0,
                          dtype=float)
                .pow(2)
                .sum(axis='columns')
                .pow(.5)
)
print(values[values < 1e16].count())` 
```

首先一个问题是，为什么选择 pandas？既然我们已经在使用 Python，为什么不在 Python 中简单实现这个？在纯 Python 中的解决方案可能如下所示：

```
`with open(fname) as f:
    counter = 0

    reader = csv.reader(f)
    for row in reader:
        result = math.sqrt(sum([int(value) ** 2 for value in row[1:]]))
        if result < 10_000_000_000_000_000:
            counter += 1

print(counter)` 
```

这个实现的代码行数大约与 pandas 版本相同，并且会避免我们学习不太直观的 pandas API。因此，这似乎是一个非常合理的选择，对吧？我们通常不这样做的主要原因实际上是速度，因为 Python 在数据处理方面非常慢。这主要是因为 Python 的优先级一直是其简单性和灵活性，而不是数据处理速度。

虽然存在使用 PyPy 解释器等其他方法，但是现代 Python 中的数据处理大多数时候都会使用 Python 作为调用更快函数的包装器。因此，当使用 Python 处理大规模数据时，我们大多数情况下会避免使用循环，并且会寻找 pandas（或类似工具）函数，其中有人用 C 或其他低级语言实现了循环。例如，`df.pow(2)` 将遍历数据帧中的每个项并将其平方，但我们不会在 Python 中循环，而是告诉 pandas 执行这个操作，pandas 将调用一个 C 函数，它比 Python 更快地执行循环和操作。在这种情况下，循环是由 pandas 使用的 NumPy 实现的。

## 在 pandas 中解析 CSV

现在，希望已经明确了我们不想自己实现处理过程，而是使用一个提供所需算法的工具以及一种快速语言，让我们来看看 Python 世界中的一些主要选项。

我将讨论的第一个也是 pandas 的一部分，即 PyArrow 引擎。代码与之前完全相同，只需进行一个更改：

```
`values = (pandas.read_csv(fname,
                          engine="pyarrow",  # <- we add this parameter to `read_csv`
                          header=None,
                          index_col=0,
                          dtype=float)` 
```

对于最终用户来说，它只需在 `read_csv` 调用中添加一个额外的参数。但在内部，用于读取 CSV 的算法从最初作为 pandas 的一部分在 Cython 中实现的原始 CSV 解析完全改变为 Arrow 库中用 C++ 实现的较新的多线程 CSV 解析，并通过 PyArrow 包提供给 Python 生态系统使用。

也许有趣的是，pandas 还包括第三个引擎，即`python`引擎，它完全用纯 Python 实现。是的，就是我说我们不想为了速度而做的事情。但是对于一些用户来说可能有我不太了解的原因，他们避免使用 Python 的 C 扩展，所以这就是原因。

使用不同的 Python 和 pandas 选项执行我们任务的时间如下：

| 描述 | 文件 / 函数 | 时间（秒） |
| --- | --- | --- |
| 使用 int 类型，纯 Python 循环和 csv 模块 | pure_python_int | 3.4547557830810547 |
| 使用 float 类型，纯 Python 循环和 csv 模块 | pure_python_float | 3.8738009929656982 |
| pandas 使用 C 引擎 | pandas_c | 1.50089430809021 |
| pandas 使用 Python 引擎 | pandas_python | 8.328583478927612 |
| pandas 使用 PyArrow 引擎和 NumPy 数据类型 | pandas_pyarrow | 0.31276631355285645 |
| pandas 使用 PyArrow 引擎和 PyArrow 数据类型 | pandas_pyarrow_arrow | 0.29172492027282715 |

我们可以看到表格中的差异是巨大的。仅仅通过将我们的 `read_csv(engine=...)` 参数从 `engine="python"` 改为 `engine="pyarrow"`，就使得 pandas 在执行完全相同任务时快了约30倍。

同样地，从默认的 C 引擎更改为使用 PyArrow 引擎使得任务变快了约5倍。如果你想知道为什么 pandas 没有将更快的引擎设为默认值，我们实际上可能会在 pandas 3.0 中做出改变，但这个话题仍在讨论中。由于这个改变涉及要求始终将 PyArrow 与 pandas 一起安装，并且 PyArrow 会使 pandas 的安装大小增加约100 Mb，这可能并不适合所有人（考虑到 WebAssembly 中的 pandas、Lambda、树莓派等）。至少目前来说，如果你关心更快地读取 CSV 文件，你可能希望手动安装 PyArrow，并在调用 `read_csv` 时使用 `engine="pyarrow"`。

## 其他 Python 选项

多年来，pandas 几乎是处理 Python 世界中 CSV 文件的唯一选择。但幸运的是，这种情况正在迅速改变，现在存在更多的选择。这里有一个包含其他工具的比较：

| 描述 | 文件 / 函数 | 时间（秒） |
| --- | --- | --- |
| pandas 使用 C 引擎 | pandas_c | 1.50089430809021 |
| pandas 使用 PyArrow 引擎和 PyArrow 数据类型 | pandas_pyarrow_arrow | 0.29172492027282715 |
| NumPy 的 loadtxt 函数 | numpy_loadtxt | 1.8354885578155518 |
| DuckDB 0.9.2 使用 SQL API | duckdb_sql | 0.8167853355407715 |
| DuckDB 0.10.0 使用 SQL API | duckdb_sql | 0.288881778717041 |
| DataFusion 使用 SQL API | datafusion_sql | 0.20633697509765625 |
| Polars 使用急切模式 | polars_eager | 0.11399054527282715 |
| Polars 懒惰模式 | polars_lazy | 0.10555672645568848 |
| Polars 流式模式 | polars_streaming | 0.11504125595092773 |
| Polars 使用 SQL API | polars_sql | 0.09796714782714844 |

我们可以看到 NumPy 的表现类似于默认的 pandas 引擎，Datafusion 表现相当不错，但最大赢家是 Polars，其性能比最快的 pandas 引擎快 3 倍。DuckDB 的结果令人惊讶，因为 DuckDB 总体上的性能与 Polars 相似。在撰写本文之前，DuckDB 的 CSV 解析器经历了一次重大重构。新版本似乎快得多，并且其性能类似于带有 PyArrow 的 pandas。在其他基准测试中（我猜这些测试对于 CSV 文件的读取和解析速度影响较小），DuckDB 通常是最快的实现之一，速度与 Polars 相似。至少在 [TPCH 基准套件](https://pola.rs/posts/benchmarks/) 中是如此。正如我之前所说，对于这种特定用例更快（或更慢）并不意味着始终更快或更慢，尽管它们之间肯定存在相关性。

对于不熟悉这些库的人来说，DuckDB 是一个分析型数据库，不需要运行服务器（通常被称为分析界的 SQLite）。Datafusion 是一个用于构建分析型数据库或数据框架库的引擎。而 Polars 则主要是 pandas 的替代品，但总体上更快，其 API 更类似于 Spark 或 R，并且在 Python 以外的世界也可用。

可以在 [此存储库](https://github.com/datapythonista/bench_csv) 中找到运行这些基准测试的代码。

## 非 Python 选项

在 Python 以外的世界肯定有很多选项（也包括 Python 内部），我无法尝试每一个。例如，我认为尝试某些选项并不合理，比如分布式系统如 Spark 或 Dask，这些系统设计用于处理更大的数据。但有一个值得尝试，那就是 R，因为它是这项任务的非常常见的选择。

尽管我对 R 的了解不深，似乎可以通过这个简单的命令来实现：

```
`Rscript -e 'sum(sqrt(rowSums(read.csv("data.csv", header=FALSE)[-1] ^ 2)) < 1e16)'` 
```

结果令人惊讶，因为 R 看起来非常慢：

| 描述 | 时间（秒） |
| --- | --- |
| 使用整型类型的纯 Python 循环和 csv 模块 | 3.486 |
| 使用 C 引擎的 pandas | 2.061 |
| 使用 Python 引擎的 pandas | 8.823 |
| 带有 PyArrow 引擎和 PyArrow 数据类型的 pandas | 0.902 |
| 使用 SQL API（Python）的 Polars | 0.300 |
| R | 31.547 |
| 使用 data.table / fread 的 R | 0.722 |

请注意，结果与前表不同，因为之前的结果未计算启动 Python 解释器的时间，但为了与 R 进行公平比较，现在需要计算这部分时间。

因此，基于这个用例和实现，Polars 比 R 快 100 倍，而 R 看起来比我们考虑的最慢的 Python 实现慢了三倍以上（即 pandas 的 `python` 引擎）。

## 我们能比 Polars 更快吗？

到目前为止，我们可以得出结论，对于这个用例，Polars 是我所知道的工具中最快的选项。但我们能做得更快吗？

答案是肯定的，因为Polars是一个通用工具，支持大多数CSV文件，但我可以实现一个针对特定文件的算法。例如，Polars支持Unicode文件、分号分隔的文件、带引号的文本……这些在这个任务中并不需要。

第二个问题是，在低级语言中是否容易实现比Polars更快的算法？答案显然是否定的。

我的Rust CSV解析器的首个实现如下：

```
`fn main()  {
  let  mut  counter: u16 =  0;
  let  mut  reader  =  csv::Reader::from_path("data.csv").unwrap();

  for  record  in  reader.records()  {
  let  record  =  record.unwrap();
  let  result  =  (
  record.get(1).unwrap().parse::<f32>().unwrap().powf(2.)
  +  record.get(2).unwrap().parse::<f32>().unwrap().powf(2.)
  +  record.get(3).unwrap().parse::<f32>().unwrap().powf(2.)
  +  record.get(4).unwrap().parse::<f32>().unwrap().powf(2.)
  +  record.get(5).unwrap().parse::<f32>().unwrap().powf(2.)
  +  record.get(6).unwrap().parse::<f32>().unwrap().powf(2.)
  +  record.get(7).unwrap().parse::<f32>().unwrap().powf(2.)
  +  record.get(8).unwrap().parse::<f32>().unwrap().powf(2.)
  ).sqrt();

  if  result  <  1e16  {
  counter  +=  1;
  }
  }
  println!("{}",  counter);
}` 
```

如你所见，这个实现相当简单，而且尽管Rust在许多情况下是一种极快的低级语言，但代码像Python一样易读。这并不是使用Rust的唯一原因，可能最大的优势在于语言设计和智能编译器，在编译时能够捕获大多数bug，特别是那些可能导致C/C++程序段错误的bug。

为了使与Polars的比较具有意义，我们需要将Polars移出Python的世界。好消息是，Polars也是用Rust构建的，我们可以在不需要Python解释器的情况下运行Polars代码，这会大大减少额外开销。这是我们使用其SQL API在Polars中实现任务的实现（Polars在Rust世界中也有类似Python的DataFrame API，但我将在此处使用SQL API）：

```
`use  polars::prelude::*;
use  polars::sql::SQLContext;

fn main()  {
  let  mut  schema  =  Schema::new();
  schema.with_column("column_1".into(),  DataType::UInt32);
  schema.with_column("column_2".into(),  DataType::Float32);
  schema.with_column("column_3".into(),  DataType::Float32);
  schema.with_column("column_4".into(),  DataType::Float32);
  schema.with_column("column_5".into(),  DataType::Float32);
  schema.with_column("column_6".into(),  DataType::Float32);
  schema.with_column("column_7".into(),  DataType::Float32);
  schema.with_column("column_8".into(),  DataType::Float32);
  schema.with_column("column_9".into(),  DataType::Float32);

  let  df  =  LazyCsvReader::new("data.csv")
  .has_header(false)
  .with_schema(Some(schema.into()))
  .finish()
  .unwrap();

  let  mut  context  =  SQLContext::new();

  context.register("data",  df);

  let  result  =  context.execute("
 SELECT COUNT(*)
 FROM data
 WHERE SQRT(
 POWER(column_2, 2)
 + POWER(column_3, 2)
 + POWER(column_4, 2)
 + POWER(column_5, 2)
 + POWER(column_6, 2)
 + POWER(column_7, 2)
 + POWER(column_8, 2)
 + POWER(column_9, 2)
 ) < 10000000000000000").unwrap().collect();

  println!("{result:?}");
}` 
```

注意，定义模式并非真正必要，没有定义模式的代码会显著更短。但当与我的自定义实现进行比较时，让Polars推断数据类型会显得不公平。

比较两种实现的时间，我得到了以下结果：

| 描述 | 时间（秒） |
| --- | --- |
| 我的实现 | 0.563 |
| Polars 1 CPU核 | 0.483 |
| Polars所有CPU核 | 0.150 |

这里有趣的部分是，Polars不仅比我的非常简单的解决方案更快，而且在强制Polars仅使用一个CPU核时仍然更快（我的实现只使用一个CPU核，但Polars默认会使用所有可用的核）。

幸运的是，这还不是结束。有一些原因使得像Polars这样的通用实现更快，还有一些可以改进我的实现的方法。

经过一些分析，我发现 Rust 处理使用 `csv` crate / package 处理文件时，分支误判的次数比逐字符迭代文件要高得多。简单地解释一下分支误判是什么，假设 CPU 有一个队列，其中包含它需要执行的所有指令，因此下一个指令在一个完成后准备好。但是，代码中有条件语句（一个 `if`），只有当条件为 `true` 或 `false` 时才能成为队列中的下一个。因此，在填充队列时，操作系统会在评估条件之前进行猜测，看起来最有可能成为下一个的指令。当猜测错误时，从 CPU 外部将实际需要运行的代码添加到队列中需要一些时间。当我们在这种低级别编写代码并且关心毫秒级性能时，我们希望帮助编译器和操作系统尽可能准确地猜测，以便队列尽可能包含正确的代码，并且尽可能少的时间丢失。

另一个因素是文件从磁盘访问的方式。有不同的选择，比如在解析之前将整个文件加载到内存中，逐行加载... 这种操作的最快选项是使用内存映射。内存映射是操作系统的一个特性（系统调用），其中内核将虚拟内存地址映射到磁盘上的文件。使用内存映射时，Linux 内核在需要之前会相当聪明地预加载数据到内存，以及将数据预加载到内存缓存中。如果我们在设置内存映射时告诉内核这些内存将如何被访问，内核甚至可以更加智能化。在我们的情况下，我们将按顺序访问文件，因此如果我们在设置内存映射时告知 Linux 内核这一点，内核将显著加快我们的程序速度。

另一个更明显的因素是并行化。现代计算机通常有多个 CPU，我们可以将 CPU 需要执行的工作分配给它们中的多个。因此，如果我们有 1 秒钟的 CPU 工作量，但有 4 个 CPU，我们可以让我们的程序在 0.25 秒内完成。实际上，时间会比 0.25 秒长得多，但我们肯定可以显著加快事情的进展。此外，许多现代 CPU 通过 SIMD 操作在单个 CPU 中提供并行性。在这种操作中，可以同时应用于多个值。我在这里的测试中没有实现 SIMD，但这可能是另一个选择。

最后，我们可以进行最后一个优化。一般来说，在处理CSV文件时，我们希望首先将每个值提取为字符串，然后将其转换为适当的类型。在这种情况下，由于我主要解析整数值，我可以在读取字符时构建数字。因此，如果我首先找到`"2"`，我只需将其转换为整数。如果之后我找到`"4"`，我将上一个值`2`乘以`10`并加上刚找到的`4`，以获得24。如果我一直重复此操作，直到找到逗号，我已经解析了数字。这种方法似乎比从整个字符串引用解析数字要快得多。在这种特殊情况下更好的是，因为我将对所有数字进行平方，所以我不关心它们是正数还是负数，我完全可以忽略符号。

在实施这些优化后，我设法实现了以下执行时间：

| 描述 | 时间（秒） |
| --- | --- |
| 我的原始实现 | 0.563 |
| Polars 1 CPU核心 | 0.483 |
| Polars所有CPU核心 | 0.150 |
| 我的单核优化实现 | 0.241 |
| 我的多核优化实现 | 0.070 |

所以，无论是使用单个CPU还是全部CPU，我的最终实现都比Polars快2倍。显然，在大多数情况下，使用Polars或其他通用工具更有意义，因为实现整个解析的开发时间只是实现整个管道的一小部分。但是在性能是最重要因素的情况下，这些结果似乎表明，与更快的通用实现相比，可以显著更快地处理CSV文件。

希望您喜欢这篇文章，请随时在[Twitter](https://twitter.com/datapythonista)关注我，以获取未来文章的更新。如果您认为文章需要修正，或者您的公司需要相关工作的帮助，或者您想讨论工作机会，请随时在[LinkedIn](https://www.linkedin.com/in/datapythonista/)上联系我。

关注我获取更多内容或联系工作机会：

[Twitter](https://twitter.com/datapythonista) / [LinkedIn](https://www.linkedin.com/in/datapythonista/)
