<!--yml

类别：未分类

日期：2024-05-27 13:34:11

-->

# 修补请求以获取乐趣和（并发）利润

> 来源：[https://blog.borrego.dev/entries/patching-requests-for-fun-and-concurrent-profit.html](https://blog.borrego.dev/entries/patching-requests-for-fun-and-concurrent-profit.html)

# 修补请求以获取乐趣和（并发）利润

2024年4月24日

因为生命太短暂，无法频繁调用`SSL_CTX_load_verify_locations()`。

## 问题在哪里？

让我们考虑以下脚本。它使用[requests](https://requests.readthedocs.io/en/latest/)库针对一个URL运行一堆并发请求，分别启用和禁用证书验证，并输出每种情况下所需的时间。

```
from time import time
from threading import Thread
import requests
import urllib3

# Mute warnings when cert verification is disabled
urllib3.disable_warnings()

def do_request(verify):
    requests.get('https://example.com', verify=verify)

def measure(verify):
    threads = [Thread(target=do_request, args=(verify,)) for _ in range(30)]

    start = time()
    for t in threads: t.start()
    for t in threads: t.join()
    end = time()

    print(end - start)

measure(verify=True)
measure(verify=False) 
```

两者之间的时间差异是多少？结果非常依赖于您的本地配置。在我的本地机器上，使用相对现代的配置（Python 3.12 + OpenSSL 3.0.2），`verify=True`时花费约`~1.2s`，而`verify=False`时花费约`~0.5s`。

这有超过100%的差异，但最初我们将其归咎于证书验证不是微不足道的，而且需要一些时间。然而，我们在一些环境中观察到了更大的差异（超过500%），决定找出问题所在。

## 发生了什么？

我们的主要请求用例是并发运行大量请求，并且我们花了一些时间对这种奇怪现象进行二分，看是否有性能优化的空间。

在验证证书后，通过对并发执行进行分析，问题变得更加清晰。这些是所有线程中花费时间最长的前三个函数调用：

```
ncalls  tottime  percall  cumtime  percall filename:lineno(function)
30/1    0.681    0.023    0.002    0.002 {method 'load_verify_locations' of '_ssl._SSLContext' objects}
30/1    0.181    0.006    0.002    0.002 {method 'connect' of '_socket.socket' objects}
60/2    0.180    0.003    1.323    0.662 {method 'read' of '_ssl._SSLSocket' objects} 
```

相反，这是没有证书验证时前三名的情况：

```
ncalls  tottime  percall  cumtime  percall filename:lineno(function)
30/1    0.233    0.008    0.001    0.001 {method 'do_handshake' of '_ssl._SSLSocket' objects}
30/1    0.106    0.004    0.002    0.002 {method 'connect' of '_socket.socket' objects}
60/2    0.063    0.001    0.505    0.253 {method 'read' of '_ssl._SSLSocket' objects} 
```

在第一种情况下，每个请求都会在`ssl`模块的`load_verify_locations()`函数中花费整整0.68秒，该函数配置一个`SSLContext`对象以使用一组根CA证书进行验证。这通过C FFI调用到OpenSSL的`SSL_CTX_load_verify_locations()`来完成，已知是[相当慢的](https://github.com/openssl/openssl/issues/16871)。这每个请求只需一次（因此左边的`30`）。

简单地看，0.68秒并不算什么大问题，但请注意，这比请求本身的网络等待时间**更长**。在高并发场景中，我们还观察到一些全局阻塞，这可能是因为每个FFI调用锁定了GIL，或者是OpenSSL本身的一些线程安全机制。无论哪种情况，随着并发请求数量的增加，性能损失都会变得越来越严重。我们也认为，这在不同OpenSSL版本之间的内部更改之间可能会更或少显著，因此在不同环境中的可变性。

当不需要证书验证时，这些调用会被跳过，这极大地提升了并发性能。

## 一个可能的修复方法

完全跳过加载根CA证书是不可能的，但也没有必要在每个请求中都这样做。在撰写本文时，每次调用`load_verify_locations()`时发生的情况如下：

+   创建了一个新的`urllib3.connectionpool.HTTPSConnectionPool`，或者

+   在每个连接中，通过urllib3的`ssl_wrap_socket()`，当连接的`ca_certs`或`ca_cert_dir`属性被设置时（参见[相关代码](https://github.com/urllib3/urllib3/blob/9929d3c4e03b71ba485148a8390cd9411981f40f/src/urllib3/util/ssl_.py#L438)）。

最近，Requests添加了一个内部的`_get_connection()`函数，用于缓存连接池，因此在大多数情况下，不再需要处理第一种情况，因为大多数HTTPS请求将共享相同的缓存`HTTPSConnectionPool`，而不是每个请求创建一个。

第二种情况在[提交的PR](https://github.com/psf/requests/pull/6667)中已解决。它创建了一个默认的`SSLContext`，其中已经加载了默认的CA证书，并修补了将每个连接与安全上下文连接起来以重用默认上下文的函数：

```
DEFAULT_SSL_CONTEXT = create_urllib3_context()
DEFAULT_SSL_CONTEXT.load_verify_locations(extract_zipped_paths(DEFAULT_CA_BUNDLE_PATH))

def _urllib3_request_context(
    ...
    cert_reqs = "CERT_REQUIRED"
    if verify is False:
        cert_reqs = "CERT_NONE"
+   elif verify is True:
+       pool_kwargs["ssl_context"] = DEFAULT_SSL_CONTEXT
+   elif isinstance(verify, str):
+       if not os.path.isdir(verify):
+           pool_kwargs["ca_certs"] = verify
+       else:
+           pool_kwargs["ca_cert_dir"] = verify
    pool_kwargs["cert_reqs"] = cert_reqs 
```

当`verify=True`时，我们使用已加载了CA证书的`SSLContext`提供连接池。正如所示，在这种情况下不设置`ca_certs`或`ca_cert_dir`，我们避免了urllib3不必要地调用`load_verify_locations()`，并且证书验证仍然按预期工作。在PR中还进行了一些额外的更改，仔细避免在不需要时设置这两个连接参数。

当`verify`不是布尔值，而是一个指示替代CA捆绑包或证书存储路径的字符串时，会发生一个小的警告。在这些情况下，我们相应地设置`ca_certs`或`ca_cert_dir`，以确保加载用户请求的替代CA捆绑包。这里还有进一步的优化空间，因为这种用例仍然会在每个请求中调用`load_verify_locations()`。也许可以通过根据请求希望使用的CA捆绑包缓存不同的`SSLContext`对象来泛化先前的方法。

* * *

更新：相应的[pull request](https://github.com/psf/requests/pull/6667)已被维护者接受并合并，并已在Requests的[2.32.0](https://github.com/psf/requests/blob/main/HISTORY.md#2320-2024-05-20)版本中发布了这些更改。
