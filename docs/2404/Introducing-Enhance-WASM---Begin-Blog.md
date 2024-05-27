<!--yml
category: 未分类
date: 2024-05-27 13:07:55
-->

# Introducing Enhance WASM — Begin Blog

> 来源：[https://begin.com/blog/posts/2024-04-08-introducing-enhance-wasm](https://begin.com/blog/posts/2024-04-08-introducing-enhance-wasm)

## Backend agnostic server-side rendering (SSR) for Web Components

Web Components are the browser native way to extend HTML. But as a primarily browser based technology they are defined with JavaScript which limits them to either rendering solely client side — which has janky performance, poor SEO, and is not optimally accessible — or within a server-side JavaScript runtime, which isn’t always an option for shops that use other backend runtimes.

**[Enhance WASM](https://enhance.dev/wasm)** unlocks **server-side rendering Web Components** for any backend runtime.

*   **Write Once, Render Anywhere**: Author standard web components and deploy them with any backend. **[Enhance WASM](https://enhance.dev/wasm)** takes care of rendering across any server environment.
*   **Seamless Integration**: Easily integrate **[Enhance WASM](https://enhance.dev/wasm)** into your existing projects with minimal setup. Extensive documentation, high quality baseline components, and broad community support make adoption a breeze.
*   **Better DX**: Stop wasting time on slow build steps and reimplementing brittle frontend code that already works. Web standards ensure rock solid reliability and performance without rewrites.
*   **Performance Optimized**: No more spinners or skeleton screens. Enjoy faster load times and improved SEO. Your users get a snappy experience, and you get better developer velocity.

We really believe this is a leapfrog moment for frontend development. Server-side rendering is a key requirement for personalized web applications. Organizations that prioritize the stability, performance and accessibility of web standards run workloads in a huge variety of backend runtimes. **Now we can build browser native web interfaces that cross the runtime chasm.**

## Try it out and get involved!

Enhance is completely open source code and we need your help! We’re opening up Enhance WASM immediately with support for **[Node](https://github.com/enhance-dev/enhance-ssr)**, **[Deno](https://github.com/enhance-dev/enhance-ssr-deno)**, **[Python](https://github.com/enhance-dev/enhance-ssr-python)**, **[Ruby](https://github.com/enhance-dev/enhance-ssr-ruby)**, **[PHP](https://github.com/enhance-dev/enhance-ssr-php)**, **[Java](https://github.com/enhance-dev/enhance-ssr-java)**, **[C#](https://github.com/enhance-dev/enhance-ssr-c-sharp)**, **[Rust](https://github.com/enhance-dev/enhance-ssr-rust)**, and **[Go](https://github.com/enhance-dev/enhance-ssr-go)**. We need your help testing and implementing support for these and other runtimes! If you want to see a runtime not mentioned here [please let us know](https://github.com/enhance-dev/enhance-ssr/issues).

Example starter projects:

## FAQ

*How are components authored?*

Components are authored exactly per the Web Components specification. Enhance WASM also enables a more backend oriented pure functional style and has several helper libraries for making common client-side patterns cleaner.

*Do I need to run client JS to render components?*

No. Web Components are completely rendered server-side HTML. You can then optionally run client-side upgrades should the element need it. It is worth noting we’ve found the majority of elements only need to be server rendered and do not require any client javascript at all.

*Can I use client-side JS with these components?*

Of course!

*What about Lit?*

We definitely hear community folk when they say, “just use Lit”, but per the Lit documentation their own SSR renderer isn’t production ready today, and even then is Node.js only. Lit is ultimately more focused on client-side upgrade than server-side usage which is totally cool! Enhance encourages SSR and “HTML first” and treats the client-side element upgrade as a progressive enhancement step. You can use Enhance to render initial HTML and Lit for client-side interactions if that makes sense for your project. We find most elements are not client-side interactive anyhow.