<!--yml

类别：未分类

日期：2024-05-27 14:47:40

-->

# 使用 Rust 为 Astro 提供服务 - Dmitry Kudryavtsev

> 来源：[https://www.yieldcode.blog/post/serving-astro-with-rust/](https://www.yieldcode.blog/post/serving-astro-with-rust/)

正如你们中的一些人可能知道的那样，过去几个月，我空闲时间都在 [JustFax](https://justfax.online) 上。我还发布了一些关于这个项目的博客文章：[这里](/post/building-a-webapp-in-rust/)，[这里](/post/webapp-localization-in-rust/)，以及 [这里](/post/rendering-emails-with-svelte/)。让我们再尝试用 Rust 和 Astro 增加这个项目的价值。

整个 JustFax 网站 ~~是~~ 曾完全采用服务器端渲染。就像古早时代一样。你输入一个 URL，服务器就会返回一个 HTML。交互性由 [htmx](https://htmx.org/) 和一堆原生 JavaScript 处理。

## 服务器端渲染的问题

### 维护模板很困难

首先，让我们不要混淆服务器端渲染（SSR）和服务器端生成（SSG）。前者根据请求即时渲染 HTML，而后者预先生成 HTML 文件以供服务器提供。

我使用了 [tera](https://docs.rs/tera/latest/tera/) 模板引擎。Tera 使用类似 Jinja 的格式。总体而言，语法很好，tera 也很出色。我甚至能够实现模板的热重载，这样我可以运行 Rust 服务器并编辑 HTML，而无需重新启动服务器。

然而，随着项目规模的扩大，维护 HTML 模板和传递给模板的上下文变得越来越困难。模板需要变量。变量通过序列化（如果我没记错是通过 `serde_json`）从 Rust 传递给 tera 引擎。实际上，它们完全脱离了彼此。这就带来了服务器端渲染的第一个问题：模板渲染中的运行时错误。

这意味着，如果我拼错了一个变量，模板将无法渲染。重构代码和重命名变量——变成了一场噩梦。在部署新版本之前，我需要手动检查每一页。我曾考虑过切换到 [askama](https://github.com/djc/askama)，这是一种类型安全、类似 Jinja 的编译模板引擎。但最终决定预先生成所有的 HTML。

此外，Jinja，或者至少它在 tera 中的实现方式，非常有限。我最终会定义纯粹用于 HTML 目的的变量，因为在 Rust 中我无法在模板内部实现相同的功能。例如，将结构体的向量转换为 JSON，以及各种翻译上的技巧。这绝不是对 tera 的批评；tera 是一个很棒的模板引擎，我未来肯定会再次使用它（如果 askama 证明不是更好的话）。我从未使用过 Jinja 或 Django，因此我认为这些可能是模板语言本身的限制，而不是工具的限制。

哦，还有一个让人讨厌的地方。在 vim 中，对 Jinja 的支持很差。所以我不得不与编辑器对抗愚蠢的错误。这段代码就是一个很好的例子：

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

LSP 会因我试图重新声明 `INITIAL_COUNTRY` 而对我大声抱怨。

### 一切都是运行时问题

使用服务器端模板，一切都成了运行时问题。忘记声明一个变量？运行时错误。扩展一个不存在的基础模板？运行时错误。

在一个有 2–3 页的小网站上，手动检查每一页很容易。但在某些时候，这变得太复杂了，我希望可以将这一步外包给某些构建步骤。也许像 askama 这样的编译模板会在这里有所帮助，但我还没有尝试过。

此外，维护 SSR 需要很多工具。Rust 在运行时很棒，但在编译时不太好。删除我使用的 tera 和其他用于模板渲染的 crate——提高了项目的编译时间。以前，在 CI 中编译 Rust 代码大约需要 4 分钟。删除 tera、与 Markdown 相关的 crates 和其他用于格式化（例如电话号码格式化）的 crates 后，我成功将编译时间减少到 3 分钟，听起来可能不算多，但这是一个很大的改进。

总的来说，如果我仔细看，HTML 完全是静态的。无需为每个请求重新渲染它，动态部分足够小，可以在浏览器中生成而不会造成任何性能损失。我也深知 Rust 在 l10n 和 i18n 方面远远落后于 JavaScript。虽然它对翻译有很好的支持（我个人喜欢 Mozilla 的 [project fluent](https://projectfluent.org/)）——但它没有用于格式化日期/数字/货币的 crates。有一些像 [icu](https://crates.io/crates/icu) 这样的 crates 正在开发中，它们本质上是利用了通用区域数据存储库（我在 2015 年写过关于 CLDR 的文章）[/post/a-different-approach-for-localizing-react-js-app/]。这将使 Rust 可以通过 `Intl` API 具有与 JavaScript 相同的功能，但目前仍在进行中。

因此，我决定转向我第二喜欢的工具。

## 你好，宇航员

我是 Astro 的忠实粉丝，事实上，这个博客就是由 Astro 提供支持的。与其他框架不同，Astro 不强制你使用特定的前端框架，你可以使用原生 JavaScript 或 TypeScript，或者混合使用它们。它足够快，可以生成数百个页面。它提供了许多网站工具，如自动生成站点地图、资源优化和打包等。

大多数我上面描述的问题也可以通过 Astro 解决。模板数据和表示形式是相互关联的，特别是如果您选择 TypeScript。例如，无法生成包含不存在的 Astro 组件的页面。Astro 对 Markdown 内容收集提供了出色的支持，这使得生成内容非常容易。当我使用 Rust 时，我不得不重新实现 Astro 的功能，但效果并不好。

L10n 和 i18n 也可以通过 Astro 解决。您可以选择任何您想要的翻译库，包括 Mozilla 的项目 fluent。而且，您可以使用 `Intl` 格式化数字、日期和货币。

Astro 的唯一问题是它是一个静态站点生成器。**等等？什么？！** 是的，我知道我说过我想从 SSR 切换到 SSG，为什么这是一个缺点？

好吧，正如我在这篇博文中描述的那样—[在 Rust 中的 Web 应用本地化](/post/webapp-localization-in-rust/)—我需要一个服务器来进行区域设置协商。在渲染初始 HTML 之前，我需要查询 cookie 和 `Accept-Language` 头，并确定访问者的首选语言。例如，可以使用 JavaScript，比如使用 `window.navigator.languages` 属性（[MDN](https://developer.mozilla.org/en-US/docs/Web/API/Navigator/languages) 中指出与 `Accept-Language` 头相同的值）。然而，这会影响可用性，因为 Google 将无法正确索引网站。

不过，Astro 也考虑到了这一点。如果您决定为您的 Astro 网站选择 `hybrid` 或 `server` 输出，您将可以访问 `Astro.preferredLocale` 和 `Astro.preferredLocaleList` 属性（如 Astro 文档中的[国际化文章](https://docs.astro.build/en/guides/internationalization/#browser-language-detection)所述）。但选择 `hybrid` 或 `server` 输出会带来问题。您现在必须选择使用 Netlify 或 Vercel 边缘函数（这些平台有官方的 Astro 适配器），或者回到 Node。我都不想这样做。我有我的区域设置协商代码在 Rust 中，我希望它保持在 Rust 中。此外，Astro 的区域设置协商并不考虑我设置的 cookie，以避免将一个希望使用英文界面的德国人发送到网站的德语版本，尽管他坚持要英文界面，这与 Google 的做法不同。

所以，我只剩下一个选择，用 Rust 为 Astro 网站提供服务。

## 用 Rust 为 Astro 网站提供服务

默认情况下，Astro 在 `dist` 文件夹中生成整个网站结构。HTML 页面以及其他资源都会在那里。现在只需要将这个文件夹暴露到网络即可。

我想使用的一种方法是通过[Caddy](https://caddyserver.com/)来提供网站服务。我已经将Caddy作为Rust服务器的反向代理，并且Caddy能够服务文件夹。然而，弄清楚Caddy的正确处理程序顺序让我很头疼。理论上，我可以将`/`指向Rust服务器上的`/`；将`/api`指向Rust服务器上的`/api`；将`/en`指向`/dist/en`；将`/de`指向`/dist/de`；等等。但是像JavaScript、sitemap、robots.txt等静态资源怎么办呢？你看，这让我很头疼。所以这个选项被搁置了。

现在让我们谈谈Rust。我使用[axum](https://github.com/tokio-rs/axum)作为我的Web框架。Axum能够使用`ServeDir`服务来提供目录。你给它一个目录的路径，它就会从那里提供文件。因此，我有一个常规的axum路由处理`/`网址；一个处理`/api`网址的路由；而其他所有东西都会回退到`ServeDir`。这覆盖了99%的文件。最后的1%是404文件。Astro允许你为自定义404页面定义`404.astro`，并且像Netlify或Vercel这样的服务可以路由到它，但是axum不能。

如果axum无法找到匹配请求的路由，也找不到在提供的`ServeDir`路径中的静态文件，你可以实现一个自定义的404处理程序。

```
let fallback_service = ServeDir::new(public_path)
 .append_index_html_on_directories(true)
 .not_found_service(Handler::with_state(handle_404, context.clone())); 
```

在`handle_404`中，你可以做各种事情。我做了本地化协商来决定要提供哪个版本的404页面，但最终归根结底是

```
match ServeFile::new(path.as_path())
 .try_call(Request::new(axum::body::Body::empty()))
 .await
{
 Ok(res) => res.into_response(),
 Err(e) => (StatusCode::NOT_FOUND).into_response()
} 
```

而`path`是到`[en|de]/404/index.html`的路径。

另外，关于Astro的另一点是它有一个很好的资产管理流水线。每个静态资源将通过Astro处理，并且将一个带哈希名的文件放在`_astro`目录下。因为文件的哈希表示了文件的内容，如果文件没有改变，那么它将拥有相同的哈希。这意味着`_astro`是一个很好的可以积极缓存的目录。

我不得不在axum中进行一些努力，以理解如何在`ServeDir`文件旁边发送头部，但最终我能够弄清楚它。

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

这基本上是通过`oneshot`请求使用`ServeDir`服务，并根据请求的`uri`设置`_astro`文件夹的缓存头。

就是这样。这就是这篇博客文章。
