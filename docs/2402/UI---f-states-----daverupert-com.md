<!--yml

category: 未分类

date: 2024-05-27 14:54:50

-->

# UI = f(statesⁿ) - daverupert.com

> 来源：[https://daverupert.com/2024/02/ui-states/](https://daverupert.com/2024/02/ui-states/) - 来源：[https://daverupert.com/2024/02/ui-states/](https://daverupert.com/2024/02/ui-states/)

“UI is a function of state” is a pretty popular saying in the front-end world. In context (*pun intended*), that’s typically referring to application or component state. I thought I’d pull that thread a little further and explore all the states that can effect the UI layer… - “UI 是状态的函数”是前端世界中非常流行的说法。在这种情况下（双关语），通常指的是应用程序或组件状态。我想进一步探讨所有可能影响 UI 层的状态…

## First-party application states - 第一方应用程序状态

Every application whether it’s a to-do list or a shopping cart or some radically complex app will have some state. State isn’t uniform and typically exists at a variety of different levels. We’ll start at the top and drill down… - 每个应用程序，无论是待办事项列表、购物车还是某些极其复杂的应用程序，都会有一些状态。状态不是统一的，通常存在于各种不同的层次上。我们将从顶层开始逐层深入…

### Global state - 全局状态

Data stores and feature gating that typically happens at the application level. - 通常在应用程序级别发生的数据存储和功能门控。

+   Stores - 存储数据的不同位置

    +   Application store - Redux, Vuex, Mobx, Signals

    +   Browser store - localStorage, sessionStorage, cookies, IndexedDB

+   Data - 不同类型的全局数据

    +   Access control data - 认证令牌、付费/未付费、地理位置、年龄、验证、会员等。

    +   User data - 姓名、图标等

    +   Collections - 例如帖子列表

    +   Session data - 会话数据

    +   … etc - … 等等

### Page/Component state - 页面/组件状态

Vince Speelman’s wonderful [Nine States of Design](https://medium.com/swlh/the-nine-states-of-design-5bfe9b3d6d85) do a great job summing up all the states that a page or component might exist in. - Vince Speelman 的精彩文章 [设计的九个状态](https://medium.com/swlh/the-nine-states-of-design-5bfe9b3d6d85) 很好地总结了页面或组件可能存在的所有状态。

+   Nothing - 一个空元素

+   Loading - 正在进行`fetch`操作

+   None - 没有返回项目

+   One - 返回单个项目

+   Some - 一些项目返回

+   Too Many - 太多项目，需要分页（或类似）

+   Incorrect - 发生了错误

+   Correct - 发生了成功事件

+   Done - 操作完成

Vince’s list is perfect to me and keeps being relevant after all these years, I would add two items. - 对我来说，文斯的清单非常完美，并在这些年后仍然保持相关性，我会再添加两项。

+   Custom states - 与您的应用程序相关的任何定制状态

+   Realtime multi-player event mesages - 想象一下聊天应用或实时股票行情中不断更新的状态。存储在组件级或全局状态中。

+   Scroll-position - 页面和组件经常需要知道它们是否在视口内滚动。

In my experience both the page and each component will contain some mixture of these states as well as being reactive to global state changes. - 根据我的经验，页面和每个组件都将包含这些状态的某种混合，并对全局状态变化做出反应。

### Element state - 元素状态

Individual elements can (and will) have their own states. At this layer, features of HTML, CSS, and ARIA start to reveal themselves. - 个别元素可以（也将）有其自己的状态。在这一层面上，HTML、CSS 和 ARIA 的特性开始显现出来。

+   [Cursor](https://developer.mozilla.org/en-US/docs/Web/CSS/cursor) state - [鼠标指针](https://developer.mozilla.org/zh-CN/docs/Web/CSS/cursor)状态

    +   `default, pointer, wait, text, move, grab, crosshair, zoom-in, zoom-out`, … etc - `default, pointer, wait, text, move, grab, crosshair, zoom-in, zoom-out`，等等

    +   Custom cursors - 自定义鼠标指针

+   [IntersectionObserverEntry](https://developer.mozilla.org/en-US/docs/Web/API/IntersectionObserverEntry/isIntersecting) - [IntersectionObserverEntry](https://developer.mozilla.org/zh-CN/docs/Web/API/IntersectionObserverEntry/isIntersecting)

    +   isIntersecting = true, false

+   Stacking context - 堆叠上下文

+   Attribute states - 反映在 HTML 中的状态

    +   Visibility = `hidden, visible` - 可见性 = `hidden, visible`

    +   Language = `dir, lang` - 语言 = `dir, lang`

    +   Functionaltiy = `contenteditable, draggable, invoketarget` - 功能 = `contenteditable, draggable, invoketarget`

    +   Display = `inert, open, popover` - 显示 = `inert, open, popover`

    +   Loading = `lazy, eager` - 加载 = `lazy, eager`

+   [伪类](https://developer.mozilla.org/en-US/docs/Web/CSS/Pseudo-classes)状态 - 在CSS中反映的状态

    +   动作 = `:hover, :active, :focus, :focus-visible, :focus-within`

    +   输入 = `:autofill, :checked, :disabled, :valid, :invalid, :user-valid, :user-invalid, :required`, … 等等

    +   显示 = `:fullscreen, :modal, :picture-in-picture`

    +   语言 = `:dir(), :lang()`

    +   地点 = `:link, :visited, :target`, …等

    +   资源 = `:playing, :paused`

    +   [CustomStateSet](https://developer.mozilla.org/en-US/docs/Web/API/CustomStateSet) = Web组件的自定义状态

    +   …等

+   [ARIA状态](https://developer.mozilla.org/en-US/docs/web/Accessibility/ARIA/Attributes) - 反映在ARIA中的用户界面状态

    +   `aria-current`

    +   `aria-expanded`

    +   `aria-pressed`

    +   `aria-hidden`

    +   …等

## 第二方用户（或设备）状态

应用程序的最终渲染方式受用户及其设备、外围设备和浏览器的影响很大。这是有意设计的，[内置在Web基础设施中](https://www.w3.org/TR/html-design-principles/#priority-of-constituencies)。

### 语言和本地化

惊喜！并非所有用户都居住在美国西部。

+   文字方向 = `ltr, rtl`

+   书写方式 = `horizontal-tb, vertical-lr, vertical-rl`

+   到服务器/CDN的距离（延迟）

+   自动翻译

+   词语较长（例如德语）

+   词语较短（例如中文）

### 设备约束

用户设备具有多样性和自定义性，可能是渲染到屏幕的最大未知瓶颈。

+   网络连接 = 光纤，电缆，Wi-Fi，5G，4G，3G，"[lie-fi](https://web.dev/articles/performance-poor-connectivity#lie-fi)"

+   视口 = `height, width, initial-scale, horizontal-viewport-segments, vertical-viewport-segments, viewport-segment-width, viewport-segment-height`, …等

+   环境常量 = `safe-area-inset-*`, `titlebar-area-*`, `keyboard-inset-*`（例如，[iPhone刘海](https://css-tricks.com/the-notch-and-css/)，圆角，[已安装的应用](https://alistapart.com/article/breaking-out-of-the-box/)）

+   像素密度 = 1x, 2x（Retina），3x，…等

+   低功耗模式

+   屏幕亮度

+   CPU速度

+   GPU/dGPU

+   L1/L2/L3缓存

+   CPU/GPU/内存竞争（例如，其他应用程序打开）

+   色域支持 = Rec2020, P3, sRGB, …等

+   键盘 = 内嵌，外置，[T9](https://en.wikipedia.org/wiki/T9_(predictive_text))，虚拟屏幕键盘，触控栏，…等

+   XR支持 = `inline, immersive-vr, immersive-ar`

### 模态

用户在如何与其设备交互上并不统一，并可能同时使用一种或多种输入和输出方式。

+   输入设备

    +   鼠标 = 单键，双键，鼠标滚轮，轨迹球，触控板，高/低 DPI

    +   键盘 = 100%，60%，10键，QWERTY，Colemak，人体工学，分体式，机械键盘，…等

    +   触摸/点击 = 粗糙指针，无悬停

    +   笔 = 精细指针，悬停，压力灵敏度

    +   手势 = 捏合缩放，二/三/四指滑动

    +   运动 = 加速度，摇一摇取消操作，碰撞，… 等等

    +   方向 = 横向，纵向，α/β/γ（360º/180º/90º，分别）旋转

    +   语音识别 = Dragon NaturallySpeaking，语音助手

    +   开关 = 按钮，啜饮，吹气

    +   眼球追踪

    +   游戏手柄

    +   XR = 3 DOF，6 DOF

+   输出

    +   屏幕

    +   文字转语音

    +   屏幕阅读器

    +   盲文

    +   屏幕放大器

    +   震动

    +   RSS?

### 浏览器状态

最后，用户的浏览器选择和首选插件决定了他们如何体验（或者希望体验）你的 UI，你的应用程序可以响应其中一些偏好。

+   用户偏好

    +   prefers-color-scheme = 浅色，深色，强制颜色

    +   prefers-reduced-motion = 减少，无偏好

    +   prefers-reduced-transparency = 减少，无偏好

    +   用户缩放 = 100% 到 400%

    +   text size = small to x-large

+   特性和功能

    +   浏览器版本 = 最新版本，前2个版本，旧版本

    +   特性检测 = `@support` 或 polyfills

    +   色域支持 = Rec2020，P3，sRGB，...等

    +   浏览器缓存命中

    +   服务工作者命中

    +   显示模式 = 全屏 | 独立 | 最小 UI | 浏览器

    +   `beforeinstallprompt`

    +   打印模式

    +   读者模式

    +   禁用 JavaScript = 是的，我真的认识那些这样做的人。

    +   睡眠选项卡

+   权限

    +   摄像头 = true, false

    +   麦克风 = true, false

    +   地理位置 = 允许，不允许，仅在使用应用时

    +   通知 = true, false

    +   文件访问 = true, false

+   插件

    +   广告拦截程序 - UBlock，Safari ITP，Ghostery，...等

    +   自定义插件/增强功能

### 用户状态

到目前为止，我们谈论的是技术，现在让我们考虑交易的另一端的真正人类。用户的身体或心理状态影响他们享受你的体验的认知或文字带宽。

## 第三方服务状态

第三方对 UI 用户体验有着不成比例的影响。

### 可用性/状态

其他服务器的状态/可用性/正常运行时间是 UI 失败的最大面源。

+   服务器硬件状态 = 在线，离线，部分可用

+   数据库状态 = 在线，离线，事务锁定

+   API 状态 = 在线，离线，部分可用

+   Web 字体服务 = 在线，离线，部分可用

+   DNS 状态 = 可能需要最多 72 小时解析

+   包依赖 = 正常工作，损坏，恶意注入，抗议软件，...等

+   资产交付和缓存状态

### 脚本注入

在 UI 上，第三方脚本注入是性能下降的主要贡献者。有些服务甚至接管了应用程序的用户体验。这通常超出你的控制范围。

+   分析/跟踪服务

+   用户会话录制服务

+   A/B 测试注入

+   指导叠加层

+   辅助功能叠加层

+   第三方认证（OAuth）服务

+   Captcha/验证服务

除非有一个未经检查的服务拖垮了应用程序，否则你不会真正体验过生活。

## UI 不仅仅是状态...

我肯定忘记了整个状态的类别。我甚至还没有涉及数百种 CSS 属性和数千个值及其潜在冲突。我没有讨论样式化表单控件的复杂性，或者使用必需的 ARIA 属性组合构建下拉菜单。我没有触及放置图像到页面上所需的[决策矩阵](https://cloudfour.com/thinks/responsive-images-101-definitions/)。也没有讨论在`head`中[正确设置所有必要的标签](https://github.com/joshbuchea/HEAD)[的顺序](https://rviscomi.github.io/capo.js/)。也没有讨论[微格式](https://microformats.org/)用于 SEO 或者设置 UI 发送分析数据所需的所有代码。

最后，雇佣擅长用户界面的人员。

* * *

*Edit 2/16/24*
