<!--yml
category: 未分类
date: 2024-05-27 14:53:35
-->

# Brane Dump: Pwned Certificates on the Fediverse

> 来源：[https://www.hezmatt.org/~mpalmer/blog/2024/01/16/pwned-certificates-on-the-fediverse.html](https://www.hezmatt.org/~mpalmer/blog/2024/01/16/pwned-certificates-on-the-fediverse.html)

Posted: Tue, 16 January 2024 | [permalink](//www.hezmatt.org/~mpalmer/blog/2024/01/16/pwned-certificates-on-the-fediverse.html) | [No comments](//www.hezmatt.org/~mpalmer/blog/2024/01/16/pwned-certificates-on-the-fediverse.html#comments)

As well as the collection and distribution of [compromised keys](https://pwnedkeys.com), the [pwnedkeys](https://pwnedkeys.com) project also matches those pwned keys against issued SSL certificates. I’m excited to announce that, as of the beginning of 2024, all matched certificates are now being published [on the Fediverse](https://botsin.space/@pwnedcerts), thanks to the [botsin.space](https://botsin.space/) Mastodon server.

Want to know which sites are susceptible to interception and interference, in (near-)real time? Do you have a burning desire to know who is issuing certificates to people that post their private keys in public? Now you can.

# How It Works

The process for publishing pwned certs is, roughly, as follows:

1.  All the certificates in [Certificate Transparency](https://certificate.transparency.dev) (CT) logs are hoovered up (using my [scrape-ct-log](https://github.com/mpalmer/scrape-ct-log) tool, the fastest log scraper in the west!), and the fingerprint of the public key of each certificate is stored in an [LMDB](https://lmdb.tech/) datafile.

2.  As new private keys are identified as having been compromised, the fingerprint of that key is checked against all the LMDB files, which map key fingerprints to certificates (actually to CT log entry IDs, from which the certificates themselves are retrieved).

3.  If one or more matches are found, then the certificates using the compromised key are forwarded to the “tooter”, which [publishes them for the world to marvel at](https://botsin.space/@pwnedcerts).

This makes it sound all very straightforward, and it is… in theory. The trick comes in optimising the pipeline so that the five million or so new certificates every day can get indexed on the one slightly middle-aged server I’ve got, without getting backlogged.

# Why Don’t You Just Have the Certificates Revoked?

Funny story about that…

I used to notify CAs of certificates they’d issued using compromised keys, which had the effect of requiring them to revoke the associated certificates. However, several CAs disliked having to revoke all those certificates, because it cost them staff time (and hence money) to do so. They went so far as to change their procedures from the standard way of accepting problem reports (emailing a generic attestation of compromise), and instead required CA-specific hoop-jumping to notify them of compromised keys.

Since the effectiveness of revocation in the WebPKI is, shall we say, “homeopathic” at best, I decided I couldn’t be bothered to play whack-a-mole with CAs that just wanted to be difficult, and I stopped sending compromised key notifications to CAs. Instead, now I’m publishing the details of compromised certificates to everyone, so that users can protect themselves directly should they choose to.

# Further Work

The astute amongst you may have noticed, in the above “How It Works” description, a bit of a gap in my scanning coverage. CAs can (and do!) issue certificates for keys that are *already* compromised, including “weak” keys that have been known about for a decade or more ([1](https://bugzilla.mozilla.org/show_bug.cgi?id=1472052), [2](https://bugzilla.mozilla.org/show_bug.cgi?id=1620772), [3](https://bugzilla.mozilla.org/show_bug.cgi?id=1789521)). However, as currently implemented, the pwnedkeys certificate checker does not automatically find such certificates.

My plan is to augment the CT scraping / cert processing pipeline to check all incoming certificates against the existing (2M+) set of pwned keys. Though, with over five million new certificates to check every day, it’s not necessarily as simple as “just hit the [pwnedkeys API](https://pwnedkeys.com/api/v1.html) for every new cert”. The poor old API server might not like that very much.

# Support My Work

If you’d like to see this extra matching happen a bit quicker, I’ve setup a [ko-fi supporters page](https://ko-fi.com/tobermorytech), where you can support my work on [pwnedkeys](https://pwnedkeys.com) and the other open source software and projects I work on by [buying me a refreshing beverage](https://ko-fi.com/tobermorytech). I would be very appreciative, and your support lets me know I should do more interesting things with the giant database of compromised keys I’ve accumulated.

* * *