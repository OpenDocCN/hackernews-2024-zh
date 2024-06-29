<!--yml

类别：未分类

日期：2024-05-27 13:24:04

-->

# 每个人都有 JavaScript，对吧？

> 来源：[https://www.kryogenix.org/code/browser/everyonehasjs.html](https://www.kryogenix.org/code/browser/everyonehasjs.html)

# 每个人都有 JavaScript，对吧？

你的用户请求你的 Web 应用程序

页面已经加载了吗？

“当用户下载你的 JS 时，他们都是非 JS 用户” — [Jake Archibald](https://twitter.com/jaffathecake/status/207096228339658752)

JavaScript 的 HTTP 请求成功了吗？

如果他们在火车上，网连接突然中断，导致你的 JavaScript 无法加载，那么就没有 JavaScript 了。

JavaScript 的 HTTP 请求完成了吗？

你有多少次看到移动浏览器加载页面时卡住，然后刷新后立即加载？

企业防火墙是否阻止了 JavaScript？

因为还有很多人这样做。

他们的 ISP 或移动运营商是否干扰了下载的 JavaScript？

[Sky 不小心屏蔽了 jQuery](http://www.theguardian.com/technology/2014/jan/28/sky-broadband-blocks-jquery-web-critical-plugin)，[Comcast 在你的脚本中插入广告](http://aaron-gustafson.com/notebook/2014/the-network-effect/)，如果你以前从未经历过这种情况，请去机场并使用他们的 WiFi。

用户是微型浏览器，获取细节嵌入到另一个应用程序中吗？

[微型浏览器随处可见](https://24ways.org/2019/microbrowsers-are-everywhere/)，每当有人将你的 URL 粘贴到聊天应用或社交媒体工具中并显示预览时，他们不会等待你的 2MB 客户端应用程序启动。

他们关闭了 JavaScript 吗？

人们[确实如此](https://gds.blog.gov.uk/2013/10/21/how-many-people-are-missing-out-on-javascript-enhancement/)。

他们有没有打开 Chrome 的数据节省模式？

如果他们使用的是 2G 或其他慢速连接，[Chrome 将禁用所有脚本](https://www.chromestatus.com/feature/4775088607985664)。

他们安装了会注入脚本或以你未预料的方式修改 DOM 的插件或附加组件吗？

有成千上万的浏览器扩展程序。你确定没有一个干扰你的 JS 吗？

他们的广告拦截器是否阻止了你意外依赖的脚本？

[许多人使用广告拦截器，可能会整体阻止某些域](https://twitter.com/heydonworks/status/1454826804464734209)。如果一个脚本依赖于另一个脚本，并且另一个脚本被阻止了，那么你的其余脚本可能无法启动。

CDN 是正常的吗？

CDNs 擅长保持稳定（这就是 CDN 的功能），但每个月的[一分钟停机](http://www.cdnperf.com/)仍会影响在那一分钟内浏览的用户。

他们的浏览器支持你编写的 JavaScript 吗？

检查 [Can I Use](http://caniuse.com/) 查看浏览器使用情况。

上述都正确吗？

那么是的，你的 JavaScript 可以工作。

**很可能。**

[渐进增强](http://jakearchibald.com/2013/progressive-enhancement-still-important/)。因为有时你的 JavaScript 就是不会工作。

[做好准备。](https://kryogenix.org/code/browser/why-availability/)

[俄语翻译可在《Frontender Magazine》](https://web.archive.org/web/20161215200626/http://frontender.info:80/everyone-has-js/)找到，由[Антон Немцев](http://twitter.com/silentimp)提供。
