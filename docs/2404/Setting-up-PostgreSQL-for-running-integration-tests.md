<!--yml

类别：未分类

日期：2024-05-27 13:21:04

-->

# 设置 PostgreSQL 以运行集成测试

> 来源：[https://gajus.com/blog/setting-up-postgre-sql-for-running-integration-tests](https://gajus.com/blog/setting-up-postgre-sql-for-running-integration-tests)

在进行测试时，实现性能和可靠性至关重要。在本文中，我将解释如何为测试设置 [PostgreSQL](https://www.postgresql.org/)，并讨论一些常见的要避免的问题。

在我们深入细节之前，让我们定义我们的目标：

+   **隔离性** – 我们希望确保每个测试都在隔离环境中运行。至少，这意味着每个测试应该有自己的数据库。这确保了测试不会互相干扰，并且可以在并行运行测试时不会出现任何问题。

+   **性能** – 我们希望确保设置 PostgreSQL 用于测试是快速的。慢速的解决方案将会对在 CI/CD 流水线中运行测试造成经济上的限制。我们提出的解决方案必须允许我们执行测试而不引入过多的开销。

本文其余部分将专注于我们尝试过的方法，有效的方法以及未成功的方法。

我们尝试的第一种方法是使用 [transactions](https://www.postgresql.org/docs/current/tutorial-transactions.html)。我们将在每个测试开始时启动一个事务，并在结束时回滚。

基本思想如下示例所示：

```
test('calculates total basket value', async () => {
 await pool.transaction(async (tx) => {
 await tx.query(sql.unsafe`
 INSERT INTO basket (product_id, quantity)
 VALUES (1, 2)
 `);

 const  total  =  await  getBasketTotal(tx);

 expect(total).toBe(20);
 });
});
```

事务方法对于简单情况（例如测试单个函数）效果很好，但在处理涉及多个组件之间的集成测试时很快变成问题。由于连接池、嵌套事务和其他因素，使事务方法工作所需的工作量意味着我们无法复制应用程序的真实行为，即无法提供我们需要的信心。

为了保持一致性，我们还希望避免混合测试方法。即使使用事务对某些测试来说已经足够，我们也希望在所有测试中保持一致的方法。

我们尝试的另一种方法是使用 SQLite。SQLite 是一个内存数据库，快速且易于设置。

与事务方法类似，SQLite 对于简单情况效果很好。然而，在处理使用 PostgreSQL 特定功能的代码路径时，它很快变成一个问题。在我们的情况下，由于使用了各种 PostgreSQL 扩展、PL/pgSQL 函数和其他 PostgreSQL 特定功能，我们无法将 SQLite 用于我们的测试。

[pglite](https://github.com/electric-sql/pglite) 提供了作为 [WASM 模块](https://developer.mozilla.org/en-US/docs/WebAssembly/JavaScript_interface/Module) 打包的 PostgreSQL，可以在 Node.js 中使用。这可能是一个很好的替代方案，尽管我们尚未尝试过。但目前不支持扩展将成为我们的障碍。

我们尝试的另一种方法是使用[`pg_tmp`](https://eradman.com/ephemeralpg/)。`pg_tmp`是一个为每个测试创建临时PostgreSQL实例的工具。

理论上，`pg_tmp`是一个很好的解决方案。它允许完全隔离测试。实际上，它比我们能够容忍的要慢得多。使用`pg_tmp`，启动和填充数据库需要几秒钟，当运行数千个测试时，这种额外开销会迅速累积。

假设您有1000个测试，每个测试需要1秒钟。如果为创建新数据库增加2秒的开销，那么额外的开销将达到2000秒（33分钟）。

如果您喜欢这种方法，您也可以考虑使用Docker容器。根据许多因素，Docker容器可能比`pg_tmp`更快。

[integresql](https://github.com/allaboutapps/integresql)是我在[HN帖子](https://news.ycombinator.com/item?id=26947964)中了解到的一个项目。它似乎是一个很好的替代方案，可以将创建新数据库的开销减少到约500毫秒。它具有一种池化机制，允许进一步减少开销。我们决定不继续这条道路，因为使用[模板数据库](#template-databases)已经满足了我们的隔离需求。

在尝试了各种方法后，我们决定结合两种方法：[*模板数据库*](#template-databases)和[*挂载内存磁盘*](#mounting-a-memory-disk)。这种方法使我们能够在不引入过多开销或复杂性的情况下，在数据库级别上隔离每个测试。

[模板数据库](https://www.postgresql.org/docs/current/manage-ag-templatedbs.html)是用作创建新数据库模板的数据库。从模板数据库创建新数据库时，新数据库与模板数据库具有相同的模式。创建新数据库的步骤如下：

1.  创建模板数据库 (`ALTER DATABASE <database_name> is_template=true;`)

1.  从模板数据库创建新数据库 (`CREATE DATABASE <new_database_name> TEMPLATE <template_database_name>;`)

使用模板数据库的主要优势在于，您无需处理管理多个PostgreSQL实例的麻烦。您可以创建副本数据库，并使每个测试在隔离环境中运行。

然而，单独使用模板数据库对于我们的用例来说并不够快。从模板数据库创建新数据库所需的时间仍然对于运行数千个测试来说太高：

```
postgres=# CREATE  DATABASE foo TEMPLATE contra;
CREATE  DATABASE
Time: 1999.758 ms (00:02.000)
```

这就是[内存挂载](#mounting-a-memory-disk)的应用场景。

注意到的另一个模板数据库的限制是，在复制过程中不允许有其他会话连接到源数据库。如果在开始时存在任何其他连接，`CREATE DATABASE`将失败；在复制操作期间，新连接到源数据库是被阻止的。使用[互斥模式](https://en.wikipedia.org/wiki/Lock_(computer_science))可以很容易地解决这个限制，但这是需要注意的事项。

解决方案的最后一部分是挂载内存磁盘。通过挂载内存磁盘，并在内存磁盘上创建模板数据库，我们可以显著减少创建新数据库的开销。

我将在下一节讨论如何挂载内存磁盘，但首先让我们看看它会带来多大的改变。

```
postgres=# CREATE  DATABASE bar TEMPLATE contra;
CREATE  DATABASE
Time: 87.168 ms
```

这是一个重大的改进，使得这种方法对我们的使用情况可行。

毫无疑问，这种方法并非没有缺点。数据存储在内存中，这意味着它不是持久的。如果数据库崩溃或服务器重新启动，数据将丢失。然而，对于运行测试来说，这并不是问题。每次创建新数据库时，数据都会从模板数据库重新创建。

我们选择的方法是使用带有内存磁盘的Docker容器。以下是设置步骤：

```
$  docker  run  \
 -p  5435:5432  \
 --tmpfs  /var/lib/pg/data  \
 -e  PGDATA=/var/lib/pg/data  \
 -e  POSTGRES_PASSWORD=postgres  \
 --name  contra-database  \
 --rm  \
 postgres:14
```

在上述命令中，我们正在创建一个Docker容器，并在`/var/lib/pg/data`挂载一个内存磁盘。我们还将`PGDATA`环境变量设置为`/var/lib/pg/data`，以确保PostgreSQL使用内存磁盘进行数据存储。最终的结果是基础数据存储在内存中，显著减少了创建新数据库的开销。

基本思想是在运行测试之前创建一个模板数据库，然后为每个测试从模板数据库创建一个新数据库。以下是如何管理测试数据库的简化版本：

```
import {
 createPool,
 sql,
 stringifyDsn,
} from  'slonik';

type  TestDatabase  = {
 destroy: () =>  Promise<void>;
 getConnectionUri: () =>  string;
 name: () =>  string;
};

const  createTestDatabasePooler  =  async (connectionUrl:  string) => {
 const  pool  =  await  createPool(connectionUrl, {
 connectionTimeout: 5_000,
 // This ensures that we don't attempt to create multiple databases in parallel.
 maximumPoolSize: 1,
 });

 const  createTestDatabase  =  async (
 templateName:  string,
 ):  Promise<TestDatabase> => {
 const  database  =  'test_'  +  uid();

 await pool.query(sql.typeAlias('void')`
 CREATE DATABASE ${sql.identifier([database])}
 TEMPLATE ${sql.identifier([templateName])}
 `);

 return {
 destroy: async () => {
 await pool.query(sql.typeAlias('void')`
 DROP DATABASE ${sql.identifier([database])}
 `);
 },
 getConnectionUri: () => {
 return  stringifyDsn({
 ...parseDsn(connectionUrl),
 databaseName: database,
 password: 'unsafe_password',
 username: 'contra_api',
 });
 },
 name: () => {
 return database;
 },
 };
 };

 return () => {
 return  createTestDatabase('contra_template');
 };
};

const  getTestDatabase  =  await  createTestDatabasePooler();
```

到此时，你可以使用`getTestDatabase`来为每个测试创建一个新数据库。`destroy`方法可用于在测试运行后清理数据库。

这种设置允许我们在多个分片上并行运行成千上万个测试，而无需任何问题。创建新数据库的开销很小，而且隔离是在数据库级别进行的。我们对这种设置提供的性能和可靠性非常满意。
