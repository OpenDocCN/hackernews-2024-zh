<!--yml
category: 未分类
date: 2024-05-27 13:34:11
-->

# Firstyear's blog-a-log

> 来源：[https://fy.blackhats.net.au/blog/2024-04-26-passkeys-a-shattered-dream/](https://fy.blackhats.net.au/blog/2024-04-26-passkeys-a-shattered-dream/)

At around 11pm last night my partner went to change our lounge room lights with our home light control system. When she tried to login, her account couldn't be accessed. Her Apple Keychain had deleted the Passkey she was using on that site.

This is just the icing on a long trail of [enshittification](https://en.wikipedia.org/wiki/Enshittification) that has undermined Webauthn. I'm over it at this point, and I think it's time to pour one out for Passkeys. The irony is not lost on me that I'm about to release a new major version of [webauthn-rs](https://crates.io/crates/webauthn-rs) today as I write this.

## The Dream

In 2019 I flew to my mates place in Sydney and spent a week starting to write what is now *the* Webauthn library for Rust. In that time I found a number of issues in the standard and contributed improvements to the Webauthn workgroup, even though it took a few years for those issues to be resolved. I started to review spec changes and participate more in discussions.

At the time there was a lot of optimism that this technology could be the end of passwords. You had three major use cases:

*   Second Factor
*   Passwordless
*   Usernameless

Second Factor was a stepping stone toward the latter two. Passwordless is where you would still type in an account name then authenticate with PIN+Touch to your security key, and usernameless is where the identity for your account was resident (discoverable) on the key. This was (from my view) seen as a niche concept by developers since really - how hard is it for a site to have a checkbox that says "remember me"?

This library ended up with [Kanidm](https://kanidm.com/) being (to my knowledge) the very first OpenSource IDM to implement passwordless (now passkeys). The experience was wonderful. You went to [Kanidm](https://kanidm.com/), typed in your username and then were prompted to type your PIN and touch your key. Simple, fast, easy.

For devices like your iPhone or Android, you would do similar - just use your Touch ID and you're in.

It was so easy, so accessible, I remember how it almost felt impossible. That authentication could be cryptographic in nature, but so usable and trivial for consumers. There really was the idea and goal within FIDO and Webauthn that this could be "the end of passwords".

This is what motivated me to continue to improve webauthn-rs. It's reach has gone beyond what I expected with parts of it being used in Firefox's authenticator-rs, a whole microcosm of Rust Identity Providers (IDPs) being created from this library and my work, and even *other* language's Webauthn implementations and password managers using our library as *the* reference implementation to test against. I can not understate how humbled I am of the influence webauthn-rs has had.

## The Warnings

However warnings started to appear that the standard was not as open as people envisaged. The issue we have is well known - Chrome controls a huge portion of the browser market, and development is tightly controlled by Google.

An example of this was the [Authenticator Selection Extension](https://www.w3.org/TR/webauthn-1/#sctn-authenticator-selection-extension).

This extension is *important* for sites that have strict security requirements because they will attest the make and model of the authenticator in use. If you know that the attestation will only accept certain devices, then the browser should filter out and only allow those devices to participate.

However Chrome simply never implemented it leading to it being [removed](https://github.com/w3c/webauthn/issues/1386). And it was removed because Chrome never implemented it. As a result, if Chrome doesn't like something in the specification they can just veto it without consequence.

Later the [justification](https://github.com/w3c/webauthn/issues/1688#issuecomment-1011516074) for this not being implemented was: "We have never implemented it because we don't feel that authenticator discrimination is broadly a good thing. ... they [users] *should* have the expectation that a given security key will broadly work where they want to use it."

*I want you to remember this quote and it's implications.*

*Users should be able to use any device they choose without penalty.*

Now I certainly agree with this notion for general sites on the internet, but within a business where we have policy around what devices may be acceptable the ability to filter devices does matter.

This makes it very possible that you can go to a corporate site, enroll a security key and it appears to work but then it will fail to register (even better if this burns one of your resident key slots that can not be deleted without a full reset of your device) since the IDP rejected the device attestation. That's right, even without this, IDP's can still "discriminate" against devices without this extension, but the user experience is much worse, and the consequences far more severe in some cases.

The kicker is that Chrome has internal feature flags that they can use for Google's needs. They can simply enable their own magic features that control authenticator models for their policy, while everyone else has to have a lesser experience.

The greater warning here is that many of these decisions are made at "F2F" or Face to Face meetings held in the US. This excludes the majority of international participants leading some voices to be stronger than others. It's hard to convince someone when you aren't in the room, even more so when the room is in a country that has a list of [travel advisories](https://www.smartraveller.gov.au/destinations/americas/united-states-america) including "Violent crime is more common in the US than in Australia", "There is a persistent threat of mass casualty violence and terrorist attacks in the US" and "Medical costs in the US are extremely high. You may need to pay up-front for medical assistance". (As an aside, there are countries that have a "do not travel" warning for less, but somehow the US gets a pass ...).

## The Descent

In 2022 Apple annouced [Passkeys](https://appleinsider.com/articles/22/06/07/apple-passkey-feature-will-be-our-first-taste-of-a-truly-password-less-future).

At the time this was just a really nice "marketing" term for passwordless, and Apple's Passkeys had the ability to oppurtunistically be usernameless. It was all in all very polished and well done.

But of course, thought leaders exist, and Apple hadn't defined what a Passkey was. One of those thought leaders took to the FIDO conference stage and announced "Passkeys are resident keys", at the same time as the unleashed a passkeys dev website (I won't link to it out of principal).

The issue is described in detail in another of [my blog posts](../2023-02-02-how-hype-will-turn-your-security-key-into-junk/) but to summarise, this push to resident keys means that security keys are excluded because they often have extremely low limits on storage, the highest being 25 for yubikeys. That simply won't cut it for most people where they have more than 25 accounts.

Now with resident keys as passkeys as users we certainly don't have the expectation that our security keys will work when we want to use them!

## The Enshittocene Period

Since then Passkeys are now seen as a way to capture users and audiences into a platform. What better way to encourage long term entrapment of users then by locking all their credentials into your platform, and even better, credentials that can't be extracted or exported in any capacity.

Both Chrome and Safari will try to force you into using either hybrid (caBLE) where you scan a QR code with your phone to authenticate - you have to click through menus to use a security key. caBLE is not even a good experience, taking more than 60 seconds work in most cases. The UI is beyond obnoxious at this point. Sometimes I think [the password game has a better ux](https://neal.fun/password-game/).

The more egregious offender is Android, which [won't even activate your security key](https://github.com/kanidm/webauthn-rs/issues/365#issuecomment-1756605203) if the website sends the set of options that are needed for Passkeys. This means the IDP gets to choose what device you enroll without your input. And of course, all the developer examples only show you the options to activate "Google Passkeys stored in Google Password Manager". After all, why would you want to use anything else?

A sobering pair of reads are the [Github Passkey Beta](https://github.com/orgs/community/discussions/54450) and [Github Passkey](https://github.com/orgs/community/discussions/67791) threads. There are instances of users whose security keys are not able to be enrolled as the resident key slots are filled. Multiple users describe that Android can not create Passkeys due to platform bugs. Some devices need firmware resets to create Passkeys. Keys can be saved on the client but not the server leading to duplicate account presence and credentials that don't work, or worse lead users to delete the [real credentials](https://bugs.webkit.org/show_bug.cgi?id=270553).

The helplessness of users on these threads is obvious - and these are technical early adopters. The users we need to be advocates for changing from passwords to passkeys. If these users can't make it work how will people from other disciplines fare?

Externally there are other issues. Apple Keychain has personally wiped out all my Passkeys on *three separate occasions*. There are external reports we have recieved of other users who's Keychain Passkeys have been wiped just like mine.

Since this blog has been published many people have contacted me with their own stories of how Passkeys have failed them in various ways. Disappointment in the technology appears to be the norm rather than the exception.

As users how are we meant to trust this technology, a technology that is trying to become the default method to protect our account security?

In order to try to resolve this the workgroup seems to be doubling down on more complex JS apis to try to patch over the issues that they created in the first place. All this extra complexity comes with fragility and more bad experiences, but without resolving the core problems.

It's a mess.

## The Future

At this point I think that Passkeys will fail in the hands of the general consumer population. We missed our golden chance to eliminate passwords through a desire to capture markets and promote hype.

Corporate interests have overruled good user experience once again. Just like ad-blockers, I predict that Passkeys will only be used by a small subset of the technical population, and consumers will generally reject them.

To reiterate - my partner, who is extremely intelligent, an avid computer gamer and *veterinary surgeon* has sworn off Passkeys because the user experience is so shit. She wants to go back to passwords.

And I'm starting to agree - a password manager gives a better experience than passkeys.

That's right. I'm here saying *passwords are a better experience than passkeys*. Do you know how much it pains me to write this sentence? (and yes, that means MFA with TOTP is still important for passwords that require memorisation outside of a password manager).

So do yourself a favour. Get something like [bitwarden](https://bitwarden.com/) or if you like self hosting get [vaultwarden](https://github.com/dani-garcia/vaultwarden). Let it generate your passwords and manage them. If you really want passkeys, put them in a password manager you control. But don't use a platform controlled passkey store, and be very careful with security keys.

And if you do want to use a security key, just use it to unlock your password manager and your email.

Within enterprise there still is a place for attested security keys where you can control the whole experience to avoid the vendor lockin parts. It still has rough edges though. Just today I found a browser that has broken attestation which is not good. You still have to dive through obnoxious UX elements that attempt to force you through caBLE even though your IDP will only accept certain security models, so you're still likely to have some confused users.

Despite all this, I will continue to maintain webauuthn-rs and it's related projects. They are still important to me even if I feel disappointed in the direction of the ecosystem.

But at this point, in [Kanidm](https://kanidm.com/) we are looking into device certificates and smartcards instead. The UI is genuinely better. Which says a lot considering the PKCS11 and PIV specifications. But at least PIV won't fall prone to attempts to enshittify it.