<!--yml
category: 未分类
date: 2024-05-27 15:01:17
-->

# Ad agency boss had two Ferraris but wouldn’t buy a server • The Register

> 来源：[https://www.theregister.com/2024/03/15/on_call/](https://www.theregister.com/2024/03/15/on_call/)

On Call Thirsty folk will tell you it's always twelve o'clock somewhere, but Friday comes but once a week and *The Register* marks it by offering another instalment of On Call, our weekly reader-contributed column that shares your stories of the weird and woeful world of work.

This week, meet a reader we'll Regomize as "Aaron" who very recently worked for a managed services provider in Australia. One of his clients was an ad agency that he described as "insanely flashy and demanding."

"The owner has two Ferraris. They have that kind of money," Aaron told On Call.

The agency also had an all-Apple environment – other than a SharePoint server that was used to store, among other things, plenty of uncompressed video and audio.

SharePoint choked on those files, and users wanted faster access. So the agency built a file server to host that in-demand data.

Except it wasn't really a server. It was a Mac Mini with some external USB drives. Data from SharePoint was stored on those drives in directories set up as SMB shares.

This arrangement was obviously unstable, and the server would become unavailable if client devices updated macOS before the server. So every time Apple issued a new OS, Aaron rushed to update the server before the mismatch meant the ad agency lost access to its files.

"This was so common that we ended up creating an entire ticket stream just to monitor for macOS update emails as soon as Apple sent them," Aaron told On Call. But even with that early warning system in place, downtime was regular.

The Ferrari-owning boss got wind of this mess and would berate Aaron and his crew whenever the server became unavailable, citing the enormous cost of outages.

Aaron countered by pointing out that a real file server would make the problem go away.

"The bosses at my MSP explicitly told everyone to remind the client of this because they were personally sick of the problem," Aaron told On Call.

But the agency boss wouldn't buy a server, so Aaron and his colleagues lived in a horrible cycle of trying to keep the server updated, falling behind, fixing it, being berated by their rich client, recommending he pay for a new server, and being told "No."

Until the day Aaron updated the Mac Mini, and the SharePoint directories on the external drives disappeared.

"I had to go on site, and try as I might the damn thing wouldn't read," Aaron recalled. "We could see the hardware ID for the external drives. But they would not mount."

After an hour without finding a fix, Aaron called a colleague who also struggled.

The two sweated it out – literally, because this incident took place on a hot Australian summer day in an airless office without air conditioning – until they figured out the fresh macOS update wasn't loading USB drivers and therefore wouldn't talk to the external disks.

Reinstalling the drivers did not work … until Aaron and his sidekick reinstalled macOS and the external drives came alive.

After three hours of downtime.

A couple of months later, the ad agency agreed to buy a new file server. A real one. Aaron built the box with a glad heart.

But the agency never paid for it.

Sick of the cycle of rushed repairs and recriminations, Aaron quit a few months later.

"And that file server was still sitting next to my desk, unpaid for, on the day I left" he told On Call. "Lord have mercy on whoever has to deal with their bullshit now."

What's the cheapest thing you've seen a boss do that made needless work for IT? [Click here to send On Call your story](mailto:oncall@theregister.com) and we may feature it on a future Friday. ®