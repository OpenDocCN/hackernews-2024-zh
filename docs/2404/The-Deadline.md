<!--yml
category: 未分类
date: 2024-05-27 13:30:21
-->

# The Deadline

> 来源：[https://www.thomas-huehn.com/2014/12/the-deadline/](https://www.thomas-huehn.com/2014/12/the-deadline/)

**The Deadline** by **Tom DeMarco** is a real project management classic. I had bought it quite some time ago, but it collected dust on the shelf, despite the fact that I had enjoyed his book “Peopleware”. Turns out, it was a mistake. The collecting dust, not the reading, mind you.

It’s a software project management book disguised as a novel. And, while it is pleasant to read, even funny at times, as a novel it’s not worth a lot. But this clever packaging makes it so much easier to plow through.

Our protagonist, a project manager that recently lost his job because he spoke his mind, is drugged and kidnapped by a charming young woman who is “acquiring” talent for the up-and-coming software industry of Monrovia, a fictional country of the former Eastern Bloc.

They’ve got lots and lots of software developers, some managers, and the desire to become the biggest software supplier of the world. Six software products have already been planned (all rip-offs of well-selling programs like Photoshop), and now it’s time to execute on that vision.

Additionally, our protagonist and his trusted senior staff intend to conduct some real-world experiments: having three different teams each for those six products, so that different approaches can be compared and quantified.

The novel introduces a problem (like personal conflicts between developers or tight deadlines) in each chapter, usually with a new character who is relevant to the issue at hand. This new character is often a world-renowned expert on this field, is flown in (or visited) and gives key insight in an afternoon. Obviously, this repeating theme is one of the reasons why the book falls short as a novel per se.

The reader needs to be acutely aware that those experts are no real-world experts, but are channelling Tom DeMarco’s convictions. That’s perfectly okay, since it’s Tom DeMarco’s presentation, but the reader must not get confused, for the characters in the book take every word of those experts as gospel, never questioning it, never finding tensions or direct contradictions to other experts’ views. The reader must not get complacent and do the same.

Every chapter ends with a few short notes on what our protagonist has learned. And there are quite a few real nuggets in there. They are never too detailed, more food-for-thought. I like that because it saves the book from becoming the chore most project management books are. Remember, no proofs for the validity are given, this is straight advice by the author. Take it or leave it.

While most of those points were pretty obvious, sometimes even trivial, I’d like to repeat some others here. Some of them are important and had at least some quality of being new to me. Some were nothing new, but spoke to me because I have personal recollections pertinent to them. Others just elicited some thoughts of my own that I don’t want to lose. So instead of re-reading the book later, I hope those notes can serve as a reminder of the meat of it, as I saw it.

## Anonymous confessions

One of the early lessons is that there should be an anonymous way to report problems up the hierarchy. In the novel it is presented as a real confessional box, complete with the ceremony based on the catholic shrift.

Of course, this mechanism is not limited to reporting own failures and problems. Actually, it’s more likely that it will lead to reports about problems in areas the reporter doesn’t have under his control, I think.

One interesting twist is that the confessor in the novel always knows very well the identity of the penitent (realistically, I think, there are only so many people with both the knowledge of some subject and no better way to report it), but never lets it show. That is probably important, because as soon as the higher-up breaks this illusion of anonymity reports will dry up.

## Risk officer

Risk management in the project is extremely important. While everyone is tasked with managing risks, there should be a designated risk officer. Furthermore, this risk officer (aided by the team, of course) must identify early indicators of important risks well before they can materialize, and then be on a constant lookout for those indicators.

## Dangers of Can-Do

While many managers see a can-do attitude as a positive trait, it poses the danger of priming the team for averting to report risks and problems up the chain of command.

## Improving productivity

There are no opportunities for short-term improvements of productivity, because every sane team member handles those low-hanging fruit on his own, in order to eliminate sources of frustration.

This may be an exaggeration, I think, because sometimes management can help making it okay to eliminate those time-killers. For example, a boss once instituted the policy that team members were explicitly allowed to declare times of the day where they don’t respond to the telephone (and announce those times), as well as put up a do-not-disturb sign at their desk in the open-plan office. The effects weren’t too big, in my recollection, mostly because those were inadequate means of addressing the root cause.

## Modelling Hunches

In a very interesting early chapter the senior management of Monrovia’s budding new software industry explicitly model their “hunches” or gut feelings about how much of an effect training and personnel fluctuations have on the work being completed. This means not just drawing diagrams, but actually using a modelling tool on the computer and quantifying the internal state transitions.

This serves not only as a tool to communicate one’s hunches to other members of the management team, and consequently to compare and refine those hunches. It also allows to plug in real-life measurements, thus improving one’s understanding of the effects at work.

That was probably the single most surprising lesson in the book for me. I have lingering doubts as to its practicality, not only the quantitative part, but also the ability to conjure up a semi-realistic model in the first place, but it looks like there are actually tools for that purpose being sold and used.

## Liking people

Actually liking people you’re working with (or especially “against”), not only faking it, makes it easier to find common ground and convince them of whatever you dearly need.

## Pressure

Pressure is often used as a way to elicit more work results and even more productivity. It is not a viable means to do that. But pressure, applied judiciously and for not too long, can signal heightened importance of some piece of work to the team. It is therefore not bad by itself, but it is usually applied in a destructive way (non-focussed: too much for too long).

Additionally, people use pressure towards their subordinates in order to demonstrate to their higher-ups that they have done everything in their power to achieve the (probably missed) goal.

## Inner doubts

Most people have some inner doubts about their abilities or intelligence. That is why they don’t dare to speak up when some document is incomprehensible, because everyone else seems to understand it perfectly well. In reality, everyone is likely to have similar thoughts, but this effect is self-feeding, and so the problem is concealed.

## Specification

The minimum requirement for a document to be considered a specification is having both of these:

*   Policy: How does the system react to events? This part is complex.
*   Inputs and outputs. These can be defined succinctly and precisely.

Personally I would add at least a third part: Data and (inner) state. Starting with data structures and only then defining the operations on them is usually advantageous.

## Ambiguities conceal conflicts

Ambiguous wording in specifications is usually there because some unresolved conflict between stakeholders is brewing and the author cannot be clear and precise, because that would decide the ongoing conflict.

That’s an insight that was new to me. Still, I think that lots of ambiguities in specifications are not because of conflicts without a resolution, but because of missing information. Why can’t the author just ask around until he knows the answer? Often because decisions pertinent to the issue haven’t been made, yet, and are some other group’s responsibility. In a way you can call it a scheduling conflict, I suppose, but I wouldn’t really sort it under the heading “conflict”.

## Staffing

Start the project with few people and do a proper design. No more than a handful of team members can contribute in a meaningful way because in that phase everyone needs to have a view of the big picture, no specialization is possible.

When the project enters phases where well-defined work packages can be worked on individually (e.g. coding), massively expand the team.

As a latecomer in a former project of mine, colleagues told me how it was in the beginning. The small design team was furiously cranking out specifications for the (dozen or so) fully staffed teams to work on, but they were hopelessly swamped, of course. So the teams mostly sat on their hands.

On the other hand, I really lamented that I wasn’t around in the beginning. So many decisions were not only ingrained into the design (and the source code!), but also a kind of folklore. The original design team had moved on, and on some subjects nobody in the company knew anymore why certain directions were taken. No one left to ask.

## Anger = fear

In a business context noone shows fear because he would lose face. Anger is a semi-accepted substitute emotion, so people lash out when afraid.

But when everyone knows about this substitution, this same tendency to prevent loss of face will stifle angry outbursts.