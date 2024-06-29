<!--yml

category: 未分类

date: 2024-05-27 15:03:30

-->

# Remix Vite 现已稳定 | Remix

> 来源：[https://remix.run/blog/remix-vite-stable](https://remix.run/blog/remix-vite-stable)

今天我们很高兴地宣布，在 Remix v2.7.0 中，[Vite](https://vitejs.dev) 的支持现在已经稳定！在[Remix Vite 最初的不稳定发布](./remix-heart-vite)之后，我们在过去几个月中，通过所有早期采纳者和社区贡献者的帮助，不断改进和扩展了它。

这就是我们的工作成果：

让我们分析自我们最初发布以来的最重要变化。

## [](#spa-mode)SPA 模式

我们所做的最重大的变化是如此重要，以至于我们将保留在后续文章中讨论它对 React 生态系统的影响。

简短来说，Remix 现在支持构建纯静态站点，不需要在生产中使用 JavaScript 服务器，同时保持 Remix 的基于文件的路由约定、自动代码拆分、路由模块预取、头部标签管理等优点。

这为[React Router](https://reactrouter.com)的消费者开辟了一条全新的迁移路径，可以在不切换到服务器渲染架构的情况下转移到 Remix — 对许多人来说，这甚至不是一个选项。 而对于任何想要将服务器引入其 Remix 应用程序的人来说，现在的迁移路径变得更加简单直接。

欲了解更多信息，请查阅[SPA 模式文档](https://remix.run/docs/future/spa-mode)。

## [](#basename-support)基础路径支持

[React Router 支持为您的应用设置基础路径](https://reactrouter.com/en/main/router-components/router)，允许您在子路径中嵌套整个应用程序 — 但这个功能在[Remix 中显然是缺失的](https://github.com/remix-run/remix/discussions/2891)。虽然可以通过手动添加路由和链接来解决这个问题，但显然不如设置一个单一的配置值方便。

随着转向 Vite，缺乏基础路径支持变得更加明显，因为[Vite 暴露了自己的“base”选项](https://vitejs.dev/config/shared-options.html#base)。 许多消费者错误地认为这将与 Remix 一起工作，但这个选项实际上与[Remix 的“publicPath”选项](https://remix.run/docs/en/main/file-conventions/remix-config#publicpath)相同。

为了避免这种混淆，不再提供`publicPath`选项（您应该使用 Vite 的`base`选项替代），而 Remix Vite 插件现在有了全新的`basename`选项。

因此，现在可以更轻松地将 Remix 应用程序嵌套在您站点的子路径中，而无需触及您的应用程序代码。

```
import { vitePlugin as remix } from "@remix-run/dev"; import { defineConfig } from "vite";   export default defineConfig({
 base: "/my-app/public/", plugins: [ remix({ basename: "/my-app", }), ], }); 
```

## [](#cloudflare-pages-support)Cloudflare Pages 支持

在我们最初不稳定的 Remix Vite 发布中，[Cloudflare Pages](https://pages.cloudflare.com) 的支持尚未完全准备好。 Cloudflare 的`workerd`运行时与 Vite 的 Node 环境完全分离，因此我们需要找到一个最佳的方法来弥合这个差距。

随着 Remix Vite 稳定推出，我们现在提供了一个内置的 Vite 插件，用于在本地开发中集成 Cloudflare 的工具。

为了在 Vite 中模拟 Cloudflare 环境，[Wrangler](https://developers.cloudflare.com/workers/wrangler) 提供了 [Node 代理到本地 `workerd` 绑定](https://developers.cloudflare.com/workers/wrangler/api/#getplatformproxy)。Remix 的 `cloudflareDevProxyVitePlugin` 为你设置了这些代理：

```
import {
 vitePlugin as remix, cloudflareDevProxyVitePlugin as remixCloudflareDevProxy, } from "@remix-run/dev"; import { defineConfig } from "vite";   export default defineConfig({
 plugins: [remixCloudflareDevProxy(), remix()], }); 
```

然后可以在你的 `loader` 或 `action` 函数中通过 `context.cloudflare` 使用这些代理：

```
export const loader = ({ context }: LoaderFunctionArgs) => {
 const { env, cf, ctx } = context.cloudflare; // ... more loader code here... }; 
```

我们仍在积极与 Cloudflare 团队合作，以确保 Remix 用户获得最佳体验。未来，通过利用 Vite 的新（仍处于试验阶段的）[Runtime API](https://vitejs.dev/guide/api-vite-runtime#vite-runtime-api)，集成可能会更加无缝，因此请继续关注进一步的更新。

欲了解更多信息，请查阅 [Remix Vite + Cloudflare 文档](https://remix.run/docs/future/vite#cloudflare)。

## [](#server-bundles)服务器捆绑

对于那些已经在 [Vercel 上运行 Remix](https://vercel.com/docs/frameworks/remix) 的人来说，你可能已经注意到 Vercel 允许你将服务器构建分成多个捆绑，不同的路由针对 [无服务器函数](https://vercel.com/docs/frameworks/remix#serverless-functions) 和 [边缘函数](https://vercel.com/docs/frameworks/remix#edge-functions)。

或许你没有意识到的是，这个功能实际上是通过 [Remix 的分支](https://www.npmjs.com/package/@vercel/remix-run-dev)（Vercel 在其 [Remix 构建器](https://github.com/vercel/vercel/blob/main/packages/remix/src/build.ts) 中使用）来实现的。

在迁移到 Vite 后，我们希望确保不需要我们的构建系统的另一个分支，因此我们一直与 Vercel 团队合作，将此功能引入 Remix Vite。现在 *任何人* — 不仅仅是 Vercel 消费者 — 都可以根据他们的喜好将他们的服务器构建分成多个捆绑。

特别感谢 Vercel，尤其是 [Nathan Rajlich](https://n8.io)，为这项工作提供帮助。有关此功能的更多信息，请查阅 [服务器捆绑文档](https://remix.run/docs/future/server-bundles)。

## [](#presets)预设

在调查 Vercel 对 Remix Vite 的支持时，明确了我们需要一种方法让其他工具和托管提供商能够定制 Vite 插件的行为，而不是深入内部或运行自己的分支。为支持这一点，我们引入了“预设”概念。

预设只能做两件事：

+   配置 Remix Vite 插件代表你进行。

+   验证已解析的配置。

预设旨在发布到 npm 并在你的 Vite 配置中使用。

Vercel 预设即将推出，我们非常期待看到社区会推出哪些其他预设 — 特别是预设可以访问所有 Remix Vite 插件选项，因此不仅仅限于托管提供商支持。

关于此功能的更多信息，包括如何创建您自己的预设，请查阅 [预设文档](https://remix.run/docs/future/presets)。

## [](#better-server-and-client-separation)更好的服务器和客户端分离

Remix 允许您使用 `.server.ts` 扩展名命名文件，以确保它们永远不会意外地出现在客户端。然而，我们发现我们之前的实现与 Vite 的 ESM 模型不兼容，因此我们不得不重新审视我们的方法。

反之，如果我们将 `.server.ts` 文件在客户端代码路径中导入，那会怎样呢？将其编译为错误会更好吗？

我们之前的方法导致运行时错误很容易溜到生产环境中。在构建过程中引发这些错误可以避免影响真正的用户，同时为开发人员提供更快速、更全面的反馈。我们很快意识到这种方法要好得多。

作为奖励，由于我们已经在这个领域工作过，我们决定添加对 `.server` *目录* 的支持，而不仅仅是文件，这样可以轻松地标记项目的整个部分为仅限服务器。

如果您希望深入了解此更改背后的原因，请查看我们关于在 Vite 中分离客户端和服务器代码的 [决策文档](https://github.com/remix-run/remix/blob/main/decisions/0010-splitting-up-client-and-server-code-in-vite.md)。

### [](#vite-env-only)vite-env-only

出于速度的考虑，Vite 惰性编译每个文件。默认情况下，Vite 假设由客户端代码引用的任何文件都是完全安全的。

Remix 自动处理路由文件中 `loader`、`action` 和 `headers` 导出的移除，确保它们对浏览器始终安全。但是，非 Remix 导出的情况如何处理呢？我们如何知道哪些应该从浏览器构建中删除？这不仅仅是路由文件的问题，而是项目中的任何模块都会受到影响。

例如，如果您想写类似这样的东西会怎么样？

```
import { db } from "~/.server/db";   // This export is server-only ❌ export const getPosts = async () => db.posts.findMany();   // This export is client-safe ✅ export const PostPreview = ({ title, description }) => (
 <article> <h2>{title}</h2> <p>{description}</p> </article> ); 
```

在当前文件的状态下，由于在客户端上使用了 `.server` 模块，Remix 将抛出编译时错误。这是件好事！您绝对不希望将仅限服务器的代码泄露给客户端。您可以通过将仅限服务器的代码拆分到自己的文件中来解决此问题，但如果不想重新构造代码，这将会很不方便，特别是当您正在迁移现有项目时！

这个问题并不特定于 Remix。实际上，它影响到任何全栈 Vite 项目，因此我们编写了一个独立的 Vite 插件称为 [vite-env-only](https://github.com/pcattori/vite-env-only) 来解决它。该插件允许您将单个 *表达式* 标记为仅限服务器或仅限客户端。

例如，当使用 `serverOnly$` 宏时：

```
import { serverOnly$ } from "vite-env-only";   import { db } from "~/.server/db";   export const getPosts = serverOnly$(async () => db.posts.findMany());   export const PostPreview = ({ title, description }) => (
 <article> <h2>{title}</h2> <p>{description}</p> </article> ); 
```

在客户端，这变成了：

```
export const getPosts = undefined;   export const PostPreview = ({ title, description }) => (
 <article> <h2>{title}</h2> <p>{description}</p> </article> ); 
```

**值得重申的是，这是一个独立的 Vite 插件，而不是 Remix 的功能。** 完全由您决定是否喜欢使用 `vite-env-only`，将服务器专用代码拆分为单独的文件，甚至带上自己的 Vite 插件。

欲知更多信息，请查阅我们关于[拆分客户端和服务器代码的文档](https://remix.run/docs/future/vite#splitting-up-client-and-server-code)。

## [](#cssurl-imports)`.css?url` 导入

从一开始，Remix 就提供了一种[管理 CSS 导入的替代模型](https://remix.run/docs/styling/css)。当导入 CSS 文件时，其 URL 被提供为一个字符串，用于在 `link` 标签中渲染：

```
import type { LinksFunction } from "@remix-run/node"; // or cloudflare/deno   import styles from "~/styles/dashboard.css";   export const links: LinksFunction = () => [{ rel: "stylesheet", href: styles }]; 
```

虽然 Vite 长期以来支持[将静态资源导入为 URL](https://vitejs.dev/guide/assets#importing-asset-as-url)，但如果 CSS 文件需要像[PostCSS](https://vitejs.dev/guide/features#postcss)（包括[Tailwind](https://tailwindcss.com)）、[CSS 模块](https://vitejs.dev/guide/features#css-modules)、[CSS 预处理器](https://vitejs.dev/guide/features#css-pre-processors)等进行任何处理，这种方式是行不通的。

随着最新版本 [Vite v5.1.0](https://vitejs.dev/blog/announcing-vite5-1) 的发布，通过 `.css?url` 导入语法现在可以完全支持 CSS 了：

```
import styles from "~/styles/dashboard.css?url"; 
```

## [](#cleaner-build-output)更清晰的构建输出

旧版 Remix 编译器将客户端和服务器构建到独立的目录中，可以独立配置。默认情况下，输出目录为客户端资产的 `public/build` 和服务器的 `build`。结果发现这种结构与 [Vite 的公共目录](https://vitejs.dev/guide/assets.html#the-public-directory)存在冲突。

由于 Vite 将文件从 `public` 目录复制到客户端构建目录中，并且 Remix 的客户端构建目录嵌套在 public 目录中，因此一些用户发现他们的 public 目录被递归地复制到了自身 🫠

为了解决这个问题，我们不得不稍微重新调整我们的构建输出。Remix Vite 现在有一个顶层的 `buildDirectory` 选项，默认为 `"build"`，导致了 `build/client` 和 `build/server` 目录的生成。

有趣的是，尽管我们只是实现了这个变更来修复一个 bug，但我们实际上更喜欢这种结构。根据我们收到的反馈，我们的早期采用者也是如此！

## [](#more-than-just-a-vite-plugin)不仅仅是一个 Vite 插件

我们最早的采用者直接运行 Vite CLI —— 用于本地开发的 `vite dev` 和用于生产构建的 `vite build && vite build --ssr`。由于缺少对 Vite 的自定义包装器，我们最初的不稳定发布中提到 Remix 现在只是一个 Vite 插件。

然而，随着服务器包的引入，我们无法继续沿用这种方式。当使用 `serverBundles` 选项时，会生成动态数量的服务器构建。我们曾假设能够定义多个输入和输出到 Vite 的 `ssr` 构建，但事实并非如此，因此 Remix 需要一种方式来编排整个构建过程。Vite 插件现在还提供了一个新的 `buildEnd` 钩子，因此您可以在 Remix 构建完成后运行自定义逻辑。

我们尽可能地保留了我们旧架构中的代码，通过最大化在我们的Vite插件中的代码量（我们很高兴这样做！），并为Remix CLI添加了`remix vite:dev`和`remix vite:build`命令。在Remix v3中，这些命令将成为默认的`dev`和`build`命令。

因此，虽然我们不再“只是一个Vite插件”，但可以说我们仍然*大部分*只是一个Vite插件 🙂

## [](#next-steps)下一步

现在Remix Vite已经稳定，您将开始看到我们的文档和模板默认迁移到Vite上。

就像我们最初的不稳定版本一样，我们为那些希望将现有的Remix项目迁移到Vite上的用户提供了一个[迁移指南](https://remix.run/docs/future/vite#migrating)。

请放心，旧的Remix编译器将继续在Remix v2中工作。然而，从现在开始，所有需要编译器集成的新功能和改进都将只针对Vite。

如果您在此过程中有任何反馈，请随时联系我们。我们很乐意听取您的意见！

## [](#thank-you)谢谢

感谢所有Remix社区的早期采纳者提供的反馈，发起的问题和提交的拉取请求。没有您，我们不可能走到今天。

我们还要特别感谢外部贡献者[Hiroshi Ogawa](https://github.com/hi-ogawa)，他在Remix Vite中提交了令人惊讶的[25个拉取请求](https://github.com/remix-run/remix/pulls?q=is%3Apr+is%3Amerged+label%3Avite+closed%3A%3C2024-02-21+author%3Ahi-ogawa) 🔥

同样，感谢Vite团队为我们提供如此出色的工具。我们很期待共同发展的未来。

💿⚡️🚀
