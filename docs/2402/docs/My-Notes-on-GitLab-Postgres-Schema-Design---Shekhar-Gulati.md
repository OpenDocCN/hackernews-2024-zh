<!--yml
category: 未分类
date: 2024-05-27 14:56:04
-->

# My Notes on GitLab Postgres Schema Design – Shekhar Gulati

> 来源：[https://shekhargulati.com/2022/07/08/my-notes-on-gitlabs-postgres-schema-design/](https://shekhargulati.com/2022/07/08/my-notes-on-gitlabs-postgres-schema-design/)

I spent some time going over the Postgres schema of Gitlab. GitLab is an alternative to Github. You can self host GitLab since it is an open source DevOps platform.

My motivation to understand the schema of a big project like Gitlab was to compare it against schemas I am designing and learn some best practices from their schema definition. I can surely say I learnt a lot.

> I am aware that best practices are sometimes context dependent so you should not apply them blindly.

The Gitlab schema file `structure.sql` [1] is more than 34000 lines of code. Gitlab is a monolithic Ruby on Rails application. The popular way to manage schema migration is using the `schema.rb` file. The reason the Gitlab team decided to adopt `structure.sql` instead is mentioned in on of their issues [2] in their issue tracker.

> Now what keeps us from using those features is the use of `schema.rb`. This can only contain standard migrations (using the Rails DSL), which aim to keep the schema file database system neutral and abstract away from specific SQL. This in turn means we are not able to use extended PostgreSQL features that are reflected in schema. Some examples include triggers, postgres partitioning, materialized views and many other great features.
> 
> In order to leverage those features, we should consider using a plain SQL schema file (`structure.sql`) instead of a ruby/rails standard schema `schema.rb`.
> 
> The change would entail switching `config.active_record.schema_format = :sql` and regenerate the schema in SQL. Possibly, some build steps would have to be adjusted, too.

Now, let’s go over the things I learnt from Gitlab Postgres schema.

Below are some of the tweets from people on this article. If you find this article useful please share and tag me [@shekhargulati](https://twitter.com/shekhargulati)

 ## 1\. Using the right primary key type for a table

In my work I have made the mistake of standardizing on primary key types. This means standardizing on either `bigint` or `uuid` so all tables will have the same type irrespective of their structure, access patterns, and growth rate.

When your database is small this does not have any visible impact but as you grow primary keys have a visible impact on storage space, write speed, and read speed. So, we should give a proper thought process on choosing the right primary key type for a table.

As I discussed in an earlier post[3] when you use Postgres native UUID v4 type instead of bigserial table size grows by 25% and insert rate drops to 25% of bigserial. This is a big difference. I also compared against ULID but it also performed poorly. One reason could be the ULID implementation.

Given this context I was interested to learn how Gitlab chooses primary key types.

Out of the 573 tables, 380 tables have bigserial primary key type, 170 have serial4 primary key type, and remaining 23 had composite primary keys.They had no table that used uuid v4 primary key or any other esoteric key type like ULID.

| Name | Description | Range | Text |
| --- | --- | --- | --- |
| `serial` | 4 bytes | 1 to 2147483647 | ~2.1 billion |
| `bigserial` | 8 bytes | 1 to 9223372036854775807 | ~9.2 quintillion |

> 1 quintillion is equal to 1000000000 billions

The decision to choose serial or bigserial is dependent on the number of records in that table.

Tables like `application_settings`, `badges`, `chat_teams`, `notification_settings`, `project_settings` use serial type. For some tables like `issues`, `web_hooks`, `merge_requests`, `projects` I was surprised to see that they had used the `serial` type.

The serial type might work for self-hosted community or enterprise versions but for Gitlab.com SaaS service this can cause issues. For example, Github had 128 million public repositories in 2020\. Even with 20 issues per repository it will cross the serial range. Also changing the type of the table is expensive. The table has to be rewritten, and you will have to wait. This will also be a problem if you have to shard the table.

I performed a quick experiment that showed that for my table with two columns and 10million records it takes 11 seconds to change the data type from integer to bigint.

```
create table exp_bs(id serial primary key, n bigint not null);

```

Insert 10million records

```
insert into exp_bs(n) select g.n from generate_series(1,10000000) as g(n);

```

Change column type from integer to bigint.

```
alter table exp_bs alter column id TYPE bigint;

```

```
ALTER TABLE
Time: 10845.062 ms (00:10.845)

```

You will also have to alter the sequence to change its type as well. This operation is quick.

```
alter sequence exp_bs_id_seq as bigint;

```

This finished in 4ms

```
ALTER SEQUENCE
Time: 4.505 ms

```

All the bigserial sequences start from 1 and go till the max value of bigint.

```
CREATE SEQUENCE audit_events_id_seq
    START WITH 1
    INCREMENT BY 1
    NO MINVALUE
    NO MAXVALUE
    CACHE 1;

```

## 2\. Use of internal and external ids

It is generally a good practice to not expose your primary keys to the external world. This is especially important when you use sequential auto-incrementing identifiers with type integer or bigint since they are guessable.

So, I was curious to know what happens when you create a Gitlab issue. Do we expose the primary key id to the external user or do we use some other id? If you expose the `issues` table primary key id then when you create an issue in your project it will not start with 1 and you can easily guess how many issues exist in the GitLab. This is both unsafe and poor user experience.

To avoid exposing your primary keys to the end user the common solution is use two ids. The first is your primary key id which remains internal to the system and never exposed to any public context. The second id is what we share with the external world. In my past experience I have used UUID v4 as the external id. As we discussed in the previous point there is a storage cost involved with using UUID.

GitLab also uses internal and external ids in tables where ids have to be shared with the external world. Tables like `issues`, `ci_pipelines`, `deployments`, `epics`, and a few others have two ids – `id` and `iid`. Below is the part of the issue schema. As shown below `iid` has integer data type.

```
CREATE TABLE issues (
    id integer NOT NULL,
    title character varying,
      project_id integer,
    iid integer,
    // rest of the columns removed
)

```

As you can see there are `id` and `iid` columns. The value of the `iid` column is shared with the end user. An issue is uniquely identified using `project_id` and `iid`. This is because there could be multiple issues with the same `iid` . To make it more clear, if you create two projects and create one issue in each of the repositories then they both need to have a visible id of 1 as shown in the example below. Both the `sg` and `sg2` projects start with issue id 1\. This is achieved using `iid`.

```
https://gitlab.com/shekhargulati123/sg/-/issues/1
https://gitlab.com/shekhargulati123/sg2/-/issues/1

```

They have a unique index on `project_id` and `iid` to quickly and efficiently fetch an issue.

```
CREATE UNIQUE INDEX index_issues_on_project_id_and_iid ON public.issues USING btree (project_id, iid);

```

## 3\. Using `text` character type with check constraints

Postgres has three character types as described in their documentation[5].

| Name | Description |
| --- | --- |
| `character varying(n)`, `varchar(n)` | variable-length with limit |
| `character(n)`, `char(n)` | fixed-length, blank padded |
| `text` | variable unlimited length |

I have mostly used `character varying(n)` or `varchar(n)` to store String values. Gitlab schema uses both `character varying(n)` and `text` but more often they use `text` type. One such example table is shown below.

```
CREATE TABLE audit_events (
    id bigint NOT NULL,
    author_id integer NOT NULL,
    entity_id integer NOT NULL,
    entity_type character varying NOT NULL,
    details text,
    ip_address inet,
    author_name text,
    entity_path text,
    target_details text,
    created_at timestamp without time zone NOT NULL,
    target_type text,
    target_id bigint,
    CONSTRAINT check_492aaa021d CHECK ((char_length(entity_path) <= 5500)),
    CONSTRAINT check_83ff8406e2 CHECK ((char_length(author_name) <= 255)),
    CONSTRAINT check_97a8c868e7 CHECK ((char_length(target_type) <= 255)),
    CONSTRAINT check_d493ec90b5 CHECK ((char_length(target_details) <= 5500))
)
PARTITION BY RANGE (created_at);

```

You can see that apart from `entity_type` all other columns have `text` type. They have used `CHECK` to define length constraints.

As mentioned in multiple posts[6,7] on the web there is not much performance difference between the two types. They both use `varlena` type under the hood.

The problem with `varchar(n)` is that if n becomes more restrictive then it will require an exclusive lock. This can cause performance issues depending on the size of the table.

The `text` column with `CHECK` constraint on the other hand does not have this issue. But it does cost a little during writes.

Let’s do a quick experiment to prove it. We will start by creating a simple table

```
create table cv_exp (id bigint primary key, s varchar(200) default gen_random_uuid() not null);
create index sidx on cv_exp (s);

```

Insert 10million records

```
insert into cv_exp(id) select g.n from generate_series(1,10000000) as g(n);

```

If we increase the length of s from 200 to 300 then it is instantaneous

```
alter table cv_exp alter column s type varchar(300);

```

```
ALTER TABLE
Time: 37.460 ms

```

But if we reduce the length of `s` from 300 to 100 then it does take considerable time.

```
alter table cv_exp alter column s type varchar(100);

```

```
ALTER TABLE
Time: 35886.638 ms (00:35.887)

```

As you can see it took 36 seconds.

Let’s do the same with the text column.

```
create table text_exp (id bigint primary key, 
s text default gen_random_uuid() not null,
CONSTRAINT check_15e644d856 CHECK ((char_length(s) <= 200)));

```

Insert 10 million records

```
insert into text_exp(id) select g.n from generate_series(1,10000000) as g(n);

```

There is no alter constraint in Postgres. You have to drop the constraint and then add a new constraint.

```
 alter table text_exp drop constraint check_15e644d856;

```

Now, add again

```
alter table text_exp add constraint check_15e644d856 CHECK ((char_length(s) <= 100));

```

```
ALTER TABLE
Time: 1870.250 ms (00:01.870)

```

So, as you can see, the `text` type with `CHECK` constraint allows you to evolve the schema easily compared to `character varying` or `varchar(n)` when you have length checks.

I also noticed that they used `character varying` where length checks are not required like as shown below.

```
CREATE TABLE project_custom_attributes (
    id integer NOT NULL,
    created_at timestamp with time zone NOT NULL,
    updated_at timestamp with time zone NOT NULL,
    project_id integer NOT NULL,
    key character varying NOT NULL,
    value character varying NOT NULL
);

```

## 4\. Naming conventions

The naming follows the following convention.

*   All tables use plural forms. For example `issues`, `projects`, `audit_events`, `abuse_reports`, `approvers`, etc.
*   Tables use module name prefix to provide a namespace. For example, all tables belonging to merge request functionality start with `merge_request` prefix as shown in the listing below.
    *   merge_request_assignees
    *   merge_request_blocks
    *   merge_request_cleanup_schedules
    *   merge_request_context_commit_diff_files
    *   merge_request_context_commits
    *   etc..
*   Names of tables and columns follow snake_case convention. The underscore is used to combine two or more words. For example, `title`, `created_at`, `is_active`.
*   Columns expressing boolean follow either of the three naming convention depending on their purpose
    *   Feature toggles. For example `create_issue`, `send_email`, `packages_enabled`, `merge_requests_rebase_enabled`, etc
    *   Entity State: Examples, `deployed`, `onboarding_complete`, `archived`, `hidden`, etc.
    *   Qualifiers – They start with `is_xxx` or `has_xxx`. For example, `is_active`, `is_sample`, `has_confluence`, etc. I think these can be expressed using the above two.
*   Indexes follow the convention `index_#{table_name}_on_#{column_1}_and_#{column_2}_#{condition}`. For example, `index_services_on_type_and_id_and_template_when_active`, `index_projects_on_id_service_desk_enabled`.

## 5\. Timestamp with timezone and without timezone

GitLab uses both `timestamp with timezone` and `timestamp without timezone`.

My understanding is that the data type `timestamp without timezone` is used when the system performs an action and data type `timestamp with time zone` is used for user actions. For example, in the SQL shown below `created_at` and `updated_at` does use `timestamp without time zone` whereas `closed_at` uses `timestamp with time zone`.

```
CREATE TABLE issues (
    id integer NOT NULL,
    title character varying,

    created_at timestamp without time zone,
    updated_at timestamp without time zone,

    closed_at timestamp with time zone,
    closed_by_id integer,
);

```

Another example is `merge_request_metrics` where `latest_closed_at`, `first_comment_at`, `first_commit_at` , and `last_commit_at` uses `timestamp with time zone` whereas `latest_build_started_at` , `latest_build_finished_at`, and `merge_at` they use `timestamp without timezone`. You might wonder why `merge_at` does not use a timezone. I think it is because the system can merge the request based on certain conditions or checks.

```
CREATE TABLE merge_request_metrics (
    id integer NOT NULL,
    latest_build_started_at timestamp without time zone,
    latest_build_finished_at timestamp without time zone,
    merged_at timestamp without time zone,
    created_at timestamp without time zone NOT NULL,
    updated_at timestamp without time zone NOT NULL,

    latest_closed_at timestamp with time zone,
    first_comment_at timestamp with time zone,
    first_commit_at timestamp with time zone,
    last_commit_at timestamp with time zone,
    first_approved_at timestamp with time zone,
    first_reassigned_at timestamp with time zone
);

```

## 6\. Foreign key constraints

A foreign key constraint is a logical association of rows between two tables. You typically use foreign keys to join tables in queries.

> A **FOREIGN KEY** constraint is a database construct, an implementation that forces the foreign key relationship’s integrity (referential integrity). Namely, it ensures that a child table can only reference a parent table when the appropriate row *exists* in the parent table. A constraint also prevents the existence of “orphaned rows” in different methods. – [Link](https://docs.planetscale.com/learn/operating-without-foreign-key-constraints)

I have consulted in multiple projects in the last couple of years where team/architects decided not to use foreign key constraints. They mainly cite performance as the reason to not use foreign key constraints.

One reason performance can degrade when you create foreign key is when you create it with `ON DELETE CASCADE` action. The way `ON DELETE CASCADE` action works is that if you delete a row in the parent table then any referencing row in the child table is also deleted within the same transaction. You might expect only one row to be deleted but you might end up deleting hundred or thousand or more child table rows as well. But, this will be an issue only when one parent row is linked with a large number of child table rows.

There are two other reasons teams don’t use foreign key constraints. These are:

*   They don’t work well with online DDL schema migration operations especially in MySQL
*   It is difficult to maintain foreign key constraints once you shard your data into multiple database servers

> MySQL compatible serverless database like PlanetScale(based on open source Vitess database) does not support foriegn keys.

So, I was curious to learn if GitLab uses Foreign key constraints or not.

GitLab uses foriegn key constraints in most tables except in a few tables like `audit_events` , `abuse_reports`, `web_hooks_logs`, `spam_logs`. I think there are two main reasons why they don’t use foreign key constraints in `audit_events` , `abuse_reports`, `web_hooks_logs`, `spam_logs`. These are:

*   These tables are immutable in nature. You don’t want to change them once entries are written to them
*   These tables can grow to millions(or more) of rows so even a small performance hit could have a big impact

The rest of the tables where GitLab uses foreign keys use both `ON DELETE CASCADE` , `ON DELETE RESTRICT` , and `ON DELETE SET NULL` actions. An example of each of them is shown below.

```
ALTER TABLE ONLY todos
    ADD CONSTRAINT fk_rails_a27c483435 FOREIGN KEY (group_id) REFERENCES namespaces(id) ON DELETE CASCADE;

ALTER TABLE ONLY projects
    ADD CONSTRAINT fk_projects_namespace_id FOREIGN KEY (namespace_id) REFERENCES namespaces(id) ON DELETE RESTRICT;

ALTER TABLE ONLY authentication_events
    ADD CONSTRAINT fk_rails_b204656a54 FOREIGN KEY (user_id) REFERENCES users(id) ON DELETE SET NULL;

```

*   `ON DELETE SET NULL` will set the referencing column on the child table to null for matching rows. It leads to orphaned rows but you can easily identify them because of NULL. In this action also, a single row deletion can lead to multiple rows getting updated in the child table. This may cause large transactions, excessive locking, and replication lag.
*   `ON DELETE RESTRICT` prevents deletion of referenced child rows. This does not cause orphaned childs as you can’t delete a parent row if there are child rows that references it. You get exceptions like as shown below.

```
  ERROR:  update or delete on table "a" violates foreign key constraint "fk_a_id" on table "b"
  DETAIL:  Key (id)=(1) is still referenced from table "b".

```

## 7\. Partitioning big tables

GitLab uses partitioning to partition tables that can grow to a huge size. This is done to improve query performance.

*   PARTITION BY RANGE: This partitioning strategy works by partitioning table data based on the chosen range. This strategy is commonly used when you need to partition time-series data. The tables `audit_events` and `web_hook_logs` use this strategy.
*   PARTITION BY LIST: This partitioning strategy works by partitioning table data based on discrete values of a column. The table `loose_foreign_keys_deleted_records` uses this strategy.
*   PARTITION BY HASH: TThe table is partitioned by specifying a modulus and a remainder for each partition. Each partition will hold the rows for which the hash value of the partition key divided by the specified modulus will produce the specified remainder. The table `product_analytics_events_experimental` uses this strategy.

You can read more about Postgres partitioning in Postgres [documentation](https://www.postgresql.org/docs/current/ddl-partitioning.html).

## 8\. Supporting LIKE search use cases with Trigrams and `gin_trgm_ops`

GitLab uses GIN(Generalized Inverted Index) indexes to perform efficient searches.

> The GIN index type was designed to deal with data types that are subdividable and you want to search for individual component values (array elements, lexemes in a text document, etc)” – [Tom Lane](https://www.postgresql.org/message-id/flat/26038.1559516834%40sss.pgh.pa.us#ccb004aefc151d913e7a274a9b30c631)

Due to the nature of the LIKE operation, which supports arbitrary wildcard expressions, this is fundamentally hard to index. One such example is the issues table where you might want to do something like search on title and description fields. So, we use the pg_trgm extension to create an index that works on trigrams.

```
CREATE INDEX index_issues_on_title_trigram ON issues USING gin (title gin_trgm_ops);
CREATE INDEX index_issues_on_description_trigram ON issues USING gin (description gin_trgm_ops);

```

The GIN index makes searches performant. Let’s see that in action.

We will create a simple table as shown below.

```
create table words(id serial primary key, word text not null);

```

Let’s insert some data. I pulled an English word list in CSV format from this [link](https://www.bragitoff.com/2016/03/english-dictionary-in-csv-format/).

```
\copy words(word) from '/Users/xxx/Aword.csv' CSV;

```

```
select count(*) from words;

```

```
 count
-------
 11616
(1 row)

```

We will create a btree index on the word column and later we will use the gin index to show its efficiency.

```
create index id1 on words using btree (word);

```

Let’s run the explain plan query.

```
EXPLAIN select * from words where word like '%bul%';

```

```
                        QUERY PLAN
-----------------------------------------------------------
 Seq Scan on words  (cost=0.00..211.20 rows=1 width=14)
   Filter: (word ~~ '%bul%'::text)
(2 rows)

```

Now, let’s drop btree index;

Install the `pg_trm` extension

```
CREATE EXTENSION pg_trgm;

```

Create the index.

```
create index index_words_on_word_trigram ON words USING gin (word gin_trgm_ops);

```

Now, let;s run explain

```
EXPLAIN select count(*) from words where word like '%bul%';

```

```
                                             QUERY PLAN
----------------------------------------------------------------------------------------------------
 Aggregate  (cost=16.02..16.03 rows=1 width=8)
   ->  Bitmap Heap Scan on words  (cost=12.01..16.02 rows=1 width=0)
         Recheck Cond: (word ~~ '%bul%'::text)
         ->  Bitmap Index Scan on index_words_on_word_trigram  (cost=0.00..12.01 rows=1 width=0)
               Index Cond: (word ~~ '%bul%'::text)
(5 rows)

```

GitLab also makes use of `tsvector` to support complete full text search.

The advantages of doing text seach in your primary datastore are:

*   Real time indexes. No lag to create index
*   Access to the complete data
*   Less complexity in your architecture

## 9\. Use of `jsonb`

As I discussed an earlier [post](https://shekhargulati.com/2022/01/08/when-to-use-json-data-type-in-database-schema-design/) I use json data type in schema design for following use cases:

1.  Dump request data that will be processed later
2.  Support extra fields
3.  One To Many Relationship where many side will not have to its own identity
4.  Key Value use case
5.  Simpler EAV design

GitLab schema design also uses `jsonb` data type in multiple tables. They use it mainly for 1 and 2 use cases in my list above. The advantage of using jsonb over storing in plain text is the efficient querying supported by Postgres on `jsonb` data type.

The table `error_tracking_error_events` stores payload in jsonb data type. This is an example of dump request data that will be processed in a later use case. I covered a similar use case in my blog post so do read that for more information.

```
CREATE TABLE error_tracking_error_events (
    id bigint NOT NULL,
    payload jsonb DEFAULT '{}'::jsonb NOT NULL,
  // rest removed
);

```

You can use a JSON schema to validate the structure of a JSON document.

Another example is the `operations_strategies` table shown below. You don’t know how many parameters you might receive so you need a flexible data type like `jsonb`.

```
CREATE TABLE operations_strategies (
    id bigint NOT NULL,
    feature_flag_id bigint NOT NULL,
    name character varying(255) NOT NULL,
    parameters jsonb DEFAULT '{}'::jsonb NOT NULL
);

```

An example of supporting extra fields use cases is shown below.

```
CREATE TABLE packages_debian_file_metadata (
    created_at timestamp with time zone NOT NULL,
    updated_at timestamp with time zone NOT NULL,
    package_file_id bigint NOT NULL,
    file_type smallint NOT NULL,
    component text,
    architecture text,
    fields jsonb,
);

```

They also use jsonb for storing data that is already in JSON format. For example, in the table `vulnerability_finding_evidences` report data is already JSON so they saved it as is in `jsonb` data type.

```
CREATE TABLE vulnerability_finding_evidences (
    id bigint NOT NULL,
    created_at timestamp with time zone NOT NULL,
    updated_at timestamp with time zone NOT NULL,
    vulnerability_occurrence_id bigint NOT NULL,
    data jsonb DEFAULT '{}'::jsonb NOT NULL
);

```

## 10\. Other tidbits

*   Auditing fields like `updated_at` are only used in tables where records can be modified. For example `issues` has an `updated_at` column. For append-only immutable log tables like `audit_events` do not have an `updated_at` column as shown below in the code snippets. `issues` table with `updated_at` column

```
  CREATE TABLE issues (
      id integer NOT NULL,
      title character varying,
      author_id integer,
      project_id integer,
      created_at timestamp without time zone,
      updated_at timestamp without time zone,
      // removed remaining columns and constraints
  );

```

`audit_events` table with no `updated_at` column.

```
  CREATE TABLE audit_events (
      id bigint NOT NULL,
      author_id integer NOT NULL,
      entity_id integer NOT NULL,
      created_at timestamp without time zone NOT NULL,
      // removed remaining columns and constraints
  )

```

*   Enums are stored as smallint rather than `character varying`. It saves space. The only problem is you can’t change the order of enum values. In the example shown below `reason` and `severity_level` are enums

```
  CREATE TABLE merge_requests_compliance_violations (
      id bigint NOT NULL,
      violating_user_id bigint NOT NULL,
      merge_request_id bigint NOT NULL,
      reason smallint NOT NULL,
      severity_level smallint DEFAULT 0 NOT NULL
  );

```

*   Optimistic locking is used in a few(8) tables like `issues` and `ci_builds` to protect against edits from multiple parties. Optimistic locking assumes that there will be minimum such conflicts of data and if it does happen then the application throws an exception and the update is ignored. Active Record supports optimistic locking if the `lock_version` field is present. Each update to the record increments the `lock_version` column and the locking facilities ensure that records instantiated twice will let the last one saved raise a `StaleObjectError` if the first was also updated. The `ci_builds` table shown below uses the ‘lock_version` column.

```
  CREATE TABLE ci_builds (
      status character varying,
      finished_at timestamp without time zone,
      trace text,

        lock_version integer DEFAULT 0,

        // removed columns
      CONSTRAINT check_1e2fbd1b39 CHECK ((lock_version IS NOT NULL))
  );

```

*   Using inet for storing ip addresses. I was not aware of the `inet` type. They have used inet in `audit_events` and `authentication_events` tables

```
  CREATE TABLE audit_events (
      id bigint NOT NULL,
      ip_address inet,
    // other columns removed for clarity
  );

```

GitLab has not used inet in all the tables that store `ip_address`. For example, in tables `ci_runners` and `user_agent_details`, they have stored it as `character varying`. I am not sure why they have not used the same type in all the tables that store ip addresses.

You should prefer `inet` over storing an ip address as a plain text type as these types offer input error handling and specialized functions.

Let’s quickly see it in action. We will start by creating a table with two fields – id, and `ip_addr`

```
   create table e (id serial primary key, ip_addr inet not null);

```

We can insert a valid record like shown below.

```
  insert into e(ip_addr) values ('192.168.1.255');

```

We can also insert the record with a mask as shown below.

```
  insert into e(ip_addr) values ('192.168.1.5/24');

```

Both these records will get inserted

```
  select id, abbrev(ip_addr) from e;

```

```
   id |     abbrev
  ----+----------------
    1 | 192.168.1.255
    2 | 8.8.8.8
    3 | 192.168.1.5/24
  (3 rows)

```

If we try to save invalid data then insert will fail.

```
  insert into e(ip_addr) values ('192.168.1');

```

```
  ERROR:  invalid input syntax for type inet: "192.168.1"
  LINE 1: insert into e(ip_addr) values ('192.168.1');

```

You can inet operators supported by Postgres to check if an ip address is contained by subnet as shown below.

```
  select * from e where ip_addr << inet '192.168.1.1/24';

```

```
   id |    ip_addr
  ----+---------------
    1 | 192.168.1.255
  (1 row)

```

If we want to check if subnet is contained or equal then we do following

```
  select * from e where ip_addr <<= inet '192.168.1.1/24';

```

```
   id |    ip_addr
  ----+----------------
    1 | 192.168.1.255
    3 | 192.168.1.5/24
  (2 rows)

```

There are many other operators and functions supported by Postgres. You can read them in the Postgres [docs](https://www.postgresql.org/docs/current/functions-net.html).

*   Postgres ‘bytea`data type is used to store`SHA` , encrypted tokens, encrypted keys, encrypted password, fingerprints, etc.
*   Postgres array types are used for storing columns with multiple values as shown below.

> Arrays are to be used when you are absolutely sure you don’t need to create any relationship between the items in the array with any other table. It should be used for a tightly coupled one to many relationship. – [Link](https://stackoverflow.com/a/56298555)

For example in the table shown below we are storing `*_ids` as an array rather than storing them in a flat manner and defining relationships with other tables. You don’t know how many users and projects will be mentioned so it will be wasteful to create columns like `mentioned_user_id1` , `mentioned_user_id2`, `mentioned_user_id3` and so on.

```
  CREATE TABLE alert_management_alert_user_mentions (
      id bigint NOT NULL,
      alert_management_alert_id bigint NOT NULL,
      note_id bigint,
      mentioned_users_ids integer[],
      mentioned_projects_ids integer[],
      mentioned_groups_ids integer[]
  );

```

Another common use case of Postgres array is to store fields like hosts, tags, urls.

```
  CREATE TABLE dast_site_profiles (
      id bigint NOT NULL,
      excluded_urls text[] DEFAULT '{}'::text[] NOT NULL,
  );
  CREATE TABLE alert_management_alerts (
    id bigint NOT NULL,
    hosts text[] DEFAULT '{}'::text[] NOT NULL,
  );
  CREATE TABLE ci_pending_builds (
      id bigint NOT NULL,
      tag_ids integer[] DEFAULT '{}'::integer[],
  );

```

## Conclusion

I learnt a lot from the GitLab schema. They don’t blindly apply the same practices to all the table designs. Each table makes the best decision based on its purpose, the kind of data it stores, and its rate of growth.

## References

1.  Gitlab schema `structure.sql` – [Link](https://gitlab.com/gitlab-org/gitlab/-/blob/master/db/structure.sql)
2.  Issue 29465: Use structure.sql instead of schema.rb – [Link](https://gitlab.com/gitlab-org/gitlab/-/issues/29465)
3.  Choosing Primary Key Type in Postgres – [Link](https://shekhargulati.com/2022/06/23/choosing-a-primary-key-type-in-postgres/)
4.  Github’s Path to 128M public repositories – [Link](https://towardsdatascience.com/githubs-path-to-128m-public-repositories-f6f656ab56b1#:~:text=There%20are%20over%20128%20million%20public%20repositories%20on%20GitHub.)
5.  Postgres Character Types Documentation – [Link](https://www.postgresql.org/docs/current/datatype-character.html)
6.  Difference between text and varchar (character varying) – [Link](https://stackoverflow.com/questions/4848964/difference-between-text-and-varchar-character-varying)
7.  CHAR(x) vs. VARCHAR(x) vs. VARCHAR vs. TEXT – [Link](https://www.depesz.com/2010/03/02/charx-vs-varcharx-vs-varchar-vs-text/)