<!--yml
category: 未分类
date: 2024-05-27 15:18:28
-->

# The Big Little Guide to Message Queues

> 来源：[https://sudhir.io/the-big-little-guide-to-message-queues](https://sudhir.io/the-big-little-guide-to-message-queues)

# The Big Little Guide to Message Queues

A guide to the fundamental concepts that underlie message queues, and how they apply to popular queueing systems available today.

Message Queues are now fairly prevalent—there are so many of them showing up so fast you'd think they were [rabbits](https://www.rabbitmq.com) with an unlimited supply of [celery](https://docs.celeryproject.org/en/stable/), resulting in an [kafkaesque](https://kafka.apache.org) situation where making a decision is like trying to catch a [stream](https://redis.io/topics/streams-intro) in your hands. If only there were fewer [simple](https://aws.amazon.com/sns) [services](https://aws.amazon.com/sqs/) that could help with [publishing and subscribing](https://cloud.google.com/pubsub), it would be so much easier to make a [zero-effort](https://zeromq.org) choice 😕

Whether we use them by themselves as-is to move data between parts of our application, or as an integral part of the architecture (like event driven systems), message queues are here to stay. In a way, they've been here all along—just without as many names. But what are they? Why are they useful? And how do we use them effectively? Which implementation do we pick? Does it even matter which one we use? And do we need to learn each of them individually, or are there more general concepts that apply to all message queues?

In this guide, we'll talk about:

*   What message queues are and their history.
*   Why they're useful and what mental models to use when reasoning about them.
*   Delivery guarantees that the queuing systems make (at-least-once, at-most-once, and exactly-once semantics).
*   Ordering and FIFO guarantees and how they effect sequencing, parallelism and performance.
*   Patterns for fan-out and fan-in: delivering one message to many systems or messages from many systems into one.
*   Notes on the pros and cons of many popular systems available today.

## What are message queues?

Message Queues are a way to transfer information between two systems. This information—a message—can be data, metadata, signals, or a combination of all three. The systems that are sending and receiving messages could be processes on the same computer, modules of the same application, services that might be running on different computers or technology stacks, or entirely different kinds of systems altogether—like transferring information from your software into an email or an SMS on the cellphone network.

The idea of a messaging system has been around a very long time, from the message boxes used for moving information between people or office departments (literally where the words *inbox* and *outbox* come from), to telegrams, to your local postal or courier service. The messaging systems in the physical world that come closest to what we have in computing are probably the [pnuematic](https://en.wikipedia.org/wiki/Pneumatic_tube) [tubes](https://www.google.com/search?q=pneumatic+tubes&source=lnms&tbm=isch&sa=X&biw=2560&bih=1366) that moved messages through buildings and cities using compressed air until a few decades ago (and are still used in some places today).

The kinds of messages we transfer today might be a note that something technical happened, like CPU usage exceeding a limit; or a business event of interest, like a customer placing an order; or a signal, like a command that tells another service to do something. The contents of each message will be driven entirely by the architecture of your application and its purposes—so for the rest of this guide, we don't need to be concerned about what's inside a message—we're more concerned with how the message gets from the system where it originates (the *producer*, *source, publisher* or *sender*) to the system where's it's supposed to go (the *consumer*, *subscriber*, *destination* or *receiver*).

## And Why Do We Need Them?

We need message queues because no system exists or works in isolation—all systems need to communicate with other systems in structured ways that they both can understand, and at a controlled speed that they both can handle. Any non-trivial process needs a way to move information between each stage of the process; any workflow needs a way to move the intermediate product between the stages of that workflow. Message queues are a great way to handle this movement. There are plenty of ways of getting these messages around using API calls, file systems, or many other abuses of the natural order of things; but all of these are ad-hoc implementations of the message queue that we sometimes refuse to acknowledge we need.

The simplest mental model for a message queue is a very long tube that you can roll a ball into. You write your message on a ball, roll it into the tube, and someone or something else receives it at the other end. There are a lot of interesting benefits with this model, some of which are:

*   We don't need to worry about *who or what* is going to receive the message – that's one less responsibility for the sender to be concerned about.
*   We don't need to worry about *when* the receiver is going to pick up the message.
*   We can put *as many* messages as we want into the tube (let's assume we have a infinitely long tube) at whatever *speed* is comfortable to us.
*   The receiver will *never be impacted* by our actions—they will pull out as many messages as they want at whatever rate is comfortable to them.
*   Neither the sender nor the receiver are concerned with *how* the other works.
*   Neither the sender nor the receiver are concerned with the *capacity or load* of the other.
*   Neither system is concerned with *where* the other one is – they may or many not reside on the same computer, network, continent or even the same planet.

Each of these advantages (and this isn't even an exhaustive list) has very important benefits in software development—what they all have in common is *decoupling*. One system is decoupled from the other in terms of responsibility, time, bandwidth, internal workings, load and geography. And decoupling is a very desirable part of any distributed or complex system—the more decoupled the parts of the system are, the easier it is to independently build, test, run, maintain and scale them.

Most systems interact with other outside or third-party systems as well—if we build a shopping site we might interact with a payment processor, and let’s say we attempt to directly communicate with the payment processor on each user click. If our system is under heavy load, we're also subjecting the other system to the same load. And vice versa—if our payment provider needs to send us millions of pieces of information about our past payment statuses, our system better be ready. The two systems are now *coupled*. The decisions and actions made by one system have a significant impact on the other, so the needs of both need to be taken into account while making every decision. Add enough other systems into the mix, like logistics or delivery systems, and we quickly have a paralysing mess that makes it difficult to decide anything at all. If one system goes down, the other systems have effectively gone down as well, for no fault of their own.

We’re also in trouble if we want to switch out any one of these systems for another one, like a new payment processor or delivery system. We’d have to make deep changes in multiple places in our application, and it’s even more difficult to build code to split our messages between multiple providers—we may want to use a ratio to load balance them or split them by geography; or dynamically switch between them based on each provider’s availability or cost.

Message queues offer the decoupling that solves a lot of these problems. If we set up a queue between two systems that need to communicate with each other, they can now go about their work without having to worry about each other at all—we put our messages aimed at any system into a queue, and we expect information from the other system to come to us through another queue as well. We now have clear points at which we can add rules or make the changes we require, without either system knowing or caring about what's different.

### So what's the catch?

Are message queues the holy grail of computing, though? Do they solve all the world's problems? No, of course not. There are plenty of situations where we might not want to use them. And we certainly don't want to use a queue just because we have one easily available and think it might be fun. There are some systems that are really simple that just don't require it—a message queue is a way to reduce complexity of communicating systems, but two communicating systems will always be more complex than one system that doesn't have to communicate. If you have a system that’s simple enough to not require communication with any others, there simply isn't any reason to reach for a queue.

There are also systems that communicate with each other, but where the complexity added by that communication added is insignificant and not worth worrying about. Or more often the systems are already coupled, in the sense that they all need to work together to function. A really common example is an application server and a data service (in an *OLTP* system). There's not much point in decoupling them with a queue, because neither can do anything useful without the direct involvement of the other.

Then there's performance to consider as well—the whole point of decoupling two systems with regards to time and load is so that they can each process information at their own pace—but we certainly would *not* want this to happen in performance sensitive applications or real-time systems. A queue might help us process more work at the same time (the receiver might have many processes working in parallel on the messages you send) but will remove any guarantees we need about the exact time taken for each piece of work. If predictability is more important than throughput, we're better off without a queue.

Using a queue might increase the time taken to process each *individual* message, but will allow you process many more messages at the same time across different computers—so your total number of messages processed per minute or hour, or *throughput*, will increase.

If we do have multiple systems that need to communicate, and that communication needs to be *durable* (if we’ve put a message into a queue, we want to be sure that the messaging system isn’t going to ‘forget’ about it) and decoupled, a message queue is indispensable.

## Arguing Semantics

There's simply no way to learn about message queues without reading and/or arguing about delivery guarantees and semantics, so we might as well get to that quickly. People who build message queues will claim that their system offers one of three delivery guarantees—that each message you put into the queue will be delivered:

*   *at-least* once.
*   *at-most* once.
*   *exactly* once.

Which guarantees we're using will have a massive impact on the design and working of our system, so let's unpack each of them one by one.

### At-Least Once

This is the most common delivery mechanism, and it's the simplest to reason about and implement. If I have a message for you, I will read it to you, and keep doing so again and again until you acknowledge it. That's it. In a system which works on an at-least-once basis, this means that when you receive a message from the queue and don't delete/acknowledge it, you will receive it again in the future, and will keep receiving it until you explicitly delete/acknowledge it.

The reason this is the most common guarantee is that it's simple and gets the job done 100% of the time—there's no edge case in which the message gets lost. Even if the receiver crashes before acknowledging the message, it will simply receive the same message again. The flip side is that you as the receiver need to plan on receiving the same message multiple times—even if you haven't necessarily experienced a crash. This is because offering at-least-once is the simplest way to protect the queueing service from missing out messages as well—if your acknowledgement doesn't reach the queueing system over the network, the message will be sent again. If there's a problem persisting your acknowledgement, the message will be sent again. If the queuing system restarts before it can properly keep track of what's been sent to you, the message will be sent again. This simple remedy of sending the message again in case of any problem on any side is what makes this guarantee so reliable.

But is message duplication/repetition a problem? That's really up to you and your application or use-case. If the message is a timestamp and a measurement, for example, there's no problem with receiving a million duplicates. But if you're moving money based on the messages, it definitely is a problem. In these cases you'll need to have a transactional (ACID) database at the receiving end, and maybe record the message ID in a unique index so that it can't be repeated. This is called using an *idempotency token* or _tombstone—_when you act on a message you store a unique permanent marker to keep track of your actions, often in the same database transaction as taking the action itself. The prevents you from repeating that action again even if the message is duplicated.

If you handle duplication, or if your messages are naturally resistant to duplication, your systems are said to be *idempotent*. This means you can safely handle receiving the same message multiple times, without corrupting your work. It also often means you can tolerate the sender sending the same message multiple times—remember that senders will usually operate on the at-least-once principle when sending messages as well. If senders are unable to record the fact that they've sent a particular message, they'll simply send it again. The senders are then responsible for making sure that they use the same tombstone or idempotency token if and when they re-send messages.

### At Most Once

This is a pretty rare semantic, used for messages where duplication is so horribly explosive (or the message so utterly unimportant) that we'd prefer not to send the message at all, rather than send it twice. At-most-once once implies that the queuing system will attempt to deliver the message to you once, but that's it. If you receive and acknowledge the message all is well, but if you don't, or anything goes wrong, that message will be lost forever—either because the queuing system has taken great pains to record the delivery to you before attempting to send it (in case the message is horribly explosive), or has not even bothered to record the message at all, and is just passing it on like a router passes on a UDP packet.

This semantic usually comes into play for messaging systems that are either acting as stateless information routers; or in those cases where a repeat message is so destructive that an investigation or reconciliation is necessary in case there's any failure.

### Exactly Once

This is the holy grail of messaging, and also the fountain of a lot of snake-oil. It implies that every message is guaranteed to be delivered and processed exactly once, no more and no less. Everyone who builds or uses distributed systems has a point in their lives where they think “how hard can this be?”, and then they either (1) learn why it's impossible, figure out idempotency, and use at-least-once, or (2) they try to build a half-assed “exactly-once” system and sell it for lots of money to those who haven't figured out (1) yet.

The impossibility of exactly-once delivery arises from two basic facts:

*   senders and receivers are imperfect
*   networks are imperfect

If you think about the problem deeply, there are a lot of things that can go wrong:

*   a sender might be unable to record (they *forget*) that they've sent the message
*   the network call to send the message might fail
*   the messaging system’s database might not be able to record the message
*   the acknowledgement that the messaging system has recorded the message might not reach the sender over the network
*   the sender might not be able to record the acknowledgement that the messaging system has received the message

Let's say all goes well while sending the message—when the messaging system tries to deliver the message to the receiver:

*   the message might not reach the receiver over the network
*   the receiver might not be able to record the message in its database
*   the acknowledgement from the receiver might not reach the messaging system over the network
*   the messaging system’s database might not be able to record that the message has been delivered

Given all the things that can go wrong, it's impossible for any messaging system to guarantee exactly-once delivery. Even if the messaging system is godlike in its perfection, most of the things that can go wrong are outside of it or in the interconnecting networks. Some systems do attempt to use the phrase “exactly once” anyway, usually because they claim their implementation will never have any of the messaging system problems mentioned above—but that doesn't mean the whole system is magically blessed with exactly-once semantics, even if the claims are actually true. It usually means that the queuing system has some form of ordering, locking, hashing, timers and idempotency tokens that will ensure it never re-delivers a messsage that's already been deleted/acknowledged—but this doesn't mean that the whole system including publisher + queue + subscriber has gained full exactly-once guarantees.

Most good messaging system engineers understand this and will [explain](https://www.lightbend.com/blog/how-akka-works-exactly-once-message-delivery) to their users why this semantic is unworkable. The simpler and more reliable way to handle messages is go back to the basics and embrace at-least-once with idempotency measures at every point on the sending, receiving and queuing process: if at first you don't succeed, retry, retry, retry...

## Ordering vs Parallelism

After delivery semantics, another common question on peoples’ minds is “why can’t we just process messages in parallel while also making sure we process them in order?”. Unfortunately this is another tradeoff imposed on us by the tyranny of logic. Doing work in a sequence and doing multiple pieces of work at the same time are always at conflict with each other. Most message queue systems will ask you to pick one—AWS SQS started by prioritising parallelism over strict ordering; but recently introduced a separate FIFO (first in, first out) queuing system as well, which maintains strict sequential ordering. Before making a choice between the two, let’s go over what the difference is and why there needs to be a difference at all.

Returning to our earlier metaphor for a queue—a long tube into which we roll messages written on a ball—we probably imagined the tube to be just a little wider than a single ball. There's really no way the balls could overtake or pass each other inside the tube, so the only way a receiver could get these messages out is one-by-one, in the order they were put in. This guarantees strict ordering, but places strong limitations on our receiver. There can *only be one* *agent* on the receiver side that's processing each message—if there was more than one, there would be no guarantee that the messages were processed in order. Because each new agent could processes each message independently, they could each finish and start on the next message at any time. If the are two agents, A & B, and Agent A receives the first message and Agent B the second; Agent B could finish processing the second message and start on the third message even before Agent A is finished processing the first message. Though the messages were *received from the queue* strictly in the order that they were put in, if there are multiple receiving agents there’s no way to say the messages will *be processed in that order*.

The agents could use a [distributed](https://redis.io/topics/distlock) [lock](https://www.postgresql.org/docs/current/explicit-locking.html#ADVISORY-LOCKS) of some kind to co-ordinate with each other, but this is basically the same as having only one agent—the lock would only allow one agent to work at any given time. This also means that one agent crashing would result in a *deadlock* with no work being done.

One way for the messaging system to guarantee order would be for the tube to refuse to give out the next ball until and unless the last ball that was received has been destroyed (the last message has been deleted/acknowledged). This is what FIFO queues in general will do—they'll provide the next message only after the last one has been acknowledged or deleted—but this means that only one agent can possibly be working at a time, even if there are *N* agents waiting to receive messages from the queue.

Sometimes, this is exactly what we want. Some operations are easier to control effectively when we only have to deal with a single agent, like enforcing rules on financial transactions; respecting [rate limits](https://redis.io/commands/incr#pattern-rate-limiter); or generally processing messages whose formats have been designed assuming they would always be processed in order. But a lot of these “benefits” are not really coming from the decision to use FIFO ordering—any scenario where we have *N* receivers that must somehow co-ordinate their work with each other will benefit from the special case of *N = 1*. The key takeaway is that requiring a guaranteed order means we have to process messages sequentially on only one receiver at a time.

This restriction also places severe pressure on the queuing system, so you'll find that FIFO queues are often more expensive and have less capacity than their parallel counterparts. This is because the same logical limits apply to the internal implementation of queuing system as well—most work needs to be constrained to a single agent or server, and that system needs to be kept reliable. Any effort to add redundancy requires synchronous co-ordination between the master and the backup services in order to maintain the ordering guarantees. In AWS SQS, the FIFO queues are about 2X more expensive than the parallel queues, and are constrained to a few hundred messages per second when strict FIFO ordering is required.

So the only way to move forward with a FIFO message queue is to accept that the entire message processing architecture is going to have an intrinsic speed limit. Many systems will support [group headings](https://docs.aws.amazon.com/AWSSimpleQueueService/latest/SQSDeveloperGuide/using-messagegroupid-property.html) inside the queue to denote what messages we want strict ordering on—we might say that all messages under the heading “payments” need to be FIFO, and all the messages under “orders” need to be FIFO, but they don't need to be FIFO with respect to each other. This allows some parallelisation inside the queue (like having two FIFO tubes instead of one), but we need to remember that the message bandwidth inside each group heading will still be limited.

### Parallel != Random

Does that mean that the ordering in parallel queues is completely random? Sometimes, yes, but most often no. In SQS, the analogy is more that instead of having one tube from the sender to receiver, there are multiple tubes. They might also branch or join each other along the way. This doesn't mean that the order of the messages you roll in are intentionally randomised in any way—across a large number of messages you'd still expect that earlier messages are generally received before the later ones. This is more a *best-effort* ordering, where some effort is make to keep the ordering intact, but because it's already logically impossible, it's simply not a big priority for the system. This also allows a messaging system like SQS to scale up to nearly infinite capacity—because if you're rolling in a lot of messages the queueing system can simply add more tubes. And as you can imagine, this will support any number of receivers at the same time, and any number of senders as well. This simplicity is what allows SQS to scale to mind-boggling numbers, including a case where [there was a queue](https://twitter.com/timbray/status/1246157403663388672) with over 250 billion messages waiting to be consumed, with the receiver reading and acknowledging over a million messages a second. And that’s just one queue operated by one customer.

Most problems that seem like they have a hard FIFO requirement can often lend themselves to parallelism and out-of-order delivery with a little bit of creativity. The sender adding a timestamp into the message is one way to help with this, like in the case where the messages are measurements where only the last one matters. In a more transactional system, the sender can often add a [monotonically increasing counter](https://www.postgresql.org/docs/current/functions-sequence.html) into the messages. If that's impossible, we might be able to handle this based on the contents of the message—if we're messaging the percentage of a file downloaded, for example, seeing 41%, 42% and 43% always means that the current value is 43%—even if we see them out of order as 41%, 43% and 42%.

While it's often a bad idea to change our systems to accommodate the tools we use, designing our messages to allow for out-of-order delivery and idempotency makes the system more resilient in general, while also letting use more parallel messaging systems—often saving time, money and a lot of operational work.

## Fan Out / In

When building a distributed system there's often a need to have the same message sent to multiple receivers—besides the usual receiver of the message, we also often want the same message sent to other places, like an archive, an audit log (for compliance and security checks) or an analyzer for our dashboards. If you're using an event driven architecture with many services, you might want to use a single *event bus* in your application, where all the messages posted into this event bus are automatically sent to all of your services. This is called a *fan-out* problem, where a message from one producer needs to reach many consumers.

The inverse problem, where a single receiver is tasked with reading the messages posted to multiple queues is also common—in the example we considered above, a receiver that was archiving all messages or creating an audit log would probably receive all the messages generated in an organisation, on every queue. It's also common in service architectures to have a function like notifications handled separately—so a notification system might need to receive messages about a new confirmed orders, failed payments, successful shipping and many more. This is a *fan-in* problem, where the messages from many producers need to reach the same consumer.

If all the producers are putting their messages directly into queues, this would be a really difficult problem to solve—we'd have to somehow intercept our queues, and reliably copy the messages into multiple queues. Building, configuring and maintaining this switchboard simply isn't worth the time or the effort—especially when we could just use *topics* instead.

One way to think about topics is that they're similar to the headings you'd see on a notice board at a school or an office. Producers post messages under a specific topic on a board, and everyone interested in that topic will see the message. The most common way messaging systems send the messages to interested receivers is an HTTP(S) request, sometimes also called a *webhook*. In a push-based system like a HTTP request, the message is pushed into the receiver whether it's ready or not. This re-introduces the coupling that we talked about earlier which we want to avoid—we don't want a situation where our receiver collapses under the crushing load of tens / hundreds / thousands / millions of webhooks over a short span of time. The answer here, again, is to just use a message queue to soak up the messages from the topics at whatever rate they're generated. The receivers can then process them at their own pace.

Automatically copying a message from one topic into one or more queues isn't strictly a message queue feature, but it is complementary—most full-featured messaging systems will offer a way to do this. Producers will still continue to put messages into a single place as usual, but this will be a topic, and internally the messages will be copied to multiple queues, each of which will be read by their respective receivers.

In AWS, the service that provides topic based messaging is the Simple Notification Service ([SNS](https://aws.amazon.com/sns/)). Here you create a topic and publish messages into it—the API to publish a message into an SNS topic is very similar to that of publishing a message into an SQS queue, and most producers don't have to care about the difference. SNS then has options available to publish that message into any number of *subscribed* SQS queues (at no extra charge). Each of these subscribed SQS queues would then be processed by their respective receivers.

If you're working with a different system like Apache Kafka, you'll see similar concepts there as well - you'll have *topics* that you publish messages into, and any number of consumers can each read all the messages in a topic. Google's Pub/Sub system integrates topics and queues as well.

This combination of these scenarios is common enough that there's a simple well established pattern to handle it:

*   Publish every message to one appropriate topic.
*   Create a queue for each receiver.
*   Link up each receiver's queue to the topics that the receiver is interested in.

Since it's usually possible to subscribe a queue to any number of topics, there's no extra plumbing required at a receiver to process messages from multiple topics. And of course, it's possible to have any number of message queues subscribed to a single topic. This kind of setup supports both fan-out as well as fan-in, and keeps your architecture open to expansion and changes in the future.

### Poison Pills & Dead Letters

As morbid as that sounds, when setting up systems to talk to multiple other systems there are bound to be mistakes that happen. The usual problem is that a subscriber is hooked up to receive messages from a topic it knows nothing about in a message format it doesn't understand. What happens? Does the subscriber ignore the message? Or does it acknowledge/delete it? Wouldn't be wrong for it to ignore it, because the message would just keep coming back again and again in an at-least-once system? But isn't it worse to delete/acknowledge a message that we aren't handling? Before we reach for philosophy books made from trees fallen in the woods, we might want to configure a *dead letter queue* on our queue. This is a feature that many queue systems give us, where if the system sees a message being sent out for processing repeatedly, unsuccessfully each time, it'll move it out into a special queue called a *dead letter queue*. We'd want to hook this queue up to an alarm of some sort, so we'll quickly know something weird is going on.

A much worse scenario is one in which the message is explosive in some way—maybe it's formatted in XML instead of JSON, or contains user generated content carrying a malformed input attack that causes your parsing code to crash... your subscriber has just swallowed a *poison pill*. What happens when this pill reaches the subscriber is heavily dependent on your technology stack, so needless to say you want to think carefully about error handling and exceptions in the subscriber code. The good news is that if you've configured a *dead letter queue*, just failing silently can be a fine option. The poison pill will eventually show up in the *dead letter queue* and can be examined. Even if the poison message is crashing your subscriber, running an automated restart with a process manager is often enough to retry the message so many times that it moves it to the dead letter queue. You do need to make sure there are no security implications, though, and remember that this is an easy [DoS attack](https://en.wikipedia.org/wiki/Denial-of-service_attack).

Remember to always verify your incoming messages, both in terms of whether the message is structured the way you expect it to be, and if you're the intended recipient.

## The Q-List

Here's a list of some of the more popular message queuing systems available right now, with a list of how the concepts we've seen so far apply to each of them. This isn't an exhaustive list of course, so let me know [@sudhirj](https://twitter.com/sudhirj) if you think there are any that are missing or misrepresented.

### AWS SNS & SQS

AWS runs two services that integrate with each other to provide full message queuing functions. The [SQS](https://docs.aws.amazon.com/AWSSimpleQueueService/latest/SQSDeveloperGuide/welcome.html) service is a pure message queue—it allows you to create a queue, send a message, and receive a message. That's it. The [`ReceiveMessage`](https://docs.aws.amazon.com/AWSSimpleQueueService/latest/APIReference/API_ReceiveMessage.html) API on on SQS queue is pull-only, so you'll need to call it whenever your receiver is ready to process a message. There's a `WaitTimeSeconds` option to block on the call wait for a message for up to 20 seconds, so an effective pattern is to poll the `ReceiveMessage` API in an infinite loop with the 20 second wait turned on.

The topics and fan-out / fan-in functions come with the integration of [SNS](https://docs.aws.amazon.com/sns/latest/dg/sns-sqs-as-subscriber.html), which works on the construct of a *topic*. This allows a message to be posted into a topic, as opposed to a queue. You can then subscribe any number of SQS queues into a topic, so messages published to the topic are copied to all subscribed queues quickly at no additional cost. You'll want to turn on  the [*raw message*](https://docs.aws.amazon.com/sns/latest/dg/sns-large-payload-raw-message-delivery.html) option, which makes posting a message into an SNS topic effectively equivalent to posting it into an SQS queue—no transformations or packaging of any sort will be applied onto the message.

SQS & SNS are both fully managed services, so there are no servers to maintain or software to install. You're charged based on the number of messages you send and receive, and AWS handles scaling to any load.

FIFO options are available on [SNS](https://aws.amazon.com/blogs/aws/introducing-amazon-sns-fifo-first-in-first-out-pub-sub-messaging/) and [SQS](https://docs.aws.amazon.com/AWSSimpleQueueService/latest/SQSDeveloperGuide/FIFO-queues.html), with different pricing and capacity guarantees. AWS uses the term *message group ID* to denote a group heading under which all messages are FIFO. Messages inside a group heading are delivered in order by not giving out the next message until the previous message is deleted.

### Google Pub/Sub

Google provides the [Pub/Sub](https://cloud.google.com/pubsub/docs/overview) service as part of their cloud platform to handle message queues and topics in an integrated service. The concept of topics exists as you'd expect it, while a queue is called a *subscription*. As expected, associating multiple subscriptions with a topic will copy the message to all associated subscriptions. Besides allowing subscribers to poll, or *pull*, messages from the subscription, Pub/Sub can also do a [*webhook*](https://cloud.google.com/pubsub/docs/push) style POST of the message to your server, letting you acknowledge it with a success return status code.

This is also a fully managed system, like AWS. You're charged based on the number of messages you send, and Google handles scaling the system to whatever capacity you need. It also has a few features that aren't available in the SNS+SQS combo, like allowing you to look into your historical record using timestamps and [replay messages](https://cloud.google.com/pubsub/docs/replay-overview).

[FIFO functionality](https://cloud.google.com/pubsub/docs/ordering) exists inside the context of an *ordering key*, which allows you to ensure that messages inside an ordering key are processed in sequence by not giving you the next message until the previous one has been acknowledged.

### AWS EventBridge

A new offering from AWS, [EventBridge](https://aws.amazon.com/eventbridge/) offers a fully managed *event bus*—this is a varition on the concept of queues and topics, where all messages are posted into a single bus with no visible topic separation. Instead, every message needs to be structured according to a standard format that has information about the message topic inside it. The bus will then read the message, and route it to whichever subscribers have expressed an interest in receiving the messages about that topic. The actual delivery mechanism from the bus to the subscriber can be an SQS queue, webhooks, or many other platform specific options. This makes it easy to manage the event bus as a separately configurable rule-based switchboard, while also allowing easy plugins for archival, auditing, monitoring, alerting, replay, etc.

### Redis Streams

Redis has a relatively new [Streams](https://redis.io/topics/streams-intro) feature that works great for message queues. It works by creating topics on the fly and adding messages to them with the `XADD` command. Reading the messages directly off the topic is possible with `XREAD`, so each subscriber can maintain its own state (the last read offset) to read through the messages. To avoid each subscriber having to maintain its current state, it makes more sense to create *consumer groups* using `XGROUP CREATE`, which are the equivalents of queues. Each message sent to a topic becomes available independently in each consumer group, which can then be subscribed to with `XREADGROUP`. Messages can be acknowledged with `XACK`.

The Redis Streams are automatically FIFO ordered using a timestamp that can either be auto-generated or manually set. This also means that messages can only be processed by one consumer agent at a time. To get around this limitation and work with many consumer agents in parallel, there's a separate non-streams-based pattern described in the documentation for `[RPOPLPUSH](https://redis.io/commands/rpoplpush#pattern-reliable-queue)`—basically `LPUSH` messages into a topic, and then `RPOPLPUSH` them into other lists, each representing a queue, or more accurately its work in progress. `LREM` works to delete/acknowledge the message.

Redis is an open source system that you can install and maintain yourself or find managed hosting for. Depending on how durable you need the system to be you might want to figure out the best [persistence](https://redis.io/topics/persistence) mechanism to use.

### Apache Kafka

[Kakfa](https://kafka.apache.org/documentation/) is a popular message broker that works on the concepts of *producers* publishing messages, called *events*, to *topics*. The events in a topic are split into *partitions*, using a partition key inside of the topic, and FIFO ordering is maintained inside every partition. Events can be streamed to *consumers* over a socket, or queried by the consumers for a more decoupled approach. For consumers that don't want to maintain state, the concept of a *consumer group* applies, same as Redis Streams. A *consumer group* is effectively a queue, where every event posted to a topic is available for processing in every associated *consumer group*.

Kafka is open source, but is a complicated to install and maintain, which makes it suitable for larger projects and teams. It scales based on how well you split your events in to partitions—the more partitions you have the more Kafka can distribute work, and each partition has only as much capacity as the server that's in charge of managing it. Managed hosting options are avaiable, but they tend to have high base costs compared to managed services like SNS+SQS, Pub/Sub or RabbitMQ.

### RabbitMQ

RabbitMQ is a popular open source message broker that supports a variety of [protocols](https://www.rabbitmq.com/protocols.html), with [direct support](https://www.rabbitmq.com/tutorials/tutorial-five-python.html) for the concepts of topics and queues. RabbitMQ operates under both *at-most-once* and *at-least-once* modes, with *at-most-once* being a fast memory based mode that occasionally writes messages to the disk if necessary (you can choose between pesisted or transient queues). If you want a more reliable, but slower, at-least-once system, you can use the operations described in the [reliability guide](https://www.rabbitmq.com/reliability.html) to ask for *confirmations* when publishing messages, and require mandatory *acknowledgements* when reading them. Queues are FIFO by default, with the option to enforce sequential processing with acknowledgements.

### NSQ

The first truly distributed message queue in this list, [NSQ](https://nsq.io) is interesting because it's built from the group up to be decentralized. There's no single point to connect to to publish or subscribe to messages—every NSQ node is effectively a full server and talks to every other node. The nodes allow you to publish messages to *topics*, and each topic can be linked to one or more *channels*—which are the equivalent of queues. Every message published to a topic is available in all its linked *channels*.

NSQ is [defaults](https://nsq.io/overview/features_and_guarantees.html) to non-durable, at-least-once, un-ordered messaging, but has a few configuration options to tweak things. It's specially worth considering if your servers are highly networked with each other and you want a system without a single point of failure.

### NATS

[NATS](https://docs.nats.io/) is a high performance distributed messaging system that's made for fast in-memory messaging. It supports topic based [broadcast](https://docs.nats.io/nats-concepts/pubsub) (topics are called *subjects*), where all messages sent to the *subject* are sent to all subscriber agents; and [distributed queues](https://docs.nats.io/nats-concepts/queue), where each message in the queue is sent to any one subscriber agent. There isn't a built-in way to have topics linked to queues, but that should be possible to do programmatically.

NATS supports both *at-most-once* and *at-least-once* delivery, and by providing a [streaming system](https://docs.nats.io/nats-streaming-concepts/intro) and an [experimental persistence system](https://github.com/nats-io/jetstream). It also has support for subscribing to multiple topics based on subject name patterns, which makes it easier to do fan-in and multi-tenancy.

NATS works great when you need a high throughput distributed system—it's also pretty easy to run, and supports complex network topology, like having regional clusters with connections between them.

## The Tail Message

These are just a few of the options available right now, and still more are being developed as distributed computing develops and cloud providers grow. I've found that the important thing when evaluating or using queueing systems is to understand the semantics & guarantees they offer. I do this by reading their architectural overviews to get rough idea of how they're implemented. Beneath the surface, the same concepts apply to all of them, just under different name and configuration options.

If you're running your workloads in a particular cloud provider, the default topic/queue system they offer will usually work fine, as long as you understand what semantics they're offering in each mode. If you're managing your own installation of a queue system, the same thing applies—except you need to be a lot more concerned about the limit imposed by the operating decisions you're making, like how many nodes you're running, failover configuration, drive space, etc.

Thanks for reading, reach out to me [@sudhirj](https://twitter.com/sudhirj) or join the discussion on [Hacker News](https://news.ycombinator.com/item?id=25591492) if you have questions or disagree with anything.

Special thanks to [@svethacvl](https://twitter.com/svethacvl) for proofreading, and [@wallyqs](https://twitter.com/wallyqs) for notes on NATS.