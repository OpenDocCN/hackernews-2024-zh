<!--yml
category: 未分类
date: 2024-05-29 12:42:57
-->

# Write OpenAPI with TypeSpec

> 来源：[https://blog.trl.sn/blog/typespec-for-openapi/](https://blog.trl.sn/blog/typespec-for-openapi/)

<main id="skip">

# Write OpenAPI with TypeSpec

I've spent the last few years at Microsoft working on an API definition language called [TypeSpec](https://typespec.io). It's essentially a super flexible protocol-agnostic DSL for describing API shapes. You can try it in your browser at the [TypeSpec playground](https://typespec.io/playground). Many things about it are exciting, but I want to talk about one thing in particular: why TypeSpec is the best way to write OpenAPI.

## OpenAPI: the good and the not-so-good

OpenAPI is pretty great at describing the low level details of how an HTTP API works. It allows software to understand the shape of an API which in turn enables myriad useful things, like generating clients and documentation, configuring API gateways, or generating test cases. The fact that OpenAPI is the most widely used language to describe HTTP APIs is a testament to these strengths.

However, after working with OpenAPI inside Azure, it is also clear to me that OpenAPI suffers a few critical weaknesses. Humans don't find OpenAPI particularly pleasant to author and review, whether in JSON or YAML. The code generation from OpenAPI is often not stellar despite heroic efforts from many in the community. It also struggles when building APIs at scale where its verbosity and lack of reusable components require significant investment in API reviews and governance process.

I believe API-first development principles are great in theory, but in practice API-first with OpenAPI faces significant headwinds. Developers are likely to opt for code-first approaches that generate OpenAPI as a build artifact. I believe TypeSpec changes the game. Let's see how!

## Enter TypeSpec

TypeSpec is designed to be protocol agonstic, but the team has invested heavily in making great OpenAPI 3.0 emit. You can use TypeSpec to define most OpenAPI documents by using its `http` standard library which provides various types and decorators to add http-specific metadata like headers, query strings, or status codes. Writing OpenAPI in TypeSpec has many advantages over using OpenAPI directly. What I'll cover today is that TypeSpecs are smaller, more readable, and can use API components.

### Terse & expressive

TypeSpec's syntax and semantics borrow heavily from TypeScript, which is one of the best languages for describing REST API shapes. TypeSpec is able to describe complex model shapes and REST endpoints with substantially less typing than OpenAPI, and the end result is much more readable. I've even heard from folks who successfully use TypeSpec syntax to discuss API shapes with non-technical stakeholders!

Let's look at an example of what I mean. You can view the resulting OpenAPI [in the playground](https://typespec.io/playground?c=aW1wb3J0ICJAdHlwZXNwZWMvaHR0cCI7DQp1c2luZyBIdHRwOw0KDQovKiogQSBjaGFyYWN0ZXIgaW4gYSBmYW50YXN5IFJQRy1zdHlsZSBnYW1lICovDQptb2RlbCBDyTB7DQogIG5hbWU6IHN0cmluZzvEEWlkOiBzYWZlaW50xRBzdGF0dXM6ICJBbGl2ZSIgfCAiRGVhZOQAkCAgY2xhc3M6IEPEBzsNCn3kAJdlbnVtxhIgeyB3YXJyaW9yOyB3aXphcmQ7IMUjQG1pblZhbHVlKDEpxA5heMYOMjApDQpzY2FsYXLFeMUWIGV4dGVuZHMgdWludDjlAPrmAMpTdGF0c%2BYAxnN0cmVuZ3Ro5ADKxzfmAM1udGVsbGlnZW5j5QDmzBxkxFhyaXR5zhnlAKFvcCBnZXTpASsoQHBhdGjsARspOuoBSTs%3D&e=%40typespec%2Fopenapi3&options=%7B%7D).

```
import "@typespec/http";
using Http;
 /** A character in a fantasy RPG-style game */
model Character {
 name: string;
 id: safeint;
 status: "Alive" | "Dead";
 class: Class;
}
 enum Class { warrior; wizard; }
 @minValue(1)
@maxValue(20)
scalar statValue extends uint8;
 model Stats {
 strength: statValue;
 intelligence: statValue;
 dexterity: statValue;
}
 op getCharacter(@path id: safeint): Character;
```

A partial TypeSpec API definition for a fantasy RPG

This TypeSpec ultimately compiles to OpenAPI that is almost 3x the number of lines, and 3.5x as many bytes. The TypeSpec code is fairly easy to read and follow—certainly familiar to TypeScript developers, but also obvious enough that anyone can get the gist.

TypeSpec unions in particular demonstrate how TypeSpec's syntax significantly improves readability. Unions provide a single terse syntax that compile to OpenAPI's `oneOf`, `anyOf`, and `enum` constructs. Consider the following TypeSpec and OpenAPI:

```
model Character {
 // ... snip other fields ...
 status: "Alive" | "Dead";
}
```

A TypeSpec union of literal types

```
status:
 type: string
 enum:
 - Alive
 - Dead
```

OpenAPI output for a union of literal types

The compiler determined that this was a union of literal types and so the right choice was to emit it as an enum. But if we have a union of object types, it compiles into an anyOf.

```
model Character {
 // ... snip other fields ...
 items: (Weapon | Armor | GenericItem)[]
}
```

A TypeSpec union of objects

```
items:
 type: array
 items:
 anyOf:
 - $ref: "#/components/schemas/Weapon"
 - $ref: "#/components/schemas/Armor"
 - $ref: "#/components/schemas/GenericItem"
```

OpenAPI output for a union of objects

Overall, TypeSpec's various language features compile to OpenAPI that is generally longer and more complex, both in terms of syntax and semantics.

### Composable & modular

The ability to encapsulate API patterns into reusable components was one of the main reasons why we started working on TypeSpec in the first place. At scale, consistency across APIs and conformance with API guidelines becomes quite costly. Monumental effort is required from both API authors and API reviewers to ensure quality.

TypeSpec's API components make consistency easy. API designers can encapsulate blessed API patterns into components that API authors import and use. In this way, API designers are assured that the API is correct simply by virtue of using the component, and API authors don't need to implement complex high-level patterns in JSON.

A common example is pagination. Using TypeSpec, we can define a template that defines the shape of our page, and use that for every paged endpoint.

```
model Page<T> {
 items: T[];
 size: int32;
 nextLink: url;
 prevLink: url;
}
 @route("/characters")
op listCharacters(): Page<Character>;
 @route("/items")
op listItems(): Page<Item>;
```

A TypeSpec for paginated endpoints

Another common use for templates is defining your standard error shapes. For example, if every endpoint might return a `403` error, we can express that with a template:

```
alias WithStandardErrors<T> = T | ForbiddenResponse;
 @route("/characters")
op listCharacters(): WithStandardErrors<Page<Character>>;
 @route("/items")
op listItems(): WithStandardErrors<Page<Item>>;
```

A TypeSpec for standard response shapes

In Azure, we have dozens of these templates, which encapsulate the API patterns we use across Azure's API surface area. We package these templates into modules that are imported by service owners when they're writing their API spec. They don't need to know that a "long-running operation" uses a particular HTTP verb with a payload of a specific shape, they just provide the long-running operation template a couple parameters specific to their API and they're done. API authors are happy, reviewers are happy.

## Wrapping up

This just scratches the surface of what it's possible to do with TypeSpec. There are many other features that combine to make TypeSpec extremely productive to use, producing OpenAPI documents that can be well over 10x the size. When writing APIs is this productive, I think API-first starts to look not only viable, but attractive.

In later posts, I'll be sharing some more details about fun things TypeSpec can do. Topics may include its support for other protocols (e.g. Protobuf, JSON Schema), TypeSpec's extensibility model and how you can add your own decorators or emit your own output using TypeScript, how TypeSpec enables higher quality code generation, and future directions around pagination and support for streaming APIs. If you're interested in a particular topic, feel free to say so on [twitter](https://twitter.com/bterlson).

</main>