<!--yml

category: 未分类

日期：2024-05-27 13:08:13

-->

# ULIDs和主键 | Dave Allie

> 来源：[https://blog.daveallie.com/ulid-primary-keys](https://blog.daveallie.com/ulid-primary-keys)

当选择数据库主键类型时，意见分歧。在为[Visibuild](https://www.linkedin.com/company/visibuild-pty-ltd/)做出这个决定时，我必须在顺序ID的简单性和非顺序ID的长期/未来益处之间选择。我选择了非顺序ID以便未来更容易处理分片和区域数据库。在众多非顺序ID的选择中，我选择了ULIDs。

这是我到达那里的方法。

让我们从大多数人开始的地方开始，使用UUIDs。

UUIDs（通用唯一标识符）是128位标签，通常表示为十六进制字符串，分为5个部分，分别包含32位、16位、16位、16位和48位。

这里是一个例子：

在UUID标准中（您可以在[RFC 4122](https://datatracker.ietf.org/doc/html/rfc4122)中了解详情），有5个不同版本的相同128位标签。这些版本之间的区别在于生成UUID所需的输入以及输出位的结构。

第1版需要知道计算机的MAC地址才能生成标签。第2版只有在每台计算机大约每7分钟生成至多一个时才能保证唯一。第3版和第5版基于提供的输入是确定性的。这使得第4版是用于可扩展的、非确定性UUID的合适选择。

当人们说UUID时，他们几乎总是指的是UUIDv4。它是最普遍和广泛支持的UUID标准，即使上面给出的示例UUID也是UUIDv4。由于UUIDv4仅基于随机性，因此它非常易于移植，并且可以在几乎不需要关于系统状态的先验知识的情况下使用。

UUIDv4由**122**位的随机性和**6**位的版本/变体标识组成。

info_outline

点击“重新生成”按钮以查看几个不同的UUIDv4。

UUIDv4中唯一的非随机部分是版本和变体。版本由4位表示，并且对于UUIDv4始终设置为4，您可以看到它作为上面第一个橙色nibble突出显示。变体由2位表示，并且对于UUIDv4标准设置为2，您可以看到它作为上面第二个橙色nibble突出显示。

info_outline

变体仅是第二个突出显示的nibble的前2位。而其余2位是随机的，意味着该nibble的值可以是8 (

`1000`

), 9 (

`1001`

), a (

`1010`

), 或 b (

`1011`

). 虽然RFC中只有5种UUID变体，但有一个[草案](https://datatracker.ietf.org/doc/html/draft-peabody-dispatch-new-uuid-format)，其中包括3种新的：UUIDv6、UUIDv7和UUIDv8。

这三个新版本都基于类似的结构：将时间位放在最重要的位置，然后是随机位的组合。它们之间的区别在于使用哪种类型的时钟来生成时间位。草案中的第7版使用Unix Epoch时间，因此很可能会在未来被采用。

即使在第7版中，也有几种不同的表示时间的方式。可以使用不同级别的子秒精度，但时间越精确，可以使用的随机位数就越少。让我们来看看UUIDv7的毫秒精度设置。

通过毫秒精度的设置，时间有**48**位，版本/变体有**6**位，单调序列有**12**位，随机性有**62**位。

UUIDv7始终将前36位用于时间戳的秒数，然后根据子秒精度使用可变数量的位数，例如我们的情况下，用于每秒内的毫秒数的位数是12位。

info_outline

如果你点击"重新生成"几次，你应该会看到时间戳的前9个字符每秒只改变一次，这些位表示的是Unix Epoch时间的秒数。最后的三位是该秒内的毫秒数。

版本和变体遵循与UUIDv4相同的设置，但版本设置为7。

UUID前面的时间戳的主要价值在于UUIDv7（以及草案中的其他新UUID）在同一子秒内是单调递增的，这意味着现在生成的UUIDv7始终比一毫秒前生成的UUIDv7要大。

您还会注意到UUIDv7中的一种新的部分，即"单调序列"部分。这个12位的部分旨在确保在同一子秒内生成的UUID也是单调递增的。在我们的情况下，对于在同一毫秒内生成的每个后续UUIDv7，单调序列计数器将增加一。请注意，这仅对在同一台机器上在同一毫秒内生成的UUIDs有意义，如果它们在同一毫秒内独立生成，则不能确保UUIDs的任何排序。

以下是在同一毫秒内生成的一些示例UUIDs：

*   `061ff8d8-e24b-7000-8092-ca1e5440d491`

+   `061ff8d8-e24b-7001-b653-1c41e471cd78`

+   `061ff8d8-e24b-7002-9bc2-82b5da559f1d`

当使用UUIDv7作为主键时，您获得了与顺序ID相同的可排序性，同时具备分布式生成的灵活性。如果您希望对创建时间使用UUIDv4进行高效排序，则需要一个单独的带索引的创建时间列，但如果使用UUIDv7，则可以使用主键作为排序列（因为它已经被索引）。

在UUIDv7规范起草之前的约4年，普遍唯一字典排序标识符（或ULID）作为一个基于GitHub的提案而非RFC草案提出。您可以在GitHub上阅读（相当简洁的）[ULID规范](https://github.com/ulid/spec)。

ULID是一个128位标签，就像UUID一样。它是可排序的，具有毫秒精度，并且是单调递增的，就像UUIDv7一样。

ULID通常表示为[Crockford的Base 32](http://www.crockford.com/base32.html)编码字符串，而不是UUID的十六进制字符串。例如，与`017eb31e-1440-b69e-d82f-5f0937f823c8`相同的值可以表示为`0GWWXY2G84DFMRVWQNJ1SRYCMC`。

为了与UUID进行比较，我将把所有ULID表示为十六进制字符串，采用相同的8、4、4、4、12段。

在ULID中，有**48**位时间和**80**位随机性。

尽管UUIDv7（具有毫秒精度）和ULID中都有48位时间信息，但ULID标准将整个48位时间编码为Unix Epoch的毫秒数，而UUIDv7将时间分为36秒位和12毫秒位。

ULID还移除了与版本和变体相关的位，给我们6位额外的空间，但移除了ULID的标识。因为UUID在标签中编码版本和变体信息，所以可以可靠地解码为其组成部分（例如时间戳和随机性）。如果要从ULID中提取时间戳，客户端需要知道正在处理的字符串是ULID，而不是无效的（或者看起来可能有效的）UUID。

ULID是单调递增的，但在同一台机器上在同一毫秒内生成的ULID将具有顺序随机部分。

这里是在同一毫秒内生成的一些ULID示例：

*   `017ece40-2a1e-63ac-a58d-e336f30c1d76`

+   `017ece40-2a1e-63ac-a58d-e336f30c1d77`

+   `017ece40-2a1e-63ac-a58d-e336f30c1d78`

ULID是UUIDv7的一个很好的替代方案。它们包含更多的随机性和简单的结构，但代价是不显式暴露版本、变体或单调计数器。相比之下，如果你在编写一些末日场景软件（假设你有一个工作的计算机），UUID是完美的选择，因为它们可以一直生成直到公元10889年，而UUIDv7仅能生成到公元4147年。

## UUIDv7和ULID的主要区别是：

*   UUIDv7

+   UUIDv7将在公元4147年前工作，而ULID将在公元10889年前工作。

| 格式 | 可排序 | 单调 | 随机性 |
| --- | --- | --- | --- |
| UUIDv4 | 否 | 否 | 122位 |
| UUIDv7 | 是 | 是 | 62位 |
| ULID | 是 | 是 | 80位* |

* - 在同一毫秒内，随机位按顺序递增。

## 在PostgreSQL中实现UUIDs/ULIDs

info_outline

这一节专为 PostgreSQL 设计，如果你使用不同的数据库引擎，你需要找其他地方了解如何实现这种类型的主键。

尽管所有这些格式都可以由客户端在插入数据库之前生成，但为了简单和一致性的目的，最好在数据库引擎内部生成它们。

幸运的是，PostgreSQL 包含了 [`uuid` 数据类型](https://www.postgresql.org/docs/14/datatype-uuid.html)，接受大小写不敏感的 128 位十六进制字符串，将其解析为二进制数据并存储为 16 字节的数据。这比将值作为字符串存储在数据库中要好得多，后者包括破折号占用 37 字节的空间，或者去除破折号后占用 33 字节的空间。

这里是一些用于填充那些 `uuid` 列并设置具有 UUID/ULID 主键表的实现细节：

PostgreSQL 通过 `pgcrypto` 或 `uuid-ossp` 扩展内置支持 UUIDv4。

```

```

CREATE EXTENSION IF NOT EXISTS pgcrypto;

SELECT gen_random_uuid(); --> f449a5bc-a221-4e9d-8819-e7f22b83d8ae

```

```

这使得使用 UUIDv4 主键设置表格非常容易：

```

```

CREATE  TABLE my_uuidv4_things(  id  UUID  NOT  NULL  DEFAULT gen_random_uuid(),  name  TEXT  NOT  NULL,

PRIMARY KEY (id) );

INSERT  INTO my_uuidv4_things(name) VALUES ('foo');

SELECT * FROM my_uuidv4_things; --                  id                  | name  ----------------------------------------+------  -- 8501364b-b669-4c17-bd98-00bad8cd8f7d | foo

```

```

在 PostgreSQL 中似乎没有任何现有的内置或扩展函数支持生成 UUIDv7。要生成 UUIDv7，可以调整下面的函数以支持正确的格式化。

在 [Go](https://github.com/iCyberon/pg_ulid) 中有几种不同的 ULID 实现，或者在 [plsql](https://github.com/geckoboard/pgulid) 中也有。然而，这两种实现都返回 ULID 的 Crockford Base 32 文本表示形式（占用 26 字节），而不是 UUID 数据类型（占用 16 字节）。由于 PostgreSQL（以及一般的关系型数据库）的自然复用特性，拥有一个一致的共享单调计数器是困难的，甚至不可能的。Go PostgreSQL 实现使用的 [Go 库](https://github.com/oklog/ulid) 包含了单调计数器的实现，然而，它并未被 Go PostgreSQL 扩展使用。plsql 实现根本没有单调计数器。

我想要一个简单的 ULID PostgreSQL 实现，而不需要生成 UUID 数据类型值的单调计数器复杂性。所以我写了一个：

```

```

CREATE  EXTENSION  IF  NOT  EXISTS pgcrypto;

CREATE  OR REPLACE  FUNCTION generate_ulid() RETURNS  uuid   AS $$   SELECT (lpad(to_hex(floor(extract(epoch  FROM clock_timestamp()) * 1000)::bigint), 12, '0') || encode(gen_random_bytes(10), 'hex'))::uuid;

$$  LANGUAGE  SQL;

SELECT generate_ulid(); --> 017eb31e-1440-b69e-d82f-5f0937f823c8

```

```

分解函数：

-   第一个部分`lpad(to_hex(floor(extract(epoch FROM clock_timestamp()) * 1000)::bigint), 12, '0')`获取自Unix纪元以来的毫秒数，并将其转换为长度为12的十六进制字符串（接下来大约500年内会有一个前导0）。

+   -   第二个部分`encode(gen_random_bytes(10), 'hex')`生成10个随机字节并将它们转换为十六进制。

二者结合成16字节（或128位）的ULID标签，最后使用`::uuid`转换为UUID数据类型。

我们的新函数现在可以作为主键的默认值使用：

```

```

CREATE  TABLE my_ulid_things(  id  UUID  NOT  NULL  DEFAULT generate_ulid(),  name  TEXT  NOT  NULL,

PRIMARY KEY (id) );

INSERT  INTO my_ulid_things(name) VALUES ('foo');

SELECT * FROM my_ulid_things; --                  id                  | name  ----------------------------------------+------  -- 017eb31e-1440-b69e-d82f-5f0937f823c8 | foo

```

```

与`gen_random_uuid()`的本地实现相比，`generate_ulid()`在生成1000万个值时执行**慢73%**，但在生成和插入100万个值时仅执行**慢31%**。

意味着，虽然ULID生成速度较慢（至少在这个实现中是这样），但插入速度却快得多。我猜想这是因为ULID的顺序和高度聚集的重要字节，使得创建索引条目的速度快得多。与此同时，UUIDv4非常稀疏，因此创建索引条目的速度要慢得多。

这是我得到这些数字的方法：

```

```

generate-benchmark.sql

```
EXPLAIN  ANALYSE  SELECT gen_random_uuid() FROM generate_series(1, 10000000); -- 6766.681ms  
EXPLAIN  ANALYSE  SELECT generate_ulid() FROM generate_series(1, 10000000); -- 11750.966ms
```

```

```

generate-insert-benchmark.sql

```
CREATE  TABLE uuid_keys(id UUID); CREATE  TABLE ulid_keys(id UUID); 
EXPLAIN  ANALYSE  INSERT  INTO uuid_keys(id) SELECT gen_random_uuid() FROM generate_series(1, 1000000); -- 1372.470ms  
EXPLAIN  ANALYSE  INSERT  INTO ulid_keys(id) SELECT generate_ulid() FROM generate_series(1, 1000000); -- 1803.472ms
```

```

ULIDs are slower than their counterparts. For me, the benefits of a sortable globally unique identifier make the
tradeoff worth it.
```
