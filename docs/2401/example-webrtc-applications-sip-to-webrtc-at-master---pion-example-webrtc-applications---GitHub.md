<!--yml

类别：未分类

日期：2024-05-27 14:29:41

-->

# example-webrtc-applications/sip-to-webrtc at master · pion/example-webrtc-applications · GitHub

> 来源：[`github.com/pion/example-webrtc-applications/tree/master/sip-to-webrtc`](https://github.com/pion/example-webrtc-applications/tree/master/sip-to-webrtc)

SIP 是接受入站 SIP 流量（邀请）并将其与 WebRTC 进行桥接的示例。这是连接电话呼叫与您的 WebRTC 应用程序的最常见方式。这是可能的，因为有优秀的 [emiago/sipgo](https://github.com/emiago/sipgo) 库。

此示例演示了接受 SIP 音频并在浏览器中播放的过程。如果您希望实现呼叫中的多个参与者，您需要进行音频混音。请参阅 [zaf/g711](https://github.com/zaf/g711) 以进行解码 + 编码。解码后，将所有样本相加，然后进行编码。

### 打开 sip-to-webrtc 示例页面

[](#open-sip-to-webrtc-example-page)

[jsfiddle.net](https://jsfiddle.net/gds05mc3/) 您应该会看到一个音频播放器、两个文本区域和一个“开始会话”的按钮

## 使用浏览器的 SessionDescription 运行 sip-to-webrtc

[](#run-sip-to-webrtc-with-your-browsers-sessiondescription-as-stdin)

在 jsfiddle 中，顶部的文本区域是您的浏览器，请将其复制并：

运行 `echo $BROWSER_SDP | sip-to-webrtc`

1.  将 SessionDescription 粘贴到一个文件中。

1.  运行 `sip-to-webrtc < my_file`

### 在您的浏览器中输入 sip-to-webrtc 的 SessionDescription

[](#input-sip-to-webrtcs-sessiondescription-into-your-browser)

将 `sip-to-webrtc` 刚刚发出的文本复制并粘贴到第二个文本区域中

### 点击“开始会话”以连接

[](#hit-start-session-to-connect)

如果成功建立了 WebRTC 会话，您将收到关于 ICEConnectionState 变为 `connected` 的日志消息。在您的浏览器和终端中。

**浏览器**

**终端**

```
Connection State has changed checking
Connection State has changed connected

Starting SIP Listener 
```

如果一切正常，现在是时候发起 SIP 邀请了。

`sip-to-webrtc` 现在正在 `:5060` 上侦听，并将接受所有邀请。当邀请被接受时，您将看到如下的日志消息。

```
Accepting SIP Invite: From: "+15550100001" <sip:+15550100001@192.168.1.93>;tag=nc8uzmZUHUTbqH0v 
```

您应该会在浏览器中听到电话呼叫的音频。

恭喜，您已经使用了 Pion WebRTC！现在开始构建一些酷炫的东西吧。
