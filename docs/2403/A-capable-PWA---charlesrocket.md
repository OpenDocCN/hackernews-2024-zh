<!--yml

类别：未分类

日期：2024-05-29 12:46:35

-->

# 一款强大的PWA • charlesrocket

> 来源：[https://failsafe.monster/posts/capable-pwa/](https://failsafe.monster/posts/capable-pwa/)

2024年3月28日

# 转换为渐进式Web应用程序

阅读时间6分钟

更新于 2024年4月2日

## [#](#问题)问题

我不是**JavaScript**的粉丝。但是我之前已经开始使用**Mozilla**提供的一些[服务工作线程](https://developer.mozilla.org/en-US/docs/Web/API/Service_Worker_API/Using_Service_Workers)示例，而[PWA](https://wikipedia.org/wiki/Progressive_web_app)已被证明非常有效。所以，让我们开始吧。

### [#](#静态部署)静态部署

我的第一个实现是从由[Zola](https://github.com/getzola/zola)模板生成的专用页面中提取缓存列表，该模板使用宏从分类、页面等中提取资产。但是除了需要过滤和丢弃大量数据之外，专门为此设置一个页面只是丑陋的。这是使用`fetch()`使其工作的唯一方式。而且我还得额外添加一个`zola build`。

**Zola**目前尚无法填充非HTML文件，并且我无法为了单个服务工作线程事件添加额外步骤而辩解添加**NPM**/等等。因此，需要一种新的方法。

### [#](#外部库)外部库

[Workbox](https://developer.chrome.com/docs/workbox)或[sw-tools](https://github.com/veiss-com/sw-tools)库可能会解决所有问题，但这太简单了。既然我无论如何都必须维护**JavaScript**，那就开始吧。

### [#](#可移植性)可移植性

*哎呀*

## [#](#服务工作线程)服务工作线程

解决方案是**缓存优先**的服务工作线程策略，并回退到离线模式。这感觉是最有效的方法。它不需要外部依赖或*额外步骤*。我可以尝试处理网络请求，但`timeout`听起来已经太慢了，也许下次吧。

### [#](#策略)策略

我决定移除硬编码/动态缓存列表，并安装一个备用页面。

```
oninstall = (event) => {  event.waitUntil( (async () => { const cache = await caches.open(cacheName); await cache.add("/offline/"); console.log("Service worker added offline page"); })(), ); }; 
```

其余内容是“按需缓存”——服务工作线程过滤有用的请求并将其写入缓存。这样，在第一次页面导航期间保存关键资源，不会出现奇怪的情况。如果请求失败（无网络），将提供一个**离线**页面。

```
onfetch = (event) => {  console.log("Service worker fetching", event.request.url); event.respondWith( caches.open(cacheName).then((cache) => { return cache .match(event.request) .then((response) => { if (response) { console.log("Service worker found response in cache:", response); return response; }   console.log( "No response for %s found in service worker cache. Fetching " + "from network", event.request.url, );   return fetch(event.request.clone()).then((response) => { console.log( "Service worker got response for %s from network: %O", event.request.url, response, );   if (response.status < 400) { console.log("Caching the response to", event.request.url); cache.put(event.request, response.clone()); } else { console.log("Service worker not caching the response to", event.request.url); }   return response; }).catch(() => caches.match("/offline/")); }) .catch((error) => { console.error("Error in service worker fetch handler:", error); throw error; }); }), ); }; 
```

站点的静态资产由我的**Zola** [主题](https://github.com/charlesrocket/halve-z)进行哈希处理，因此这种策略非常合适。

### [#](#缓存)缓存

通过`cacheName`进行清理 - 所有先前（旧）的缓存记录在服务工作线程激活期间被清除，保持浏览器环境的清洁。

```
onactivate = (event) =>  {  event.waitUntil( (async () => { const keys = await caches.keys(); return keys.map(async (cache) => { if(cache !== cacheName) { console.log('Removing old service worker cache '+cache); return await caches.delete(cache); } }) })() ) }; 
```

尽管如此，我希望找到一个好方法来“过期”缓存记录，仅依赖硬编码的缓存名称可能会成为一个问题。

### [#](#重新验证)重新验证

为处理“过期”资源，我将`fetch`事件切换到**陈旧但仍重新验证**策略：

```
onfetch = (event) => {  console.log("Service worker fetching", event.request.url); event.respondWith(caches.open(cacheName).then((cache) => { return cache.match(event.request).then((cachedResponse) => { const fetchedResponse = fetch(event.request).then((networkResponse) => { if (networkResponse.status < 400) { console.log("Caching the response to", event.request.url); cache.put(event.request, networkResponse.clone()); } else { console.log("Service worker not caching the response to", event.request.url); }   return networkResponse; }).catch(() => caches.match("/offline/"));   return cachedResponse || fetchedResponse; }); })); }; 
```

### [#](#预缓存)预缓存

在确定缓存事件后，我想要正确支持**离线模式**。这种标准方法是使用后台同步API。快速检查表明，这是一个挑剔的解决方案，并且支持非常有限。光是这一点就足以寻找解决方法了。我从头开始。

首先，我需要生成缓存列表，因此我直接在HTML `&LThead>`中应用了我的宏并直接应用其逻辑，以便在链接服务工作者的加载器脚本时使用具有`data-cache`标签属性的输出。

第二，我需要一种方法来让缓存列表与服务工作者“同步”。搜索给我带来了`postMessage()`服务工作者[方法](https://developer.mozilla.org/en-US/docs/Web/API/ServiceWorker/postMessage)，它“向工作者发送消息”。太棒了。为了在另一侧捕获消息，需要实现`message`事件：

```
onmessage = (event) => {  console.log("I am the service worker"); }; 
```

现在，什么阻止我在服务工作者安装期间重复我一直在做的事情？我在服务工作者激活后发送了一条消息，检查了请求类型，并开始使用**消息**中的URL填充缓存。奏效了。

```
onmessage = (event) => {  if (event.data.type === "PRECACHE") { const data = event.data.payload; console.log("Service worker started precache", data); event.waitUntil( (async () => { const cache = await caches.open(cacheName); await cache.addAll(data) .catch((error) => console.log("Service worker failed precache", error)); })(), ); } }; 
```

缓存已满，所有资源都包括在内，我在绝对/相对链接之间没有问题（尽管这可能并非一个好的“特性”）。硬编码的缓存列表与关键资产重新引入，以及离线页面，所有这些在安装期间被缓存。我还开始在安装后仅请求预缓存，以避免冗余获取：

```
const data = new String(document.currentScript.getAttribute('data-cache')); const precacheList = data.split(" "); const registerServiceWorker = async () => {  if ("serviceWorker" in navigator) { try { const registration = await navigator.serviceWorker.register("/sw.js", { scope: "/", });   if (registration.installing) { console.log("Service worker installing"); navigator.serviceWorker.ready.then((registration) => { console.log("Service worker requesting precache"); registration.active.postMessage({ payload: precacheList, type: "PRECACHE", }); });   } else if (registration.waiting) { console.log("Service worker installed"); } else if (registration.active) { console.log("Service worker active"); }   } catch (error) { console.error("Service worker registration failed", error); } } };   registerServiceWorker(); 
```

这个设置提供了一个完全支持离线的网站。服务工作者在安装过程中部署关键文件，然后预缓存其他所有内容。

## [#](#Conclusion)结论

看起来我只是为了避免接触**Halve-Z**中的CSS才去做任何事情。尽管如此，这是一个很好的练习。我建立了一个简单而功能强大的**PWA**，而不会危及工作流程或用户体验。所有代码都可以在主题的拉取请求[#22](https://github.com/charlesrocket/halve-z/pull/22)、[#23](https://github.com/charlesrocket/halve-z/pull/23)和[#24](https://github.com/charlesrocket/halve-z/pull/24)中找到。
