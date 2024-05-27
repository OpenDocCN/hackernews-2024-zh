<!--yml
category: 未分类
date: 2024-05-27 13:05:30
-->

# The greatest bug I never fixed - The blog of makandra

> 来源：[https://blog.makandra.com/2010/04/the-greatest-bug-i-never-fixed/](https://blog.makandra.com/2010/04/the-greatest-bug-i-never-fixed/)

Bugs [ain't fun](http://use.perl.org/use.perl.org/_schwern/journal/40260.html). Except when they are. This is the story of the greatest bug I never fixed.

In an earlier life I wrote addons for [World of Warcraft](http://www.wow-europe.com). Aside from being addicted to the game at the time, it was a mindblowing experience for someone obsessed with plugin architectures and the evolution of public APIs. It was also a great excuse to learn [Lua](http://en.wikipedia.org/wiki/Lua_%28programming_language%29), which is a fun language.

One of the better addons a friend and I built during that period was *FriendNet*. FriendNet helped to deal with the greatest challenge in the game: Finding people to play with who aren't dicks. The addon took your in-game friend list and shared it with all your friends. In return, they shared their friends with you. As a result the number of people you could trust weren't dicks increased.

Because any form of network communication was forbidden by the addon API, we had to come up with a hack to distribute those friend lists. What we did was serialize those lists into in-game chat messages and whisper them to the receiving player's avatar.

Yes, years before your Mom discovered Facebook we were tunnelling social graphs over the chat channels of a virtual world. We should have gotten funding.

Anyway, FriendNet did have some quality issues. P2P applications are hard to get right, and you can't just make people upgrade every time you release a bugfix. So we had all those different versions talking with each other, resulting in a lot of emergent behaviour and hard to reproduce bugs.

One bug was especially hard to pin down: Once every few days FriendNet would encounter a corrupt message, resulting in parts of the addon GUI to be messed up. We received enough bug reports to believe that the issue was real, but we could never reproduce the bug no matter how hard we tried. It was the bug from hell. Eventually we gave up, blamed it on a buggy API and moved on.

The solution came to me years after I stopped playing the game. When I realized what had gone wrong, the light almost blinded me.

You know, your character can [get drunk in World of Warcraft](http://www.wowwiki.com/Drunk). After drinking enough virtual booze, your screen will start to blur and your character will no longer move in straight lines. Also when you type something into the chat like

```
Penelope says: I'm so wasted 
```

It will come out like this:

```
Penelope says: I'm sho washted ...hic! 
```

Because FriendNet serialized all of its communication into chat messages, it sent corrupted data whenever the broadcasting player was drunk.

FriendNet has long since disappeared from the addon sites you could once download it from. I doubt it would even boot up in a current version of the game. But even though I never got to fix that bug, I still remember it with fond amusement.