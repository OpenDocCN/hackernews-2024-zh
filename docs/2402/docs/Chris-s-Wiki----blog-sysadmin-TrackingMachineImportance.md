<!--yml
category: 未分类
date: 2024-05-27 14:38:51
-->

# Chris's Wiki :: blog/sysadmin/TrackingMachineImportance

> 来源：[https://utcc.utoronto.ca/~cks/space/blog/sysadmin/TrackingMachineImportance](https://utcc.utoronto.ca/~cks/space/blog/sysadmin/TrackingMachineImportance)

Today [we had a significant machine room air conditioning failure in our main machine room](https://mastodon.social/@cks/111881322231361439), one that certainly couldn't be fixed on the spot ('glycol all over the roof' is not a phrase you really want to hear about your AC's chiller). To keep the machine room's temperature down, we had to power off as many machines as possible without too badly affecting the services we offer to people here, [which are rather varied](/~cks/space/blog/sysadmin/OurDifferentSysadminEnvironment). Some choices were obvious; all of [our SLURM nodes](/~cks/space/blog/sysadmin/SlurmHowWeUseIt) that were in the main machine room got turned off right away. But others weren't things we necessarily remembered right away or we weren't clear if they were safe to turn off and what effects it would have. In the end we took several rounds of turning servers off, looking at what was left, spotting remaining machines, and turning more things off, and we're probably not done yet.

(We have secondary machine room space and we're probably going to have to evacuate servers into it, too.)

One thing we could do to avoid this flailing in the future is to explicitly (try to) keep track of which machines are important and which ones aren't, to pre-plan which machines we could shut down if we had a limited amount of cooling or power. If we documented this, we could avoid having to wrack our brains at the last minute and worry about dependencies or uses that we'd forgotten. Of course documentation isn't free; there's an ongoing amount of work to write it and keep it up to date. But possibly we could do this work as part of deploying machines or changing their configurations.

(This would also help identify machines that we didn't need any more but hadn't gotten around to taking out of service, which we found a couple of in this iteration.)

Writing all of this just in case of further AC failures is probably not all that great a choice of where to spend our time. But writing down this sort of thing can often help to clarify how your environment is connected together in general, including things like what will probably break or have problems if a specific machine (or service) is out, and perhaps which people depend on what service. This can be valuable information in general. The machine room archaeology of 'what is this machine, why is it on, and who is using it' can be fun occasionally, but you probably don't want to do it regularly.

(Will we actually do this? I suspect not. When we deploy and start using a machine its purpose and so on feel obvious, because we have all of the context.)