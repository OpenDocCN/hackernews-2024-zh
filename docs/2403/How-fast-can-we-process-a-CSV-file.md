<!--yml
category: 未分类
date: 2024-05-27 14:45:43
-->

# How fast can we process a CSV file

> 来源：[https://datapythonista.me/blog/how-fast-can-we-process-a-csv-file](https://datapythonista.me/blog/how-fast-can-we-process-a-csv-file)

Follow me for more content or contact for work opportunities:
[Twitter](https://twitter.com/datapythonista) / [LinkedIn](https://www.linkedin.com/in/datapythonista/)

## Introduction

Comma-separated values (CSV) are an extremely popular format to store tabular data because of their simplicity and how easy is to write them. The file can be directly read by a human, as opposed to more efficient binary formats like parquet, for example:

```
`name,age
Maryam,23
Mèng yáo,56
Ebunoluwa,31` 
```

I don't think I'm wrong to think that any data professional or enthusiast had to process a CSV file at some point, whether it's for practice datasets like the Titanic or Iris datasets, for data exported by a client from a spreadsheet or other use cases.

If you care about performance, you may want to avoid CSV files. But since our data sources are often like our family, we can't make a choice, we'll see in this blog post how to process a CSV file as fast as possible.

Since I've been in the pandas core development team for more than 5 years, and a pandas user for much longer than that, I'm very biased on thinking that the de facto standard for many people to process a CSV file is something like this:

```
`import pandas

df = pandas.read_csv("people.csv")
df["age"].mean()` 
```

While I think this is still today a reasonable choice, in this blog post I will show many other options, with a particular focus on speed.

Before I continue, I need to make two **important disclaimers**:

*   Execution speed is not always what you care about. Having clear and maintainable code is quite often more important. Time to market can also be a bigger priority. Remember what Donald Knuth said, "premature optimization is the root of all evil".
*   Benchmarks don't extrapolate well. The fastest approach for the problem I'm solving, is probably not fastest approach for your problems. Saying "tool X is the fastest" is in general meaningless unless you have the context of what exactly is the task being measured, what is the amount of data, its distribution, the hardware being used...

## The problem

For this blog post I defined a simple CSV file and a single task. The CSV has one million rows, a first column with an id that we will ignore, and 8 columns with integer random numbers in the range -2e16 to 2e16.

The numbers represent the coordinates of points in an 8-D space, and the task is to find how many of them are closer than 1e16 units to the origin. In practice, this is as simple as computing:

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

The idea with this task is to have some CPU computations, but not so many so reading and parsing the file is still very significant.

As mentioned earlier, this task can be very different from the tasks you perform. For example, adding one extra column with a very long text that we don't need for the task would increase the amount of time the computer will be reading from disk compared to the amount of time the CPU is being used, and results would differ. While this task may be somehow arbitrary, I think it can teach us a lot about performance.

## Back to the basics

I said before that using pandas is, at least in my point of view, kind of de facto standard. This is how this particular task could look like when addressed in pandas:

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

One first question is, why pandas? Since we are already using Python, why don't simply implement this in Python? A solution in pure Python could look like this:

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

This implementation has around the same number of lines of code than the pandas version, and would save us from learning the not very intuitive pandas API. So, it seems like a very reasonable option, right? The main reason we don't usually do this is actually speed, as Python is a very slow language for data processing. This is mostly because the priority for Python has always been its simplicity and flexibility, not speed at data processing.

While other approaches like using the PyPy interpreter exist, modern data processing in Python has mostly converged in using Python as a wrapper to call faster functions written in faster languages. So, when using Python for processing data of significant size, we mostly avoid loops, and we look for a pandas (or similar tool) function where someone implemented the loop for us in C or another lower level language. For example `df.pow(2)` will loop over every item in our dataframe and square it, but we don't loop in Python, we tell pandas to do it, and pandas will call a C function that performs the loop and the operation much faster than Python would. In this case the loop it's implemented in NumPy which pandas uses.

## CSV parsing in pandas

Now that it's hopefully clear that we don't want to implement our processing ourselves, but instead use a tool that provides the algorithms we need with a fast language, let's have a look at some of the main options in the Python world.

The first one I'm going to discuss is also part of pandas, the PyArrow engine. The code would be exactly like before, making just one change:

```
`values = (pandas.read_csv(fname,
                          engine="pyarrow",  # <- we add this parameter to `read_csv`
                          header=None,
                          index_col=0,
                          dtype=float)` 
```

For the final users, it simply involves adding an extra parameter to the `read_csv` call. But internally, the algorithm used to read the CSV changes completely from the original CSV parsing implemented as part of pandas in Cython (a Python-like language that transpiles to C), to a much newer multithread implementation of CSV parsing implemented in C++ in the Arrow library and made available to the Python ecosystem via the PyArrow package.

It's probably interesting to know that pandas also includes a third engine, the `python` engine which is implemented in pure Python. Yes, exactly what I said we don't want to do for speed. But there may be reasons I'm mostly unaware of for some users to avoid using Python C extensions, so there it is.

The time to execute our task with the different Python and pandas options is next:

| Description | File / Function | Time (seconds) |
| --- | --- | --- |
| Pure Python looping with csv module using int types | pure_python_int | 3.4547557830810547 |
| Pure Python looping with csv module using float types | pure_python_float | 3.8738009929656982 |
| pandas with C engine | pandas_c | 1.50089430809021 |
| pandas with Python engine | pandas_python | 8.328583478927612 |
| pandas with PyArrow engine and NumPy dtypes | pandas_pyarrow | 0.31276631355285645 |
| pandas with PyArrow engine and PyArrow dtypes | pandas_pyarrow_arrow | 0.29172492027282715 |

We can see in the table that the differences are huge. Just changing our `read_csv(engine=...)` parameter from `engine="python"` to `engine="pyarrow"` makes pandas around 30x times faster to perform the same exact task.

Also, changing from the default C engine to using the PyArrow engine makes the task 5x times faster. If you are wondering why pandas doesn't make the much faster engine the default, we may actually make it in pandas 3.0, but the topic it's still being discussed. Since the change involves requiring PyArrow to be always installed with pandas, and PyArrow increases the installation size of pandas by around 100 Mb, it may not be the best option for everyone (think of pandas in WebAssembly, Lambdas, in a Raspberry Pi...). At least for now, if you care about reading CSV files faster, you may want to install PyArrow manually, and use the `engine="pyarrow"` in your calls to `read_csv`.

## Other Python options

For many years pandas was almost the only option for processing a CSV file in the Python world. But luckily this is changing quickly, and more options exist now. Here there is a comparison including other tools:

| Description | File / Function | Time (seconds) |
| --- | --- | --- |
| pandas with C engine | pandas_c | 1.50089430809021 |
| pandas with PyArrow engine and PyArrow dtypes | pandas_pyarrow_arrow | 0.29172492027282715 |
| NumPy with loadtxt function | numpy_loadtxt | 1.8354885578155518 |
| DuckDB 0.9.2 with SQL API | duckdb_sql | 0.8167853355407715 |
| DuckDB 0.10.0 with SQL API | duckdb_sql | 0.288881778717041 |
| DataFusion with SQL API | datafusion_sql | 0.20633697509765625 |
| Polars in eager mode | polars_eager | 0.11399054527282715 |
| Polars in lazy mode | polars_lazy | 0.10555672645568848 |
| Polars in streaming mode | polars_streaming | 0.11504125595092773 |
| Polars with SQL API | polars_sql | 0.09796714782714844 |

We can see how NumPy performs similar to the default pandas engine, datafusion performs quite well, but the clear winner is Polars, performing 3x times faster than the fastest pandas engine. The results with DuckDB are surprising, as DuckDB in general has performance similar to Polars. A major refactoring of DuckDB CSV parser happened just before writing this blog post. The new version seems much faster, and its performance is similar to pandas with PyArrow. In other benchmarks (which I guess depend less on how fast a CSV file is read and parsed, DuckDB is usually one of the fastest implementations, with similar speed to Polars. At least in the [TPCH benchmark suite](https://pola.rs/posts/benchmarks/). As I said earlier, being faster (or slower) for this particular use case doesn't mean being consistently faster or slower, even if correlation surely exists.

For people not familiar with these libraries, DuckDB is an analytics database that doesn't require a running server (the SQLite of analytics it's usually called). Datafusion is an engine to build analytics databases or dataframelibraries on top of it. And Polars is mostly a pandas replacement, but in general faster, with an API more similar to Spark or R, and available also outside of the Python world.

The code to run these benchmarks can be found in [this repository](https://github.com/datapythonista/bench_csv).

## Non-Python options

There are surely many options outside of the Python world (and also inside the Python world), and I can't try each of them. For example I don't think it makes sense to try some of them, like distributed systems such as Spark or Dask, which are design to process much bigger data. But there is one worth trying, R, as it's a very common option for this task.

While my knowledge of R isn't great, seems like it can be implemented with this simple command:

```
`Rscript -e 'sum(sqrt(rowSums(read.csv("data.csv", header=FALSE)[-1] ^ 2)) < 1e16)'` 
```

The results are surprising, as R seems to be very slow:

| Description | Time (seconds) |
| --- | --- |
| Pure Python looping with csv module using int types | 3.486 |
| pandas with C engine | 2.061 |
| pandas with Python engine | 8.823 |
| pandas with PyArrow engine and PyArrow dtypes | 0.902 |
| Polars with SQL API (Python) | 0.300 |
| R | 31.547 |
| R with data.table / fread | 0.722 |

Note that the results are different than in the previous table because previous results didn't account for the time to launch the Python interpreter, but to make the comparison with R fair, it needs to be accounted now.

So based on this use case and implementation, Polars is 100x times faster than R, and R seems to be more than 3x times slower than the slowest Python implementation we considered (the `python` engine of pandas).

## Can we be faster than Polars?

Until this point, we can conclude that for this use case Polars is the fastest option of the tools I'm aware of. But can we be even faster?

And the answer is yes, since Polars is a generic tool that supports most CSV files, but I can implement an algorithm targeted at this particular file. For example, Polars supports unicode files, files delimited by semicolon, quoted text... which we do not need for this task.

The second question is, is it easy to implement an algorithm faster than Polars in a low level language? The answer is clearly no.

A first implementation of my CSV parser in Rust is next:

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

As you can see, the implementation is quite simple, and while Rust is an extremely fast low level language in many cases the code is as readable as Python. This is not the only reason to use Rust, as probably the biggest advantage is the language design and the smart compiler, that catch most bugs at compilation time, in particular most bugs that would make a C/C++ program segfault.

To make the comparison with Polars meaningful, we need to move Polars outside of the Python world. The nice thing is that Polars is also built in Rust and we can run Polars code without the need of the Python interpreter, which would add a significant overhead. This is the implementation of our task in Polars using its SQL API (Polars also have a dataframe API in the Rust world similar to the Python one, but I'll use the SQL API here):

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

Note that defining the schema is not really needed, and the code would be significantly shorter without it. But making Polars infer the data types would be unfair when comparing with my custom implementation.

Comparing the times of both implementations I get these results:

| Description | Time (seconds) |
| --- | --- |
| My implementation | 0.563 |
| Polars 1 CPU core | 0.483 |
| Polars all CPU cores | 0.150 |

The interesting part here is that Polars is not only faster than my very simple solution, but it's still faster when forcing Polars to use only one CPU (my implementation only uses one CPU, but Polars by default will use all avaialble ones).

Luckily, this is not the end. There are some reasons why a generic implementation like Polars is faster, and some things that can be done to improve my implementation.

After some analysis, I found out that the way Rust is processing the file with the `csv` crate / package causes a much higher number of branch mispredictions than iterating the file character by character. To briefly explain what a branch misprediction is, let's say that a CPU has a queue with all the instructions it needs to execute, so the next is ready when one finishes. But then code has conditional statements (an `if`) and only one of the code when the condition is `true` or when the condition is `false` can be the next in the queue. So, when populating the queue the operating system will make a guess on what the result of the condition is going to be before evaluating it and really knowing. And it'll add to the queue the instructions that seems more likely to be the next. When the guess is wrong, it takes some time to bring the actual code that needs to run to the queue from outside the CPU. When we write code at this low level, and we care about performance to the millisecond we want to help the compiler and the operating system make the best guesses so the queue contains the right code as often as possible and the minimum amount of time is lost.

Another factor is how the file is accessed from disk. There are different options, like loading all the file into memory before parsing it, loading one row at a time... The fastest option for this kind of operation is using a memory map. Memory maps are a feature (syscall) of operating systems where the kernel will map virtual memory addresses to a file in disk. When using memory maps the Linux kernel is quite smart at preloading data into memory before it's needed, as well as to preload the data into the memory caches. It can be even smarter if we give the kernel hints on how this memory is going to be accessed. In our case, we are going to access the file in sequential order, so if we let the Linux kernel know this when setting up the memory map, the kernel is going to speed up our program significantly.

A more obvious factor is paralellization. Modern computers will in general have multiple CPUs, and we can split the work the CPU needs to do among them. So, if we have work for 1 second of CPU, but we have 4 CPUs, we can get our program finish in 0.25 seconds. In practice the time will be much more than 0.25 seconds, but we can surely speed up things significantly. Also, many modern CPUs provide parallelism in a single CPU via SIMD operations. Where an operation can be applied to multiple values at the same time. I didn't implement SIMD in my tests here, but that could be another option.

Finally, there is one last optimization we could perform. In general, when processing a CSV we want to first extract each value as a string, and then cast it to the appropriate type. In this case, since I was mostly parsing integer values, I can build the number while reading the characters. So, if I first find a `"2"` I just cast it to an integer. If after that I find a `"4"`, I multiply the previous value `2` by `10` and add the `4` I just found to get the 24\. If I keep repeating this until I find the comma, I already have the number parsed. This approach seems to be significantly faster than parsing the number from a reference to the whole string. What's even better in this particular case, since I'll square all numbers, I don't care if they are positive or negative, and I can completely ignore the sign.

After implementing those optimizations, I managed to achieve the next execution times:

| Description | Time (seconds) |
| --- | --- |
| My original implementation | 0.563 |
| Polars 1 CPU core | 0.483 |
| Polars all CPU cores | 0.150 |
| My optimized implementation single core | 0.241 |
| My optimized implementation multicore | 0.070 |

So, whether using a single CPU or all, my final implementation is 2x times faster than Polars. Obviously, using Polars or another generic tool makes more sense in most cases, as the development time of implementing the pipeline is a fraction than implementing the whole parsing from scratch. But in cases when performance is a top priority, these results seem to show that it's possible to process CSV files significantly faster than the faster generic implementation.

I hope you enjoyed this post, feel free to follow me on [Twitter](https://twitter.com/datapythonista) to stay updated of future posts. And feel free to connect and contact me on [LinkedIn](https://www.linkedin.com/in/datapythonista/) if you think the article needs fixes, or if your company needs help with related work or you want to discuss job opportunities.

Follow me for more content or contact for work opportunities:
[Twitter](https://twitter.com/datapythonista) / [LinkedIn](https://www.linkedin.com/in/datapythonista/)