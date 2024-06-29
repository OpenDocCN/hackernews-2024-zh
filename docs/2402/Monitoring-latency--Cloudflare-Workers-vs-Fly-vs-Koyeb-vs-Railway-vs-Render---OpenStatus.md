<!--yml

类别: 未分类

日期: 2024-05-27 14:59:42

-->

# 监控延迟: Cloudflare Workers vs Fly vs Koyeb vs Railway vs Render | OpenStatus

> 来源：[https://www.openstatus.dev/blog/monitoring-latency-cf-workers-fly-koyeb-raylway-render](https://www.openstatus.dev/blog/monitoring-latency-cf-workers-fly-koyeb-raylway-render)
> 
> ⚠️ 我们使用每个提供商的默认设置，并进行数据中心到数据中心的请求。真实世界的应用程序结果将会有所不同。⚠️

想知道哪家云服务提供商的延迟最低？

在这篇文章中，我使用 [OpenStatus](https://www.openstatus.dev) 比较了 [Cloudflare Workers](#cloudflare-workers), [Fly](#flyio), [Koyeb](#koyeb), [Railway](#railway) 和 [Render](#render) 的延迟。

我在每个提供商提供的最便宜或免费的层次部署了应用程序。

为了这个测试，我使用了一个基本的 [Hono](https://hono.dev) 服务器，返回一个简单的文本响应。

```
const  app  =  new  Hono();
app.use("*", logger());

app.use("*", poweredBy());

app.get("/", (c) => {
 return c.text(
 "Just return the desired http status code, e.g. /404 🤯 \nhttps://www.openstatus.dev",
 );
});
```

您可以在[这里](https://github.com/openstatusHQ/status-code)找到代码，它是开源的 😉。

OpenStatus 每 **10 分钟**从位于阿姆斯特丹、亚伯汗、香港、约翰内斯堡、圣保罗和悉尼的 **6 个位置** 监控我们的端点。

这是测试我们自己产品并改进它的好方法。

让我们分析过去两周的数据。

## Cloudflare workers[](#cloudflare-workers)

Cloudflare Workers 是由 Cloudflare 提供的无服务器平台。它允许您使用 JavaScript/TypeScript 构建新应用程序。您可以免费部署超过 275 个网络位置上的 100 个 worker 脚本。

### Latency metrics[](#latency-metrics)

### Timing metrics[](#timing-metrics)

| 区域 | DNS (ms) | 连接 (ms) | TLS 握手 (ms) | TTFB (ms) | 传输 (ms) |
| --- | --- | --- | --- | --- | --- |
| AMS | 17 | 2 | 17 | 27 | 0 |
| GRU | 38 | 2 | 13 | 28 | 0 |
| HKG | 19 | 2 | 13 | 29 | 0 |
| IAD | 24 | 1 | 14 | 30 | 0 |
| JNB | 123 | 168 | 182 | 185 | 0 |
| SYD | 51 | 1 | 11 | 25 | 0 |

我可以注意到，约翰内斯堡的延迟大约是其他监视器的十倍。

通过 Cloudflare 请求，我可以获取处理请求的 worker 的位置，在响应头中使用 `Cf-ray`。

| 检查区域 | 工作区域 | 请求次数 |
| --- | --- | --- |
| HKG | HKG | 1831 |
| SYD | SYD | 1831 |
| AMS | AMS | 1831 |
| IAD | IAD | 1831 |
| GRU | GRU | 1791 |
| GRU | GIG | 40 |
| JNB | AMS | 741 |
| JNB | MUC | 4 |
| JNB | HKG | 5 |
| JNB | SIN | 6 |
| JNB | NRT | 8 |
| JNB | EWR | 10 |
| JNB | CDG | 82 |
| JNB | FRA | 276 |
| JNB | LHR | 699 |
| JNB | AMS | 741 |

我可以看到从 JNB 发出的所有请求从未路由到附近的数据中心。

除了约翰内斯堡的奇怪路由错误外，Cloudflare 的 workers 在全球范围内速度很快。

我没有经历过任何冷启动问题。

## Fly.io[](#flyio)

Fly.io 简化了全球范围内部署和运行服务器端应用程序。开发者可以在全球用户附近部署其应用程序，以实现低延迟和高性能。它使用轻量级的 Firecracker VM 来轻松部署 Docker 镜像。

### 延迟指标[](#延迟指标-1)

### 时间指标[](#时间指标-1)

| 地区 | DNS（毫秒） | 连接（毫秒） | TLS 握手（毫秒） | TTFB（毫秒） | 传输（毫秒） |
| --- | --- | --- | --- | --- | --- |
| AMS | 6 | 1 | 8 | 1469 | 0 |
| GRU | 5 | 0 | 4 | 1431 | 0 |
| HKG | 4 | 0 | 5 | 1473 | 0 |
| IAD | 3 | 0 | 5 | 1470 | 0 |
| JNB | 24 | 0 | 5 | 1423 | 0 |
| SYD | 3 | 0 | 3 | 1489 | 0 |

DNS 很快，我们的检测器正尝试连接同一数据中心的地区，但我们的机器冷启动使我们变慢，导致 TTFB 高。

这是我们在 Fly.io 上的配置：

```
app = 'statuscode'
primary_region = 'ams'

[build]
 dockerfile = "./Dockerfile"

[http_service]
 internal_port = 3000
 force_https = true
 auto_stop_machines = true
 auto_start_machines = true
 min_machines_running = 0
 processes = ['app']

[[vm]]
 cpu_kind = 'shared'
 cpus = 1
 memory_mb = 256
```

我们服务器的主要地区是阿姆斯特丹，fly 实例在一段不活动时间后会被暂停。

机器启动缓慢，日志显示启动时间为 `1.513643778s`。

```
2024-02-14T11:24:16.107 proxy[286560ea703108] ams [info] Starting machine

2024-02-14T11:24:16.322 app[286560ea703108] ams [info] [ 0.035736] PCI: Fatal: No config space access function found

2024-02-14T11:24:16.533 app[286560ea703108] ams [info] INFO Starting init (commit: bfa79be)...

2024-02-14T11:24:16.546 app[286560ea703108] ams [info] INFO Preparing to run: `/usr/local/bin/docker-entrypoint.sh bun start` as root

2024-02-14T11:24:16.558 app[286560ea703108] ams [info] INFO [fly api proxy] listening at /.fly/api

2024-02-14T11:24:16.565 app[286560ea703108] ams [info] 2024/02/14 11:24:16 listening on [fdaa:3:2ef:a7b:10c:3c9a:5b4:2]:22 (DNS: [fdaa::3]:53)

2024-02-14T11:24:16.611 app[286560ea703108] ams [info] $ bun src/index.ts

2024-02-14T11:24:16.618 runner[286560ea703108] ams [info] Machine started in 460ms

2024-02-14T11:24:17.621 proxy[286560ea703108] ams [info] machine started in 1.513643778s

2024-02-14T11:24:17.628 proxy[286560ea703108] ams [info] machine became reachable in 7.03669ms 
```

#### OpenStatus 生产指标[](#openstatus-生产指标)

如果您更新您的 fly.toml 文件以包含以下内容，则可以实现零冷启动并实现更好的延迟。

```
 min_machines_running = 1 
```

这是我们在 Fly.io 上部署的生产服务器数据。

> 我们在生产中使用 Fly.io，并且机器从不休眠，效果更好。

## Koyeb[](#koyeb)

Koyeb 是一个开发者友好的无服务器平台，允许全球应用部署，无需运维、服务器或基础设施管理。Koyeb 提供免费的入门计划，包括一个 Web 服务、一个数据库服务。该平台专注于开发者的部署简易性和可扩展性。

### 延迟指标[](#延迟指标-2)

### 时间指标[](#时间指标-2)

| 地区 | DNS（毫秒） | 连接（毫秒） | TLS 握手（毫秒） | TTFB（毫秒） | 传输（毫秒） |
| --- | --- | --- | --- | --- | --- |
| AMS | 50 | 2 | 17 | 107 | 0 |
| GRU | 139 | 65 | 75 | 407 | 0 |
| HKG | 48 | 2 | 13 | 321 | 0 |
| IAD | 35 | 1 | 12 | 129 | 0 |
| JNB | 298 | 1 | 11 | 720 | 0 |
| SYD | 97 | 1 | 10 | 711 | 0 |

请求头显示我们的请求均未缓存。它们包含 `cf-cache-status: dynamic`。Cloudflare 处理 Koyeb 边缘层。[https://www.koyeb.com/blog/building-a-multi-region-service-mesh-with-kuma-envoy-anycast-bgp-and-mtls](https://www.koyeb.com/blog/building-a-multi-region-service-mesh-with-kuma-envoy-anycast-bgp-and-mtls)

我们的请求遵循以下路线：

```
Cf workers -> koyeb Global load balancer -> koyeb backend 
```

让我们看看我们命中了 cf workers 的地方

| 检测地区 | Workers 地区 | 请求数量 |
| --- | --- | --- |
| AMS | AMS | 1866 |
| GRU | GRU | 504 |
| GRU | IAD | 38 |
| GRU | MIA | 688 |
| GRU | EWR | 337 |
| GRU | CIG | 299 |
| HKG | HKG | 1866 |
| IAD | IAD | 1866 |
| JNB | JNB | 1861 |
| JNB | AMS | 1 |
| SYD | SYD | 1866 |

我们命中了 Koyeb 全局负载均衡器地区：

| 检测地区 | Koyeb 全局负载均衡器 | 请求数量 |
| --- | --- | --- |
| AMS | FRA1 | 1866 |
| GRU | WAS1 | 1866 |
| HKG | SIN1 | 1866 |
| IAD | WAS1 | 1866 |
| JNB | PAR1 | 4 |
| JNB | SIN1 | 1864 |
| JNB | FRA1 | 1 |
| JNB | SIN1 | 1866 |

我们已在法兰克福数据中心部署了我们的应用。

## 铁路[](#railway)

铁路是一个云平台，专为构建、部署和监控应用程序而设计，无需平台工程师。它通过提供无缝部署和监控能力简化了应用程序开发过程。

### 延迟指标[](#latency-metrics-3)

### 时间指标[](#timing-metrics-3)

| Region | DNS (ms) | Connection (ms) | TLS Handshake (ms) | TTFB (ms) | Transfert (ms) |
| --- | --- | --- | --- | --- | --- |
| AMS | 9 | 21 | 18 | 158 | 0 |
| GRU | 14 | 115 | 127 | 178 | 0 |
| HKG | 8 | 45 | 54 | 225 | 0 |
| IAD | 7 | 2 | 14 | 65 | 0 |
| JNB | 18 | 193 | 178 | 319 | 0 |
| SYD | 21 | 108 | 105 | 280 | 0 |

标头不提供任何信息。

铁路正在使用 Google Cloud Platform。它是唯一一个在免费计划中不允许我们选择特定区域的服务。我们的测试应用将位于 `us-west1` 俄勒冈州波特兰市。我们可以看到 IAD 的延迟是最低的。

默认情况下，我们的应用没有缩减到 0\. 它总是在运行。我们没有任何冷启动。

## 渲染[](#render)

渲染是一个平台，简化了部署和扩展 Web 应用程序和服务。它提供自动化 SSL、自动扩展、对流行框架的本地支持以及从 Git 进行一键部署的功能。该平台专注于简易性和开发者的生产力。

### 延迟指标[](#latency-metrics-4)

### 时间指标[](#timing-metrics-4)

| Region | DNS (ms) | Connection (ms) | TLS Handshake (ms) | TTFB (ms) | Transfert (ms) |
| --- | --- | --- | --- | --- | --- |
| AMS | 20 | 2 | 7 | 107 | 0 |
| GRU | 61 | 2 | 6 | 407 | 0 |
| HKG | 76 | 2 | 6 | 321 | 0 |
| IAD | 15 | 1 | 5 | 129 | 0 |
| JNB | 36 | 161 | 167 | 720 | 0 |
| SYD | 103 | 1 | 4 | 711 | 0 |

标头不提供任何信息。

我们已在法兰克福数据中心部署了我们的应用。

根据渲染文档，免费套餐在 15 分钟不活动后会关闭服务。然而，我们的应用每 10 分钟被监控访问一次。我们永远不应该缩减到 0。

```
Render spins down a Free web service that goes 15 minutes without receiving inbound traffic. Render spins the service back up whenever it next receives a request to process. 
```

我认为失败是由于我们应用的冷启动引起的。我们有一个默认的超时时间为 30 秒，而渲染应用程序需要最多 50 秒才能启动。我们可能已经达到了冷热启动之间的转折点。

## 结论[](#conclusion)

这是我们测试的结果：

| 提供商 | 正常运行时间 | Ping 失败 | 总 Ping 次数 | 平均延迟 (ms) | P75 (ms) | P90 (ms) | P95 (ms) | P99 (ms) |
| --- | --- | --- | --- | --- | --- | --- | --- | --- |
| CF Workers | 100 | 0 | 10,956 | 182 | 138 | 690 | 778 | 991 |
| Fly.io | 100 | 0 | 10,952 | 1,471 | 1,514 | 1,555 | 1,626 | 2,547 |
| Koyeb | 100 | 0 | 10,955 | 536 | 738 | 881 | 1,013 | 1,525 |
| 铁路 | 99.991 | 1 | 10,955 | 381 | 469 | 653 | 661 | 850 |
| Render | 99.89 | 12 | 10,946 | 451 | 447 | 591 | 707 | 902 |

如果你注重低延迟，Cloudflare Workers 是快速全球性能的最佳选择，没有冷启动问题。它们能高效地将你的应用部署到全球各地。

对于多区域部署，请考虑 Koyeb 和 Fly.io。

对于特定区域的部署，Railway 和 Render 是不错的选择。

选择云服务提供商不仅涉及延迟问题，还要考虑用户体验和定价。

我们在生产环境中使用 Fly.io，并对其感到满意。

#### [Vercel](#vercel)

我没有在这次测试中包括 Vercel。但我们有一篇博客文章比较 Vercel 的无服务器与边缘与无服务器。你可以在[这里](https://www.openstatus.dev/blog/monitoring-latency-vercel-edge-vs-serverless)找到它。

如果你想监控你的 API 或网站，请在[OpenStatus](https://www.openstatus.dev/app/sign-up?ref=blog-monitoring)上创建一个账户。
