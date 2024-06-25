<!--yml

分类：未分类

日期：2024-05-27 14:26:38

-->

# 2024 年的新博客 | 克里斯·尼古拉斯 | Liveblocks 的开发者体验

> 来源：[`chrisnicholas.dev/blog/a-new-blog-for-2024`](https://chrisnicholas.dev/blog/a-new-blog-for-2024)

我已经很长时间没有发布博客文章了，整整两年。部分原因是之前的博客花费了太多精力来创建帖子。我需要更好的 DX。几个晚上前，在阅读了[Pedro Duarte](https://ped.ro/writing/website-refresh-2023)、[Rauno Freiberg](https://rauno.me/craft/vercel)和[Lee Robinson](https://leerob.io/blog/2023)的文章后，我感到很有启发，我决定重新开始。

所以这就是我的目标——我想要一个具有极佳 DX 的博客，尽可能地让创建帖子变得简单。我还想要*极简*，这样我就可以在移动端和桌面端使用单一设计。不要拖拖拉拉；我希望这个博客在 5 天内建立并发布！最后，我想尝试一些新的库和技术。好的，让我们开始吧。

在工作中，我大部分时间都在写作和编程，所以尝试设计是一个很好的改变。这里有一些细节。

对于头部，我的目标是创建一个动画背景，结合了*极光*、*光束*和*彩虹*。我对结果非常满意，而且这实际上是纯 CSS——尝试在下面更改`hue-rotate`以模拟动画。

显示头部背景的交互式演示

```
.aurora {
  filter: hue-rotate(0deg) saturate(7.0) blur(30px);
}
```

所有悬停在极光上的文字都充满了其底层颜色——这是一个微小的触摸，真正统一了设计。您只需简单地将`luminosity` [混合模式](https://www.ctnicholas.dev/articles/which-blend-mode)应用于文字即可实现这一点。

光度文本效果在极光上的演示。

我给`body`应用了一点点胶片颗粒。对我来说，这使页面感觉像一个一致的整体；当你向下滚动时，噪音会附着在内容上，有点像在纸上看到纹理。

一个交互式演示，允许您在页面上更改噪声级别。

```
.noise {
  opacity: 0.025;
}
```

我不建议大多数网站使用噪声，而且很容易过度使用，但我喜欢我的博客上微妙的电影效果。

我正在使用[Catalyst](https://tailwindcss.com/blog/introducing-catalyst)，这是 Tailwind 团队的一个新的 UI 工具包。很明显，这些组件的制作考虑到了使其在浅色和深色模式下都完美无缺地可访问和像素完美。

Tailwind UI Catalyst 演示

Catalyst 为您提供一组文件，可下载到您的项目中，并且您可以从这些文件中导出您的组件。[Heroicons](https://heroicons.com) 也已更新，提供了适合 Catalyst 的新图标。

```
import { Button } from "../components/Button";
import { PaperAirplaneIcon } from "@heroicons/react/16/solid";

function Component() {
 return (
 <Button type="submit" color="rose">
 <PaperAirplaneIcon />
 Send message
 </Button>
 );
}
```

如果您想要进一步自定义您的组件，您可以直接编辑下载的文件。每个文件都包含[Headless UI](https://headlessui.com/)组件，其中包含精心注释的类列表。

```
<HeadlessListBoxButton
 className={clsx(
 // Basic layout
 "group relative block w-full",

 // Focus ring
 "after:pointer-events-none after:absolute ...",

 // Disabled state
 "data-[disabled]:opacity-50 before:data-[d..."
 )}
 // ...
/>
```

Catalyst 仍处于 alpha 阶段，所以请注意，可能会出现重大变更！

当我们谈论 Tailwind 时，我使用了一个名为[TWC](https://react-twc.vercel.app/)的新库来节省构建可重用组件的时间。它接管了你已经写了无数次的样板代码（转发引用、添加`clsx`等），并将其变成了一行代码。

```
const Card = twc.div`rounded-lg border bg-slate-100 text-white shadow-sm`;

<Card ref={ref} className={["my-6", hidden && "hidden"]} {...props} />;
```

我使用[Next.js](https://nextjs.org/)构建了我的新博客，几乎每个组件都是服务器组件。感觉相当迅速，我喜欢。

#### 布局文件

你有没有注意到，在切换页面时标题动画会继续运行？这是因为它放在了一个 Next.js[布局文件](https://nextjs.org/docs/pages/building-your-application/routing/pages-and-layouts#layout-pattern)中，这意味着在路由更改时不会刷新。

在更改页面时的标题动画。

</blog/a-new-blog-for-2024/header-animation.mp4>

```
// layout.tsx
export default function Layout({ children }) {
 return (
 <>
 <HeaderAnimation />
 <main>{children}</main>
 </>
 );
}
```

#### OG 图片生成

我以前博客的一个主要痛点是手动创建每个社交媒体图片。如今，你可以在 Next.js 中[动态生成 OpenGraph 图片](https://nextjs.org/docs/app/api-reference/file-conventions/metadata/opengraph-image)，而且很容易上手。

```
// /blog/[slug]/opengraph-image.tsx
export const runtime = "edge";
export const alt = "...";
export const size = { width: 1200, height: 630 };

export default async function Image({ params }) {
 return new ImageResponse(
 (
 <div style={{ display: "flex", color: "#000" }}>
 <div>A new blog for 2024</div>
 ...
 </div>
 ),
 { ...size }
 );
}
```

我的博客使用`.mdx`文件，并在其中穿插了自定义 React 组件。我听说[Contentlayer](https://contentlayer.dev/)有很多好评，所以我试了试——使用起来非常轻松，而且真的加快了我的开发速度。更改模式非常容易，类型安全的设置也很棒。

#### 工作原理

你可以生成一个`Post`模式，例如包含`title`和`date`，然后要求它从`/posts`文件夹加载文件。

contentlayer.config.ts

```
export const Post = defineDocumentType(() => ({
 name: "Post",
 filePathPattern: `/posts/**/*.mdx`,
 contentType: "mdx",
 fields: {
 title: { type: "string", required: true },
 date: { type: "date", required: true },
 },
}));
```

接下来，在`/posts`中创建你的`.mdx`文件。

```
---
title: A new blog for 2024
date: 2024-01-02
---

It’s been a _long_ time since I’ve publis…
```

然后简单地导入你的帖子，具有完全的类型安全性，并呈现你的`mdx`内容。

```
import { allPosts, type Post } from "contentlayer/generated";
import { useMDXComponent } from "next-contentlayer/hooks";

// { title: "A new blog for 2024", ... }
const post: Post = allPosts[0];

// <p>It’s been a <em>long time</em> since I’ve publis…
export default function Page() {
 const MDXContent = useMDXComponent(post.body.code);
 return <MDXContent />;
}
```

我已经归档了以前博客的所有帖子，*ctnicholas.dev*，因为将它们转移到新站点需要太长时间，尽管它们仍然可以阅读。

我的博客还在不断完善中，还有很多我想要添加的功能，例如：

但是很容易陷入“再加一个功能…”的循环中，而最重要的是要开始运送！所以这就是——谢谢你的阅读。
