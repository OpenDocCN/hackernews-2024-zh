<!--yml

category: 未分类

date: 2024-05-27 13:04:46

-->

# ETag 和 HTTP 缓存 | Redowan's Reflections

> 来源：[https://rednafi.com/misc/etag_and_http_caching/](https://rednafi.com/misc/etag_and_http_caching/)

HTTP `ETag`头的一个好用例是`GET`请求的客户端端HTTP缓存。除了`ETag`头外，缓存工作流还要求您操作其他条件HTTP头，如`If-Match`或`If-None-Match`。然而，它们的交互有时可能会感到有些令人困惑。

每次我需要解决这个问题时，我最终会花费一些时间浏览相关的MDN文档 ^(^(^(以激发我的记忆。到目前为止，我已经做了足够多的次数来证明花时间写这篇文章是合理的。)))

## 缓存`GET`端点的响应[#](#caching-the-response-of-a-get-endpoint)

基本工作流程如下：

+   客户端向服务器发出`GET`请求。

+   服务器以`200 OK`状态响应，包括所请求的内容和一个`ETag`头。

+   客户端缓存响应和`ETag`值。

+   对于对同一资源的后续请求，客户端将包含带有其缓存的`ETag`值的`If-None-Match`头。

+   服务器独立重新生成`ETag`并检查客户端发送的`ETag`值是否与生成的值匹配。

    +   如果它们匹配，服务器将以`304 Not Modified`状态响应，表示客户端缓存版本仍然有效，客户端从缓存中提供资源。

    +   如果它们不匹配，服务器会响应`200 OK`状态，包括新内容和新的`ETag`头，提示客户端更新其缓存。

```
Client                                 Server  |                                       | |----- GET Request -------------------->| |                                       | |<---- Response 200 OK + ETag ----------| |     (Cache response locally)          | |                                       | |----- GET Request + If-None-Match ---->|  (If-None-Match == previous ETag) |                                       | |       Does ETag match?                | |<---- Yes: 304 Not Modified -----------|  (No body sent; Use local cache) |       No: 200 OK + New ETag ----------|  (Update cached response) |                                       | 
```

我们可以通过GitHub的REST API套件通过GitHub CLI测试此工作流^(. 如果您已安装CLI并对自己进行了身份验证，则可以像这样发出请求:)

这要求获取与用户`rednafi`关联的数据。响应如下：

```
HTTP/2.0 200 OK Etag: W/"b8fdfabd59aed6e0e602dd140c0a0ff48a665cac791dede458c5109bf4bf9463"   {  "login":"rednafi", "id":30027932, ... } 
```

我已经截断了响应主体并省略了此讨论无关的头部。您可以看到HTTP状态码为`200 OK`，服务器已包含`ETag`头。

`W/`前缀表示使用弱验证器 ^(用于验证缓存内容。使用弱验证器意味着当服务器比较响应有效载荷以生成哈希时，它不会逐位进行比较。因此，如果您的响应是JSON，则更改JSON的格式不会更改`ETag`头的值，因为两个具有相同内容但格式不同的JSON有效载荷在语义上是相同的。)

看看如果我们再次发出相同的请求，同时通过`If-None-Match`头传递`ETag`值会发生什么。

```
gh api -i -H \  'If-None-Match: W/"b8fdfabd59aed6e0e602dd140c0a0ff48a665cac791dede458c5109bf4bf9463"' \ /users/rednafi 
```

这返回：

```
HTTP/2.0 304 Not Modified Etag: "b8fdfabd59aed6e0e602dd140c0a0ff48a665cac791dede458c5109bf4bf9463"   gh: HTTP 304 
```

这意味着客户端中的缓存响应仍然有效，并且无需从服务器重新获取。因此，当用户请求时，客户端可以编码为向用户提供先前缓存的数据。

一些需要记住的关键点：

+   在发送具有`If-None-Match`头的`ETag`值时，始终将您的`ETag`值包装在双引号中，就像规范所述^。

+   使用`If-None-Match`头传递ETag值意味着当来自客户端的ETag值与服务器的不匹配时，客户端请求被视为成功。当值匹配时，服务器将返回`304 Not Modified`，并且没有正文。

+   如果我们正在编写一个符合规范的服务器，当涉及到`If-None-Match`时，规范告诉我们^对ETag使用弱比较。这意味着即使数据表示中有轻微变化，客户端仍然可以使用弱ETag验证缓存。

+   如果客户端是浏览器，它会自动管理缓存并发送条件请求，无需额外操作。

## 撰写一个允许客户端缓存的服务器[#](#writing-a-server-that-enables-client-side-caching)

如果您提供静态内容，可以配置负载均衡器以启用此缓存工作流程。但对于动态`GET`请求，服务器需要做更多工作以允许客户端缓存。

下面是一个简单的Go服务器，用于动态`GET`请求的上述工作流程：

```
package main   import (  "crypto/sha256" "encoding/hex" "fmt" "net/http" "strings" )   // calculateETag generates a weak ETag by SHA-256-hashing the content // and prefixing it with W/ to indicate a weak comparison func calculateETag(content string) string {  hasher := sha256.New() hasher.Write([]byte(content)) hash := hex.EncodeToString(hasher.Sum(nil)) return fmt.Sprintf("W/\"%s\"", hash) }   func main() {  http.HandleFunc("/", func(w http.ResponseWriter, r *http.Request) { // Define the content within the handler content := `{"message": "Hello, world!"}` eTag := calculateETag(content)   // Remove quotes and W/ prefix for If-None-Match header comparison ifNoneMatch := strings.TrimPrefix( strings.Trim(r.Header.Get("If-None-Match"), "\""), "W/")   // Generate a hash of the content without the W/ prefix for comparison contentHash := strings.TrimPrefix(eTag, "W/")   // Check if the ETag matches; if so, return 304 Not Modified if ifNoneMatch == strings.Trim(contentHash, "\"") { w.WriteHeader(http.StatusNotModified) return }   // If ETag does not match, return the content and the ETag w.Header().Set("ETag", eTag) // Send weak ETag w.Header().Set("Content-Type", "application/json") w.WriteHeader(http.StatusOK) fmt.Fprint(w, content) })   fmt.Println("Server is running on http://localhost:8080") http.ListenAndServe(":8080", nil) } 
```

+   服务器通过创建SHA-256哈希为其内容生成弱`ETag`，并在前面加上`W/`来表示它用于弱比较。

    您可以使`calculateETag`函数与格式无关，这样如果JSON格式更改但内容不变，哈希值仍然相同。当前的`calculateETag`实现容易受格式更改的影响，但我保留了这一点以保持代码简短。

+   在传递内容时，服务器在响应头中包含这个弱`ETag`，允许客户端缓存内容和`ETag`一起。

+   对于后续请求，服务器会检查客户端是否在`If-None-Match`头中发送了一个`ETag`，并通过独立生成哈希来弱比较当前内容的`ETag`。

+   如果ETag匹配，表示内容没有显著更改，服务器以`304 Not Modified`状态回复。否则，它以`200 OK`状态重新发送内容并更新`ETag`。当这种情况发生时，客户端知道现有缓存仍然有效，可以无需更改地提供服务。

您可以通过运行`go run main.go`来启动服务器，然后从不同的控制台开始像这样进行请求：

```
curl -i  http://localhost:8080/foo 
```

这将与JSON响应一起返回`ETag`头：

```
HTTP/1.1 200 OK Content-Type: application/json Etag: W/"1d3b4242cc9039faa663d7ca51a25798e91fbf7675c9007c2b0470b72c2ed2f3" Date: Wed, 10 Apr 2024 15:54:33 GMT Content-Length: 28   {"message": "Hello, world!"} 
```

现在，您可以使用以下值进行另一个请求

`If-None-Match`头中的ETag：

```
curl -i -H \  'If-None-Match: "1d3b4242cc9039faa663d7ca51a25798e91fbf7675c9007c2b0470b72c2ed2f3"' \ http://localhost:8080/foo 
```

这将返回一个`304 Not Modified`响应，无需正文：

```
HTTP/1.1 304 Not Modified Date: Wed, 10 Apr 2024 15:57:25 GMT 
```

在实际情况下，您可能会将中间件中的缓存部分提取出来，以便所有HTTP`GET`请求都可以从客户端缓存中获取，避免重复。

## 有一件事需要注意[#](#one-thing-to-look-out-for)

在编写支持缓存的服务器时，确保系统设置使服务器对于相同内容始终返回相同的`ETag`，即使多个服务器在负载均衡器后面工作。如果这些服务器对于相同内容给出不同的ETag，会影响客户端如何缓存该内容。

客户端使用ETag来判断内容是否已更改。如果`ETag`值未更改，他们知道内容相同，就不会再次下载，节省带宽并加快访问速度。但是如果服务器之间的ETag不一致，客户端可能会下载已经存在的内容，浪费带宽并降低速度。

这种不一致性也意味着服务器最终会处理更多的内容请求，而客户端本可以直接从缓存中使用这些内容，如果ETag是一致的话。

## 最近的帖子

+   [Protobuf协议的契约](https://rednafi.com/misc/protobuffed_contracts/)*   [TypeIs在Python中做了我以为TypeGuard会做的事情](https://rednafi.com/python/typeguard_vs_typeis/)*   [跨越CORS的十字路口](https://rednafi.com/misc/crossing_the_cors_crossroad/)*   [Go语言中的功能不全选项模式](https://rednafi.com/go/dysfunctional_options_pattern/)*   [艾因斯特效应](https://rednafi.com/zephyr/einstellung_effect/)*   [Go语言中的策略模式](https://rednafi.com/go/strategy_pattern/)*   [Go语言中的贫血堆栈跟踪](https://rednafi.com/go/anemic_stack_traces/)*   [Go语言中的重试函数](https://rednafi.com/go/retry_function/)*   [Go语言中的类型断言与类型开关](https://rednafi.com/go/type_assertion_vs_type_switches/)*   [在pytest中修补pydantic设置](https://rednafi.com/python/patch_pydantic_settings_in_pytest/)
