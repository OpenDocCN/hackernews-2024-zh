<!--yml
category: 未分类
date: 2024-05-27 13:29:48
-->

# How we manage individual user concurrencies with Nginx and Lua

> 来源：[https://www.browserless.io/blog/managing-concurrencies-with-nginx-and-lua](https://www.browserless.io/blog/managing-concurrencies-with-nginx-and-lua)

**In this article we’ll look at the challenges of managing individual concurrency limits, hosting thousands of browsers for our users.**

‍

There’s countless articles about straightforward load balancing. Even we have one about using [NGINX for scaling Puppeteer](https://www.browserless.io/blog/horizontally-scaling-chrome).

> *“It’s the same way you scale anything, containerize it and put a load balancer in front. Boring.”* - u/express-set, reddit

So, I wanted to share details about our complex custom load balancers. This is a behind the scenes info about how we dynamically limit concurrent connections per user. It’s particularly important for us since browsers are notoriously [messy to host](https://www.browserless.io/blog/advanced-issues-when-managing-chrome-on-aws), and our thousands of users can change to plans with different concurrency limits at any time.

To address this challenge effectively, we must be able to adjust and update connection limits on-the-fly to accommodate different customer plans and usage patterns.

## First, some context about what we’re load balancing

This article is about my work managing concurrencies for Browserless.

Our service provides a pool of hosted Chrome, Firefox and WebKit browsers. We have thousands of users, each with limits of at least 25 browsers depending on their plan. These are primarily spread between our two main server locations of London and San Francisco.

These browsers are for using with Puppeteer, Playwright or via one of our APIs. That means they start up, running a script for some amount of seconds or minutes, then close. 

At the extreme end, some of our users will open up a hundred browsers, do a large batch of web scraping, then do nothing for a week. Meanwhile, we have users fairly regularly starting browsers 24/7 for tasks such as generating PDF exports of dashboards.

 So, it’s a rapidly fluctuating environment with a range of usage requirements.

## **Plan of Attack**

Breaking down the challenge into a few steps went as follows:

1.  **Identifying the Optimal Location -** Recognizing that all traffic passes through the load balancer, I determined it to be the ideal point for implementing connection limits.
2.  **Limitation of Embedded Nginx Module -** Initial exploration revealed that the embedded [Nginx module for limiting connections](https://nginx.org/en/docs/http/ngx_http_limit_conn_module.html) lacked the capability to adjust limits dynamically, prompting us to explore alternative solutions.
3.  **Leveraging OpenResty and Lua Scripting -** I opted to leverage the versatility of OpenResty and Lua scripting to overcome the limitations of the embedded Nginx module. This decision was driven by the flexibility and extensibility offered by Lua scripting.
4.  **Utilizing `lua-resty-limit-trafic` Library -** I discovered the [lua-resty-limit-trafic](https://github.com/openresty/lua-resty-limit-traffic) library during my research, an official solution for limiting traffic. This library offers dedicated modules for controlling both request frequency and concurrent connections, aligning perfectly with our requirements.

## Implementation

I developed a custom limiter module based on the `lua-resty-limit-trafic` library, comprising three main functions:

1.  **GetAllowedConcurrency -** This function retrieves the concurrency limit for a given customer token. It fetches the limit either from a database or a local cache. In cases where the limit cannot be retrieved, a default limit is applied.
2.  **LimitRequestsByToken -** Utilizing the lua-resty-limit-trafic library, this function increments the number of connections associated with a specific token. It checks if the limit is exceeded and throws an error accordingly. The limit is dynamically set based on the customer's concurrency limit.
3.  ‍**LeavingConnection -** Upon the closure of a connection, this function decreases the number of active connections associated with the token.

The following code snippet showcases the implementation of these functions, demonstrating how the lua-resty-limit-trafic library is utilized to manage connection limits dynamically.

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

To integrate the limiter module into the Nginx configuration, we import it into the default.conf file. The module is invoked within the `access_by_lua_block` and `log_by_lua_block` directives, leveraging the Authorization header as the identifier for each customer token.

It's imperative to create a `lua_shared_dict` to store the number of existing connections per token.

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

Additionally, to avoid `module not found` errors, the `lua_package_path` directive must be configured in the nginx.conf file to specify the location of the limiter module.

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

At this point we are limiting connections as planned. However, prior to implementing this solution, we encountered some challenges that needed to be addressed.

## Issues Faced

When using the limiter, we encountered escalating CPU and memory consumption over time, indicating the presence of a memory leak. This posed a significant challenge as it necessitated periodic service restarts to prevent crashes, prompting an exhaustive investigation to identify and resolve the underlying issue.

I began troubleshooting by disconnecting non-essential components. First, I removed the database connection, but unfortunately, it didn't resolve the issue. Next, I removed the local cache, yet the problem persisted.

As a last resort, I decided to test removing the import `local limits = require "limiter"` and consolidating all code into a single file. Surprisingly, this resolved the issue.

I finally identified the problem. Prior to editing the nginx.conf file, I was using the [package.path](https://www.lua.org/manual/5.1/manual.html#pdf-package.path) within the Lua script to import my custom Lua folder.

```
 package.path = "/usr/local/balancer/lua/?.lua;" .. package.path 

```

An important oversight occurred: each time the script ran, the package.path increased because I was concatenating my folder name to the same value in every request. Consequently, after a few requests, inspecting the value of package.path revealed an accumulation of folder names.

```
 package.path = "/usr/local/balancer/lua/?.lua;/usr/local/balancer/lua/?.lua;/usr/local/balancer/lua/?.lua;..." 

```

Initially, it seemed that concatenating folder names to package.path was the correct approach, as suggested by the Lua official documentation. However, this method caused the mentioned problems in our specific use case. Eventually, I referred to the [lua-nginx-module](https://github.com/openresty/lua-nginx-module#lua_package_path) documentation to find the correct way to import modules, and the problem was resolved by adding the `lua_package_path` directive to the nginx.conf file.

Below is a graphical representation illustrating the consumption of resources before and after implementing the fix.

Additionally, to enhance performance and minimize database queries, we incorporated a local cache with a Time-to-Live (TTL) mechanism to store customer limits. This optimization strategy effectively reduced latency and alleviated the strain on the database.

## **How It Turned Out**

In the end, the new limiter worked great. It now effectively manages concurrent connections per customer to keep the service healthy and stable, while preventing service abuses and maintaining healthy worker processes.

If you would like to see it in action, go ahead and grab a [Browserless trial](https://www.browserless.io/pricing) and have a play with using the concurrent browsers.