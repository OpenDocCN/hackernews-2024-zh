<!--yml
category: 未分类
date: 2024-05-29 12:37:59
-->

# Programmers are bad at managing state | Read the Tea Leaves

> 来源：[https://nolanlawson.com/2020/12/29/programmers-are-bad-at-managing-state/](https://nolanlawson.com/2020/12/29/programmers-are-bad-at-managing-state/)

“Have you tried turning it off and back on again?” is one of the most familiar tropes associated with tech support. But as someone who is often asked by family members for help with misbehaving devices, I find it to be one of my most effective tools.

The solution is primitive, but the logic behind it is surprisingly profound: programmers are bad at managing state.

As a programmer, I understand this fact intuitively. When a program I’ve written has first booted up, it is in its most pristine, perfect state. I have lovingly crafted every variable and array to be exactly how I intend it to be. If I have automated tests, then this is the state that is most heavily tested.

It’s only after this initial bootup phase that everything starts to go to hell. The user clicked on something! The user typed in a field! How could they? How could they besmirch my beautiful, perfect program with their grubby little hands?

Before you know it, the user has not only clicked and typed – they’ve pressed the browser’s back and forward buttons, and the refresh button, and they’ve booted up the program after a few weeks of inactivity, so that some of the data has become stale, or maybe they installed an update and now they’ve got some locally cached files that were designed for an older version of the app… It’s here that users run into the countless errors, crashes, and freezes associated with software that finds itself in a state that the original programmers didn’t intend.

This is why “turn it off and on again” works so well. You put the software back in a state that the programmers predicted, and poof, everything works again. Sometimes, though, you have to take this logic even further.

Recently my wife was having problems launching Steam. She hadn’t run it in a few years, so she reinstalled it and clicked the icon, but it refused to load. I googled around and couldn’t find a solution, so, on a hunch, I deleted the `~/Library/Application Support/Steam` folder, where Steam was apparently storing data, and relaunched it. Poof! It worked.

Another time, she was having trouble with a web app that was stuck on a loading bar, no matter how many times she refreshed. So I opened up the handy Chrome DevTools “Application” tab, clicked [“Clear storage”](https://developers.google.com/web/tools/chrome-devtools/manage-data/local-storage#clear-storage), and what do you know? After refreshing and logging in again, everything worked.

Another time, her MacBook refused to print anything on our HP printer after a macOS update. After about an hour of searching various web forums, it turned out that I needed to run a program called [HP Uninstaller](https://support.hp.com/us-en/document/c02440673) to remove old HP software. Poof! Everything worked.

Having a dedicated program to clean up your own program’s files may seem a bit ridiculous, like an admission of defeat, but I actually think it’s kind of brilliant. As a programmer, it’s impossible to predict all the states that your program can end up in. You can use something like [XState](https://xstate.js.org/) to help visualize it, but when you start multiplying the possible states by the cached configuration files by the different versions of the software by the *different teams of people* who built each version… it just becomes unfeasible to plan for every outcome. (Let alone write tests for every possible state!) So rather than having software that doesn’t work, maybe it’s better to just admit your own human frailty, and offer a way for users to start from scratch.

Mozilla’s [“Refresh Firefox”](https://support.mozilla.org/en-US/kb/refresh-firefox-reset-add-ons-and-settings) feature is another brilliant application of this principle. If you haven’t used Firefox in a while, then it will pop up a little alert offering to blow everything away and start from scratch. This is probably a great way to avoid attrition to other browsers: I’m sure plenty of people switch from Browser A to Browser B because on day 1, Browser B feels so much faster – not because B is actually superior to A, but because B isn’t bogged down with a bunch of extensions, settings, history, leftover update files, etc.

If you’ve ever reinstalled Windows or Android from scratch, and observed how remarkably fast and lightweight everything suddenly feels, then you’ve probably also seen this phenomenon in action.

I don’t have a solution to this problem. Software is complicated, and unless you have the luxury of writing code that can be tested with formal proofs, it’s impossible to predict every possible state that your code can be in. So maybe the best bet is to just provide some kind of escape hatch, so that at least it’s easy to “turn it off and back on again.”