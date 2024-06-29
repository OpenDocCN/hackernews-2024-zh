<!--yml

类别：未分类

日期：2024-05-27 15:01:26

-->

# 完全开源的SQLite加密

> 来源：[https://blog.turso.tech/fully-open-source-encryption-for-sqlite-b3858225](https://blog.turso.tech/fully-open-source-encryption-for-sqlite-b3858225)

SQLite是许多人信赖的数据库，但它有一个不接受贡献的封闭社区。正因如此，尽管SQLite非常多才多艺，但在SQLite核心用例之外重要的改进往往无法完成，即使有人愿意去做。

这是[Turso](https://turso.tech)开始[libSQL](https://turso.tech/libsql)项目的驱动力，一个SQLite的全开放贡献分支。我们在Turso的使命是赋予用户生产力，以便他们在生产中使用SQLite来支持他们的应用程序。所以在像本地复制、自动备份到S3和无服务器模式等功能之后，我们正在向libSQL添加另一个对生产工作负载至关重要的功能：**静态加密**。

## [#](#how-does-it-work-)[](#how-does-it-work)它是如何工作的？

新的加密功能完全开源，供所有人使用，不依赖于Turso平台。为了理解它是如何工作的，让我们快速复习一下如何在第一次使用libSQL时使用它。

因为SQLite是嵌入到应用程序中的库，SQLite的用户必须用我们的SDK替换他们的SQLite使用。在下面的示例中，使用Typescript时，客户端通过传递一个URL来创建，这个URL在这种情况下是本地文件系统中的一个文件。

这个非常简单的示例将在一次传递中创建一个表，向其中添加数据，并进行查询：

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

类似于SQLite，这会在您的本地计算机上创建一个文件，您可以使用SQLite自身检查它：

```
$ file sqlite.db
sqlite.db: SQLite 3.x database, last written using SQLite version 3044000, file counter 1, database pages 2, cookie 0x1, schema 4, UTF-8, version-valid-for 1

$ sqlite3 sqlite.db ".schema"
CREATE TABLE my_table(var integer); 
```

加密文件很简单。你只需在客户端构造函数中添加一个额外的属性，`encryptionKey`：

```
const db = createClient({
  url: 'file:sqlite-enc.db',
  encryptionKey: process.env.ENCRYPTION_KEY,
}); 
```

在使用新的客户端配置执行相同代码后，文件已经不再是以前的样子了！

```
$ file sqlite-enc.db
sqlite-enc.db: data

$ sqlite3 sqlite-enc.db ".schema"
Error: file is not a database 
```

从SQLite的角度来看，这只是原始字节，无法读取。但通过传递正确的加密密钥，我们的程序可以轻松读取它：

```
const db = createClient({
  url: 'file:sqlite-enc.db',
  encryptionKey: process.env.ENCRYPTION_KEY,
});

await db.execute('SELECT * FROM my_table'); 
```

## [#](#how-does-it-work-)[](#how-does-it-work-1)它是如何工作的？

在创建libSQL分支时，我们的一个指导原则是，社区通过一个开放给任何人的通用SQLite分支会得到更好的服务。SQLite的特殊用途操作的分支在开放中存在，不是什么新鲜事，但由于其狭窄的特性，它们得到的分发不如它们应得的多。

加密并不例外。事实上，存在许多分支，这些分支为SQLite添加了加密功能。经过一些研究、代码分析和仔细考虑，我们决定将代码移至我们的分支，比起编写全新的加密代码或要求用户加载扩展，效果更好。其中一个项目特别适合我们，[SQLite多重加密](https://utelle.github.io/SQLite3MultipleCiphers/)。因为它是根据MIT许可证授权的，我们刚刚将代码移至libSQL中。

将代码移到libSQL使我们能够在加密基础上构建其他内容，并提供我们在前一节展示过的透明、开箱即用的体验。

密码器可按数据库配置。我们目前支持SQLCipher（默认）和wxSQLite3的AES 256位。

无需解密整个文件即可读取数据：页面单独加密，特别是针对libSQL，我们已经扩展工作，确保WAL也被加密，以及发送到S3的备份也是如此。

## [#](#what-does-it-mean-for-turso-users-)[](#what-does-it-mean-for-turso-users)对Turso用户意味着什么？

Turso基础设施中的所有卷都由我们的云服务提供商在静止状态下进行加密。虽然这对各种情况已经足够好，但本文描述的文件级加密可用于提供额外的安全级别。

嵌入式副本加密即将推出，并且我们计划在未来增加两个强大的数据库安全功能：

1.  **自带密钥（BYOK）**用于静止时加密：允许您使用由您提供的密钥加密您的任何或所有数据库，并在每个请求中传递给我们。加密密钥永远不会存储在我们的服务器上，确保即使我们的基础设施发生违规，您的数据也是安全的。

1.  **静止时每数据库加密**：允许您为每个数据库使用不同的密钥。对于遵循租户数据库模式的用户，每个租户可以拥有自己的密钥，确保即使出现错误，也不会混淆用户数据。我们可以管理密钥，或者您可以使用BYOK自行管理。

如果您想参加这些高级功能的早期私人测试版，请填写[此表格](https://docs.google.com/forms/d/e/1FAIpQLSd5wRfM214ljUP9W66FhugGiji%5FP45qMchieTqNn1wdNsw8xg/viewform)。
