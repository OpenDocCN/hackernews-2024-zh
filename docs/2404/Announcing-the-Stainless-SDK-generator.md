<!--yml
category: 未分类
date: 2024-05-27 13:29:51
-->

# Announcing the Stainless SDK generator

> 来源：[https://www.stainlessapi.com/blog/announcing-the-stainless-sdk-generator](https://www.stainlessapi.com/blog/announcing-the-stainless-sdk-generator)

Stainless generates the official client libraries for OpenAI, Anthropic, Cloudflare, and more. Today, we’re making the Stainless SDK generator available to every developer with a REST API.

During our private beta, we’ve been able to help millions of developers integrate faster and more reliably with the latest features of some of the world’s most powerful and exciting APIs.

The Stainless SDK generator accepts an OpenAPI specification and uses it to produce quality SDKs in multiple programming languages. As your API evolves, our automated generator continuously pushes changes, ensuring that your SDKs remain up-to-date—even as you make [arbitrary custom edits](https://app.stainlessapi.com/docs/guides/patch-custom-code) to the generated code.

Here’s a quick, real-world example of the code we can do for you, and how you configure it with Stainless:
‍

```
 from cloudflare import Cloudflare

client = Cloudflare()
client.zones.create(account={"id": "xxxx"}, name="my zone", type="full") 

```

```
 import Cloudflare from "cloudflare";

async function main() {
  const cloudflare = new Cloudflare();
  const zone = await cloudflare.zones.create({
    account: { id: "xxx" },
    name: "example.com",
    type: "full",
  });
} 

```

```
 package main

import (
    "context"

    "github.com/cloudflare/cloudflare-go/v2"
    "github.com/cloudflare/cloudflare-go/v2/zones"
)

func main() {
    client: = cloudflare.NewClient()
    zone,
    err: = client.Zones.New(context.Background(), zones.ZoneNewParams {
        Account: cloudflare.F(zones.ZoneNewParamsAccount {
            ID: cloudflare.F("023e105f4ecef8ad9ca31a8372d0c353"),
        }),
        Name: cloudflare.F("example.com"),
        Type: cloudflare.F(zones.ZoneNewParamsTypeFull),
    })
} 

```

```
 resources:
  zones:
    methods:
      create: post /v1/zones 

```

 *See the source code for these endpoints in* [*TypeScript*](https://github.com/cloudflare/cloudflare-typescript/blob/54faa1bd35ae5f394e37e8ac7f75c36f738baff9/src/resources/zones/zones.ts#L27-L34)*,* [*Python*](https://github.com/cloudflare/cloudflare-python/blob/77cd2adc0ea0e48fb7419dfe6d48a7a5af78f35c/src/cloudflare/resources/zones/zones.py#L129-L177)*, and* [*Go*](https://github.com/cloudflare/cloudflare-go/blob/21580812b4a3fb3f215ea1f539c2c14e2fa123bb/zones/zone.go#L50-L61)*.*

‍

> “The decision to use Stainless has allowed us to move our focus from building the generation engine to instead building high-quality schemas to describe our services.
> 
> In the span of a few months, we have gone from inconsistent, manually maintained SDKs to automatically shipping over 1,000 endpoints across three language SDKs with hands-off updates.”
> 
> *Jacob Bednarz, API Platform Tech Lead, Cloudflare (*[*see blog post*](https://blog.cloudflare.com/lessons-from-building-an-automated-sdk-pipeline)*)*

‍

# **Backstory: scaling SDKs at Stripe**

From [the very first day that stripe.com existed on the internet](https://web.archive.org/web/20111213031731/https://stripe.com/), SDKs were a big part of the pitch to developers. Today, well over 90% of Stripe developers make well over 90% of requests to the Stripe API through the SDKs. As the front door to the API, the SDKs are how developers think about Stripe—to most people, it’s `stripe.charges.create()`, not `POST /v1/charges`.

Personally, I hadn’t appreciated this until I joined Stripe in 2017—but by then, the growing scope of the Stripe API had exceeded our capacity to build SDKs by hand sustainably. Making manual changes across 7 different programming languages whenever we shipped a new endpoint was toilsome and error-prone.

Not only that, TypeScript had eaten the world. Now, developers expect typeahead and documentation-on-hover directly in their text editor. Building SDKs that support comprehensive static types in a variety of languages was totally unthinkable without code generation.

The team had spent over a year exploring existing open-source code generation tools before concluding that none could meet Stripe’s quality standards. Most codegen tools either work in only one language or just use string templating, which leaves you constantly fiddling with issues like trailing commas in Go and invalid indentation in Python. We needed to build our own codegen tool that enabled us to “learn once, write everywhere” and easily produce clean, correct code across all our languages.

Over a weekend, I hacked together a surprising mashup of JSX and the internals of prettier to enable product developers to quickly template quality code that came well-formatted out of the box. Over the next several months, dozens of engineers helped convert the SDKs to codegen, matching our carefully handcrafted code byte for byte.

By later that year, I was pairing with a colleague to [produce the first official TypeScript definitions](https://twitter.com/stripe/status/1222944951853432832) for the Stripe API.

‍

# **Scaling SDKs for everybody**‍

After I left Stripe, engineers kept asking me how to build great SDKs for their API. I didn’t have a great answer—most companies don’t have several engineer-years lying around to build whole a suite of high-quality code generators spanning a range of popular languages.

In early 2022, I set out to bootstrap a company, and Lithic became our first customer, making Stainless ramen-profitable from day one. Their head of product had previously been the PM of SDKs at Plaid, where she’d seen firsthand both how valuable SDKs are and how hard it is to codegen decent ones with the openapi-generator.

Despite the allure of a small, bootstrapped company, I felt bad limiting its impact to the small number of clients I could handle by myself. What’s more, I also got asked how to keep OpenAPI specs up-to-date and valid, how to evolve API versions, how to design RESTful pagination, how to set up API Keys, and a million other problems my old team had already solved at Stripe.

Eventually it became clear that the world needed a comprehensive developer platform—from docs to request logs to rate-limiting—that could enable REST to live up to its potential.

Sequoia soon became our first investor, shortly followed by great angels like Cristina Cordova, Guillermo Rauch, Calvin French-Owen, and dozens more.

There was clearly a huge opportunity to advance the whole REST ecosystem and help a ton of people ship better APIs.

‍

# **Advancing the API ecosystem**

APIs are the dendrites of the internet. Literally all internet software connects through APIs—they make up the vast majority of internet traffic.

Today, the API ecosystem is deeply fragmented:

1.  GraphQL. Great for frontends, but not built for server-to-server interactions and hasn’t worked out well for public APIs.
2.  gRPC. Great for microservices, but doesn’t work for frontends and is unpopular for public APIs.
3.  REST. Works for frontends, microservices, and public APIs. It’s simple, flexible, and aligned with web standards—but also messy and hard to get right.

Engineering organizations should be able to use one API technology for everything—their frontends, their microservices, and their public API—and have a good experience everywhere.

At Stainless, rather than trying to fit GraphQL or gRPC into the square holes they weren’t designed for—or invent some new 15th standard—we are staunch believers that “REST done right” can deliver this vision.

We want to build great open-source standards and tooling that bring the benefits of GraphQL (types, field selection/expansion, standards) and gRPC (types, speed, versioning) to REST.

When Stainless REST is realized, you’ll be able to start building a full-stack application using our API layer and have at least as good of a frontend experience as you would have had with GraphQL. When you add a Go microservice, you’ll be able to interconnect with typed clients, efficient packets, and low latency. And then—uniquely—when your biggest customer asks you for an external API, you’ll be able to just say “yes” and change `internal(true)` to `internal(false)` instead of rewriting the whole thing.

Today, our SDK announcement tackles the most salient problem with REST: type safety.

Our next project is building out a development framework that enables users to ship quality, typesafe REST APIs from any TypeScript backend. With the upcoming Stainless API framework, you declare the shape and behavior of your API in declarative TypeScript code and get an OpenAPI specification, documentation, and typed frontend client without a build step.

We’re building the framework around REST API design conventions that support rich pagination, consistent errors, field inclusion and selection, and normalized caching on the frontend. These conventions, influenced by best practices at Stripe, can help your team achieve consistent, high-quality results without untold hours of bikeshedding.

# **Building Stainless**

> “The cool part about this is that you can define your API once and get client libraries in every language for free, whether you are an expert in those languages or not.
> 
> It’s rare that a startup will have people who know Python, Node, Ruby, Rust, Go, Java, etc, etc. But now they can market to all those developers at once.”
> 
> *Calvin French-Owen, co-founder, Segment*

Producing a good SDK is more involved than many developers may realize, especially when relying on code generation. The details matter, and it’s not just about pretty code—it’s about making the right choices and balancing some challenging tradeoffs between the characteristics of REST APIs and the idioms of the language at hand.

Here are a few generic examples:

*   How do you handle response enums in Java? The obvious approach can result in crashes when adding a new variant in the future.
*   If your API introduces a union type, how do you express that in Go, given that the language does not have a standard way to express union types?
*   If unexpected data comes back from the server—whether due to a beta feature, an edge case, or a bug—how do you expose that data to the user? Should the client library treat it like an error? Is there an idiomatic way to achieve this across every programming language? Finding a good solution requires carefully weighing conflicting type safety and runtime safety considerations.
*   Should you automatically retry on 429 or 503 errors? How quickly? What if the API is experiencing a production outage?
*   What should you call the method for `/v1/invoices/{id}/void` in a Java client library? (hint: it can’t be `void`).

Note that last problem can’t simply be decided by a machine—it requires context about the rest of the API, and must be decided by a human (even if an LLM can offer a first guess).

The `void` endpoint example is obviously an edge case, but the general question of what to name each method and type is as pernicious as it is pedestrian. If an SDK generator infers all names directly from the OpenAPI specification—particularly a specification generated from other sources—users may be confronted with nonsensical types like `AccountWrapperConfigurationUnionMember4` that raise questions about your company’s overall engineering quality.

If you ship an SDK without first addressing these issues, you risk locking yourself into design blunders that you may not be able to resolve later without breaking backwards compatibility in ways that are highly disruptive to users.

We shared the `void` example above because it is easy to understand, but there are a range of potential pitfalls that are even more subtle and abstruse that inevitably arise in non-trivial APIs. We can build tools to identify such issues, but deciding how to resolve them is often beyond the scope of what can be achieved with automation—even with AI. You need a human being with relevant context and sound judgement to assess the options and make an informed decision.

From experience, we knew that thoughtful SDK development is a lot more difficult than it seems—auditing every single type name in a typical, medium-sized API requires scanning through tens of thousands of lines of code.

To enable every developer to ship with the same level of care that we devote to our enterprise clients, we created an SDK Studio that highlights potential problems and makes it easy to quickly scan through all the things you may want to review before shipping a v1:

The Stainless SDK Studio

To start using the Stainless SDK generator, all you need is an OpenAPI specification.

Within a few minutes, you’ll get alpha SDKs you can publish to package managers—and after a bit of polishing, something you’re proud to release as v1.0.0.

To get started, check out [our documentation](https://app.stainlessapi.com/docs/) or [connect your GitHub account](https://app.stainlessapi.com/login).