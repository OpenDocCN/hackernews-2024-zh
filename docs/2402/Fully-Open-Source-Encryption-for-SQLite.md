<!--yml
category: 未分类
date: 2024-05-27 15:01:26
-->

# Fully Open Source Encryption for SQLite

> 来源：[https://blog.turso.tech/fully-open-source-encryption-for-sqlite-b3858225](https://blog.turso.tech/fully-open-source-encryption-for-sqlite-b3858225)

SQLite is a database trusted by many, but it has a closed community that is not open to contributions. Because of that, despite SQLite’s incredible versatility, improvements that are important outside the central use cases SQLite tend not to get done, even though there are people willing to do them.

That was the driving force behind us at [Turso](https://turso.tech) starting the [libSQL](https://turso.tech/libsql) project, an Open Contribution fork of SQLite. Our mission at Turso is to empower users to use SQLite in production to power their applications. So after features like native replication, automatic backups to S3 and a serverless mode, we are adding to libSQL yet another feature that is critical to production workloads: **encryption at rest**.

## [#](#how-does-it-work-)[](#how-does-it-work)How does it work?

The new encryption feature is fully Open Source, available to everybody, and doesn’t depend on the Turso platform. To understand how it works, let’s take a quick refresher at how to use libSQL in the first place.

Because SQLite is a library that gets embedded into the application, users of SQLite have to replace their usage of SQLite with our SDK. In the example below, using Typescript, the client is created by passing a URL, which in this case is a file in the local filesystem.

This very simple example will just in one pass create a table, add data to it, and query:

```
import { createClient } from '@libsql/client';

const db = createClient({
  url: 'file:sqlite.db',
});

await db.batch([
  'CREATE TABLE IF NOT EXISTS my_table(var integer)',
  'INSERT INTO my_table (var) values (1)',
  'SELECT * FROM my_table',
]); 
```

Similar to SQLite, this creates a file on your local machine, which you can inspect using SQLite itself:

```
$ file sqlite.db
sqlite.db: SQLite 3.x database, last written using SQLite version 3044000, file counter 1, database pages 2, cookie 0x1, schema 4, UTF-8, version-valid-for 1

$ sqlite3 sqlite.db ".schema"
CREATE TABLE my_table(var integer); 
```

Encrypting the file is simple. All you have to do is pass one more property in the client constructor, `encryptionKey`:

```
const db = createClient({
  url: 'file:sqlite-enc.db',
  encryptionKey: process.env.ENCRYPTION_KEY,
}); 
```

After executing the same code with the new client configuration, the file is not what it once was!

```
$ file sqlite-enc.db
sqlite-enc.db: data

$ sqlite3 sqlite-enc.db ".schema"
Error: file is not a database 
```

From SQLite’s point of view, this is just raw bytes, and can’t be read. But passing the right encryption key, our program can read it just fine:

```
const db = createClient({
  url: 'file:sqlite-enc.db',
  encryptionKey: process.env.ENCRYPTION_KEY,
});

await db.execute('SELECT * FROM my_table'); 
```

## [#](#how-does-it-work-)[](#how-does-it-work-1)How does it work?

One of our guiding principles when creating the libSQL fork, is that the community would be better served by a general purpose fork of SQLite that was open to anybody. Forks of SQLite for special purpose operations exist in the open and are nothing new, but due to their narrow nature, they see less distribution than they deserve.

Encryption is not an exception. In fact, many forks exist that add encryption to SQLite. After some research, code analysis and careful consideration, we decided that just moving the code to our fork would yield better results than writing a whole new batch of encryption code, or requiring users to load extensions. One project in particular was very suitable for us, [SQLite Multiple Ciphers](https://utelle.github.io/SQLite3MultipleCiphers/). Since it is licensed under MIT, we have just moved the code into libSQL.

Moving the code into libSQL allows us to build other things on the foundation of encryption, and to provide the transparent, out-of-the-box experience we have shown in the previous section.

The cipher is configurable per database. We currently support SQLCipher (default) and wxSQLite3’s AES 256 Bit.

It is not necessary to decrypt the entire file to read data: pages are encrypted individually, and for libSQL in particular, we have extended to work to make sure that the WAL is also encrypted, as well as the backups that are sent to S3.

## [#](#what-does-it-mean-for-turso-users-)[](#what-does-it-mean-for-turso-users)What does it mean for Turso users?

All volumes in the Turso infrastructure are encrypted at rest by our cloud provider. While this is good enough for a variety of cases, the file-level encryption described in this post can be used to provide an extra level of security.

Encryption of embedded replicas is coming soon, and were also planning to add two powerful database security additions in the future:

1.  **Bring Your Own Key (BYOK)** for Encryption At Rest: Allows you to encrypt any or all of your databases with a key that is provided by you, and passed to us on every request. The encryption key is never stored at our servers, giving you peace of mind that even in the case of a breach of our infrastructure, your data is safe.
2.  **Per-Database Encryption** At Rest: Allows you to use different keys for each of your databases. For users following the pattern of per-tenant database, each tenant can have their own key, making sure that even if you make a mistake, data for your users is never mixed up. We can manage the keys, or you can do it, with BYOK.

If you’d like to join an early private beta for these premium features, please fill out [this form](https://docs.google.com/forms/d/e/1FAIpQLSd5wRfM214ljUP9W66FhugGiji%5FP45qMchieTqNn1wdNsw8xg/viewform).