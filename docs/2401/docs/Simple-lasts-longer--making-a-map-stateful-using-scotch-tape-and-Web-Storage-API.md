<!--yml
category: 未分类
date: 2024-05-27 14:38:53
-->

# Simple lasts longer: making a map stateful using scotch tape and Web Storage API

> 来源：[https://newsletter.pnote.eu/p/simple-lasts-longer](https://newsletter.pnote.eu/p/simple-lasts-longer)

*This is less useful than I thought.*

I [made a map](https://newsletter.pnote.eu/p/mapping-space-invader-mosaics-in-paris) of the collectible [Space Invader](https://en.wikipedia.org/wiki/Invader_(artist)) mosaics and [put it online](https://pnote.eu/projects/invaders/). I thought it would be helpful for planning walks (and runs 🚀) with some Space Invader hunting along the way.

But I missed one problem. Can you spot the issue?

Wow, so many mosaics! And yes, that’s the problem.

With almost 1500 invaders in Paris, to use a map to *actually* plan your walks, you need to keep track of the invaders you already collected. It’s no fun to run in circles :). (And the official app doesn’t give you any points for repeatedly taking photos of the same mosaic. It actually mocks you and asks if you’re drunk 🙃)

My map couldn’t track what's already "collected”. It was perfectly simple and perfectly “stateless”.

Stateless websites are easy to manage. You just need to publish a few static files on the Internet. The website looks the same to every user and it never changes (unless you publish an update).

To keep track of the mosaics flashed by each user, I need to teach the website to store the list of what’s already collected … somewhere. But where?

*   The traditional answer is to run a **database** and store the collection of each user there.

*   A more trendy, newer option is to use a 3rd party **service** that acts like a database, but runs on someone else’s server. [Firebase](https://firebase.google.com/docs/database) is a well-known example.

The second approach is less work than the first one (the website can directly talk to Firebase, I don’t need to set up any servers myself), but both require some work. Worse yet, they increase the complexity of the project, adding a dependency on another service.

Complexity kills projects 🔪 (especially the fragile hobby projects of distracted minds 💫), so let’s do something simpler. Could we store the list of collected invaders directly in the browser of each user?

Thanks to the [Web Storage API](https://developer.mozilla.org/en-US/docs/Web/API/Web_Storage_API), any website can store a small amount of data on your computer/phone, and it will be persisted through reboots. That’s promising!

I modified the map website to include buttons for marking each invader as flashed, and made it persist the state in the local Web Storage.

This works by default, without any permission prompts. Using Chrome developer tools, we can inspect the state the map stores in the browser. It’s just the list of invader IDs:

This is very simple and it works! You can see the published updated map **[here](https://pnote.eu/projects/invaders/map/).**

No state on the server needed. No user accounts, no authentication, no “I forgot my password” problem. Also, when we don’t store any data on the server, we cannot lose or leak it. ([Local-first software](https://www.inkandswitch.com/local-first/).)

If this solution is so nice, why all the major websites insist on creating an account and storing our state server-side?

For many reasons, one of which is that when the data is stored locally in the browser, it’s not synced across devices and not backed up. If the user re-installs their browser, they lose all state.

Since we’re into simplicity, maybe we can allow users to take care of backup themselves?

It only took 50 lines of code to add a little popup with a feature to import/export the list of collected mosaics. And ChatGPT was happy to practically write all of it:

We ended up with a solution that tries to be the best of all worlds. The website remains simple and it doesn’t have to manage user state. But the users can still mark them as collected, and even export and backup their collection if they want to.

Hobby projects are fragile. They can only last if they’re simple to maintain, otherwise keeping them working becomes tedious and the author eventually gives up.

I’m still maintaining a database-based hobby project I started 15 years ago. It ended up growing a small community that relies on it. I’m happy about it, but I feel the ongoing maintenance cost of keeping the app running and up to date.

Wiser from this experience, these days I try to build all hobby projects in a way that minimizes dependencies and general fragility. If needed, the app should keep running with zero modifications for the decade to come 🤞.

**Simple is not only faster to launch, it also lasts longer.**

Are you thinking of starting any hobby project in 2024? How simple can you make it for the initial version?

*   📝 [Enso](https://write.sonnet.io/), a text editor where you can only write, but not edit. Great for breaking through writer’s block: just let the thoughts flow

*   🎶 [muted.io](https://muted.io/), a toolkit of music theory tools, running in the browser

*   💻 [Turing Complete](https://turingcomplete.game/), a computer science game where you build a computer from increasingly complex components, starting with logic gates

A wave of cold weather is expected in Paris anytime now. Great time for reading and writing with a cup of tea. Or maybe looking up that mulled wine recipe.

Stay warm! 💫
– Przemek