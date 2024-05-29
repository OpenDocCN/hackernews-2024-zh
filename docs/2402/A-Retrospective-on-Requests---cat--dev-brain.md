<!--yml
category: 未分类
date: 2024-05-29 13:25:52
-->

# A Retrospective on Requests - cat /dev/brain

> 来源：[https://blog.ian.stapletoncordas.co/2024/02/a-retrospective-on-requests](https://blog.ian.stapletoncordas.co/2024/02/a-retrospective-on-requests)

After writing my [thoughts on github3.py](/2024/01/a-retrospective-on-github3py) down here, someone was asking about Python libraries that people recommend new Python developers read for learning how to structure code and use the packaging systems. My initial reaction was "Don't listen to folks that tell you to read `python-requests` as a good example." When asked for explanation, I gave it but realized that most people only know that Requests does the right things for them for TLS and some other HTTP behaviour, but I doubt anyone understands just how bad the project is internally.

## API Design

One thing many people can likely agree on is the design of the interface for `python-requests` is (mostly) intuitive and lends itself to rapid development, prototyping, and is very nice when working in a REPL. If you're unfamiliar, here are some examples:

```
import requests

# Basic HTTP GET
url: str = "https://requests.readthedocs.io/en/latest/"
r = requests.get(url)
print(r.text)

# Does the right thing with TLS
url: str = "https://expired.badssl.com/"
try:
   r = requests.get(url)
except requests.exceptions.SSLError:
   print("Unable to verify expired certificate")

# POSTing JSON data easily without having to encode it yourself or specify
# headers
url: str = "https://httpbin.org/post"
r = requests.post(url, json={"some": "data"}) 
```

The one problem here is how often people *stop* at the functions defined at the root of the module. The way these functions work is by each instantiating a new `requests.Session`. The `Session` object is so often forgotten and ignored but can dramatically improve one's experience with `python-requests`. It is what provides connection pooling (so if you're making repeated calls to the same domain, it will attempt to keep connections alive for you, pool them, and reuse them so there's less time spent setting up a connection) it provides a way to configure many of the other aspects of the request that you otherwise need to pass as function parameters, and many other ergonomic benefits.

### What's Wrong with the API

Well for one thing, what you see here is far from all of the API surface area. There are *many* more parameters to the function, some of which cooperate with each other and some of which don't. Above we already had the example of sending JSON data; but before that many people have relied on other data serialization formats. The next simplest looking one is `application/x-www-form-urlencoded`. An example of how to do that with `python-requests` is:

```
import requests

s: requests.Session = requests.Session()
url: str = "https://httpbin.org/post"
r = s.post(url, data={"urlencode", "me"}) 
```

After that, because `python-requests` heavily depends on `urllib3`, we can easily support `multipart/form-data`. That can be done in a few ways:

```
import requests

s: requests.Session = requests.Session()
url: str = "https://httpbin.org/post"
# Let's just send a file
r = s.post(url, files={"fileA": open(filename, "rb")})
# NOTE: Do not use files this way, this is purely for simplicity of example
# code

# Let's send a file and some other form elements
r = s.post(
   url,
   data={"wait": "is this urlencoded too?"},
   files={"fileA": open(filename, "rb")},
)

# Let's send two form fields, a basic file, and then a file with a custom
# name, custom part content-type, and additional custom headers for the
# part
r = s.post(
   url,
   data={"wait": "is this urlencoded too?", "form-encoded": "no"},
   files={
      "fileA": open(filename, "rb"),
      "fileB": (
         "custom-filename.xml",
         open(other_filename, "rb"),
         "application/xml",
         {"X-Custom-Part-Header": "value"},
   },
) 
```

If you're unavailable, these will look like:

```
Content-Length: ...
Content-Type: multipart/form-data; boundary=---------------------------9051914041544843365972754266

-----------------------------9051914041544843365972754266
Content-Disposition: form-data; name="wait"

is this urlencoded too?
-----------------------------9051914041544843365972754266
Content-Disposition: form-data; name="form-encoded"

no
-----------------------------9051914041544843365972754266
Content-Disposition: form-data; name="fileA"; filename="a.txt"
Content-Type: text/plain

Content of a.txt.

-----------------------------9051914041544843365972754266
Content-Disposition: form-data; name="fileB"; filename="custom-filename.xml"
Content-Type: application/xml
X-Custom-Part-Header: value

<root><elems><elem>Item</elem></elems></root>

-----------------------------9051914041544843365972754266-- 
```

This is primarily the last one but it has all the items because each request builds upon the last.

Using `data` here, makes things confusing for a lot of people. Some expect that value to be URL form encoded but it doesn't. Sometimes they expect that a file in `files` be URL form encoded. And yes, these use-cases are documented but if people don't know that they want to use `multipart/form-data` encoding, they might not look at the section that describes that interaction.

Furthermore, what happens if someone does:

```
import requests

s: requests.Session = requests.Session()
url: str = "https://httpbin.org/post"
# This is broken, if you see this searching for how to handle things, don't
# copy this but read below
r = s.post(url, json={"a": "b"}, files={"fileA": open(filename, "rb")}) 
```

I expect no one could guess what the interaction **unless** they've already encountered this. The actual behaviour here is that the `json` argument is completely ignored (well [not completely](https://github.com/psf/requests/blob/96b22fa18c00831656ee4b286bf1c9062459b00a/src/requests/models.py#L494-L570), we serialize the data and then throw it away).

This leads to one other way in which users can easily frustrate themselves:

```
import requests

s: requests.Session = requests.Session()
url: str = "https://httpbin.org/post"
# This is broken, if you see this searching for how to handle things, don't
# copy this but read below
r = s.post(url, json={"a": "b"}, headers={"Content-Type": "application/xml"})
r = s.post(url, files={"a": open(filename, "rb")}, headers={"Content-Type": "multipart/form-data"})
r = s.post(url, data={"form": "encoded"}, headers={"Content-Type": "multipart/form-data"})
r = s.post(url, data={"form": "encoded"}, headers={"Content-Length": "1000000"})
# This may not do what some people expect. Many people tend to expect this
# to  **replace** c=d to the query string, but instead it appends c=e so
# the query string becomes a=b&c=d&c=e. In reality, there's no intuitive
# behaviour here for everyone
r = s.post("https://httpbin.org/get?a=b&c=d", params={"c": "e"}) 
```

If you specify your own header above but are relying on Requests to serialize native Python objects for you, you risk using the wrong Content-Type which will cause at best `400 Bad Request` responses and at worst, *very* wrong behaviour in a server. If you override the `Content-Length` header, that can cause many other issues including intermediaries either terminating your request before it reaches the destination, or them rewriting the header (depending on the type of intermediary). Either way, it's not the behaviour *indicated* by the code.

### How Could It Be Better?

This is a great API for a proof of concept and it makes things simple. That said, it hides a lot of what it does and in some ways that causes issues for users.

There are many ways this could be improved for users. There's no one right answer here. Here are some ideas though that can help with some of the problems above:

1.  Keep the semi-functional API but change how the parameters work:

    1.  Get rid of `json` and `files`, consolidate everything into `data` **but** provide classes like `JSONData` or `MultipartFormData` or `UrlEncodedFormData` which can take the basic Python structures. These classes would all implement a [protocol](https://peps.python.org/pep-0544/) (or interface/[abstract base class](https://docs.python.org/3/library/abc.html)) allowing users that care about customizing aspects of this to do so. It's them very explicit and obvious what the behaviour is.
    1.  Make the class above responsible for validating that conflicting headers have not been specified and raising an exception otherwise. (Ideally this logic is baked into the class users would inherit from so they need specify an attribute with the headers they care about (with reasonable defaults) and the base class would do the rest.)
    1.  Provide something a bit nicer than the class names suggested above that is still clear enough

1.  Choose something a bit more familiar to other languages:

    1.  Many other languages have a Request object that can be built up and operated on. Many have excellent APIs around the way headers can be manipulated.
    1.  Provide a builder object for the Request object to easily chain methods together (this looks like the builders in [pyca/cryptography](https://cryptography.io/en/latest/) or more generically [the builder pattern](https://en.wikipedia.org/wiki/Builder_pattern))

I think the first would be the least disruptive and most likely to be accepted by users, but to make it work well, I'd be pretty firm about `data` not accepting anything that doesn't implement the protocol.

The latter feels better long term though because I don't know many developers that only work in Python and never in another language such that they haven't seen the benefits of that pattern.

## TLS Just Works™

The feature that I think attracted **most** people to `python-requests` initially was how easy it was to get verified TLS by default. This was not easy with `httplib`/`http.client` or `urllib`, `urllib2`/`urllib.request` at the time. `httplib2` and `urllib3` (both third-party HTTP clients) made it easy to get verified TLS *if* you knew where your TLS trust bundle was. But many users didn't. So the additional thing `python-requests` did was create `certifi` to package the [Mozilla Trust Bundle](https://wiki.mozilla.org/index.php?title=CA/IncludedCertificates&redirect=no) and rely on that as well to always provide a reliable source for a CA root trust bundle.

### ... Until It Doesn't

Mozilla licenses most things under [MPL 2.0](https://www.mozilla.org/en-US/MPL/2.0/) which is often argued about whether or not it's permissible within various companies under policies (if they have any) around open source libraries.

Further, many people create things that live in zip files as executables with `python-requests` and Python's handling of files that are not python containing data that are packaged appropriately and need to be accessed from that zip file from code inside the zip file is not the greatest.

Additionally, people build those things but don't consider that potentially someone might need to provide alternative settings for TLS:

*   They might be talking to an internal hosted instance with a private CA and private chain of trust. `certifi` will not be helpful there.
*   They may need to provide a client certificate and key to perform mutual TLS authentication (a.k.a., mTLS).

To remedy this, `python-requests` added support for the environment variables `CURL_CA_BUNDLE` and `REQUESTS_CA_BUNDLE` to solve the first point. Great! Fantastic! Except that up until recently, if you did `export REQUESTS_CA_BUNDLE=` you effectively disabled all TLS verification for anything using `python-requests` that sees that exported environment variable.

### ... And It's a Bit Too Limited

When `python-requests` was created there was no `ssl.SSLContext` object in Python, and only after Python 2.7's lifespan was increased did it get backported to 2.7. Even still, it's continued to become a better and better interface to configuring TLS for Python. We've never had a good way to integrate that into `python-requests` because of how that could interact with existing settings and configurations. To make it easy to use, we'd likely have to release a new major version and create significant breaking changes. As it stands, the current maintainers do not have enough time to:

*   Implement those changes
*   Coach and review contributions for those changes
*   Handle the influx of people complaining about the changes or needing assistance upgrading

## Cookies

Python comes with `http.cookies` and `http.cookiejar` and has had versions of those since Python 2\. For almost as long, there has been a custom cookie jar implementation for `python-requests`. The idea is to make the cookie jar feel like a Python dictionary. This might seem like a great ergonomic design, except that it's possible for two different domains to set cookies in the same Session with the same name, for example:

```
Set-Cookie: sessionId=38afes7a8; Domain=example.com; Path=/user/home; HttpOnly; Secure; SameSite=strict 
```

```
Set-Cookie: sessionId=29bedt6b9; Domain=google.com; Path=/; HttpOnly; Secure; SameSite=strict 
```

So if you use the same `requests.Session` for these cookies, how do you then access the cookie jar by cookie name (e.g., `sessionId`)? You get an exception!

That's not intuitive at all and it's why most (if not all) cookie jar implementations in languages always want you to use a real method that requires the domain name.

Ignoring the poorly thought out user experience, there's also a problem of cookie jar policies not working correctly with the `python-requests` cookie jar and no way to set a default one without significantly more work - so for the people who do care, their lives are measurably worse than if the library hadn't tried to handle cookies at all.

## Timeouts

`python-requests` supports setting timeouts per request, e.g.,

```
import requests

url: str = "..."
s: requests.Session = requests.Session()
# Timeout if connecting to the remote takes more than 2 seconds, or if it
# takes more than 2 seconds for the remote to write bytes in response
r = s.get(url, timeout=2)
# Timeout after 2s during connect, after 10s if no data written
r = s.get(url, timeout=(2, 10)) 
```

This works well, except, once again when the behaviour is surprising to the user.

`python-requests` will try each record in a DNS lookup when trying to connect to a remote. If this is unclear to you, a DNS lookup on a domain can return multiple results, e.g.,

```
; <<>> DiG 9.18.20 <<>> salesforce.com
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 22322
;; flags: qr rd ra; QUERY: 1, ANSWER: 8, AUTHORITY: 0, ADDITIONAL: 1

;; OPT PSEUDOSECTION:
; EDNS: version: 0, flags:; udp: 65494
;; QUESTION SECTION:
;salesforce.com.                        IN      A

;; ANSWER SECTION:
salesforce.com.         120     IN      A       184.31.10.133
salesforce.com.         120     IN      A       184.31.3.130
salesforce.com.         120     IN      A       104.109.10.129
salesforce.com.         120     IN      A       184.25.179.132
salesforce.com.         120     IN      A       23.1.35.132
salesforce.com.         120     IN      A       23.1.99.130
salesforce.com.         120     IN      A       104.109.11.129
salesforce.com.         120     IN      A       23.1.106.133

;; Query time: 63 msec
;; MSG SIZE  rcvd: 171 
```

So if we were making a request to `salesforce.com`, and the first seven IP addresses were unreachable, you would be waiting approximately 14 seconds at worst. Further if that eighth takes 1.8 seconds, your real-time wait (also known as "wall clock") is at least 15.8 seconds before you start receiving data. If you have a different read timeout, the server can take almost up to that to send the data. So let's say `23.1.106.133` takes 14 seconds to write it's first byte and your timeout is set to 15 seconds, then your wall clock experience is over 30 seconds. That's very different from 17 seconds like you might otherwise expect.

## Retries

Today, if you revisit the [SSLError example](requests-sslerror) and inspect that exception itself you'll see something like:

```
requests.exceptions.SSLError: HTTPSConnectionPool(host='expired.badssl.com', port=443): Max retries exceeded with url: / (Caused by SSLError (SSLCertVerificationError(1, '[SSL: CERTIFICATE_VERIFY_FAILED] certificate verify failed: certificate has expired (_ssl.c:1000)'))) 
```

This is quite difficult to read, but allow me to briefly parse this for you:

1.  First we caught a `SSLCertVerificationError` and re-raised that as an `SSLError` with the original exception wrapped inside
2.  Then we caught the `SSLError` in the retry logic built into `urllib3` and realized we were out of retries so we wrapped the `SSLError` and raised a `MaxRetryError` exception indicating we'd exhausted retries
3.  Then requests caught the `MaxRetryError`, inspected it and raised an `SSLError` of it's own.

What I want to emphasize here is point 2\. This trips up so many users because they see `Max retries exceeded with url:` and didn't explicitly configure retries so they assume `python-requests` is doing something intelligent. It's not, it's setting `max_retries=0` by default.

To be clear, this isn't related to us attempting to connect to multiple addresses in a DNS record either. This is purely, "we've established a connection and are retrying the request".

There are a few ways to configure retries today in `python-requests` that [I've already documented here](/2014/12/retries-in-requests). What would be better would be to not have to reach into `urllib3` at all. Further, it would be significantly better if the exceptions weren't converted to strings such that after a layer or two you no longer can inspect the exceptions themselves. Again, if I show some examples of the exception above:

```
>>> sslerr
SSLError(MaxRetryError("HTTPSConnectionPool(host='expired.badssl.com', port=443): Max retries exceeded with url: / (Caused by SSLError(SSLCertVerificationError(1, '[SSL: CERTIFICATE_VERIFY_FAILED] certificate verify failed: certificate has expired (_ssl.c:1000)')))"))
>>> sslerr.args
(MaxRetryError("HTTPSConnectionPool(host='expired.badssl.com', port=443): Max retries exceeded with url: / (Caused by SSLError(SSLCertVerificationError(1, '[SSL: CERTIFICATE_VERIFY_FAILED] certificate verify failed: certificate has expired (_ssl.c:1000)')))"),)
>>> sslerr.args[0]
MaxRetryError("HTTPSConnectionPool(host='expired.badssl.com', port=443): Max retries exceeded with url: / (Caused by SSLError(SSLCertVerificationError(1, '[SSL: CERTIFICATE_VERIFY_FAILED] certificate verify failed: certificate has expired (_ssl.c:1000)')))")
>>> sslerr.args[0].args
"HTTPSConnectionPool(host='expired.badssl.com', port=443): Max retries exceeded with url: / (Caused by SSLError(SSLCertVerificationError(1, '[SSL: CERTIFICATE_VERIFY_FAILED] certificate verify failed: certificate has expired (_ssl.c:1000)')))" 
```

This hides a great deal of information from the user which is not present in either the `SSLError` or `MaxRetryError` exception instances.

Without being able to introspect things you don't get a great way of detecting what happened in your code to more intelligently handle an exception.

To be clear, `urllib3` 's retries are *excellent*, but for `python-requests` users, it doesn't behave exactly as they'd expect. They don't get any information about what happened and they have very little visibility into how it all works since it happens in a different place. Furthermore, if they have some code that integrates with a `Session` object to authenticate or track a rate-limit, then this does not interact with that at all. As a result, a lot of functionality is lost when those users end up using `urllib3` retries (because there are no `python-requests` retries).

As an interim to anything improving in `python-requests` (which it likely won't), I **strongly** recommend [stamina](https://github.com/hynek/stamina)

## Transport Adaptors

Transport Adapters were a great idea that were introduced in 1.0\. These were intended to help users customize the behaviour of things and allow them to support URI schemes other than `http` and `https` (e.g., `file`, `unix`). This is an undisputed success (with a bunch of bumps along the way). There are libraries that implement those two popular schemes already.

Where things tend to fall apart is when you look at what can be specified in a Transport Adapter. `python-requests` uses a `urllib3.PoolManager` for the default `HTTPAdapter`. There are configuration options for that manager that are configurable in the default adapter so folks who know they need to alter those can create a new one with the values they want to use. The problem occurs when there are other options or request parameters that at this point *need* to be specified at the adapter layer. If there are additional options you want to configure on the connection's socket, you need to effectively sub-class the `HTTPAdapter` to add things to the initialization of the `PoolManager`. .

If you've ever used Golang's `net/http` library, you may have seen that the `Client` type has a `Transport` member defined as an interface `RoundTripper` . A popular pattern looks like:

```
import  "net/http"

type  CustomTransport  struct  {
  T  http.RoundTripper
}

func  (t  CustomTransport)  RoundTrip(req  *http.Request)  (*http.Response,  error)  {
  newReq  :=  modifyRequest(req)
  return  t.T.RoundTrip(req)
}

func  modifyRequest(req  *http.Request)  *http.Request  {
  reqCopy  :=  new(http.Request)
  *reqCopy  =  *req
  // Do something with copy
  return  reqCopy
}

func  main()  {
  client  :=  &http.Client{
  Transport:  CustomTransport{http.DefaultClient.Transport},
  }
} 
```

This is very easy to compose multiple transports. Behaviour like this is not easily or readily implemented with `python-requests`. It's possible depending on how you structure things or need them to work, but by and large, most people can't do this because of what they tend to want to change in their custom adapter. This means that if you find yourself needing a custom adapter, you suddenly need to understand a lot more about both `python-requests` and `urllib3` as well as how the adapter is used in `python-requests`. This is the right idea, but the wrong implementation and at this point, it's very hard to fix without breaking lots of users.

## Request Preparation

As you may have guessed from my [thoughts on github3.py](/2024/01/a-retrospective-on-github3py), I have strong opinions about things that need to be simple. Today, under the covers the API functional parameters are converted to a `Request` object which is then "prepared" into a `PreparedRequest`. The `PreparedRequest` is mutable and is what does all of the preparation. All of the code that prepares thing is tied to the `PreparedRequest` class. Organization aside, these classes are intended to be the interface for people who want to do things very differently from how `python-requests` intends. Do you want to do weird things with your URL? Use a `Request`, `prepare()` it, then `Session.send()` it. What could go wrong? Well:

*   authentication is stored as a property on a `Session`, is a parameter to the functional API, and is also something you can attach to a `Request`
*   likewise, a `Session` stores cookies and will attach them to appropriate requests (based on the cookie settings) and you can pass cookies into your request
*   headers can be set on a `Session` and are sent into a `Request` as well

So what happens when you need to use this flow? Well, for that reason we added `Session.prepare_request` but still there are issues people run into. Some people use this flow to avoid aspects of a `Session` but the most reliable way of getting what you expect from the flow is to involve the `Session`.

## Authentication

By and large, this actually typically works pretty well. If you're using a `Session` to only ever talk to one remote, you can attach the authentication to the `Session` and have it applied to every request. If you're not, the basic authentication types that you can attach to a request don't really understand that. It is trivial to write something, however, that does understand that as an authentication handler for Requests ([requests-toolbelt has a handler for this](https://toolbelt.readthedocs.io/en/latest/authentication.html#authhandler)). Beyond that, as I alluded to above when talking about retries, if you want to leverage the excellent retry logic baked into urllib3, you no longer get the authentication handler to apply it's context to a new request. So even the toolbelt's authentication handler couldn't handle the case where you've told it how to manage credentials for 2 or more domains and one redirects you to the other. If urllib3 handles that redirect as part of it's retry logic, then you'll likely end up with an exception instead of a response that can trigger the re-authentication flow.

## Internal Design May Be Artful, But It's Not Good Software

After receiving our first security disclosure, I was told that Requests wasn't a serious project but instead one person's art project and thus we shouldn't fix the vulnerability. This was despite the project being touted as being used by multiple international government agencies, political campaigns, and boasting about it's #1 download spot on PyPI. So when I say it might be artful, I'm trying to take a neutral stance on what is art and what isn't art and whether the internals of Requests are actually beautiful art.

As far as I'm concerned, the factoring of the code is very poor. It makes it borderline unmaintainable.

### `SessionRedirectMixin`

Requests handles redirects because early on, urllib3 either didn't or it's handling elided the intermediate request and response history hiding it from the user (I really don't remember which). Either way, in the great 1.0 refactor that pissed off nearly every user and wiped out all of the hard fought test coverage, we got a [mixin class](https://en.wikipedia.org/wiki/Mixin) that is only ever mixed into a single class which it shares a module with. The class has a lot of redirect logic in it and it expects a lot of methods and properties that are defined on the class it's mixed into. In other words, it's not really a mixin. It's also not a neat abstraction or useful refactoring.

I think it could have been a step in the right direction with code that didn't expect to have the ability to make a new request but instead took "configuration" and spat out an instruction to the Session as to what to do next (follow the redirect, stop due to a non-redirect status, etc.). That is not what we got.

### utils

Requests has always had modules intended for its use and only its use. At some point, we tried to document them for ourselves and that led to people using them externally and then filing bugs against them. This is a perfect example of 3 adults who did not consent explicitly to the use, but Python says that we must have because we didn't hide them well enough.

### Connection Pooling

One feature requests always advertised as if it was its own was connection pooling. However, that feature was always based entirely on urllib3's connection pooling logic. And on top of that, we didn't even use it correctly.

Today, each Transport Adapter instantiates its own [PoolManager](https://urllib3.readthedocs.io/en/stable/reference/urllib3.poolmanager.html#urllib3.PoolManager). This is to avoid having to manage [Connection Pools](https://urllib3.readthedocs.io/en/stable/reference/urllib3.connectionpool.html) in the Transport Adapter. However, a single Pool Manager could be used for both adapters without issue. There's no security reason not to do this and certainly no performance reason to prefer separate pools.

## Requests Often Believes It Knows Better than urllib3

Until [recently](https://github.com/psf/requests/pull/6226/files), Requests worked around urllib3 to do chunked transfer-encoding requests. This required digging into urllib3 a bit more than that library reasonably expects users to do and introduced bugs long term as urllib3 continued to evolve. On its face this is one of those technical debt items that makes sense in the moment but in reality should have been contributed to urllib3 and then immediately removed from Requests.

I can speak for the fact that we (the current maintainers) just don't have that kind of time but this ties back into a post I wrote recently about [saying no](https://blog.ian.stapletoncordas.co/2023/11/no-should-be-your-default). If I had known better, I would absolutely have argued against accepting this in `python-requests` at all. That would have provided some pressure to move this to urllib3 directly and make it work for `python-requests` rather than the opposite way things happened.

## Proxies

While `python-requests` can handle proxies, it can't handle sophisticated proxies that modern businesses rely on as well. Specifically, its support for TLS over HTTPS proxies is very poor. In order to support internal proxies that likely have their own trust bundles, we would need to significantly change the API of Requests. There are many ways we could do this, but every time we add to the API, it adds to the maintenance burden and it invariably upsets someone.

There was a very promising [pull request to add support for HTTPS proxies](https://github.com/psf/requests/pull/5665) and it does *the right thing* by relying on the logic in urllib3\. However, adding that support correctly creates a conundrum. How do we support those custom TLS bundles? The change goes about this by expanding the API by adding a parameter to the API. The number of parameters to the request methods is already huge and there is no end of drift between the documentation of those parameters on the various places they're supported.

My preference here is to do something that makes what is happening clearer, but that's also a different kind of API expansion. More directly, I'd like to continue supporting the current way of specifying proxies as a dictionary mapping the protocol to the proxy URI and then translate that to a dictionary that maps the protocol to an object that holds the configuration. Then for folks that know what they need for this, they can instead reach for that object directly, e.g.,

```
import ssl

import requests
from requests.proxies import Proxies Proxy

proxy_ssl_context = ssl.SSLContext()
proxy_ssl_context.load_verify_location(cafile="/etc/pki/corp.net/cacerts.pem")
proxy_ssl_context.verify_mode = ssl.CERT_REQUIRED

url: str = "..."
proxies = Proxies.from_dict({
   "http": "http://plain.proxy",
   "https": Proxy(
      uri="https://encrypted.proxy",
      ssl_context=proxy_ssl_context,
   ),
})
r: requests.Response = requests.get(url, proxies=proxies) 
```

Under the covers, any dictionary that was the prior API would be transparently converted to this `Proxies` object. The difficulty here is that `Session` has an attribute called `proxies` and people expect it to be a dictionary (reasonably so because that's what it has been for roughly a decade). This brings us then to the next pattern I truly abhor.o

## Compatibility Objects That Pretend To Be Standard Types

So many things were initially designed to be a simple built-in type in Python and as the user base grew we found that to be insufficient. We always need something that gives us the ability to do things that give users an escape hatch of sorts to do more complex things.

To then preserve backwards compatibility, we reach for the ability to implement those interfaces into the new objects and pretend that the code relying on the old thing will just continue to work. This almost always goes sideways eventually.

For one thing, the already often incorrect typeshed hints for `python-requests` are immediately wrong again. For another, people tend to make changes in `python-requests` as drive-by contributions that end up breaking these subtly which we don't discover for quite a while because they're so subtle.

## Test Coverage Is Abysmal

Of course, everything above is both a contributing factor to and a result of the abysmal test coverage in `python-requests`. The measure of coverage may not itself be horrible. But the test cases are not an appropriate number of permutations to determine the behaviour of some of the things users try that are not wise or advisable. As a result, sometimes something appears to work for a given user and when something else changes subtly it is broken. In reality, the API should have been designed to prevent those things from ever working but that never happened and breaking that now would appear to many as a violation of backwards compatibility.

## Stable APIs and Behaviour

Last but not least, after 1.0 there was a very concerted effort to not break users like that again, ever. That has led to a significant amount of friction in `python-requests`. We struggle often to be able to make an informed decision about what will not cause issues for users versus what will. This means that completely reasonable requests are blocked until we have the time and ability to make a backwards incompatible release (e.g., 3.0 - which itself already has a lot of baggage tied to it). This leads invariably to frustrated users and maintainers.

## Conclusion

There's a lot of history in `python-requests`. There's a lot of reasons for it to have existed but there's also a lot of reasons why despite it's "for Humans" tag line, it really doesn't feel that way for a lot of people.

There's a lot that can be improved, but really if you're asking yourself why hasn't it been just look back at the very last section. The project effectively has had it's feet cemented to the ground by one very disruptive release many years ago and hasn't been able to move on. Similarly, we'd love to fix a lot of these, and I've started countless times to try to redesign things with backwards compatibility to allow us to move past these, but those changes are huge refactors that we just don't have the appetite for as a team of three. Beyond that, we struggle with what to name the next version.

A lot of promises were made without any apparent intention of following through on them for a 3.0 release. And now, if we were to release a 3.0, we'd have to explain why all that promised work wasn't done (and to the folks that sent money, what happened to their money).

In short, the project feels dead. That's a shame, but that's my feeling on the matter. It's hard to introduce new, necessary, and beneficial features. It's hard to fix gnarly bugs. It's hard to improve the user experience and it's consistently been because of one particular person over the years. A few of us have tried to bring that stability but it has never seemed to be enough.