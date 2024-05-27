<!--yml
category: 未分类
date: 2024-05-27 13:19:15
-->

# WebSocketStream: integrating streams with the WebSocket API  |  Capabilities  |  Chrome for Developers

> 来源：[https://developer.chrome.com/docs/capabilities/web-apis/websocketstream](https://developer.chrome.com/docs/capabilities/web-apis/websocketstream)

Prevent your app from getting drowned in WebSocket messages or flooding a WebSocket server with messages by applying backpressure.

## Background

### The WebSocket API

The [WebSocket API](https://developer.mozilla.org/docs/Web/API/WebSockets_API) provides a JavaScript interface to the [WebSocket protocol](https://tools.ietf.org/html/rfc6455), which makes it possible to open a two-way interactive communication session between the user's browser and a server. With this API, you can send messages to a server and receive event-driven responses without polling the server for a reply.

### The Streams API

The [Streams API](https://developer.mozilla.org/docs/Web/API/Streams_API) allows JavaScript to programmatically access streams of data chunks received over the network and process them as desired. An important concept in the context of streams is [backpressure](https://developer.mozilla.org/docs/Web/API/Streams_API/Concepts#Backpressure). This is the process by which a single stream or a pipe chain regulates the speed of reading or writing. When the stream itself or a stream later in the pipe chain is still busy and isn't yet ready to accept more chunks, it sends a signal backwards through the chain to slow delivery as appropriate.

### The Problem with the current WebSocket API

#### Applying backpressure to received messages is impossible

With the current WebSocket API, reacting to a message happens in [`WebSocket.onmessage`](https://developer.mozilla.org/docs/Web/API/WebSocket/onmessage), an `EventHandler` called when a message is received from the server.

Let's assume you had an application that needs to perform heavy data crunching operations whenever a new message is received. You would probably set up the flow similar to the code below, and since you `await` the result of the `process()` call, you should be good, right?

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

Wrong! The problem with the current WebSocket API is that there is no way to apply backpressure. When messages arrive faster than the `process()` method can handle them, the render process will either fill up memory by buffering those messages, become unresponsive due to 100% CPU usage, or both.

#### Applying backpressure to sent messages is non-ergonomic

Applying backpressure to sent messages is possible, but involves polling the [`WebSocket.bufferedAmount`](https://developer.mozilla.org/docs/Web/API/WebSocket/bufferedAmount) property, which is inefficient and non-ergonomic. This read-only property returns the number of bytes of data that have been queued using calls to [`WebSocket.send()`](https://developer.mozilla.org/docs/Web/API/WebSocket/send), but not yet transmitted to the network. This value resets to zero once all queued data has been sent, but if you keep calling `WebSocket.send()`, it will continue to climb.

## What is the WebSocketStream API?

The WebSocketStream API deals with the problem of non-existent or non-ergonomic backpressure by integrating streams with the WebSocket API. This means backpressure can be applied "for free", without any extra cost.

### Suggested use cases for the WebSocketStream API

Examples of sites that can use this API include:

*   High-bandwidth WebSocket applications that need to retain interactivity, in particular video and screen-sharing.
*   Similarly, video capture and other applications that generate a lot of data in the browser that needs to be uploaded to the server. With backpressure, the client can stop producing data rather than accumulating data in memory.

## Current status

| Step | Status |
| --- | --- |
| 1\. Create explainer | [Complete](https://github.com/ricea/websocketstream-explainer/blob/master/README.md) |
| 2\. Create initial draft of specification | [In progress](https://github.com/ricea/websocketstream-explainer/blob/master/README.md) |
| 3\. Gather feedback & iterate on design | [In progress](#feedback) |
| 4\. Origin trial | [Complete](https://developers.chrome.com/origintrials/#/view_trial/1977080236415647745) |
| 5\. Launch | Not started |

## How to use the WebSocketStream API

### Introductory example

The WebSocketStream API is promise-based, which makes dealing with it feel natural in a modern JavaScript world. You start by constructing a new `WebSocketStream` and passing it the URL of the WebSocket server. Next, you wait for the connection to be `opened`, which results in a [`ReadableStream`](https://developer.mozilla.org/docs/Web/API/ReadableStream/ReadableStream) and/or a [`WritableStream`](https://developer.mozilla.org/docs/Web/API/WritableStream/WritableStream).

By calling the [`ReadableStream.getReader()`](https://developer.mozilla.org/docs/Web/API/ReadableStream/getReader) method, you finally obtain a [`ReadableStreamDefaultReader`](https://developer.mozilla.org/docs/Web/API/ReadableStreamDefaultReader), which you can then [`read()`](https://developer.mozilla.org/docs/Web/API/ReadableStreamDefaultReader/read) data from until the stream is done, that is, until it returns an object of the form `{value: undefined, done: true}`.

Accordingly, by calling the [`WritableStream.getWriter()`](https://developer.mozilla.org/docs/Web/API/WritableStream/getWriter) method, you finally obtain a [`WritableStreamDefaultWriter`](https://developer.mozilla.org/docs/Web/API/WritableStreamDefaultWriter), which you can then [`write()`](https://developer.mozilla.org/docs/Web/API/WritableStreamDefaultWriter/write) data to.

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

#### Backpressure

What about the promised backpressure feature? As I wrote above, you get it "for free", no extra steps needed. If `process()` takes extra time, the next message will only be consumed once the pipeline is ready. Likewise the `WritableStreamDefaultWriter.write()` step will only proceed if it is safe to do so.

### Advanced examples

The second argument to WebSocketStream is an option bag to allow for future extension. Currently the only option is `protocols`, which behaves the same as the [second argument to the WebSocket constructor](https://developer.mozilla.org/docs/Web/API/WebSocket/WebSocket#Parameters:%7E:text=respond.-,protocols):

```
const chatWSS = new WebSocketStream(CHAT_URL, {protocols: ['chat', 'chatv2']});
const {protocol} = await chatWSS.opened; 
```

The selected `protocol` as well as potential `extensions` are part of the dictionary available via the `WebSocketStream.opened` promise. All the information about the live connection is provided by this promise, since it is not relevant if the connection fails.

```
const {readable, writable, protocol, extensions} = await chatWSS.opened; 
```

### Information about closed WebSocketStream connection

The information that was available from the [`WebSocket.onclose`](https://developer.mozilla.org/docs/Web/API/WebSocket/onclose) and [`WebSocket.onerror`](https://developer.mozilla.org/docs/Web/API/WebSocket/onerror) events in the WebSocket API is now available via the `WebSocketStream.closed` promise. The promise rejects in the event of an unclean close, otherwise it resolves to the code and reason sent by the server.

All possible status codes and their meaning is explained in the [list of `CloseEvent` status codes](https://developer.mozilla.org/docs/Web/API/CloseEvent#Status_codes).

```
const {code, reason} = await chatWSS.closed; 
```

### Closing a WebSocketStream connection

A WebSocketStream can be closed with an [`AbortController`](https://developer.mozilla.org/docs/Web/API/AbortController). Therefore, pass an [`AbortSignal`](https://developer.mozilla.org/docs/Web/API/AbortSignal) to the `WebSocketStream` constructor.

```
const controller = new AbortController();
const wss = new WebSocketStream(URL, {signal: controller.signal});
setTimeout(() => controller.abort(), 1000); 
```

As an alternative, you can also use the `WebSocketStream.close()` method, but its main purpose is to permit specifying the [code](https://developer.mozilla.org/docs/Web/API/CloseEvent#Status_codes) and reason which is sent to the server.

```
wss.close({code: 4000, reason: 'Game over'}); 
```

### Progressive enhancement and interoperability

Chrome is currently the only browser to implement the WebSocketStream API. For interoperability with the classic WebSocket API, applying backpressure to received messages is not possible. Applying backpressure to sent messages is possible, but involves polling the [`WebSocket.bufferedAmount`](https://developer.mozilla.org/docs/Web/API/WebSocket/bufferedAmount) property, which is inefficient and non-ergonomic.

#### Feature detection

To check if the WebSocketStream API is supported, use:

```
if ('WebSocketStream' in window) {
  // `WebSocketStream` is supported!
} 
```

## Demo

On supporting browsers, you can see the WebSocketStream API in action in the embedded iframe, or [directly on Glitch](https://websocketstream-demo.glitch.me/).

## Feedback

The Chrome team wants to hear about your experiences with the WebSocketStream API.

### Tell us about the API design

Is there something about the API that doesn't work like you expected? Or are there missing methods or properties that you need to implement your idea? Have a question or comment on the security model? File a spec issue on the corresponding [GitHub repo](https://github.com/ricea/websocketstream-explainer/issues), or add your thoughts to an existing issue.

### Report a problem with the implementation

Did you find a bug with Chrome's implementation? Or is the implementation different from the spec? File a bug at [new.crbug.com](https://new.crbug.com). Be sure to include as much detail as you can, simple instructions for reproducing, and enter `Blink>Network>WebSockets` in the **Components** box. [Glitch](https://glitch.com/) works great for sharing quick and easy reproduction cases.

### Show support for the API

Are you planning to use the WebSocketStream API? Your public support helps the Chrome team to prioritize features and shows other browser vendors how critical it is to support them.

Send a tweet to [@ChromiumDev](https://twitter.com/ChromiumDev) using the hashtag [`#WebSocketStream`](https://twitter.com/search?q=%23WebSocketStream&src=typed_query&f=live) and let us know where and how you're using it.

## Helpful links

## Acknowledgements

The WebSocketStream API was implemented by [Adam Rice](https://github.com/ricea) and [Yutaka Hirano](https://github.com/yutakahirano). Hero image by [Daan Mooij](https://unsplash.com/@daanmooij) on [Unsplash](https://unsplash.com/photos/91LGCVN5SAI).