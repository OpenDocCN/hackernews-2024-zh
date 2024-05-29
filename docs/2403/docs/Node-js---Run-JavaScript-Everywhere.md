<!--yml
category: 未分类
date: 2024-05-29 12:29:17
-->

# Node.js — Run JavaScript Everywhere

> 来源：[https://nodejs.org/en](https://nodejs.org/en)

```
// server.mjs
import { createServer } from 'node:http';
 const server = createServer((req, res) => {
 res.writeHead(200, { 'Content-Type': 'text/plain' });
 res.end('Hello World!\n');
});
 // starts a simple http server locally on port 3000
server.listen(3000, '127.0.0.1', () => {
 console.log('Listening on 127.0.0.1:3000');
});
 // run with `node server.mjs` 
```