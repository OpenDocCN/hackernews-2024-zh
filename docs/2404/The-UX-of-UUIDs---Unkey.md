<!--yml

类别：未分类

日期：2024-05-27 13:06:59

-->

# UUID 的用户体验 | Unkey

> 来源：[https://unkey.dev/blog/uuid-ux](https://unkey.dev/blog/uuid-ux)

TLDR：请不要这样做：

```
 1https://company.com/resource/c6b10dd3-1dcf-416c-8ed8-ae561807fcaf
```

* * *

## [基线：确保全局唯一性](https://unkey.dev/blog/uuid-ux#the-baseline-ensuring-global-uniqueness)

在系统中，唯一标识符对于区分个体实体至关重要。它们为确保每个项目、用户或数据片段具有唯一身份提供了可靠的方式。通过保持唯一性，应用程序可以有效管理和组织信息，实现高效运作并促进数据完整性。

让我们不要假装自己是谷歌或 AWS，他们在这方面有特殊需求。对我们来说，128 位安全生成的 UUID 已经足够了。有许多库可以生成 UUID，或者你可以回退到你选择的语言的标准库。在本文中，我将使用 TypeScript 的示例，但其基本思想适用于任何语言。

```
 1const id = crypto.randomUUID();
```

```
 2
```

到这里停下也是一个选择，但让我们借此机会通过小而有效的迭代改进用户体验：

1.  让它们易于复制

1.  前缀

1.  更高效的编码

1.  改变长度

### [复制 UUID 是很烦人的](https://unkey.dev/blog/uuid-ux#copying-uuids-is-annoying)

现在尝试双击复制这个 UUID：

```
 1c6b10dd3-1dcf-416c-8ed8-ae561807fcaf
```

如果你幸运，你可能得到了整个 UUID，但对于大多数人来说，他们只得到了一个部分。提升唯一标识符的可用性的一种方法是使它们易于复制。通过去除 UUID 中的连字符，用户可以简单地双击标识符即可复制。通过消除手动选择和复制粘贴的需求，这一小改变可以极大地提升用户在处理标识符时的体验。

在所有语言中，去除连字符可能是微不足道的，这里是你可以在 js/ts 中实现的方法：

```
 1const id = crypto.randomUUID().replace(/-/g,  "");
```

```
 2
```

现在试着复制它，会更加友好！

### [前缀](https://unkey.dev/blog/uuid-ux#prefixing)

你曾经在开发环境中意外使用过生产 API 密钥吗？我曾经有过，这并不好玩。我们可以通过为系统中的不同环境或资源添加有意义的前缀来帮助用户区分它们。例如，Stripe 在生产环境的秘密密钥中使用前缀`sk_live_`，或者在客户标识符中使用`cus_`。通过引入这些前缀，我们可以确保清晰度，并减少混淆的可能性，特别是在存在多个环境的复杂系统中。

```
 1const id =  `hello_${crypto.randomUUID().replace(/-/g,  "")}`;
```

```
 2
```

命名前缀就像命名变量一样是一门艺术。你希望描述清楚，但又尽可能简短。我将在下文分享我们的方式。

### [在 base58 编码中编码](https://unkey.dev/blog/uuid-ux#encoding-in-base58)

与其使用十六进制表示标识符，我们也可以考虑更高效地编码它们，例如 base58。Base58 编码使用更大的字符集，并避免了模棱两可的字符，例如大写的`I`和小写的`l`，从而生成更短的标识符字符串而不影响可读性。

举个例子，一个长度为8个字符的base58字符串，可以存储大约30000倍于一个8字符十六进制字符串的状态。而长度为16的base58字符串则可以存储889.054.070倍于其组合数。

你可以使用你的语言的标准库来完成这个任务，但你也可以使用像[nanoid](https://github.com/ai/nanoid)这样的库，这种库几乎支持所有语言。

```
 1import  { customAlphabet }  from  "nanoid";
```

```
 2export  const nanoid =  customAlphabet(
```

```
 3  "123456789ABCDEFGHJKLMNPQRSTUVWXYZabcdefghijkmnopqrstuvwxyz",
```

```
 4);
```

```
 5  
```

```
 6const id =  `prefix_${nanoid(22)}`;
```

```
 7
```

我们生成了一个长度为22个字符的ID，在编码上可以比UUID多约100倍，同时又比UUID短10个字符。

|  | 字符 | 长度 | 总状态数 |
| --- | --- | --- | --- |
| UUID | 16 | 32 | 2^122 = 5.3e+36 |
| Base58 | 58 | 22 | 58^22 = 6.2e+38 |

*状态越多，碰撞抵抗能力越高，因为生成相同ID需要更多的尝试次数（平均情况下，如果你的算法是真正随机的话）*

### [](#changing-the-entropy)改变熵

并非所有的标识符都需要高级别的碰撞抵抗能力。在某些情况下，较短的标识符可能已经足够，这取决于应用程序的具体要求。通过降低标识符的熵，我们可以生成更短的ID，同时仍然保持可接受的唯一性水平。

缩短ID长度可能很好，但是你需要小心并确保你的系统免受ID碰撞的影响。幸运的是，在数据库层面很容易实现这一点。在我们的MySQL数据库中，我们大多数情况下使用ID作为主键，数据库保护我们免受碰撞的影响。如果一个ID已经存在，我们只需生成一个新的ID并再试一次。如果我们的碰撞率显著增加，我们可以简单地增加所有未来ID的长度，问题就解决了。

| 长度 | 例子 | 总状态数 |
| --- | --- | --- |
| nanoid(8) | re6ZkUUV | 1.3e+14 |
| nanoid(12) | pfpPYdZGbZvw | 1.4e+21 |
| nanoid(16) | sFDUZScHfZTfkLwk | 1.6e+28 |
| nanoid(24) | u7vzXJL9cGqUeabGPAZ5XUJ6 | 2.1e+42 |
| nanoid(32) | qkvPDeH6JyAsRhaZ3X4ZLDPSLFP7MnJz | 2.7e+56 |

## [](#conclusion)结论

通过实施这些改进，我们可以增强应用程序中唯一标识符的可用性和效率。这将为用户和开发人员提供更好的体验，因为他们在系统中与各种实体进行交互和管理时，可以更轻松地复制标识符，区分不同的环境，或实现更短和更可读的标识符字符串。这些策略可以为用户友好和强大的识别系统做出贡献。

## [](#ids-and-keys-at-unkey)Unkey的ID和键

最后，我想在这里分享我们的实现方式以及我们在我们的[codebase](https://github.com/unkeyed/unkey/blob/main/internal/id/src/index.ts)中如何使用它。我们使用一个简单的函数，它接受一个类型化的前缀，然后为我们生成ID。这样我们可以确保对于同一类型的ID，我们总是使用相同的前缀。当你的系统中有多种类型的ID时，这特别有用。

```
 1import  { customAlphabet }  from  "nanoid";
```

```
 2export  const nanoid =  customAlphabet(
```

```
 3  "123456789ABCDEFGHJKLMNPQRSTUVWXYZabcdefghijkmnopqrstuvwxyz",
```

```
 4);
```

```
 5  
```

```
 6const prefixes =  {
```

```
 7 key:  "key",
```

```
 8 api:  "api",
```

```
 9 policy:  "pol",
```

```
10 request:  "req",
```

```
11 workspace:  "ws",
```

```
12 keyAuth:  "key_auth",  
```

```
13 vercelBinding:  "vb",
```

```
14 test:  "test",  
```

```
15}  as  const;
```

```
16  
```

```
17export  function  newId(prefix:  keyof  typeof prefixes):  string  {
```

```
18  return  [prefixes[prefix],  nanoid(16)].join("_");
```

```
19}
```

在我们的代码库中使用时，我们可以确保始终为正确类型的ID使用正确的前缀。

```
 1import  { newId }  from  "@unkey/id";
```

```
 2  
```

```
 3const id =  newId("workspace");
```

```
 4
```

```
 5  
```

```
 6const id =  newId("keyy");
```

```
 7
```

* * *

我们在这里主要讨论了标识符，但API密钥实际上也只是一个标识符。它只是一种用于验证请求的特殊类型的标识符。我们对API密钥使用与标识符相同的策略。您可以添加前缀来让用户知道他们正在查看的密钥类型，并且可以在合理范围内指定密钥的长度。API密钥的碰撞比ID更为严重，因此我们强制执行安全限制。

在API密钥中添加公司标识前缀是非常常见的做法。例如，[Resend](https://resend.com) 使用 `re_`，而 [OpenStatus](https://openstatus.dev) 使用 `os_` 前缀。这样一来，用户可以快速识别密钥的用途。

```
 1const key =  await unkey.key.create({
```

```
 2 apiId:  "api_dzeBEZDwJ18WyD7b",
```

```
 3 prefix:  "blog",
```

```
 4 byteLength:  16,
```

```
 5  
```

```
 6});
```

```
 7  
```

```
 8
```

```
 9
```
