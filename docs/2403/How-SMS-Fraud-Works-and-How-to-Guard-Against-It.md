<!--yml
category: 未分类
date: 2024-05-29 12:43:57
-->

# How SMS Fraud Works and How to Guard Against It

> 来源：[https://technicallythinking.substack.com/p/how-sms-fraud-works-and-how-to-guard-against-it](https://technicallythinking.substack.com/p/how-sms-fraud-works-and-how-to-guard-against-it)

With Twitter [disabling](https://blog.twitter.com/en_us/topics/product/2023/an-update-on-two-factor-authentication-using-sms-on-twitter) text message two-factor authentication, I thought it’d be fun to do a deep-dive into how SMS fraud works and how app developers can guard against it.

It’s a fascinating story of perverse incentives, short-sighted regulation, and technical ingenuity.

Let’s dig in! 👇

> Note to all subscribers of [Mindful Musings](https://apuchitnis.substack.com/) — this post (and others that will follow) are part of a new series that I’m calling *Technically Thinking*. The focus of this section will be more engineering-related, with deep dives, introduction pieces, and more for engineers and hackers.
> 
> If that’s not your cup of tea, feel free to opt-out of Technically Thinking in your subscription — I won’t mind! 😊

To start, let’s recap Twitter’s recent announcement:

In plain English, this simply means that only users of the paid version of Twitter will get a code sent to their phone during login.

The key to understanding SMS fraud is understanding that some numbers are *[premium](https://en.wikipedia.org/wiki/Premium-rate_telephone_number)*. If you want to call or send an SMS to this number, it’ll cost *you* some money — typically tens of cents — and the owner of the number gets a portion of those tens of cents for themselves.

Owners of these phone numbers typically offer legitimate services that cost money to supply and offer value to their users, such as tele-voting, dating, and tech support.

However, these numbers can be gamed for easy profit 🤑

A bad actor, let’s call him *Bob*, gets hold of several premium phone numbers. Bob could be a hacker, or could be a mobile phone network operator gone bad.

Bob finds a web service that will send text messages to his premium phone numbers. These messages could be two-factor authentication codes, one time passwords, or any other text message sent to the user as part of the service (eg [partiful.com](http://partiful.com) sends event reminders via SMS).

Bob finds a way to make the service send *thousands* of SMSs to his premium phone numbers. This might be very easy. The front end service might be easy to manipulate, and the backend endpoints might be unprotected and easy to reverse-engineer.

Even worse, many services use a standardised endpoint for sending SMSs. This makes it vastly easier to for Bob to find sites to attack. For example, if the service uses a third party for authenticating users and sending out 2FA or OTP codes, such as Auth0, then the endpoint for sending SMSs is mostly known: all Bob needs to do is to figure out a way to discover the Auth0’s ID for a web service (fairly easy, since the web service’s front-end makes a request containing this ID), and then they can attack *all* sites that use that third party service.

Finally, Bob makes the service send thousands of SMSs to his premium phone numbers. The web service loses 💵💵💵, and Bob profits.

There’s no one silver bullet to prevent SMS fraud. But here are a few ideas that could work:

*   If using a third-party service to authenticate users, such as Auth0, you could obfuscate the endpoint used to send SMSs. Whilst this won’t prevent an attack outright, it does make it much harder to discover that an attack is possible.

    Similar to how a bike thief targets the easiest to steal bikes, a good hacker would move onto web services that are easier to hack. My hunch is that this approach would work for well enough for the [long tail](https://en.wikipedia.org/wiki/Long_tail) of apps.

*   Block all requests from IPs that originate in cloud providers, fraudulent ISPs, or are otherwise sketchy. This should be fairly simple to implement — many services exist that allow you to rate the quality of an IP address — and would probably be very effective.

*   Add IP-based [rate-limiting](https://en.wikipedia.org/wiki/Rate_limiting) to the endpoint that sends out SMS to block Bob’s attack. If set up correctly, this won’t affect legitimate users. However, this only works against a simple attack. If Bob architects his attack to send requests from a variety of IP addresses — a *distributed* attack — then this wouldn’t work.

*   Only send an SMS to a specific phone number a small number of times before blocking that phone number for a cool-off period. We could do this on the front-end, but if Bob is determined, he could figure out the backend endpoint to attack instead. Blocking the phone number on the backend is harder: it requires keeping a record of phone numbers and their recent login attempts.

*   Force the user to solve a CAPTCHA before sending them an SMS. Whilst this approach works well at blocking attackers — solving the CAPTCHA is hard and expensive to do at scale — it does degrade the user’s experience of the service.

*   Identify and block *premium* rate phone numbers, using [libphonenumber](https://github.com/google/libphonenumber). Whilst this seems promising, I don’t know how reliable the data and how effective this approach is.

*   Only send text messages to paid accounts. This is the approach Twitter has gone with. It’s not a bad option, but as you can see from the list above, there’re many other approaches you could take.

*   Block mobile phone network operators with a high number of fraudulent users. This would block clearly bad network operators, but wouldn’t work well if the network has many legitimate users.

*   Use WhatsApp to send messages instead. WhatsApp is free unlike SMS, but not all users across the world use WhatsApp.

A good solution would make use of enough of the approaches above, prioritising by time investment and effectiveness, until the attackers move onto easier targets.

I’ve got some personal experience implementing the above measures, and have a story or two to share about how my team handled the fallout. But that’s a story for another time… 👨‍💻

This brings me to my final point:

> IMO, Twilio — a dominant SMS API — has a **huge**  **opportunity** to offer SMS fraud protection as a (free? 🙏) add-on to their standard APIs.
> 
> Since Twilio has data on fraudulent phone numbers and carriers across all their accounts, Twilio are in the unique position to block bad numbers and carriers *fast* — before they becomes a big issue for multiple web services.
> 
> Twilio can even detect invalid phone numbers outright using [Silent Network Auth](https://www.twilio.com/docs/verify/sna) — a next-generation authentication mechanism — and it feels like this utility ought to be *shared* between their users.

I’d love to hear any other thoughts, ideas and approaches folks have used — please share by writing a comment below, and we can all learn.

That’s it for now — protect those endpoints, and have a great week!

*There’s an excellent discussion on [HackerNews](https://news.ycombinator.com/item?id=34972712).*