<!--yml

category: 未分类

date: 2024-05-27 13:38:15

-->

# 发布 v3.0.0 · traefik/traefik · GitHub

> 来源：[https://github.com/traefik/traefik/releases/tag/v3.0.0](https://github.com/traefik/traefik/releases/tag/v3.0.0)

**重要提示：** 请阅读[迁移指南](https://doc.traefik.io/traefik/v3.0/migration/v2-to-v3/)。

**增强功能：**

+   **[consul]** ConsulCatalog StrictChecks（[#10388](https://github.com/traefik/traefik/pull/10388) 由 [djenriquez](https://github.com/djenriquez) 提交）

+   **[docker,docker/swarm]** 拆分Docker提供者（[#9652](https://github.com/traefik/traefik/pull/9652) 由 [ldez](https://github.com/ldez) 提交）

+   **[docker,service]** 在ServersLoadBalancer上添加权重（[#10372](https://github.com/traefik/traefik/pull/10372) 由 [juliens](https://github.com/juliens) 提交）

+   **[ecs]** 添加仅保留健康ECS任务的选项（[#8027](https://github.com/traefik/traefik/pull/8027) 由 [Michampt](https://github.com/Michampt) 提交）

+   **[file]** 在SIGHUP信号上重新加载提供者文件配置（[#9993](https://github.com/traefik/traefik/pull/9993) 由 [sokoide](https://github.com/sokoide) 提交）

+   **[healthcheck]** 支持gRPC健康检查（[#8583](https://github.com/traefik/traefik/pull/8583) 由 [jjacque](https://github.com/jjacque) 提交）

+   **[healthcheck]** 为服务健康检查添加状态选项（[#9463](https://github.com/traefik/traefik/pull/9463) 由 [guoard](https://github.com/guoard) 提交）

+   **[http]** 通过HTTP获取配置时支持自定义标头（[#9421](https://github.com/traefik/traefik/pull/9421) 由 [kevinpollet](https://github.com/kevinpollet) 提交）

+   **[http3]** 将HTTP/3移出实验部分（[#9570](https://github.com/traefik/traefik/pull/9570) 由 [sdelicata](https://github.com/sdelicata) 提交）

+   **[k8s,hub]** 删除已弃用代码（[#9804](https://github.com/traefik/traefik/pull/9804) 由 [ldez](https://github.com/ldez) 提交）

+   **[k8s,k8s/gatewayapi]** 支持跨命名空间引用 / GatewayAPI ReferenceGrants（[#10346](https://github.com/traefik/traefik/pull/10346) 由 [pascal-hofmann](https://github.com/pascal-hofmann) 提交）

+   **[k8s,k8s/gatewayapi]** 支持在GatewayAPI TLS路由中使用HostSNIRegexp（[#9486](https://github.com/traefik/traefik/pull/9486) 由 [ddtmachado](https://github.com/ddtmachado) 提交）

+   **[k8s,k8s/gatewayapi]** 升级网关API到v1.0.0（[#10205](https://github.com/traefik/traefik/pull/10205) 由 [mmatur](https://github.com/mmatur) 提交）

+   **[k8s/crd,k8s]** 支持将文件路径作为Kubernetes令牌值的输入参数（[#10232](https://github.com/traefik/traefik/pull/10232) 由 [sssash18](https://github.com/sssash18) 提交）

+   **[k8s/gatewayapi]** 添加设置网关状态地址的选项（[#10582](https://github.com/traefik/traefik/pull/10582) 由 [kevinpollet](https://github.com/kevinpollet) 提交）

+   **[k8s/gatewayapi]** 切换支持实验通道（[#10435](https://github.com/traefik/traefik/pull/10435) 由 [SantoDE](https://github.com/SantoDE) 提交）

+   **[k8s/gatewayapi]** 添加设置 Gateway 状态地址的选项 ([#10582](https://github.com/traefik/traefik/pull/10582) 由 [kevinpollet](https://github.com/kevinpollet))

+   **[k8s/gatewayapi]** 在 k8s Gateway API 中添加对 HTTPRequestRedirectFilter 的支持 ([#9408](https://github.com/traefik/traefik/pull/9408) 由 [romantomjak](https://github.com/romantomjak))

+   **[k8s/gatewayapi]** 在过滤器扩展引用中处理中间件 ([#10511](https://github.com/traefik/traefik/pull/10511) 由 [youkoulayley](https://github.com/youkoulayley))

+   **[k8s/ingress,k8s/crd,k8s,k8s/gatewayapi]** 在 routerTransform 中使用 runtime.Object ([#10523](https://github.com/traefik/traefik/pull/10523) 由 [juliens](https://github.com/juliens))

+   **[k8s/ingress,k8s]** 添加选项以禁用 IngressClass 查找 ([#9281](https://github.com/traefik/traefik/pull/9281) 由 [jandillenkofer](https://github.com/jandillenkofer))

+   **[k8s/ingress,k8s]** 移除 networking.k8s.io/v1beta1 APIVersion 的支持 ([#9949](https://github.com/traefik/traefik/pull/9949) 由 [rtribotte](https://github.com/rtribotte))

+   **[logs]** 引入静态配置提示 ([#10351](https://github.com/traefik/traefik/pull/10351) 由 [rtribotte](https://github.com/rtribotte))

+   **[logs,performance]** Traefik 日志的新记录器 ([#9515](https://github.com/traefik/traefik/pull/9515) 由 [ldez](https://github.com/ldez))

+   **[logs,plugins]** 在插件 API 调用中重试 ([#9530](https://github.com/traefik/traefik/pull/9530) 由 [ldez](https://github.com/ldez))

+   **[logs,provider]** 改进提供者日志 ([#9562](https://github.com/traefik/traefik/pull/9562) 由 [ldez](https://github.com/ldez))

+   **[logs]** 改进测试日志记录断言 ([#9533](https://github.com/traefik/traefik/pull/9533) 由 [ldez](https://github.com/ldez))

+   **[marathon]** 移除 Marathon 提供者 ([#9614](https://github.com/traefik/traefik/pull/9614) 由 [rtribotte](https://github.com/rtribotte))

+   **[metrics,tracing,accesslogs]** 移除对内部资源的可观察性 ([#9633](https://github.com/traefik/traefik/pull/9633) 由 [rtribotte](https://github.com/rtribotte))

+   **[metrics,tracing]** 升级 opentelemetry 依赖项 ([#10472](https://github.com/traefik/traefik/pull/10472) 由 [mmatur](https://github.com/mmatur))

+   **[metrics]** 支持通过 Unix Socket 发送 DogStatsD 指标 ([#10199](https://github.com/traefik/traefik/pull/10199) 由 [liamvdv](https://github.com/liamvdv))

+   **[metrics]** 移除 InfluxDB v1 指标中间件 ([#9612](https://github.com/traefik/traefik/pull/9612) 由 [tomMoulard](https://github.com/tomMoulard))

+   **[metrics]** 升级 OpenTelemetry 依赖项 ([#10181](https://github.com/traefik/traefik/pull/10181) 由 [mmatur](https://github.com/mmatur))

+   **[metrics]** 支持在指标中使用 gRPC 和 gRPC-Web 协议 ([#9483](https://github.com/traefik/traefik/pull/9483) 由 [longit644](https://github.com/longit644))

+   **[中间件,accesslogs]** 记录 TLS 客户端主体（[#9285](https://github.com/traefik/traefik/pull/9285) 由 [xmessi](https://github.com/xmessi) 提交）

+   **[中间件,metrics,tracing,otel]** 添加 OpenTelemetry 跟踪和指标支持（[#8999](https://github.com/traefik/traefik/pull/8999) 由 [tomMoulard](https://github.com/tomMoulard) 提交）

+   **[中间件]** 默认禁用 Content-Type 的自动检测（[#9546](https://github.com/traefik/traefik/pull/9546) 由 [sdelicata](https://github.com/sdelicata) 提交）

+   **[中间件]** 添加 gRPC-Web 中间件（[#9451](https://github.com/traefik/traefik/pull/9451) 由 [juliens](https://github.com/juliens) 提交）

+   **[中间件]** 支持 Brotli 压缩（[#9387](https://github.com/traefik/traefik/pull/9387) 由 [glinton](https://github.com/glinton) 提交）

+   **[中间件]** 将 IPWhiteList 重命名为 IPAllowList（[#9457](https://github.com/traefik/traefik/pull/9457) 由 [wxmbugu](https://github.com/wxmbugu) 提交）

+   **[中间件,authentication,tracing]** 为跟踪添加捕获的头部选项（[#10457](https://github.com/traefik/traefik/pull/10457) 由 [rtribotte](https://github.com/rtribotte) 提交）

+   **[中间件,authentication]** 添加 `forwardAuth.addAuthCookiesToResponse`（[#8924](https://github.com/traefik/traefik/pull/8924) 由 [tgunsch](https://github.com/tgunsch) 提交）

+   **[中间件,metrics]** 稳定的 HTTP 指标 Semconv OTLP（[#10421](https://github.com/traefik/traefik/pull/10421) 由 [mmatur](https://github.com/mmatur) 提交）

+   **[中间件]** 作为不推荐使用的方式重新引入 IpWhitelist 中间件（[#10341](https://github.com/traefik/traefik/pull/10341) 由 [mmatur](https://github.com/mmatur) 提交）

+   **[中间件]** 在没有 Accept-Encoding 头部的情况下禁用 br 压缩（[#10178](https://github.com/traefik/traefik/pull/10178) 由 [robin-moser](https://github.com/robin-moser) 提交）

+   **[中间件]** 为压缩中间件实现 includedContentTypes 选项（[#10207](https://github.com/traefik/traefik/pull/10207) 由 [rjsocha](https://github.com/rjsocha) 提交）

+   **[中间件]** 添加 `rejectStatusCode` 选项到 `IPAllowList` 中间件（[#10130](https://github.com/traefik/traefik/pull/10130) 由 [jfly](https://github.com/jfly) 提交）

+   **[中间件]** 将 v2.11 合并到 v3.0（[#10426](https://github.com/traefik/traefik/pull/10426) 由 [mmatur](https://github.com/mmatur) 提交）

+   **[中间件]** 添加 ResponseCode 到 CircuitBreaker（[#10147](https://github.com/traefik/traefik/pull/10147) 由 [fahhem](https://github.com/fahhem) 提交）

+   **[nomad]** 允许空的服务（[#10375](https://github.com/traefik/traefik/pull/10375) 由 [chrispruitt](https://github.com/chrispruitt) 提交）

+   **[nomad]** 在 Nomad 提供程序中支持多个命名空间（[#9332](https://github.com/traefik/traefik/pull/9332) 由 [0teh](https://github.com/0teh) 提交）

+   **[插件]** 添加对 Traefik 的 http-wasm 插件支持（[#10189](https://github.com/traefik/traefik/pull/10189) 由 [zetaab](https://github.com/zetaab) 提交）

+   **[plugins]** 升级 http-wasm 主机到 v0.6.0 以支持使用 v0.4.0 的客户端 ([#10475](https://github.com/traefik/traefik/pull/10475) 由 [jcchavezs](https://github.com/jcchavezs))

+   **[rancher]** 移除 Rancher v1 提供者 ([#9613](https://github.com/traefik/traefik/pull/9613) 由 [tomMoulard](https://github.com/tomMoulard))

+   **[rules]** 恢复 v2 规则匹配器 ([#10339](https://github.com/traefik/traefik/pull/10339) 由 [rtribotte](https://github.com/rtribotte))

+   **[rules]** 从 HTTP 多路复用器中移除 containous/mux ([#9558](https://github.com/traefik/traefik/pull/9558) 由 [tomMoulard](https://github.com/tomMoulard))

+   **[rules]** 更新路由语法 ([#9531](https://github.com/traefik/traefik/pull/9531) 由 [skwair](https://github.com/skwair))

+   **[server]** 为 EntryPoints 添加 SO_REUSEPORT 支持 ([#9834](https://github.com/traefik/traefik/pull/9834) 由 [aofei](https://github.com/aofei))

+   **[server]** 重构服务器负载均衡器以使用 WRR ([#9431](https://github.com/traefik/traefik/pull/9431) 由 [juliens](https://github.com/juliens))

+   **[server]** 允许默认入口点定义 ([#9100](https://github.com/traefik/traefik/pull/9100) 由 [applejag](https://github.com/applejag))

+   **[sticky-session]** 支持设置黏性 cookie 的最大生存期 ([#10176](https://github.com/traefik/traefik/pull/10176) 由 [Patrick0308](https://github.com/Patrick0308))

+   **[tls,tcp,service]** 添加 TCP 服务器传输支持 ([#9465](https://github.com/traefik/traefik/pull/9465) 由 [sdelicata](https://github.com/sdelicata))

+   **[tls,service]** 在 Traefik 和后端服务器之间支持 SPIFFE mTLS ([#9394](https://github.com/traefik/traefik/pull/9394) 由 [jlevesy](https://github.com/jlevesy))

+   **[tls]** 添加 Tailscale 证书解析器 ([#9237](https://github.com/traefik/traefik/pull/9237) 由 [kevinpollet](https://github.com/kevinpollet))

+   **[tls]** 支持 SNI 路由与 Postgres STARTTLS 连接 ([#9377](https://github.com/traefik/traefik/pull/9377) 由 [rtribotte](https://github.com/rtribotte))

+   **[tracing,otel]** 迁移到 opentelemetry ([#10223](https://github.com/traefik/traefik/pull/10223) 由 [zetaab](https://github.com/zetaab))

+   **[tracing]** 支持 OTEL_PROPAGATORS 配置追踪传播 ([#10465](https://github.com/traefik/traefik/pull/10465) 由 [youkoulayley](https://github.com/youkoulayley))

+   **[webui,middleware,k8s/gatewayapi]** 支持 RequestHeaderModifier 过滤器 ([#10521](https://github.com/traefik/traefik/pull/10521) 由 [rtribotte](https://github.com/rtribotte))

+   **[webui]** 在 webui 的列表和详情页面中添加路由器优先级 ([#9004](https://github.com/traefik/traefik/pull/9004) 由 [bendre90](https://github.com/bendre90))

+   重新引入丢弃的 v2 动态配置 ([#10355](https://github.com/traefik/traefik/pull/10355) 由 [rtribotte](https://github.com/rtribotte))

+   移除废弃的选项 ([#9527](https://github.com/traefik/traefik/pull/9527) 由 [sdelicata](https://github.com/sdelicata))

**Bug 修复：**

+   **[consul,tls]** 为 Consul Connect TCP 服务启用了 TLS（[#10140](https://github.com/traefik/traefik/pull/10140) 由 [rtribotte](https://github.com/rtribotte) 提交）

+   **[docker]** 修复了注释中的结构名称（[#10503](https://github.com/traefik/traefik/pull/10503) 由 [hishope](https://github.com/hishope) 提交）

+   **[k8s/crd,k8s]** 为 CRD 添加了缺失的断路器响应代码（[#10625](https://github.com/traefik/traefik/pull/10625) 由 [ldez](https://github.com/ldez) 提交）

+   **[k8s/crd,k8s]** 删除 Kubernetes CRD 提供程序关于支持版本的警告（[#10414](https://github.com/traefik/traefik/pull/10414) 由 [nmengin](https://github.com/nmengin) 提交）

+   **[logs]** 避免累积发送匿名使用日志（[#10579](https://github.com/traefik/traefik/pull/10579) 由 [mmatur](https://github.com/mmatur) 提交）

+   **[logs]** 将 Traefik 命令错误日志级别改为错误级别（[#9569](https://github.com/traefik/traefik/pull/9569) 由 [tomMoulard](https://github.com/tomMoulard) 提交）

+   **[logs]** 修复了日志级别（[#9545](https://github.com/traefik/traefik/pull/9545) 由 [ldez](https://github.com/ldez) 提交）

+   **[metrics]** 修复了 OpenTelemetry 指标（[#9962](https://github.com/traefik/traefik/pull/9962) 由 [rtribotte](https://github.com/rtribotte) 提交）

+   **[metrics]** 修复了 OpenTelemetry 服务名称（[#9619](https://github.com/traefik/traefik/pull/9619) 由 [tomMoulard](https://github.com/tomMoulard) 提交）

+   **[metrics]** 修复了开放连接指标（[#9656](https://github.com/traefik/traefik/pull/9656) 由 [mpl](https://github.com/mpl) 提交）

+   **[metrics]** 移除配置重新加载失败指标（[#9660](https://github.com/traefik/traefik/pull/9660) 由 [rtribotte](https://github.com/rtribotte) 提交）

+   **[metrics]** 修复了 OpenTelemetry 单元测试（[#10380](https://github.com/traefik/traefik/pull/10380) 由 [mmatur](https://github.com/mmatur) 提交）

+   **[metrics]** 修复了 ServerUp 指标（[#9534](https://github.com/traefik/traefik/pull/9534) 由 [kevinpollet](https://github.com/kevinpollet) 提交）

+   **[middleware,authentication,metrics,tracing]** 对齐了 OpenTelemetry 追踪和指标配置（[#10404](https://github.com/traefik/traefik/pull/10404) 由 [rtribotte](https://github.com/rtribotte) 提交）

+   **[middleware]** 修复了当禁用压缩时 brotli 响应状态码的问题（[#10396](https://github.com/traefik/traefik/pull/10396) 由 [rtribotte](https://github.com/rtribotte) 提交）

+   **[middleware]** 允许短的健康检查间隔和长的超时时间（[#9832](https://github.com/traefik/traefik/pull/9832) 由 [kevinmcconnell](https://github.com/kevinmcconnell) 提交）

+   **[middleware]** 修复了 GrpcWeb 中间件，以在转换为普通 gRPC 消息后清除 ContentLength（[#9782](https://github.com/traefik/traefik/pull/9782) 由 [CleverUnderDog](https://github.com/CleverUnderDog) 提交）

+   **[provider,tls]** 将 tscert 依赖项升级到 28a91b69a046（[#10668](https://github.com/traefik/traefik/pull/10668) 由 [kevinpollet](https://github.com/kevinpollet) 提交）

+   **[规则]** 重新设计主机和HostRegexp匹配器（由[tomMoulard](https://github.com/tomMoulard)提交，[#9559](https://github.com/traefik/traefik/pull/9559)）

+   **[规则]** 支持匹配器v2中路径/路径前缀的正则表达式（由[youkoulayley](https://github.com/youkoulayley)提交，[#10546](https://github.com/traefik/traefik/pull/10546)）

+   **[粘性会话, 服务器]** 为wrr负载均衡器粘性cookie设置sameSite字段（由[sunyakun](https://github.com/sunyakun)提交，[#10066](https://github.com/traefik/traefik/pull/10066)）

+   **[tcp]** 在Postgres StartTLS钩子中查看第一个字节时，不记录EOF或超时错误（由[rtribotte](https://github.com/rtribotte)提交，[#9663](https://github.com/traefik/traefik/pull/9663)）

+   **[tls,服务器]** 计算https转发TLS路由的优先级（由[rtribotte](https://github.com/rtribotte)提交，[#10288](https://github.com/traefik/traefik/pull/10288)）

+   **[tls,服务]** 强制执行默认服务器传输SPIFFE配置（由[jlevesy](https://github.com/jlevesy)提交，[#9444](https://github.com/traefik/traefik/pull/9444)）

+   **[webui]** 检测仪表板资产的内容类型（由[tomMoulard](https://github.com/tomMoulard)提交，[#9622](https://github.com/traefik/traefik/pull/9622)）

+   **[webui]** 添加缺失的Docker Swarm标志（由[ldez](https://github.com/ldez)提交，[#10529](https://github.com/traefik/traefik/pull/10529)）

+   **[webui]** 修复：检测仪表板内容类型（由[ldez](https://github.com/ldez)提交，[#9594](https://github.com/traefik/traefik/pull/9594)）

+   修复使用空格分隔键和值之间的标志的退化（由[ldez](https://github.com/ldez)提交，[#10445](https://github.com/traefik/traefik/pull/10445)）

**文档:**

**杂项:**
