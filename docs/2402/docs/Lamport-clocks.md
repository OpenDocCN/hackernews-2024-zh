<!--yml
category: 未分类
date: 2024-05-27 14:44:15
-->

# Lamport clocks

> 来源：[https://blog.fponzi.me/2024-02-02-lamport-clocks.html](https://blog.fponzi.me/2024-02-02-lamport-clocks.html)

# Lamport clocks

Published on 2024-02-02 | Last update: 2024-05-23 | 12 min read

<details class="toc-container" open=""><summary>Table of Contents</summary></details>

Last weekend I wanted to get some inspiration to write some TLA+ spec and I got my hands back on the paper [Time, Clocks, and the Ordering of Events in a Distributed System by Leslie Lamport](http://lamport.azurewebsites.net/pubs/time-clocks.pdf).

This is the most cited paper by Lamport. It introduces the concept of "happened before" and **Logical Clock**, often times referred to as *Lamport clock*, and gives an example on how to use them to solve the distributed mutual exclusion problem.

This post is a quick summary of the paper and has a walk-through of the specification I've written based on the mutual exclusion algorithm described in the paper. While writing the spec for this post, I've also [contributed](https://github.com/hwayne/learntla-v2/pull/78) with some documentation to the learntla website.

## What problem are they trying to solve?

An "event" could be created after something important happened in our system, e.g., some instruction is executed. In our distributed system, assume we have a central authority that wants to know which of two events happened first. These events happened on processes running on two different machines. The first thought could be to have both processes append the current timestamp to the event before forwarding it to the central authority. That could work in theory, but the problem is that the timestamp is derived by a physical chip running on two different machines. Even just getting them synchronized is hard; using NTP, for example, it will carry some error. Assuming an operator did not mistakenly set the wrong time, they still are subjected to an effect called "clock drifting" that would cause them to go out of sync over time.

First, the paper tries to answer this question: can we find which event happened first without using a physical clock? As we shall see, the answer is *not always*. The second part of the paper strengthens the properties of the first part by introducing physical clocks.

## What's the "happened-before" relation?

If we visualized a sequence of events in a process, "$a$" would show up before "$b$" if "$a$" *happened before* "$b$".

Let's define the "*happened before*" relation, with symbol "$\rightarrow$":

1.  If $a$ and $b$ are events in the same process, and $a$ comes before $b$, then $a \rightarrow b$.

2.  If $a$ is the sending of a message $m$ by a process and $b$ is the receiving of $m$ by another process, then $a \rightarrow b$.

3.  If $a \rightarrow b$ and $b \rightarrow c$ then $a \rightarrow c$. Two distinct events $a$ and $b$ are said to be concurrent if $a \nrightarrow b$ and $b \nrightarrow a$.

Assuming that $a \nrightarrow a$ for any event $a$, this relation defines an irreflexive partial ordering of the events.

The "not always" from the previous paragraph is derived from the fact that for two concurrent events, we cannot tell which one happened before. The message exchange is a synchronization point.

## What's a logical clock then?

Every process has a number in their local state which can be thought as the current clock. The clock is increased after each event, and every event gets its own timestamp. In this way, if $a \rightarrow b$ then $a$'s timestamp will be smaller than $b$. This concept is made formal by what Lamport calls the Clock condition.

In a process $P_i$, let's have a function $C_i$ that assigns a timestamp number $C_i\langle a \rangle$ to the event $a$. Then the Clock Condition is defined as:

$For\ any\ events\ a,b:\ if\ a \rightarrow b\ then\ C\langle a \rangle < C\langle b \rangle$

It's also interesting to note that the converse is not true. If $C\langle a \rangle < C\langle b\rangle$, it is possible that $a$ and $b$ are two concurrent events (running at the same time).

To synthesize, a logical clock can be thought as a way to add a timestamp to an event without using a physical clock.

But how can we use it in practice in our algorithms? The paper gives two rules:

1.  Each process $P_i$ increments $C_i$ between two successive events.

2.  If event $a$ is sending a message $m$ by process $P_i$, then the message $m$ contains timestamp $T_m = C \langle a \rangle$ . Upon receiving a message m, process $P_i$ sets $C_i$ greater than or equal to its present value and greater than $T_m$.

## How can we get a total ordering?

Lamport presents a way to arbitrarily extend the → relation to a ⇒ relation that represents a total order. To break the ties, each process gets assigned a unique numeric id. When ordering events, first sort by the timestamps. If the timestamps are equal, then we sort by the process id. This will yield a total ordering.

As mentioned, this is an arbitrary way to break the ties. It opens up to some anomalies on how the users perceived the passage of time and how the system perceived it. The paper explains that a user could send a request to some service, call their friend on the phone and tell them to send the same request, and the second request could end up with a lower timestamp.

Then, we have only two-way around this. Either, when calling their friend, the first user provides the timestamp attached to their request and the second user attaches it to their request; or we need a system of clocks that can satisfy the following **strong** clock condition:

$$For\ any\ events\ a,b:\ a \rightarrow b\ \equiv C\langle a \rangle < C\langle b \rangle$$

This is saying that if $C\langle a \rangle < C\langle b\rangle$, it means that a happened before b. In general, logical clocks don't satisfy the strong clock condition. The rest of the paper is devoted to creating a new system of clocks using physical clocks that is able to satisfy the strong clock condition.

It's interesting to know that a solution to implement such a system of clocks without using physical clocks is by using Vector clocks. Vector clocks keep track of time using a vector of clocks instead of a simple scalar like Lamport clocks. Physical clocks are used in practice in systems like Google's TrueTime and in Hybrid clocks.

## How can we solve the mutual exclusion problem using Lamport clocks?

The last part that I would like to write about is a practical example of how to use Lamport clocks to solve a mutual exclusion problem as presented in the paper. An instance of this problem would be when two processes need to access a shared resource, like a disk.

Let's put some rules:

1.  A process which has been granted the resource must release it before it can be granted to another process.

2.  Different requests for the resource must be granted in the order in which they are made.

3.  If every process that is granted the resource eventually releases it, then every request is eventually granted.

The proposed algorithm is decentralized (no one node is more important than others) and doesn't deal with failures.

The local state of each process has:

For simplicity, each process can send messages to every other process, messages are never lost, and they arrive in the order in which they are sent.

The algorithm solves the mutual exclusion problem using six actions. Each action maps to an event that causes an increase of the local timestamp:

1.  To request the resource, process $P_i$ sends the message $<<RequestResource,\ P_i,\ T_m>>$ to every other process, and puts that message on its own request queue, where Tm is the timestamp of the message.

2.  When process $P_j$ receives the message $<<RequestResource,\ P_i,\ T_m>>$ requests resource, (i) it places it on its request queue and (ii) sends a (timestamped) acknowledgment message to $P_i$.

3.  Whenever $P_i$ receives the $AckRequestResource$ it appends it to its local acknowledgments set.

4.  To release the resource, process $P_i$ removes any $<<RequestResource,\ P_i,\ T_m>>$ message from its request queue and sends a timestamped $P_i$ $ReleaseResource$ message to every other process.

5.  When process $P_j$ receives a $P_i$ $ReleaseResource$ message, it removes any $<<RequestResource,\ P_i,\ T_m>>$ requests resource message from its request queue.

6.  Process $P_i$ is granted the resource when the following two conditions are satisfied: (i) There is a $<<RequestResource,\ P_i,\ T_m>>$ in its request queue which is ordered before any other request in its queue by the relation ⇒. (To define the relation " ⇒ " for messages,

    we identify a message with the event of sending it.) (ii) $P_i$ has received a message from every other process timestamped later than Tm.

### The TLA+ specification

From this blueprint, I've written a more formal specification using TLA+ and pluscal. I'll provide a short walkthrough here covering only the interesting parts, but you can find the full spec [here](https://github.com/FedericoPonzi/tla-plus-specs/).

```
---- MODULE lamport_clock ----

CONSTANT Processes, ProcessCanFail, MaxTimestamp
ASSUME IsFiniteSet(Processes)
ASSUME Cardinality(Processes) > 1
ASSUME \A p \in Processes : p \in Nat
ASSUME MaxTimestamp \in Nat
ASSUME ProcessCanFail \in BOOLEAN 
```

Constants available are:

*   `Processes`: a set of processes like `{0, 1, 2}`.

*   `ProcessCanFail`: a boolean, if set to true it will show how liveness property is broken.

*   `MaxTimestamp`: to keep the model bounded, I've set a limit on the max possible timestamp. Otherwise, TLC wouldn't know when to stop model checking.

The assume statements are helpful for basic typechecking and make sure the configuration is correct.

```
(* --fair algorithm mutual_exclusion {

    variables resource_owner = {},
              mailbox = [p \in Processes |-> EmptyMailbox]; 

    define {
        CanSendMessage(myself) == \A p \in Processes : mailbox[p] = EmptyMailbox
				SortFunction(seq1, seq2) ==
            IF seq1.ts > seq2.ts
            THEN FALSE
            ELSE IF seq1.ts < seq2.ts
            THEN TRUE
            ELSE IF seq1.proc > seq2.proc
            THEN FALSE
            ELSE IF seq1.proc < seq2.proc
            THEN TRUE
            ELSE FALSE
        \* SAFETY: Only one process is allowed in the critical section
        Inv == Cardinality(resource_owner) <= 1 
```

The resource owner is a set of the current owner of the resource or, in other words, the process currently in its critical section.

The mailbox is used to communicate across processes. It allows one message at a time; it's simple, but it works. An alternative would be to keep al messages around in a message board in [Linda tuplespaces style](https://en.wikipedia.org/wiki/Linda_(coordination_language)).

The `define` block has some useful definitions. `SortFunction` is used to create the total order. `Inv` is the safety invariant used to check that only a single process is in the critical section.

```
fair process (p \in Processes)
    variables local_timestamp = 0,
              ack_request_resource = {},
              requests_queue = {}; 
```

These variables are the local state of the process, as explained in the previous section.

The body of the process is used to define the actions above.

```
while (TRUE) {
        either { 
            (*********************
                1\. To request the resource, process Pi sends the message <<RequestsResource, Pi, Tm>> 
                    to every other process, and puts that message on its own request queue, where Tm is the timestamp of the message.
            **********************)
            await /\ CanSendMessage(self) 
                  /\ Cardinality(ack_request_resource) = 0;
            BumpTimestamp();
            requests_queue := requests_queue \union {[ msg |-> RequestResource, proc |-> self, ts |-> local_timestamp]};
            mailbox := [p \in Processes |-> IF p = self THEN EmptyMailbox ELSE [ msg |-> RequestResource, proc |-> self, ts |-> local_timestamp]];
            ack_request_resource := {[ msg |-> AckRequestResource, proc |-> self, ts |-> local_timestamp]}; 
```

The conditions to enter this block are:

*   All mailboxes are empty or in other words, `p` can deliver a message to all the other processes

*   `p` hasn't sent a request resource yet. This works because at the end of this action, `p` has added an `AckRequestResource, self, local_timestamp` to the local set. In this way, the cardinality of ack_request_resources can be used to understand if `p` has sent a `RequestResource` - without keeping additional state around. Avoiding sending multiple requests without having acquired the resource is just a way to simplify the spec.

```
(*********************
2\. When process Pj receives the message <<RequestsResource, Pi, Tm>> requests resource, 
    (i) it places it on its request queue and 
    (ii) sends a (timestamped) acknowledgment message to Pi.
**********************)
await mailbox[self].msg = RequestResource;
requests_queue := requests_queue \union {mailbox[self]};
BumpTimestampO(mailbox[self].ts);

\* Respond with an Ack
await mailbox[mailbox[self].proc] = EmptyMailbox; 
mailbox[mailbox[self].proc] := IF ProcessCanFail /\ \A proc \in Processes \ {self}: proc > self \* if process can fail and this is the smallest pid
                                THEN EmptyMailbox
                                ELSE [ msg |-> AckRequestResource, proc |-> self, ts |-> local_timestamp];
EMPTY_MAILBOX:
mailbox[self] := EmptyMailbox;
BumpTimestamp(); 
```

The mailbox is a "global" map variable. I can use `mailbox[self]` to access p's allocated mailbox. Because a message has shape `msg, proc, ts`, I can access the sender's mailbox using `mailbox[mailbox[self].proc]` .

If `ProcessCanFail` is set to true, the smallest process `p` will never send the ack back, simulating a failure. This halts the system and breaks the liveness property.

```
(*********************
    5\. When process Pj receives a Pi ReleasesResource message, it removes any <<RequestsResource, Pi, Tm>> 
       requests resource message from its request queue.
**********************)
await self \in resource_owner  /\ CanSendMessage(self);
requests_queue := SeqToSet(Tail(SetToSortSeq(requests_queue, SortFunction))); 
BumpTimestamp();
mailbox := [p \in Processes |-> IF p = self THEN EmptyMailbox ELSE <<ReleaseResource, self, local_timestamp>>];
resource_owner :=  resource_owner \ {self};
ack_request_resource := {}; 
```

To remove the `RequestResource` from the `requests_queue`, it is sufficient to order the requests using the sort function and discard the head of the result. This is safe thanks to the assumptions we made of no lost messages.

```
(*********************
6\. Process Pi is granted the resource when the following two conditions are satisfied: 
    (i) There is a <<RequestsResource, Pi, Tm>> in its request queue which is ordered before any other 
        request in its queue by the relation ⇒. (To define the relation " ⇒ " for messages, we identify a message
        with the event of sending it.).
    (ii) Pi has received a message from every other process timestamped later than Tm.
**********************)
await /\ Cardinality(requests_queue) > 0 
      /\ Head(SetToSortSeq(requests_queue, SortFunction)).proc = self
      /\ Cardinality(ack_request_resource) = Cardinality(Processes)
      /\ self \notin resource_owner;

resource_owner := resource_owner \union {self}; 
```

Locally each `p` can now check if they're allowed to access their critical section. The `self \notin resource_owner` is just to reduce the number of state changes. As `resource_owner` is just a set, nothing bad would happen by re-entering this action.

```
MaxtimestampConstraint == \A proc \in Processes: local_timestamp[proc] < MaxTimestamp 
```

This definition can be used as a state constraint to keep the model bounded. With this, I can then add in the cfg file:

```
CONSTRAINTS 
    MaxtimestampConstraint 
```

And now TLC will stop when this constraint returns TRUE. This approach is well described here: [https://learntla.com/topics/unbound-models.html#topic-unbound-models](https://learntla.com/topics/unbound-models.html#topic-unbound-models)

```
\* LIVENESS: either we eventually get ownership, or we're at the edge of the timestamp boundary.
ResourceAcquisition == \A proc \in Processes : Cardinality(ack_request_resource[proc]) > 0 ~> \/ proc \in resource_owner 
                                                                                              \/ local_timestamp[proc] = MaxTimestamp 
```

As we saw, when `p` sends a `RequestResource` message it will append an ack in the local queue. The liveness property says that for any process that has requested access, eventually they will get ownership of the resource. I had to rule out the case in which we run out of timestamps, because in that case, the system cannot make further progress. This means that I had to disable the `CHECK_DEADLOCK` parameter from the config as well.

This concludes the walk-through, you can find the full spec on [GitHub](https://github.com/FedericoPonzi/tla-plus-specs/). Interestingly, after the spec was respecting the properties, I was able to refactor it by removing unneeded states and unneeded checks, leading to a short and simpler spec; in a "Test Driven Development" style.

A possible improvement would be to specify the mutual exclusion problem in a separate specification, and use refinement to check that this algorithm solves that problem.

## Conclusion

In this article, I provided an overview of Logical Clocks introduced by Lamport in his paper *[Time, Clocks, and the Ordering of Events in a Distributed System by Leslie Lamport](http://lamport.azurewebsites.net/pubs/time-clocks.pdf)*.

Logical clocks can be used to create a partial ordering among a set of events in a distributed system. The relation "happened before" is able to define a logical passage of time by using cause-effect rather than relying on physical clocks. Physical clocks are inaccurate and hard to keep in sync across different machines and long distances.

To break ties and get to a total ordering of the events, Lamport suggests ordering by timestamp and then by the process id. This is arbitrary in a sense that we define an ordering on concurrent events that could differ from the real time ordering. This can lead to anomalies. To get rid of them, we need a system of clock that satisfies the strong clock condition.

I believe that a practical example is very useful to understand how to use Lamport clocks in practice. A TLA+ specification is presented to solve the mutual exclusion problem. TLA+ specs are a very nice and fun way to play with algorithm without having to worry about the irrelevant details.

While writing the spec for this post, I've also [contributed](https://github.com/hwayne/learntla-v2/pull/78) with some documentation to the learntla website.

## References