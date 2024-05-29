<!--yml
category: 未分类
date: 2024-05-27 15:22:53
-->

# Why I Moved My Blog from IPFS to a Server | Neiman’s Blog

> 来源：[https://neimanslab.org/2024-01-31/why-i-moved-my-blog-ipfs-to-server.html](https://neimanslab.org/2024-01-31/why-i-moved-my-blog-ipfs-to-server.html)

Written by Neiman
on January 31, 2024

# Why I Moved My Blog from IPFS to a Server

It’s safe to say I was a pioneer of IPFS + ENS websites. When I set up my first ENS+IPFS website in March 2019 there were no more than 15 others. Between 2019 to 2022 I co-built an IPFS+ENS browser extension (Almonit), an IPFS+ENS search engine (Esteroids), and of course, my personal blog was available only in IPFS+ENS.

But today I moved my blog back to a server, and I’d like to discuss why.

What got me excited about peer-to-peer websites like IPFS is that, theoretically, the more visitors a website has, the more robust, censorship-resistant, and scalable it is. Again, theoretically.

Do you know how popular torrent files seem to live forever? I wanted the same but for websites. I imagined a website that is hard to ddos (Robust), difficult to block (Censorship-resistant), and the more readers it has, the faster it is to access as more readers help to spread the content (Scalable).

I imagined a website with a big “Pin Me” button (pinning in IPFS is like seeding in BitTorrent). If a reader presses the button they will help serve the website.

In practice this didn’t work out really, and for several reasons.

1.  IPFS users mostly don’t run their own nodes or software. Instead, they use gateways. It’s an educated guess I’m making based on what I see in the community, and based on the fact that it’s quite an inconvenience to run your own IPFS node. But even if you do run your own node, the fact you access a website doesn’t mean you pin it. Not at all.

    This is a huge difference from BitTorrent where the only way to get content is to run your own software, and when you download something you also share it, by default.

    Hence, most readers will not help share a website, but even for the ones who will there are still extra complications:

2.  Websites are dynamic objects. Their content is being updated all the time. If you just pin the content of the current version of a website, that’s not much help.

    What most IPFS websites do is use a name system that points to the latest version of its content. It’s usually either IPNS, the internal name system of IPFS, or ENS, Ethereum Name System. But IPFS doesn’t include yet an easy command to always pin the latest content of IPNS, and if someone uses ENS, it means that whoever pins it also needs to listen to Ethereum blockchain events, a huge extra challenge on its own to do without a centralized service.

3.  To make things worse, it’s actually quite hard to get IPFS content to be available in a browser in a reliable way!

    For example, I wanted my IPFS blog to be available in all major gateways, all IPFS nodes, Brave browser (which supports IPFS natively), and js-libp2p & helia (the js libraries of IPFS). I didn’t find a reliable way to achieve that on my own.

    **Long rant:** I pinned content from my own server and played forever with the settings and definitions, but couldn’t get the content to be available everywhere. Worse was Helia, where I just couldn’t manage to get my content accessible from Helia within the browser without connecting directly to my own node. But then what’s p2p network about it?

    I found out that there’s a service, [cid.contact](https://cid.contact), called a “Content Routing” service. It’s written in the “about section” of cid.contact that it’s related to Filecoin, but for some reason, it holds routing data for IPFS, which as far as I know is a different network. The address cid.contact is hard-coded into Helia’s code in the version I used at least, and it was clear that if a content is indexed by cid.contact, then it’s reachable almost everywhere, but if it’s not indexed - it’s not reachable always.

    I couldn’t figure out how to index my content in cid.contact. Honestly, I’m not sure I wanted to. Because what’s the point? it seems to just add a dependency on a centralized service. I could try to run my own indexer and define it in my website, but again, centralization. What’s the economic model for these indexers? Which actors do we expect to run them in the long term?

    The text in cid.contact says that at the current size of IPFS, it’s unreasonable to expect the DHT to handle routing efficiently on its own. This kind of makes sense, but what’s the alternative to that, that doesn’t break up the technology pros?

By now I got tired of the constant struggle for my IPFS blog to function well. At least for a short while, I want a simple, classic working solution. The blog you’re reading now is built with Jekyll and is hosted on my own 10$ server.

don’t get me wrong, I’m still an IPFS fanboy. It’s a great project managed very well. It just doesn’t fit a personal blog needs yet.

That said, It’s difficult to follow the constant development and innovation of IPFS or Filecoin without this becoming a day job. Did I miss some trivial solution or a recent innovation? If yes, let me know. There are no comments here yet, but I am available via old-style email (neiman@hackerspace.pl), or in Mastodon (@neiman@mastodon.social).