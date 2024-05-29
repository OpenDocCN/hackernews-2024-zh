<!--yml
category: 未分类
date: 2024-05-27 14:28:15
-->

# Block cookie banners on Firefox | Firefox Help

> 来源：[https://support.mozilla.org/en-US/kb/cookie-banner-reduction](https://support.mozilla.org/en-US/kb/cookie-banner-reduction)

Since the development of stricter consumer privacy laws that specifically target the usage and retention of user data, such as [GDPR](https://wikipedia.org/wiki/General_Data_Protection_Regulation), websites have adopted legal notices to inform users that they are using technologies such as cookies to store data on the user's device. To address the increasing frustration caused by these cookie banners on websites, [Firefox version](/en-US/kb/find-what-version-firefox-you-are-using) 120 introduces the cookie banner blocker. This feature is designed to make your browsing experience smoother by taking care of these annoying banners automatically. In this article, we'll show you how it works now, and how we intend to expand on it in the future.

# Why are cookie banners a problem?

Before we dive into the solution, let's understand why cookie banners can be a hassle:

*   **Annoying prompts:** Cookie banners, as you've likely experienced, can be quite annoying. They interrupt your browsing to ask a question that can often be confusing and irrelevant to your browsing, before allowing you to refocus on why you navigated to that site in the first place. Their goal being to create a sense of urgency, making it feel like you must choose between trading your data for access to the site, or navigating away in hopes of protecting your privacy.
*   **Deceptive designs:** Many websites employ tricky tactics when it comes to their cookie banners. They may design them in a way that makes it difficult for you to customize your settings. For example, they might make the Accept All Cookies button prominent, colorful and appealing, while burying the **Customize Settings** option in small text or a less noticeable location. This design choice can be misleading, effectively nudging users towards accepting all cookies without considering the consequences.
*   **Prompt fatigue:** The sheer volume of cookie banners that users encounter daily can lead to what we call “prompt fatigue”. This phenomenon occurs when users are bombarded with these pop-ups so frequently that they ignore or mindlessly accept them without actually reading the contents or understanding the implications (assuming they are written in a manner that is easy to understand in the first place). Sites effectively betting that you would trade your data to save you the several seconds of your life it takes to opt out over and over again. In this environment, the websites that employ the most deceptive patterns gain an unfair advantage, as they effectively exploit this fatigue to get you to hand over your data, even when it is not needed.

# How it works

When it comes to handling those cookie banners, the cookie banner blocker employs two clever methods:

*   **Cookie Injection:** This method involves setting an “opt-out” cookie whenever possible. Think of it like a digital note that tells the website, “I don't want to see this banner again.” It's a preemptive way of getting rid of those banners before they even have a chance to appear.
*   **Auto Click CSS Selector:** In cases where setting an opt-out cookie isn't an option, the Cookie Banner Blocker uses another technique. It effectively clicks on the banner's “decline” or “reject” option in a way that mimics what you might do manually. This action dismisses the banner as if you made a choice yourself.

So, in a nutshell, the cookie banner blocker is like your automated assistant that takes care of annoying cookie banners by either preemptively telling the website you opt out or by clicking on your behalf to make them go away. This way, you can spend less time opting out and more time browsing.

# Which banners are blocked?

The cookie banner blocker works by using a careful selection of websites that we've put together. We've compiled this list by considering the most frequently visited sites in our key regions to have the broadest impact. We're actively working to expand this coverage by incorporating support for consent management platforms and other top sites we haven’t gotten to yet.

# Get started

The cookie banner blocker is available starting from Firefox version 120, and it's automatically enabled for users in Germany browsing in Private Browsing Mode. Here's how you can make the most of it:

1.  **Visit supported websites:** The cookie banner blocker works on websites that have a decline option in their cookie banners.
2.  **Automatic decline:** When you visit a supported website, if the banner has a decline option, Firefox will automatically decline it for you.
3.  **Enjoy the benefits:** You can now browse without the interruption of annoying cookie banners.

# Customize Your Experience

If you want to enable or disable the cookie banner blocker for specific websites or altogether, here's how:

*   **Enable:** By default, it's on in private windows for users in Germany.
*   **Disable:** You can turn it off through preferences if you wish.

# Integrated cookie protections

Firefox has additional cookie protections that work seamlessly with the cookie banner blocker:

# Why Germany and private browsing mode?

Our initial launch in Germany and private browsing mode has specific reasons:

*   Private browsing mode displays cookie banners repeatedly, making this feature especially useful. Germany, as a part of the European Union, is a prominent market where cookie banners are noticeable due to GDPR.
*   We plan to gather insights from this launch before potentially expanding the feature to a broader audience.

By following these instructions, you can take full advantage of the cookie banner blocker in Firefox, simplifying your online experience while ensuring your privacy. Happy browsing!