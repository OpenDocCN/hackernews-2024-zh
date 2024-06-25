<!--yml

分类：未分类

日期：2024 年 5 月 27 日 14:24:04

-->

# Microdot：另一个 Python Web 框架 - miguelgrinberg.com

> 来源：[`blog.miguelgrinberg.com/post/microdot-yet-another-python-web-framework`](https://blog.miguelgrinberg.com/post/microdot-yet-another-python-web-framework)

我刚意识到我从未在这个博客上写过关于[Microdot](https://github.com/miguelgrinberg/microdot)，我的自己的 Python Web 框架。我几天前发布了 Microdot 2.0，所以我想现在是一个很好的时机来做一个迟到的公告，并告诉你为什么这个世界需要另一个 Python Web 框架。

但在我告诉你关于 Microdot 的原因和历史之前，让我分享一些它的特点：

+   类似于 Flask 的语法，但没有神奇/晦涩的部分（没有应用程序/请求上下文）

+   大到足以与 MicroPython 一起工作，同时也与 CPython 兼容

+   与 asyncio 完全兼容

+   支持 Websocket

+   支持服务器发送事件（SSE）

+   使用 Jinja（CPython）和 uTemplate（MicroPython）进行模板支持

+   支持跨域请求共享（CORS）

+   用户会话存储在加密签名的 Cookie 中

+   在 MicroPython 上使用自己的最小 Web 服务器，并与 CPython 上的任何 ASGI 或 WSGI Web 服务器集成

+   包含测试客户端用于单元测试

感兴趣吗？继续阅读了解更多关于 Microdot 的信息。

## 为什么 Microdot

回到 2019 年，我正在基于 ESP8266 微控制器的硬件项目上工作。对于这个项目的软件部分，我使用了[MicroPython](https://micropython.org/)，这是 Python 语言的一种替代实现，专为小型设备设计。

我想在设备上托管一个小型基于 Web 的界面，但令我惊讶的是，我找不到任何可用的 Web 框架。诸如 Flask 或 Bottle 之类的东西在 MicroPython 下不起作用，因为它们对它来说太大了。我能找到的唯一一个 MicroPython Web 框架叫做“picoweb”，它需要一个非官方的 MicroPython 语言分支，对我来说是个硬伤。

Microdot 的诞生源于需要一个尽可能接近 Flask 的 Web 框架，但设计用于官方和积极维护的 MicroPython 版本。

我最终创建了 Microdot，并用它完成了我的项目。我还将源代码放在 GitHub 上，因为对我来说很明显 MicroPython 生态系统在 Web 框架方面存在空白。虽然它受到了硬件倾向开发者越来越多的关注，但在 Python 社区的很多人眼中，它几乎在过去的 5 年里一直处于低调状态。以下是 Microdot 历史上的主要事件时间表：

| 日期 | 事件 |
| --- | --- |
| 2019 年 4 月 | Microdot 0.1，首次公开发布 |
| 2022 年 8 月 | Microdot 1.0，具有同步基础实现和异步扩展 |
| 2023 年 12 月 | Microdot 2.0，重设计为 100％异步 |

## Microdot 代码是什么样子？

Microdot 实际上与 Flask 非常相似。你可以将 Web 路由写成装饰函数：

```
from microdot import Microdot

app = Microdot()

@app.get('/')
async def index(request):
    return {'hello': 'world'}

app.run(debug=True) 
```

`@app.get` 装饰器用于定义 GET 请求。类似的装饰器也适用于 POST、PATCH、PUT 和 DELETE。像 Flask 一样，Microdot 自动处理 OPTIONS 和 HEAD 请求。

Microdot 2 是异步的，所以最好将处理程序函数编写为 `async def` 函数。在 CPython 中，常规的 `def` 函数在线程执行器中执行（就像 FastAPI 一样），但是大多数运行 MicroPython 的硬件设备缺乏线程支持，因此在该平台上的常规函数只会阻塞它们运行的时间。

Flask 中有一个我不喜欢的方面是应用程序和请求上下文的使用，因为这些增加了不必要的复杂性。我不想要那个，所以 Microdot 中的请求处理程序函数只是接收一个请求对象作为第一个位置参数。

与 Flask 一样，处理程序函数的返回值是响应，如果返回的是字典，Microdot 会自动将响应格式化为 JSON。它还支持自定义状态码和标头的第二和第三个返回值。Microdot 还通过使用生成器复制了 Flask 中流式响应的工作方式，并且还支持异步生成器。

我喜欢 Flask 中蓝图提供的功能，但在这里我再次感到 Flask 使一切都变得太复杂了。在 Microdot 中，你可以创建多个应用程序实例并使用 `mount()` 方法将它们组合起来：

```
api = Microdot()

@api.get('/users')
async def get_users():
    pass

main_app = Microdot()
main_app.mount(api, url_prefix='/api')  # /api/users route is added to main_app 
```

Microdot 原生支持 WebSocket 和服务器发送事件。以下是使用这些功能的示例端点：

```
from microdot.websocket import with_websocket
from microdot.sse import with_sse

@app.route('/echo')
@with_websocket
async def echo(request, ws):
    while True:
        data = await ws.receive()
        await ws.send(data)

@app.route('/events')
@with_sse
async def events(request, sse):
    for i in range(10):
        await asyncio.sleep(1)
        await sse.send({'counter': i}) 
```

希望这段代码中没有什么神秘或奇幻的东西，你可以完全理解它只是阅读它。

如果你想要渲染 HTML 模板，Microdot 可以使用 Jinja 或 uTemplate，后者是 MicroPython 中主要（唯一？）可用的模板库。模板可以同步（阻塞 asyncio 循环）或异步地渲染。对于较大的模板，最好使用异步接口，因为这样可以提高并发性。模板还可以作为常规或异步生成器进行流式传输，这是提高性能和并发性的另一种方式。

用户会话的实现方式与 Flask 类似，但我决定将用户会话编写为 [JWTs](https://en.wikipedia.org/wiki/JSON_Web_Token)，这在 [debug](https://jwt.io/) 方面相当容易。

如果你想了解更多关于 Microdot 的功能，我为它编写了 [文档](https://microdot.readthedocs.io/en/stable/)。我还在 GitHub 仓库中提供了一个不错的 [示例集合](https://github.com/miguelgrinberg/microdot/tree/main/examples)。

## Microdot 有多小？

这不是一个直接的问题，因为与 Flask、FastAPI 和大多数其他框架不同，Microdot 采用模块化结构，因此可以根据使用的功能来测量许多不同的配置。在使用 CPython 时，这并不重要，但对于在微控制器上运行的 MicroPython 项目，能够精确选择所需的内容并丢弃不需要的部分是有用的。

核心 Microdot 框架以一个名为 *microdot.py* 的 Python 源文件形式呈现。下面我使用 [cloc](https://github.com/AlDanial/cloc) 实用程序来计算当前版本的此文件有多少行代码：

|  | 文件 | 空白 | 注释 | 代码 |
| --- | --- | --- | --- | --- |
| Microdot 2.0.1（最小版） | 1 | 216 | 456 | 728 |

接下来，我计算了相同版本的完整 Microdot 配置，包括其所有功能：

|  | 文件 | 空白 | 注释 | 代码 |
| --- | --- | --- | --- | --- |
| Microdot 2.0.1（完整版） | 11 | 435 | 759 | 1577 |

只是为了让你了解这有多小，我们来看看 Flask 和 FastAPI 有多大。为了使比较更一致，我将 Werkzeug 与 Flask 一起使用，并将 Starlette 与 FastAPI 一起使用。

|  | 文件 | 空白 | 注释 | 代码 |
| --- | --- | --- | --- | --- |
| Flask 3.0.0 | 24 | 1775 | 3211 | 3854 |
| Werkzeug 3.0.1 | 59 | 4043 | 5634 | 11704 |
| 总计 | 83 | 5818 | 8845 | 15558 |
|  | 文件 | 空白 | 注释 | 代码 |
| --- | --- | --- | --- | --- |
| FastAPI 0.106.0 | 40 | 2052 | 5884 | 9839 |
| Starlette 0.27.0 | 34 | 1008 | 477 | 4969 |
| 总计 | 74 | 3060 | 6361 | 14808 |

从这里你可以看到，Flask 和 FastAPI 都与它们的主要支持依赖项结合使用，每个约有 15K 行代码，而 Microdot 有 1.5K 行，大约是其较大竞争对手的 10%（如果你只需要基本的 Web 路由功能，则为 5%）。

由于我将 Werkzeug 和 Starlette 视为 Flask 和 FastAPI 的一部分，你可能会对 Microdot 依赖的第三方依赖关系以及是否有大型依赖关系感到好奇。单文件最小版本根本不需要任何依赖项，甚至包含自己的 Web 服务器。一些可选功能确实需要第三方依赖项，这与较大的框架相比较。与 Flask 一样，你需要添加一个模板库（Jinja 或 uTemplate）来使用模板。对于用户会话，Microdot 依赖于 PyJWT，而 Flask 则使用其自己的 Itsdangerous 包。

## 我应该切换到 Microdot 吗？

对于一个 MicroPython 项目，我认为 Microdot 是一个很好的框架选择，尽管现在有一些在 2019 年并不存在的其他框架。

如果你正在使用 CPython，你当然可以随意转换，但我建议如果你对当前的 Web 框架感到满意，最好还是保持不变。如果你想更好地利用服务器资源，换一个好的理由是很难做出预测，但根据项目的不同，你可能会在相同的硬件上装载一个或两个额外的 Web 服务器工作进程，只需从一个更大的框架切换到 Microdot 后节省 RAM。但正如我所说的，除非对你来说大小非常重要，否则我不会去经历迁移的痛苦。

我想要问的是，在你开始一个新项目时，请记住 Microdot。如果你最终使用了它，我想知道它对你有什么作用，所以请联系我！
