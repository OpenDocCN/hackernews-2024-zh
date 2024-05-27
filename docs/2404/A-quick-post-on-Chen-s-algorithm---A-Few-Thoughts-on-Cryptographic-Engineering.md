<!--yml
category: 未分类
date: 2024-05-27 13:14:30
-->

# A quick post on Chen’s algorithm – A Few Thoughts on Cryptographic Engineering

> 来源：[https://blog.cryptographyengineering.com/2024/04/16/a-quick-post-on-chens-algorithm/](https://blog.cryptographyengineering.com/2024/04/16/a-quick-post-on-chens-algorithm/)

**Update** **(April 19):** *Yilei Chen [announced the discovery of a bug](http://www.chenyilei.net/) in the algorithm, which he does not know how to fix. This was independently discovered by Hongxun Wu and Thomas Vidick. At present, the paper does not provide a polynomial-time algorithm for solving LWE.*

If you’re a normal person — that is, a person who doesn’t obsessively follow the latest cryptography news — you probably missed last week’s cryptography bombshell. That news comes in the form of a new e-print authored by Yilei Chen, “[Quantum Algorithms for Lattice Problems](https://eprint.iacr.org/2024/555)“, which has roiled the cryptography research community. The result is now being evaluated by experts in lattices and quantum algorithm design (*and to be clear, I am not one!*) but if it holds up, it’s going to be quite a bad day/week/month/year for the applied cryptography community.

Rather than elaborate at length, here’s quick set of five bullet-points giving the background.

(1) Cryptographers like to build modern public-key encryption schemes on top of mathematical problems that are believed to be “hard”. In practice, we need problems with a specific structure: we can construct *efficient* solutions for those who hold a secret key, or “trapdoor”, and yet also admit *no efficient solution* for folks who don’t. While many problems have been considered (and often discarded), most schemes we use today are based on three problems: [factoring](https://en.wikipedia.org/wiki/Integer_factorization) (the RSA cryptosystem), [discrete logarithm](https://en.wikipedia.org/wiki/Discrete_logarithm) (Diffie-Hellman, DSA) and [elliptic curve discrete logarithm problem](https://en.wikipedia.org/wiki/Elliptic-curve_cryptography) (EC-Diffie-Hellman, ECDSA etc.)

(2) While we would like to believe our favorite problems are fundamentally “hard”, we know this isn’t really true. Researchers [have devised algorithms](https://en.wikipedia.org/wiki/Shor%27s_algorithm) that solve all of these problems quite efficiently (i.e., in polynomial time) — provided someone figures out how to build a quantum computer powerful enough to run the attack algorithms. *Fortunately such a computer has not yet been built!*

(3) Even though quantum computers are not yet powerful enough to break our public-key crypto, the mere threat of *future* quantum attacks has inspired industry, government and academia to join forces [Fellowship-of-the-Ring-style](https://csrc.nist.gov/projects/post-quantum-cryptography) in order to tackle the problem right now. This isn’t merely about future-proofing our systems: even if quantum computers take decades to build, future quantum computers could break encrypted messages we send *today*!

(4) One conspicuous outcome of this fellowship is [NIST’s Post-Quantum Cryptography (PQC) competition](https://www.nist.gov/news-events/news/2022/07/nist-announces-first-four-quantum-resistant-cryptographic-algorithms): this was an open competition designed to standardize *“post-quantum” cryptographic schemes*. Critically, these schemes must be based on different mathematical problems — most notably, problems that *don’t* seem to admit efficient quantum solutions.

(5) Within this new set of schemes, the most popular class of schemes are based on problems related to mathematical objects called *[lattices](https://en.wikipedia.org/wiki/Lattice-based_cryptography)*. NIST-approved schemes based on lattice problems include [Kyber](https://en.wikipedia.org/wiki/Kyber) and [Dilithium](https://pq-crystals.org/dilithium/) (which I [wrote about recently](https://blog.cryptographyengineering.com/2023/11/30/to-schnorr-and-beyond-part-2/).) Lattice problems are also the basis of several efficient [fully-homomorphic encryption](https://blog.cryptographyengineering.com/2012/01/02/very-casual-introduction-to-fully/) (FHE) schemes.

This background sets up the new result.

Chen’s (not yet peer-reviewed) [preprint](https://eprint.iacr.org/2024/555) claims anew quantum algorithm that efficiently solves the “shortest independent vector problem” (SIVP, as well as GapSVP) in lattices with specific parameters. If it holds up, the result could (with numerous important caveats) allow future quantum computers to break schemes that depend on the hardness of specific instances of these problems. The good news here is that even if the result is correct, the vulnerable parameters are very specific: Chen’s algorithm does not *immediately* apply to the [recently-standardized NIST algorithms](https://www.nist.gov/news-events/news/2022/07/nist-announces-first-four-quantum-resistant-cryptographic-algorithms) such as Kyber or Dilithium. Moreover, the exact concrete complexity of the algorithm is not instantly clear: it may turn out to be impractical to run, even if quantum computers become available.

But there is a saying in our field that *attacks only get better.* If Chen’s result can be improved upon, then quantum algorithms could render obsolete an entire generation of “post-quantum” lattice-based schemes, forcing cryptographers [and](https://security.apple.com/blog/imessage-pq3/) [industry](https://signal.org/blog/pqxdh/) [back](https://bughunters.google.com/blog/5108747984306176/google-s-threat-model-for-post-quantum-cryptography) to the drawing board.

In other words, both a great technical result — and possibly a mild disaster.

As previously mentioned: I am neither an expert in lattice-based cryptography nor quantum computing. The folks who *are* those things are very busy trying to validate the writeup*:* and more than a few big results have fallen apart upon detailed inspection.For those searching for the latest developments, here’s a [nice writeup by Nigel Smart](https://nigelsmart.github.io/LWE.html) that doesn’t tackle the correctness of the quantum algorithm (see updates at the bottom), but does talk about the possible implications for FHE and PQC schemes (TL;DR: bad for some FHE schemes, but really depends on the concrete details of the algorithm’s running time.) And here’s another [brief note on a “bug” that was found in the paper](https://eprint.iacr.org/2024/583.pdf), that seems to have been quickly [addressed by the author.](http://www.chenyilei.net/)

Up until this week I had intended to write another long wonky post about complexity theory, lattices, and what it all meant for applied cryptography. But now I hope you’ll forgive me if I hold onto that one, for just a little bit longer.