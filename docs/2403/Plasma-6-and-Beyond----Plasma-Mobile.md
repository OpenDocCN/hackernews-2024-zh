<!--yml
category: 未分类
date: 2024-05-29 11:59:31
-->

# Plasma 6 and Beyond! - Plasma Mobile

> 来源：[https://plasma-mobile.org/2024/03/01/plasma-6/](https://plasma-mobile.org/2024/03/01/plasma-6/)

The Plasma Mobile team is happy to announce the release of Plasma 6 for mobile devices!

It has been a long year of development and porting since our last major release of the shell, as well as mobile applications. This post will outline many of the highlights!

## Housekeeping

The website was refreshed in anticipation for this release. Notably, the [installation](https://plasma-mobile.org/get) page was enhanced to better outline information about the various distributions that provide Plasma Mobile.

During the Plasma 6 development period, [postmarketOS](https://postmarketos.org/) graciously provided us with a "nightly" repository of KDE packages tracking git repositories. We have found this critically useful for development. [You can find instructions on how to use this repository here](https://wiki.postmarketos.org/wiki/Nightly).

## Plasma

[Devin wrote a blog post covering some technical details](https://espi.dev/posts/2023/12/plasma-mobile-towards-6/) covering the big aspect of the work porting all of the shell components to Qt and Plasma 6 APIs. A lot of functional work was done as well.

### Homescreen

With Plasma 6, we are once again switching back to the Folio homescreen as default. The homescreen will once again have customizable pages to place apps and widgets, as well as an app drawer and search. Devin spent time rewriting it from scratch, addressing the limitations that existed in Plasma 5, in particular the customizations are now tied to each page, no longer causing reordering issues from screen rotations.

Notable Features:

*   Customizable pages to place apps and widgets
*   Folders
*   Drag and drop customization
*   Widgets (still some known issues, see below)
*   App drawer (swipe up)
*   KRunner search (swipe down)
*   Row-column flipping for screen rotations
*   Customizable row and column counts
*   Customizable page transitions
*   Import and exporting of homescreen layouts as files
*   and more!

### Interoperability

In Plasma 5, Plasma Mobile required several config files to be installed on the system in order to set some settings needed in Plasma. For Plasma 6, Devin created a new settings service that automatically handles this situation. This allows for interoperability with Plasma Desktop being simultaneously installed on the system, and eliminates the need to install custom configuration.

### First time setup

Devin added an initial setup window that guides users to configure some basic aspects of the system, such as Wi-Fi and cellular settings.

### Authentication dialog

Devin worked on porting the authentication dialog so that it has a mobile form factor.

### Docked mode

Devin worked on adding a "docked mode" quicksetting which, when activated, enables window decorations and minimize/maximize/close buttons, and also stops enforcing application windows to be in fullscreen.

### Navigation panel

Devin added a setting to always show the keyboard toggle button. He also made it so that if there is enough screen space, the panel goes on the bottom for tablets rather than on the side.

### Task switcher

The task switcher was moved from the Plasma shell and into KWin to improve code maintainability.

### Vibrations

In Plasma 5, the default vibration speed was hardcoded to the PinePhone's motor, which could not register lower vibration durations. This resulted in other devices having overly strong vibration effects. Vibrations now default to an acceptable level for other devices, and can still be configured in the shell settings module.

### Flashlight

The flashlight quicksetting in Plasma 5 is hardcoded to the PinePhone. Florian worked on making it so that it now works with all phones in Plasma 6!

### Settings

Devin improved the cellular settings module so that interaction tasks are asynchronous and do not freeze the UI. He also fixed unnecessary password prompts from being shown when the module is first opened.

Devin ported the Wi-Fi settings module to mobile form components, making it consistent with other modules. He overhauled the time settings module to newer components as well, making setting the time and date more intuitive. He also did some work to ensure the UI does not freeze when applying settings.

Joshua refactored the look and feel of the Push Notifications KCM and added UnifiedPush support to NeoChat so you'll never miss a message, even when NeoChat is closed (that is, if you want to get notifications, of course!)

Mathis created a new icon for the application.

![](img/1a20cce536f6c3fbdc2070f0d49f0aeb.png)

## Known Issues

Regrettably, our team resources are limited, and so we still have several outstanding issues that were not addressed in time for this release:

*   **Scrolling in Qt6 with a touchscreen has very low inertia compared to Qt5** and we cannot change the platform default, see [this](https://invent.kde.org/teams/plasma-mobile/issues/-/issues/273), reported in upstream to Qt [here](https://bugreports.qt.io/browse/QTBUG-121500)
*   **Some homescreen widgets open popups** every time they are moved around
*   **Limited widget selection**. Currently we just have existing widgets from the Plasma Desktop
*   **The screen recording quicksetting was removed for the time being**. It needs further investigation on porting it to new APIs
*   **Gesture-only mode was removed for the time being**. We need to investigate making KWin's gesture infrastructure fit our needs
*   **Sometimes application windows extend under the navigation bar**. We need to investigate this regression with KWin
*   **The task switcher is slow on the PinePhone in particular**. We need to investigate the performance of the KWin effect

There are other issues also being tracked in our [issue tracker](https://invent.kde.org/plasma/plasma-mobile/-/wikis/Issue-Tracking).

## Applications

A tremendous amount of work went into the massive effort behind porting all of our applications to Qt6 and KF6\. We would like to extend a big thank you to everyone involved!

Many major announcements for applications are in the [Megarelease](https://kde.org/announcements/megarelease/6) post, so be sure to check there too!

### Photos (Koko)

While the application is still called Koko, user-facing text now calls it "Photos" to make it easier for you to find. (Devin Lin, KDE Gear 24.02, [Link](https://invent.kde.org/graphics/koko/-/merge_requests/133))

A new icon was created for the application, replacing the gorilla (Mathis Brüchert & Devin Lin, KDE Gear 24.02, [Link](https://invent.kde.org/graphics/koko/-/merge_requests/134))

![](img/ce7c33ecdd4c404530ecd7a0b8144aaa.png)

### Clock

The Clock app now pauses MPRIS media sources when an alarm or timer starts ringing and resumes (previously paused) sources once the alarm is dismissed (Luis Büchi, KDE Gear 24.02, [Link](https://invent.kde.org/utilities/kclock/-/merge_requests/109))

### Calculator

A new configuration page was added to the calculator app, allowing to set decimal places, angle units and a parsing mode (Michael Lang, KDE Gear 24.02, [Link](https://invent.kde.org/utilities/kalk/-/merge_requests/83))

We improved the history view to allow for the removal of entries, and the drawer-style area for extra functions got replaced with a swipe view (Michael Lang, KDE Gear 24.02, [Link 1](https://invent.kde.org/utilities/kalk/-/merge_requests/78), [Link 2](https://invent.kde.org/utilities/kalk/-/merge_requests/79))

A number of new features and QoL improvements were added, including automatic font resizing when entering long expressions, better rendering of exponents, and more actual calculation features: random number, base n root/log, differential functions, standard deviations, and a lot more... Oh! and we switched math the engine to libqalculate! (Michael Lang, KDE Gear 24.02, [Link 1](https://invent.kde.org/utilities/kalk/-/merge_requests/77), [Link 2](https://invent.kde.org/utilities/kalk/-/merge_requests/75), [Link 3](https://invent.kde.org/utilities/kalk/-/merge_requests/80), [Link 4](https://invent.kde.org/utilities/kalk/-/merge_requests/73))

## Kasts

Apart from some minor visual changes related to the Qt6 and KF6 update and several fixes regarding images and playback controls, Kasts will now update podcasts dramatically faster by not parsing RSS feeds that haven't changed (Bart De Vries, KDE Gear 24.02 [Link 1](https://invent.kde.org/multimedia/kasts/-/merge_requests/141))

The color theme (e.g. Breeze light, Breeze dark) can now be selected manually (Bart De Vries, KDE Gear 24.02 [Link 2](https://invent.kde.org/multimedia/kasts/-/merge_requests/140))

Network checks for metered connections can now be disabled altogether. This can be useful on systems that are not able to reliably report the connection status (Bart De Vries, KDE Gear 24.02 [Link 3](https://invent.kde.org/multimedia/kasts/-/merge_requests/146))

## Hash-o-Matic

Hash-o-Matic is a new application that lets you verify the authenticity of files by using their MD5, SHA-256 and SHA-1 hashes or their PGP signature.

Hash-o-Matic also let you generate hash for a file and compare two files.

## Contributing

Do you want to help with the development of Plasma Mobile? We are a group of volunteers doing this in our free time, and are ~~desperately~~ looking for new contributors, **beginners are always welcome**!

See our [community](https://plasma-mobile.org/join) page to get in touch!

We are always happy to get more helping hands, no matter what you want to do, but we especially need support in these areas:

Even if you do not have a compatible phone or tablet, you can also help us out with application development, as you can easily do that from a desktop!

Of course, you can also help with other things besides coding! For example, take Plasma Mobile for a spin to help us test bugs and triage reports! Check out the [device support for each distribution](https://www.plasma-mobile.org/get/) and find the version which will work on your phone.

Another option would be to help us in making these blog posts more regular again. We really don't want another 6 months of silence, but writing these take a surprising amount of time and effort.

If you have any further questions, view our [documentation](https://invent.kde.org/plasma/plasma-mobile/-/wikis/home), and consider joining our [Matrix channel](https://matrix.to/#/#plasmamobile:matrix.org). Let us know what you would like to work on or where you need support to get going!

Our [issue tracker documentation](https://invent.kde.org/plasma/plasma-mobile/-/wikis/Issue-Tracking) also gives information on how and where to report issues.

## On a final note...

We would like to thank all of the contributors across KDE that have made this megarelease possible! We could not have done it without you. Thank you!

![](img/a1cd68ab1db3b952e06d8f812117c646.png)