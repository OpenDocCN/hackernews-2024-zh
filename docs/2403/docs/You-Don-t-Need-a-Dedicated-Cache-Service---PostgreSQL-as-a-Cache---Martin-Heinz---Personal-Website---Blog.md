<!--yml
category: 未分类
date: 2024-05-27 14:39:33
-->

# You Don't Need a Dedicated Cache Service - PostgreSQL as a Cache | Martin Heinz | Personal Website & Blog

> 来源：[https://martinheinz.dev/blog/105](https://martinheinz.dev/blog/105)

PostgreSQL became a go-to SQL database for many developers over the past couple of years. While being an SQL database, Postgres also includes a lot of features that make it suitable for other uses, e.g. using it as NoSQL database (JSON and HStore datatypes) or vector database.

Another - more unusual and possibly unexpected use-case for Postgres - is using it as a cache!

In this article we will explore how we can leverage features of PostgreSQL to turn it into fully featured and very efficient caching service.

## Why?

You might be asking: *"Why would anyone do this, though?"*, To answer that question, let's take a look at what we might expect from a traditional caching service:

*   Expiration - Ability to set expiration times for cached data so that the cache doesn't store outdated information
*   Eviction - Remove less frequently used data when the cache is full
*   Invalidation - Overwrite data when it changes
*   Performance - Main reason to use cache is to avoid slow database queries. SQL databases generally also have slower writes compared to caches
*   No persistence - Caching services will generally have limited or no persistence
*   Key-value storage

Well, as we will see in the next section, we can get all of the above when using PostgreSQL, on top of that we also get the following as a bonus:

*   Familiar interface - Using SQL and common PostgreSQL client libraries, makes it easier to integrate into application
*   Cost - No need to set up and maintain another service. This decreases operational costs. You also don't need a Redis/Memcached/... expert to maintain it

## How

With the *"Why?"* out of the way, let's now talk about *"how"*. To implement cache in PostgreSQL effectively, we will use `UNLOGGED` table(s). How are these different from normal tables?

`UNLOGGED` tables don't generate *WAL ([Write Ahead Log](https://www.postgresql.org/docs/current/wal-intro.html))* information. That gives us huge improvements in write performance and saves us some disk space. There's obviously a trade-off - `UNLOGGED` tables aren't crash-safe - without WAL record, if database server crashes, an unlogged table is automatically truncated, but with cache we don't really expect proper persistence, so that's OK. Additionally, `UNLOGGED` tables are only available on primary, not on replicas, so no distributed cache, which might or might not be an issue, you decide.

That's the theory, now let's actually try it out. In the examples I will be using PostgreSQL running in container, if you want to follow along you can use:

```
 docker volume create pgdata
docker run -d \
  --name postgres \
  -v pgdata:/var/lib/postgresql/data \
  -e POSTGRES_PASSWORD=strongpassword \
  -p 5432:5432 \
  postgres:15.3-bullseye 
```

To then create a `cache` table we can run:

```
 CREATE UNLOGGED TABLE cache (
    id serial PRIMARY KEY,
    key text UNIQUE NOT NULL,
    value jsonb,
    inserted_at timestamp);

CREATE INDEX idx_cache_key ON cache (key); 
```

Only difference from normal table is the `UNLOGGED` keyword. As for the columns, here we use `JSONB` values, but you could use whatever suits your needs, e.g. `text`, `varchar` or `hstore`. We also include `inserted_at` column, which will be used for cache invalidation. Optionally, we also create an index for better read performance.

As was already mentioned, one of the features that we expect from caching service is ability to expire records. To do this in PostgreSQL, we can create a stored procedure that removes old rows periodically:

```
 CREATE OR REPLACE PROCEDURE expire_rows (retention_period INTERVAL) AS
$$
BEGIN
    DELETE FROM cache
    WHERE inserted_at < NOW() - retention_period;

    COMMIT;
END;
$$ LANGUAGE plpgsql;

CALL expire_rows('60 minutes'); -- This will remove rows older than 1 hour 
```

We will need to call this `expire_rows` procedure on a schedule. To do that we can use `pg_cron` extension. To use it, you will need to install it on OS level and then create the extension in database. You can do that using `Dockerfile` and scripts included in this [Gist](https://gist.github.com/MartinHeinz/2ea26ad81a5984de6befdacf855a9255).

After installing (`CREATE EXTENSION pg_cron;`), we can schedule the procedure call with:

```
 -- Create a schedule to run the procedure every hour
SELECT cron.schedule('0 * * * *', $$CALL expire_rows('1 hour');$$);

-- List all scheduled jobs
SELECT * FROM cron.job; 
```

If you don't want to install the extension for this, then you can alternatively write a trigger that runs everytime a row is inserted (I wouldn't recommend this though):

```
 CREATE OR REPLACE FUNCTION expire_rows_func (retention_hours integer) RETURNS void AS
$$
BEGIN
    DELETE FROM cache
    WHERE inserted_at < NOW() - (retention_hours || ' hours')::interval;
END;
$$ LANGUAGE plpgsql;

CREATE OR REPLACE FUNCTION expire_rows_func_trigger() RETURNS trigger AS
$$
BEGIN
    PERFORM expire_rows_func (1);
    RETURN NEW;
END;
$$ LANGUAGE plpgsql;

CREATE TRIGGER cache_cleanup_trigger
    AFTER INSERT ON cache
    FOR EACH ROW
    EXECUTE FUNCTION expire_rows_func_trigger(); 
```

Obliviously, the actual expiry/purge schedule depends on your data and use-case.

That was expiration, but what about eviction (removing old data to make space for new records)?

I'd consider eviction optional, considering that expiration should keep the size down, but we could implement that as well - we can add `last_read timestamp` column which would get updated on every read. We could then run stored procedure every once-in-a-while to clean-up rows that haven't been used recently, giving us an LRU cache. You decide whether updating rows on every read is worth it.

With that, we created a simple cache, which has fast writes, fast reads, key-value storage, better persistence than traditional caching services, cache expiration, eviction and invalidation, without having to deploy yet another (costly) service.

## Performance

So far, I mostly only mentioned how great it is to use PostgreSQL as a cache, but obviously, there are also downsides. One of them being performance, which will be (slightly) worse than with purpose-built optimized caching service. How much worse, depends on your data and usage patterns (read-heavy or write-heavy operations, data sizes, types of queries, etc.)

Benchmarking and comparing `UNLOGGED` tables against Memcached or Redis is out-of-scope of this article, but if you want to test performance (and you should) you could start with generating some data:

```
 INSERT INTO cache (key, value, inserted_at)
VALUES
    ('key1', '{"field1": "value1", "field2": "value2"}', NOW() - INTERVAL '1 hour'),
    ('key2', '{"field1": "value3", "field2": "value4"}', NOW() - INTERVAL '2 hours'),
    ('key3', '{"field1": "value5", "field2": "value6"}', NOW() - INTERVAL '3 hours'),
    ('key4', '{"field1": "value7", "field2": "value8"}', NOW() - INTERVAL '4 hours'),
    ('key5', '{"field1": "value9", "field2": "value10"}', NOW() - INTERVAL '5 hours');

-- Insert more data
INSERT INTO cache (key, value, inserted_at)
SELECT 'key' || s,
       ('{"field1": "value' || s || '"}')::jsonb,
       NOW() - (s || ' hours')::interval
FROM generate_series(1, 25) AS s; 
```

And then analyze queries to get a sense of what the performance might look like:

```
 EXPLAIN ANALYZE SELECT * FROM cache WHERE key = 'key1';

EXPLAIN ANALYZE INSERT INTO cache (key, value, inserted_at)
VALUES ('new_key', '{"field1": "new_value1", "field2": "new_value2"}', NOW()); 
```

I would also recommend reading [this great article](https://www.crunchydata.com/blog/postgresl-unlogged-tables) about `UNLOGGED` tables which shows more details about the feature including some performance comparisons.

## Closing Thoughts

In my opinion, most of the time, you don't need an additional service or special database. There's a reason why half the new fancy DBs are implemented on top of good old SQL databases. Just look at the list of [PostgreSQL-derived databases](https://wiki.postgresql.org/wiki/PostgreSQL_derived_databases). Not to mention things like [Timescale](https://www.timescale.com/) and many others, which are really just PostgreSQL with some extra features sprinkled on top.

While purpose-built, optimized solutions have their place, it's good to consider pros and cons of running extra service for every little thing, such as cache, scheduler, vector database, etc.

Maybe, just maybe, using one tool for multiple things and saving costs and overhead outweighs the few benefits the extra service provides.