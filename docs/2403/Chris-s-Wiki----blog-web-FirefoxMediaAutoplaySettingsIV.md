<!--yml

类别：未分类

日期：2024-05-29 12:48:47

-->

# Chris's Wiki :: blog/web/FirefoxMediaAutoplaySettingsIV

> 来源：[https://utcc.utoronto.ca/~cks/space/blog/web/FirefoxMediaAutoplaySettingsIV](https://utcc.utoronto.ca/~cks/space/blog/web/FirefoxMediaAutoplaySettingsIV)

我一直在一个相当不错的在线音乐资源中购买数字音乐（最近被收购了，再次使人们对其长期未来感到紧张）。除了你购买的无 DRM 无损下载外，这个地方还允许你通过 Web 浏览器流式传输专辑，我的情况是 Firefox 的一个实例。最近，我注意到，我的工作 Firefox 实例可以无缝地从我正在流式传输的专辑的一首曲目过渡到下一首曲目，无论我在哪个唱片公司的子站点上，而我的家用 Firefox 则不会；当一首曲目结束时，家用 Firefox 通常会暂停而不是开始播放下一首曲目。

（我听了我购买的专辑和我还没有听过的专辑。前者通过我的集合主页播放，而后者则散布在各种不同的 URL 上，因为每个唱片公司或艺术家都会为其发行的唱片设置一个 <label>.<mumble>.com 子域名，然后每张唱片都有自己的页面。出于明显的原因，我很久以前就设置了我的家用 Firefox，允许我的集合主页自动播放音乐，这样可以无缝地从一首曲目过渡到下一首。）

两个浏览器实例通常在偏好设置 → 隐私和安全中设置禁止自动播放（参见[允许或阻止 Firefox 中的媒体自动播放](https://support.mozilla.org/en-US/kb/block-autoplay)），检查每个站点的设置显示，我的工作 Firefox 实际上没有授予任何站点自动播放权限，而我的家用 Firefox 则有一个特定供应商的各种子域名列表，允许其自动播放。在 spelunking 我的 about:config 后，我确定这是一个 [`media.autoplay.blocking_policy`](/~cks/space/blog/web/FirefoxMediaAutoplaySettingsIII) 的差异，工作 Firefox 将其设置为默认值 '0'，而我的家用 Firefox 则有一个 [长期设置为 '2'](/~cks/space/blog/web/FirefoxMediaAutoplaySettingsIII)。

正如[Mozilla关于阻止媒体自动播放的wiki页面](https://wiki.mozilla.org/Media/block-autoplay)中所讨论的，默认设置允许您与该选项卡进行交互后自动播放，而我的设置为'2'要求您始终点击才能（重新）开始音频或视频播放（除非该网站已被授予自动播放权限）。从历史上看，我将其设置为'2'是为了阻止Youtube在我的第一个视频播放结束后自动播放第二个视频。实际上，由于Youtube自己的视频播放器中的“禁用自动播放”设置，这种用法在实践中已经被功能上废弃了（尽管如果我忘记在此Firefox会话中打开该设置或Youtube在播放列表中并忽略该设置时，它仍然可以防止自动播放）。

（对于Youtube和这个数字音乐源，设置为'1'，即[短暂用户手势激活](https://wiki.mozilla.org/Media/block-autoplay#Transient_user_gesture_activation)，对我来说在功能上等同于'2'，因为视频或音轨完成播放通常会超过五秒，这意味着在网站希望进入下一个内容时权限已经过期。）

由于我更频繁地查看多轨专辑而不是观看Youtube视频（在这个Firefox上），而且如今的Youtube确实有可靠的“禁用自动播放”设置，因此我选择将`media.autoplay.blocking_policy`设置为'0'，在我用于这些内容的工作Firefox实例中，我已经将其更改为'0'，并且我在家里的Firefox中也做了同样的更改。如果我为这个音乐源设置了[自定义配置文件](/~cks/space/blog/web/FirefoxExtraProfilesHack)，我可以避免这种情况，但我还没有完全达到那个程度。

（我确实希望Firefox允许，有效地，作为每个站点自动播放权限的一部分，这样的站点设置，但我也理解为什么他们不这样做；我相信偏好和每个站点设置的用户界面复杂性会是一个值得一看的东西。）

（如果我当时想去查看[我之前关于此事的笔记](/~cks/space/blog/web/FirefoxMediaAutoplaySettingsIII)，我可能会立即找到`media.autoplay.blocking_policy`，但是我没有想到去这里查看，尽管我知道多年来我已经对Firefox媒体自动播放进行了很多调整。我过去的自己在这里写下的东西并不保证我的未来（现在）的自己会记得它们的存在。）

PS：实际上，我在自动转到我正在查看的专辑的下一首曲目方面来回移动，因为当前的“一首歌结束后停止”行为确实避免了我不经意地听完整个专辑。如果我发现自己无意中听太多理论上我只是在查看的专辑，我会把设置改回来。
