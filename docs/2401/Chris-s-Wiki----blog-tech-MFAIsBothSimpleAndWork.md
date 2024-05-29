<!--yml
category: 未分类
date: 2024-05-27 14:41:47
-->

# Chris's Wiki :: blog/tech/MFAIsBothSimpleAndWork

> 来源：[https://utcc.utoronto.ca/~cks/space/blog/tech/MFAIsBothSimpleAndWork](https://utcc.utoronto.ca/~cks/space/blog/tech/MFAIsBothSimpleAndWork)

Over on the Fediverse [I said something a while back](https://mastodon.social/@cks/111564970574073609):

> I have a rant bubbling around my head about 'why I'll never enable MFA for Github'. The short version is that I publish code on GH because it's an easy and low effort way to share things. So far, managing MFA and MFA recovery is neither of those; there's a lot of hassles, worries, work to do, and 'do I trust you to never abuse my phone number in the future?' questions (spoiler, no).
> 
> I'll deal with MFA for work. I won't do it for things I'm only doing for fun, because MFA makes it not-fun.

[Basic MFA](/~cks/space/blog/tech/MFABasicOptionsIn2023) is ostensibly pretty simple these days. You get a trustworthy app for your smartphone (that's two strikes right there), you scan the QR code you get when you enable MFA on your account, and then afterward you use your phone to generate the MFA [TOTP](https://en.wikipedia.org/wiki/Time-based_one-time_password) code that you type in when logging in along with your password. That's a little bit more annoying than the plain password, but think of the security, right?

But what if your phone is lost, damaged, or unusable because it has a bulging battery and it's taking a week or two to get your carrier to exchange it for a new one (which happened to us with our work phones)? Generally you get some one time use special codes, but now you have to store and manage them (obviously not on the phone). If you're cautious about losing access to your phone, you may want to back up the TOTP QR code and secret itself. Both the recovery codes and the TOTP secret are effectively passwords and now you need to handle them securely; if you use a password manager, it may or may not be willing to store them securely for you. Perhaps you can look into [age](https://age-encryption.org/).

(Printing out your recovery codes and storing the paper somewhere leaves you exposed to issues like a home fire, which is an occasion where you might also lose your phone.)

Broadly, [no website has a good account recovery story for MFA](/~cks/space/blog/tech/MFAAccountRecoveryDistrust). Figuring out how to deal with this is not trivial and is your problem, not theirs. And while TOTP is not the in MFA thing these days, the story is in many ways worse with physical hardware tokens, because you can't back them up at all (unlike TOTP secrets). Some environments will back up software [Passkeys](https://en.wikipedia.org/wiki/WebAuthn), but so far only between the same type of thing and often at the price of synchronizing things like all of your browser state.

However, all of this is basically invisible in the simple MFA story. The simple MFA story is that everything magically just works and that you can turn it on without problems or serious risks. Of course, websites have a good reason for pushing this story; they want their users to turn on MFA, for various reasons. My belief is that the gap between the simple MFA story and the actual work of doing MFA in a way that you can reliably maintain access to your account is dangerous, and sooner or later this danger is going to become painfully visible.

(Like many other versions of [mathematical security](/~cks/space/blog/tech/SecurityIsPeople), the simple MFA story invites blaming people (invariably called 'users' when doing this) when something goes wrong. They should have carefully saved their backup codes, not lost track of them; they should have sync'd their phone's TOTP stores to the cloud, or done special export and import steps when changing phones, or whatever else might have prevented the issue. This is as wrong as it always is. [Security is not math, it is people](/~cks/space/blog/tech/SecurityIsPeople).)