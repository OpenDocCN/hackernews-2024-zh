<!--yml

category: 未分类

date: 2024-05-27 14:29:45

-->

# 你的测试套件是否考虑到了**弱事务隔离**？ | 关于质量

> 来源：[https://concerningquality.com/txn-isolation-testing/](https://concerningquality.com/txn-isolation-testing/)

事务隔离是一种你学到后会让你感到恐惧的东西。具体来说，有一些*弱*事务隔离级别允许一些相当意外的行为。像 Jepsen 这样的工具用于测试数据库的一般隔离保证，但检查应用层与隔离异常相关的问题相当不常见。这些异常可能会影响实际的领域逻辑，所以理解它们以及我们如何测试它们是很重要的。

# 什么是**弱事务隔离**？

事务隔离意味着对数据库的并发事务将彼此独立。这是 ACID 中的“I”。不幸的是，在这种情况下，“独立”是一个光谱，并且实际上支持不同的隔离级别，每个级别的行为略有不同。

这是一个快速示例脚本，它对数据库（Postgres）进行并发查询：

```
create table txn_iso (ival int);
insert into txn_iso (ival) values(1); 
```

```
import { Pool, Transaction } from 'https://deno.land/x/postgres/mod.ts';

const pool = new Pool({
  user: 'postgres',
  hostname: 'localhost',
  database: 'postgres',
  port: 5433,
  password: 'test1234',
}, 10);

async function runQuery(
  txn: Transaction,
  query: string,
  args: (string | number)[],
  beforeMsg: string,
  afterMsg: (result: any) => string
) {
  console.log(beforeMsg);
  const result = await txn.queryObject(query, args);
  console.log(afterMsg(result));
}

async function readTransaction() {
  const query = 'select ival from txn_iso';
  const printResult = (result: any) => `Read result: ${result.rows[0].ival}`;

  const client = await pool.connect();
  const txn = client.createTransaction();

  await txn.begin();

  await runQuery(txn, query, [], 'Executing first read...',  printResult);

  // Wait for concurrent write to occur
  await new Promise(resolve => setTimeout(resolve, 2000));

  await runQuery(txn, query, [], 'Executing second read...', printResult);

  await txn.commit();

  await client.release();
}

async function writeTransaction() {
  await new Promise(resolve => setTimeout(resolve, 1000));
  const client = await pool.connect();
  const txn = client.createTransaction();

  await txn.begin();

  const updateVal = Math.floor(Math.random() * 1000);
  const updateMsgBefore = `Updating ival to ${updateVal}...`;
  const query = 'update txn_iso set ival = $1';
  await runQuery(txn, query, [updateVal], updateMsgBefore, () => 'ival updated');

  await txn.commit();

  await client.release();
}

await Promise.allSettled([readTransaction(), writeTransaction()]); 
```

此脚本同时执行两个事务：一个事务两次读取 `txn_iso.ival` 列的值，另一个事务修改该列的值。为了让第二次读取发生在写操作之后，这里插入了一些休眠。问题是：两次读取会返回相同的值吗？

在 Postgres 中，使用默认的事务隔离级别，答案出乎意料地是否定的。这是运行脚本的一个示例输出：

```
Executing first read...
Read result: 839
Updating ival to 79...
ival updated
Executing second read...
Read result: 79 
```

第一次读取将返回读事务开始时列的值，但第二次读取将返回并发写事务更新的值。这是因为默认级别是提交读，它允许非可重复读取。非可重复读取意味着在同一事务的范围内，对同一列的查询可能返回不同的结果！这在 Postgres 中并不是唯一的 - 提交读也是 Oracle 和 SQL Server 中的默认隔离级别^。

这对很多人来说是令人惊讶的，而且理所当然，因为它似乎违背了事务的定义。但这是因为提交读是一个*弱*事务隔离级别。弱隔离意味着事务并不真正独立于彼此，一个并发事务的影响可以在另一个事务中看到。ANSI SQL 标准定义了 4 个隔离级别^（除了最严格的 Serializable 外，所有级别都是弱的，并允许事务之间的某种干扰）。

希望现在清楚为什么这是一个问题了。如果你有一个重要的列值，比如用户的账户余额，你可能会在同一个事务中查询多个不同的值，这肯定会导致领域逻辑错误。我们的测试套件会捕捉到这种错误吗？这取决于我们如何设置测试。

# 模拟并发连接

测试出现事务隔离异常的困难在于测试必须模拟多个并发连接。测试用例几乎总是具有隐含的假设，即它们由单个用户执行，并且隔离异常不会在该场景中出现。

例如，以下是一个简化的代码，用于从具有透支保护的帐户进行出站转账：

```
interface BalanceRepository {
  getBalance(txn: Transaction): Promise<number>;
  updateBalance(txn: Transaction, amount: number): Promise<void>;
}

async function checkOverdraftProtection(txn: Transaction, balanceRepo: BalanceRepository, amount: number) {
  const balance =  await balanceRepo.getBalance(txn);
  if (balance >= amount) {
    return;
  }

  balanceRepo.updateBalance(txn, balance + 100);
}

async function applyFundTransfer(txn: Transaction, balanceRepo: BalanceRepository, amount: number) {
  const balance = await balanceRepo.getBalance(txn);
  if (balance < amount) {
    console.error("Insufficient funds");
    return;
  }

  await balanceRepo.updateBalance(txn, balance - amount);
}

async function transferFunds(balanceRepo: BalanceRepository, amount: number) {
  const client = await pool.connect();
  const txn = client.createTransaction();

  await txn.begin();

  await checkOverdraftProtection(txn, balanceRepo, amount);
  await applyFundTransfer(txn, balanceRepo, amount);

  await txn.commit();

  await client.release();
}

const protectedBalanceRepo = {
  balance: 90,
  async getBalance(txn: Transaction) {
    return this.balance;
  },
  async updateBalance(txn: Transaction, amount: number) {
    this.balance = amount;
  }
}

await transferFunds(protectedBalanceRepo, 100);
console.assert(protectedBalanceRepo.balance === 90); 
```

我们想要测试的主要逻辑是透支保护在没有足够资金覆盖转账时添加额外资金，并且最终余额正确。为了测试这一点，我们将所有查询都放在一个`BalanceRepository`接口后面，并创建一个`protectedBalanceRepo`，该接口开始时资金不足，但根据透支保护更新余额。

这是从单个用户的角度来看的操作，因此是单个DB连接，因此不会出现资金不足的错误。但是，正如我们在Read Committed示例中看到的那样，另一个并发事务可能会影响多次读取的值。因此，模拟并发事务的一种方法是简单地忽略透支保护，并直接指定不同的余额结果。

```
const concurrentWriteRepo = {
  currBalance: 0,
  balances: [100, 90],
  async getBalance(txn: Transaction) {
    return this.balances[this.currBalance++];
  },
  async updateBalance(txn: Transaction, amount: number) {
  }
}

...

await transferFunds(concurrentWriteRepo, 100); 
```

此测试双重设置了两种不同的余额结果：首先将返回100，这将绕过透支保护，但下一个余额检查将返回90，这将导致资金不足的错误。在现实生活中，这种情况可能发生的一种方式是如果多个人都可以访问同一个帐户，并在彼此之间的时间接近时发起转账。

对于这种失败，有一个简单的修复方法：只需不要多次读取余额，而是将采样的余额传递给任何需要它的函数：

```
async function transferFunds(balanceRepo: BalanceRepository, amount: number) {
  const client = await pool.connect();
  const txn = client.createTransaction();

  await txn.begin();

  const balance = await balanceRepo.getBalance(txn);
  const protectedBalance = await checkOverdraftProtection(txn, balance, balanceRepo, amount);
  await applyFundTransfer(txn, protectedBalance, balanceRepo, amount);

  await txn.commit();

  await client.release();
} 
```

现在`checkOverdraftProtection`和`applyFundTransfer`的逻辑可以更改为使用余额值而不是查询它。这也意味着`checkOverdraftProtection`在应用保护后必须返回余额，因为`applyFundTransfer`过去用第二个余额查询获取此值，并且使用预保护余额将导致资金不足的错误。

通过避免多次读取，这解决了可重复读取的异常，但仍然存在一个主要问题：多个并发事务之间存在竞争条件，可能导致余额不正确。

# 竞争条件和可串行性

要显示错误，我们可以针对实际数据库同时执行两次资金转账，并引入写入延迟，以便我们可以控制最后写入的是哪一个（即使没有这个，我们也会看到错误，但这样可以减少非确定性）：

```
create table accounts (balance int);
insert into accounts (balance) values (100); 
```

```
async function transferFunds(balanceRepo: BalanceRepository, amount: number, writeDelay?: number) {
  const client = await pool.connect();
  const txn = client.createTransaction();

  await txn.begin();

  const balance = await balanceRepo.getBalance(txn);

  if (writeDelay) {
    await new Promise(resolve => setTimeout(resolve, writeDelay));
  }

  const protectedBalance = await checkOverdraftProtection(txn, balance, balanceRepo, amount);
  await applyFundTransfer(txn, protectedBalance, balanceRepo, amount);

  await txn.commit();

  await client.release();
}

const postgresBalanceRepo = {
  async getBalance(txn: Transaction): Promise<number> {
    const result = await txn.queryObject('select balance from accounts');
    return result.rows[0].balance;
  },
  async updateBalance(txn: Transaction, amount: number) {
    await txn.queryObject('update accounts set balance = $1', [amount]);
  }
}

async function runInTransaction<T>(f: (txn: Transaction) => Promise<T>) {
  const conn = await pool.connect()
  const txn = conn.createTransaction()
  await txn.begin();

  const result = await f(txn);

  await txn.commit();
  await conn.release();

  return result
}

// Setup
await runInTransaction((txn) => {
  return postgresBalanceRepo.updateBalance(txn, 100);
});

// Run two concurrent fund transfers
await Promise.allSettled([transferFunds(postgresBalanceRepo, 80, 2000), transferFunds(postgresBalanceRepo, 60)]);

const balance = await runInTransaction((txn) => {
  return postgresBalanceRepo.getBalance(txn)
});

console.assert(balance === 60); 
```

运行时，转账将看到相同的初始余额（100），但具有写入延迟的转账将覆盖另一个中设置的余额。这也意味着任何一个转账都不会触发透支保护，也不会出现资金不足的错误，最终余额将是错误的20。

这是一个 *序列化异常*。 Postgres 文档将其定义为：

> 成功提交一组事务的结果与依次运行这些事务的所有可能顺序的结果不一致。

这里只有两种资金转账的可能顺序：

1.  转账 80，然后转账 60

1.  转账 60，然后转账 80

由于初始余额为 100，这两种情况都应触发第二次转账的透支保护，结果余额应为 60（100 + 100 - 80 - 60）。 当事务不遵守可串行化时，竞争条件可能存在，这就是这里发生的情况 - 启动了两次资金转账，但只有一次被记入账户，因为有并发竞争。 这被称为“丢失更新”问题。

有几种不同的方法可以解决这个问题，但最简单的方法是使用 `FOR UPDATE` 在事务期间锁定行：

```
const postgresBalanceRepo = {
  async getBalance(txn: Transaction): Promise<number> {
    const result = await txn.queryObject('select balance from accounts FOR UPDATE');
    return result.rows[0].balance;
  },
  ...
} 
```

再次来自 Postgres 文档：

> FOR UPDATE 使由 SELECT 语句检索到的行像更新一样被锁定。 这阻止它们被其他事务锁定，修改或删除，直到当前事务结束。

这意味着每个事务将锁定`accounts`行，直到它完成，即事务将以可串行化的方式执行。 值得注意的是，这现在变慢了。 没有锁定，事务可以真正并发运行，但现在它们必须等待争夺对同一帐户的余额更新。 这是必要的正确行为，但值得理解权衡。

值得注意的是，仅仅使用测试替身是无法解决此问题的，因为修复程序在真实的 Postgres 存储库实现中。 在许多情况下，测试替身对独立于数据库的应用程序代码进行测试是有用的，但事务隔离是一种情况，其中不同级别之间的差异微妙地不同，所以最简单的方法是针对真实情况进行测试。

* * *

事务隔离对应用程序有重大影响，无论是性能还是对领域逻辑的影响。 对真实数据库进行集成测试非常重要，以确保弱事务隔离级别不是并发错误的原因。 为了暴露此类错误，我们必须在测试用例中执行至少两个并发事务。 不幸的是，这可能需要一定数量的基于时间的协调，这永远不是理想的，但通常是必要的，当像数据库这样的工具具有不受我们控制的不透明的非确定性行为时。

不过，这是我们可以而且应该在应用程序级别进行测试的东西。

* * *
