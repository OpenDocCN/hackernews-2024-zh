<!--yml

category: 未分类

date: 2024-05-27 13:29:48

-->

# 我们如何通过 Nginx 和 Lua 管理单个用户并发性

> 来源：[https://www.browserless.io/blog/managing-concurrencies-with-nginx-and-lua](https://www.browserless.io/blog/managing-concurrencies-with-nginx-and-lua)

**本文将探讨管理单个并发限制的挑战，为我们的用户提供数千个浏览器。**

‍

有关简单负载平衡的文章数不胜数。即使我们也有一篇关于使用 [NGINX 扩展 Puppeteer 规模](https://www.browserless.io/blog/horizontally-scaling-chrome) 的文章。

> *“这就像你扩展任何东西一样，容器化并放置一个负载均衡器在前面。无聊。”* - u/express-set, reddit

因此，我想分享一些关于我们复杂定制负载平衡器的详细信息。这是关于我们如何动态限制每个用户并发连接的幕后信息。对于我们来说尤为重要，因为浏览器在 AWS 上托管是臭名昭著的 [混乱](https://www.browserless.io/blog/advanced-issues-when-managing-chrome-on-aws)，而我们的成千上万用户随时可以切换到具有不同并发限制的计划。

要有效解决这一挑战，我们必须能够实时调整和更新连接限制，以适应不同的客户计划和使用模式。

## 首先，关于我们负载平衡的一些背景

本文是关于我在 Browserless 管理并发的工作。

我们的服务提供一组托管的 Chrome、Firefox 和 WebKit 浏览器。我们有成千上万的用户，每个用户的限制至少为 25 个浏览器，具体取决于他们的计划。这些主要分布在我们的伦敦和旧金山的两个主要服务器位置之间。

这些浏览器是用于 Puppeteer、Playwright 或通过我们的 API 之一使用的。这意味着它们启动，运行脚本一段时间，然后关闭。

在极端情况下，我们的一些用户会打开一百个浏览器，进行大规模的网页抓取，然后一周什么也不做。与此同时，我们有用户以相当规律的方式全天候启动浏览器，执行生成仪表板的 PDF 导出等任务。

因此，这是一个需求范围广泛且快速变化的环境。

## **攻略**

将挑战分解为几个步骤的过程如下：

1.  **确定最佳位置 -** 认识到所有流量都通过负载均衡器经过，我确定这是实施连接限制的理想点。

1.  **嵌入式 Nginx 模块的限制 -** 初步探索显示，嵌入式 [Nginx 限制连接的模块](https://nginx.org/en/docs/http/ngx_http_limit_conn_module.html) 缺乏动态调整限制的能力，促使我们探索替代方案。

1.  **利用OpenResty和Lua脚本 -** 我选择利用OpenResty和Lua脚本的多功能性来克服内置Nginx模块的限制。这个决定是基于Lua脚本所提供的灵活性和可扩展性。

1.  **利用`lua-resty-limit-trafic`库 -** 在我的研究中，我发现了[lua-resty-limit-trafic](https://github.com/openresty/lua-resty-limit-traffic)库，这是一个用于限制流量的官方解决方案。该库提供了专门的模块来控制请求频率和并发连接，与我们的要求完全匹配。

## 实施

我基于`lua-resty-limit-trafic`库开发了一个自定义的限制器模块，包括三个主要功能：

1.  **GetAllowedConcurrency -** 此功能检索给定客户令牌的并发限制。它可以从数据库或本地缓存中获取限制。在无法检索限制的情况下，会应用默认限制。

1.  **LimitRequestsByToken -** 利用lua-resty-limit-trafic库，此功能增加了与特定令牌关联的连接数。它检查限制是否超出，并相应地抛出错误。限制是基于客户并发限制动态设置的。

1.  **LeavingConnection -** 在连接关闭时，此功能会减少与令牌关联的活动连接数。

以下代码片段展示了这些功能的实现，演示了如何利用lua-resty-limit-trafic库动态管理连接限制。

```
 local limit_conn = require 'resty.limit.conn'
local limits = {}

local default_limit = 5;

function limits.GetAllowedConcurrency(token)
    -- Here you can connect your database or a local cache to get limits, 
    -- in case you can not get limits could be usefull defining a default
    return default_limit;
end

function limits.LimitRequestsByToken(token, limitsDictionary)
    local limit = limits.GetAllowedConcurrency(token)
    local limitManager, err = limit_conn.new(limitsDictionary, limit, 0, 0.5)
    if not limitManager then
        ngx.log(ngx.ERR, "failed to instantiate a resty.limit.conn object: ", err)
        return ngx.exit(500)
    end

    local delay, err = limitManager:incoming(token, true)
    if not delay then
        if err == "rejected" then
            return ngx.exit(429)
        end
        ngx.log(ngx.ERR, "failed to limit req: ", err)
        return ngx.exit(500)
    end

    if limitManager:is_committed() then
        local ctx = ngx.ctx
        ctx.limit_conn = limitManager
        ctx.limit_conn_key = token
        ctx.limit_conn_delay = delay
    end
end

function limits.LeavingConnection()
    local ctx = ngx.ctx
    local limitManager = ctx.limit_conn
    if limitManager then
        local key = ctx.limit_conn_key
        assert(key)
        local conn, err = limitManager:leaving(key)
        if not conn then
            ngx.log(ngx.ERR, "failed to record the connection leaving ", "request: ", err)
            return
        end
    end
end

return limits 

```

要将限制器模块集成到Nginx配置中，我们需要将其导入到default.conf文件中。该模块在`access_by_lua_block`和`log_by_lua_block`指令中被调用，利用授权头作为每个客户令牌的标识符。

非常重要的一点是要创建一个`lua_shared_dict`来存储每个令牌的现有连接数。

```
 lua_shared_dict limit_conn_store 100m;
server {
    listen       80;
    server_name  localhost;

    location / {
        access_by_lua_block {
          local limits = require "limiter"
          local token = ngx.var.http_authorization
          limits.LimitRequestsByToken(token, 'limit_conn_store')
        }

        log_by_lua_block {
          local limits = require "limiter"
          limits.LeavingConnection()
        }
        echo_sleep 5.0;
        echo 'request completed';
    }

    error_page   500 502 503 504  /50x.html;
    location = /50x.html {
        root   /usr/local/openresty/nginx/html;
    }
} 

```

另外，为了避免`module not found`错误，必须在nginx.conf文件中配置`lua_package_path`指令，以指定限制器模块的位置。

```
 worker_processes  1;
events {
    worker_connections  1024;
}
http {
    lua_package_path '/YOUR_LUA_SCRIPTS_FOLDER/?.lua;;';
    include       mime.types;
    default_type  application/octet-stream;
    sendfile        on;
    keepalive_timeout  65;
    include /etc/nginx/conf.d/*.conf;
} 

```

此时，我们正在按计划限制连接。然而，在实施此解决方案之前，我们遇到了一些需要解决的挑战。

## 面临的问题

在使用限制器时，我们发现随着时间的推移，CPU和内存消耗不断升高，表明存在内存泄漏的问题。这构成了一个重大挑战，因为它需要定期重启服务以防止崩溃，促使我们对潜在问题进行详尽的调查以识别和解决。

我开始通过断开非必要的组件来进行故障排除。首先，我移除了数据库连接，但不幸的是，它并没有解决问题。接下来，我移除了本地缓存，然而问题依然存在。

作为最后的尝试，我决定测试删除导入 `local limits = require "limiter"` 并将所有代码合并到单个文件中。令人惊讶的是，这解决了问题。

我最终确定了问题。在编辑 nginx.conf 文件之前，我在 Lua 脚本中使用了 [package.path](https://www.lua.org/manual/5.1/manual.html#pdf-package.path) 来导入我的自定义 Lua 文件夹。

```
 package.path = "/usr/local/balancer/lua/?.lua;" .. package.path 

```

发生了一个重要的疏忽：每次运行脚本时，package.path 都会增加，因为我在每个请求中将我的文件夹名连接到相同的值上。因此，在几次请求后，检查 package.path 的值显示了文件夹名的累积。

```
 package.path = "/usr/local/balancer/lua/?.lua;/usr/local/balancer/lua/?.lua;/usr/local/balancer/lua/?.lua;..." 

```

最初，似乎将文件夹名连接到 package.path 是正确的方法，正如 Lua 官方文档建议的那样。然而，在我们特定的使用场景中，这种方法导致了上述问题。最终，我参考了 [lua-nginx-module](https://github.com/openresty/lua-nginx-module#lua_package_path) 文档，找到了正确导入模块的方法，并通过在 nginx.conf 文件中添加 `lua_package_path` 指令来解决了问题。

下面是一张图形表示，在实施修复前后资源消耗的变化。

另外，为了提高性能并减少数据库查询，我们引入了一个带有生存时间（TTL）机制的本地缓存，用于存储客户限制。这种优化策略有效地减少了延迟并减轻了数据库的压力。

## **How It Turned Out**

最终，新的限流器效果非常好。它现在有效地管理每个客户的并发连接，保持服务的健康和稳定，同时防止服务滥用并保持良好的工作进程。

如果您想看看它如何运行，请继续并试用一下 [Browserless 试用版](https://www.browserless.io/pricing)，并尝试使用并发浏览器。
