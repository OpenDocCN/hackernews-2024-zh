<!--yml

category: 未分类

date: 2024-05-27 14:32:05

-->

# Microsoft Edge悄悄吞并Chrome数据

> 来源：[https://reclaimthenet.org/microsoft-edge-sucks-up-chrome-data-without-permission](https://reclaimthenet.org/microsoft-edge-sucks-up-chrome-data-without-permission)

大多数使用微软产品的人 - 包括Windows和Edge浏览器在内 - 现在应该已经习惯了接受这样一个事实：他们运行的软件和他们认为自己拥有的数据实际上并不是他们控制的东西。

或许正是因为这个“功能”如此令人震惊，即使在禁用了Microsoft浏览器的导入工具的情况下，Edge仍然自动导入Google Chrome中打开的标签，这个问题已经被人们知晓数月之久，但没有得到解决（假设它是一个bug的话）。

但看起来这像是公司思维中的一个“bug”，其中之一 - 并不完全是软件问题。毕竟，这绝对是一项功能。

Edge和Chrome都基于同一Chromium引擎，这应该使得“操作”更容易；而在涉及侵入性和对终端用户行为的争议性做法上，微软和谷歌可谓是一丘之貉。

而当他们感觉有必要时，他们彼此之间也并不特别友好。一些报道表明，这种“偷窃标签”的特性实际上只是微软试图从Chrome那里夺取用户并强迫他们毫无选择地转向Edge的一种方式。

一位Verge [记者](https://www.theverge.com/24054329/microsoft-edge-automatic-chrome-import-data-feature) 和一位Windows及Chrome（偶尔还用Edge）用户描述了这一经历，核心问题是，在Windows更新和重启后，他们默认的Chrome浏览器中打开的标签被导入到了Edge中。

毫不奇怪，用户并没有被要求同意这些操作中的任何一个。在这里，Windows/Edge的使用体验可以用一句话来概括：“起初我甚至没有意识到我在使用Edge，我很困惑为什么所有（从Chrome导入的）标签突然间都退出登录了。”

而确实，在edge://settings/profiles/importBrowsingData选项中，设置为禁止自动访问“最近的浏览数据”，即Edge通过吸收Chrome中打开的标签的类似“博格”的同化来导入数据。

但抵抗肯定不是徒劳的：使用开源操作系统和浏览器，最深层次的权限访问和决策完全掌握在你手中。

与此同时，如今重新安装Windows的用户将在这个阶段了解到浏览器之争背后的原因。

一则安装提示说：

“在你的确认下，Microsoft Edge将定期从Windows设备上其他浏览器中获取数据。这些数据包括你的收藏夹、浏览历史记录、Cookie、自动填充数据、扩展、设置以及其他浏览数据。”

就像用户所报告的那样，无论你是否确认过，它们都在背后运作。毕竟这是微软，这家科技恐龙在回应相关媒体查询时并没有感到紧迫性。
