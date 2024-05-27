<!--yml
category: 未分类
date: 2024-05-27 13:06:59
-->

# The UX of UUIDs | Unkey

> 来源：[https://unkey.dev/blog/uuid-ux](https://unkey.dev/blog/uuid-ux)

TLDR: Please don't do this:

```
 1https://company.com/resource/c6b10dd3-1dcf-416c-8ed8-ae561807fcaf
```

* * *

## [](#the-baseline-ensuring-global-uniqueness)The baseline: Ensuring global uniqueness

Unique identifiers are essential for distinguishing individual entities within a system. They provide a reliable way to ensure that each item, user, or piece of data has a unique identity. By maintaining uniqueness, applications can effectively manage and organize information, enabling efficient operations and facilitating data integrity.

Let’s not pretend like we are Google or AWS who have special needs around this. Any securely generated UUID with 128 bits is more than enough for us. There are lots of libraries that generate one, or you could fall back to the standard library of your language of choice. In this blog, I'll be using Typescript examples, but the underlying ideas apply to any language.

```
 1const id = crypto.randomUUID();
```

```
 2
```

Stopping here is an option, but let's take the opportunity to enhance the user experience with small yet effective iterative changes:

1.  Make them easy to copy
2.  Prefixing
3.  More efficient encoding
4.  Changing the length

### [](#copying-uuids-is-annoying)Copying UUIDs is annoying

Try copying this UUID by double-clicking on it:

```
 1c6b10dd3-1dcf-416c-8ed8-ae561807fcaf
```

If you're lucky, you got the entire UUID but for most people, they got a single section. One way to enhance the usability of unique identifiers is by making them easily copyable. This can be achieved by removing the hyphens from the UUIDs, allowing users to simply double-click on the identifier to copy it. By eliminating the need for manual selection and copy-pasting, this small change can greatly improve the user experience when working with identifiers.

Removing the hyphens is probably trivial in all languages, here’s how you can do it in js/ts:

```
 1const id = crypto.randomUUID().replace(/-/g,  "");
```

```
 2
```

Try copying it now, it’s much nicer!

### [](#prefixing)Prefixing

Have you ever accidentally used a production API key in a development environment? I have, and it’s not fun. We can help the user differentiate between different environments or resources within the system by adding a meaningful prefix. For example, Stripe uses prefixes like `sk_live_` for production environment secret keys or `cus_` for customer identifiers. By incorporating such prefixes, we can ensure clarity and reduce the chances of confusion, especially in complex systems where multiple environments coexist.

```
 1const id =  `hello_${crypto.randomUUID().replace(/-/g,  "")}`;
```

```
 2
```

Naming prefixes is an art just like naming variables. You want to be descriptive but be as short as possible. I'll share ours further down.

### [](#encoding-in-base58)Encoding in base58

Instead of using a hexadecimal representation for identifiers, we can also consider encoding them more efficiently, such as base58\. Base58 encoding uses a larger character set and avoids ambiguous characters, such as upper case `I` and lower case `l` resulting in shorter identifier strings without compromising readability.

As an example, an 8-character long base58 string, can store roughly 30.000 times as many states as an 8-char hex string. And at 16 chars, the base58 string can store 889.054.070 as many combinations.

You can probably still do this with the standard library of your language but you could also use a library like [nanoid](https://github.com/ai/nanoid) which is available for most languages.

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

We generated a 22 character long ID here, which can encode ~100x as many states as a UUID while being 10 characters shorter.

|  | Characters | Length | Total States |
| --- | --- | --- | --- |
| UUID | 16 | 32 | 2^122 = 5.3e+36 |
| Base58 | 58 | 22 | 58^22 = 6.2e+38 |

*The more states, the higher your collision resistance is because it takes more generations to generate the same ID twice (on average and if your algorithm is truly random)*

### [](#changing-the-entropy)Changing the entropy

Not all identifiers need to have a high level of collision resistance. In some cases, shorter identifiers can be sufficient, depending on the specific requirements of the application. By reducing the entropy of the identifiers, we can generate shorter IDs while still maintaining an acceptable level of uniqueness.

Reducing the length of your IDs can be nice, but you need to be careful and ensure your system is protected against ID collissions. Fortunately, this is pretty easy to do in your database layer. In our MySQL database we use IDs mostly as primary key and the database protects us from collisions. In case an ID exists already, we just generate a new one and try again. If our collision rate would go up significantly, we could simply increase the length of all future IDs and we’d be fine.

| Length | Example | Total States |
| --- | --- | --- |
| nanoid(8) | re6ZkUUV | 1.3e+14 |
| nanoid(12) | pfpPYdZGbZvw | 1.4e+21 |
| nanoid(16) | sFDUZScHfZTfkLwk | 1.6e+28 |
| nanoid(24) | u7vzXJL9cGqUeabGPAZ5XUJ6 | 2.1e+42 |
| nanoid(32) | qkvPDeH6JyAsRhaZ3X4ZLDPSLFP7MnJz | 2.7e+56 |

## [](#conclusion)Conclusion

By implementing these improvements, we can enhance the usability and efficiency of unique identifiers in our applications. This will provide a better experience for both users and developers, as they interact with and manage various entities within the system. Whether it's copying identifiers with ease, differentiating between different environments, or achieving shorter and more readable identifier strings, these strategies can contribute to a more user-friendly and robust identification system.

## [](#ids-and-keys-at-unkey)IDs and keys at Unkey

Lastly, I'd like to share our implementation here and how we use it in our [codebase](https://github.com/unkeyed/unkey/blob/main/internal/id/src/index.ts). We use a simple function that takes a typed prefix and then generates the ID for us. This way we can ensure that we always use the same prefix for the same type of ID. This is especially useful when you have multiple types of IDs in your system.

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

And when we use it in our codebase, we can ensure that we always use the correct prefix for the correct type of id.

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

I've been mostly talking about identifiers here, but an api key really is just an identifier too. It's just a special kind of identifier that is used to authenticate requests. We use the same strategies for our api keys as we do for our identifiers. You can add a prefix to let your users know what kind of key they are looking at and you can specify the length of the key within reason. Colissions for API keys are much more serious than ids, so we enforce secure limits.

It's quite common to prefix your API keys with something that identifies your company. For example [Resend](https://resend.com) are using `re_` and [OpenStatus](https://openstatus.dev) are using `os_` prefixes. This allows your users to quickly identify the key and know what it's used for.

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