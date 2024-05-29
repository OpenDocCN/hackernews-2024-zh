<!--yml
category: 未分类
date: 2024-05-29 12:37:14
-->

# Closure | Post-quantum cryptography is too damn big.

> 来源：[https://dadrian.io/blog/posts/pqc-signatures-2024/](https://dadrian.io/blog/posts/pqc-signatures-2024/)

Large-scale quantum computers are capable of breaking all of the common forms of asymmetric cryptography used on the Internet today. Luckily, they don’t exist yet. The Internet-wide transition to post-quantum cryptography began in 2022 when NIST announced their final candidates for key exchange and signatures in the [NIST PQC competition](https://csrc.nist.gov/projects/post-quantum-cryptography). There is [plenty](https://blog.cloudflare.com/pq-2024/) [written](https://bughunters.google.com/blog/5108747984306176/google-s-threat-model-for-post-quantum-cryptography) about the [various algorithms](https://durumcrustulum.com/2024/02/24/how-to-hold-kems/) and [standardization](https://csrc.nist.gov/projects/pqc-dig-sig) [processes](https://wiki.ietf.org/group/sec/PQCAgility) that are underway.

The conventional wisdom is that it will take a long time to transition to post-quantum cryptography, so we need to start standardizing and deploying things *now*, even though quantum computers are not actually visible on the horizon. We’ll take the best of what comes out the NIST competitions, and deploy it.

Unfortunately, there has not been enough discussion about how what NIST has standardized is simply not good enough to deploy on the public web in most cases. We need better algorithms. Specifically, we need algorithms that use fewer bytes on the wire—a KEM that when embedded in a TLS ClientHello is still under one MTU, a signature that performs on par with ECDSA that is no larger than RSA-2048, and a sub-100 byte signature where we can optionally handle a larger public key.

To understand why, we’ll look at the current state of HTTPS. Cryptography is primarily used in five ways for HTTPS on the public web:

*   **Symmetric Encryption/Decryption**: The actual data for HTTP(2) is transmitted as data inside a TLS connection using some authenticated cipher (AEAD) such as AES-GCM. This is largely [already secure](https://words.filippo.io/dispatches/post-quantum-age/) against quantum computers.
*   **Key Agreement**: Symmetric cryptography requires a secret key. Key agreement is the process in which two parties mutually generate a secret key. TLS 1.3 traditionally used Elliptic Curve Diffie-Hellman for key agreement. All non-post-quantum key exchange mechanisms, including Diffie-Hellman, are broken by quantum computers.
*   **Server Identity**: Servers are authenticated via X.509 certificates. At minimum, a server certificate (leaf certificate) contains a public key, and a signature from an intermediate certificate. The intermediate certificate contains another public key, and a signature from an trusted root certificate.
*   **Issuance Transparency**: The [public Web PKI](https://dadrian.io/blog/posts/certificates-explained) relies on trusted third-parties known as *Certification Authorities* to validate domain ownership. Certificates are publicly logged, and servers attest that their certificates are included in the logs. This provides a deterrent for malicious certificate issuance, since any certificate that is maliciously issued to an attacker for some site will be publicly visible, and has the potential to be detected. Servers achieve issuance transparency by providing at least two *Signed Certificate Timestamps*, usually embedded in the certificate itself.
*   **Handshake Authentication**: The identity of the server needs to be bound to the connection itself during the TLS handshake. In TLS 1.3, this is provided by a signature over the server key share message from the key in the server certificate in the CertificateVerify message.

There is a threat from *future* quantum computers to encrypted network connections *today* in the form of [“harvest now, decrypt later”](https://en.wikipedia.org/wiki/Harvest_now,_decrypt_later) attacks. To defend against this, we only need to ensure that key agreement and symmetric encryption are “quantum resistant” (secure in the presence of quantum computers). Luckily, symmetric encryption is already quantum resistant, and so defending against harvest-now-decrypt-later only requires updating the key exchange algorithm to a post-quantum variant.

The remaining uses of cryptography in HTTPS—server identity, issuance transparency, and handshake authentication—will eventually need to transition to post-quantum variants. In the current structure of TLS, this means replacing all signatures with post-quantum variants. However, the need to do so, while no less *important* ^(than transitioning key exchange, is less *urgent*. This matches the actions of browsers, who are [actively](https://www.reddit.com/r/firefox/comments/1827g86/tls_13_hybridized_kyber_support_for_firefox/) [deploying](https://blog.chromium.org/2023/08/protecting-chrome-traffic-with-hybrid.html) post-quantum key exchange algorithms. An X25519 key exchange involves the client and server transmitted 32 bytes each. The NIST winner for key agreement, [ML-KEM (Kyber)](https://csrc.nist.gov/pubs/fips/203/ipd), involves the client sending 1,184 bytes and the server sending 1,088 bytes.)

However, no widely-used browser has started deploying post-quantum signatures.

This is because post-quantum signatures and their corresponding public keys are too damn big. There are 5 signatures and 2 public keys transmitted during an average TLS handshake for HTTPS:

*   The leaf certificate has 1 signing public key of the site, and 1 signature from the intermediate certificate.
*   The intermediate certificate has 1 signing public key, used to the validate the signature on the leaf, and 1 signature from the key on the root certificate, which is used to validate the authenticity of the intermediate certificate. The root certificate and its embedded public key are predistributed to clients.
*   The handshake itself is has 1 signature from the private key corresponding to the public key in the leaf certificate.
*   Each Signed Certificate Timestamp (SCT) contains one signature. The public key used to the validate the signature is predistributed to clients. Most certificates have 2 SCTs and therefore 2 additional signatures.

The current breakdown of key and signature sizes in TLS is roughly:

*   Root certificates often contain RSA keys, as do intermediate certificates. Root certificates are predistributed, and intermediates are provided by the server, alongside the leaft certificate. An RSA intermediate certificate has a 4096-bit (512 byte) signature, and a 2048-bit (256 byte) public key.
*   An ECDSA leaf certificate has a 32-byte key and a 256-byte RSA signature from the intermediate.
*   The handshake contains a 64-byte ECDSA signature.
*   Each SCT contains a 64-byte ECDSA signature.

In total, this is 512 + 256 + 256 + 32 + 64 + 2*64 = 1,248 bytes of signatures and public keys in a normal TLS handshake for HTTPS. Of the winning signature algorithms from the first NIST PQC competition, [ML-DSA (Dilithium)](https://csrc.nist.gov/pubs/fips/204/ipd) is the only signature algorithm that could be used in the context of TLS ^(and it has 1,312-byte public keys and 2,420-byte signatures. This means *a single ML-DSA public key is bigger than all of the 5 signatures and 2 public keys currently transmitted during an HTTPS connection*. In a direct “copy-and-replace” of current signature algorithms with ML-DSA, a TLS handshake would contain 5*2420 + 2*1312 = 14,724 bytes of signatures and public keys, an over 10x increase.)

Barring a large-scale quantum computer staring us in the face, this is not a tenable amount of data to send simply to *open* a connection. As a baseline reality check, we should not be sending over 1% of a 3.5" floppy disk purely in signatures and public keys^.

In more concrete terms, for the server-sent messages, [Cloudflare found](https://blog.cloudflare.com/pq-2024/) that every 1K of additional data added to the server response caused median HTTPS handshake latency increase by around 1.5%. For the ClientHello, Chrome saw a 4% increase in TLS handshake latency when they deployed ML-KEM, which takes up approximate 1K of additional space in the ClientHello. This pushed the size of the ClientHello greater than the standard maximum transmission unit (MTU) of packets on the Internet, ~1400 bytes, causing the ClientHello to be fragmented over two underlying transport layer (TCP or UDP) packets^.

Assuming ML-KEM is here to stay, this means if we want to keep the total latency impact of post-quantum cryptography under 10%^(, we need to make all of the authentication happen in under ~4K of additional bytes in the server reponse messages. Unfortunately, a *single* ML-DSA signature/public key pair is ~4K bytes. ML-DSA is too big to deploy to mitigate a threat that does not yet have a timeline to exist.)

There is some good news on the horizon. NIST recognized that the signatures were quite large, and is running a [follow-on competition](https://csrc.nist.gov/projects/pqc-dig-sig) for smaller, faster signatures. Unfortunately, the [leaders](https://pqshield.github.io/nist-sigs-zoo/) in that competition are not quite there yet, but some do have potential:

*   **Unbalanced Oil and Vinegar (UOV)**: UOV has big public keys (66K!), but signatures are 94 bytes, which is on par with the current 64 bytes from an ECDSA signature in an SCT. The 66K public key size is acceptable because the public keys for Certificate Transparency logs are predistributed, and there’s only a small number of logs (~10). UOV is not a solution for root certificates—there’s too many root certificates and root stores would be too big to embed in a binary.
*   **SQISign**: SQISign has 64-byte keys and 177-byte signatures. If used for certificates and the handshake signature, it would be 2*64 + 3*177 = 659 bytes. This is compared to the current RSA+ECDSA approach, which is 4096/8 + 2048/8 + 2048/8 + 32 + 64 = 1,120 bytes. SQISign is a net win (and comparable to an ECDSA-only chain)! Unforunately, SQISign is incredibly slow. For SQISign to be feasible, it needs around a 10,000x performance improvement in signing speed, and a 100x performance improvement in verification.
*   **Mayo**: Mayo is possibly feasible. Mayo1 has 1,168-byte public keys and 321-byte signatures, which makes it a candidate for use in certificates and for handshake authentication (1168*2 + 321*3 = 3,299 bytes). Mayo2 has 5,488-byte keys, but only 180-byte signatures, which makes it a candidate for SCTs if UOV doesn’t pan out.

There’s a couple other performance knobs we can attempt to tweak, but they all require larger changes to how HTTPS, TLS, and the Web PKI interact than doing a straight “copy-and-replace” with PQC algorithms.

*   **Intermediate ellison**: Predistributing known intermediate certificates to browsers would save ~1.5K bytes for the median intermediate certificate. This doesn’t fundamentally change any of the feasibility of the NIST candidates, but it likely helps Mayo stay within bounds of what’s currently feasible.
*   **Merging SCTs and Certificates**: Experimental proposals such as [Merkle-Tree Certificates](https://datatracker.ietf.org/doc/draft-davidben-tls-merkle-tree-certs/) merge the certificate and SCTs into a single object with a single hash-based proof of authenticity. This would reduce the handshake to only require a single handshake signature and a single public key in the merkle-tree certificate, alongside a hash-based inclusion proof. Unfortunately, it makes some tradeoffs that are likely not feasible for non-browser applications, such as requiring delayed (hourly) batch issuance, and requiring clients to be up to date relative to a transparency server. Solutions in this form may be a performance optimization for browser clients, but are likely not feasible for non-browser clients. That being said, handshake latency matters considerably less for non-browser clients.
*   **Shrink the size of the root store**: A post-quantum root store with fewer than 10 certificates containing UOV public keys would be within an order of magnitude of the size of current root stores.

This means that combining Mayo and UOV with other changes to the PKI *may* be enough to transition to quantum-resistant authentication in the WebPKI. Unfortunately, all of this armchair design remains subject to several risks:

*   The performance impact is almost definitely larger than what Cloudflare measured. Cloudflare’s experiment likely was primarily ideal clients (enterprise users on desktop) accessing a login page (served by a low-RTT edge server).
*   The performance impact is also probably worse than Chrome current sees! The 4% latency increase is the net increase across all connections once ML-KEM was added to the key shares in the ClientHello. At this point in the deployment, ML-KEM is primarily only supported by Google properties and Cloudflare. Most servers are *not* selecting ML-KEM and responding with the ~1K of an ML-KEM encapsulation. As ML-KEM deployment expands, the 4% latency impact seen by Chrome will get *worse* as the cipher finishes standardization and servers start to deploy ML-KEM.
*   The security of Mayo and UOV might not hold. This would not be the first time a promising post-quantum algorithm [turns out to be broken](https://securitycryptographywhatever.com/2022/08/11/hot-cryptanalytic-summer-with-steven-galbraith/).
*   Even if the performance estimates are accurate, it may be that 10% is still too much of a performance hit unless quantum computers are immenient.

Putting this all together, even a drastic change like Merkele Tree Certs, when combined with ML-DSA or Mayo for handshake authentication, is likely still too big and only suitable for browser clients.

So what can we do to derisk all this? Well, for any solution, we need to get better at trust anchor agility, intermediate suppresion, and PKI migrations. This is [happening already](https://datatracker.ietf.org/doc/draft-davidben-tls-trust-expr/).

The best thing we could do to make the post-quantum transition more feasible is to come up with better algorithms that have performance characteristics no worse than RSA-2048\. Specifically:

1.  A post-quantum KEM that fits in a single MTU when combined with the rest of the TLS ClientHello
2.  A 10,000x signing speed improvement and 100x verification speed improvement in SQISign (or a new, equivalent algorithm with these characteristics)

To some extent, this may be yelling for the impossible. Unfortunately, using ML-DSA for the Web PKI in its current form is also impossible.