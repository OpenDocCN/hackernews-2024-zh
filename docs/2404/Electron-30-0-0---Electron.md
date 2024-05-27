<!--yml
category: 未分类
date: 2024-05-27 13:28:49
-->

# Electron 30.0.0 | Electron

> 来源：[https://www.electronjs.org/blog/electron-30-0](https://www.electronjs.org/blog/electron-30-0)

Electron 30.0.0 has been released! It includes upgrades to Chromium `124.0.6367.49`, V8 `12.4`, and Node.js `20.11.1`.

* * *

The Electron team is excited to announce the release of Electron 30.0.0! You can install it with npm via `npm install electron@latest` or download it from our [releases website](https://releases.electronjs.org/releases/stable). Continue reading for details about this release.

If you have any feedback, please share it with us on [Twitter](https://twitter.com/electronjs) or [Mastodon](https://social.lfx.dev/@electronjs), or join our community [Discord](https://discord.com/invite/electronjs)! Bugs and feature requests can be reported in Electron's [issue tracker](https://github.com/electron/electron/issues).

## Notable Changes[​](#notable-changes "Direct link to Notable Changes")

### Highlights[​](#highlights "Direct link to Highlights")

*   ASAR Integrity fuse now supported on Windows ([#40504](https://github.com/electron/electron/pull/40504))
    *   Existing apps with ASAR Integrity enabled may not work on Windows if not configured correctly. Apps using Electron packaging tools should upgrade to `@electron/packager@18.3.1` or `@electron/forge@7.4.0`.
    *   Take a look at our [ASAR Integrity tutorial](https://www.electronjs.org/docs/latest/tutorial/asar-integrity) for more information.
*   Added [`WebContentsView`](https://www.electronjs.org/docs/latest/api/web-contents-view) and [`BaseWindow`](https://www.electronjs.org/docs/latest/api/base-window) main process modules, deprecating & replacing `BrowserView` ([#35658](https://github.com/electron/electron/pull/35658))
    *   `BrowserView` is now a shim over `WebContentsView` and the old implementation has been removed.
    *   See [our Web Embeds documentation](https://www.electronjs.org/docs/latest/tutorial/web-embeds) for a comparison of the new `WebContentsView` API to other similar APIs.
*   Implemented support for the [File System API](https://developer.mozilla.org/en-US/docs/Web/API/File_System_API) ([#41827](https://github.com/electron/electron/commit/cf1087badd437906f280373decb923733a8523e6))

### Stack Changes[​](#stack-changes "Direct link to Stack Changes")

*   Chromium `124.0.6367.49`
*   Node `20.11.1`
*   V8 `12.4`

Electron 30 upgrades Chromium from `122.0.6261.39` to `124.0.6367.49`, Node from `20.9.0` to `20.11.1`, and V8 from `12.2` to `12.4`.

### New Features[​](#new-features "Direct link to New Features")

*   Added a `transparent` webpreference to webviews. ([#40301](https://github.com/electron/electron/pull/40301))
*   Added a new instance property `navigationHistory` on webContents API with `navigationHistory.getEntryAtIndex` method, enabling applications to retrieve the URL and title of any navigation entry within the browsing history. ([#41662](https://github.com/electron/electron/pull/41662))
*   Added new `BrowserWindow.isOccluded()` method to allow apps to check occlusion status. ([#38982](https://github.com/electron/electron/pull/38982))
*   Added proxy configuring support for requests made with the `net` module from the utility process. ([#41417](https://github.com/electron/electron/pull/41417))
*   Added support for Bluetooth ports being requested by service class ID in `navigator.serial`. ([#41734](https://github.com/electron/electron/pull/41734))
*   Added support for the Node.js [`NODE_EXTRA_CA_CERTS`](https://nodejs.org/api/cli.html#node_extra_ca_certsfile) CLI flag. ([#41822](https://github.com/electron/electron/pull/41822))

### Breaking Changes[​](#breaking-changes "Direct link to Breaking Changes")

#### Behavior Changed: cross-origin iframes now use Permission Policy to access features[​](#behavior-changed-cross-origin-iframes-now-use-permission-policy-to-access-features "Direct link to Behavior Changed: cross-origin iframes now use Permission Policy to access features")

Cross-origin iframes must now specify features available to a given `iframe` via the `allow` attribute in order to access them.

See [documentation](https://developer.mozilla.org/en-US/docs/Web/HTML/Element/iframe#allow) for more information.

#### Removed: The `--disable-color-correct-rendering` command line switch[​](#removed-the---disable-color-correct-rendering-command-line-switch "Direct link to removed-the---disable-color-correct-rendering-command-line-switch")

This switch was never formally documented but its removal is being noted here regardless. Chromium itself now has better support for color spaces so this flag should not be needed.

#### Behavior Changed: `BrowserView.setAutoResize` behavior on macOS[​](#behavior-changed-browserviewsetautoresize-behavior-on-macos "Direct link to behavior-changed-browserviewsetautoresize-behavior-on-macos")

In Electron 30, BrowserView is now a wrapper around the new [WebContentsView](https://www.electronjs.org/docs/latest/api/web-contents-view) API.

Previously, the `setAutoResize` function of the `BrowserView` API was backed by [autoresizing](https://developer.apple.com/documentation/appkit/nsview/1483281-autoresizingmask?language=objc) on macOS, and by a custom algorithm on Windows and Linux. For simple use cases such as making a BrowserView fill the entire window, the behavior of these two approaches was identical. However, in more advanced cases, BrowserViews would be autoresized differently on macOS than they would be on other platforms, as the custom resizing algorithm for Windows and Linux did not perfectly match the behavior of macOS's autoresizing API. The autoresizing behavior is now standardized across all platforms.

If your app uses `BrowserView.setAutoResize` to do anything more complex than making a BrowserView fill the entire window, it's likely you already had custom logic in place to handle this difference in behavior on macOS. If so, that logic will no longer be needed in Electron 30 as autoresizing behavior is consistent.

The `inputFormType` property of the params object in the `context-menu` event from `WebContents` has been removed. Use the new `formControlType` property instead.

#### Removed: `process.getIOCounters()`[​](#removed-processgetiocounters "Direct link to removed-processgetiocounters")

Chromium has removed access to this information.

## End of Support for 27.x.y[​](#end-of-support-for-27xy "Direct link to End of Support for 27.x.y")

Electron 27.x.y has reached end-of-support as per the project's [support policy](https://www.electronjs.org/docs/latest/tutorial/electron-timelines#version-support-policy). Developers and applications are encouraged to upgrade to a newer version of Electron.

| E30 (Apr'24) | E31 (Jun'24) | E32 (Aug'24) |
| --- | --- | --- |
| 30.x.y | 31.x.y | 32.x.y |
| 29.x.y | 30.x.y | 31.x.y |
| 28.x.y | 29.x.y | 30.x.y |

## What's Next[​](#whats-next "Direct link to What's Next")

In the short term, you can expect the team to continue to focus on keeping up with the development of the major components that make up Electron, including Chromium, Node, and V8.

You can find [Electron's public timeline here](https://www.electronjs.org/docs/latest/tutorial/electron-timelines).

More information about future changes can be found on the [Planned Breaking Changes](https://github.com/electron/electron/blob/main/docs/breaking-changes.md) page.