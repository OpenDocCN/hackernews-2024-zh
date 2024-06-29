<!--yml

类别：未分类

日期：2024-05-27 13:19:15

-->

# WebSocketStream: 将流与 WebSocket API 集成 | 能力 | 开发者 Chrome

> 来源：[https://developer.chrome.com/docs/capabilities/web-apis/websocketstream](https://developer.chrome.com/docs/capabilities/web-apis/websocketstream)

通过施加反压力来防止您的应用程序被 WebSocket 消息淹没，或者向 WebSocket 服务器发送大量消息。

## 背景

### WebSocket API

[WebSocket API](https://developer.mozilla.org/docs/Web/API/WebSockets_API) 提供了一个 JavaScript 接口到 [WebSocket 协议](https://tools.ietf.org/html/rfc6455)，使得在用户的浏览器和服务器之间可以打开一个双向交互式通信会话成为可能。使用此 API，你可以向服务器发送消息并接收事件驱动的响应，而无需轮询服务器等待响应。

### 流 API

[流 API](https://developer.mozilla.org/docs/Web/API/Streams_API) 允许 JavaScript 可以编程访问通过网络接收到的数据块流，并根据需要处理它们。在流的上下文中的一个重要概念是 [反压力](https://developer.mozilla.org/docs/Web/API/Streams_API/Concepts#Backpressure)。这是通过单一流或管道链调节读取或写入速度的过程。当流本身或管道链中的后续流仍在忙碌并且尚未准备好接受更多数据块时，它会通过链向后发送信号以适当地减慢交付速度。

### 当前 WebSocket API 的问题

#### 对接收的消息施加反压力是不可能的

使用当前 WebSocket API，在 [`WebSocket.onmessage`](https://developer.mozilla.org/docs/Web/API/WebSocket/onmessage) 中响应消息。

假设你有一个应用程序，每当接收到新消息时都需要进行大量的数据处理操作。你可能会设置类似下面的流程，并且由于你在 `await` `process()` 调用的结果，你应该没问题，对吧？

```
// A heavy data crunching operation.
const process = async (data) => {
  return new Promise((resolve) => {
    window.setTimeout(() => {
      console.log('WebSocket message processed:', data);
      return resolve('done');
    }, 1000);
  });
};

webSocket.onmessage = async (event) => {
  const data = event.data;
  // Await the result of the processing step in the message handler.
  await process(data);
}; 
```

错了！当前 WebSocket API 的问题在于没有办法施加反压力。当消息比 `process()` 方法处理它们的速度更快时，渲染过程将会通过缓冲这些消息填满内存，或者由于 CPU 使用率达到 100%，变得无响应，或者两者兼而有之。

#### 对发送的消息施加反压力是非人性化的

对发送的消息施加背压是可能的，但涉及轮询[`WebSocket.bufferedAmount`](https://developer.mozilla.org/docs/Web/API/WebSocket/bufferedAmount)属性，这是低效且不符合人体工程学的。这个只读属性返回已通过调用[`WebSocket.send()`](https://developer.mozilla.org/docs/Web/API/WebSocket/send)排队但尚未传输到网络的数据字节数。一旦所有排队的数据都发送完毕，该值将重置为零，但如果你继续调用`WebSocket.send()`，它将继续增加。

## 什么是WebSocketStream API?

WebSocketStream API通过将流集成到WebSocket API中解决了不存在或不符合人体工程学的背压问题。这意味着可以免费应用背压，无需额外成本。

### WebSocketStream API的建议用例

可以使用此API的网站示例包括：

+   需要保持互动性的高带宽WebSocket应用，特别是视频和屏幕共享。

+   同样，视频捕获和其他在浏览器中生成大量数据并需要上传到服务器的应用程序。通过背压，客户端可以停止生成数据，而不是在内存中积累数据。

## 当前状态

| 步骤 | 状态 |
| --- | --- |
| 1\. 创建说明文档 | [完成](https://github.com/ricea/websocketstream-explainer/blob/master/README.md) |
| 2\. 创建规范初稿 | [进行中](https://github.com/ricea/websocketstream-explainer/blob/master/README.md) |
| 3\. 收集反馈并迭代设计 | [进行中](#feedback) |
| 4\. 原型试验 | [完成](https://developers.chrome.com/origintrials/#/view_trial/1977080236415647745) |
| 5\. 发布 | 未开始 |

## 如何使用WebSocketStream API

### 入门示例

WebSocketStream API是基于Promise的，这使得在现代JavaScript世界中处理它感觉自然。你首先通过构造一个新的`WebSocketStream`并传递WebSocket服务器的URL来开始。接下来，等待连接`opened`，这将产生一个[`ReadableStream`](https://developer.mozilla.org/docs/Web/API/ReadableStream/ReadableStream)和/或一个[`WritableStream`](https://developer.mozilla.org/docs/Web/API/WritableStream/WritableStream)。

通过调用[`ReadableStream.getReader()`](https://developer.mozilla.org/docs/Web/API/ReadableStream/getReader)方法，最终可以获得一个[`ReadableStreamDefaultReader`](https://developer.mozilla.org/docs/Web/API/ReadableStreamDefaultReader)，然后可以从中读取数据，直到流结束，即返回形式为`{value: undefined, done: true}`的对象为止。

因此，通过调用[`WritableStream.getWriter()`](https://developer.mozilla.org/docs/Web/API/WritableStream/getWriter)方法，最终可以获取[`WritableStreamDefaultWriter`](https://developer.mozilla.org/docs/Web/API/WritableStreamDefaultWriter)，然后可以向其[`write()`](https://developer.mozilla.org/docs/Web/API/WritableStreamDefaultWriter/write)数据。

```
 const wss = new WebSocketStream(WSS_URL);
  const {readable, writable} = await wss.opened;
  const reader = readable.getReader();
  const writer = writable.getWriter();

  while (true) {
    const {value, done} = await reader.read();
    if (done) {
      break;
    }
    const result = await process(value);
    await writer.write(result);
  } 
```

#### 背压

承诺的背压功能怎么样？如上所述，您可以"免费"获得它，无需额外步骤。如果`process()`花费额外时间，只有在管道准备好后才会消耗下一个消息。同样地，`WritableStreamDefaultWriter.write()`步骤只有在安全的情况下才会进行。

### 高级示例

WebSocketStream的第二个参数是一个选项包，允许未来扩展。目前唯一的选项是`protocols`，其行为与[WebSocket构造函数的第二个参数](https://developer.mozilla.org/docs/Web/API/WebSocket/WebSocket#Parameters:%7E:text=respond.-,protocols)相同：

```
const chatWSS = new WebSocketStream(CHAT_URL, {protocols: ['chat', 'chatv2']});
const {protocol} = await chatWSS.opened; 
```

所选的`协议`以及可能的`扩展`都是通过`WebSocketStream.opened`承诺可用的字典的一部分。由于连接失败时没有相关信息，因此此承诺提供了有关活动连接的所有信息。

```
const {readable, writable, protocol, extensions} = await chatWSS.opened; 
```

### 关于已关闭的WebSocketStream连接的信息

以前在WebSocket API的[`WebSocket.onclose`](https://developer.mozilla.org/docs/Web/API/WebSocket/onclose)和[`WebSocket.onerror`](https://developer.mozilla.org/docs/Web/API/WebSocket/onerror)事件中可用的信息现在通过`WebSocketStream.closed`承诺可用。如果出现非干净关闭，该承诺会拒绝，否则会解析为服务器发送的代码和原因。

所有可能的状态代码及其含义在[`CloseEvent`状态代码列表](https://developer.mozilla.org/docs/Web/API/CloseEvent#Status_codes)中有详细解释。

```
const {code, reason} = await chatWSS.closed; 
```

### 关闭WebSocketStream连接

可以使用[`AbortController`](https://developer.mozilla.org/docs/Web/API/AbortController)来关闭WebSocketStream。因此，请将[`AbortSignal`](https://developer.mozilla.org/docs/Web/API/AbortSignal)传递给`WebSocketStream`构造函数。

```
const controller = new AbortController();
const wss = new WebSocketStream(URL, {signal: controller.signal});
setTimeout(() => controller.abort(), 1000); 
```

作为替代方案，您还可以使用`WebSocketStream.close()`方法，但其主要目的是允许指定发送到服务器的[代码](https://developer.mozilla.org/docs/Web/API/CloseEvent#Status_codes)和原因。

```
wss.close({code: 4000, reason: 'Game over'}); 
```

### 逐步增强和互操作性

Chrome目前是唯一实现WebSocketStream API的浏览器。为了与传统的WebSocket API互操作，对接收消息应用背压是不可能的。对发送消息应用背压是可能的，但涉及到轮询[`WebSocket.bufferedAmount`](https://developer.mozilla.org/docs/Web/API/WebSocket/bufferedAmount)属性，这是低效且不符合人体工程学原则的。

#### 特性检测

要检查WebSocketStream API是否受支持，请使用：

```
if ('WebSocketStream' in window) {
  // `WebSocketStream` is supported!
} 
```

## 演示

在支持的浏览器上，您可以在嵌入的iframe中或[直接在Glitch上](https://websocketstream-demo.glitch.me/)看到WebSocketStream API的运行情况。

## 反馈

Chrome团队希望听听你对WebSocketStream API的使用体验。

### 谈谈API设计

API有什么不符合你预期的地方吗？或者有需要实现你想法的方法或属性缺失吗？对安全模型有疑问或评论吗？请在相应的[GitHub仓库](https://github.com/ricea/websocketstream-explainer/issues)上提交规范问题，或将你的想法添加到现有问题中。

### 报告实现中的问题

发现Chrome的实现有bug了吗？或者实现与规范不同？请在[new.crbug.com](https://new.crbug.com)报告bug。确保尽可能详细地描述，包括简单的重现指南，并在**组件**框中输入`Blink>Network>WebSockets`。[Glitch](https://glitch.com/)非常适合分享快速且简单的复现案例。

### 支持API

你计划使用WebSocketStream API吗？你的公开支持帮助Chrome团队优先考虑功能，并向其他浏览器供应商展示支持这些功能的重要性。

发一条推文到[@ChromiumDev](https://twitter.com/ChromiumDev)，使用[`#WebSocketStream`](https://twitter.com/search?q=%23WebSocketStream&src=typed_query&f=live)，告诉我们你在哪里以及如何使用它。

## 有用的链接

## 致谢

WebSocketStream API由[Adam Rice](https://github.com/ricea)和[Yutaka Hirano](https://github.com/yutakahirano)实现。[Daan Mooij](https://unsplash.com/@daanmooij)在[Unsplash](https://unsplash.com/photos/91LGCVN5SAI)上的英雄图片。
