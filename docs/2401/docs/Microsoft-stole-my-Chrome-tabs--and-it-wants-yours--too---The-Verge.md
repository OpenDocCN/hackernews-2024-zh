<!--yml
category: 未分类
date: 2024-05-27 15:18:09
-->

# Microsoft stole my Chrome tabs, and it wants yours, too - The Verge

> 来源：[https://www.theverge.com/24054329/microsoft-edge-automatic-chrome-import-data-feature](https://www.theverge.com/24054329/microsoft-edge-automatic-chrome-import-data-feature)

Last week, I turned on my PC, installed a Windows update, and rebooted to find Microsoft Edge automatically open with the Chrome tabs I was working on before the update. I don’t use Microsoft Edge regularly, and I have Google Chrome set as my default browser. Bleary-eyed at 9AM, it took me a moment to realize that Microsoft Edge had simply taken over where I’d left off in Chrome. I couldn’t believe my eyes.

I never imported my data into Microsoft Edge, nor did I confirm whether I wanted to import my tabs. But here was Edge automatically opening after a Windows update with all the Chrome tabs I’d been working on. I didn’t even realize I was using Edge at first, and I was confused why all my tabs were suddenly logged out.

After the shock wore off, I looked to make sure I hadn’t accidentally allowed this behavior. I found a setting in Microsoft Edge that imports data from Google Chrome on each launch. “Always have access to your recent browsing data each time you browse on Microsoft Edge,” reads Microsoft’s description of the feature in Edge. This setting was disabled, and I had never been asked to turn it on. You can check for the setting at edge://settings/profiles/importBrowsingData.

*Microsoft Edge with my Chrome tabs.*

Photo by Tom Warren / The Verge

So I went to install the same Windows update on a laptop, which actually resulted in it failing and my having to do a system restore. Once the system restore was complete, the same thing happened. Edge opened automatically with all of my Chrome tabs. I haven’t been able to replicate the behavior on other PCs, but a number of X users [replied to my post](https://twitter.com/tomwarren/status/1750175894306439601) about this saying they have experienced the same thing in the past.

It all seems to be related to a largely unreported import feature in Microsoft Edge that I had confirmed was disabled on my systems. Zach Edwards, a privacy and data supply chain researcher, freshly installed Windows and [replied to my post on X](https://twitter.com/thezedwards/status/1750952950598672455) to note he had found a new prompt during setup that reads:

> With your confirmation, Microsoft Edge will regularly bring in data from other browsers available on your Windows device. This data includes your favorites, browsing history, cookies, autofill data, extensions, settings, and other browsing data.

Microsoft says the data import is completed locally and stored locally but that it will be sent to Microsoft if you sign in and sync your browsing data. Microsoft displays a big blue accept button to encourage Windows users to enable the feature, with a darker “not now” button if you want to opt out.

I’m not entirely sure how Microsoft Edge automatically opened and imported my Chrome browser data without this setting enabled. I did notice on my PC that a full-screen prompt appeared after installing the update that morning, much like the ones Microsoft uses to encourage people to switch to Edge and Bing after a Windows update. The prompt appeared but disappeared in less than a second, so it’s possible this dialog crashed and Edge decided to do the import anyway.

I’ve asked Microsoft for comment on what I witnessed, but the company has not responded in time for publication.

I’m not alone in experiencing this, either. Multiple Windows users have been reporting this for months, turning to [Reddit](https://www.reddit.com/r/microsoft/comments/17c7w4i/edge_is_secretly_importing_my_data_from_google/) and [Microsoft’s own support forums](https://answers.microsoft.com/en-us/microsoftedge/forum/all/how-to-make-edge-stop-importing-from-chrome/4bcb43bb-b54a-4413-b24c-ea43b49cea54) for help. [Posters have confirmed](https://answers.microsoft.com/en-us/microsoftedge/forum/all/how-do-i-stop-edge-from-importing-bookmarks-and/22d7af7f-d111-481d-9ef3-13a703e61c18) the same feature is disabled, but the data imports keep happening.

Maybe it was just a bug for me this time, or perhaps I’m about to experience what others have been complaining about for months. But it’s clear Microsoft has been rolling out a feature to automatically import Chrome, Firefox, or other browser data from any web browser that isn’t Edge. It might be a useful feature for some, but if it’s faulty, then Microsoft needs to fix this quickly.

Given the full-screen prompt that disappeared just before I experienced the issue, how Microsoft convinces people to turn this setting on will also be important. Microsoft has already been [producing full-screen prompts](https://twitter.com/tomwarren/status/1464624781471404035) after some Windows updates that try to tempt people to switch to Microsoft Edge and Bing as defaults.

*Microsoft regularly uses full-screen prompts to try to get Chrome users to switch to Edge.*

Screenshot by Tom Warren / The Verge

It’s not always obvious to consumers what they’re actually changing, especially when Microsoft disguises things as “recommended” browser settings. It’s easy just to hit a big blue yes button instead of a “not now” one that’s less prominent. [Microsoft has a history](/23935029/microsoft-edge-forced-windows-10-google-chrome-fight) of using the sort of tactics we’ve seen from bloatware and spyware developers to promote its web browser. Those have ranged from Windows updates that have [launched Edge](/21310611/microsoft-edge-browser-forced-update-chromium-editorial) and pinned it to the desktop and taskbar without permission to [polls](/23930960/microsoft-edge-google-chrome-poll-why-try-another-browser) or [prompts](/2021/12/2/22813733/microsoft-windows-edge-download-chrome-prompts) when you attempt to download Chrome.

Microsoft Edge is [actually good](/23523118/microsoft-edge-browser-chrome-internet-firefox-explorer), so I sure hope the team building it isn’t about to resort to [more tricks](/23935029/microsoft-edge-forced-windows-10-google-chrome-fight) to get Chrome users to use it.