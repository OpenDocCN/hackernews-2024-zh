<!--yml
category: 未分类
date: 2024-05-27 13:17:26
-->

# What makes concurrency so hard? • Buttondown

> 来源：[https://buttondown.email/hillelwayne/archive/what-makes-concurrency-so-hard/](https://buttondown.email/hillelwayne/archive/what-makes-concurrency-so-hard/)

<date>April 16, 2024</date>

# What makes concurrency so hard?

## Is it something about human brains, or something about the problem domain?

A lot of my formal specification projects involve concurrent or distributed system. That's in the sweet spot of "difficult to get right" and "severe costs to getting it wrong" that leads to people spending time and money on writing specifications. Given its relevance to my job, I spend an awful lot of time thinking about the nature of concurrency.

As the old joke goes, concurrency one of the two hardest things in computer science. There are lots of "accidental" reasons why: it's hard to test, it's [not composable](https://stefan-marr.de/2014/07/why-is-concurrent-programming-hard/), bugs can stay latent for a long time, etc. Is there anything that makes it *essentially* hard? Something that makes concurrent software, by its very nature, more difficult to write than synchronous software?

The reason I hear most often is that humans think linearly, not concurrently, so are ill-equipped to reason about race conditions. I disagree: in my experience, humans are *very* good at concurrent reasoning. We do concurrent reasoning every time we drive a car!

More generally, some studies find that if you frame [concurrent systems in human terms](https://citeseerx.ist.psu.edu/document?repid=rep1&type=pdf&doi=77c3ef40423deeed3c73647df93f6f0ecdab2d69) ("[meatspace modeling](https://buttondown.email/hillelwayne/archive/i-am-a-sql-injection-attack-5019/)"), people get quite good at finding the race conditions. So while concurrency might be difficult to reason about, I don't think it's because of a fault in our brains.

In my opinion, a better basis is **state space explosion**. Concurrency is hard because concurrent systems can be in a lot of different possible states, and the number of states grows much faster than anyone is prepared for.

### Behaviors, Interleavings, and Nondeterminism

Take agents ^({1, 2, … n}, which each executes a linear sequence of steps. Different agents may have different algorithms. Think writing or reading from a queue, incrementing a counter, anything like that. The first process takes p1 atomic steps to complete, the second p2, etc. Agents complete their programs strictly linearly, but another process can interleave after every atomic step. If we have algorithms A1A2 and B1B2, they could execute as A1B1A2B2 or A1B1B2A2, but not A1**B2B1**A2.)

Under those conditions, here's an equation for how many possible orderings of execution (**behaviors**) can happen:

```
`(p1+p2+...)!
------------
p1!*p2!*...` 
```

If we have three agents all executing the same 2-step algorithm, that's `6!/8 = 90` distinct behaviors. Each step in the sequence can potentially lead to a distinct state, so there's 6*90=540 maximum distinct states (MDS). Any one of those can potentially be the "buggy" one.

In most cases, the actual state space will be significantly smaller than the MDS, as different behaviors will cross the same states. Counterbalancing this is just how *fast* this formula grows. Three 3-step agents gives us 1700 possible behaviors (15K MDS), four 2-step agents instead have 2500 (20K MDS). And this is all without any kind of nondeterminism! If one step in one of the agents can nondeterministically do one of three things (send message M₁, send message M₂, crash), that triples the number of behaviors and the MDS.

It's pretty common for complex concurrent systems to have millions or tens of millions of states. Even the toy model I use in [this article on model optimization](https://learntla.com/topics/optimization.html) has about 8 million distinct states (though with some work you can get it much lower). I mostly think about state spaces in terms of performance because large state spaces take a lot longer to model-check. But it's also why concurrency is essentially hard to reason about. If my theoretical MDS is two million states, my practical state space is just 1% that size, and my human brain can reason through 99.9% of the remaining states… that still leaves 20 edge cases I missed.

### Shrinking the State Space

Here's a heuristic I use a lot:

> **All means of making concurrency 'easier' are concerned first and foremost with managing the state space.**

It's not 100% true (no heuristic is) but it's like 60% true, and that's good enough. State spaces grow quickly and bigger state spaces cause problems. If we want to make maintainable concurrent systems, we need to start by shrinking the space.

Like look at threads. Threads share memory, and the thread scheduler has a lot of freedom to suspend threads. So you have lots of steps (potentially one per line of code) ^(and any interleaving can lead to a distinct state. I can use programming constructs like mutexes and barriers to "prune" the state space and give me the behaviors I want, but given how big the state space can be, I have to do a lot of pruning to get the right behaviors. I can make mistakes in implementation, "misshape" the space (like by adding a deadlock), or not notice a buggy state I need to remove. Threads are very error prone.)

I could instead switch to memory-isolated processes. The scheduler can still schedule whatever interleavings it wants but the processes can't muck with each other's internal memory. Internally a process is still executing A1A2A3A4, but if the only steps that involve external resources (or interprocess communication) are A2 and A4, then we only need to think about how A2A4 affects the state space. Three four-step agents have 35k interleavings, three two-step agents have only 90\. That's a big improvement!

What else can we do? Low-level atomic instructions do more in a single step, so there's no room for interleaving. Database transactions take a lot of physical time but represent only one [logical time step](https://buttondown.email/hillelwayne/archive/physical-vs-logical-time/). Data mutations create new steps, which immutable data structures avoid by definition.

Languages have constructs to better prune the resulting state space: go's channels, promises/futures/async-await, nurseries, etc. I think you can also treat promises as a way of forcing "noninterleaving": wait until a future is ready to execute in full (or to the next yield point) and before execution. *Please* don't quote me on this.

I *think* [CRDTs](https://en.wikipedia.org/wiki/Conflict-free_replicated_data_type) reduce state space by permitting interleavings, but arranging things so that external changes are commutative: A1B1 and B1A1 lead to the same final result, so there are not distinct states.

Again, this is all a *very* rough heuristic.

### Limits

To a first-order approximation, smaller state space == good. But this doesn't account for the topology of the space: some spaces are gnarlier than others. One that has lots of different cycles will be harder to work with than one that's acyclic. ^(Different forms of nondeterminism also matter: "the agent continues or restarts" leads to more complex behavior than "the writer sends message M₁ or M₂." These are both places where "humans are bad at reasoning about concurrency" appears again. We often work through concurrency by reduces groups of states to "state equivalence classes" that all have the same properties. complicated state spaces have more equivalence classes to work through.)

(Another way "humans are bad at reasoning about concurrency" can be a real thing: we might not *notice* that something is nonatomic.)

Some high-level paradigms can lead to particular state space topologies that have fewer interleavings or ones that have more equivalent states. I've heard people claim that [fork-join](https://en.wikipedia.org/wiki/Fork%E2%80%93join_model) and pipe-filter are especially easy for humans to reason about, which I take to mean "doesn't lead to a gnarly state space". Maybe also event-loops? Where does the actor model fit into all of this?

Another limit: there's a difference between buggy *states* and buggy *behaviors*. Some behaviors can go entirely through safe states but still cause an bug like "never reaches consistency". This is called a "liveness bug", and I talk more about them [here](https://www.hillelwayne.com/post/safety-and-liveness/). Liveness bugs are much harder to reason about.

Okay, that's enough philosophical rambling about concurrency. Concurrency is hard, don't feel bad if you struggle with it, it's not you, it's combinatorics.

* * *

### Video Appearance

I was on David Giard's *Technology and Friends* talking about TLA+. Check it out [here](https://www.youtube.com/watch?v=DERER_0RKGws)!

*If you're reading this on the web, you can subscribe [here](/hillelwayne). Updates are once a week. My main website is [here](https://www.hillelwayne.com).*