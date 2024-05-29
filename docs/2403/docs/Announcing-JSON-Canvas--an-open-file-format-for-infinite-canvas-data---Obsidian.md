<!--yml
category: 未分类
date: 2024-05-27 14:49:39
-->

# Announcing JSON Canvas: an open file format for infinite canvas data - Obsidian

> 来源：[https://obsidian.md/blog/json-canvas/](https://obsidian.md/blog/json-canvas/)

Today we’re excited to announce that the [Obsidian Canvas](https://obsidian.md/canvas) file format is now called [JSON Canvas](https://jsoncanvas.org/) and has its own site, specification, and open source resources at [jsoncanvas.org](https://jsoncanvas.org/).

JSON Canvas can be implemented freely as an import, export, and storage format for [any app or tool](https://jsoncanvas.org/docs/apps/). All the resources associated with JSON Canvas are open source under the MIT license, and can be found [on GitHub](https://github.com/obsidianmd/jsoncanvas).

For the [release](https://obsidian.md/changelog/2022-12-05-desktop-v1.1.0/) of Obsidian Canvas we created the `.canvas` format with an open spec. We created this format because we felt it was essential to follow the [principles](https://obsidian.md/about) that have guided us since the start. Your Obsidian data should always be stored locally, accessible offline, completely in your control, in open file formats that are [easy to retrieve and read](https://stephango.com/file-over-app). For notes we achieve this by using plain text `.md` files with Markdown syntax, a widely supported format. However, infinite canvas data does not yet have a similarly established format.

While infinite canvas tools are not new, they have been quickly [growing in popularity](https://infinitecanvas.tools/). The JSON Canvas format was created in hopes of providing longevity, readability, interoperability, and extensibility to data created with infinite canvas apps. The format is designed to be easy to parse and give users ownership over their data.

The JSON Canvas spec is currently at [version 1.0](https://jsoncanvas.org/spec/1.0). The spec is relatively conservative, it does not support every feature that canvas apps may want to implement. However we think it is a useful starting point to build upon, and we plan to continue improving on it over time.