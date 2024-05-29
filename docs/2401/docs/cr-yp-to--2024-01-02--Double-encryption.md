<!--yml
category: 未分类
date: 2024-05-27 14:28:54
-->

# cr.yp.to: 2024.01.02: Double encryption

> 来源：[https://blog.cr.yp.to/20240102-hybrid.html](https://blog.cr.yp.to/20240102-hybrid.html)

# The cr.yp.to [blog](index.html)

* * *

<details><summary>Table of contents (Access-I for index page)</summary>

| **2024.01.02: Double encryption:** Analyzing the NSA/GCHQ arguments against hybrids. #nsa #quantification #risks #complexity #costs |
| [**2023.11.25: Another way to botch the security analysis of Kyber-512:**](20231125-kyber.html) Responding to a recent blog post. #nist #uncertainty #errorbars #quantification |
| [**2023.10.23: Reducing "gate" counts for Kyber-512:**](20231023-clumping.html) Two algorithm analyses, from first principles, contradicting NIST's calculation. #xor #popcount #gates #memory #clumping |
| [**2023.10.03: The inability to count correctly:**](20231003-countcorrectly.html) Debunking NIST's calculation of the Kyber-512 security level. #nist #addition #multiplication #ntru #kyber #fiasco |
| [**2023.06.09: Turbo Boost:**](20230609-turboboost.html) How to perpetuate security problems. #overclocking #performancehype #power #timing #hertzbleed #riskmanagement #environment |
| [**2022.08.05: NSA, NIST, and post-quantum cryptography:**](20220805-nsa.html) Announcing my second lawsuit against the U.S. government. #nsa #nist #des #dsa #dualec #sigintenablingproject #nistpqc #foia |
| [**2022.01.29: Plagiarism as a patent amplifier:**](20220129-plagiarism.html) Understanding the delayed rollout of post-quantum cryptography. #pqcrypto #patents #ntru #lpr #ding #peikert #newhope |
| [**2020.12.06: Optimizing for the wrong metric, part 1: Microsoft Word:**](20201206-msword.html) Review of "An Efficiency Comparison of Document Preparation Systems Used in Academic Research and Development" by Knauff and Nejasmic. #latex #word #efficiency #metrics |
| [**2019.10.24: Why EdDSA held up better than ECDSA against Minerva:**](20191024-eddsa.html) Cryptosystem designers successfully predicting, and protecting against, implementation failures. #ecdsa #eddsa #hnp #lwe #bleichenbacher #bkw |
| [**2019.04.30: An introduction to vectorization:**](20190430-vectorize.html) Understanding one of the most important changes in the high-speed-software ecosystem. #vectorization #sse #avx #avx512 #antivectors |
| [**2017.11.05: Reconstructing ROCA:**](20171105-infineon.html) A case study of how quickly an attack can be developed from a limited disclosure. #infineon #roca #rsa |
| [**2017.10.17: Quantum algorithms to find collisions:**](20171017-collisions.html) Analysis of several algorithms for the collision problem, and for the related multi-target preimage problem. #collision #preimage #pqcrypto |
| [**2017.07.23: Fast-key-erasure random-number generators:**](20170723-random.html) An effort to clean up several messes simultaneously. #rng #forwardsecrecy #urandom #cascade #hmac #rekeying #proofs |
| [**2017.07.19: Benchmarking post-quantum cryptography:**](20170719-pqbench.html) News regarding the SUPERCOP benchmarking system, and more recommendations to NIST. #benchmarking #supercop #nist #pqcrypto |
| [**2016.10.30: Some challenges in post-quantum standardization:**](20161030-pqnist.html) My comments to NIST on the first draft of their call for submissions. #standardization #nist #pqcrypto |
| [**2016.06.07: The death of due process:**](20160607-dueprocess.html) A few notes on technology-fueled normalization of lynch mobs targeting both the accuser and the accused. #ethics #crime #punishment |
| [**2016.05.16: Security fraud in Europe's "Quantum Manifesto":**](20160516-quantum.html) How quantum cryptographers are stealing a quarter of a billion Euros from the European Commission. #qkd #quantumcrypto #quantummanifesto |
| [**2016.03.15: Thomas Jefferson and Apple versus the FBI:**](20160315-jefferson.html) Can the government censor how-to books? What if some of the readers are criminals? What if the books can be understood by a computer? An introduction to freedom of speech for software publishers. #censorship #firstamendment #instructions #software #encryption |
| [**2015.11.20: Break a dozen secret keys, get a million more for free:**](20151120-batchattacks.html) Batch attacks are often much more cost-effective than single-target attacks. #batching #economics #keysizes #aes #ecc #rsa #dh #logjam |
| [**2015.03.14: The death of optimizing compilers:**](20150314-optimizing.html) Abstract of my tutorial at ETAPS 2015\. #etaps #compilers #cpuevolution #hotspots #optimization #domainspecific #returnofthejedi |
| [**2015.02.18: Follow-You Printing:**](20150218-printing.html) How Equitrac's marketing department misrepresents and interferes with your work. #equitrac #followyouprinting #dilbert #officespaceprinter |
| [**2014.06.02: The Saber cluster:**](20140602-saber.html) How we built a cluster capable of computing 3000000000000000000000 multiplications per year for just 50000 EUR. #nvidia #linux #howto |
| [**2014.05.17: Some small suggestions for the Intel instruction set:**](20140517-insns.html) Low-cost changes to CPU architecture would make cryptography much safer and much faster. #constanttimecommitment #vmul53 #vcarry #pipelinedocumentation |
| [**2014.04.11: NIST's cryptographic standardization process:**](20140411-nist.html) The first step towards improvement is to admit previous failures. #standardization #nist #des #dsa #dualec #nsa |
| [**2014.03.23: How to design an elliptic-curve signature system:**](20140323-ecdsa.html) There are many choices of elliptic-curve signature systems. The standard choice, ECDSA, is reasonable if you don't care about simplicity, speed, and security. #signatures #ecc #elgamal #schnorr #ecdsa #eddsa #ed25519 |
| [**2014.02.13: A subfield-logarithm attack against ideal lattices:**](20140213-ideal.html) Computational algebraic number theory tackles lattice-based cryptography. |
| [**2014.02.05: Entropy Attacks!**](20140205-entropy.html) The conventional wisdom says that hash outputs can't be controlled; the conventional wisdom is simply wrong. |</details> 

* * *

## 2024.01.02: Double encryption: Analyzing the NSA/GCHQ arguments against hybrids. #nsa #quantification #risks #complexity #costs

In 2019, Google and Cloudflare ran an experiment where they upgraded HTTPS for many ["real users’ connections"](https://blog.cloudflare.com/towards-post-quantum-cryptography-in-tls/) to use post-quantum encryption. They then reported [statistics](https://blog.cloudflare.com/the-tls-post-quantum-experiment/) on how affordable this was. Google had also run a similar [experiment](https://security.googleblog.com/2016/07/experimenting-with-post-quantum.html) in 2016: "While it's still very early days for quantum computers, we're excited to begin preparing for them, and to help ensure our users' data will remain secure long into the future."

Sounds great! Except that, oops, the 2016 experiment ran into [patent trouble](20220129-plagiarism.html). And, oops, half of the post-quantum connections in the 2019 experiment used SIKE, which in 2022 was shown by [public](https://eprint.iacr.org/2022/975) [attacks](https://eprint.iacr.org/2022/1026) to be efficiently breakable.

Upgrading connections to SIKE wasn't just failing to protect those connections against future quantum computers. It was encrypting the connections using an algorithm broken by *today's* computers. The only reason this wasn't giving the user data away to today's attackers is that these experiments were using "hybrids": continuing to encrypt with elliptic-curve cryptography while *adding* a layer of encryption with the new algorithm. The elliptic-curve cryptography was [X25519](https://ianix.com/pub/curve25519-deployment.html), which was already the ["most commonly used key exchange algorithm"](https://blog.cloudflare.com/towards-post-quantum-cryptography-in-tls/) in 2019 and today is used in the ["vast majority"](https://eprint.iacr.org/2023/734.pdf) of TLS connections.

Post-quantum deployment today has progressed far beyond experiments. OpenSSH upgraded in April 2022 to ["the hybrid Streamlined NTRU Prime + x25519 key exchange method by default"](https://www.openssh.com/txt/release-9.0). Google upgraded its internal communication in November 2022 to a hybrid of ["NTRU-HRSS and X25519"](https://cloud.google.com/blog/products/identity-security/why-google-now-uses-post-quantum-cryptography-for-internal-comms), and upgraded Chrome in August 2023 to support ["X25519Kyber768"](https://blog.chromium.org/2023/08/protecting-chrome-traffic-with-hybrid.html) in TLS.

All of these upgrades use a post-quantum layer to *try* to protect today's user data against future quantum computers, and at the same time keep a pre-quantum layer so that they don't *lose* security if something goes wrong with the post-quantum system.

This shouldn't be controversial. However, [NSA](https://nsarchive2.gwu.edu/NSAEBB/NSAEBB441/) stated in May 2022 that it ["does not expect to approve"](https://web.archive.org/web/20220524232249/https://twitter.com/mjos_crypto/status/1433443198534361101/photo/1) hybrids. After the July 2022 SIKE break (see also the section on hybrids in my August 2022 blog post ["NSA, NIST, and post-quantum cryptography"](20220805-nsa.html)), NSA issued a [September 2022 FAQ](https://web.archive.org/web/20220908002357/https://media.defense.gov/2022/Sep/07/2003071836/-1/-1/0/CSI_CNSA_2.0_FAQ_.PDF) that retreats slightly from "does not expect to approve" to "will not require". NSA's FAQ continues *discouraging* hybrids.

It's important to realize that NSA's public positions have a major influence on the cryptographic market. The United States [military budget](https://en.wikipedia.org/wiki/Military_budget_of_the_United_States) is approaching a trillion dollars a year, and NSA [controls](https://web.archive.org/web/20221022163808/https://www.jcs.mil/Portals/36/Documents/Library/Instructions/CJCSI%206510.02F.pdf?ver=qUEnOsWpGPcGGMFTb4yYVA%3D%3D) the cryptographic part of that purchasing.

This doesn't mean that NSA has unlimited power. See, e.g., the quotes [above](#x25519deployment) regarding X25519 deployment in TLS. [Pretty](https://www.ssllabs.com/ssltest/viewClient.html?name=Chrome&version=80&platform=Win%2010&key=170) [much](https://www.ssllabs.com/ssltest/viewClient.html?name=Firefox&version=73&platform=Win%2010&key=171) [every](https://www.ssllabs.com/ssltest/viewClient.html?name=Safari&version=12.1.2&platform=MacOS%2010.14.6%20Beta&key=161) [browser](https://www.ssllabs.com/ssltest/viewClient.html?name=Edge&version=18&platform=Win%2010&key=160) and [server](https://www.ssllabs.com/ssltest/) supports X25519 and uses it by default, although there are exceptions, such as [nsa.gov](https://www.ssllabs.com/ssltest/analyze.html?d=nsa.gov) preferring P-256. As for hybrids, NIST and other standardization organizations can and should **require** upgrades to use hybrids, for example with the following language for key-encapsulation mechanisms (KEMs): "Any upgrade from pre-quantum encryption to this KEM shall retain the pre-quantum encryption and add this KEM as an extra layer of defense, rather than removing the pre-quantum encryption."

In this blog post, I'll look at NSA's arguments against hybrids. I'll also look at the anti-hybrid arguments in a [statement](https://web.archive.org/web/20231104161635/https://www.ncsc.gov.uk/whitepaper/next-steps-preparing-for-post-quantum-cryptography) from NSA's [friends](https://www.theguardian.com/uk-news/2013/aug/01/nsa-paid-gchq-spying-edward-snowden) at [GCHQ](https://www.theguardian.com/uk-news/2021/may/25/gchqs-mass-data-sharing-violated-right-to-privacy-court-rules).

**The NSA anti-hybrid arguments.** Let's consider the following quotes from NSA's September 2022 FAQ:

*   "Hybrids add complexity to protocols, as designers need to incorporate additional negotiation and error handling."

    This is not true. A protocol using a signature system doesn't care that the signature system is (1) internally using a pre-quantum signature system and a post-quantum signature system and (2) internally requiring both of those signatures to pass verification. Similarly, a protocol using a KEM doesn't care that the KEM is internally hashing together pre-quantum and post-quantum session keys.

    One *can* instead support hybrids at the protocol layer, and this can even have some advantages; but this doesn't require additional negotiation, doesn't require additional error handling, and, most importantly, isn't required in the first place.

*   "Hybrid deployments introduce additional interoperability concerns, now that all algorithms plus the method of hybridization must be features common to all parties to a communication."

    This is also not true. Upgrades require paying attention to interoperability, but upgrading to a hybrid KEM works the same way as upgrading to a non-hybrid KEM. The OpenSSH upgrade to `sntrup761x25519-sha512` worked the same way that an upgrade to a hypothetical non-hybrid `sntrup761-sha512` would have worked.

*   "Many hybrid solutions being developed externally do not facilitate the transition to strictly-QR solutions, so implementers would need to anticipate an additional significant transition from hybrid to QR as classical algorithms lose their utility."

    "QR" is jargon for "quantum-resistant", which in turn is the marketing department falsely claiming that the *goal* is an established *fact*. But let's read this as "post-quantum" and focus on what the "transition" argument is saying.

    Here's the scenario to think about. Imagine public demonstrations of low-cost quantum attacks showing that X25519 has negligible security value. Imagine the community then deciding to move from `sntrup761x25519-sha512` to just `sntrup761-sha512`. An initial transition to `sntrup761x25519-sha512`, followed by a transition to `sntrup761-sha512`, is *two* transitions!

    But what makes this second transition "significant"? It isn't a security upgrade. Yes, software changes take time to test and deploy, but if this second transition is such a problem that we should be insisting on having just one transition, then why should that one transition be to `sntrup761-sha512` rather than to `sntrup761x25519-sha512`?

    NSA's argument here is a puzzling combination of (1) arguing that it's important to minimize the number of transitions and (2) arguing that if there's a transition to a hybrid then it will be important to subsequently transition *away* from the hybrid. This doesn't make sense unless you assume that there's something wrong with hybrids in the first place. In other words, this two-transition argument isn't an independent argument against hybrids.

*   "Perhaps most importantly, hybrid solutions make implementations even more complex, so one must balance the risk of flaws in an increasingly complex implementation with the risk of a cryptanalytic breakthrough."

    The scenario here is much more immediate. Let's say you've designed a cryptographic application using X25519 for encryption. You decide to upgrade from pre-quantum encryption to post-quantum encryption. You sensibly use a hybrid. Along with the existing X25519 code, there's new code for whichever KEM—let's call it UltraKEM. You then compute a hybrid hash: this means hashing the X25519 session key, the UltraKEM session key, the X25519 public keys, and the UltraKEM ciphertext. You use the output of this hash as the new session key.

    Skipping the hybrid would certainly reduce code volume in the long run: once all clients and servers support UltraKEM, you'd be able to eliminate the X25519 code and the code for the hybrid hash.

    Could the X25519 code have bugs? Yes. X25519 is [easier](https://cr.yp.to/papers.html#nistecc) to implement correctly than NSA's ECDH using P-256, but this doesn't mean that the risk of someone implementing X25519 incorrectly is zero.

    But why is NSA not mentioning the risk of bugs in the UltraKEM software? Isn't this *obviously* a bigger risk?

    The UltraKEM software is newer and more complicated than the X25519 software. Any reasonable risk assessment will conclude that the probability of an exploitable arithmetic bug or timing leak or randomness-usage mistake in the UltraKEM software is higher than the probability of such a exploit in the X25519 software. Furthermore, if the first type of exploit happens, its impact is destroying the security of UltraKEM (user data is exposed today), while an X25519+UltraKEM hybrid reduces the damage (user data is exposed to a future quantum computer). If the second type of exploit happens, its impact is merely dropping the security of X25519+UltraKEM down to the security of UltraKEM.

    Consider [KyberSlash](https://kyberslash.cr.yp.to), a timing variation that was in [most](https://kyberslash.cr.yp.to/libraries.html) Kyber libraries at the beginning of December 2023. I posted a [demo](https://groups.google.com/a/list.nist.gov/g/pqc-forum/c/ldX0ThYJuBo/m/uIOqRF5BAwAJ) on 30 December 2023 recovering Kyber's complete secret key from timings of the beginning-of-December-2023 Kyber reference code. Are we supposed to believe that this is the last vulnerability in post-quantum software?

    Let's be clear about what NSA needs to have happen to support its scenario of what I'll call a *hybrid reversal*, namely the X25519+UltraKEM software being *less* secure than the UltraKEM software. There needs to be a program-partitioning disaster in the X25519 software, such as an exploitable buffer overflow; or there needs to be some similarly devastating bug in the lines of code for the hybrid hash, such as omitting the UltraKEM session key from the hash input.

    We have tools automatically checking that there's no data flow from the X25519 input to any branch conditions or array indices, so how exactly is a buffer overflow supposed to happen? As for a bug in the hybrid hash, how is this supposed to slip past interoperability testing?

    I'm not saying a hybrid reversal is impossible to achieve. But NSA's wording ("risk of flaws in an increasingly complex implementation") makes the reader think about the overall amount of code for an X25519+UltraKEM hybrid, and suggests, incorrectly, that any flaws in this code are an argument against hybrids. NSA doesn't acknowledge the possibility of flaws in the UltraKEM code making UltraKEM less secure than the hybrid, as illustrated by KyberSlash. NSA also doesn't distinguish between (1) the possibility of flaws in the X25519 code reducing the security of X25519+UltraKEM to the security of UltraKEM and (2) the possibility of a hybrid reversal.

    Maybe NSA will respond that, oh, sorry, it wasn't thinking about bugs in the post-quantum software, and didn't know about KyberSlash, and, really, how common can this sort of thing be? But a separate document shows that, for some "national security" data, NSA uses two independent encryption layers ["to mitigate the ability of an adversary to exploit a single cryptographic implementation"](https://web.archive.org/web/20220524232250/https://www.nsa.gov/Portals/75/documents/resources/everyone/csfc/threat-prevention.pdf). How does NSA reconcile this with its FAQ not even mentioning the risk of an exploit of a post-quantum implementation?

    Meanwhile the word "breakthrough" in NSA's FAQ suggests that mathematical breaks of post-quantum systems are rare; and NSA's word "balance" suggests that the risks of those breaks are no higher than the risks from hybrid reversals. Certainly a problem in cryptographic software can be as devastating as a mathematical break (and there are cases where patching a [software problem](https://cr.yp.to/papers.html#cachetiming) is [harder](https://eprint.iacr.org/2019/996) than switching to a different cryptosystem); but why should we think that the *probability* of a hybrid reversal is anywhere near the probability of post-quantum systems being broken?

*   "More security products fail due to implementation or configuration errors than failures in their underlying cryptographic algorithms. Therefore, spending limited resources to add cryptographic complexity can potentially weaken security."

    This needs clarification. What exactly is included in "products"? Is NSA claiming that vulnerabilities in non-cryptographic code somehow count as arguments against hybrids? If the focus is on cryptography: is NSA excluding, e.g., the 1990s, when ["95%"](https://link.springer.com/chapter/10.1007/3-540-45539-6_1) of SSL connections used RSA-512, which was breakable no matter how competent the implementation was? Also, are failures weighted by severity? What's the weight assigned to, e.g., the [Flame malware](https://en.wikipedia.org/wiki/Flame_(malware)), which was discovered in 2012 and exploited an MD5 weakness? Also, how is keeping the existing X25519 code supposed to be an example of "spending limited resources to add cryptographic complexity"? Is NSA talking about the effort to write the code for the hybrid hash?

**Quantifying cryptographic risks.** Let's look more closely at the notion that a mathematical break of a post-quantum system would be a "breakthrough".

As an analogy, what would you think of someone using the word "breakthroughs" for discoveries of bugs in published software? Shockingly ignorant, right?

There's broad awareness that programming is an error-prone activity. There's ample data quantifying this. Big software projects track their [bugs](https://bugzilla.mozilla.org/describecomponents.cgi?product=Firefox). Cross-cutting papers [study](https://www.vuminhle.com/pdf/issta16.pdf) bugs in multiple projects, [study](https://www.alexopoulos.ch/files/TOPS2020.pdf) bug discoveries across time, [study](https://arxiv.org/pdf/2103.07189.pdf) the impact of different software-engineering practices upon bug rates, etc. Programmer beliefs are often [contradicted](https://www.cs.ucdavis.edu/~devanbu/belief+evidence.pdf) by studies.

Is there broad awareness that the processes of designing and selecting post-quantum cryptosystems are error-prone? Readers might occasionally hear news about a broken cryptosystem, or might bump into a paper describing a break, but how are readers supposed to figure out how *common* this is? Where are the studies of how often cryptosystems fail?

NSA stated in 2021 that ["NSA has confidence in the NIST PQC process"](https://datatracker.ietf.org/meeting/112/materials/slides-112-lamps-hybrid-non-composite-multi-certificate-00). Typical readers of NSA's 2022 FAQ will end up with the impression that hybrids are dealing with a hypothetical failure mode rather than a real failure mode. A "breakthrough" doesn't sound like something to be expected. NSA doesn't mention SIKE. A reader who has heard separately about SIKE could easily think that it's an isolated example.

Maybe you're thinking something like this: "Isn't it in fact an isolated example? Aren't almost all post-quantum proposals holding up just fine? Otherwise, wouldn't I have heard more news about breaks?"

I have a new paper, titled ["Quantifying risks in cryptographic selection processes"](https://cr.yp.to/papers.html#qrcsp), that takes NIST's post-quantum competition as a case study:

*   Out of the 69 round-1 submissions to the competition in 2017: **48%** are broken by now, meaning that the smallest proposed parameters are now known to be easier to break than AES-128. (AES-128 was the minimum security level allowed in the competition.)

*   Out of the 48 submissions that were unbroken during round 1: **25%** are broken by now.

*   Out of the 28 submissions *selected by NIST in 2019 for round 2*: **36%** are broken by now.

These are shockingly high failure rates. No, SIKE isn't an isolated example.

Don't let yourself be fooled by confident-sounding salesmen. If you run into people claiming that selection mechanism S reliably selects cryptosystems that won't be broken within 10 years, try asking for references to (1) the publication of a clear definition of mechanism S and (2) the publication, at least 10 years later, of a study of the quantitative failure rate of mechanism S. People aren't demonstrating any predictive power if they merely point to an unbroken system and claim reasons for its security. Keep in mind that a *breakable* system is *unbroken* until enough time has been spent to find the attack algorithm, and incorrect claims of its security in the meantime can be generated by pure confirmation bias.

**Quantifying cryptographic code complexity.** Another unquantified aspect of NSA's text is its claim that "hybrid solutions make implementations even more complex".

If the words are taken purely at face value then they're clearly correct, and just as clearly useless for making decisions. Yes, X25519+UltraKEM software is more complex than UltraKEM software; so what? UltraKEM software is more complex than NullKEM software; is that supposed to be an argument for NullKEM? (NullKEM has empty keys, empty ciphertexts, and session keys where every bit is 0.)

What the reader understands NSA to be saying is that there's an *important* difference in complexity between X25519+UltraKEM software and UltraKEM software. The statement from GCHQ, covered [below](#gchq), explicitly says that there is "significantly" more complexity. This begs a quantitative question: how much software are we talking about?

I have another new paper, titled ["Analyzing the complexity of reference post-quantum software"](https://cr.yp.to/papers.html#pqcomplexity), that takes the reference software for the following lattice-based KEMs as case studies: `kyber512`, `kyber768`, `kyber1024`, `ntruhps2048509`, `ntruhps2048677`, `ntruhps4096821`, `ntruhrss701`, `sntrup653`, `sntrup761`, `sntrup857`, `sntrup953`, `sntrup1013`, and `sntrup1277`.

The paper applies consistent rules to streamline the reference software for each KEM, ending up with, e.g., 381 lines for `ntruhps4096821`, 385 lines for `ntruhrss701`, 472 lines for `kyber1024`, and 478 lines for `sntrup1277`. There are also subroutines for hashing, such as a 67-line `hash/sha3256`, and for constant-time sorting (in the case of `ntruhps` and `sntrup`), such as a 33-line `sort/uint32`. The paper includes further metrics (e.g., cyclomatic complexity, which is a snobbish way to talk about counting functions plus branches), combinations of multiple KEM sizes (e.g., 574 lines for `kyber512` plus `kyber1024`), and in-depth analyses of how the lines of code are being spent.

For comparison, extracting the X25519 software from [TweetNaCl](https://tweetnacl.cr.yp.to), and reformatting it the same way as in the new paper, produces 156 lines, which is under half the size of any of these lattice KEMs. Sure, I should also count the lines of code for a hybrid hash, but these numbers easily justify what I wrote [above](#morecomplicated) about a generic post-quantum UltraKEM: "The UltraKEM software is newer and more complicated than the X25519 software."

**Quantifying complexity of fast code.** One can criticize this code-size comparison as being only for *simple* code. What about an application that switches to *fast* code?

For X25519, there are many microarchitecture-specific optimizations available in [lib25519](https://lib25519.cr.yp.to/speed.html), but let's assume the application is satisfied with the performance of John Harrison's Intel/AMD assembly-language [software](https://github.com/awslabs/s2n-bignum/blob/main/x86/curve25519/curve25519_x25519.S), which is 2197 lines, or the equivalent [software](https://github.com/awslabs/s2n-bignum/blob/main/arm/curve25519/curve25519_x25519.S) for ARM. These come with formally verified proofs in HOL Light that the software [correctly](https://github.com/awslabs/s2n-bignum/blob/main/arm/proofs/curve25519_x25519.ml) [computes](https://github.com/awslabs/s2n-bignum/blob/main/x86/proofs/curve25519_x25519.ml) X25519 on all inputs, so the length of the code isn't a correctness concern: it simply reflects the difficulty of writing (and verifying) the code in the first place.

If I've counted correctly, the Kyber-768 AVX2 software in [libjade](https://github.com/formosa-crypto/libjade/tree/main/src/crypto_kem/kyber) is 4589 lines plus the hashing subroutines. (This is [mostly](https://eprint.iacr.org/2023/215) verified.) The line counts aren't directly comparable since the code style is somewhat different, but I think a direct comparison would again show that the fast X25519 code is under half the complexity of the fast code for a single Kyber parameter set.

Note that formal verification of software correctness changes the risk analysis of hybrids: to the extent it's used, it reduces the chance of software bugs, so it increases the relative weight that has to be placed on mathematical breaks. (The X25519 code from TweetNaCl is also [verified](https://eprint.iacr.org/2021/428).)

**The GCHQ anti-hybrid arguments.** Let's move on to considering quotes from GCHQ's November 2023 statement:

*   "There are greater costs to PQ/T hybrid schemes than those with a single algorithm. PQ/T hybrid schemes will be more complex to implement and maintain and will also be less efficient."

    The "more complex" part is echoing NSA and is covered [above](#complexity). I'll say more [below](#efficiency) about the "less efficient" part. "PQ/T" is jargon for "post quantum/traditional".

*   "PQ/T hybrid key establishment mechanisms should be designed carefully to ensure the hybridisation mechanism does not allow additional attacks. As of November 2023, advice on how to do this is currently in development by ETSI. [Next paragraph:] Proposed PQ/T hybrid schemes for authentication can be significantly more complex than those used for confidentiality, due to the need to make sure both signatures verify in a robust way. There has also been significantly less research activity into PQ/T schemes for authentication than for confidentiality, and there is not yet guidance or a consensus on how to do this in a secure way."

    Um, what?

    Sign the message with both signature systems. On the verifier side, accept the message only if both signatures pass verification.

    If the words "there is not yet guidance" are supposed to be a claim that this hasn't been written down: see, e.g., Section 3.3 of [PQCRYPTO deliverable 2.5](https://www.pqcrypto.eu/deliverables/d2.5.pdf) from 2018. The reason there's no research activity here is that this is a trivially solved problem.

    For encryption, simple hashing solutions are presented in ["KEM combiners"](https://eprint.iacr.org/2018/024), also from 2018. We don't need to wait for "development by ETSI".

*   "PQ/T hybrid authentication within a PKI requires either a PKI which can generate and sign traditional and post-quantum digital signatures, or two parallel PKIs (one for traditional and one for post-quantum digital signatures). This additional complexity and the difficulty in migrating PKIs mean that a single migration to a fully post-quantum PKI is preferred to adopting an intermediate PQ/T hybrid PKI."

    Migrating a PKI from, say, Ed25519 signatures to Ed25519+UltraSign signatures works the same way as migrating a PKI from Ed25519 signatures to non-hybrid UltraSign signatures. Regarding "single migration", see [above](#transition) regarding NSA's "transition" argument.

*   "In the future, if a CRQC exists, traditional PKC algorithms will provide no additional protection against an attacker with a CRQC. At this point, a PQ/T hybrid scheme will provide no more security than a single post-quantum algorithm but with significantly more complexity and overhead."

    The assumption that there's a CRQC (this is jargon for "cryptographically relevant quantum computer") doesn't justify the conclusion that pre-quantum cryptography provides "no additional protection" and "no more security than a single post-quantum algorithm".

    Concretely, think about a demo showing that spending a billion dollars on quantum computation can break a thousand X25519 keys. Yikes! We should be aiming for much higher security than that! We don't even want a billion-dollar attack to be able to break *one* key! Users who care about the security of their data will be happy that we deployed post-quantum cryptography. But are the users going to say "Let's turn off X25519 and make each session a million dollars cheaper to attack"? I'm skeptical. I think users will need to see much cheaper attacks before agreeing that X25519 has negligible security value.

    On the flip side, "complexity" is covered [above](#complexity), and I'll say more [below](#efficiency) about "overhead".

*   "With this in mind, technical system and risk owners should weigh the reasons for and against PQ/T hybrid schemes including interoperability, implementation security, and protocol constraints, as well as the complexity, cost of maintaining a more complex system, and the need to complete the migration twice (once to a PQ/T hybrid scheme and again to PQC-only algorithms as a future end state)."

    There are no new anti-hybrid arguments here.

**Quantifying cryptographic costs.** Let's now consider GCHQ's unquantified efficiency claims.

Obviously the continued use of X25519 has *some* overhead: 32 bytes more in the public key, 32 bytes more in the ciphertext, and some number of CPU cycles. But the real question is whether these costs matter for the application.

I have another new paper, titled ["Predicting performance for post-quantum encrypted-file systems"](https://cr.yp.to/papers.html#pppqefs), that looks at the current use of public-key cryptography to encrypt stored files (for example, in Microsoft's EFS), and predicts the dollar cost of various post-quantum KEMs in this application.

Part of the paper is collecting data on the purchase costs of storage, communication, and CPU time, to be able to convert bytes and cycles into dollars. In particular, Internet data transmission costs roughly 2^(−40) dollars per byte, and computation costs roughly 2^(−51) dollars per CPU cycle. That paper focuses on comparing the costs added by various post-quantum KEMs in one application, but the purchase costs of bytes and cycles are application-independent and also shed light on the question of whether the cost of a hybrid can be a problem.

A server sending 32 bytes and receiving 32 bytes for an X25519 key exchange incurs data-communication costs of roughly 2^(−34) dollars. The software mentioned [above](#x25519assembly) from John Harrison takes about 2^(17) cycles for key generation plus shared-secret generation, so also roughly 2^(−34) dollars, for a total cost of roughly 2^(−33) dollars.

Saying that the server can do billions of X25519 key exchanges for a dollar doesn't end the analysis: it begs the question of how many key exchanges the application is carrying out. But let's compare this cost to the cost of a Kyber key exchange.

Sending an 800-byte key and receiving a 768-byte ciphertext for the smallest Kyber option, Kyber-512, costs roughly 2^(−29) dollars. Including X25519 adds roughly 7% to that.

Are we supposed to believe that an application can afford the cost of Kyber-512 but *can't* afford to multiply that cost by 1.07? Um, ok, it's conceivable, but where's the evidence? How many readers would expect the words "less efficient" to be referring to such a small performance gap? If there's going to be *any* comment on such a small gap, how about *stating the numbers* so that the reader isn't misled?

Maybe GCHQ will try to defend its claim by pointing to slower X25519 implementations, such as the TweetNaCl implementation mentioned [above](#tweetnacl). But those implementations are for applications where this slowness is just fine: think of an application where each user is starting "only" a million sessions. These applications make GCHQ's cost argument even weaker.

**The importance of quantification.** I'll close this blog post with some observations on the role of unquantified claims inside the NSA/GCHQ arguments.

The most important message that readers pick up from NSA's text is that breaks of post-quantum systems are rare. I've given numbers to the contrary. NSA could respond that, well, each of those breaks was a "breakthrough" (first definition I found in a [dictionary](https://www.merriam-webster.com/dictionary/breakthrough): "a sudden advance especially in knowledge or technique"), and that NSA's text wasn't actually claiming anything about *how often* this happens.

In other words, because the word "breakthrough" isn't quantified, it's unfalsifiable. But the word plays a central role in NSA's FAQ: it influences perceptions of the risk of post-quantum systems being broken, and influences decision-making processes regarding hybrids.

A useful defense mechanism for readers is to watch for statements that *could* have been quantified but *aren't*. If NSA honestly believes that post-quantum systems are rarely broken, why isn't NSA quantifying this? If NSA honestly believes that hybrids pose important complexity concerns, why isn't NSA quantifying the complexity? If GCHQ honestly believes that hybrids pose important cost concerns, why isn't GCHQ quantifying those costs?

Sure, these are multi-billion-dollar spy agencies with an incentive to weaken cryptography, but that's not the point here. It's useful to ask the same question if you see an academic claiming that post-quantum systems are rarely broken: why is the claim not quantified?

The actual information regarding breaks and complexity and costs is of obvious importance for cryptographic decisions. Unfortunately, this information is scattered through the cryptographic literature. Most cryptosystem breaks aren't tracked centrally. There are many papers with byte counts and cycle counts for specific operations, but it's much less common to see costs quantified in context. There are comments here and there on code complexity, but this is very far from systematic. It's *possible* for skeptical readers to collect the numbers that NSA and GCHQ are omitting, but this takes work.

Early in the NIST post-quantum competition, Tanja Lange and I prepared an overview slide showing which submissions had been broken. We included [updated](https://cr.yp.to/talks/2017.12.28/slides-dan+nadia+tanja-20171228-latticehacks-16x9.pdf) [versions](https://cr.yp.to/talks/2018.12.14/slides-dan+tanja-20181214-pqcrypto-4x3.pdf) [in](https://cr.yp.to/talks/2018.12.28/slides-dan+tanja-20181228-pqcrypto-16x9.pdf) [a](https://cr.yp.to/talks/2019.06.10/slides-dan+tanja-20190610-pqcrypto-4x3.pdf) [series](https://cr.yp.to/talks/2020.02.06/slides-dan+tanja-20200206-horror-16x9.pdf) [of](https://cr.yp.to/talks/2020.09.12/slides-dan+tanja-20200912-pqcrypto-16x9.pdf) [talks](https://cr.yp.to/talks/2022.12.29/slides-dan+tanja-20221229-pqcrypto-16x9.pdf). After new post-quantum signature systems were submitted to NIST in 2023, Thom Wiggers assembled a [signature zoo](https://pqshield.github.io/nist-sigs-zoo/) tracking which of those have been broken (along with key sizes etc.). There are other examples of cryptosystem tracking. I'd like this to be much more systematic, so that readers can easily find data on, e.g., the chance of a cryptosystem break escaping detection for N years. There have to be clear specifications of what counts as a break—and obviously this can't be limited to attack *demos*; the goal is security against large-scale attackers, not just security against academic experiments. There has to be clear tracing so that the data collection can be spot-checked.

I have the luxury of being able to take time to investigate topics that I think are important, without worrying about whether this produces papers fitting the scope of existing conferences or journals. So I have the aforementioned papers with case studies of quantifying cryptographic [risks](https://cr.yp.to/papers.html#qrcsp) and [code complexity](https://cr.yp.to/papers.html#pqcomplexity) and [dollar costs](https://cr.yp.to/papers.html#pppqefs). But I'd like to see much more work on these topics. I'd like to see the cryptographic community making clear to junior researchers that the cryptographic community cares about these topics and is providing venues for publishing papers on these topics.

Code complexity and dollar costs are traditional engineering topics, and should fit easily into existing venues for cryptographic engineering, such as [CHES](https://ches.iacr.org/) and the [Journal of Cryptographic Engineering](https://link.springer.com/journal/13389). But what about cryptographic risk analysis? Risk analysis is already a common topic in the software-engineering literature, but I don't think typical software-engineering venues would consider mathematical breaks of cryptosystems to be within scope.

One possibility is the Journal of Cryptology. My paper on [cryptographic competitions](https://cr.yp.to/papers.html#competitions) is a risk-analysis paper in that journal. Another possibility is IACR's new journal, [IACR Communications in Cryptology](https://cic.iacr.org/), which explicitly includes "applied aspects of cryptography" in its scope.

Ultimately what matters here is that, as illustrated by the NSA/GCHQ arguments about hybrids, the process of making cryptographic decisions raises quantitative questions. We owe it to the users to carefully investigate those questions.

[2024.01.09 edit: Fixed link to [https://kyberslash.cr.yp.to/libraries.html](https://kyberslash.cr.yp.to/libraries.html).]

* * *

**Version:** This is version 2024.01.09 of the 20240102-hybrid.html web page.