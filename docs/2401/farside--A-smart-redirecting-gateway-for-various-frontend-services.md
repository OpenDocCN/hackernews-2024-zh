<!--yml

类别：未分类

日期：2024-05-27 15:23:39

-->

# farside：各种前端服务的智能重定向网关。

> 来源：[https://sr.ht/~benbusby/farside/](https://sr.ht/~benbusby/farside/)

内容

1.  [关于](#about)

1.  [演示](#demo)

1.  [工作原理](#how-it-works)

1.  [Cloudflare](#regarding-cloudflare)

1.  [开发](#development)

    1.  [编译](#compiling)

    1.  [环境变量](#environment-variables)

### [#](#about)关于

用于 FOSS 替代前端的重定向服务。

[Farside](https://farside.link) 提供链接，自动重定向到以隐私为导向的替代前端服务的工作实例，如 Nitter、Libreddit 等。这允许用户更可靠地访问特定服务的所有可用公共实例，同时帮助更均匀地分发流量到所有实例，并避免性能瓶颈和速率限制。

Farside 还与大多数浏览器中的基本重定向扩展平滑集成。有关简单示例设置，请参阅 [维基](https://github.com/benbusby/farside/wiki/Browser-Extension)。

### [#](#demo)演示

Farside 的链接遵循以下结构：`farside.link/<service>/<path>`

例如：

^(注意：此表未列出所有可用服务。有关支持的前端的完整列表，请参阅：[https://farside.link](https://farside.link))

Farside 还接受指向“父”服务的 URL，并将重定向到适当的前端服务，例如：

### [#](#how-it-works)工作原理

应用程序通过一个内部定时的 cron 任务运行，每 5 分钟查询 [services.json](https://git.sr.ht/~benbusby/farside/tree/HEAD/services.json) 中定义的所有服务的所有实例。对于每个实例，只要实例响应时间 <5 秒且返回成功的响应代码，则将该实例添加到该特定服务的可用实例列表中。如果不是，则在下一个更新周期之前将其丢弃。

Farside 的路由非常简单，只有以下几条路由：

+   `/`

    +   应用的主页，显示每项服务的所有在线实例。

+   `/:service/*glob`

    +   将用户重定向到特定服务的工作实例的主要端点，指定路径

    +   例如：`/libreddit/r/popular` 将导航到 `<libreddit 实例 URL>/r/popular`

        +   如果提供的服务实际上是指向“父”服务的 URL（即“youtube.com”而不是“piped”或“invidious”），Farside 将确定要用于指定 URL 的正确前端。

    +   注意，并非必须输入路径。例如，`/libreddit` 仍然会将用户重定向到一个可用的 libreddit 实例。

+   `/_/:service/*glob`

    +   实现与主要 `/:service/*glob` 端点相同的重定向，但在浏览器历史记录中保留一个简短的登陆页面，以便通过导航快速跳转到实例。

    +   例如：`/_/nitter` -> nitter 实例 A ->（返回上一页）-> nitter 实例 B -> ...

    +   *注意：使用 JavaScript 保留页面历史记录*

当使用 `/:service/...` 端点请求服务时，Farside 从数据库请求工作实例列表，并从列表中随机返回一个实例，并将该实例作为新条目添加到数据库中，以便在随后的对该服务的请求中删除。例如：

用户导航到 `/nitter` 并被重定向到 `nitter.net`。下一个请求 `/nitter` 的用户将确保不会被重定向到 `nitter.net`，而是将被重定向到一个单独的（随机的）工作实例。该实例现在将取代 `nitter.net` 成为“保留”实例，并将 `nitter.net` 返回到可用 Nitter 实例列表中。

此前选择实例的“保留”操作旨在确保更好地将流量分配到每个服务的可用实例。

Farside 还为所有请求内置了 IP 速率限制，每个 IP 每秒只允许一个请求。

### [#](#regarding-cloudflare)关于 Cloudflare

使用 Cloudflare 部署的每个受支持服务的实例在使用 [farside.link](https://farside.link) 时不会被包括在内。如果您想要访问也使用 Cloudflare 的实例（以及不使用 Cloudflare 的实例），则可以使用 [cf.farside.link](https://cf.farside.link) 或自行部署 Farside 并在运行时设置 `FARSIDE_SERVICES_JSON=services-full.json`。

如果您决定使用 [cf.farside.link](https://cf.farside.link) 或使用 `services-full.json` 提供的完整实例列表，请注意 Cloudflare 会采取措施阻止使用 Tor（以及一些 VPN）的站点访客，并且他们旨在将整个网络集中到其服务后面的任务最终与 Farside 要解决的问题相悖。请谨慎使用。

### [#](#development)开发

要在不编译的情况下运行 Farside，您可以执行以下步骤：

+   安装依赖项：`mix deps.get`

+   初始化数据库内容：`FARSIDE_CRON=0 mix run -e Farside.Instances.sync`

+   运行 Farside：`mix run --no-halt`

#### [#](#compiling)编译

您可以按以下步骤创建一个独立的 Farside 应用程序。在示例中，Farside 可执行文件被复制到 `/usr/local/bin`，但可以移动到任何首选目标。请注意，该可执行文件仍依赖于构建所在机器的 C 运行时，因此如果您想要更具可移植性的二进制文件，则应在具有较旧库版本的系统上构建 Farside。

```
MIX_ENV=cli && mix deps.get && mix release
cp _build/cli/rel/bakeware/farside /usr/local/bin
sudo chmod +x /usr/local/bin/farside
farside 
```

#### [#](#environment-variables)环境变量

| 名称 | 目的 |
| --- | --- |
| FARSIDE_TEST | 如果启用，将绕过实例可用性检查，并将所有实例添加到池中。 |
| FARSIDE_PORT | 运行 Farside 的端口（默认值：`4001`） |
| FARSIDE_DATA_DIR | 用于存储实例数据的目录路径（默认值：`/tmp`） |
| FARSIDE_SERVICES_JSON | 用于选择实例的 JSON 文件（默认值：`services.json`） |
| FARSIDE_CRON | 设置为 0 以停用计划的实例可用性检查（默认值为启用）。 |
