<!--yml
category: 未分类
date: 2024-05-27 14:37:25
-->

# Release v1.1.0 · observablehq/framework · GitHub

> 来源：[https://github.com/observablehq/framework/releases/tag/v1.1.0](https://github.com/observablehq/framework/releases/tag/v1.1.0)

### Windows support (and Node 18, too) 🎉

Framework now supports Windows! Thank you upvoting [#90](https://github.com/observablehq/framework/issues/90). 👍 Framework internally uses POSIX path delimiters (forward slashes `/`), and now converts to the Windows path delimiters (backward slashes `\`) as needed when reading or writing files. Please reach out if you encounter any issues.

We also now support Node 18\. We previously had a runtime dependency on [tsx](https://github.com/privatenumber/tsx), allowing us to ship Framework’s source as TypeScript; now we publish pre-built JavaScript. This reduces the number of moving parts and means we no longer depend on [module customization hooks](https://nodejs.org/docs/latest-v20.x/api/module.html#customization-hooks) added in Node 20.6.

### Self-hosted `npm:` imports 🎉

Framework now automatically downloads `npm:` imports from npm during preview and build! This means better performance and security for your users as your built site has **no external code dependencies**. For example, you can now enforce a [content security policy](https://developer.mozilla.org/en-US/docs/Web/HTTP/CSP) on your deployed site that disallows cross-origin requests. It also improves performance during local preview since you only need to download libraries once, and means you can develop offline (say from an airplane without wi-fi). Downloads from npm are cached in the `.observablehq/cache/_npm` folder within the source root. You can clear the cache and restart the server to re-fetch the latest versions of libraries from npm.

Self-hosting of `npm:` imports applies to static imports, dynamic imports, and import resolutions (`import.meta.resolve`), provided the specifier is a static string literal. For example to load D3:

```
import * as d3 from "npm:d3";
```

```
const d3 = await import("npm:d3");
```

In both cases above, the latest version of `d3` is resolved and downloaded from jsDelivr as an ES module, including all of its transitive dependencies. Implicit `npm:` imports, as when you use the built-in `d3` provided by Framework’s standard library in Markdown, are also now self-hosted. (If you still want to load a library from the CDN at runtime rather than self-hosting, you can import a URL, but we don’t recommend this as it is slower and means your site could break when a new version of the library is published.)

Transitive static imports are also registered as module preloads (`<link rel="modulepreload">`), such that the requests happen in parallel and as early as possible, rather than being chained. This dramatically improves performance on page load. Framework also now preloads `npm:` imports for `FileAttachment` methods, such as `d3-dsv` for CSV files.

Import resolutions allow you to download files from npm. These files are automatically downloaded for self-hosting, too. For example, to load U.S. county geometry:

```
const data = await fetch(import.meta.resolve("npm:us-atlas/counties-albers-10m.json")).then((r) => r.json());
```

Framework automatically downloads files as needed for [recommended libraries](https://observablehq.com/framework/javascript/imports#implicit-imports), and resolves import resolutions in transitive static and dynamic imports. For example, `npm:@duckdb/duckdb-wasm` needs WebAssembly bundles, `npm:katex` needs a stylesheet and fonts, *etc.* For any dependencies that are not statically analyzable (such as `fetch` calls or dynamic arguments to `import`) you can call `import.meta.resolve` to register additional files to download from npm.

Oh, and we’re working on `jsr:` imports for our next release, too. [#956](https://github.com/observablehq/framework/issues/956)

### CommonMark compatibility 🎉

Framework previously had a quirk with blank lines in Markdown. Now it doesn’t, so you can write bog-standard Markdown with no nasty surprises! 😌 In particular, you can now write Markdown *within* HTML blocks per the [GitHub-flavored Markdown specification](https://github.github.com/gfm/#html-blocks):

```
<div class="card">

## This is a card
### With a subtitle

And some *Markdown* text!

</div>
```

### Content-hashed files names on build

Framework now inserts a content hash into published file names in `_file` and `_import`. This ensures cache-breaking on deploy and enables [cache-control: immutable](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Cache-Control#immutable) to further improve performance. When deploying to Observable, you’ll automatically get optimized caching.

### Draft mode

The new **draft** option allows you to exclude certain pages from the build. This allows you to develop pages locally but prevent them from being published until you are ready to share. To mark a page as draft, use front matter:

### markdown-it plugins

You can now register additional [markdown-it](https://github.com/markdown-it/markdown-it) [plugins](https://www.npmjs.com/search?q=keywords:markdown-it-plugin) using the [**markdownIt** config option](https://observablehq.com/framework/config#markdownit). For example, to use the [markdown-it-footnote](https://github.com/markdown-it/markdown-it-footnote) plugin:

```
import type MarkdownIt from "markdown-it";
import MarkdownItFootnote from "markdown-it-footnote";

export default {
  markdownIt: (md: MarkdownIt) => md.use(MarkdownItFootnote)
};
```

### And many other improvements…

[Apache ECharts](https://observablehq.com/framework/lib/echarts) is now available by default as `echarts` in Markdown. (Though we still prefer our own [Observable Plot](https://observablehq.com/framework/lib/plot) and [D3](https://observablehq.com/framework/lib/d3) libraries for visualization!)

Framework now does live-updating of files referenced in static HTML, such as `<img>` and `<video>` elements, and stylesheets. Framework also does live-updating if you edit your [custom stylesheet](https://observablehq.com/framework/config#style). Framework now skips unused built-in modules and minifies CSS during build.

Framework’s built-in search now correctly indexes text written in any [Unicode writing system](https://en.wikipedia.org/wiki/Script_(Unicode)). Diacritics and HTML entities are normalized before indexing; for example, you can now type “manana” and retrieve pages containing “mañana” and “ma&ntilde;ana”. Indexing no longer crashes when a title is not a string. The home page is now included in the index. The new [**keywords** option](https://observablehq.com/framework/search) allows indexing of additional boosted words in the front matter. And search results no longer show a spurious scrollbar.

`FileAttachment.url()` now returns an absolute URL rather than a relative path. `FileAttachment.mimeType` is now `"application/octet-stream"` instead of `"null"` when the MIME type is not known. Older browsers that do not yet support the CSS `:has()` selector are now supported. Framework no longer considers the `HOSTNAME` and `PORT` environment variables when starting the preview server; use the `--host` and `--port` command-line arguments instead. Markdown headers now have text-wrap: balance applied in default styles.

All this in less than three weeks since launch! 😅 Thanks for all the feedback and encouragement, y’all.

## New Contributors

**Full Changelog**: [`v1.0.0...v1.1.0`](https://github.com/observablehq/framework/compare/v1.0.0...v1.1.0)