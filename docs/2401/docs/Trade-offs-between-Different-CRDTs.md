<!--yml
category: 未分类
date: 2024-05-27 14:36:01
-->

# Trade-offs between Different CRDTs

> 来源：[https://interjectedfuture.com/trade-offs-between-different-crdts/](https://interjectedfuture.com/trade-offs-between-different-crdts/)

What are the trade-offs between different kinds of CRDTs (Conflict-free Replicated Data Types)? Most introductory talks cover just state-based and operations-based CRDTs because that's what the original paper formulated. But since then, there have been other variations, and I haven't seen much written about them in blog posts, so I'll cover their trade-offs here.

The basics of [CRDTs](https://crdt.tech/) are covered in a lot of other different places [^1], so I won't try to speed run it here. Let's just jump into the differences.

## State-based CRDTs (Convergent CRDTs)

In State-based CRDTs, a replica's state can be updated by a merge operation with another replica's state. But if the [network is unreliable and can deliver events out of order](https://architecturenotes.co/fallacies-of-distributed-systems/), how would all the replicas achieve convergence and agreement of all their states? The trick is simply to restrict ourselves to data structures and operations that are immune to net-splits and out-of-order delivery.

First, the internal data structure of the CRDT must be monotonically increasing. Second, the merge operation has to be commutative, associative, and idempotent with respect to the internal data structures.

These constraints make the CRDT immune to the unreliable network. As long as all replicas eventually see every state update event, they're guaranteed to converge to the same state. Due to the use of vector clocks, replicas coming in and out of the network are easily handled. The internal bookkeeping grows linearly with the number of replicas.

The rest of the design problem is how to build useful data structures out of only monotonically increasing elements. Luckily, there is already a [menagerie of CRDTs](https://github.com/pfrazee/crdt_notes/tree/68c5fe81ade109446a9f4c24e03290ec5493031f#portfolio-of-basic-crdts) that can model numbers, arrays, maps, strings, and other common data structures.

State-based CRDTs are theoretically sound, but in practice, it has a glaring downside: they require sending the entire state of a replica over the network to other replicas. This can be prohibitive for all but the smallest states like counters and registers. That's because a CRDT needs to maintain internal bookkeeping of "which replica said what" at some logical time in order to resolve conflicts for any new state changes. The end-user queries this internal bookkeeping for a current value, like a functional view calculated from the actual state, the internal bookkeeping. For anything beyond a counter or register, this internal bookkeeping can be too big for sets and maps. And as the number of replicas grows, it can grow too big too quickly.

Therefore, most current CRDT implementations are not state-based but op-based.

💡

- Uses merge function to converge
- Merge function needs to be commutative, associative, and idempotent
- Internal data structure needs to be monotonically increasing

👍

- Easy syncing protocol that just broadcasts new state to other replicas
- Does not need to keep a history to sync (as we'll see later)

👎

- Sending the entire state over the wire is impractical for all but the simplest CRDTs
- Internal data structure grows linearly with the number of replicas
- Harder to accommodate replicas that come in and out of the network

## Operation-based CRDTs (Commutative CRDTs)

Current practical implementations of CRDTs opt for a way to ship smaller pieces of data over the wire while retaining the same properties as a state-based CRDT. We will give up a merge function and instead use a defined set of operations that can be used on the state. It's akin to defining [a command in an effect system](https://guide.elm-lang.org/effects/). Like before, the operations will need to be constrained to give us the property of uncoordinated syncing between replicas. The operations will need to be both commutative and associative, but unlike the merge function, they do not have to be idempotent.

Without idempotency, the syncing protocol now cannot just be an out-of-order delivery of updates like in state-based CRDTs. Instead, we require the operations to be delivered in causal order. When a replica syncs and catches up to the latest state, the operations are applied in causal order. Any concurrent operations will result in the same state, due to the commutative and associative properties of the operations. We're shifting part of the complexity of the merge function into the sync protocol.

Our internal bookkeeping to converge to the same state with concurrent operations doesn't use vector clocks. Instead, we can keep a log of the causal history of all the operations that occurred. Our internal bookkeeping no longer grows with the number of replicas, but instead with the number of events.

Op-based CRDTs can scale easily with the number of replicas and with an unknown number of replicas that come in and out of the network easily. However, like blockchains, it has an ever-growing internal bookkeeping if it wants to allow any replica to sync regardless of how long it has been offline.

There are two ways to address this ever-growing bookkeeping. One or the other may be feasible depending on the requirements of your application.

The first is to adopt what accountants do: close books at the end of every month and quarter. All replicas keep a rolling window of the latest N days of data and throw away data that's older than N days. And if a replica has been offline longer than a certain number of days, then they cannot be expected to sync and will need to reload from the latest.

The second is to keep all the history, but find ways to compress it. This is what the CRDT library [Automerge does](https://youtu.be/x7drE24geUw?si=syBk3NTxeDQekk30&t=3201). This can sound like a bad idea, but we already have software that keeps its entire history around that developers use every day: [Git](https://git-scm.com/book/en/v2/Git-Internals-Git-Objects). Of course, not all application requirements allow us to do this, but in my opinion, we don't do enough of it. Disks are comparatively cheap now, and the [benefits of immutability](https://interjectedfuture.com/persistent-data-structure-redux/) far outweigh the downsides.

To query for the value of an op-based CRDT, we can apply all operations from the history log to an empty state to get the current state. Alternatively, we can keep a cache of the latest state in memory and update it on every operation.

💡

- Applies operations to converge state
- Operations need to be commutative and associative
- Internal data structure needs to be monotonically increasing

👍

- Small amount of data sent over the network to sync
- Easier to accommodate replicas that come in and out of the network
- Data is immutable, hence diffs, undos, and comparisons are easy.

👎

- Syncing protocol needs to implement causal delivery of operations
- Internal data structure is a history of operations that grows linearly with the number of operations

## Delta-state CRDTs

Delta-state CRDTs try to solve the data-over-the-wire problem differently. Instead of breaking the merge function into operations to send over the wire, we try to calculate the minimum amount of state to send to other replicas to sync.

There are two versions of delta-state CRDTs. In the [first δ-state CRDT paper](https://arxiv.org/pdf/1410.2803.pdf), we do something similar to op-based CRDTs: we create a set of operations, called delta-mutators) that are used to update the state. But instead of sending these operations over the wire, delta-mutators generate a diff between the states before the delta-mutator was applied and after it was applied. The diff is then stored in a buffer to be sent out to all other replicas.

But this buffer isn't a queue. The buffer holds the diff between the current state and the state when the buffer was last sent to other replicas. This is possible because the delta-mutator diffs are composable. This means if we have two diffs, d[12] (a diff between State X[1] and State X[2]) and d[23] (a diff between State X[2] and State X[3]), the composition of the two diffs would be D[13] (computed with d[12] ⨆ d[23]). Hence, the application of d[12] and then d[23] is the same as the application of the composition D[13]. The paper calls these compositions, delta-groups.

And because the diffs exist in the same join-semilattice as the states, you can use the same merge function to merge the diffs received from other replicas into your own local state.

The [second Δ-state paper](https://web.archive.org/web/20230607175939/https://novasys.di.fct.unl.pt/~alinde/publications/a12-van_der_linde.pdf) takes a slightly different approach. While the 𝛿-state CRDT has a replica send the same diff to all other replicas, in the Δ-state CRDT, we notice that the information about which replica has which part of the state is already encoded in the vector clock internal bookkeeping of the CRDTs. Hence, we can tailor the diff that we send to each replica. Therefore, we can toss the buffer and calculate the exact diff of the state that another replica needs. But by trading off the buffer, you now need to keep track of tombstones.

Delta-CRDTs retain the idempotency property of state-based CRDTs but do not require lots of bandwidth to sync replicas.

💡

- Merge either full state or deltas to converge
- Merge function needs to be commutative, associative, and idempotent
- Internal data structure needs to be monotonically increasing

👍

- Small amount of data sent over the network to sync
- Syncing protocol is simple and just needs to broadcast to all other replicas

👎

- Internal data structure grows with the number of replicas

- Internal data structure can get complex, like using

[Dots](https://www.bartoszsypytkowski.com/optimizing-state-based-crdts-part-2/)

.

- Merge function implementation can get complex.

## The Search Continues

While all of these CRDTs have strengths and weaknesses, I'm looking for something where I don't have to compromise. Next time, I'll cover my search into Merkle CRDTs. In the meantime, [follow me on twitter](https://twitter.com/iamwil).

* * *

[^1]: Here's some introductory material on CRDTs I found helpful
- [An interactive intro to CRDTs](https://jakelazaroff.com/words/an-interactive-intro-to-crdts/)
- [An introduction to state-based CRDTs](https://www.bartoszsypytkowski.com/the-state-of-a-state-based-crdts/)
- [CRDTs for non-academics](https://www.youtube.com/watch?v=vBU70EjwGfw)
- [CRDT: The Hard Parts](https://www.youtube.com/watch?v=x7drE24geUw)
- [Readings in CRDTs](https://christophermeiklejohn.com/crdt/2014/07/22/readings-in-crdts.html)
- [crdt.tech](https://crdt.tech)