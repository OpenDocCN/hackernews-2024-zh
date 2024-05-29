<!--yml
category: 未分类
date: 2024-05-29 12:41:26
-->

# A simple, asymmetric AKE · Timelessness

> 来源：[https://dannyvanheumen.nl/post/simple-asymmetric-ake/](https://dannyvanheumen.nl/post/simple-asymmetric-ake/)

# A simple, asymmetric AKE

Thu, Mar 28, 2024 ❝A simple, (as-of-yet unidentified) asymmetric Authenticated Key Exchange❞ <details><summary>Contents</summary></details>

In an experiment, I came across a need for a simple authenticated key exchange. It started as a stop-gap measure that is just a key exchange, no considerations for any kind of attacker. Just the minimal working solution with standard building blocks. From there, I extended the solution to protect against an attacker.

## Introduction

*The use-case* is a user and a “service-provider” of some kind, in my case a device. The device responds to requests, performs computations in a separate computing environment and is, in this particular case, connected by USB port. There is sensitive information involved. The device, however, does not have storage capability. Computations are based on and initiated through the input of the user that interacts with the device. Initiative lies with the user; all risk lies with the judgement of the user whether or not the session is established correctly.

*From the device’s perspective*, it only needs to act on requests. We can assume that if the user is willing to interact with it, it must be safe to respond. Or from a different perspective: if the user is willing to send their security-sensitive information, then it must be okay to do perform the computation and respond. A confidential session that additionally guarantees the (only) other party is the one that initiated the session, is all that is needed. This other party supplies data, and receives results.

*From the user’s perspective*, there are additional concerns: the user must be ensured that the device is the actual device that it expects, i.e. there must be no mistaken identity, and they must be sure that communication is secure, if they decide to continue after establishing a secure session, i.e. performing the AKE to completion.

### The device: TKey

*Note: some of the feedback was a result of confusion on what kind of device is actually involved. Although I prefer to establish a solid abstract definition, this section should make the description and requirements more concrete.*

The specific device I have in mind for this protocol, is [tillitis TKey](https://tillitis.se/products/tkey/ "TKey - Tillitis"). This devices contains an inaccessible device-secret (*UDS*) and loads any program provided. The device itself has a small firmware that is capable of initialization and receiving bytes that are the program-binary. When a program is loaded, a secret is derived that is specific to: device + program-binary + user-supplied secret, called the *CDI*. Then the firmware jumps to the entry-point of the program to start execution, and consequently hands over the control to the program.

The *TKey* does not have storage capability. Programs have a few (hardware) features to work with, and *CDI* as a unique secret. The *CDI* value is the [digest of a hash-computation](https://dev.tillitis.se/intro/#measured-boot--secrets "Introduction - TKey Developer Handbook") of *UDS*, *program-binary* and *user-supplied secret*. Consequently, a program-binary must be byte-exact, which offers interesting opportunities such as that it protects users from malicious programs, which may attempt to steal this secret. Using the *CDI*, and a hash-function, for example *Blake2s* that is provided in firmware, a program can derive several secrets. The *identity* as discussed for this protocol, would be such a secret value. Therefore, the *identity*, with overwhelming likelihood, proves that the device, program-binary and user-supplied secret are the expected values.

Any concerns w.r.t. device security, safety of secrets, potential vulnerabilities in programming, etc. are all valid criticism. However, the post itself is scoped to the protocol itself.

## The Design

**requirements**:

*   A two-party secure session is established between client and device,
*   Protected against MitM,
*   Ephemeral session-keys,
*   Device gets authenticated,
*   Client does not require authentication,
*   No need for a session ID,

**prerequisite: no (assumptions on) client’s identity**: The device starts off with only its own program state, needed for its own (internal) operation. Any data that device works with, is provided by client. This has the benefit that it cannot mistakenly expose sensitive data or mistake the identity of a user upon establishing a session. The device works with whatever data the initiator of the session provides. Client is only (implicitly) authenticated in that the secure session ensures that only the other party is able to communicate within the established session.

**prerequisite: session-scoped state**: The device’s state must be session-scoped, such that no data is leaked between sessions or spills over into next session, if applicable.

## Protocol

The following is built on top of a Diffie-Hellman Key Exchange, with minimal practical adaptations to make it work in situations with an attacker. The protocol is proved for a passive attacker, with an attempt to prove for an active attacker currently stalled because of technical issues.

with:

*   prefix `pk_` indicating public key, and
*   prefix `sk_` indicating secret/private key
*   `digest = HASH(key, content)`
*   `ciphertext = ENCRYPT(key, plaintext)`
*   `plaintext = DECRYPT(key, ciphertext)`
*   `signature = SIGN(secret_key, content)`
*   `VALIDATE(public_key, signature, content)`

**Precondition** an identity `pk_identity` acquired at an earlier time.

```
User                                                              Device
========================================================================
knows `pk_identity`                   knows `sk_identity`, `pk_identity`
generates `sk_user`, `pk_user`

User ----------------------------[pk_user]----------------------> Device

                                      generates `sk_device`, `pk_device`
                         sessionkey = HASH(pk_user^sk_device, pk_device)
                                  signature = SIGN(sk_identity, pk_user)
                                enc_sig = ENCRYPT(sessionkey, signature)

User <----------------------[pk_device,enc_sig]------------------ Device

sessionkey = HASH(pk_device^sk_user, pk_device)
proof = DECRYPT(sessionkey, enc_sig)
VALIDATE(pk_identity, proof, pk_user) 
```

Execution concludes successfully when the proof validates successfully. Any corruption results in failure to validate the signature. Given that all risk lies with the user, anything other than successful validation is reason for termination.

**After successful execution**:

*   a confidential session is established with a shared, ephemeral session-key,
*   we are assured that `pk_device` is authentic,
*   we are assured that `pk_user` was received unchanged,
*   we are assured this is the authentic device, i.e. ownership over identity proved.

**Failed execution**:

*   received `pk_user` is invalid/corrupted
*   received `pk_device` is invalid/corrupted
*   `sessionkey` cannot be produced due to `pk_device`
*   validation of `pk_user` fails due to:
    *   `proof` is not authentic `signature`:
        *   incorrect `sessionkey`
        *   due to corrupted `enc_sig`
        *   signature over corrupted `pk_user`
    *   device does not own `sk_identity`

Note: the protocol does not reflect the use of a prefix/context to prevent use of signature for other than identification. This may need to be considered when used.

### Insights

*   User’s D-H public key doubles as challenge-nonce, therefore there is no opportunity to swap out the nonce for one with a known signature, and not freely selectable because that requires a corresponding private key.
*   The challenge-response signature is encrypted with established session-key, thus preventing (raw) replay of the response-message.
*   Device’s D-H public key, additionally, is used as input in keyed-hash, with D-H’s resulting shared secret as key. Consequently, both shared secret and device’s public key are fixed. In a two-party set-up, user’s public key must therefore also be fixed if one expects to compute the same shared secret.
*   Given that there is no persistence and client is expected to initiate and provide data, device needs only to rely on a secure session to keep data confidential.

## Analysis

At this time, I have not completed a full analysis of the protocol. The following is, thusfar, based on intuition and informal reasoning, directed by the notions of the eCK security-model, and later (partially) confirmed by results of a symbolic prover.

Security properties for authenticated key exchange according to the extended Canetti-Krawczyk (eCK) model.

*   *(Implicit) Key Agreement*
    *   `client`/`device`: by virtue of establishing a two-party secure session using ephemeral key exchange.
*   *Key Confirmation*
    *   `client`: by implication of successful signature validation against `pk_user`.
    *   `device`: by implication of client following up.
*   *Known Key Security*
    *   Session keys are fully based on random data, with contributions from both parties.
*   *Security against Unknown Key Share attacks*
    *   Only 2 participants in the key exchange. Proper use of D-H prevents MitM. Only two parties involved, therefore no need to share or otherwise relay keys.
    *   `device`: assume that any counterpart in established secure session is valid. Further made acceptable by the fact that all data is provided by counterpart in session. Proper key exchange prevents interference from any third-party.
    *   `client`: no need to share keys. Each session is freshly established. Sessions are always between one `device` and one `client`.
*   *Security against Key Compromise Impersonation (KCI) attacks*
    *   The Identity keypair is only used for signing the challenge-nonce that is sent to the client. Therefore, there is no opportunity where a signature can be misinterpreted.
*   *Forward Secrecy*
    *   The AKE is initiated with inputs based on random data, with the pre-shared *identity public key* being the only exception.

Granted, the eCK model is (presumably) followed in spirit rather than strictly, as this is an asymmetric AKE and, for example, in case of *Key Confirmation* `device` does not get concrete confirmation. The requirements are such that this is not problem in practice.

## Symbolic prover

The following [Verifpal](https://www.verifpal.com/ "Verifpal: Cryptographic Protocol Analysis for Students and Engineers") proof-script passes for a passive attacker. A session established by client with device is successful if the protocol is completed without failure.

Note, though, that at the moment of completion, only `client` knows the results and whether it is safe to continue. This is okay, because `client` must be convinced that `device` is authentic and communication is secure, before requesting `device`’s services and sending data.

The Verifpal proof-script:

```
attacker[passive]
principal Device[
	knows private identity_secret
	identity = G^identity_secret
]
// Assume knowledge of identity public key prior to initiation of the AKE.
// (Modeled here as a guarded/guaranteed transfer: '[identity]')
Device -> User: [identity]
principal User[
	generates dhe_user_secret
	dhe_user_public = G^dhe_user_secret
]

// Initiation of a secure session.
User -> Device: dhe_user_public
principal Device[
	generates dhe_device_secret
	dhe_device_public = G^dhe_device_secret
	ss_device = dhe_user_public^dhe_device_secret
	sessionkey_device = HASH(ss_device, dhe_device_public)
	signature = SIGN(identity_secret, dhe_user_public)
	proof_encrypted = ENC(sessionkey_device, signature)
]
Device -> User: dhe_device_public,proof_encrypted
principal User[
	ss_user = dhe_device_public^dhe_user_secret
	sessionkey_user = HASH(ss_user, dhe_device_public)
	proof = DEC(sessionkey_user, proof_encrypted)
	_ = SIGNVERIF(identity, dhe_user_public, proof)?
]

queries[
	confidentiality? identity_secret
	confidentiality? dhe_device_secret
	confidentiality? dhe_user_secret
	confidentiality? ss_device
	confidentiality? ss_user
	confidentiality? sessionkey_device
	confidentiality? sessionkey_user
	freshness? dhe_user_secret
	freshness? dhe_device_secret
	freshness? ss_device
	freshness? ss_user
	freshness? sessionkey_device
	freshness? sessionkey_user
	equivalence? ss_user, ss_device
	equivalence? sessionkey_user, sessionkey_device
	equivalence? proof, signature
	// Authentication-queries provide no significance with a passive attacker.
	authentication? User -> Device: dhe_user_public
	authentication? Device -> User: dhe_device_public
	authentication? Device -> User: proof_encrypted
] 
```

The current issue with the “*active attacker*” analysis, is that the symbolic prover will prematurely fail on successful manipulation of the intermediate values, such as `ss_device`, `ss_user`, etc. The argument is that confidentiality is broken.

The queries should be performed *after* the protocol completes (success)fully. `SIGNVERIF(identity, dhe_user_public, proof)?`, a checked (`?`) verification statement, should fail for any manipulation of `dhe_user_public` as the signature is validated against the original/authentic `dhe_user_public` generated by `client`. However, it does not. Therefore it proceeds to list all the violations of confidentiality of intermediate variables that are the possible as a result of corrupting the transferred `dhe_user_public`.

By my (informal) reasoning, if an attacker corrupts any data, `client`’s final signature validation must fail. Therefore, if final validation succeeds, authentic data must have been used and an attacker did not manipulate the established secure session.

### Open issue

The [issue is reported](https://lists.symbolic.software/pipermail/verifpal/2023/000454.html "[Verifpal] Checked SIGNVERIF passed, but checked ASSERT fails (was: Mistake or missed case? SIGNVERIF with original of sent value)") to Verifpal. Unfortunately, at the time of writing, there has not been any follow-up yet. A reduced sample is used to further narrow down the problem as part of the email conversation.

The protocol, when analyzed for an active attacker, produces a result that need never happen. The claim is: with a `nil` (corrupted) public key, a `nil` message, and a `nil` signature, the validation would pass. This is correct of course. However, given that `client` itself generates `pk_user`, there is no need to insert any received (potentially corrupted) value. Therefore, the failing case need never occur.

## Rationale for identity-challenge

One particular design choice that I had not discussed is that of using the Diffie-Hellman public key for the identity-challenge. This seems to be a somewhat uncommon choice. The rationale behind this is as follows; let’s discuss the various possibilities that can occur:

Note that the “*proof of identity*” here, is the signature over the D-H public key of `client`.

1.  A legitimate device with its (authentic) keypair produces a proof of identity.
    *This is intended use-case where there is no malicious activity.*
2.  An adversary (Mallory) impersonates the expected device, without having access to the *identity* keypair.
    *Mallory impersonates `device`, making `client` think they’re talking to `device`. Given no access to the keypair, Mallory can only have access to previously produced signatures by `device`. Given that signatures are encrypted when transferred, Mallory only has access to signatures produced in his own sessions with `device`. Providing the right signature without access to the identity-keypair is akin to having the corresponding private key of the D-H keypair, given the size of the key-space.*
3.  Man-in-the-Middle attack on session between `client` and `device`.
    *If Mallory is able to perform this attack, then he possesses the same ephemeral keypair as `client`. Therefore he can establish his own session with `device` with same keypair and use the signature-proof from `device`, sent to him, to forward to `client`. To do this, Mallory requires access to `client`’s generated keypair of the moment, either by solving the computational Diffie-Hellman problem or by invading `client` process’ memory, or similar invasive actions.*
4.  Identity-keypair is known/exposed; Mallory possesses `device`’s keypair.
    *Mallory can fully impersonate `device`. Given it controls the keypair, Mallory can respond to any D-H public key challenge from client with proof-signature.*

The first case is the normal use-case, as intended. The last two cases involve breaking fundamental cryptographic mechanisms or active attacks, both are actions taken outside of the protocol. Both of these cases involve asymmetric cryptography, so are particularly vulnerable to quantum-computing attacks. However, these attacks are not yet possible.

The remaining case, (2.), means we make proving the identity equally complicated as breaking the security of the key-exchange. It is true that if a legitimate client connects twice to `device` with the same key, we could then respond with the proof-signature of the first session. However, “*freshness*” of the ephemeral session keys is also a requirement. Note that it is to the benefit of `client` to not save signatures and attacker does not have access to the (encrypted) signature without the ability to eavesdrop on the session. Ideally, we would never use the same keypair twice. For an attacker, unless he can manipulate `client`’s choice, their choice of public key should be unpredictable, then eavesdropping is impossible. He cannot compute corresponding private key, and he cannot collect the corresponding signature. (Note that this implies that ephemeral D-H public key should be unpredictable not only for attackers, but also for `device`, which are complementary.)

## Feedback

This post is discussed on [Hacker News](https://news.ycombinator.com/item?id=39813537 "A simple, (as-of-yet unidentified) asymmetric Authenticated Key Exchange (Hacker News)").

Most feedback so far is rooted in initial misunderstanding of the situation. Consequently, topics such as *attestation*, confusion about how a device can work without any kind of storage, and need for authentication, arise. I have addressed a direct comment on the introduction being unclear by adding the section about the specific device. I expect this is sufficient to align reader’s expectations of the abstract descriptions and requirements with an actual use-case.

Following are a few comments that address this feedback for readers who finish with similar questions.

*   The *identity* is used to identify a specific device instantiation, as opposed to a physical piece of hardware. For the same reason, even if the criticism of *attestation* applies, it is not valid here. The *identity* is more specific than what the attestation process targets.
*   Similarly, there was confusion over how unique this *identity* is, what causes it to change, risks and impact when this *identity* would leak. Again, so far rooted in confusion/lack-of-information rather than criticism of the protocol itself.
*   A check for genuine hardware, which most likely is simpler than a full attestation process and IIUC does not apply a full certificate-chain, is already provided. The protocol as described, solves a problem under the assumption that the hardware is trustworthy. (At least at the start. An attack that swaps/removes the hardware is still considered a valid attack.)
*   Why not use established crypto? This is a valid question. Most prevalent suggestion is to use TLS which is quite an elaborate protocol and assumes the PKI certificate infrastructure to be available. Another suggestion is Noise Protocol, which I will have another look at.
*   Concerns around lack of RNG. This was further clarified. There is no RNG. There is a “true RNG” for an entropy-source and there is a hash-function available already in firmware. It is possible to at least approximate a CSPRNG.
*   Concerns over lack of certificate-chain. This was rooted in not having sufficient information about the device. Verification of the device is already a provided feature.
*   Other suggestions hinted at coordinated certification with the device manufacturer. However, this predisposes a centralized, coordinated approach and/or qualification-process, therefore makes several assumptions and trade-offs to make the suggestion viable.
*   Criticisms regarding trust-on-first-use (TOFU) are similarly rooted in misunderstanding at what moment the source-secret becomes available and the extent of the uniqueness.
*   A criticism on whether Verifpal is a good choice of symbolic prover: obviously, if the bugs are not resolved or other concerns remain, it is not a good choice. One could switch to a different solver. However, there is no reason to dismiss any solver simply because it is not known. Furthermore, the proof is used as confirmation, next to manual analysis.

Considering the feedback that I have read, there is not yet an indication that there is a problem with the protocol itself. Many comments are, strictly speaking, off-topic for the protocol. I decided to add a paragraph explaining the specifics of the intended device.

## Changelog

This article will receive updates, if necessary.

*   *2024-03-28* Added sections on description of the concrete device, rationale for identity-challenge, feedback from discussions.
*   *2024-03-23* Typos, formatting.
*   *2024-03-23* Initial version.