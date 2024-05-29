<!--yml
category: 未分类
date: 2024-05-27 14:47:56
-->

# Built-in workaround for applications hiding under the MacBook Pro notch

> 来源：[https://flaky.build/built-in-workaround-for-applications-hiding-under-the-macbook-pro-notch](https://flaky.build/built-in-workaround-for-applications-hiding-under-the-macbook-pro-notch)

💡

**TL;DR:** You can adjust MacOS whitespace settings from the command line to display more application icons in the top right section of the menu bar.

## [](#cbd74db3d6d549f4a966ee049c6616e3 "Hidden Apps Beneath the 14” and 16” MacBook Pro Notch")Hidden Apps Beneath the 14” and 16” MacBook Pro Notch

A frustrating aspect of the new MacBook Pro models is the notch. The notch itself isn't the problem; rather, it's that Apple hasn't automatically adjusted the menu bar icons so they don't hide behind the notch when many apps are running.

My colleagues often suggest purchasing

[Bartender](https://www.macbartender.com/)

for about 20€ to solve this issue. While it offers many features, I've refused to pay for a 3rd party solution to Apple's poor design decision. I have nothing against Bartender but I just don’t want to install yet another app into my machine to solve such a simple problem.

Recently, I discovered a free, built-in macOS workaround that doesn't require installing Bartender or any other additional apps. See the results in the images below:

### [](#00131674fa014ca6a177465547292c21 "Adjusting the Menu Bar Whitespace Settings from the Command Line")Adjusting the Menu Bar Whitespace Settings from the Command Line

You can modify the default padding and spacing in the Menu bar by opening **Terminal.app** and executing the following commands:

```
# Change the whitespace settings value
defaults -currentHost write -globalDomain NSStatusItemSelectionPadding -int 6
defaults -currentHost write -globalDomain NSStatusItemSpacing -int 6

# After running these commands, you need to log out and log back in
```

You can adjust the values from `0` to `6` to accommodate even more icons. Personally, I found `6` to be a good fit. This will allow you to add more items to your menu bar before they will be hidden under the notch. I estimate that with the value of `6` I can fit 21 apps to the menu bar with my 14” MBP. If your apps will again be hidden under the notch you can just lower this value.

‼️

**NOTE:** Applying this workaround will allow you to fit more items to your menu bar but the same issue will resurface if you keep adding more apps which run in the menu bar.

### [](#c2c7b65789f64fa38e111a3c48b0fa64 "Reverting to the Original Values")Reverting to the Original Values

If you're unhappy with the results, you can delete the settings by executing the following commands:

```
# Revert to the original values
defaults -currentHost delete -globalDomain NSStatusItemSelectionPadding
defaults -currentHost delete -globalDomain NSStatusItemSpacing

# After running these commands, you need to log out and log back in
```

### [](#8d04b1e76bdc4b5280fa3d914b4a7efa "Sources")Sources

I first learned about these settings in this answer on the Apple-themed Stack Exchange, Ask Different:

### [](#81f90025480e4081af6df0854230090c "Changelog")Changelog

*   **12-02-2024 3:17 PM**: I clarified that I have nothing against [Bartender](https://www.macbartender.com) after [Eric_WVGG shared a good point in HN](https://news.ycombinator.com/item?id=39343919#39344587). I haven’t been writing a lot before and it seems I still have work to do to be more considerate towards others 🙇

*   **12-02-2024 9:12 PM:** I renamed the article from `Native macOS fix for applications hiding under the MacBook Pro notch` to `Built-in MacOS workaround for applications hiding under the MacBook Pro notch` to avoid confusion.