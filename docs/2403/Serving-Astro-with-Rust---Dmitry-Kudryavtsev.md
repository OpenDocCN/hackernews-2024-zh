<!--yml
category: 未分类
date: 2024-05-27 14:47:40
-->

# Serving Astro with Rust - Dmitry Kudryavtsev

> 来源：[https://www.yieldcode.blog/post/serving-astro-with-rust/](https://www.yieldcode.blog/post/serving-astro-with-rust/)

As some of you might know, for the past few months, my free time is spent on [JustFax](https://justfax.online). I also published few blog posts about this project: [here](/post/building-a-webapp-in-rust/), [here](/post/webapp-localization-in-rust/), and [here](/post/rendering-emails-with-svelte/). Let’s try to milk this project some more, this time with Rust and Astro.

The entire website for JustFax ~~is~~ was fully server rendered. Like in the old times. You would hit a URL, and the server would respond with an HTML. Interactivity was handled by [htmx](https://htmx.org/) and a bunch of vanilla JavaScript.

## The problems with server side rendering

### Templates are hard to maintain

Firstly, let’s not confuse Server Side Rendering (SSR) and Server Side Generation (SSG). The former renders the HTML on demand per request, while the latter generates HTML files beforehand to be served by a server.

I was using the [tera](https://docs.rs/tera/latest/tera/) templating engine. Tera works with a format similar to Jinja. Overall, the syntax is nice, and tera is great. I was even able to implement hot reloading of templates, so I could run the rust server and edit the HTML, without the need to restart the server.

However, as the project grew bigger, it became harder to maintain the HTML templates and, more importantly, the context I would pass to the templates. Templates need variables. Variables are passed from Rust to the tera engine by means of serialization (if I’m not mistaken via `serde_json`). In reality, they are completely detached from each other. And this brings me to the first issue of server side rendering: errors in rendering the templates are **run time**.

This means that if I misspelled a variable, the template would fail to render. Refactoring code, and renaming variables—became a nightmare. I needed to check each page manually before committing to deploying a new version. I was considering switching to [askama](https://github.com/djc/askama), which is a type-safe, Jinja-like, compiled template engine. But eventually decided to generate all the HTML beforehand.

In addition to that, Jinja, or at least the way it’s implemented in tera, is very limiting. I would end up defining variables that are used purely for HTML purposes, in Rust, because I wasn’t able to achieve the same inside a template. Examples include scenarios like: converting a vector of structs into JSON; and various translation shenanigans. This is in no way a criticism towards tera; tera is a great templating engine, and I would definitely use it again in the future (if askama won’t prove to be better). I never worked with Jinja or Django, so I assume these might be limitations of the templating language itself, rather than the tool.

Oh, and one last annoyance. Jinja is poorly supported in vim. So I had to fight the editor with stupid errors. One great example is from this piece of code:

```
{% if country is defined and country is string -%}
const INITIAL_COUNTRY = "{{country}}";
{% else -%}
 {% if LANG == "en" -%}
 const INITIAL_COUNTRY = "us";
 {% elif LANG == "de" -%}
 const INITIAL_COUNTRY = "de";
 {% elif LANG == "fr" -%}
 const INITIAL_COUNTRY = "fr";
 {% endif %}
{% endif -%} 
```

The LSP would yell at me for trying to re-declare `INITIAL_COUNTRY`.

### Everything is a run-time problem

With server side templates, everything becomes a run-time problem. Forgot to declare a variable? Run time errors. Extending a base template that does not exist? Run time errors.

On a small website with 2–3 pages, it’s easy to check each page manually. But at some point it becomes too complicated, and I wish I could outsource this to some build step. Maybe compiled templates like askama would help here, but I haven’t tried them.

In addition to that, maintaining SSR requires a lot of tools. Rust is amazing in run-time, but not that in compile-time. Removing tera and other crates that I used that helps with template rendering—improved the compile time of the project. Previously, it would take about 4 minutes to compile the rust code in CI. After removing tera, markdown related crates, and other crates used for formatting (for example phone number formatting)—I was able to reduce the compilation time to 3 minutes, which might not sound like a lot, but this is a great improvement.

In the end, if I look at it, the HTML was completely static. There was no need to re-render it for each request, and the dynamic parts were small enough to be able to generate them in the browser without incurring any performance penalty. I also learned the hard way that Rust is way behind JavaScript in terms of l10n and i18n. While it has great support for translations (I personally like [project fluent](https://projectfluent.org/) by Mozilla)—it has no crates to format dates/numbers/currencies. There are crates like [icu](https://crates.io/crates/icu) that are in development, that essentially tap into the Common Locale Data Repository ([I wrote about CLDR back in 2015](/post/a-different-approach-for-localizing-react-js-app/)). This would allow Rust to have the same capabilities as JavaScript has through the `Intl` API, but currently it’s still a work-in-progress.

And so, I decided to turn to my second favorite tool.

## Hello Astro-naut

I’m a big fan of Astro, and in fact this blog is powered by Astro. Astro, unlike other frameworks, does not force you to use a particular frontend framework, you can use vanilla JavaScript or TypeScript, or you can mix them all together. It’s fast enough to generate hundreds of pages. It comes with a lot of tools for websites like automatic sitemap generation, assets optimization and bundling, etc.

Most of the issues I described above are also solved by Astro. Template data and representation, are connected, especially if you opt for TypeScript. It will be impossible to generate a page that includes a non-existing Astro component, for example. Astro comes with superb support for Markdown content collection, which makes it super easy to generate content. When I was using Rust, I had to sort-of reimplement what Astro does. It didn’t work well.

L10n and i18n are also solved with Astro. You can pick any translation library you want, including project fluent from Mozilla. And you have access to `Intl` for formatting numbers, dates, and currencies.

The only problem with Astro is that it’s a static site generator. **Wait? What?!** Yes, I know I said that I wanted to switch to SSG from SSR, how come it’s a downside?

Well, as I described in this blog post—[Web app localization in Rust](/post/webapp-localization-in-rust/)—I need a server in order to do locale negotiation. Before rendering the initial HTML, I need to query cookies and the `Accept-Language` header, and determine the preferred language of the visitor. This can be done with JavaScript, for example by using the `window.navigator.languages` property (which [MDN](https://developer.mozilla.org/en-US/docs/Web/API/Navigator/languages) claims to be the same values as `Accept-Language` header). However, this would harm usability, as Google won’t be able to index the website correctly.

Astro, does, however take care of this as well. If you decide to opt for `hybrid` or `server` output for your Astro website, you will gain access to `Astro.preferredLocale` and `Astro.preferredLocaleList` properties (as describe in the [Internalization article](https://docs.astro.build/en/guides/internationalization/#browser-language-detection) from Astro Docs). But opting in for `hybrid` or `server` outputs creates a problem. You are now locked into either using Netlify or Vercel edge functions (which has official Astro adapters), or go back to Node. Neither of them I wanted to do. I have my locale negotiation code in Rust, I want it to stay in Rust. In addition to that, the locale negotiation by Astro does not take into account the cookie that I set in order to avoid sending an English speaking German person to the German version of the website, even though he insisted for English interface, unlike Google is doing.

And so, I was left with the only option, serve Astro website with Rust.

## Serving Astro website with Rust

Astro generates, by default, your entire website structure in the `dist` folder. HTML pages as well as additional resources will be there. All is left, is just to expose this folder to the web.

One approach I wanted to use, is to serve the website by [Caddy](https://caddyserver.com/). I already use Caddy as reverse-proxy for the Rust server, and Caddy is capable of serving folders. However, figuring out the correct handlers order for Caddy—made my cabesa hurt. You see, theoretically I could point `/` to `/` on Rust server; `/api` to `/api` on Rust server; `/en` to `/dist/en`; `/de` to `/dist/de`; etc. But what do I do with static resources like JavaScript, sitemap, robots.txt, etc? You see, it makes my head hurt. So this option was out of the window.

Let’s now talk about Rust. I use [axum](https://github.com/tokio-rs/axum) as my web framework. Axum is capable of serving directories using the `ServeDir` service. You give it a path to a directory, and it will serve the files from there. So I have a regular axum route that handles the `/` url; route that handle `/api` url(s); and everything else is fallen back to `ServeDir`. This covers 99% of the files. The last 1% are 404 files. Astro will allow you to define `404.astro` for a custom 404 page, and services like Netlify or Vercel will be able to route to it, but not axum.

If axum fails to find a matching route for the request, nor it finds a static file inside the provided `ServeDir` path, you are able to implement a custom 404 handler.

```
let fallback_service = ServeDir::new(public_path)
 .append_index_html_on_directories(true)
 .not_found_service(Handler::with_state(handle_404, context.clone())); 
```

Inside `handle_404` you can do all kinds of things. I do locale negotiation to decide which version of the 404 page to serve, but eventually it boils down to

```
match ServeFile::new(path.as_path())
 .try_call(Request::new(axum::body::Body::empty()))
 .await
{
 Ok(res) => res.into_response(),
 Err(e) => (StatusCode::NOT_FOUND).into_response()
} 
```

While `path` is the path to `[en|de]/404/index.html`.

Another thing with Astro, is that it has a great asset management pipeline. Each static resource will be processed by Astro and a hashed name file will be put it `_astro` directory. Since the hash of the file represents the hashed content, if the file did not change, it means that it will have the same hash. And this means that `_astro` is a great directory to cache aggressively.

I had to fight a bit with axum in order to understand how to send headers alongside `ServeDir` file, but eventually I was able to figure it out.

```
let public_path = Path::new(&config.server.www_dir);
let fallback_service = ServeDir::new(public_path)
 .append_index_html_on_directories(true)
 .not_found_service(handle_404.into_service());
 Router::new()
 .fallback(get(|req: Request| async move {
 let (mut parts, body) = req.into_parts();
 let uri: OriginalUri = parts.extract().await?;
 let req = Request::from_parts(parts, body);
 match fallback_service.oneshot(req).await {
 Ok(mut res) => match res.status() {
 StatusCode::OK => {
 if uri.path().contains("/_astro/")
 {
 res.headers_mut().insert(
 "Cache-Control",
 "public, max-age=31536000".parse().unwrap(),
 );
 }
 Ok(res)
 }
 _ => Ok(res),
 },
 Err(e) => {
 tracing::error!("fallback_service error: {}", e);
 Err(e)
 }
 }
 }))
 .layer(service_layer) 
```

This essentially uses the `ServeDir` service through the `oneshot` request, and based on the request `uri` it sets the cache headers for `_astro` folder.

That’s it. That’s the blog post.