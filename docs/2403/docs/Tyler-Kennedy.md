<!--yml
category: 未分类
date: 2024-05-27 15:00:28
-->

# Tyler Kennedy

> 来源：[https://tkte.ch/articles/2024/03/15/parsing-urls-in-python.html](https://tkte.ch/articles/2024/03/15/parsing-urls-in-python.html)

tl;dr - Try [can_ada](https://github.com/tktech/can_ada) if you need to parse URLs in Python.

## URLs

Parsing URLs *correctly* is surprisingly hard. Who even defines what a "correct" URL is? URLs have evolved drastically since they were originally defined in [1994](https://tools.ietf.org/html/rfc1738). The [WHATWG](https://whatwg.org/) has a [URL specification](https://url.spec.whatwg.org/) that is comprehensive and has helped standardize the behavior of URLs across browsers, but this specification still isn't universal and ambiguities like "how many slashes are you allowed" can definitely get on [your nerves](https://daniel.haxx.se/blog/2017/01/30/one-url-standard-please):

> So browsers accept URLs written with thousands of forward slashes instead of two. That is not a good reason for the spec to say that a URL may legitimately contain a thousand slashes. I’m totally convinced there’s no critical content anywhere using such formatted URLs and no soul will be sad if we’d restricted the number to a single-digit. So we should. And yeah, then browsers should reject URLs using more.

*However*, if you're creating something new and want to handle URLs, the WHATWG's URL specification probably is the best place to start.

## URLs in Python

If you're working in Python, you'd probably start with the built-in `urllib` module. It's been around forever, but unfortunately it's not compliant with *any* URL specification, either the much older [rfc3978](https://tools.ietf.org/html/rfc3986):

> RFC 3986 is considered the current standard and any future changes to urlparse module should conform with it. The urlparse module is currently not entirely compliant with this RFC due to defacto scenarios for parsing, and for backward compatibility purposes, some parsing quirks from older RFCs are retained. The testcases in test_urlparse.py provides a good indicator of parsing behavior.

... or the WHATWG URL Parser spec:

> The WHATWG URL Parser spec should also be considered. We are not compliant with it either due to existing user code API behavior expectations (Hyrum's Law). It serves as a useful guide when making changes.

Having existed for over 16 years, so many projects depend on the urllib module parsing URLs in *exactly* the way it does that it's unlikely to ever change in any significant fashion.

## Ada

The [Ada](https://github.com/ada-url/ada) project is a new (2024) attempt to create a URL parsing library that adheres to the WHATWG URL specification and works [really, really fast](https://github.com/ada-url/ada?tab=readme-ov-file#ada-is-fast), parsing 7 URLs for every 1 parsed by cURL. Written in C++, it now has bindings to several other languages, including [Python](https://github.com/ada-url/ada-python), and has become the URL parsing library used by Node.js as of version 18 to great success:

> Since Node.js 18, a new URL parser dependency was added to Node.js — Ada. This addition bumped the Node.js performance when parsing URLs to a new level. Some results could reach up to an improvement of 400%.

The ada-python binding is perfectly functional, and the official binding for the project. However, the ada-python bindings are built on [CFFI](https://cffi.readthedocs.io/en/latest/), an approach that has the binding between C and Python written in Python itself, which loses some of the performance benefits of using Ada in the first place when most of the time is spent just making the function call.

## can_ada

[Daniel Lemire](https://lemire.me), one of the developers behind the Ada project asked me to [take a look](https://github.com/ada-url/ada-python/pull/1#issuecomment-1550405501) at the ada-python bindings and out of that was born [can_ada](https://github.com/tktech/can_ada), a new Python binding that uses [pybind11](https://pybind11.readthedocs.io/en/stable/) and template magic to generate the binding code, which is then compiled into a Python extension module. This approach has the potential to be much faster than the ada-python bindings, and indeed when comparing the two bindings, the new can_ada binding is about 2x faster than the ada-python bindings which in turn is about 2x faster than `urllib.parse`!

```
`---------------------------------------------------------------------------------
Name (time in ms)              Min                 Max                Mean 
---------------------------------------------------------------------------------
test_can_ada_parse         54.1304 (1.0)       54.6734 (1.0)       54.3699 (1.0) 
test_ada_python_parse     107.5653 (1.99)     108.1666 (1.98)     107.7817 (1.98)
test_urllib_parse         251.5167 (4.65)     255.1327 (4.67)     253.2407 (4.66)
---------------------------------------------------------------------------------` 
```

At the same time, using the pybind11 approach allows for a very succinct and [readable](https://github.com/TkTech/can_ada/blob/main/src/binding.cpp) binding definition, coming in at just **60** lines of code and almost a 1:1 with the underlying C++ API.

Binary builds are available now for CPython 3.7 to 3.12, and PyPy 3.7 to 3.9 on many OS's and architectures. Install it with pip or [get the source](https://github.com/tktech/can_ada):

## Example

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

Compare this to the urllib version, which is not WHATWG compliant:

```
`import urllib.parse
urlstring = "https://www.GOoglé.com/./path/../path2/"
url = urllib.parse.urlparse(urlstring)
# prints www.googlé.com
print(url.hostname)
# prints /./path/../path2/
print(url.path)` 
```