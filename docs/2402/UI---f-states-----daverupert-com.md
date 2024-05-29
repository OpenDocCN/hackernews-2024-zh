<!--yml
category: 未分类
date: 2024-05-27 14:54:50
-->

# UI = f(statesⁿ) - daverupert.com

> 来源：[https://daverupert.com/2024/02/ui-states/](https://daverupert.com/2024/02/ui-states/)

“UI is a function of state” is a pretty popular saying in the front-end world. In context (*pun intended*), that’s typically referring to application or component state. I thought I’d pull that thread a little further and explore all the states that can effect the UI layer…

## First-party application states

Every application whether it’s a to-do list or a shopping cart or some radically complex app will have some state. State isn’t uniform and typically exists at a variety of different levels. We’ll start at the top and drill down…

### Global state

Data stores and feature gating that typically happens at the application level.

*   Stores - different locations for storing data
    *   Application store - Redux, Vuex, Mobx, Signals
    *   Browser store - localStorage, sessionStorage, cookies, IndexedDB
*   Data - different types of global data
    *   Access control data - authentication tokens, paid/unpaid, geolocated, age, verified, member, etc.
    *   User data - name, icon, etc
    *   Collections - e.g. list of posts
    *   Session data
    *   … etc

### Page/Component state

Vince Speelman’s wonderful [Nine States of Design](https://medium.com/swlh/the-nine-states-of-design-5bfe9b3d6d85) do a great job summing up all the states that a page or component might exist in.

*   Nothing - An empty element
*   Loading - A `fetch` is happening
*   None - No items returned
*   One - A single item comes back
*   Some - A few items comes back
*   Too Many - Too many items, need pagination (or similar)
*   Incorrect - An error occurred
*   Correct - A success happened
*   Done - The operation finished

Vince’s list is perfect to me and keeps being relevant after all these years, I would add two items.

*   Custom states - Any bespoke or custom states relevant to your application
*   Realtime multi-player event mesages - Picture the constant updating state in a chat app or realtime stock ticker. Stored at the component level or thrown into global state.
*   Scroll-position - Pages and components often need to know if they’re scrolled in or out of viewport.

In my experience both the page and each component will contain some mixture of these states as well as being reactive to global state changes.

### Element state

Individual elements can (and will) have their own states. At this layer, features of HTML, CSS, and ARIA start to reveal themselves.

*   [Cursor](https://developer.mozilla.org/en-US/docs/Web/CSS/cursor) state
    *   `default, pointer, wait, text, move, grab, crosshair, zoom-in, zoom-out`, … etc
    *   Custom cursors
*   [IntersectionObserverEntry](https://developer.mozilla.org/en-US/docs/Web/API/IntersectionObserverEntry/isIntersecting)
    *   isIntersecting = true, false
*   Stacking context
*   Attribute states - states reflected in HTML
    *   Visibility = `hidden, visible`
    *   Language = `dir, lang`
    *   Functionaltiy = `contenteditable, draggable, invoketarget`
    *   Display = `inert, open, popover`
    *   Loading = `lazy, eager`
*   [Pseudo-class](https://developer.mozilla.org/en-US/docs/Web/CSS/Pseudo-classes) states - states reflected in CSS
    *   Action = `:hover, :active, :focus, :focus-visible, :focus-within`
    *   Input = `:autofill, :checked, :disabled, :valid, :invalid, :user-valid, :user-invalid, :required`, … etc.
    *   Display = `:fullscreen, :modal, :picture-in-picture`
    *   Language = `:dir(), :lang()`
    *   Location = `:link, :visited, :target`, … etc.
    *   Resource = `:playing, :paused`
    *   [CustomStateSet](https://developer.mozilla.org/en-US/docs/Web/API/CustomStateSet) = custom states for web components
    *   … etc
*   [ARIA states](https://developer.mozilla.org/en-US/docs/web/Accessibility/ARIA/Attributes) - [user-facing states](https://css-tricks.com/user-facing-state/) reflected in ARIA
    *   `aria-current`
    *   `aria-expanded`
    *   `aria-pressed`
    *   `aria-hidden`
    *   … etc

## Second-party user (or device) states

The user of the application and their device, peripherals, and browser have a lot of say in how the final application renders. This is by design and [built into the foundations of the web](https://www.w3.org/TR/html-design-principles/#priority-of-constituencies).

### Language and localization

Surprise! Not all users live in US-West-2.

*   Text direction = `ltr, rtl`
*   Writing mode = `horizontal-tb, vertical-lr, vertical-rl`
*   Distance to server/CDN (latency)
*   Auto-translations
*   Words are long (e.g. German)
*   Words are short (e.g. Chinese)

### Device constraints

A user’s device has a lot of variation and customization and may be your biggest unknown bottleneck for rendering to glass.

*   Network connection = Fiber, cable, wi-fi, 5G, 4G, 3G, “[lie-fi](https://web.dev/articles/performance-poor-connectivity#lie-fi)”
*   Viewport = `height, width, initial-scale, horizontal-viewport-segments, vertical-viewport-segments, viewport-segment-width, viewport-segment-height`, … etc.
*   Environment contstants = `safe-area-inset-*`, `titlebar-area-*`, `keyboard-inset-*`(e.g, [iPhone Notch](https://css-tricks.com/the-notch-and-css/), rounded corners, [installed apps](https://alistapart.com/article/breaking-out-of-the-box/))
*   Pixel density = 1x, 2x (Retina), 3x, … etc.
*   Low-power mode
*   Screen brightness
*   CPU speed
*   GPU/dGPU
*   L1/L2/L3 cache
*   CPU/GPU/Memory contention (e.g., other apps open)
*   Color-gamut support = Rec2020, P3, sRGB, …etc
*   Keyboard = Embedded, External, [T9](https://en.wikipedia.org/wiki/T9_(predictive_text)), Virtual On-screen, Touchbar, … etc
*   XR support = `inline, immersive-vr, immersive-ar`

### Modalities

Users aren’t uniform in how they interact with their devices and may be using one or any combination of inputs and outputs all at once.

*   Inputs
    *   Mouse = one-button, two-button, mousewheel, trackball, touchpad, high/low DPI
    *   Keyboard = 100%, 60%, 10-key, querty, colemak, Ergonomic, split, mechanical, … etc
    *   Touch/Tap = coarse pointer, no hover
    *   Stylus = fine pointer, hover, pressure sensitivity
    *   Gestures = pinch-zoom, two/three/four-finger swipe
    *   Motion = accelleration, shake-to-undo, bump, … etc
    *   Orientation = landscape, portrait, alpha/beta/gamma (360º/180º/90º, respectively) rotation
    *   Speech recognition = Dragon NaturallySpeaking, voice assistants
    *   Switches = button, sip, puff
    *   Eye-tracking
    *   Gamepad
    *   XR = 3 DOF, 6 DOF
*   Outputs
    *   Screen
    *   Text-to-speech
    *   Screen reader
    *   Braille
    *   Screen magnifier
    *   Vibration
    *   RSS?

### Browser states

Finally, a user’s browser choice and preferred plugins determines a lot about how they experience (or would prefer to experience) your UI and your application can be responsive to some of those preferences.

*   User preferences
    *   prefers-color-scheme = light, dark, forced-colors
    *   prefers-reduced-motion = reduce, no-preference
    *   prefers-reduced-transparency = reduce, no-preference
    *   user zoom = 100% to 400%
    *   text size = small to x-large
*   Features and functionality
    *   Browser version = latest version, last 2 versions, older
    *   Feature detection = `@support` or polyfills
    *   Color-gamut support = Rec2020, P3, sRGB, …etc
    *   Browser cache hit
    *   Service worker hit
    *   Display mode = fullscreen | standalone | minimal-ui | browser
    *   `beforeinstallprompt`
    *   Print mode
    *   Reader mode
    *   JavaScript disabled = yes, I actually know people who do this.
    *   Sleeping tabs
*   Permissions
    *   Camera = true, false
    *   Microphone = true, false
    *   Geolocation = allowed, not allowed, only while using the app
    *   Notifications = true, false
    *   File access = true, false
*   Plugins
    *   Ad blockers - UBlock, Safari ITP, Ghostery, … etc
    *   Custom plugins/Boosts

### User states

Thus far we’ve talked about technology, now let’s consider the actual human being at the other end of the transaction. A user’s physical or mental state impact their cognitive or literal bandwidth to enjoy your experience.

## Third-party service states

Third parties have an outsized impact on the user experience of a UI.

### Availability/Status

The status/availability/uptime of other servers is the biggest surface area for failure for a UI.

*   Server hardware status = online, offline, partial availability
*   Database status = online, offline, transaction locked
*   API status = online, offline, partial availability
*   Web font service = online, offline, partial availability
*   DNS status = may take up to 72 hours to resolve
*   Package dependencies = working, broken, malicious injection, protestware, … etc
*   Asset delivery and caching status

### Script injections

Third-party script injections are the biggest contributor to performance degradation on a UI. Some services even take over the user experience of an application. This is often out of your control.

*   Analytics/tracking services
*   User session recording services
*   A/B test injections
*   Instructional overlays
*   Accessibility overlays
*   Third-party authentication (OAuth) services
*   Captcha/verification services

You haven’t truly lived life until one of these unchecked services takes down an application.

## There’s more to UI than just state…

I’m sure I’ve forgotten whole categories of state. I haven’t even gotten into the hundreds of CSS properties and the thousands of values and their potential conflicts. I didn’t talk about the intricacies of styling form controls or building dropdowns with the required ARIA property combinations. I didn’t touch on [the decision matrix required to put an image on the page](https://cloudfour.com/thinks/responsive-images-101-definitions/). Nor does this get into [setting all the proper tags in the `head`](https://github.com/joshbuchea/HEAD) [in the right order](https://rviscomi.github.io/capo.js/). Nor does this discuss [microformats](https://microformats.org/) for SEO or all the code you need to properly setup a UI to send analtyics data.

In closing, hire people who are good at UI.

* * *

*Edit 2/16/24*