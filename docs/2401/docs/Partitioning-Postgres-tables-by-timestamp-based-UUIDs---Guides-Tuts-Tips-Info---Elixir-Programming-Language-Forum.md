<!--yml
category: 未分类
date: 2024-05-27 14:43:39
-->

# Partitioning Postgres tables by timestamp based UUIDs - Guides/Tuts/Tips/Info - Elixir Programming Language Forum

> 来源：[https://elixirforum.com/t/partitioning-postgres-tables-by-timestamp-based-uuids/60916](https://elixirforum.com/t/partitioning-postgres-tables-by-timestamp-based-uuids/60916)

Recently we partitioned a table with 28 million rows by its ULID to improve query speeds and reduce query timeouts. To do this I ended up pulling together various bits of information across the internet and thought it might be worth compressing it into a guide for future reference and discussion on areas I might have missed!

# The problem

We had a very large table (28 million) rows that we were getting timeouts querying on anything other than the primary key. No matter what indexes we threw at the table we couldn’t get queries to complete in an acceptable time for use on the front end of the application.

# Constraints

The table in question was where we stored and processed payloads from our customers, so it needed to be very quick for individual lookups in order to update its state as it was processed into the application and fast for broader retrieval to display on the front end of our application.

# Problems with partitioning

Partitioning had always been an option and something we had attempted previously. Our original attempt to speed it up had been partitioning the table on the insertion time of the record but, if you’ve ever done partitioning before you will know, you cannot have a globally unique index.

> To create a unique or primary key constraint on a partitioned table, the partition keys must not include any expressions or function calls and the constraint’s columns must include all of the partition key columns. This limitation exists because the individual indexes making up the constraint can only directly enforce uniqueness within their own partitions; therefore, the partition structure itself must guarantee that there are not duplicates in different partitions.

Without the globally unique index for the id of the payload meant that queries for a specific payload had to check every partition. This made the individual processing of a given payload too slow to make this a workable scenario.

# Partitioning with ULIDs

The solution then was to have the id of the payload be a ULID based on its `inserted_at` and partition the tables on the timestamp component of the ULID it generated on the database! This means we could retain a global index for quick individual row lookups and have a way to ignore irrelevant partitions based on the id.

This took a few different steps…

## Preparing the database

Postgres doesn’t with with ULIDs directly but between Ecto and Postgres they are cast to a UUID in the database. We needed a method for Postgres to take a ULID and convert it to the corresponding UUID that we can work with in SQL.

We added the functions [here](https://github.com/scoville/pgsql-ulid) to a migration to make them available on the database.

While you’re on the database it’s worth ensuring that [`enable_partition_pruning`](https://www.postgresql.org/docs/current/ddl-partitioning.html#DDL-PARTITION-PRUNING) is enabled as this is how Postgres is able to ignore irrelevant partitions.

## Functions for manipulating ULIDs

To create the partitions we needed to be able to get the time portion of the ULID and get the bounds of any ids possible within the time.

This required slicing the time portion of the ULID and setting either “0” or “Z” depending on which bound we needed.

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

## Ensuring that timestamp and ids timestamp match

This is not strictly necessary for tables where the insertion time is close enough to the autogeneration of the id that you wont get too much drift but if there could be drift, or you want to partition on an timestamp that wont necessarily correlate with the id here is how. The following is all performed on the schema.

Start by turning off the autogeneration of the id.

```
@primary_key {:id, ULID, autogenerate: false} 
```

Check if the changeset has an id, and if it doesn’t add one based on the timestamp you want to use. In the following example I have used `occurred_at`.

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

Finally add it as part of your changeset.

```
 def changeset(changeset, attrs) do
    changeset
    # ...
    |> create_id_if_not_exsits()
    # other methods 
```

## Database migration

I cannot take full credit for this as it’s adapted from [Matthew Johnston’s work on partitioning](https://mattjohnston.co/posts/postgres-partitions-elixir-migration/), changed to work with id’s.

We need to start with a couple of helper functions to help us partition on month. You might want something different so feel free to modify to fit your requirements.

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

We use the functions above and our ULID functions earlier to have a query that will generate the query to make each partition.

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

Finally we can tie it all together with out table creation!

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

That’s it you’ve just created your partitioned table based on the timestamp based UUID!

## Improving your query speeds

Now your Ecto queries can utilize the id to only query partitions that contain the data they are looking for skipping other partitions that don’t hold any relevant data. We use a combination of the database and ULID functions we created earlier in the `where` call.

```
 query
    |> where(
        query,
        [p],
        fragment("? >= ulid_to_uuid(?)", p.id, ^ULIDUtils.datetime_to_ulid(from, :start))
    ) 
```

# Conclusion

So we went from queries taking multiple minutes to sub-second, zero timeouts and we were also able to retain very quick lookups on individual rows!

I hope that this helps others after me, and if you’ve got any suggestions on where you might do something slightly differently let me know! (I’ve only been working with Elixir for a little over a year now and I suspect there are some code smells etc in the above)