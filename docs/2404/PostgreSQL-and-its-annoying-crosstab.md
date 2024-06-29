<!--yml

类别：未分类

日期：2024-05-27 12:58:17

-->

# PostgreSQL 及其令人讨厌的交叉表

> 来源：[https://blog.aurelianix.com/2024/04/04/postgresql-and-its-annoying-crosstab/](https://blog.aurelianix.com/2024/04/04/postgresql-and-its-annoying-crosstab/)

今天，我不得不从我通常的任务中转移到帮助同事解决查询的任务。任务看似简单：在单个查询中收集关于表的所有列的元数据。这是 PostgreSQL 中的一个函数，将返回具有以下列的表：

+   `table_name`

+   `column_name`

+   `data_type`

+   `total_tows`

+   `not_null_count`

+   `unique_count`

+   `max_value`（用于整数）

+   `min_value`（用于整数）

+   `avg_value`（用于整数）

+   `max_length`（用于字符串）

+   `min_length`（用于字符串）

+   `avg_length`（用于字符串）

+   `space_count_max`（用于字符串）

+   `space_count_min`（用于字符串）

+   `space_count_avg`（用于字符串）

+   `max_date`（用于日期）

+   `min_date`（用于日期）

如果每列都单独查询，那么速度会很慢，因为在没有索引的情况下需要进行全表扫描。

让我们创建一个测试表：

```
CREATE TABLE my_table
(
    string_col text,
    int_col    int
);
-- create 1000 rows with strings '1 2 ... 10' or null for string_col
-- create 1000 rows with random values between 1 and 100 or null for int_col
INSERT INTO my_table
SELECT CASE
           WHEN RANDOM() < 0.9 THEN (SELECT STRING_AGG(words::text, ' ') FROM GENERATE_SERIES(1, 10) words)
           ELSE NULL END,
       CASE WHEN RANDOM() < 0.9 THEN FLOOR(RANDOM() * 100) ELSE NULL END
FROM GENERATE_SERIES(1, 1000); 
```

## 获取统计信息，以宽表格式呈现

解决方案的第一部分是构建一个查询，返回元数据的宽表，类似于（为了简洁起见缩短了）：

```
SELECT
        COUNT(*) total_rows,
        -- stats for int_col
        COUNT(int_col) int_col_null_count,
        COUNT(DISTINCT int_col) int_col_unique_count,
        MAX(int_col) int_col_max_value,
        MIN(int_col) int_col_min_value,
        AVG(int_col) int_col_avg_value,
        -- stats for string_col
        COUNT(string_col) string_col_null_count,
        COUNT(DISTINCT string_col) string_col_unique_count,
        MAX(LENGTH(string_col)) string_col_max_length,
        MIN(LENGTH(string_col)) string_col_min_length,
        AVG(LENGTH(string_col)) string_col_avg_length,
        MAX(LENGTH(string_col) - LENGTH(REPLACE(string_col, ' ', ''))) string_col_space_count_max,
        MIN(LENGTH(string_col) - LENGTH(REPLACE(string_col, ' ', ''))) string_col_space_count_min,
        AVG(LENGTH(string_col) - LENGTH(REPLACE(string_col, ' ', ''))) string_col_space_count_avg
FROM my_table; 
```

## 获取统计信息，以高表格式呈现

思路是一次性收集表的统计信息，然后从中建立某种透视表。由于我们使用动态 SQL，这并不太难。我们可以使用侧向连接将数据变成高表：

```
WITH stats AS (
    SELECT
        COUNT(*) total_rows,
        -- stats for int_col
        COUNT(int_col) int_col_not_null_count,
        COUNT(DISTINCT int_col) int_col_unique_count,
        MAX(int_col) int_col_max_value,
        MIN(int_col) int_col_min_value,
        AVG(int_col) int_col_avg_value,
        -- stats for string_col
        COUNT(string_col) string_col_not_null_count,
        COUNT(DISTINCT string_col) string_col_unique_count,
        MAX(LENGTH(string_col)) string_col_max_length,
        MIN(LENGTH(string_col)) string_col_min_length,
        AVG(LENGTH(string_col)) string_col_avg_length,
        MAX(LENGTH(string_col) - LENGTH(REPLACE(string_col, ' ', ''))) string_col_space_count_max,
        MIN(LENGTH(string_col) - LENGTH(REPLACE(string_col, ' ', ''))) string_col_space_count_min,
        AVG(LENGTH(string_col) - LENGTH(REPLACE(string_col, ' ', ''))) string_col_space_count_avg
    FROM my_table
)
SELECT l.* FROM stats
    CROSS JOIN LATERAL(
        VALUES
        -- int_col stats
        ('int_col', 'total_rows', total_rows::text),
        ('int_col', 'not_null_count', int_col_not_null_count::text),
        ('int_col', 'unique_count', int_col_unique_count::text),
        ('int_col', 'max_value', int_col_max_value::text),
        ('int_col', 'min_value', int_col_min_value::text),
        ('int_col', 'avg_value', int_col_avg_value::text),
        -- string_col stats
        ('string_col', 'total_rows', total_rows::text),
        ('string_col', 'not_null_count', string_col_not_null_count::text),
        ('string_col', 'unique_count', string_col_unique_count::text),
        ('string_col', 'max_length', string_col_max_length::text),
        ('string_col', 'min_length', string_col_min_length::text),
        ('string_col', 'avg_length', string_col_avg_length::text),
        ('string_col', 'space_count_max', string_col_space_count_max::text),
        ('string_col', 'space_count_min', string_col_space_count_min::text),
        ('string_col', 'space_count_avg', string_col_space_count_avg::text)
    ) AS l(column_name, meta_key, value); 
```

这将产生如下所示的结果：

| column_name | meta_key | value |
| --- | --- | --- |
| int_col | total_rows | 1000 |
| int_col | not_null_count | 904 |
| int_col | unique_count | 100 |
| int_col | max_value | 99 |
| int_col | min_value | 0 |
| int_col | avg_value | 50.4115044247787611 |
| string_col | total_rows | 1000 |
| string_col | not_null_count | 915 |
| string_col | unique_count | 1 |
| string_col | max_length | 20 |
| string_col | min_length | 20 |
| string_col | avg_length | 20.0000000000000000 |
| string_col | space_count_max | 9 |
| string_col | space_count_min | 9 |
| string_col | space_count_avg | 9.0000000000000000 |

## 将数据透视为表

现在，我们需要将这些数据进行透视。PostgreSQL 的交叉表正是我们需要的，但是让它正常工作有点麻烦。我们首先需要安装 `tablefunc` 扩展：

```
CREATE EXTENSION IF NOT EXISTS tablefunc; 
```

`crosstab` 接受一个字符串作为参数，其中包含将要返回数据以进行透视的查询。我们可以使用上面生成的巨大 SQL，但是要大 10 倍，因为哪个表只有 2 列？查询将如下所示：

```
SELECT * FROM crosstab('SELECT column_name, meta_key, value FROM ([GIANT SUBQUERY HERE]) t ORDER BY 1, 2') AS ct (
    column_name text,
    total_rows text,
    not_null_count text,
    unique_count text,
    max_value text,
    min_value text,
    avg_value text,
    max_length text,
    min_length text,
    avg_length text,
    space_count_max text,
    space_count_min text,
    space_count_avg text
); 
```

现在让我们来看看结果：

| column_name | total_rows | not_null_count | unique_count | max_value | min_value | avg_value | max_length | min_length | avg_length | space_count_max | space_count_min | space_count_avg |
| --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
| int_col | 1000 | 904 | 100 | 99 | 0 | 50.4115044247787611 | null | null | null | null | null | null |
| string_col | 1000 | 915 | 1 | 20 | 20 | 20.0000000000000000 | 9 | 9 | 9.0000000000000000 | null | null | null |

呃……这不是我们想要的。`string_col`的统计数据向左移动了。它没有按名称旋转，而是填满了列。使用`crosstab`时，第一列将被旋转。如果我们想要将值旋转为具有更多列的表格（例如，在这里，`int_col`将旋转为6列，而`string_col`将旋转为9列），我们需要向`crosstab`提供第二个参数。建议按顺序排列行，以便每个参数都按顺序排列。

```
SELECT * FROM crosstab(
    'SELECT column_name, meta_key, value, row_number() OVER (PARTITION BY column_name ORDER BY meta_key) FROM ([GIANT SUBQUERY HERE]) t ORDER BY 1, 2',
    -- list of all result columns, sorted by name
    'VALUES
        (''avg_length''),
        (''avg_value''),
        (''max_length''),
        (''max_value''),
        (''min_length''),
        (''min_value''),
        (''not_null_count''),
        (''space_count_avg''),
        (''space_count_max''),
        (''space_count_min''),
        (''total_rows''),
        (''unique_count'')
    ') AS ct (
    -- MUST BE IN SAME ORDER AS THE LIST ABOVE
    column_name text,
    avg_length text,
    avg_value text,
    max_length text,
    max_value text,
    min_length text,
    min_value text,
    not_null_count text,
    space_count_avg text,
    space_count_max text,
    space_count_min text,
    total_rows text,
    unique_count text
); 
```

让我们来看看结果：

| column_name | avg_length | avg_value | max_length | max_value | min_length | min_value | not_null_count | space_count_avg | space_count_max | space_count_min | total_rows | unique_count |
| --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
| int_col | null | 50.4115044247787611 | null | 99 | null | 0 | 904 | null | null | null | 1000 | 100 |
| string_col | 20.0000000000000000 | null | 20 | null | 20 | null | 915 | 9.0000000000000000 | 9 | 9 | 1000 | 1 |

啊，看起来更像了。

这种方法有一些缺点：

1.  所有数据类型必须转换为文本，因为我们的高格式要求静态类型。

1.  Crosstab真的很烦人，容易出错。

1.  你不能包含比要旋转成的列更多的列。如果结果中有一个在旋转列表中没有值的列，`crosstab`函数将会抱怨。

但是，你知道吗？它有效。现在对我来说已经足够好了。如果您有任何建议来改进这一点，请务必告诉我。

PS：我学到了在postgres中还可以使用美元引用字符串来转义多行字符串。

```
SELECT $$This is
a long string with 'quotes'$$; 
```

啊，感觉好多了。
