<!--yml
category: 未分类
date: 2024-05-27 14:24:04
-->

# Microdot: Yet Another Python Web Framework - miguelgrinberg.com

> 来源：[https://blog.miguelgrinberg.com/post/microdot-yet-another-python-web-framework](https://blog.miguelgrinberg.com/post/microdot-yet-another-python-web-framework)

I just realized that I have never written on this blog about [Microdot](https://github.com/miguelgrinberg/microdot), my very own web framework for Python. I have released Microdot 2.0 a few days ago, so I guess this is a good time to make a belated announcement, and tell you why this world needs yet another Python web framework.

But before I tell you about the reasons and the history of Microdot, let me share some of its features:

*   Flask-like syntax, but without the magical/obscure parts (no application/request contexts)
*   Small enough to work with MicroPython, while also being compatible with CPython
*   Fully compatible with asyncio
*   Websocket support
*   Server-Sent Events (SSE) support
*   Templating support with Jinja (CPython) and uTemplate (MicroPython)
*   Cross-Origin Request Sharing (CORS) support
*   User sessions stored on cryptographically signed cookies
*   Uses its own minimal web server on MicroPython, and integrates with any ASGI or WSGI web servers on CPython
*   Included test client to use in unit tests

Interested? Keep reading to learn more about Microdot.

## Why Microdot

Back in 2019, I was working on a hardware project based on the ESP8266 microcontroller. For the software portion of this project I used [MicroPython](https://micropython.org/), an alternative implementation of the Python language that is designed for small devices.

I wanted to host a small web-based interface on the device, and to my surprise I could not find any usable web frameworks. Things such as Flask or Bottle do not work under MicroPython because they are too big for it. The only MicroPython web framework I could find was one called "picoweb", which required an unofficial fork of the MicroPython language, which to me was a deal breaker.

Microdot was born out of needing to have a web framework that was as close as possible to Flask, but designed to run on an official and actively maintained version of MicroPython.

I ended up creating Microdot, and used it to complete my project. I also put the source code on GitHub, since it was obvious to me that there was a hole in the MicroPython ecosystem in terms of web frameworks. While it has received growing attention from hardware inclined developers, it managed to stay below the radar for a lot of people in the Python community for almost 5 years now. Here is a timeline of the major events in Microdot's history:

| Date | Event |
| --- | --- |
| April 2019 | Microdot 0.1, first public release |
| August 2022 | Microdot 1.0, with a synchronous base implementation and an asyncio extension |
| December 2023 | Microdot 2.0, redesigned as 100% asynchronous |

## How Does Microdot Code Look Like?

Microdot is actually very similar to Flask. You write web routes as decorated functions:

```
from microdot import Microdot

app = Microdot()

@app.get('/')
async def index(request):
    return {'hello': 'world'}

app.run(debug=True) 
```

The `@app.get` decorator is used to define GET requests. There are similar decorators for POST, PATCH, PUT and DELETE. Like Flask, Microdot handles OPTIONS and HEAD requests automatically.

Microdot 2 is asynchronous, so it is best to write handler functions as `async def` functions. Under CPython, regular `def` functions execute in a thread executor (like FastAPI), but most hardware devices that run MicroPython lack threading support, so regular functions on that platform just block for as long as they run.

One aspect in Flask that I do not like is the use of application and request contexts, as these add an unnecessary layer of complexity. I did not want to have that, so request handler functions in Microdot just receive a request object as a first positional argument.

As with Flask, the return value from a handler function is the response, and Microdot automatically formats the response as JSON if a dictionary is returned. It also supports second and third returned values for custom status code and headers. Microdot also copies the way streamed responses work in Flask with the use of generators, and also supports asynchronous generators.

I like the functionality offered by Blueprints in Flask, but here once again I feel Flask makes everything too complicated. In Microdot, you can create multiple application instances and use the `mount()` method to combine them:

```
api = Microdot()

@api.get('/users')
async def get_users():
    pass

main_app = Microdot()
main_app.mount(api, url_prefix='/api')  # /api/users route is added to main_app 
```

Microdot has native support for WebSocket and Server-Sent Events. Here are example endpoints that use these features:

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

Hopefully there is nothing mysterious or magical in this code and you can understand it fully just from reading it.

If you want to render HTML templates, Microdot can use Jinja or uTemplate, the latter being the main (only?) templating library available for MicroPython. Templates can be rendered synchronously (blocking the asyncio loop) or asynchronously. For larger templates it is best to use the async interface as that improves concurrency. Templates can also be streamed as regular or asynchronous generators, one more way to improve performance and concurrency.

User sessions are implemented in a way that is also similar to Flask, but I decided to write user sessions as [JWTs](https://en.wikipedia.org/wiki/JSON_Web_Token), which are fairly easy to [debug](https://jwt.io/).

If you want to learn more about Microdot's features, I have written [documentation](https://microdot.readthedocs.io/en/stable/) for it. I also offer a nice collection of [examples](https://github.com/miguelgrinberg/microdot/tree/main/examples) in the GitHub repository.

## How Small is Microdot?

This is not a straightforward question to answer, because unlike Flask, FastAPI and most other frameworks, Microdot has a modular structure, so there are a number of different configurations that can be measured depending on what features are used. When working with CPython this is not very important, but for MicroPython projects running on microcontrollers it is useful to be able to pick and choose exactly what you need and drop the parts that you don't.

The core Microdot framework comes in a single *microdot.py* Python source file. Below I used the [cloc](https://github.com/AlDanial/cloc) utility to count how many lines of code the current version of this file has:

|  | files | blank | comment | code |
| --- | --- | --- | --- | --- |
| Microdot 2.0.1 (minimal) | 1 | 216 | 456 | 728 |

Next I calculated the full Microdot configuration for the same version, including all of its features:

|  | files | blank | comment | code |
| --- | --- | --- | --- | --- |
| Microdot 2.0.1 (full) | 11 | 435 | 759 | 1577 |

Just so that you have an idea of how small this is, let's see how big Flask and FastAPI are. To make this a more consistent comparison, I've included Werkzeug with Flask, and Starlette with FastAPI.

|  | files | blank | comment | code |
| --- | --- | --- | --- | --- |
| Flask 3.0.0 | 24 | 1775 | 3211 | 3854 |
| Werkzeug 3.0.1 | 59 | 4043 | 5634 | 11704 |
| Total | 83 | 5818 | 8845 | 15558 |

|  | files | blank | comment | code |
| --- | --- | --- | --- | --- |
| FastAPI 0.106.0 | 40 | 2052 | 5884 | 9839 |
| Starlette 0.27.0 | 34 | 1008 | 477 | 4969 |
| Total | 74 | 3060 | 6361 | 14808 |

From this you can see that both Flask and FastAPI, when combined with their main supporting dependency have about 15K lines of code each, while Microdot has 1.5K or roughly 10% of the size of its bigger competitors (or 5% if you only need basic web routing features).

Since I'm counting Werkzeug and Starlette as part of Flask and FastAPI respectively, you may be curious about what third-party dependencies Microdot relies on, and if there are any big ones. The single-file minimal version does not require any dependencies at all, and it even includes its own web server. Some of the optional features do require third-party dependencies, and this is comparable to the larger frameworks. Like Flask, you will need to add a templating library (Jinja or uTemplate) to use templates. For user sessions, Microdot relies on PyJWT, while Flask uses its own Itsdangerous package.

## Should I Switch to Microdot?

For a MicroPython project I think Microdot is a great framework to use, even though there are now a couple others that did not exist back in 2019.

If you are using CPython you are certainly welcome to switch if you want to, but my recommendation would be to stay with your current web framework if you are happy with it. A good reason to switch would be if you want to make better use of your server's resources. It's really hard to make predictions, but depending on the project you may be able to fit an extra web server worker or two on the same hardware, just from RAM savings after switching from a larger framework to Microdot. But as I said, I wouldn't go through the pain of a migration unless size is very important to you.

What I would like to ask is that you keep Microdot in mind for when you start a new project. If you end up using it, I would like to know how it works for you, so please reach out!