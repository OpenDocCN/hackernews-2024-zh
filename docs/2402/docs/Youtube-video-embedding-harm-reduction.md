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

*   [`credentialless`](https://developer.mozilla.org/en-US/docs/Web/Security/IFrame_credentialless) to load youtube in a blank disposable context, without access to the origin's network, cookies, and storage data.
*   `allowfullscreen` because some people like it
*   `referrerpolicy` set to not leak your [referer](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Referer)
*   `sandbox` to only allow javascript execution and SOP. Downloads, forms, modals, screen orientation, pointer lock, popups, presentation session, [storage access](https://developer.mozilla.org/en-US/docs/Web/API/Storage_Access_API) and thus third-party cookies, top-navigation, … are all denied.
*   `allow` with [every single directives](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Permissions-Policy#directives) set to "absolutely-fucking-not", and yes, they have to be all set one by one, and check regularly is new directive were added, because there is [no deny-all](https://github.com/w3c/webappsec-permissions-policy/issues/208) in the [spec](https://w3c.github.io/webappsec-permissions-policy/). It seems that every browser has its own list of directives, chrome is using [this one](https://github.com/w3c/webappsec-permissions-policy/blob/main/features.md) while firefox' prefers the [MDN one](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Permissions-Policy#directives), and of course the two differ. No doubt this was designed with privacy, simplicity, maintainability and security in mind.
*   `src` set to `www.youtube-nocookie.com` instead of `youtube.com`. Both are official Google urls, but the former doesn't do tracking via cookies, and disables API and interaction and interaction logging. Amusingly, it's the player used on `whitehouse.gov`.
*   `csp` set to `sandbox allow-scripts allow-same-origin;` for compatibility's sake, just in case. I'd love to use a more restrictive policy, but the spec doesn't allow to provide one, except if the embedded website explicitly allows it, and of course youtube doesn't.
*   `loading="lazy"` in case people don't scroll far enough to see the video, no need to make them do queries to Google for no reasons.

Don't forget to put a `title` for [accessibility's sake](https://developer.mozilla.org/en-US/docs/Web/HTML/Element/iframe#accessibility_concerns).