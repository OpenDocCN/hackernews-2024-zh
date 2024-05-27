<!--yml
category: 未分类
date: 2024-05-27 13:39:13
-->

# A brief history of web development. And why your framework doesn't matter.

> 来源：[https://gebna.gg/blog/brief-history-of-web-development](https://gebna.gg/blog/brief-history-of-web-development)

# A brief history of web development. And why your framework doesn't matter.

#### 2024-04-25

History is very important. It helps keep our perspectives in check. This is how I remember the crazy journey that web development has gone through in the past few years. This is my telling of it. So I will omit events. I may mess up the timeline. This whole article may not be of any use to you. But for me, putting all of it into writing helped me appreciate how much things stay the same no matter how much they change. So if you’re curious and undeterred, allow me to start with …

### The good ol’ days

The simplest architecture to build a website would be to have one server to handle everything. That means that there’s no frontend router, no hydration, no AJAX, and definitely no cache/store like [Apollo](https://www.apollographql.com/) or [Redux](https://redux.js.org/). It’s very simple, the user requests an HTML page, the user gets an HTML page back. Oh and mutations will all be handled by native HTML forms. This used to be how everything worked. Not much JavaScript was written. Just whatever language you picked for backend and HTML/CSS. This worked because the web was made to be a decentralized space for sharing information. The websites made in those days were very static in nature and very little mutations happened so it was fine for the page to refresh after each form submission. It was also fine to load an entirely new HTML document every time the user navigated to another page.

But then the web started to change. Websites started becoming more and more like apps. More functionality was needed on the client side. So more JavaScript was written. Browser APIs were not enough so [jQuery](https://jquery.com/) was made. And before we knew it, the language that was made to display sparkles and counters on websites took over everything and we started writing single page applications or SPAs.

### Not the SPA you’re thinking of

A SPA basically absolves the backend of its’ HTML/CSS duties. Servers don’t return HTML anymore. They expose REST endpoints. Each one returns JSON in response. In SPAs, when the user requests `yourdomain.com/any-thing`, the server responds with an empty HTML document that has a JavaScript that then takes over. The frontend handles fetching data, rendering html pages, and routing between them. This is when frameworks like [Knockout.js](https://knockoutjs.com/), [Backbone.js](https://backbonejs.org/), and [Angular.js](https://angularjs.org/) were released (late 2010). Three years later, React was released with its’ JSX syntax and one way data-binding. And it made writing web apps easier than ever. Or so we thought.

If a user submits a form, they don’t expect the page to refresh anymore. Since it’s an *app*, it should behave like one. If the data mutates, new data should just appear. Meaning developers had to write code to maintain *state* and *react* to any change in the state by *re-rendering* parts of the HTML accordingly. Imagine the poor frontend devs doing that in a large scale app like **Facebook** with so many moving parts to take care of. Imagine the poor backend devs needing to create numerous REST endpoints to accommodate all that functionality.

### Facebook

Say you’re a frontend dev in the ads team at Facebook. A new feature is proposed by the product manager, you go through the requirements they wrote and you start implementing. You know the feature is going to need access to some data from the backend but you’re at ease because your backend REST endpoints are well documented. You search the API docs, and you find a couple APIs that seem to fit what you need. You start writing your frontend code to find out that the docs are outdated and one of endpoints has been removed because it’s no longer in use. You let your project manager know and they assign a backend dev to work with you. After a couple days of waiting, they message you saying they added the endpoint you needed. You thank them and continue implementing the feature only to run into another problem. The *other* endpoint returns way too much data and is way too slow to be used within the context of the new feature you’re implementing. Oh and the product manager messaged you saying that requirements changed. So you repeat the previous cycle all over again.

What went wrong there ? a few things, but I’ll focus on two:

1.  The API endpoints and there documentation are two separate things and so they can go out of sync.
2.  There’s no runtime guarantee that the API returns what it should. Essentially there’s no strict contract between the server and the client. It can return any JSON and the response will still be 200 OK.

Now you can probably solve the problems above by writing more automated tests and/or investing in doc-gen tools like [Swagger](https://swagger.io/). But these things treat the symptoms (bugs) while not curing the disease (lack of contract). So in 2012, Facebook engineers thought of something better.

### GraphQL

[GraphQL](https://graphql.org/) fixes the above problems by changing APIs from a set of untyped REST endpoints like this:

*   `GET /posts`
*   `POST /posts`
*   `GET /posts/:id`
*   `DELETE /posts/:id`

To a strictly typed, queryable schema like this:

```
type Post {
    id: ID!
    body: String
    date: Date
    Author: User
}

type User {
    id: ID!
    username: String
    first_name: String
    last_name: String
    full_name: String
    name: String @deprecated
    avatar_url: Url
}

scalar Url
scalar Date

type Query {
    Post(id: ID!): Post
    Posts(limit: Int, skip: Int, sort_field: String, sort_order: String): [Post]
    User(id: ID!): User
}

type Mutation {
    createPost (
        body: String
    ): Post
    deletePost(id: ID!): Tweet
}
```

Now because the GraphQL server always checks coming requests and outgoing responses against the schema we define, we now have our contract, our runtime guarantee that the frontend is only requesting fields that exist and the backend will either respond with them or with an error. A few positive side effects of this design are:

*   No need for explicit documentation anymore. The schema is most of the documentation you need. And it’s always up to date.
*   The client can pick precisely the fields they need. No need to request the entire `User` object if you just need `id` and a `username`.
*   Frontend state management becomes way simpler thanks to GraphQL clients like [Relay](https://relay.dev/), [Urql](https://commerce.nearform.com/open-source/urql/docs/), And [Apollo](https://www.apollographql.com/).

So that’s it. A SPA frontend that is powered by a GraphQL server. That’s the *ideal* architecture for **every** web app. I mean, Facebook uses it so it must be the best option right ? … right ?

### Remember websites ?

What if one needs to create a blog ? Typically, all a blog has to do is show some markdown and be SEO friendly. SPAs were notoriously difficult for SEO crawlers to index because it takes a few more hundred milliseconds for JavaScript to render on the client. So what then ? Does one have to go back to [Wordpress](https://wordpress.org/) ?

[**Atwood’s Law**](https://blog.codinghorror.com/the-principle-of-least-power/):

> Any application that *can* be written in JavaScript, *will* eventually be written in JavaScript.

By that point [Node.js](https://nodejs.org/) had picked up enough steam. We were beginning to see a new generation of tooling that was birthed from the idea that JavaScript is going to run **everywhere**. Not just the browser. Among these tools was [Gatsby](https://www.gatsbyjs.com/). It allowed you to write “websites” in React. It came riding on a wave of “headless” CMS startups that were trying to compete in this new market. All of a sudden we saw terms like ”[JAM stack](https://jamstack.org/)” thrown around. All it meant is decoupling presentation from data. Which was not a new idea. It turned out to be a very short lived trend for JAM stack. Gatsby couldn’t catch a break. Just as it was about to gain some traction as the defacto Node.js/React website generator, a new challenger approached.

### Next.js

Back then (around 2016), React was not opinionated. It was a library that only concerned itself with rendering and state. While other frameworks like Angular.js came bundled with everything you may need to get up and running, React opted to allow the community to figure things out. And figure out they did. We had a new JavaScript bundler every other Tuesday, a new router every Sunday, a new CSS-in-JS solution every Friday, and a new state management library every f***ken day.

Enters Next.js. Your very own open-source / blazingly-fast (not really) / batteries-included React framework. While things like Gatsby offered a choice between a dynamic SPA or a static pre-rendered website, Next.js supported multiple rendering modes. By default it pre-rendered pages which made site generators like Gatsby obsolete. In addition to pre-rendering, it supported server-side rendering for cases when page content can’t be known at build time and must be rendered depending on request params. It also came with client-side rendering support which also made it a viable option for SPAs. Furthermore, you could mix and match between all these rendering modes. The way it works is once the user requests a certain page from the server, the server renders an HTML document with all the elements needed to view the page. The JavaScript would take over and hydrate all the elements with the necessary event listeners for the page to be interactive. A client side router would also take over. This meant that we don’t need to choose between websites and apps anymore. As Next gave us the SEO/performance benefits of server-side-rendering while maintaining the app-like user experience. It meant the death of the website/web app dichotomy.

### The Cloud™

Let’s take the timeline back a few years. At around 2006, another change brewing in parallel was the rise of “The Cloud”. In simple terms, it means you don’t have to use your bare metal servers anymore. You can use Amazon’s. Yes, Amazon had a few servers they did not need and decided to start [AWS](https://aws.amazon.com/). Amazon’s approach was unique compared the OGs that existed at that time. Companies like IBM and Oracle went for big/exclusive enterprise contracts that aimed to take over the IT department of a company using their proprietary tools. AWS offered a number of independent services that anyone could buy. Make no mistake, the end goal for AWS was the exact same: take over the IT department of your company. They just had you (the developer) work for it yourself. Genius if you ask me.

In practice, AWS’s approach of offering a large number of independent services that you can stitch together to architect your app worked well for large companies with dedicated sys-admin/ops teams. It didn’t work quite well for the new wave of JavaScript developers that just graduated bootcamps. Why do I have to fiddle with 15 services and an abhorrent dashboard to deploy my twitter clone ? And so a new generation of cloud companies was birthed from the market’s need for simpler cloud platforms. To my knowledge most of which were built on top of AWS. They just made the most common use cases easier than ever. Coming with things like CI/CD, scaling, analytics, and CDN out of the box and packaged in a beautiful dashboard. Among these companies was a company named [Vercel](https://vercel.com/). The company that made Next.js.

### JavaScript that’s a lot more like C#

The more JavaScript took over, the more people started to realize that it was inadequate for its’ new role as the one language to rule them all. Among its’ many inadequacies is being a dynamically typed language. Which meant that building libraries and big applications using JavaScript was a tedious process of `console.log`ing everything and hoping you get what you need. So a statically typed JavaScript was needed.

Meanwhile, a team at Google was building Angular 2 (because the first one wasn’t enough). They needed a typed JavaScript with support for decorator syntax. They were about to create their own language when they discovered that Microsoft had beat them to it and created [TypeScript](https://www.typescriptlang.org/).

TypeScript is simply a compiler that offers static type analysis to your JavaScript code. Its’ syntax was very close to JavaScript so adopting it was very simple. Facebook tried to compete with [Flow](https://flow.org/). But TypeScript’s familiarity ended up winning and Flow … flew into the mist (sorry). But why stop at static type analysis and syntactic sugar ? Why not make the most out of our build step ?

### Svelte

Driven by the belief that frameworks should only augment/enhance the web platform not replace it or change it, [Svelte](https://svelte.dev/) was made. While most frameworks at the time of its’ release had to ship a large JavaScript bundle to handle rendering and reactivity, Svelte was just a compiler. There’s no Svelte script that gets shipped to the browser. The Svelte compiler generates some highly optimized reactivity code that’s much much smaller and faster than the rest.

The team behind it also made [SvelteKit](https://kit.svelte.dev/). A framework on top of Svelte that handles bundling, routing, and pretty much everything you may need in a web framework. Its’ very mature routing and data-loading patterns have made me give up on using GraphQL in most of my projects. Why spend time and effort securing and optimizing a GraphQL schema when I can throw a database call in a `load` function and have everything be typed server to client ? It’s an absolute win when it comes to app development. Similar techniques are also available in other frameworks like [Nuxt](https://nuxt.com/), [Next](https://nextjs.org/), and [SolidStart](https://start.solidjs.com/getting-started/what-is-solidstart). This makes the overhead cost of GraphQL development very hard to justify in *most* cases.

### HTMX: there and back again

Remember the good ol’ days ? When the server sent HTML instead of JSON ? [HTMX](https://htmx.org/) gives all the power *back* to the server. You include an HTMX script tag on all your pages. Any mutation that happens to the state will result in HTMX swapping an element on the screen with whatever the backend sent. This is not a particularly new idea. For example, [GitHub](https://github.com/) is already built that way using [Ruby](https://www.ruby-lang.org/en/). But HTMX revitalized it by [posting memes](https://twitter.com/htmx_org/status/1306234341056344065) on twitter. And I love it.

We must also appreciate the appeal it has to programmers who are not in the JavaScript bubble. Imagine you’ve been writing Go, Java, or anything that’s proper, and you went to make a website. People started telling you to use JavaScript. But why include JavaScript to begin with ? Why not do the whole thing in the one language you like? HTMX offers that to people without coupling it with a complete backend framework like [Rails](https://rubyonrails.org/) does it. You pick whatever backend language you like and as long as you’re returning HTML in response, you’re good to go. And speaking of Rails, why is that framework not dead yet ? Why is PHP still a thing ? And most importantly, why on earth is React still alive ?

### React is not going to die

I know it’s very hard. But let’s look at this objectively. As of the time of writing (early 2024), is React *better* at any particular thing compared to all the other mainstream UI frameworks ? **No, it is not**. Even Angular, the framework everyone loves to hate has been getting updates that objectively improve performance, maintainability, and developer experience.

Meanwhile React seems to be wandering aimlessly. We started with class components, then we changed some life cycle hooks, then everything became “functional”. Everyone was praising the new “functional” API with `useState` and `useEffect`. But people quickly realized that `useEffect` was a very bad abstraction given the way React works. It was a foot gun. And for a while React devs didn’t know what was the alternative ? It was at this time that Vercel started hiring a few of React’s core maintainers. It was at this time when Next.js and React essentially became one and the same. And [server components](https://www.joshwcomeau.com/react/server-components/) became the new ~~foot-gun~~ API the React team is recommending. All that and people are yet to figure out the “React” way of doing things.

Regardless of all that pain, React is still [the most downloaded framework on NPM](https://npmtrends.com/@angular/core-vs-react-vs-solid-js-vs-svelte-vs-vue) compared to other frameworks. It’s not even close. But why ?

Look around you. No technology that was once mainstream ever dies. People rarely pick “the best tool for the job”. They say that. But what they mean is “the tool I’m familiar with the most”. That’s it. That’s why React is probably not dying anytime soon. But it doesn’t matter.

### Really, it does not.

Wanna keep using React ? Wanna switch to something better like Svelte ? Wanna avoid JavaScript like the plague and use HTMX ? It doesn’t matter to the end user. As long as you’re providing value to people and/or having fun doing it, you’re good. Don’t feel bad about your technical choices because someone on the internet wants you to. Programming is a craft. There’s way more leeway than the person who’s selling you a course about “framework X”, “language Y”, or a book about “clean code/design/architecture” wants you to believe. And putting all this history into writing only serves to enforce this sentiment. **Things will change**. And no matter how much they do, we’ll still be programmers. Our primary function is to write useful programs. Have fun 🍉