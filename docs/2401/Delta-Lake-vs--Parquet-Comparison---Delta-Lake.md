-\>yml

分类：未分类

日期：2024-05-27 14:56:02

-\>

# Delta Lake vs. Parquet Comparison | Delta Lake

> 来源：[https://delta.io/blog/delta-lake-vs-parquet-comparison/](https://delta.io/blog/delta-lake-vs-parquet-comparison/)

本文介绍了 Delta Lake 和 Parquet 表格之间的区别，以及为什么 Delta Lake 几乎总是实际用例中更好的选择。Delta Lake 具有 Parquet 表格的所有优点以及许多其他对数据从业者至关重要的功能。这就是为什么使用 Delta Lake 而不是 Parquet 表格几乎总是有利的原因。

当数据在单个文件中时，Parquet 表格是可以的，但是当数据在多个文件中时，它们很难管理并且不必要地慢。Delta Lake 可以轻松地管理多个 Parquet 文件中的数据。

让我们比较一下 Parquet 表格和 Delta 表格的基本结构，以更好地理解 Delta Lake 的优势。

## Parquet 文件的基本特征

Parquet 是一种不可变的、二进制的、列式文件格式，与 CSV 等基于行的格式相比具有几个优点。以下是 Parquet 文件与 CSV 文件相比的核心优势：

+   Parquet 文件的列式特性允许查询引擎挑选单独的列。对于基于行的文件格式，查询引擎必须读取所有列，即使对查询无关的列也是如此。

+   Parquet 文件在元数据中包含模式信息，因此查询引擎不需要推断模式，/ 用户在读取数据时不需要手动指定模式。

+   与基于行的文件格式相比，列式文件格式如 Parquet 文件更易于压缩。

+   Parquet 文件将数据存储在行组中。每个行组都有每列的最小/最大统计信息。Parquet 允许查询引擎针对特定查询跳过整个行组，这在读取数据时可能会带来巨大的性能提升。

+   Parquet 文件是不可变的，阻止了手动更新源数据的反模式。

查看 [这个视频以了解 Parquet 优于 CSV 的五个原因](https://www.youtube.com/watch?v=9LYYOdIwQXg)。

您可以将小型数据集保存在单个 Parquet 文件中，而不会出现可用性问题。通常，对于小数据集，单个 Parquet 文件提供给用户的数据分析体验要比 CSV 文件好得多。

但是，数据从业者经常将大型数据集拆分成多个 Parquet 文件。管理多个 Parquet 文件并不理想。

## 在多个 Parquet 文件中存储数据的挑战

Parquet 表格由数据存储中的文件组成。以下是磁盘上一堆 Parquet 文件的样子。

[复制](#)

```
some_folder/
  file1.parquet
  file2.parquet
  …
  fileN.parquet 
```

单个 Parquet 文件的许多可用性优势也适用于 Parquet 数据湖 - 在一个 Parquet 文件或多个 Parquet 文件上进行列修剪非常容易。

使用 Parquet 表格的一些挑战包括：

+   Parquet 数据湖没有 ACID 事务

+   从 Parquet 表格中删除行不容易

+   没有 DML 事务

+   没有变更数据反馈

+   缓慢的文件列表检索开销

+   获取用于跳过文件的统计信息的昂贵页脚读取

+   没有办法重命名、重新排序或删除列而不重写整个表。

+   还有很多

Delta Lake 使管理 Parquet 表变得更容易和更快。Delta Lake 也经过优化，可以防止数据表损坏。让我们看看 Delta Lake 是如何构造的，以了解它是如何提供这些功能的。

## Delta Lake 的基本结构

Delta Lake 在事务日志中存储元数据，表数据存储在 Parquet 文件中。以下是 Delta 表的内容。

[复制](#)

```
some_folder/
  _delta_log
    00.json
    01.json
    …
    n.json
  file1.parquet
  file2.parquet
  …
  fileN.parquet 
```

这是 Delta 表的可视化表示：

你可以通过查看[协议](https://github.com/delta-io/delta/blob/master/PROTOCOL.md)来查看完整的 Delta Lake 规范。让我们看看 Delta Lake 如何使文件列表操作更快。

## Delta Lake vs. Parquet：文件列表

当你想要读取 Parquet 数据湖时，你必须执行文件列表操作，然后读取所有数据。在列出所有文件之前，你无法读取数据。请参见以下示例：

在 UNIX 文件系统上，列出文件并不太昂贵。对于云中的数据，文件列表操作速度较慢。基于云的文件系统是键/值对象存储，与类 UNIX 文件系统不相似。键值存储在列出文件时速度较慢。

Delta Lake 将 Parquet 文件的路径存储在事务日志中，以避免执行昂贵的文件列表。Delta Lake 不需要列出云对象存储中的所有 Parquet 文件来获取它们的路径。它只需在事务日志中查找文件路径。

云对象存储不擅长列出嵌套在目录中的文件。在基于云的系统中使用 Hive 风格分区存储的文件可能需要花费几分钟甚至几小时来计算文件列表操作。

最好依赖于事务日志来获取表中文件的路径，而不是执行文件列表操作。

## Delta Lake vs. Parquet：小文件问题

增量更新的大数据系统可能会生成大量小文件。当增量更新频繁发生且针对 Hive 分区数据集时，小文件问题尤为突出。

当读取具有许多小文件的数据集时，数据处理引擎性能表现不佳。通常希望文件大小在 64 MB 到 1 GB 之间。你不希望有 1 KB 的微小文件，这些文件需要过多的 I/O 开销。

数据从业者通常希望将小文件合并成较大的文件，这个过程称为“小文件合并”或“装箱”。

假设你有一个包含 10,000 个查询速度缓慢的小文件的数据集。你可以将这 10,000 个小文件合并成一个包含 100 个合适大小文件的数据集。

如果你正在使用普通的 Parquet 数据湖，你需要自己编写小文件合并代码。使用 Delta Lake，你只需简单运行`OPTIMIZE`命令，Delta Lake 将为你处理小文件合并。

ETL 管道通常会处理新文件。在普通的 Parquet 数据湖中，有两种类型的新文件：新数据和已压缩成较大文件的旧数据。您不希望下游系统重新处理已处理过的旧数据。Delta Lake 有一个 `data_change=False` 标志，可以让下游系统区分新数据和只是现有数据的压缩版本的新文件。Delta Lake 对于生产 ETL 管道来说更加优秀。

Delta Lake 使小文件压缩比普通的 Parquet 表更容易。查看[此博客文章关于使用 OPTIMIZE 进行小文件压缩](https://delta.io/blog/2023-01-25-delta-lake-small-file-compaction-optimize/)以了解更多信息。

## Delta Lake 对比 Parquet：ACID 事务

数据库支持事务，与不支持事务的数据系统相比，可以防止一系列数据错误。

Parquet 表不支持事务，因此它们很容易损坏。假设您正在向现有 Parquet 数据湖追加大量数据，并且在写操作中间集群失败。那么，您的表中将有几个部分写入的 Parquet 文件。

部分写入的文件会破坏后续的读取操作。计算引擎将尝试读取损坏的文件并报错。您需要手动识别所有损坏的文件并删除它们以修复数据湖。一个损坏的表通常会破坏组织中的许多数据系统，并需要紧急修复 - 非常不愉快。

Delta Lake 支持事务，因此您永远不会因为写操作在中途出错而损坏 Delta Lake。如果在向 Delta 表写入数据时集群失败，Delta Lake 将简单地忽略部分写入的文件，后续的读取操作不会中断。事务还有许多其他好处，这只是一个例子。

## Delta Lake 对比 Parquet：列剪枝

如果能够将更少的数据发送到计算集群，查询运行速度就会更快。基于列的文件格式允许您从表中挑选特定的列，而基于行的文件格式则需要将所有列发送到集群。

Delta Lake 和 Parquet 都是列式存储的，因此您可以通过列剪枝（也称为列投影）从数据集中挑选特定的列。与 Parquet 相比，列剪枝对 Delta Lake 并不是一个优势，因为它们都支持此功能。Delta Lake 在底层使用 Parquet 文件存储数据。

但是，对于存储在类似 CSV 或 JSON 这样的基于行的文件格式中的数据，无法进行列剪枝，因此与基于行的文件格式相比，这对 Delta Lake 是一个重大的性能优势。

## Delta Lake 对比 Parquet：文件跳过

Delta 表在事务日志中存储关于底层 Parquet 文件的元数据信息。读取 Delta 表的事务日志并找出可以跳过的文件是非常快速的。

Parquet 文件在页脚中存储行组的元数据，但获取所有页脚并构建整个表的文件级别元数据速度较慢。它需要一个文件列表操作，而我们已经讨论过文件列表操作可能很慢。

Parquet 不支持文件级别的跳过，但可以进行行组过滤。

## Delta Lake 与 Parquet：谓词推送过滤

Parquet 文件在页脚中包含的元数据统计信息可以被数据处理引擎利用，以更有效地运行查询。

发送较少的数据到计算集群是使查询运行更快的一个好方法。您可以通过发送更少的列或行数据到引擎来实现这一点。

Parquet 文件页脚还包含文件中每列的最小/最大统计信息（技术上，最小/最大统计信息是针对每个行组跟踪的，但让我们保持简单）。根据查询的方式，您可以跳过整个行组。例如，假设您正在运行一个过滤操作，并且想要找到所有 `col4=65` 的值。如果有一个 Parquet 文件，其中 `col4` 的最大值为 34，您就知道该文件不包含您查询所需的任何相关数据。您可以完全跳过它。

数据跳过的效果取决于您的查询可以跳过多少个文件。但这种策略可以提供 10 倍至 100 倍甚至更多的速度提升 - 这是至关重要的。

当您读取单个 Parquet 文件时，将元数据放在 Parquet 文件页脚中是可以接受的。如果您有 10,000 个文件，您不希望必须读取所有文件的页脚，收集整个湖泊的统计信息，然后再运行查询。那将是太多的开销。

Delta Lake 将元数据统计信息存储在事务日志中，因此查询引擎在运行查询之前不需要读取所有单独的文件并收集统计信息。从事务日志中获取统计信息要更有效率。

## Delta Lake 的 Z Order 索引

当数据被 Z Ordered 时，跳过操作效率更高。当相似的数据位于相邻位置时，可以跳过更多的数据。

[为什么 Delta Lake 是 pandas 分析的最佳存储格式](https://www.youtube.com/watch?v=A8bvJlG6phk) 的 Data & AI Summit 演讲展示了如何在 Delta 表中对数据进行 Z Ordering 可以显著减少查询的运行时间。

在 Delta 表中对数据进行 Z Ordering 是很容易的。在 Parquet 表中对数据进行 Z Ordering 则不容易。请参阅有关 [Delta Lake Z Order](https://delta.io/blog/2023-06-03-delta-lake-z-order/) 的博客文章以了解更多信息。

## Delta Lake 与 Parquet：重命名列

Parquet 文件是不可变的，因此无法修改文件以更新列名。如果要更改列名，请将其读入 DataFrame，更改名称，然后重新写入整个文件。重命名列可能是一项昂贵的计算。

Delta Lake 抽象了物理列名和逻辑列名的概念。物理列名是 Parquet 文件中的实际列名。逻辑列名是人类在引用列时使用的列名。

Delta Lake 允许用户通过更改逻辑列名快速重命名列，这是一个纯元数据操作。这只是 Delta 事务日志中的一个简单条目。

并没有快速的方法来更新 Parquet 表的列名。您需要读取所有数据，重命名列，然后重新编写所有数据。对于大数据集来说，这是一个缓慢的过程。

## Delta Lake vs. Parquet：删除列

Delta Lake 还允许您快速删除列。您可以向 Delta 事务日志添加一个条目，并指示 Delta 在未来操作中忽略列 - 这是一个纯元数据操作。

Parquet 表要求您读取所有数据，通过查询引擎删除列，然后重新编写所有数据。这对于一个相对较小的操作来说是一次广泛的计算。

有关如何从 Delta 表中删除列的更多信息，请参阅[此博客文章](https://delta.io/blog/2022-08-29-delta-lake-drop-column/)。

## Delta Lake vs. Parquet：模式强制执行

通常情况下，您希望允许追加具有与现有表匹配的模式的 DataFrame，并拒绝具有不匹配模式的 DataFrame 的追加。

对于 Parquet 表，您需要手动编写此模式强制执行。默认情况下，您可以将任何模式的 DataFrame 追加到 Parquet 表中（除非它们已在元数据存储中注册，并通过元数据存储提供模式强制执行）。

Delta Lake 具有内置的模式强制执行，可防止可能损坏 Delta Lake 的昂贵错误。有关模式强制执行的更多信息，请参阅[此文章](https://delta.io/blog/2022-11-16-delta-lake-schema-enforcement/)。

您还可以在 Delta 表中绕过模式强制执行，并随时间更改表的模式。

## Delta Lake vs. Parquet：模式演化

有时，您可能想要向 Delta Lake 添加额外的列。也许您依赖于数据供应商，他们已经向您的数据源添加了新列。您不希望以空列重写所有现有数据，以便可以向表中添加新列。您希望有一些模式灵活性。您只想写入具有附加列的新数据，并保留所有现有数据不变。

Delta Lake 允许模式演化，因此您可以无缝地向数据集添加新列，而无需运行大型计算。这是另一个通常在实际数据应用程序中非常有用的便利功能。有关模式演化的更多信息，请参阅[这篇博文](https://www.databricks.com/blog/2019/09/24/diving-into-delta-lake-schema-enforcement-evolution.html)。

假设您将DataFrame追加到具有不匹配架构的Parquet表中。在这种情况下，您必须记住每次读取表时设置特定选项，以确保准确的结果。查询引擎通常在确定Parquet表的架构时采取捷径。它们查看一个文件的架构，然后假定所有其他文件具有相同的架构。

当手动设置标志时，引擎可以查看Parquet表中所有文件的架构，以确定整个表的架构。检查所有文件的架构会更加耗费计算资源，因此默认情况下不会设置。Delta Lake的架构演变比Parquet提供的更好。

## Delta Lake vs. Parquet：检查约束

你也可以对列应用自定义SQL检查，以确保追加到表中的数据具有指定的形式。

仅仅检查字符串列的架构可能还不够。您可能还希望确保字符串与某种正则表达式模式匹配，并且列不包含`NULL`值。

Parquet表不支持像Delta Lake那样的检查约束。了解更多，请参阅[Delta Lake Constraints and Checks](https://delta.io/blog/2022-11-21-delta-lake-contraints-check/)的博客文章。

## Delta Lake vs. Parquet：版本化数据

Delta表可以有许多版本，用户可以轻松地在不同版本之间“时间旅行”。版本化数据在监管要求、审计目的、实验和撤消错误时非常有用。

版本化数据还会影响引擎执行某些事务的方式。例如，当你“覆盖”Delta表时，你不会从存储中物理删除文件。你只是将现有文件标记为已删除，但实际上并不删除它们。这被称为“逻辑删除”。

Parquet表不支持版本化数据。当您从Parquet表中删除数据时，您实际上是从存储中删除它，这被称为“物理删除”。

逻辑数据操作更好，因为它们更安全，允许撤消错误。如果您覆盖了Parquet表，则是不可逆的错误（除非有单独的机制备份数据）。在Delta表中撤消覆盖事务很容易。

了解更多，请参阅[Why PySpark append and overwrite operations are safer in Delta Lake than Parquet tables](https://delta.io/blog/2022-11-01-pyspark-save-mode-append-overwrite-error/)的博客文章。

## Delta Lake vs. Parquet：时间旅行

版本化数据还允许您轻松切换到您的Delta Lake的不同版本之间，这被称为时间旅行。

时间旅行在各种情况下都很有用，详细描述请参阅[Delta Lake Time Travel](https://delta.io/blog/2023-02-01-delta-lake-time-travel/)的文章。Parquet表不支持时间旅行。

Delta Lake 需要保留一些数据版本以支持时间旅行，如果您不需要历史数据版本，则会增加不必要的存储成本。Delta Lake 为您可选地删除这些遗留文件提供了便利。

## Delta Lake `vacuum`命令

您可以使用 Delta Lake 的`vacuum`命令删除旧的遗留文件。例如，您可以将保留期设置为 30 天，然后运行`vacuum`命令，这将允许您删除所有超过 30 天的不必要数据。

当然，它不会删除所有超过 30 天的旧数据。如果当前版本的 Delta Lake 中仍需要“旧”数据，则不会删除它。

一旦运行了`vacuum`命令，就无法回滚到 Delta Lake 的早期版本。例如，如果您将保留期设置为 7 天并执行了`vacuum`命令，那么无法在 60 天前的 Delta Lake 版本中进行时间旅行。

有关 Delta Lake `vacuum`命令的更多信息，请参阅这篇博客文章[the Delta Lake VACUUM command](https://delta.io/blog/2023-01-03-delta-lake-vacuum-command/)。

## Delta Lake 回滚

Delta Lake 还可以轻松将整个 lake 回退到较早版本。假设您在星期三插入了一些数据，后来发现数据有误。您可以轻松将整个 Delta Lake 回滚到星期二的状态，从而撤消所有在星期三所做的错误。

如果您已经运行了`vacuum`命令，就不能将 Delta Lake 回滚到比保留期更早的版本。因此，在清理 Delta Lake 之前务必小心。

有关如何使用还原来回退 Delta Lake 表到先前版本的博客文章，请参阅[How to Rollback a Delta Lake Table to a Previous Version with Restore](https://delta.io/blog/2022-10-03-rollback-delta-lake-restore/)。

## Delta Lake vs. Parquet：删除行

您可能希望能够从表中删除行，尤其是为了符合诸如 GDPR 等法规要求。Delta Lake 可以轻松执行最小化删除操作，而从 Parquet lake 删除行则不容易。

假设您有一个用户希望删除其帐户并从系统中删除所有数据。您的表中存储了部分他们的数据。您的表有 50,000 个文件，而这位特定客户的数据存在于其中的 10 个文件中。

Delta Lake 可以轻松运行删除命令，并有效地重写受影响的十个文件而不包含客户数据。Delta Lake 还可以轻松编写标记已删除行的文件（删除向量），从而使此操作运行得更快。

如果您有一个 Parquet 表，唯一方便的操作就是读取所有数据，过滤出特定用户的数据，然后重新写入整个表。这将花费很长时间！

手动识别包含用户数据的 10 个文件并重写这些特定文件是繁琐且容易出错的。这正是您更愿意将其委托给您的 Lakehouse 存储系统而非自己执行的任务类型。

查看关于[如何从 Delta Lake 表中删除行](https://delta.io/blog/2022-12-07-delete-rows-from-delta-lake-table/)的博客文章以了解更多信息。同时，确保查看关于[Delta Lake 删除向量](https://delta.io/blog/2023-07-05-deletion-vectors/)的博客文章，以了解删除操作可以更快地运行的原因。

## Delta Lake 对比 Parquet：合并事务

Delta Lake 提供了一个强大的合并命令，允许您更新行、执行插入更新、构建慢变化维度表等。

Delta Lake 使执行合并命令并在幕后高效更新最小数量的文件变得简单，类似于高效实现删除命令。

如果你使用 Parquet 表，就无法使用合并命令。你需要自己实现所有底层的合并细节，这是具有挑战性且耗时的。

参阅[Delta Lake 合并博客文章](https://delta.io/blog/2023-02-14-delta-lake-merge/)以及它如何使数据操作语言操作（`INSERT`、`UPDATE` 和 `DELETE`）更容易。

## Delta Lake 表的其他优点

Delta Lake 具有许多其他优点，本文未讨论，但您可以查阅这些文章以了解更多信息：

## 结论

本文向您展示了 Delta Lake 通常比 Parquet 表更好。Delta Lake 使执行常见的数据操作变得简单，如删除列、重命名列、删除行和 DML 操作。Delta Lake 还支持事务和模式强制执行，因此您很少会损坏表格。Delta Lake 将文件元数据抽象到一个事务日志中，并支持 Z Ordering，因此您可以更快地运行查询。

当您与不支持 Delta Lake 的系统进行接口时，Parquet 表仍然是有用的。如果下游系统无法读取 Delta Lake 格式，则可能需要将 Delta Lake 转换为 Parquet 表。Delta 表将数据存储在 Parquet 文件中，因此从 Delta 表转换为 Parquet 表很容易。使用其他系统读取 Delta 表是一个微妙的话题，但已构建了许多 Delta Lake 连接器，因此您很可能能够使用所选查询引擎读取 Delta 表。

查看[此博客文章](https://delta.io/blog/2022-08-02-delta-2-0-the-foundation-of-your-data-lake-is-open/)以了解更多关于不断增长的 Delta Lake 连接器生态系统的信息。
