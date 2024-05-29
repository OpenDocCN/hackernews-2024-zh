<!--yml
category: 未分类
date: 2024-05-27 14:44:58
-->

# What are React Server Components?

> 来源：[https://www.builder.io/blog/why-react-server-components](https://www.builder.io/blog/why-react-server-components)

 Server Components represent a new type of React component specifically designed to operate exclusively on the server. And unlike client components, their code stays on the server and is never downloaded to the client. This design choice offers multiple benefits to React apps. Let's take a closer look at these benefits.

**Zero-bundle sizes**

First, in terms of bundle sizes, Server Components do not send code to the client, allowing large dependencies to remain server-side. This benefits users with slower internet connections or less capable devices by eliminating the need to download, parse, and execute JavaScript for these components. Additionally, it removes the hydration step, speeding up app loading and interaction.

**Direct access to server-side resources**

Second, by having direct backend access to server-side resources like databases or file systems, Server Components enable efficient data fetching and rendering without needing additional client-side processing. Leveraging the server's computational power and proximity to data sources, they manage compute-intensive rendering tasks and send only interactive pieces of code to the client.

**Enhanced security**

Third, Server Components' exclusive server-side execution enhances security by keeping sensitive data and logic, including tokens and API keys, away from the client-side.

**Improved data fetching**

Fourth, Server Components enhance data fetching efficiency. Typically, when fetching data on the client-side using `useEffect`, a child component cannot begin loading its data until the parent component has finished loading its own. This sequential fetching of data often leads to poor performance.

The main issue is not the round trips themselves, but that these round trips are made from the client to the server. Server Components enable applications to shift these sequential round trips to the server side. By moving this logic to the server, request latency is reduced, and overall performance is improved, eliminating client-server waterfalls.

**Caching**

Fifth, rendering on the server enables caching of the results, which can be reused in subsequent requests and across different users. This approach can significantly improve performance and reduce costs by minimizing the amount of rendering and data fetching required for each request.

**Faster initial page load and First Contentful Paint**

Sixth, Initial Page Load and First Contentful Paint (FCP) are significantly improved with Server Components. By generating HTML on the server, pages render immediately without the delay of downloading, parsing, and executing JavaScript.

**Improved SEO**

Seventh, regarding Search Engine Optimization (SEO), the server-rendered HTML is fully accessible to search engine bots, enhancing the indexability of your pages.

**Efficient streaming**

Lastly, there's streaming. Server Components allows the rendering process to be divided into manageable chunks, which are then streamed to the client as soon as they are ready. This approach allows users to start seeing parts of the page earlier, eliminating the need to wait for the entire page to finish rendering on the server.

Here’s an example of a `ProductList` page server component: