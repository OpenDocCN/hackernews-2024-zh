<!--yml

类别：未分类

日期：2024-05-27 14:43:39

-->

# 使用基于时间戳的 UUID 对 Postgres 表进行分区 - 指南/教程/提示/信息 - Elixir 编程语言论坛

> 来源：[https://elixirforum.com/t/partitioning-postgres-tables-by-timestamp-based-uuids/60916](https://elixirforum.com/t/partitioning-postgres-tables-by-timestamp-based-uuids/60916)

最近，我们通过其 ULID 对表进行了分区，以提高查询速度并减少查询超时。为了做到这一点，我最终在互联网上汇总了各种信息，认为将其压缩成一个指南以供将来参考和讨论我可能忽略的领域可能是值得的！

# 问题

我们有一个非常庞大的表（2800 万行），除了主键之外，我们在任何其他内容上都会遇到查询超时。无论我们向表中添加什么索引，我们都无法在可接受的时间内完成查询，以便在应用程序的前端使用。

# 约束

所涉及的表是我们从客户那里存储和处理有效负载的表，因此对于个体查找来说，它需要非常快以便在处理到应用程序中时更新其状态，并且对于更广泛的检索来说，它需要快速以在我们应用程序的前端显示。

# 分区的问题

分区一直是一个选择，并且是我们之前尝试过的东西。我们最初试图加快速度是对记录的插入时间进行分区，但是，如果您以前进行过分区，您将知道，您不能拥有全局唯一索引。

> 要在分区表上创建唯一或主键约束，分区键不能包含任何表达式或函数调用，约束的列必须包含所有分区键列。存在此限制是因为构成约束的各个索引只能直接在自己的分区内强制执行唯一性；因此，分区结构本身必须保证不同分区中没有重复项。

没有针对有效负载 ID 的全局唯一索引意味着对特定有效负载的查询必须检查每个分区。这使得对给定有效负载的个体处理过程过慢，无法使其成为可行的情况。

# 使用 ULID 进行分区

那时的解决方案是使有效负载的 ID 成为基于其`inserted_at`的 ULID，并根据数据库生成的 ULID 的时间戳组件对表进行分区！这意味着我们可以保留快速个体行查找的全局索引，并有一种忽略基于 ID 的无关分区的方法。

这经历了几个不同的步骤…

## 准备数据库

Postgres 不直接使用 ULID，但在 Ecto 和 Postgres 之间，它们在数据库中被转换为 UUID。我们需要一种方法，让 Postgres 接受 ULID 并将其转换为我们可以在 SQL 中使用的相应 UUID。

我们将函数[添加到这里](https://github.com/scoville/pgsql-ulid)以便将它们可用于数据库中。

当您在数据库上时，值得确保[`enable_partition_pruning`](https://www.postgresql.org/docs/current/ddl-partitioning.html#DDL-PARTITION-PRUNING)已启用，因为这是 PostgreSQL 能够忽略不相关分区的方式。

## 用于操作 ULID 的函数

要创建分区，我们需要能够获取 ULID 的时间部分，并获取在时间范围内可能存在的任何 id 的边界。

这需要切片 ULID 的时间部分，并根据我们需要的边界设置“0”或“Z”。

```
 @doc """
  Slices the time portion of a ULID
  """
  def ulid_time(ulid), do: String.slice(ulid, 0..9)

  @doc """
  Returns the time based start of a ULID
  """
  def ulid_start_time(ulid), do: ulid_time(ulid) |> String.pad_trailing(26, "0")

  @doc """
  Returns the time based end of a ULID
  """
  def ulid_end_time(ulid), do: ulid_time(ulid) |> String.pad_trailing(26, "Z")

  @doc """
  Returns the time based ULID with bounds from a Date or DateTime
  """
  def datetime_to_ulid(datetime, bound \\ nil)

  def datetime_to_ulid(date = %Date{}, bound),
    do: DateTime.new!(date, Time.new!(0, 0, 0)) |> datetime_to_ulid(bound)

  def datetime_to_ulid(datetime = %DateTime{}, bound) do
    case bound do
      nil -> DateTime.to_unix(datetime, :millisecond) |> Ecto.ULID.generate()
      :start -> datetime_to_ulid(datetime) |> ulid_start_time()
      :end -> datetime_to_ulid(datetime) |> ulid_end_time()
    end
  end

  def datetime_to_ulid(naive_datetime = %NaiveDateTime{}, bound),
    do: DateTime.from_naive!(naive_datetime, "Etc/UTC") |> datetime_to_ulid(bound) 
```

## 确保时间戳和 id 的时间戳匹配

对于插入时间与 id 的自动生成时间接近的表来说，这并不是绝对必要的，但如果可能存在偏差，或者您想要根据不一定与 id 相关联的时间戳进行分区，那么这是如何实现的。 以下操作全部在架构中执行。

首先关闭自动生成 id 的功能。

```
@primary_key {:id, ULID, autogenerate: false} 
```

检查更改集是否具有 id，如果没有，则根据您要使用的时间戳添加一个。 在以下示例中，我使用了`occurred_at`。

```
 defp create_id_if_not_exsits(changeset) do
    case get_change(changeset, :id) do
      nil ->
        occurred_at = get_change(changeset, :occurred_at)
        put_change(changeset, :id, ULIDUtils.datetime_to_ulid(occurred_at))

      _ ->
        changeset
    end
  end 
```

最后，将其添加为更改集的一部分。

```
 def changeset(changeset, attrs) do
    changeset
    # ...
    |> create_id_if_not_exsits()
    # other methods 
```

## 数据库迁移

我不能完全归功于这一点，因为它是从[马修·约翰斯顿（Matthew Johnston）关于分区的工作](https://mattjohnston.co/posts/postgres-partitions-elixir-migration/)改编而来的，改成了与 id 相匹配。

我们需要从几个辅助函数开始，以帮助我们根据月份进行分区。 您可能希望使用不同的东西，所以请随时修改以满足您的要求。

```
 defp beginning_of_month(date) do
    if date.day == 1 do
      date
    else
      Date.add(date, -(date.day - 1))
    end
  end

  defp calculate_next_month(date, 0), do: date

  defp calculate_next_month(date, months) do
    next = Date.add(date, Date.days_in_month(date))
    calculate_next_month(next, months - 1)
  end

  def months_from(date, num_months) do
    start_date = beginning_of_month(date)

    for months <- 0..num_months do
      calculate_next_month(start_date, months)
    end
  end 
```

我们使用上述函数以及我们之前的 ULID 函数来生成查询，以创建每个分区的查询。

```
def create_partition_query(table, date) do
    start_date = date

    stop_date =
      date
      |> Date.add(Date.days_in_month(date))
      |> Date.add(-1)

    month =
      start_date.month
      |> Integer.to_string()
      |> String.pad_leading(2, "0")

    stop_datetime = DateTime.new!(stop_date, Time.new!(23, 59, 59, 999_999))
    end_ulid = datetime_to_ulid(stop_datetime, :end)

    start_ulid = datetime_to_ulid(start_date, :start)

    """
    CREATE TABLE #{table}_p#{start_date.year}#{month}
    PARTITION OF #{table} FOR VALUES
    FROM (ulid_to_uuid('#{start_ulid}'))
    TO (ulid_to_uuid('#{end_ulid}'))
    """
  end 
```

最后，我们可以通过我们的表创建将所有内容联系起来！

```
 create table(:payloads, primary_key: false, options: "PARTITION BY RANGE(id)") do
      add(:id, :binary_id, null: false, primary_key: true)

      # ... other fields and references
    end

    # Create the default table that will ingest entries that don't match the ranges.
    # This is not strictly necessary but means that id's that aren't in the ranges we
    # declared don't create an error
    execute("CREATE TABLE payloads_default PARTITION OF payloads DEFAULT")

    # Create 48 months worth of partitions starting from 2022-01-01
    for month <- ULIDUtils.months_from(~D[2022-01-01], 48) do
      query = ULIDUtils.create_partition_query("payloads", month)
      execute(query)
    end 
```

就这样，您刚刚基于基于时间戳的 UUID 创建了分区表！

## 提升您的查询速度

现在，您的 Ecto 查询可以利用 id 仅查询包含所需数据的分区，跳过不包含任何相关数据的其他分区。 我们在`where`调用中使用了我们早些时候创建的数据库和 ULID 函数的组合。

```
 query
    |> where(
        query,
        [p],
        fragment("? >= ulid_to_uuid(?)", p.id, ^ULIDUtils.datetime_to_ulid(from, :start))
    ) 
```

# 结论

所以，我们从需要花费几分钟的查询变为几秒钟内完成，零超时，并且我们还能够保留对单个行进行非常快速的查找！

我希望这对我之后的其他人有所帮助，如果你有任何建议，可以告诉我在哪里可能会有一些略有不同的地方！（我只是在 Elixir 上工作了一年多一点，我怀疑上面的代码中可能存在一些代码气味等）
