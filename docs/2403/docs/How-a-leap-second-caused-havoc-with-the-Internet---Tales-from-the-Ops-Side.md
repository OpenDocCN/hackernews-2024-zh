<!--yml
category: 未分类
date: 2024-05-27 14:30:27
-->

# How a leap second caused havoc with the Internet - Tales from the Ops Side

> 来源：[https://www.talesfromtheopsside.com/blog/how-a-leap-second-caused-havoc-with-the-internet/](https://www.talesfromtheopsside.com/blog/how-a-leap-second-caused-havoc-with-the-internet/)

[stack.io](https://www.stack.io/) CEO, Hany Fahim, encountered an unexpected puzzle. This puzzle went beyond technical challenges and forced him to think deeply about the ripple effects space and time have on Ops, and the Earth itself. 

**The missing connection**

Without warning, on a summer evening in June 2012, multiple sites went down. At first, Hany thought it might be an attack, but this didn’t make sense because other sites seemed to be operational. Even odder, several systems reported a high load. While an increase in workload isn’t uncommon for a single customer, simultaneous upticks across systems are unusual. 

When Hany logged in to investigate, answers eluded him. He only had more questions.

“I noticed that the system was fairly responsive. For a system complaining about a high load, I would expect it to be sluggish.”

Several different technologies, such as MySQL, Ruby, and Java, sharing nothing in common, all experienced the same symptoms. Hany compares the puzzle to someone’s phone, TV, and radio all acting up in the same way, even though they are different devices. What link did these seemingly disconnected systems share? 

**An answer in time**

After using strace (the diagnostic equivalent of popping open the hood of a malfunctioning car), Hany finally found the missing link!

“MySQL, Java, Ruby—they were all asking the CPU for the exact same thing over and over and over again: *What time is it*?”

Hany didn’t understand why these systems were all obsessed with the time. Turning to the Internet, he discovered something highly unusual and fascinating! An extra [leap second](https://en.wikipedia.org/wiki/Leap_second) had just been added to clocks around the world that day.

Hany didn’t understand how a leap second could cause so much havoc, but in a last-ditch effort he tried the “universal fix to all things tech: turning it off and on.” 

Incredibly, this simple solution seemed to make the systems happy. Everything was back up and running as if nothing out of the ordinary had even happened.

**A research rabbit hole**

Hany remained puzzled over what had just occurred, so he decided to do some digging. He soon discovered that Reddit, Mozilla, and even airlines had been impacted. 

He learned that a leap second calls into question what humans know – or think we know – about time. A leap second is “an extra second occasionally applied to accommodate the difference between precise time and imprecise observed time.” 

Precise – or solar – time is what humans know when measuring time, but it doesn’t take into account inconsistencies in the Earth’s rotation. 

Hany explains: “Due to physics, at certain times of the year, the Earth moves faster along its orbit than others. How do we base time off of this, then?” It turns out that we don’t.  

In addition to some days being longer than others based on these inconsistencies, major events, such as earthquakes, can also speed up the earth’s rotation. Even human-made structures have the ability to affect time.

“NASA has calculated that the water stored in the Three Gorges Dam in China has increased the length of the day by 0.06 microseconds,” notes Hany.

**The ripple effects of a leap second**

Clocks are based on Coordinated Universal Time (UTC), which operates as a kind of go-between for precise and imprecise time. Usually, leap seconds are announced with about six months’ notice. But how could this increase in time affect Ops?

“The problem with leap seconds is everyone has a different way of implementing it. For Linux systems, which is what we use, as well as a large portion of the Internet, the day can only ever have 86,400 seconds.”

Diving into the source code of Linux, Hany discovered that five years prior, someone in the Linux community flagged a potential issue with its leap second implementation. A deadlock, or a computer freeze, could occur. To fix this, a single line of code was changed. A comment was left behind with this change to note one seemingly small caveat: “The only possible side effect of this removal might be that the timer fires one second too late after a leap second.’ Scarily, this side effect is what took down large portions of the Internet in 2012.

Hany journeys further into the intriguing – and sometimes mind-boggling – rules of the universe and its impact on Ops in [this informative podcast](https://www.talesfromtheopsside.com/blog/episode-2-earthquakes-and-the-moon/)!