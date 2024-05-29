<!--yml
category: 未分类
date: 2024-05-27 14:29:10
-->

# A CAP tradeoff in the wild - decomposition ∘ al

> 来源：[https://decomposition.al/blog/2023/12/31/a-cap-tradeoff-in-the-wild/](https://decomposition.al/blog/2023/12/31/a-cap-tradeoff-in-the-wild/)

There’s a classic tradeoff between safety and liveness in the context of replicated data systems, originally proposed by Eric Brewer and later made precise by Seth Gilbert and Nancy Lynch. The idea is that, in a networked system of servers that is vulnerable to partitions (that is, where messages betweeen servers can be arbitrarily delayed, perhaps forever), it’s impossible to ensure both that when servers respond to requests, it’s with the “right” response (*consistency*, a safety property), and that every request eventually receives a response (*availability*, a liveness property).

Imagine that we have a system with two servers, call ‘em Alice and Bob, that currently can’t talk to each other because of a network partition. Just then, a client asks Alice for some piece of information. In the meantime, Bob may have a more up-to-date copy of that information – but Alice can’t talk to Bob to find out. So Alice has a dilemma: she can wait – perhaps forever – for the partition to heal, so that she can see if Bob has more up-to-date information than she does, sacrificing availability. Or Alice can just tell the client what she knows, which might be stale, sacrificing consistency. In the presence of network partitions, there’s no way to avoid sacrificing one or the other. This is the (in)famous CAP theorem that [Gilbert and Lynch formalized in 2002](https://www.comp.nus.edu.sg/~gilbert/pubs/BrewersConjecture-SigAct.pdf). (I think their [2012 follow-up article](https://decomposition.al/CSE232-2023-09/readings/cap-perspectives.pdf) is a more enlightening read.)

This [bug filed against Kubernetes in 2018 and still open](https://github.com/kubernetes/kubernetes/issues/59848) came up in a discussion with one of my PhD students not too long ago, and I think it’s a pretty amazing example of a CAP tradeoff in the wild.

Figure 2 of [this HotOS 2021 paper](https://sigops.org/s/conferences/hotos/2021/papers/hotos21-s11-sun.pdf) by Xudong Sun et al. illustrates the bug. The left column of the figure, labeled “history”, represents “the truth” about cluster state, as seen by etcd, a strongly consistent data store. However, rather than talking directly to etcd, various Kubernetes components (known as “apiservers” and “kubelets”) only see that state indirectly, via various levels of caches. The apiservers API-1 and API-2, represented by the middle two columns in the figure, each maintain such a cache.

In the example run shown in the figure, API-2 is “experiencing network connectivity issues with etcd”, and so its cache is behind. Meanwhile, pod P1 is created on kubelet K1 and then moved to K2, which requires deleting it on K1 and then creating it on K2\. API-1 is aware of the fact that P1 has moved from K1 to K2, but API-2 isn’t.

Then K1 crashes, comes back up, reads the cluster state (which says that P1 should still exist on K1!) off API-2, and as a result, recreates P1 on K1\. Now we’re violating a safety property which says that two pods with the same name shouldn’t exist.

As the [bug report explains](https://github.com/kubernetes/kubernetes/issues/59848), this is supposed to be an “HA” (high-availability) setup, which is why both API-1 and API-2 exist at all. But the fact that they (and, crucially, their caches) *do* both exist leads us to a CAP tradeoff, because [caching is replication](https://twitter.com/lindsey/status/1446931576185524225), and that tradeoff unsurprisingly rears its head when the inevitable network partition occurs.

API-1 and API-2 are analogous to Bob and Alice in my example above. It’s not a perfect analogy, since in my example scenario, the partition is preventing *direct* communication between Alice and Bob, whereas in the Kubernetes bug scenario, the partition is preventing *indirect* communication between API-2 and API-1, via the shared state of etcd. Either way, though, the point is that API-2 and API-1 can’t communicate, because API-2 can’t talk to etcd, which means it can’t get the information that API-1 has. API-2 therefore has to choose between safety and liveness. It chooses liveness – a justifiable choice! – and so a safety invariant gets violated.

Among the dozens of comments on GitHub are some folks asking if you could just read directly from etcd, instead of from the caches on the two apiservers. This would indeed sidestep the tradeoff – you can’t have replicas that disagree if you only have one replica! But others in the thread are pointing out that doing that would create an untenable scalability bottleneck. And back and forth it goes.

I can’t blame the Kubernetes developers for not yet being able to settle on a solution here, because there isn’t any solution that doesn’t sacrifice something desirable. A comment from Josh Berkus in the middle of the thread sums it up nicely with [“Welcome to the CAP theorem, I guess?”](https://github.com/kubernetes/kubernetes/issues/59848#issuecomment-960282426)

To end with a possibly-inflammatory aside, I find Figure 1 in the aforementioned [HotOS paper](https://sigops.org/s/conferences/hotos/2021/papers/hotos21-s11-sun.pdf) to be pretty telling: we’re sitting on top of etcd, this strongly-consistent data store, but between it and us are layers upon layers of caches. If you’re just going to be reading from a possibly-stale cache at some level above, did you really need etcd at all?