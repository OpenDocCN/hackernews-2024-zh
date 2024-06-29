<!--yml

category: 未分类

date: 2024-05-29 13:27:30

-->

# Youtube video embedding harm reduction

> 来源：[https://dustri.org/b/youtube-video-embedding-harm-reduction.html](https://dustri.org/b/youtube-video-embedding-harm-reduction.html)

Embedding external content on a website in the current enshittocene period is more annoying than ever, so here is a copy-pasteable snippet to embed a youtube video while reducing its tracking and nuisance capabilities as much as possible:

```
`<iframe
    credentialless
    allowfullscreen
    referrerpolicy="no-referrer"
    sandbox="allow-scripts allow-same-origin"
    allow="accelerometer 'none'; ambient-light-sensor 'none'; autoplay 'none'; battery 'none'; bluetooth 'none'; browsing-topics 'none'; camera 'none'; ch-ua 'none'; display-capture 'none'; domain-agent 'none'; document-domain 'none'; encrypted-media 'none'; execution-while-not-rendered 'none'; execution-while-out-of-viewport 'none'; gamepad 'none'; geolocation 'none'; gyroscope 'none'; hid 'none'; identity-credentials-get 'none'; idle-detection 'none'; keyboard-map 'none'; local-fonts 'none'; magnetometer 'none'; microphone 'none'; midi 'none'; navigation-override 'none'; otp-credentials 'none'; payment 'none'; picture-in-picture 'none'; publickey-credentials-create 'none'; publickey-credentials-get 'none'; screen-wake-lock 'none'; serial 'none'; speaker-selection 'none'; sync-xhr 'none'; usb 'none'; web-share 'none'; window-management 'none'; xr-spatial-tracking 'none'"
    csp="sandbox allow-scripts allow-same-origin;"
    width="560"
    height="315"
    src="https://www.youtube-nocookie.com/embed/jfKfPfyJRdk"
    title="lofi hip hop radio 📚 - beats to relax/study to"
    frameborder="0"
    loading="lazy"
></iframe>` 
```

+   [`credentialless`](https://developer.mozilla.org/en-US/docs/Web/Security/IFrame_credentialless) 可以在一个空白的一次性上下文中加载 YouTube，而不访问来源网络、Cookie 和存储数据。

+   `allowfullscreen` because some people like it

+   `referrerpolicy` set to not leak your [referer](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Referer)

+   `sandbox` 设置为仅允许执行 JavaScript 和 SOP。下载、表单、模态框、屏幕方向、指针锁定、弹出窗口、演示会话、[存储访问](https://developer.mozilla.org/en-US/docs/Web/API/Storage_Access_API)，以及第三方 Cookie 等全部被拒绝。

+   `allow` 中的[每个单独指令](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Permissions-Policy#directives)都被设置为"absolutely-fucking-not"，是的，它们必须一个一个地设置，并且定期检查是否有新指令被添加，因为[规范](https://w3c.github.io/webappsec-permissions-policy/)中没有拒绝全部的选项。看起来每个浏览器都有自己的指令列表，Chrome 使用[这个](https://github.com/w3c/webappsec-permissions-policy/blob/main/features.md)，而Firefox 更喜欢[MDN 的这个](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Permissions-Policy#directives)，当然这两者是不同的。毫无疑问，这是以隐私、简单性、可维护性和安全性为设计目标。

+   `src` 设置为 `www.youtube-nocookie.com` 而不是 `youtube.com`。两者都是 Google 的官方 URL，但前者不通过 Cookie 进行追踪，并禁用 API 和交互记录。有趣的是，这是用于 `whitehouse.gov` 上的播放器。

+   `csp` set to `sandbox allow-scripts allow-same-origin;` for compatibility's sake, just in case. I'd love to use a more restrictive policy, but the spec doesn't allow to provide one, except if the embedded website explicitly allows it, and of course youtube doesn't.

+   `loading="lazy"` in case people don't scroll far enough to see the video, no need to make them do queries to Google for no reasons.

不要忘记为了[辅助功能的缘故](https://developer.mozilla.org/en-US/docs/Web/HTML/Element/iframe#accessibility_concerns)设置`title`。
