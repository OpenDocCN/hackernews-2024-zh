<!--yml

category: 未分类

date: 2024-05-27 15:20:28

-->

# 给 TypeScript 中的对象 ID 添加类型安全 - Kravchyk 的

> 来源：[`www.kravchyk.com/adding-type-safety-to-object-ids-typescript/`](https://www.kravchyk.com/adding-type-safety-to-object-ids-typescript/)

我正在为函数的选项对象创建类型，其中的一个选项属性将接受项的 ID（基于字符串的 UUID）或特殊值，例如 `'currentNode'`。

最初我会想出类似这样的东西：

```
`interface InsertOptions {   target: string | 'currentNode' }   function insert(options: InsertOptions): void {   }   insert({ target: 'activeNode' });` Code language: TypeScript (typescript)
```

[试一试](https://www.typescriptlang.org/play?#code/JYOwLgpgTgZghgYwgAgJIgM7TAeQA5jAD2myA3gFDLID0Nam2yoYRyYAFisJALbIAjAJ7MAJsiJR2XZIkIA3bnyrs4UAOYQwALmQYwUUOuQAfZAHIEAVyhQI4AHJFREcxQC+FCjCsgEhEmZGKDAACiICYkxddCwQ-ADMAEpdeSJgcUpqOg8vUDiwslUNLV1zOWBFJxdzZHckgG5aegAFOAwMI1lkBAAbCDVekV8WexdxCMTkeTheqxQAd2Be3uQQIjBkOwwrXs3QWRBkaChJIA)

这种方法有 2 个问题：

+   currentNode 没有自动类型提示

+   由于 `curentNode` 也只是一个 `string`，所以对特殊选项没有类型检查。

`type NodeId = string` 几乎没什么用，因为它只是字符串的别名，在类型检查的角度来看和字符串是一样的。

解决这个问题的一种方法是将特殊选项包装在一个对象中：

```
`interface InsertOptions {
  target: string | { target: 'activeNode'}
}

function insert(options: InsertOptions): void {

}

insert({ target: 'KQER3XVBdD3u' }); 
insert({ target: { target: 'activeNode' } }); 
insert({ target: { target: 'currentNode' } });` Code language: TypeScript (typescript)
```

[试一试](https://www.typescriptlang.org/play?#code/JYOwLgpgTgZghgYwgAgJIgM7TAeQA5jAD2myA3gFDLJhxQDmEYAXMhmFKPcgD7k11GLZAHJEhAG4QAckQAmEEQF8KKijACuIBIRLJQWKGAAURAsUyt0h3OZIYAlKwlFgc8lWQB6L6ooUDbGMyAQYmVhEAaQBFAFEAJQBmAA0ANQAhOQARRI0RZCUHAG5vL2RUuAAbNwDMIJDaMOEGwXDRcWApWQV8pQLi0vKqmsCjYNChVhamiIQNKCgIcG7FAv6Sn2RYhaIoIA)

这会生效，类型检查和自动扩展建议都会生效，但看起来一点也不好看…

幸运的是，TypeScript 自 4.1 版本以来就有了 [模板文字类型](https://www.typescriptlang.org/docs/handbook/release-notes/typescript-4-1.html)。可以使用它们创建只有 ID 才会有的类型，比如：

```
`type NodeId = `node_${string}`` Code language: JavaScript (javascript)
```

所以现在我们可以这样做：

```
`interface InsertOptions {
  target: NodeId | 'activeNode'
}

function insert(options: InsertOptions): void {

}

insert({ target: 'activeNode' }); 
insert({ target: 'node_KQER3XVBdD3u' }); 
insert({ target: 'currentNode' });` Code language: TypeScript (typescript)
```

[试一试](https://www.typescriptlang.org/play?#code/C4TwDgpgBAcg9gEwgSQVAvFABgO0RAfQBIBvAZ2ACcBLHAcwF8sAoZ24CSgMwEMBjaMhxlOwAPJhg1OMKglmUKMB6U6EYAC5Y+VFAA+UAOT8pANwjwkh5g1ZcArjj5SZUWiMrAAFHEnThWkIe4n4yZACUWqZw1GjyigD0CTas7qJeJEoqappGJtTmlhCGUAzhANxQSVAAajwANrFswumZyqrqWoZ4SAQA0gCKAKIASgDMABo1AEIIACJj9iVlldV1jQjNwRlZHbmGfPaUlBA4wEXLFVUJUEPHcJRAA)

现在目标类型不再是字符串，TS 可以区分 ID 和特殊选项之间的区别，这意味着特殊选项会经过类型检查，并会显示在自动类型建议中。

这不仅解决了提到的 2 个问题，还在 TypeScript 中引入了 ID 的类型安全。每种类型的项都可以分配不同类型的 ID，例如

```
`type UserId = `user_${string}`;
type GroupId = `group_${string}`;

function updateUser(id: UserId): void {

}

declare const id: GroupId;

updateUser(id);` Code language: JavaScript (javascript)
```

[试一试](https://www.typescriptlang.org/play?#code/C4TwDgpgBAqgzhATgSQCZQLxQAYFcGID6AJAN5zCICWAdgOYC+2A3AFCiRQDiiA9rmDSYcdPgJLlKtRi1asAZrhoBjYFV40oA1AENgEeEgAUVVAC5YBNAEoLAN16mopVlCgB6d6wZzPUZAC2OnS00HC8ARAA7gAWSNC0UMBxUMq8qNAg-FAxOnbQOppOdjrUOgBGADbQyXpQVHBQOtxigqisGcqVpdBpNBT15i38bWys2noGBCao1swe7lAAooh8iBYAgoh0uJE0wFC88kng0ADk2KIjEhTU9Exn9Y00vAc6cHBUdDQV1Um8UDApR0kX0iEOxw45zwBBuUnu2DOQA)

我已经使用了一年了，实现类型安全的 ID 已经几次防止我混淆实体类型。

这样做的好处不仅仅体现在编译时，而且在运行时也同样如此。你可以通过检查其 ID 前缀来检测对象的类型，而且非常方便的是，每当对象在日志中出现时，你只需看一眼 ID 就立刻知道它是什么。
