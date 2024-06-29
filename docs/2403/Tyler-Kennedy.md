<!--yml

category: 未分类

date: 2024-05-27 15:00:28

-->

# Tyler Kennedy

> 来源：[https://tkte.ch/articles/2024/03/15/parsing-urls-in-python.html](https://tkte.ch/articles/2024/03/15/parsing-urls-in-python.html)

简而言之 - 如果你需要在Python中解析URL，请尝试[can_ada](https://github.com/tktech/can_ada)。

## URLs

正确地解析URL*是非常困难的。甚至谁来定义一个“正确”的URL呢？自[1994](https://tools.ietf.org/html/rfc1738)年以来，URL已经发生了巨大变化。[WHATWG](https://whatwg.org/)有一个[URL规范](https://url.spec.whatwg.org/)，它全面并帮助标准化了URL在浏览器中的行为，但这个规范仍然不是普遍适用的，像“允许有多少斜杠”这样的歧义确实会让[你恼火](https://daniel.haxx.se/blog/2017/01/30/one-url-standard-please)：

> 因此，浏览器接受用成千上万个正斜杠而不是两个的URL写法。这并不是规范说URL可能合法包含成千上万个斜杠的充分理由。我完全确信，世界上没有任何关键内容使用这种格式化的URL，如果我们限制数量为个位数，也没有人会因此而伤心。所以我们应该这样做。然后，浏览器应拒绝使用更多的URL。

*然而*，如果你正在创建新的内容，并希望处理URL，WHATWG的URL规范可能是开始的最佳选择。

## URLs in Python

如果你在Python中工作，你可能会从内置的`urllib`模块开始。它已经存在很长时间了，但不幸的是它也不符合*任何*URL规范，包括更早的[rfc3978](https://tools.ietf.org/html/rfc3986)：

> RFC 3986被认为是当前的标准，任何对urlparse模块的未来更改都应符合它。由于现实场景的解析和为了向后兼容性，保留了一些来自旧版RFC的解析怪癖。test_urlparse.py中的测试用例提供了解析行为的良好指标。

... 或者WHATWG URL解析器规范：

> WHATWG URL解析器规范也应予以考虑。由于现有用户代码API行为期望（Hyrum's Law），我们也不完全符合它。在进行更改时，它作为一个有用的指南。

由于存在超过16年，许多项目依赖于urllib模块以*完全相同*的方式解析URL，这使得它不太可能在任何重大方式上发生变化。

## Ada

[Ada](https://github.com/ada-url/ada)项目是一个新的（2024年）尝试，旨在创建一个符合WHATWG URL规范且非常[快速](https://github.com/ada-url/ada?tab=readme-ov-file#ada-is-fast)的URL解析库，每1个由cURL解析的URL可以解析7个。它用C++编写，现在已经有多种其他语言的绑定，包括[Python](https://github.com/ada-url/ada-python)，并且作为Node.js 18版本的URL解析库已经取得了巨大的成功：

> 自从Node.js 18版本以来，新增了一个名为Ada的URL解析器依赖项到Node.js中 —— 这一增加将Node.js解析URL的性能提升到一个新的水平。一些结果可以达到提高400%。

ada-python绑定功能完善，也是该项目的官方绑定。然而，ada-python绑定是建立在[CFFI](https://cffi.readthedocs.io/en/latest/)之上的，这种方法使得在Python中编写C和Python之间的绑定，失去了使用Ada时获得的性能优势，因为大部分时间都花在了函数调用上。

## can_ada

[Daniel Lemire](https://lemire.me)是Ada项目背后的开发者之一，他要求我 [看一看](https://github.com/ada-url/ada-python/pull/1#issuecomment-1550405501) ada-python绑定，从而诞生了[can_ada](https://github.com/tktech/can_ada)，一个新的Python绑定，使用[pybind11](https://pybind11.readthedocs.io/en/stable/)和模板魔法生成绑定代码，然后编译为Python扩展模块。这种方法潜力巨大，比ada-python绑定快得多，实际上，比较这两种绑定时，新的can_ada绑定速度大约是ada-python绑定的两倍，而ada-python绑定又比`urllib.parse`快两倍！

```
`---------------------------------------------------------------------------------
Name (time in ms)              Min                 Max                Mean 
---------------------------------------------------------------------------------
test_can_ada_parse         54.1304 (1.0)       54.6734 (1.0)       54.3699 (1.0) 
test_ada_python_parse     107.5653 (1.99)     108.1666 (1.98)     107.7817 (1.98)
test_urllib_parse         251.5167 (4.65)     255.1327 (4.67)     253.2407 (4.66)
---------------------------------------------------------------------------------` 
```

同时，采用pybind11方法可以非常简洁和[可读](https://github.com/TkTech/can_ada/blob/main/src/binding.cpp)的绑定定义，仅仅60行代码，几乎与底层C++ API一一对应。

CPython 3.7到3.12和PyPy 3.7到3.9的二进制构建现已提供在许多操作系统和架构上。通过pip安装它或者[获取源代码](https://github.com/tktech/can_ada)：

## 示例

```
`import can_ada
urlstring = "https://www.GOoglé.com/./path/../path2/"
url = can_ada.parse(urlstring)
# prints www.xn--googl-fsa.com, the correctly parsed domain name according
# to WHATWG
print(url.hostname)
# prints /path2/, which is the correctly parsed pathname according to WHATWG
print(url.pathname)` 
```

将其与不符合WHATWG规范的urllib版本进行比较：

```
`import urllib.parse
urlstring = "https://www.GOoglé.com/./path/../path2/"
url = urllib.parse.urlparse(urlstring)
# prints www.googlé.com
print(url.hostname)
# prints /./path/../path2/
print(url.path)` 
```
