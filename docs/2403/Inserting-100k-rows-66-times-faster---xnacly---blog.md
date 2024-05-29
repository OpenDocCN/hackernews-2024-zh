<!--yml
category: 未分类
date: 2024-05-29 12:31:43
-->

# Inserting 100k rows 66 times faster | xnacly - blog

> 来源：[https://xnacly.me/posts/2024/faster-inserts/](https://xnacly.me/posts/2024/faster-inserts/)

In the process of implementing the initial data syncing logic for a mostly offline application I noticed my database abstraction taking a solid 2 minutes for inserting short of 750k rows into a single database table.

I set out to fix this bottleneck.

## Problems and Issues [##](#problems-and-issues)

A new issue arose before i could even tackle the aforementioned performance problems: At first I used the spread syntax for a variadic amount of function parameters.

```
1class Database { 2 async create(...items) {} 3} 4new Database().create(...new Array(750_000)); 
```

This polluted the call stack and promptly caused a stack overflow:

```
 1/home/teo/programming_wsl/blog_test/main.js:7 2new Database().create(...new Array(750_000)); 3 ^ 4 5RangeError: Maximum call stack size exceeded 6 at Object.<anonymous> (/home/teo/programming_wsl/blog_test/main.js:7:16) 7 at Module._compile (node:internal/modules/cjs/loader:1275:14) 8 at Module._extensions..js (node:internal/modules/cjs/loader:1329:10) 9 at Module.load (node:internal/modules/cjs/loader:1133:32) 10 at Module._load (node:internal/modules/cjs/loader:972:12) 11 at Function.executeUserEntryPoint [as runMain] (node:internal/modules/run_main:83:12) 12 at node:internal/main/run_main_module:23:47 13 14Node.js v19.9.0 
```

Believe me this seems obvious in retrospect, but it took me a solid day of combing trough typescript source maps to find the cause.

### Takeaway

I will never use the spread operator again - looking at you react.js

## First Estimates and Benchmarking [##](#first-estimates-and-benchmarking)

Fixing this issue allowed me to call my `create` function and pass the rows to insert in. The first tests were nothing short of disappointing: 57 seconds for inserting 500k rows, 1min 10 seconds for inserting 750k rows - this was too slow.

```
 1 async create(items) { 2 return new Promise((res, rej) => { 3 this.connection.serialize(() => { 4 this.connection.run("BEGIN TRANSACTION"); 5 for (let i = 0; i < items.length; i++) { 6 try { 7 this.connection.run( 8 "INSERT INTO user (name, age) VALUES (?, ?)", 9 items[i]?.name, 10 items[i]?.age 11 ); 12 } catch (e) { 13 this.connection.run("ROLLBACK TRANSACTION"); 14 return rej(e); 15 } 16 } 17 this.connection.run("COMMIT"); 18 return res(); 19 }); 20 }); 21 } 
```

My next idea was to get specific and reproducible numbers, thus i created a benchmark, for the simplified `create` implementation above, with differing loads, starting with 10 rows and stopping at a million rows:

```
 1const Database = require("./main.js"); 2 3const DB = new Database(); 4 5const amount = [10, 100, 1000, 10_000, 100_000, 1_000_000]; 6const data = { name: "xnacly", age: 99 }; 7 8describe("create", () => { 9 for (const a of amount) { 10 let d = new Array(a).fill(data); 11 test(`create-${a}`, async () => { 12 await DB.create(d); 13 }); 14 } 15}); 
```

Measured times:

|  | `create` |
| --- | --- |
| 10 rows | 2ms |
| 100 rows | 1ms |
| 1,000 rows | 13ms |
| 10,000 rows | 100ms |
| 100,000 rows | 1089ms |
| 1,000,000 rows | 11795ms |

## Approaches [##](#approaches)

My first idea was to omit calls in the hot paths that obviously are contained in the create methods loop for inserting every row passed into it. After taking a look, i notice there are no heavy or even many operations here. How could I improve the database interaction itself? This was the moment I stumbled upon bulk inserts.

```
1INSERT INTO user (name, age) VALUES ("xnacly", 99); 
```

The naive example implementation makes a database call for every row it wants to insert. Bulk inserting reduces this to a single call to the database layer by appending more value tuples to the above statement:

```
1INSERT INTO user (name, age) VALUES 2 ("xnacly", 99), 3 ("xnacly", 99), 4 ("xnacly", 99); 
```

Using the above to implement a faster `create` method as follows:

```
 1 async createFast(items) { 2 if (!items.length) return Promise.resolve(); 3 let insert = "INSERT INTO user (name, age) VALUES "; 4 insert += new Array(items.length).fill("(?,?)").join(","); 5 let params = new Array(items.length * 2); 6 let i = 0; 7 for (const item of items) { 8 params[i] = item.name; 9 params[i + 1] = item.age; 10 i += 2; 11 } 12 return new Promise((res, rej) => { 13 this.connection.serialize(() => { 14 this.connection.run("BEGIN TRANSACTION"); 15 try { 16 this.connection.run(insert, params); 17 } catch (e) { 18 this.connection.run("ROLLBACK TRANSACTION"); 19 return rej(e); 20 } 21 this.connection.run("COMMIT"); 22 }); 23 return res(); 24 }); 25 } 
```

Extending our tests for `createFast`:

```
1describe("createFast", () => { 2 for (const a of amount) { 3 let d = new Array(a).fill(data); 4 test(`createFast-${a}`, async () => { 5 await DB.createFast(d); 6 }); 7 } 8}); 
```

|  | `create` | `createFast` | Improvement |
| --- | --- | --- | --- |
| 10 rows | 2ms | 1ms | 2x |
| 100 rows | 1ms | 1ms | / |
| 1,000 rows | 13ms | 3ms | 4.3x |
| 10,000 rows | 100ms | 22ms | 4.5x |
| 100,000 rows | 1089ms | 215ms | 5.1x |
| 1,000,000 rows | 11795ms | 1997ms | 5.9x |

### Info

The before benchmarks were done on the following system:

*   CPU: Intel(R) Core(TM) i5-6200U CPU @ 2.30GHz 4 cores
*   RAM: 8,0 GB DDR3 1600 MHz

The system was under heavy load while benchmarking, thus the recorded times are still pretty slow.

## Escaping from micro benchmarks into the real world [##](#escaping-from-micro-benchmarks-into-the-real-world)

Applying the results from the micro benchmarks to my real world projects database layer resulted in significant runtime improvements, up to 66x faster.

|  | `create` | `createFast` | Improvement |
| --- | --- | --- | --- |
| 10 rows | 7ms | 2ms | 3.5x |
| 100 rows | 48ms | 1ms | 48x |
| 1000 rows | 335ms | 9ms | 37.2x |
| 10000 rows | 2681ms | 80ms | 33.5x |
| 100000 rows | 25347ms | 390ms | 66x |