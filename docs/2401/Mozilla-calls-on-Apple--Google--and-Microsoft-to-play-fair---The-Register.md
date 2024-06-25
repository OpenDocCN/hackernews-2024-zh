<!--yml

类别：未分类

日期：2024年5月27日15:13:38

-->

# Mozilla呼吁苹果、谷歌和微软公平竞争 • The Register

> 来源：[https://www.theregister.com/2024/01/25/mozilla_apple_google_browser_wars/](https://www.theregister.com/2024/01/25/mozilla_apple_google_browser_wars/)

Mozilla决定更加积极地表达苹果、谷歌和微软设定的技术要求，这些要求阻碍了其Firefox网络浏览器的发展，从而损害了竞争。

这些竞争壁垒中的许多已经讨论多年了——在技术界和与监管机构的闭门会议中。在这些年里，Firefox在全球浏览器市场份额持续下滑，如今，[仅略超过三个百分点](https://gs.statcounter.com/browser-market-share#monthly-200901-202306)的浏览器用户偏爱Firefox——这是平台规则和无情营销的影响。

随着对调整浏览器竞争环境的[重新游说](https://www.theregister.com/2022/02/28/apple_apps_challenge/)的兴起，以及[欧洲数字市场法](https://www.theregister.com/2022/04/26/apple_ios_browser/)（DMA）的即将到来的三月六日的合规截止日期——Mozilla决定详细说明它认为这些问题及其造成的伤害。

"多年来，Mozilla一直与平台供应商展开对话，以解决这些问题，" 开发团队 [宣布](https://blog.mozilla.org/netpolicy/2024/01/19/platform-tilt/)。"随着公众对此事的持续关注和监管环境的不断发展，我们认为是时候使用我们开发新兴技术标准的相同透明流程和工具来发布这些关切了。"

Mozilla的关切记录在一个[摘要仪表板](https://mozilla.github.io/platform-tilt/)和一个GitHub的[问题跟踪器](https://github.com/mozilla/platform-tilt/issues)中，这些记录列举并解释了竞争对手浏览器制造的技术障碍，这些障碍使Firefox处于竞争劣势。

在仪表板的顶部——按字母顺序和巧合地按实际重要性——是[Apple App Store审核指南的2.5.6规则](https://developer.apple.com/app-store/review/guidelines/#software-requirements)，该规则要求iOS上的所有浏览器使用苹果的WebKit渲染引擎。

这一要求的后果是，所有iOS的浏览器本质上都是苹果的WebKit内核——这使得开发者很难区分他们的产品。

在苹果的iOS上，缺乏浏览器竞争和差异化一直是Mozilla的痛点，也是谷歌的痛点。谷歌推动了Chromium浏览器引擎的开发，该引擎驱动其自己的Chrome浏览器和微软的Edge浏览器。至少在除了苹果以外的所有平台上都是这样。

苹果针对 iOS 的 WebKit 规则已经受到了 [英国](https://www.theregister.com/2023/12/02/uk_cma_wins_appeal_apple/) 和 [欧洲](https://www.theregister.com/2023/11/02/apple_safari_browser/) 监管机构的调查，可能不会再持续太久。

Mozilla 提到的其他技术障碍虽然对竞争有害，但受到的关注较少。

例如，iOS [不允许](https://github.com/mozilla/platform-tilt/issues/2) 第三方应用程序使用 Safari 的多进程架构，除非它们通过 WebKit 进行使用。此外，iOS [不允许](https://github.com/mozilla/platform-tilt/issues/3) 创建既可写又可执行的内存区域——但它为 WebKit 和其他苹果应用程序做了一个例外。

此外还有苹果的 iOS [辅助功能 API](https://github.com/mozilla/platform-tilt/issues/4)，Mozilla 表示，这些 API 部分未记录，以致其他浏览器制造商难以支持辅助功能网络标准。

列表还包括：[消息集成限制](https://github.com/mozilla/platform-tilt/issues/5)，[浏览器数据导入限制](https://github.com/mozilla/platform-tilt/issues/6)，[默认浏览器设置和检查限制](https://github.com/mozilla/platform-tilt/issues/9)，[基于来源的域限制](https://github.com/mozilla/platform-tilt/issues/11)，[浏览器扩展限制](https://github.com/mozilla/platform-tilt/issues/15)，以及[测试版测试限制](https://github.com/mozilla/platform-tilt/issues/16)。

谷歌因未允许第三方 Android 浏览器[导入浏览数据](https://github.com/mozilla/platform-tilt/issues/7)，有时[忽视用户的默认浏览器选择](https://github.com/mozilla/platform-tilt/issues/8)，以及将 Android 与 Google 搜索集成，使得用户使用 Firefox 进行搜索时[搜索结果变差](https://github.com/mozilla/platform-tilt/issues/12)而备受指责。

微软也未能幸免。Windows 业务被指责未能允许第三方浏览器[设置为默认浏览器](https://github.com/mozilla/platform-tilt/issues/10)，允许 Edge 更容易地[将自己设置为默认浏览器](https://github.com/mozilla/platform-tilt/issues/13)，以及[打开 Edge 中的某些功能](https://github.com/mozilla/platform-tilt/issues/14)，而不考虑用户选择的默认浏览器。

互联网社区成员在[问题帖子](https://github.com/mozilla/platform-tilt/issues)中进一步确定了平台差异。

"人们应该有选择权，而选择权需要存在可行的替代方案，" Mozilla 写道。"替代方案和竞争对每个人都有好处，但只有在竞争环境公平的情况下才能蓬勃发展。现在不是，但如果平台供应商愿意的话，也不难解决。"

苹果继续[努力减少](https://open-web-advocacy.org/blog/apple-filing-eu-appstores/)在 DMA 下的义务，对于评论请求没有做出回应。谷歌也没有。

微软回应称“没有任何可分享的内容。” ®
