<!--yml
category: 未分类
date: 2024-05-27 15:20:28
-->

# Adding type safety to object IDs in TypeScript - Kravchyk's

> 来源：[https://www.kravchyk.com/adding-type-safety-to-object-ids-typescript/](https://www.kravchyk.com/adding-type-safety-to-object-ids-typescript/)

I was creating a type for an options object of a function, one of the options properties would accept an ID of the item (string based UUID) or a special value, i.e. `'currentNode'`.

Initially I would come up with something like this:

```
`interface InsertOptions {   target: string | 'currentNode' }   function insert(options: InsertOptions): void {   }   insert({ target: 'activeNode' });` Code language: TypeScript (typescript)
```

[Try It](https://www.typescriptlang.org/play?#code/JYOwLgpgTgZghgYwgAgJIgM7TAeQA5jAD2myA3gFDLID0Nam2yoYRyYAFisJALbIAjAJ7MAJsiJR2XZIkIA3bnyrs4UAOYQwALmQYwUUOuQAfZAHIEAVyhQI4AHJFREcxQC+FCjCsgEhEmZGKDAACiICYkxddCwQ-ADMAEpdeSJgcUpqOg8vUDiwslUNLV1zOWBFJxdzZHckgG5aegAFOAwMI1lkBAAbCDVekV8WexdxCMTkeTheqxQAd2Be3uQQIjBkOwwrXs3QWRBkaChJIA)

There are 2 problems with this approach:

*   No autotype hints for currentNode
*   No type checking for the special option since `curentNode` is a also just a `string`.

`type NodeId = string` would hardly help since it’s just an alias for string and is the same from type checking perspective.

One way to solve this would be to wrap the special option in an object:

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

[Try It](https://www.typescriptlang.org/play?#code/JYOwLgpgTgZghgYwgAgJIgM7TAeQA5jAD2myA3gFDLJhxQDmEYAXMhmFKPcgD7k11GLZAHJEhAG4QAckQAmEEQF8KKijACuIBIRLJQWKGAAURAsUyt0h3OZIYAlKwlFgc8lWQB6L6ooUDbGMyAQYmVhEAaQBFAFEAJQBmAA0ANQAhOQARRI0RZCUHAG5vL2RUuAAbNwDMIJDaMOEGwXDRcWApWQV8pQLi0vKqmsCjYNChVhamiIQNKCgIcG7FAv6Sn2RYhaIoIA)

This is going to work, both type checking and autoexpand suggestion will work, but it does not look nice at all…

Fortunately TypeScript has [template literal types](https://www.typescriptlang.org/docs/handbook/release-notes/typescript-4-1.html) since version 4.1\. It’s possible to use them to create a type that only an ID would have, like:

```
`type NodeId = `node_${string}`` Code language: JavaScript (javascript)
```

So now we can do:

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

[Try It](https://www.typescriptlang.org/play?#code/C4TwDgpgBAcg9gEwgSQVAvFABgO0RAfQBIBvAZ2ACcBLHAcwF8sAoZ24CSgMwEMBjaMhxlOwAPJhg1OMKglmUKMB6U6EYAC5Y+VFAA+UAOT8pANwjwkh5g1ZcArjj5SZUWiMrAAFHEnThWkIe4n4yZACUWqZw1GjyigD0CTas7qJeJEoqappGJtTmlhCGUAzhANxQSVAAajwANrFswumZyqrqWoZ4SAQA0gCKAKIASgDMABo1AEIIACJj9iVlldV1jQjNwRlZHbmGfPaUlBA4wEXLFVUJUEPHcJRAA)

Now the target type is not string anymore and TS can see the difference between the ID and the special options, meaning that the special options are type checked and will show in autotype suggestions.

This does not only solve the 2 problems mentioned, but also introduces type safety of IDs in TypeScript. Each type of item can have a different type of ID assigned, I.e.

```
`type UserId = `user_${string}`;
type GroupId = `group_${string}`;

function updateUser(id: UserId): void {

}

declare const id: GroupId;

updateUser(id);` Code language: JavaScript (javascript)
```

[Try It](https://www.typescriptlang.org/play?#code/C4TwDgpgBAqgzhATgSQCZQLxQAYFcGID6AJAN5zCICWAdgOYC+2A3AFCiRQDiiA9rmDSYcdPgJLlKtRi1asAZrhoBjYFV40oA1AENgEeEgAUVVAC5YBNAEoLAN16mopVlCgB6d6wZzPUZAC2OnS00HC8ARAA7gAWSNC0UMBxUMq8qNAg-FAxOnbQOppOdjrUOgBGADbQyXpQVHBQOtxigqisGcqVpdBpNBT15i38bWys2noGBCao1swe7lAAooh8iBYAgoh0uJE0wFC88kng0ADk2KIjEhTU9Exn9Y00vAc6cHBUdDQV1Um8UDApR0kX0iEOxw45zwBBuUnu2DOQA)

I have been using it for a year now and implementing type safe IDs has already prevented me a few times already from mixing entity types.

The benefit of this is not just at compile time, but also at runtime. You can use this to detect the type of object by checking its ID prefix and it’s also very convenient that whenever the object pops up in the logs, you immediately know what it is just by looking at the ID.