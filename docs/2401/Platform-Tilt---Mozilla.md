<!--yml
category: 未分类
date: 2024-05-27 14:58:29
-->

# Platform Tilt - Mozilla

> 来源：[https://mozilla.github.io/platform-tilt/](https://mozilla.github.io/platform-tilt/)

There are at least three prominent Windows features that open URLs in Microsoft Edge and not in the current default browser. The user’s default browser choice should be respected when web pages are opened by built-in operating system features.

The first is Windows Search, also known as Start Menu Search, and formerly known as Cortana. The UI for this feature is represented by a taskbar search box or search button (depending on user settings), and a search suggestions / results UI that appears when activated and updates as the user types. The suggestions and results UI also appears if the user starts typing when the start menu is open, and by the WIN+S hotkey. All links from this UI, whether they initiate web searches or link directly to articles or results, open in Microsoft Edge regardless of the user’s default browser.

The second is the new Windows Copilot, currently only available on Windows 11, which appears as a docked window on the right side of the screen. If Copilot produces links in its responses, or offers other links within its rendering area, these links open in Microsoft Edge regardless of the user’s default browser.

The third are Windows “widgets” which are called “news and interests” on Windows 10, a UI surface area which can be activated by a taskbar button. These show information like news, weather, stocks, and sports scores. On Windows 11 new widgets can be added from 3rd parties. Regardless, all links to a web page from widgets will open in Microsoft Edge regardless of the user’s default browser.