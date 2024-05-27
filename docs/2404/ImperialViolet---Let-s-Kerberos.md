<!--yml
category: 未分类
date: 2024-05-27 13:03:21
-->

# ImperialViolet - Let's Kerberos

> 来源：[https://www.imperialviolet.org/2024/04/07/letskerberos.html](https://www.imperialviolet.org/2024/04/07/letskerberos.html)

(I think this is worth pondering, but I don’t mean it *too* seriously—don’t panic.)

Are the sizes of post-quantum signatures [getting you down](https://dadrian.io/blog/posts/pqc-signatures-2024/)? Are you despairing of deploying a post-quantum Web PKI? Don’t fret! Symmetric cryptography is post-quantum too!

When you connect to a site, also fetch a record from DNS that contains a handful of “CA” records. Each contains:

*   a UUID that identifies a CA
*   E[CA-key](server-CA-key, AAD=server-hostname)
*   A key ID so that the CA can find “CA-key” from the previous field.

“CA-key” is a symmetric key known only to the CA, and “server-CA-key” is a symmetric key known to the server and the CA.

The client finds three of these CA records where the UUID matches a CA that the client trusts. It then sends a message to each CA containing:

*   E[CA-key’](client-CA-key) — i.e. a key that the client and CA share, encrypted to a key that only the CA knows. We’ll get to how the client has such a value later.
*   A key ID for CA-key’.
*   E[client-CA-key](client-server-key) — the client randomly generates a client–server key for each CA.
*   The CA record from the server’s DNS.
*   The hostname that the client is connecting to.

The CA can decrypt “client-CA-key” and then it can decrypt “server-CA-key” (from the DNS information that the client sent) using an AAD that’s either the client’s specified hostname, or else that hostname with the first label replaced with `*`, for wildcard records.

The CA replies with E[server-CA-key](client-server-key), i.e. the client’s chosen key, encrypted to the server. The client can then start a TLS connection with the server, send it the three encrypted client–server keys, and the client and server can authenticate a Kyber key-agreement using the three shared keys concatenated.

Both the client and server need symmetric keys established with each CA for this to work. To do this, they’ll need to establish a public-key authenticated connection to the CA. So these connections will need large post-quantum signatures, but that cost can be amortised over many connections between clients and servers. (And the servers will have to pass standard challenges in order to prove that they can legitimately speak for a given hostname.)

Some points:

*   The CAs get to see which servers clients are talking to, like OCSP servers used to. Technical and policy controls will be needed to prevent that information from being misused. E.g. CAs run audited code in at least SEV/TDX.
*   You need to compromise at least three CAs in order to achieve anything. While we have Certificate Transparency today, that’s a post-hoc auditing mechanism and a single CA compromise is still a problem in the current WebPKI.
*   The CAs can be required to publish a log of server key IDs that they recognise for each hostname. They could choose not to log a record, but three of them need to be evil to compromise anything.
*   There’s additional latency from having to contact the CAs. However, one might be able to overlap that with doing the Kyber exchange with the server. Certainly clients could cache and reuse client-server keys for a while.
*   CAs can generate new keys every day. Old keys can continue to work for a few days. Servers are renewing shared keys with the CAs daily. (ACME-like automation is very much assumed here.)
*   The public-keys that parties use to establish shared keys are very long term, however. Like roots are today.
*   Distrusting a CA in this model needn’t be a Whole Big Thing like it is today: Require sites to be set up with at least five trusted CAs so that any CA can be distrusted without impact. I.e. it’s like distrusting a Certificate Transparency log.
*   Revocation by CAs is easy and can be immediately effective.
*   CAs should be highly available, but the system can handle a CA being unavailable by using other ones. The high-availability part of CA processing is designed to be nearly stateless so should scale very well and be reasonably robust using anycast addresses.