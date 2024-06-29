<!--yml

category: 未分类

date: 2024-05-27 12:50:33

-->

# 在Cloudflare Workers中用Python编写 · Cloudflare Workers文档

> 来源：[https://developers.cloudflare.com/workers/languages/python/](https://developers.cloudflare.com/workers/languages/python/)

Cloudflare Workers为Python提供了一流的支持，包括对以下内容的支持：

+   Python的大部分[标准库](/workers/languages/python/stdlib/)

+   所有[绑定](/workers/runtime-apis/bindings/)，包括[Workers AI](/workers-ai/)、[Vectorize](/vectorize)、[R2](/r2)、[KV](/kv)、[Queues](/queues)、[Durable Objects](/durable-objects/)、[服务绑定](/workers/runtime-apis/bindings/service-bindings/) 等。

+   [环境变量](/workers/configuration/environment-variables/)和[密钥](/workers/configuration/secrets/)

+   一个强大的[外部函数接口（FFI）](/workers/languages/python/ffi)，允许您直接从Python中使用JavaScript对象和函数 — 包括所有[运行时API](/workers/runtime-apis/)

+   [内置包](/workers/languages/python/packages)，包括[FastAPI<svg xmlns="http://www.w3.org/2000/svg" fill="none" stroke="currentcolor" stroke-width="1.5" stroke-linecap="round" stroke-linejoin="round" viewBox="0 0 16 16" role="img" aria-labelledby="title-4744738674102027"><title id="title-4744738674102027">外部链接图标</title></svg> 打开外部链接](https://fastapi.tiangolo.com/)，[Langchain<svg xmlns="http://www.w3.org/2000/svg" fill="none" stroke="currentcolor" stroke-width="1.5" stroke-linecap="round" stroke-linejoin="round" viewBox="0 0 16 16" role="img" aria-labelledby="title-4744738674102027"><title id="title-4744738674102027">外部链接图标</title></svg> 打开外部链接](https://pypi.org/project/langchain/)，[httpx<svg xmlns="http://www.w3.org/2000/svg" fill="none" stroke="currentcolor" stroke-width="1.5" stroke-linecap="round" stroke-linejoin="round" viewBox="0 0 16 16" role="img" aria-labelledby="title-4744738674102027"><title id="title-4744738674102027">外部链接图标</title></svg> 打开外部链接](https://www.python-httpx.org/) 等。

## 开始

```
 git clone https://github.com/cloudflare/python-workers-examples  cd python-workers-examples/01-hello  npx wrangler@latest dev 
```

一个Python Worker可以简单到只有三行代码：

```
src/entry.py from js import Response  def  on_fetch(request): return Response.new("Hello World!") 
```

类似于使用[JavaScript](/workers/languages/javascript)、[TypeScript](/workers/languages/typescript)或[Rust](/workers/languages/rust/)编写的Workers，Python worker的主入口点是[`fetch`处理程序](/workers/runtime-apis/handlers/fetch)。在Python Worker中，此处理程序称为`on_fetch`。

要在本地运行Python Worker，您可以使用[Wrangler](/workers/wrangler/)，Cloudflare Workers的CLI工具：

要将Python Worker部署到Cloudflare，请运行[`wrangler deploy`](/workers/wrangler/commands/#deploy)：

```
 npx wrangler@latest deploy 
```

## 模块

Python workers可以跨多个文件分割。让我们创建一个新的Python文件，称为`src/hello.py`：

```
src/hello.py def  hello(name): return  "Hello, "  + name +  "!" 
```

现在，我们可以修改`src/entry.py`来使用新的模块。

```
src/entry.py from hello import hello  from js import Response  def  on_fetch(request): return Response.new(hello("World")) 
```

一旦您编辑了`src/entry.py`，Wrangler将自动检测更改并重新加载您的Worker。

## `Request`接口

传递给您的`fetch`处理程序的`request`参数是一个JavaScript请求对象，通过外部函数接口公开，允许您直接从Python代码中访问它。

让我们尝试编辑Worker以接受POST请求。我们从[Request的文档](/workers/runtime-apis/request)中了解到，在`async`函数中可以调用`await request.json()`来解析请求体作为JSON。在Python Worker中，您可以这样写：

```
src/entry.py from js import Response  from hello import hello  async  def  on_fetch(request): name =  (await request.json()).name return Response.new(hello(name)) 
```

编辑`src/entry.py`后，Wrangler应该会自动重新启动本地开发服务器。现在，如果您发送一个带有适当主体的POST请求，您的Worker应该会回复一个个性化的消息。

```
 $ curl  --header  "Content-Type: application/json"  \ --request POST \ --data  '{"name": "Python"}' http://localhost:8787 
```

## `env`参数

除了`request`参数外，还将`env`参数传递给Python `fetch`处理程序，并可用于访问[环境变量](/workers/configuration/environment-variables/)、[秘密](/workers/configuration/secrets/)和[绑定](/workers/runtime-apis/bindings/)。

例如，让我们尝试在Python Worker中设置和使用环境变量。首先，在您的Worker的`wrangler.toml`中添加环境变量：

```
wrangler.toml name  =  "hello-python-worker"  main  =  "src/entry.py"  compatibility_flags  =  ["python_workers"]  compatibility_date  =  "2024-03-20"  [vars]  API_HOST  =  "example.com" 
```

然后，您可以通过`env`参数访问`API_HOST`环境变量：

```
src/entry.py from js import Response  async  def  on_fetch(request, env): return Response.new(env.API_HOST) 
```

## 进一步阅读
