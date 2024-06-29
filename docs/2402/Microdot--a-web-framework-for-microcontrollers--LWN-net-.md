<!--yml

category: 未分类

date: 2024-05-27 14:33:27

-->

# Microdot：微控制器的Web框架 [LWN.net]

> 来源：[https://lwn.net//Articles/959067/](https://lwn.net//Articles/959067/)

| **请考虑订阅LWN**订阅是LWN.net的生命线。如果您喜欢这些内容并希望看到更多类似内容，请订阅支持，这将有助于LWN继续繁荣发展。请访问[此页面](/subscribe/)注册并保持LWN在网上的存在。 |
| --- |

由**Jake Edge**撰写

2024年1月23日

有许多不同的Python [Web框架](https://wiki.python.org/moin/WebFrameworks)，从微型框架到全栈框架。最近引起我注意的是[Microdot](https://github.com/miguelgrinberg/microdot)，这是一个“极小型Python和MicroPython的Web框架”；因为它针对[MicroPython](https://micropython.org/)，所以可以运行“物联网”（IoT）设备的用户界面，例如。此外，它受到[Flask](https://flask.palletsprojects.com/en/3.0.x/)的启发，这使它对许多潜在的Web开发者来说应该是相当熟悉的。

Microdot由Miguel Grinberg创建，他还创建了著名的[Flask Mega-Tutorial](https://blog.miguelgrinberg.com/post/the-flask-mega-tutorial-part-i-hello-world)，这对许多人来说是介绍Flask Web框架的入门。尽管Flask被认为是一个微框架，但它仍然需要完整的CPython环境来运行；另一个Python微框架在2019年的LWN文章中与Flask并列，名为[Bottle](https://bottlepy.org/docs/dev/)，有类似的需求。Microdot诞生是因为Grinberg在2019年希望找到一个适用于MicroPython的框架，但没有找到合适的东西。

MicroPython是Python的精简版，适用于微控制器；LWN [在2023年5月看了版本 1.20](/Articles/931051/)。MicroPython与CPython之间存在[许多差异](https://docs.micropython.org/en/latest/genrst/index.html)，但它们有足够的重叠，使得Python和Web程序员在使用Microdot时感到宾至如归。因为Microdot也可以在CPython下运行，所以代码可以来回移植，尽管MicroPython当然也可以在常规CPU上运行。

正如Grinberg在他的宣布博客文章中所示，“Hello World” Web应用看起来非常简单：

```
    from microdot import Microdot

    app = Microdot()

    @app.get('/')
    async def index(request):
        return {'hello': 'world'}

    app.run(debug=True)

```

`@app.get()`装饰器描述了匹配的URL，`index()`函数返回作为响应使用的数据。在这种情况下，返回字典会导致Microdot将响应格式化为JSON；状态码或自定义标头可以作为多值返回的第二和第三个元素。

在示例中突出显示的一件事是`index()`的定义中的"`async def`"。虽然Microdot 1.0在2022年8月发布时具有同步基础和[`asyncio`](https://docs.python.org/3/library/asyncio.html)扩展，但在2022年12月发布的Microdot 2.0则设计为完全异步。在Microdot 2.0中仍然可以使用普通的`def`，但在通常不支持线程的微控制器上，Web服务器的响应性能会受到影响。因此，在使用`async def`定义的处理程序内部，需要使用`await`来处理需要I/O的处理，而不是同步等待操作返回控制。Grinberg在PyCon 2017上做了一场关于[异步Python的讲座](/Articles/726600/)，对不熟悉这个主题的读者可能会感兴趣。

虽然Microdot受到Flask的启发，但并不受其束缚。特别是，Grinberg表示他喜欢Flask的[蓝图](https://flask.palletsprojects.com/en/3.0.x/blueprints/)用于模块化Web应用程序，但认为Flask的机制过于复杂。因此，Microdot引入了在URL层次结构中的不同点上["挂载"子应用程序](https://microdot.readthedocs.io/en/stable/intro.html#mounting-a-sub-application)的概念。正如文档所示，可以在两个单独的文件中创建`customer_app`和`orders_app`，然后将它们导入到主应用程序源文件中，以便通过类似方式创建完整的应用程序：

```
    def create_app():
        app = Microdot()
        app.mount(customers_app, url_prefix='/customers')
        app.mount(orders_app, url_prefix='/orders')
        return app

```

这允许顾客和订单的功能被写成它们在层次结构的顶层，然后它们可以被安装到它们实际应出现的位置；任何重新组织都不需要更改子应用程序，只需使用子应用程序代码内部的相对URL进行不同的挂载点。

对于其页面的渲染，Microdot [支持](https://microdot.readthedocs.io/en/stable/extensions.html#rendering-templates) 熟悉的[Jinja模板引擎](https://jinja.palletsprojects.com/en/3.1.x/)，但仅适用于CPython。对于MicroPython，可以使用[`utemplate`](https://github.com/pfalcon/utemplate)，它也可以在CPython上工作；它实现了Jinja语法的简化版本。其他[扩展](https://microdot.readthedocs.io/en/stable/extensions.html) 提供了WebSocket、安全用户会话、服务器发送事件（SSE）和其他功能。

在其最简配置中，Microdot 的代码量略多于 700 行（加上注释和空行）在一个单文件中；完整配置稍多于 11 个文件。格林伯格将其与 Flask 和 [FastAPI](https://fastapi.tiangolo.com/) 进行了对比，后者在包括其主要依赖项（[Werkzeug](https://werkzeug.palletsprojects.com/en/3.0.x/) 和 [Starlette](https://www.starlette.io/)）时代码量约为其十倍。同时，Microdot 的第三方依赖性很少，正如人们可能猜测的那样：

> 这个单文件的极简版本完全不需要任何依赖，并且甚至包含自己的 Web 服务器。一些可选功能确实需要第三方依赖，这与更大的框架类似。像 Flask 一样，你需要添加一个模板库（Jinja 或 uTemplate）来使用模板。对于用户会话，Microdot 依赖于 PyJWT，而 Flask 则使用其自己的 Itsdangerous 包。

[部署 Web 应用程序](https://microdot.readthedocs.io/en/stable/extensions.html#deploying-on-a-production-web-server) 在 MicroPython 上使用内置的 Web 服务器，但 CPython 可以使用 [Asynchronous Server Gateway Interface](https://asgi.readthedocs.io/en/latest/) (ASGI) 服务器或支持其类似前身的 [Web 服务器网关接口](https://wsgi.readthedocs.io/en/latest/what.html) (WSGI)。TLS 也 [受支持](https://microdot.readthedocs.io/en/stable/intro.html#web-server-configuration)，因此可以通过 HTTPS 进行服务。文档中链接了每种服务器类型和两种模板引擎的几个示例程序。虽然 WSGI 是同步协议，但 Microdot 本身在 `asyncio` 事件循环中运行，因此应仍然使用 `async def` 和 `await` [进行编写](https://microdot.readthedocs.io/en/stable/intro.html#concurrency)。

总体来看，Microdot 看起来是微控制器新项目的一个不错选择。格林伯格特别不建议人们放弃他们当前的 CPython 框架来支持 Microdot，除非他们对当前框架不满意或确实需要 Microdot 可提供的内存节省。现在有多个 MicroPython Web 框架和工具列在 [Awesome MicroPython 网站](https://awesome-micropython.com/#web)上，所以比格林伯格五年前发现的选项要多得多。一个简单的 Web 应用程序几乎是与无头、网络连接设备的理想接口，因此寻找使其开发更加轻松的方法是受欢迎的。从这个角度来看，Microdot 值得一试。

* * *

(

[登录](https://lwn.net/Login/?target=/Articles/959067/)

发表评论)
