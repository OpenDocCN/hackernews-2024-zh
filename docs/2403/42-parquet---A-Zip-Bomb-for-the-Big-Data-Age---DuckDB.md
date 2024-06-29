<!--yml

category: 未分类

date: 2024-05-29 12:41:35

-->

# 42.parquet – 大数据时代的 Zip Bomb – DuckDB

> 来源：[https://duckdb.org/2024/03/26/42-parquet-a-zip-bomb-for-the-big-data-age.html](https://duckdb.org/2024/03/26/42-parquet-a-zip-bomb-for-the-big-data-age.html)

# 42.parquet – 大数据时代的 Zip Bomb

Hannes Mühleisen2024-03-26

*TL;DR：一个 42 kB 的 Parquet 文件可以包含超过 4 PB 的数据。*

[Apache Parquet](https://parquet.apache.org) 已经成为表格数据交换的事实标准。通过使用二进制、列式和*压缩*的数据表示，它远比其可怕的表兄弟 CSV 要好得多。此外，Parquet 文件附带足够的元数据，使得文件可以在没有额外信息的情况下被正确解释。大多数现代数据工具和服务都支持读写 Parquet 文件。

然而，Parquet 文件并非没有危险：例如，损坏的文件可能会使那些在解释内部偏移量等方面不太谨慎的读取器崩溃。但即使是完全有效的文件也可能存在问题，导致崩溃和服务停机，正如我们将在下面展示的那样。

对于天真的防火墙和病毒扫描器，一个相当知名的攻击是[Zip Bomb](https://en.wikipedia.org/wiki/Zip_bomb)，其中一个著名的例子是[42.zip](https://www.unforgettable.dk)，之所以命名为 42 是因为当然[42 是完美的数字](https://en.wikipedia.org/wiki/42_(number)#The_Hitchhiker's_Guide_to_the_Galaxy)，而且该文件只有 42 千字节大小。这个完全有效的 zip 文件里包含了许多其他的 zip 文件，再次包含其他的 zip 文件，以此类推。最终，如果试图解压所有这些，你将得到 4 PB 的数据。果然是大数据。

Parquet 文件支持多种方法来压缩数据。只有 42 千字节大小的 Parquet 文件能创建多大的表？让我们来看看！为了可移植性的原因，我们为 DuckDB 实现了我们自己的[Parquet 读写器](/docs/data/parquet/overview)。实现它时，学习 Parquet 格式是不可避免的。

一个 Parquet 文件由一个或多个行组组成，行组包含列，而列则包含所谓的包含实际数据的页。在其他编码方式中，Parquet 支持[字典编码](https://en.wikipedia.org/wiki/Dictionary_coder)，其中我们首先有一个包含字典的页，然后是引用字典而不是包含普通值的数据页。对于长值如分类字符串经常重复的列，这种方式更为高效，因为字典引用可以更小。

让我们利用这一点。我们写入一个只有一个值的字典，并反复引用它。在我们的示例中，我们使用一个64位整数，这是可能的最大值，因为为什么不呢？然后，我们使用parquet中指定的`RLE_DICTIONARY` [行程长度编码](https://en.wikipedia.org/wiki/Run-length_encoding) 来引用这个字典条目。所指定的编码有些奇怪，因为由于某种原因它结合了位填充和行程长度编码，但基本上我们可以使用可能的最大行程长度，即`2^31-1`，略高于20亿。由于字典很小（一个条目），我们重复的值是0，指向唯一的条目。包括其所需的元数据头和尾（就像Parquet中的所有元数据一样，这是使用[Thrift](https://thrift.apache.org)编码的），此文件仅133字节大。以133字节表示20亿个8字节整数并不算太糟糕，即使它们都是相同的值。

但我们可以从这里开始。列可以包含多个页面，引用*同一个*字典，因此我们可以一次又一次地重复我们的数据页，每次只向文件添加31个字节，但向文件所代表的表中添加20亿个值。我们还可以使用另一个技巧来增加数据大小：正如前面提到的，Parquet 文件包含一个或多个行组，这些行组存储在文件末尾的一个 Thrift 尾部中。该行组中的每列包含字节偏移量（`data_page_offset`和相关内容），指向存储列页面的文件中的位置。没有什么能阻止我们添加多个行组，*所有这些行组都引用同一个字节偏移量*，这个偏移量是我们存储了些许顽皮字典和数据页面的地方。我们添加的每个行组在逻辑上都重复所有页面。当然，添加行组还需要存储元数据，因此在添加页面（20亿个值）和行组（比它重复的任何其他行组多2倍）之间存在某种权衡关系。

经过一些调整，我们发现如果将数据页面重复1000次，并重复行组290次，我们最终得到一个[Parquet 文件](https://github.com/hannes/fortytwodotparquet/raw/main/42.parquet)，大小为42千字节，但包含*622万亿*个值（准确地说是622,770,257,630,000）。如果将此表在内存中实现，将需要超过*4PB*的内存，这终于是一个真实的[大数据](https://motherduck.com/blog/big-data-is-dead/)示例，巧合地与上述原始`42.zip`大小大致相同。

我们还提供了[生成此文件所用的脚本](https://github.com/hannes/fortytwodotparquet/blob/main/create-parquet-file.py)，希望它能够更好地用于测试 Parquet 读取器。我们希望已经表明，Parquet 文件可能会带来一些危害，因此在将其插入到某些流程中时一定要格外小心。虽然 DuckDB *可以* 从我们的文件中读取数据（例如，通过使用 `LIMIT`），但如果你要它完整地读取所有数据，最好先泡杯咖啡。
