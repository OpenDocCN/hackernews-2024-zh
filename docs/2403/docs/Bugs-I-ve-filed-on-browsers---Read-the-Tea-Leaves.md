<!--yml
category: 未分类
date: 2024-05-27 14:36:57
-->

# Bugs I’ve filed on browsers | Read the Tea Leaves

> 来源：[https://nolanlawson.com/2024/03/03/bugs-ive-filed-on-browsers/](https://nolanlawson.com/2024/03/03/bugs-ive-filed-on-browsers/)

I think filing bugs on browsers is one of the most useful things a web developer can do.

When faced with a cross-browser compatibility problem, a lot of us are conditioned to just search for some quick workaround, or to keep cycling through alternatives until something works. And this is definitely what I did earlier in my career. But I think it’s too short-sighted.

Browser dev teams are just like web dev teams – they have priorities and backlogs, and they sometimes let bugs slip through. Also, a well-written bug report with clear steps-to-repro can often lead to a quick resolution – especially if you manage to [nerd-snipe](https://xkcd.com/356/) some bored or curious engineer.

As such, I’ve filed a lot of bugs on browsers over the years. For whatever reason – stubbornness, frustration, some highfalutin sense of serving the web at large – I’ve made a habit of nagging browser vendors about whatever roadblock I’m hitting that day. And they often fix it!

So I thought it might be interesting to do an analysis of the bugs I’ve filed on the major browser engines – Chromium, Firefox, and WebKit – over my roughly 10-year web development career. I’ve excluded older and lesser-known browser engines that I never filed bugs on, and I’ve also excluded Trident/EdgeHTML, since the original “Microsoft Connect” bug tracker seems to be offline. (Also, I was literally paid to file bugs on EdgeHTML for a good 2 years, so it’s kind of unfair.)

Some notes about this data set, before people start drawing conclusions:

*   Chromium is a bit over-represented, because I tend to use Chromedriver-related tools (e.g. Puppeteer) a lot more than other browser automation tools.
*   WebKit is kind of odd in that a lot of these bugs turned out to be in proprietary Apple systems (Safari, iOS, etc.) rather than WebKit proper. (At least, this is what I assume the enigmatic `rdar://` response means. [^([1])](#footnote-1))
*   I excluded one bug from Firefox that was actually on MDN (which uses the same bug tracker).

## The data

So without further ado, here is the data set:

| Browser | Filed | Open | Fixed | Invalid | Fixed% |
| --- | --- | --- | --- | --- | --- |
| Chromium | 27 | 4 | 14 | 9 | 77.78% |
| Firefox | 18 | 3 | 8 | 7 | 72.73% |
| WebKit | 25 | 6 | 12 | 7 | 66.67% |
| **Total** | **70** | **13** | **34** | **23** | **72.34%** |

**Notes:** For “Invalid,” I’m being generous and including “duplicate,” “obsolete,” “wontfix,” etc. For “Fixed%,” I’m counting only the number fixed as a proportion of valid bugs.

Some things that jump out at me from the data set:

*   The “Fixed%” is pretty similar for all three browsers, although WebKit’s is a bit lower. When I look at the unfixed WebKit bugs, 2 are related to iOS rather than WebKit, one is in WebSQL (RIP), and the remaining 3 are honestly pretty minor. So I can’t really blame the WebKit team. (And [one of those minor issues](https://bugs.webkit.org/show_bug.cgi?id=263663) wound up in [Interop 2024](https://wpt.fyi/results/?label=master&label=experimental&aligned&view=interop&q=label%3Ainterop-2024-accessibility), so it may get fixed soon.)
*   For the 3 open Firefox issues, 2 of them are quite old and have terrible steps-to-repro (mea culpa), and the remaining one is [a minor CSS issue related to shadow DOM](https://bugzilla.mozilla.org/show_bug.cgi?id=1739682).
*   For the 4 open Chromium issues, [one of them](https://issues.chromium.org/issues/40890306) is actually obsolete (I pinged the thread), 2 are quite minor, and [the remaining one](https://issues.chromium.org/issues/40656738) is partially fixed (it works when [BFCache](https://web.dev/articles/bfcache) is enabled).
*   I was surprised that the total number of bugs filed on Firefox wasn’t even lower. My hazy memory was that I had barely filed any bugs on Firefox, and when I did, it usually turned out that they were following the spec but the other browsers weren’t. (I learned to really triple-check my work before filing bugs on them!)
*   6 of the bugs I filed on WebKit were for IndexedDB, which definitely matches my memory of hounding them with bug reports for IDB. (In comparison, I filed 3 IDB bugs on Chromium and 0 on Firefox.)
*   As expected, 5 issues I filed on Chromium were due to ChromeDriver, DevTools, etc.

If you’d like to peruse the raw data, it can be found below:

<details><summary>Chromium data</summary>

| Status | ID | Title | Date |
| --- | --- | --- | --- |
| Fixed | [41495645](https://crbug.com/41495645) | Chromium leaks Document/Window/etc. when navigating in multi-page site using CDP-based heapsnapshots | Feb 22, 2024 01:02AM |
| New | [40890306](https://crbug.com/40890306) | Reflected ARIA properties should not treat setting undefined the same as null | Mar 2, 2024 05:50PM |
| New | [40885158](https://crbug.com/40885158) | Setting outerHTML on child of DocumentFragment throws error | Jan 8, 2024 10:52PM |
| Fixed | [40872282](https://crbug.com/40872282) | aria-label on a should be ignored for accessible name calculation | Jan 8, 2024 11:24PM |
| Duplicate | [40229331](https://crbug.com/40229331) | Style calculation takes much longer for multiple s vs one big | Jun 20, 2023 09:14AM |
| Fixed | [40846966](https://crbug.com/40846966) | customElements.whenDefined() resolves with undefined instead of constructor | Jan 8, 2024 10:13PM |
| Fixed | [40827056](https://crbug.com/40827056) | ariaInvalid property not reflected to/from aria-invalid attribute | Jan 8, 2024 08:33PM |
| Obsolete | [40767620](https://crbug.com/40767620) | Heap snapshot includes objects referenced by DevTools console | Jan 8, 2024 11:53PM |
| New | [40766136](https://crbug.com/40766136) | Restoring selection ranges causes text input to ignore keypresses | Jan 8, 2024 07:20PM |
| Fixed | [40759641](https://crbug.com/40759641) | Poor style calculation performance for attribute selectors compared to class selectors | May 5, 2021 01:40PM |
| Obsolete | [40149430](https://crbug.com/40149430) | performance.measureMemory() disallowed in headless mode | Feb 27, 2023 12:26AM |
| Obsolete | [40704787](https://crbug.com/40704787) | Add option to disable WeakMap keys and circular references in Retainers graph | Jan 8, 2024 05:52PM |
| Fixed | [40693859](https://crbug.com/40693859) | Chrome crashes due to WASM file when DevTools are recording trace | Jun 24, 2020 07:20AM |
| Fixed | [40677812](https://crbug.com/40677812) | npm install chrome-devtools-frontend fails due to preinstall script | Jan 8, 2024 05:17PM |
| New | [40656738](https://crbug.com/40656738) | Navigating back does not restore focus to clicked element | Jan 8, 2024 03:26PM |
| Obsolete | [41477958](https://crbug.com/41477958) | Compositor animations recalc style on main thread every frame with empty requestAnimationFrame | Aug 29, 2019 04:06AM |
| Duplicate | [41476815](https://crbug.com/41476815) | OffscreenCanvas convertToBlob() is >300ms slower than | Feb 18, 2020 11:51PM |
| Fixed | [41475186](https://crbug.com/41475186) | Insertion and removal of overflow:scroll element causes large style calculation regression | Aug 26, 2019 01:34AM |
| Obsolete | [41354172](https://crbug.com/41354172) | IntersectionObserver uses root’s padding box rather than border box | Nov 12, 2018 09:43AM |
| Obsolete | [41329253](https://crbug.com/41329253) | word-wrap:break-word with odd Unicode characters causes long layout | Jul 11, 2017 01:50PM |
| Fixed | [41327511](https://crbug.com/41327511) | IntersectionObserver boundingClientRect has inaccurate width/height | Jun 1, 2019 08:28PM |
| Fixed | [41267419](https://crbug.com/41267419) | Chrome 52 sends a CORS preflight request with an empty Access-Control-Request-Headers when all author headers are CORS-safelisted | Mar 18, 2017 02:27PM |
| Fixed | [41204713](https://crbug.com/41204713) | IndexedDB blocks DOM rendering | Jan 24, 2018 04:03PM |
| Fixed | [41189720](https://crbug.com/41189720) | chrome://inspect/#devices flashing “Pending authorization” for Android device | Oct 5, 2015 03:59AM |
| Obsolete | [41154786](https://crbug.com/41154786) | Chrome for iOS: openDatabase causes DOM Exception 11 or 18 | Feb 9, 2015 08:30AM |
| Fixed | [41151574](https://crbug.com/41151574) | Empty IndexedDB blob causes 404 when fetched with ajax | Mar 16, 2015 11:37AM |
| Fixed | [40400696](https://crbug.com/40400696) | Blob stored in IndexedDB causes null result from FileReader | Feb 9, 2015 10:34AM |</details> <details><summary>Firefox data</summary>

| ID | Summary | Resolution | Updated |
| --- | --- | --- | --- |
| [1704551](https://bugzilla.mozilla.org/show_bug.cgi?id=1704551) | Poor style calculation performance for attribute selectors compared to class selectors | FIXE | 2021-09-02 |
| [1861201](https://bugzilla.mozilla.org/show_bug.cgi?id=1861201) | Support ariaBrailleLabel and ariaBrailleRoleDescription reflection | FIXE | 2024-02-20 |
| [1762999](https://bugzilla.mozilla.org/show_bug.cgi?id=1762999) | Intervening divs with ids reports incorrect listbox options count to NVDA | FIXE | 2023-10-10 |
| [1739154](https://bugzilla.mozilla.org/show_bug.cgi?id=1739154) | delegatesFocus changes focused inner element when host is focused | FIXE | 2022-02-08 |
| [1707116](https://bugzilla.mozilla.org/show_bug.cgi?id=1707116) | Replacing shadow DOM style results in inconsistent computed style | FIXE | 2021-05-10 |
| [1853209](https://bugzilla.mozilla.org/show_bug.cgi?id=1853209) | ARIA reflection should treat setting null/undefined as removing the attribute | FIXE | 2023-10-20 |
| [1208840](https://bugzilla.mozilla.org/show_bug.cgi?id=1208840) | IndexedDB blocks DOM rendering | — | 2022-10-11 |
| [1531511](https://bugzilla.mozilla.org/show_bug.cgi?id=1531511) | Service Worker fetch requests during ‘install’ phase block fetch requests from main thread | — | 2022-10-11 |
| [1739682](https://bugzilla.mozilla.org/show_bug.cgi?id=1739682) | Bare ::part(foo) CSS selector selects parts inside shadow roots | — | 2024-02-20 |
| [1331135](https://bugzilla.mozilla.org/show_bug.cgi?id=1331135) | Performance User Timing entry buffer restricted to 150 | DUPL | 2019-03-13 |
| [1699154](https://bugzilla.mozilla.org/show_bug.cgi?id=1699154) | :focus-visible – JS-based focus() on back nav treated as keyboard input | FIXE | 2021-03-19 |
| [1449770](https://bugzilla.mozilla.org/show_bug.cgi?id=1449770) | position:sticky inside of position:fixed does’t async-scroll in Firefox for Android (and asserts in ActiveScrolledRoot::PickDescendant() in debug build) | WORK | 2023-02-23 |
| [1287221](https://bugzilla.mozilla.org/show_bug.cgi?id=1287221) | WebDriver:Navigate results in slower performance.timing metrics | DUPL | 2023-02-09 |
| [1536717](https://bugzilla.mozilla.org/show_bug.cgi?id=1536717) | document.scrollingElement.scrollTop is incorrect | DUPL | 2022-01-10 |
| [1253387](https://bugzilla.mozilla.org/show_bug.cgi?id=1253387) | Safari does not support IndexedDB in a worker | FIXE | 2016-03-17 |
| [1062368](https://bugzilla.mozilla.org/show_bug.cgi?id=1062368) | Ajax requests for blob URLs return 0 as .status even if the load succeeds | DUPL | 2014-09-04 |
| [1081668](https://bugzilla.mozilla.org/show_bug.cgi?id=1081668) | Blob URL returns xhr.status of 0 | DUPL | 2015-02-25 |
| [1711057](https://bugzilla.mozilla.org/show_bug.cgi?id=1711057) | :focus-visible does not match for programmatic keyboard focus after mouse click | FIXE | 2021-06-09 |
| [1471297](https://bugzilla.mozilla.org/show_bug.cgi?id=1471297) | fetch() and importScripts() do not share HTTP cache | WORK | 2021-03-17 |</details> <details><summary>WebKit data</summary>

| ID | Resolution | Summary | Changed |
| --- | --- | --- | --- |
| [225723](https://bugs.webkit.org/show_bug.cgi?id=225723) | — | Restoring selection ranges causes text input to ignore keypresses | 2023-02-26 |
| [241704](https://bugs.webkit.org/show_bug.cgi?id=241704) | — | Preparser does not download stylesheets before running inline scripts | 2022-06-23 |
| [263663](https://bugs.webkit.org/show_bug.cgi?id=263663) | — | Support ariaBrailleLabel and ariaBrailleRoleDescription reflection | 2023-11-01 |
| [260716](https://bugs.webkit.org/show_bug.cgi?id=260716) | FIXE | adoptedStyleSheets (ObservableArray) has non-writable length | 2023-09-03 |
| [232261](https://bugs.webkit.org/show_bug.cgi?id=232261) | FIXE | :host::part(foo) selector does not select elements inside shadow roots | 2021-11-04 |
| [249420](https://bugs.webkit.org/show_bug.cgi?id=249420) | DUPL | :host(.foo, .bar) should be an invalid selector | 2023-08-07 |
| [249737](https://bugs.webkit.org/show_bug.cgi?id=249737) | FIXE | Setting outerHTML on child of DocumentFragment throws error | 2023-07-15 |
| [251383](https://bugs.webkit.org/show_bug.cgi?id=251383) | INVA | Reflected ARIA properties should not treat setting undefined the same as null | 2023-10-25 |
| [137637](https://bugs.webkit.org/show_bug.cgi?id=137637) | — | Null character causes early string termination in Web SQL | 2015-04-25 |
| [202655](https://bugs.webkit.org/show_bug.cgi?id=202655) | — | iOS Safari: timestamps can be identical for consecutive rAF callbacks | 2019-10-10 |
| [249943](https://bugs.webkit.org/show_bug.cgi?id=249943) | — | Emoji character is horizontally misaligned when using COLR font | 2023-01-04 |
| [136888](https://bugs.webkit.org/show_bug.cgi?id=136888) | FIXE | IndexedDB onupgradeneeded event has incorrect value for oldVersion | 2019-07-04 |
| [137034](https://bugs.webkit.org/show_bug.cgi?id=137034) | FIXE | Completely remove all IDB properties/constructors when it is disabled at runtime | 2015-06-08 |
| [149953](https://bugs.webkit.org/show_bug.cgi?id=149953) | FIXE | Modern IDB: WebWorker support | 2016-05-11 |
| [151614](https://bugs.webkit.org/show_bug.cgi?id=151614) | FIXE | location.origin is undefined in a web worker | 2015-11-30 |
| [156048](https://bugs.webkit.org/show_bug.cgi?id=156048) | FIXE | We sometimes fail to remove outdated entry from the disk cache after revalidation and when the resource is no longer cacheable | 2016-04-05 |
| [137647](https://bugs.webkit.org/show_bug.cgi?id=137647) | FIXE | Fetching Blob URLs with XHR gives null content-type and content-length | 2017-06-07 |
| [137756](https://bugs.webkit.org/show_bug.cgi?id=137756) | INVA | WKWebView: JavaScript fails to load, apparently due to decoding error | 2014-10-20 |
| [137760](https://bugs.webkit.org/show_bug.cgi?id=137760) | DUPL | WKWebView: openDatabase results in DOM Exception 18 | 2016-04-27 |
| [144875](https://bugs.webkit.org/show_bug.cgi?id=144875) | INVA | WKWebView does not persist IndexedDB data after app close | 2015-05-28 |
| [149107](https://bugs.webkit.org/show_bug.cgi?id=149107) | FIXE | IndexedDB does not throw ConstraintErrors for unique keys | 2016-03-21 |
| [149205](https://bugs.webkit.org/show_bug.cgi?id=149205) | FIXE | IndexedDB openKeyCursor() returns primaryKeys in wrong order | 2016-03-30 |
| [149585](https://bugs.webkit.org/show_bug.cgi?id=149585) | DUPL | Heavy LocalStorage use can cause page to freeze | 2016-12-14 |
| [156125](https://bugs.webkit.org/show_bug.cgi?id=156125) | INVA | Fetching blob URLs with query parameters results in 404 | 2022-05-31 |
| [169851](https://bugs.webkit.org/show_bug.cgi?id=169851) | FIXE | Safari sends empty “Access-Control-Request-Headers” in preflight request | 2017-03-22 |</details> 

## Conclusion

I think cross-browser compatibility has improved a lot over the past few years. We have projects like [Interop](https://github.com/web-platform-tests/interop) and [Web Platform Tests](https://github.com/web-platform-tests/wpt/), which make it a lot more streamlined for browser teams to figure out what’s broken and what they should prioritize.

So if you haven’t yet, there’s no better time to get started filing bugs on browsers! I’d recommend first searching for your issue in the right bug tracker ([Chromium](https://issues.chromium.org/issues), [Firefox](https://bugzilla.mozilla.org), [WebKit](https://bugs.webkit.org)), then creating a minimal repro (CodePen, JSBin, plain HTML, etc.), and finally just including as much detail as you can (browser version, OS version, screenshots, etc.). I’d also recommend reading [“How to file a good browser bug”](https://web.dev/articles/how-to-file-a-good-bug).

Happy bug hunting!

## Footnotes

[Some](https://nolanlawson.com/2024/03/03/bugs-ive-filed-on-browsers/#comment-237539) [folks](https://news.ycombinator.com/item?id=39586419) have pointed out to me that `rdar://` links can mean just about anything. I always assumed it meant that the bug got re-routed to some internal team, but I guess not.