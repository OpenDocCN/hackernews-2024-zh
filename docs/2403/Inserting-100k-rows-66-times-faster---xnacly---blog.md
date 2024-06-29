<!--yml

分类：未分类

日期：2024-05-29 12:31:43

-->

# 插入100k行速度提升66倍 | xnacly - 博客

> 来源：[https://xnacly.me/posts/2024/faster-inserts/](https://xnacly.me/posts/2024/faster-inserts/)

在实现大部分离线应用的初始数据同步逻辑过程中，我注意到我的数据库抽象化在将近75万行插入单个数据库表时需要花费长达2分钟的时间。

我开始解决这个瓶颈。

## 问题和挑战 [##](#problems-and-issues)

在我甚至能够解决上述性能问题之前，出现了一个新问题：起初，我使用了散布语法来变化函数参数的可变数量。

```
1class Database { 2 async create(...items) {} 3} 4new Database().create(...new Array(750_000)); 
```

这污染了调用堆栈，并迅速导致堆栈溢出：

```
 1/home/teo/programming_wsl/blog_test/main.js:7 2new Database().create(...new Array(750_000)); 3 ^ 4 5RangeError: Maximum call stack size exceeded 6 at Object.<anonymous> (/home/teo/programming_wsl/blog_test/main.js:7:16) 7 at Module._compile (node:internal/modules/cjs/loader:1275:14) 8 at Module._extensions..js (node:internal/modules/cjs/loader:1329:10) 9 at Module.load (node:internal/modules/cjs/loader:1133:32) 10 at Module._load (node:internal/modules/cjs/loader:972:12) 11 at Function.executeUserEntryPoint [as runMain] (node:internal/modules/run_main:83:12) 12 at node:internal/main/run_main_module:23:47 13 14Node.js v19.9.0 
```

相信我，这在回顾中似乎很明显，但我花了一整天的时间来梳理TypeScript源映射以找到原因。

### 结论

我永远不会再使用散布操作符了 - 瞧你，React.js

## 首次估计和基准测试 [##](#first-estimates-and-benchmarking)

解决这个问题使我能够调用我的 `create` 函数，并将要插入的行传递给它。最初的测试结果令人失望：插入50万行需要57秒，插入75万行需要1分10秒 - 这太慢了。

```
 1 async create(items) { 2 return new Promise((res, rej) => { 3 this.connection.serialize(() => { 4 this.connection.run("BEGIN TRANSACTION"); 5 for (let i = 0; i < items.length; i++) { 6 try { 7 this.connection.run( 8 "INSERT INTO user (name, age) VALUES (?, ?)", 9 items[i]?.name, 10 items[i]?.age 11 ); 12 } catch (e) { 13 this.connection.run("ROLLBACK TRANSACTION"); 14 return rej(e); 15 } 16 } 17 this.connection.run("COMMIT"); 18 return res(); 19 }); 20 }); 21 } 
```

我的下一个想法是获取具体和可重复的数字，因此我创建了一个基准测试，用于上述简化的 `create` 实现，负载不同，从10行开始到一百万行结束：

```
 1const Database = require("./main.js"); 2 3const DB = new Database(); 4 5const amount = [10, 100, 1000, 10_000, 100_000, 1_000_000]; 6const data = { name: "xnacly", age: 99 }; 7 8describe("create", () => { 9 for (const a of amount) { 10 let d = new Array(a).fill(data); 11 test(`create-${a}`, async () => { 12 await DB.create(d); 13 }); 14 } 15}); 
```

测量时间：

|  | `create` |
| --- | --- |
| 10 行 | 2ms |
| 100 行 | 1ms |
| 1,000 行 | 13ms |
| 10,000 行 | 100ms |
| 100,000 行 | 1089ms |
| 1,000,000 行 | 11795ms |

## 方法 [##](#approaches)

我的第一个想法是在明显包含在创建方法循环中的热路径中省略调用。仔细一看，我注意到这里没有重的或者很多操作。我该如何改进数据库交互本身？这是我偶然发现批量插入的时刻。

```
1INSERT INTO user (name, age) VALUES ("xnacly", 99); 
```

天真的示例实现为每行插入都进行一次数据库调用。通过将更多值元组附加到上述语句，批量插入可将其减少为对数据库层的单一调用：

```
1INSERT INTO user (name, age) VALUES 2 ("xnacly", 99), 3 ("xnacly", 99), 4 ("xnacly", 99); 
```

使用上述内容实现更快的 `create` 方法如下：

```
 1 async createFast(items) { 2 if (!items.length) return Promise.resolve(); 3 let insert = "INSERT INTO user (name, age) VALUES "; 4 insert += new Array(items.length).fill("(?,?)").join(","); 5 let params = new Array(items.length * 2); 6 let i = 0; 7 for (const item of items) { 8 params[i] = item.name; 9 params[i + 1] = item.age; 10 i += 2; 11 } 12 return new Promise((res, rej) => { 13 this.connection.serialize(() => { 14 this.connection.run("BEGIN TRANSACTION"); 15 try { 16 this.connection.run(insert, params); 17 } catch (e) { 18 this.connection.run("ROLLBACK TRANSACTION"); 19 return rej(e); 20 } 21 this.connection.run("COMMIT"); 22 }); 23 return res(); 24 }); 25 } 
```

扩展我们对 `createFast` 的测试：

```
1describe("createFast", () => { 2 for (const a of amount) { 3 let d = new Array(a).fill(data); 4 test(`createFast-${a}`, async () => { 5 await DB.createFast(d); 6 }); 7 } 8}); 
```

|  | `create` | `createFast` | 改进 |
| --- | --- | --- | --- |
| 10 行 | 2ms | 1ms | 2x |
| 100 行 | 1ms | 1ms | / |
| 1,000 行 | 13ms | 3ms | 4.3x |
| 10,000 行 | 100ms | 22ms | 4.5x |
| 100,000 行 | 1089ms | 215ms | 5.1x |
| 1,000,000 行 | 11795ms | 1997ms | 5.9x |

### 信息

之前的基准测试是在以下系统上完成的：

+   CPU：Intel(R) Core(TM) i5-6200U CPU @ 2.30GHz 4 cores

+   RAM：8.0 GB DDR3 1600 MHz

在基准测试时系统负载较重，因此记录的时间仍然相当慢。

## 从微基准测试中走出来 [##](#escaping-from-micro-benchmarks-into-the-real-world)

将微基准测试结果应用到我的真实项目数据库层，导致显著的运行时改进，最高提速达到66倍。

|  | `create` | `createFast` | 改进 |
| --- | --- | --- | --- |
| 10 行 | 7ms | 2ms | 3.5倍 |
| 100 行 | 48ms | 1ms | 48倍 |
| 1000 行 | 335ms | 9ms | 37.2倍 |
| 10000 行 | 2681ms | 80ms | 33.5倍 |
| 100000 行 | 25347ms | 390ms | 66倍 |
