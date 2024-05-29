<!--yml
category: 未分类
date: 2024-05-29 12:36:36
-->

# Port-scanning the fleet and trying to put out fires

> 来源：[https://rachelbythebay.com/w/2024/03/21/scan/](https://rachelbythebay.com/w/2024/03/21/scan/)

## Port-scanning the fleet and trying to put out fires

There was this team which was running a pretty complicated data storage, leader election and "discovery" service. They had something like 3200 machines and had something like 300 different clusters/cells/ensembles/...(*) running across them. This service ran something kind of like etcd, only not that.

The way it worked was that a bunch of "participant" machines would start an election process, and then they'd decide who was going to lead them for a while. That leader got to handle all of the write traffic and it did all of the usual raft/paxos-ish spooky coordination stuff amongst the participants, including updating the others, and dealing with hosts that go away and come back later, and so on. It's all table stakes for this kind of service.

This group of clusters had started out relatively simple but had grown into a monster over the years. Nobody probably expected them to have hundreds of clusters and thousands of machines, but they now did, and they were having trouble keeping track of everything. There were constant outages, and since they were so low in the stack, when they broke, lots of other stuff broke.

I wanted to know just what the ground truth looked like and so started something really stupid from my development machine. It would take a list of their servers and would crawl them, interrogating the TCP ports on which the service ran. This was only about 10 ports per machine, so while it sounded obnoxiously high, it was still possible for prototyping purposes.

On these ports, there were simple text-based commands which could be sent, and it would return config information about what that particular instance was running. It was possible to derive the identity of the cluster from that. Given all of this and a scrape of the entire fleet, it was possible to see which host+port combinations were actually supporting any given cluster, and thus see how well they were doing.

Early results from this terrible manual scraping started showing promise. Misconfigurations were showing up all over the place - clusters that are supposed to have 5 hosts but only have 3 in practice with the other two missing in action somewhere, clusters with non-standard host counts, clusters in the wrong spots, and so on.

To get away from the "printf | nc the world in cron" thing, we wound up writing this dumb little agent thing that would run on all of the ~3200 hosts. It would do the same crawling, but it would happen over loopback so it was a good bit faster by removing long hauls over the production network from the equation. It also took the load of polling ~32000 ports off my singular machine, and was inherently parallel.

It was now possible to just query an agent and get a list of everything running on that box. It would refresh things every minute, so it was far more current than my terrible script which might run every couple of hours (since it was so slow). This made things even better, and so we needed an aggregator.

We did some magic to make each of these agents create a little "beacon" somewhere any time they were run. Our "aggregator" process would start up and would subscribe to the spot where beacons were being created. It would then schedule the associated host for checks, where it would speak to the agent on that host and ask for a copy of its results.

So now we had an agent on every one of the ~3200 hosts, each polling 10 local ports, plus an aggregator that talked to the ~3200 agents and refreshed the data from them.

Finally, all of the data was available in one place with a single query that was really fast. The next step was to write a bunch of simple "dashboard" web pages which allowed anyone to look at the entire fleet, or to narrow it down by certain parameters - a given cluster (of these servers), a given region, data center, whatever.

With all of this visible with just a few clicks, it was pretty clear that we needed something more to actually find the badness for us. It was all well and good to go clicking around while knowing what things are supposed to look like, but there were supposed to be rules about this sort of thing: this many hosts in a cluster, no more than N hosts per failure domain, and more.

...

Failure domains are a funny thing. Let's say you have five hosts which form a quorum and which are supposed to be high-availability. You'd probably want to spread them around, right? If they were serving clients from around the world, maybe you'd put them in different locations and never put two in the same spot? If something violated that, how would you know?

Here's an example of bad placement. We had this one cluster which was supposed to be spread out throughout an entire region which was composed of multiple datacenter buildings, each with multiple (compute) clusters in it, with different racks and so on down the line. But, because it had been turned up early in the life of that region when only a handful of hosts had existed, all of them were in the same two or three racks.

Worse still, those racks were physically adjacent. Put another way, if the servers had arms and hands, they could have high-fived each other across the hot and cold aisles in the datacenter suite. That's how close together they were. One bad event in a certain spot would have wiped out all of their data.

We had to write a schema which would let us express limits for a given cluster - how many regions it should be in, the maximum number of members per host, rack, (compute) cluster, building, region, etc. Then we wrote a tool to let us create rules, and then started using that to churn out rulesets. Next we came up with some tools which would fetch the current state of affairs (from the agent/aggr combo) and compare it to the rulesets. Anything out of "compliance" would show up right away.

...

Then there was the problem of managing the actual ~3200 hosts. With a footprint that big, there's always something happening. A location gets turned up and new hosts appear. Another location is taken down after the machines get too old and those hosts go away. We kept having outages where a decom would be scheduled, and then someone far away would run a script with a bunch of --force type commands, and it would just yank the machines and wipe them. It had no regard for what they were actually doing, and they managed to clobber a bunch of stuff this way. It just kept happening.

This is when I had to do something that does not scale. I said to the decom crew that they should treat any host owned by this team as off limits because we do not have things under control. That means never *ever* running a decom script against these hosts while they are still owned by the team.

I further added that while we're working to get things under control, if for some reason a decom is blocked due to this decree of mine, they are to contact me, any time of day or night, and I will get them unblocked... somehow. I figured it was my way of showing that I had "skin in the game" for making such a stupid and unreasonable demand.

I've often said that the way to get something fixed is to make sure someone is in the path of the badness so they will feel it when something screws up. This was my way of doing exactly that.

We stopped having decom-related outages. We instead started having these "fire drill" type events where one or two people on the team (and me) would have to drop what they were doing and spend a few hours manually replacing machines in various clusters to free them up.

Obviously, this couldn't stand, and so we started in on another project. This one was more of a "fleet manager", where a dumb little service would keep track of which machines the team owned, and it would store a series of bits for each one that I called "intents".

There were only three bits per host: drain, release, freeze. Not all combinations were valid.

If no bits were set on a host, that meant it was intended for production use. If it has a server on it, that's fine. If someone needs a replacement, it's potentially available (assuming it meets the other requirements, like being far enough away from the other participants).

If the "drain" bit was set, that meant it was not supposed to be serving. Any server on it should be taken off by replacing it with an available host which itself isn't marked for "drain" (or worse).

The "release" bit meant that if a host no longer had anything running on it, then it should be released back to the machine provisioning system. In doing this, the name of the machine changed, and thus the ownership (and responsibility) for it left the team, and it was no longer our problem. The people doing decoms would take it from there.

"Freeze" was a special bit which was intended as a safety mechanism to stop a runaway automation system. If that bit was set on a host, none of the tools would change anything on it. It's one of those things where you should never need to use it, but you'll be sorry if you don't write it and then need it some day.

"Drain" + "release" meant "keep trying to kick instances off this host and don't add any new ones", and then "once it becomes empty, give it back".

Other combinations of the bits (like "release" without "drain") were invalid and were rejected by the automation.

I should note that this was meant to be level-triggered, meaning on every single pass, if a host had a bit set and yet wasn't matching up with that intent or those intents, something should try to drain it, or give it away, or whatever. Even if it failed, it should try again on the next pass, and failures should be unusual and thus reported to the humans.

...

Then there was also the pre-existing system which took config files and used it to install instances on machines. This system worked just fine, but it only did that part of the process. It didn't close the loop and so many parts of the service lifecycle wound up unmanaged by it.

Looking back at this, you can now see that we could establish a bunch of "sets" with the data available.

Configs: "where we told it to run"

Agent + aggregator: "where it's actually managing to run"

Checker: "what rules these things should be obeying"

Fleet manager: "which machines should be serving (or not), which machines we should hang onto (or give back)".

Doing different operations on those sets yielded different things.

[configs] x [agent/aggr] = hosts which are doing what they are supposed to be doing, hosts which are supposed to be serving but aren't for some reason, and hosts which are NOT supposed to be running but are running it anyway. It would find sick machines, failures in the config system, weird hand-installed hack jobs in dark corners, and worse.

[agent/aggr] x [checker] = clusters which are actually spread out correctly, and clusters which are actually spread out incorrectly, (possibly because of bad configs, but could be any reason).

[agent/aggr] x [fleet manager] = hosts which are serving where that's okay, hosts which need to be drained until empty, and hosts which are now empty and can be given back.

[configs] x [checker] = are out-of-spec clusters due to the configs telling them to be in the wrong spot, or is something else going on? You don't really need to do this one, since if the first one checks out, then you know that everything is running exactly what it was told to run.

[configs] x [fleet manager] = if you ever get to a point where you completely trust that the configs are being implemented by the machines (because some other set operations are clear), then you could find mismatches this way. You wouldn't necessarily have to resort to the empirical data, and indeed, could stop scanning for it.

For that matter, the whole port-scanning agent/aggr combination shouldn't have needed to exist in theory, but in practice, independent verification was needed.

I should point out that my engagement with this team was not viewed kindly by management, and my reports about what had been going on ultimately got me in trouble more than anything else. It's kind of amazing, considering I was working with them as a result of a direct request for reliability help, but shooting the messenger is nothing new. This engagement taught me that a lot of so-called technical problems are in fact rooted in human issues, and those usually come from management.

There's more that happened as part of this whole process, but this post has gotten long enough.

...

(*) I'm using "clusters" here to primarily refer to the groups of 5, 7, or 9 hosts which participated in a quorum and kept the state of the world in sync. Note that there's also the notion of a "compute cluster", which is just a much larger group of perhaps tens of thousands of machines (all with various owners), and that does show up in this post in a couple of places, and is called out explicitly when it does.