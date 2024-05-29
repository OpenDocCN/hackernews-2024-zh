<!--yml
category: 未分类
date: 2024-05-29 13:27:08
-->

# The Tech Industry Doesn’t Understand Consent - Dhole Moments

> 来源：[https://soatok.blog/2024/02/27/the-tech-industry-doesnt-understand-consent/](https://soatok.blog/2024/02/27/the-tech-industry-doesnt-understand-consent/)

Thanks to Samantha Cole at 404 Media, we are now aware that [Automattic plans to sell user data from Tumblr and WordPress.com](https://www.404media.co/tumblr-and-wordpress-to-sell-users-data-to-train-ai-tools/) (which is the host for my blog) for “AI” products.

In response to journalists probing this shady decision from Automattic leadership, the company said nothing but published [a statement](https://web.archive.org/web/20240228025838/https://automattic.com/2024/02/27/protecting-user-choice/).

This statement, which was presumably filtered through more lawyers than their CEO’s [recent Twitter rambling against trans users](https://twitter.com/kthorjensen/status/1761599420330823935) (or [Automattic employees’ statement about his conduct](https://web.archive.org/web/20240228022939/https://staff.tumblr.com/post/743224389484625920/a-message-from-a-few-of-the-trans-staff-at-tumblr) for that matter), betrays a critical misunderstanding of what consent is.

> We are also working directly with select AI companies as long as their plans align with what our community cares about: attribution, **opt-outs,** and control.
> 
> Emphasis mine

This is not the tech industry’s most egregious lack of understanding of consent in recent years. That dishonor belongs to [LegalFling](https://www.vice.com/en/article/paqvn7/dont-fuck-anybody-who-wants-to-get-your-consent-uploaded-to-the-blockchain-legalfling-app): A blockchain app for sexual consent.

However, this is still pretty stupid, and the result of an insidious trend that doesn’t get questioned enough in software engineering circles. So I’m asking that my readers shout this from the fucking rooftops.

## Opt-Out Is NOT Consent

Opt-out is “our lawyers told us to make this an option to cover our ass, but we don’t want you to actually do it”.

Opt-out is “if you missed the memo, we assume we have your consent”.

The default state of any decision regarding user data should be opted out. Users should instead be required to opt in for your decision to take effect, and they must not be coerced into doing so.

If consent is not explicitly given by an informed user, you haven’t received consent at all, and to pretend otherwise is unethical.

Your users don’t fucking care about opt-out. We care about opt-in.

### “But Soatok, That’ll Hurt Our Revenue”

If you *have to* make money doing unethical things, or by following dubious practices that don’t actually respect other users’ autonomy, then you should go out of business.

 [https://www.youtube.com/embed/AaU6tI2pb3M?version=3&rel=1&showsearch=0&showinfo=1&iv_load_policy=1&fs=1&hl=en-US&autohide=2&wmode=transparent](https://www.youtube.com/embed/AaU6tI2pb3M?version=3&rel=1&showsearch=0&showinfo=1&iv_load_policy=1&fs=1&hl=en-US&autohide=2&wmode=transparent)

VIDEO

End of.

## What Automattic Should Do

If Automattic wants to make things right, they must do two things and could do a third thing (but I’m not holding my breath):

First, nuke the existing opt-out mechanism and replace it with an opt-in mechanism. If nobody checks it, then don’t include their data in the sale to Midjourney or OpenAI.

Second, they should make the permission for third-parties more granular. Some of us don’t care about third parties, but do NOT want “AI” companies using their data to enable plagiarism.

Third, if you want to go the extra mile, add support for a plugin that uses [Nightshade](https://nightshade.cs.uchicago.edu/whatis.html) on all hosted media in all WordPress.com plans, including free plans, to increase your users’ protection against LLM scrapists. [See below.](#nightshade)

This is my open challenge to Automattic leadership to do better.

## This Issue is Bigger than Automattic

The tech industry has gotten very bad at respecting users’ consent lately. Your options are no longer “Yes” and “No” anymore. Instead it’s “Yes” or “Maybe Later”, without a “Never” option.

Even worse, you can rarely uninstall the crapware that nags you with these consent dialogs.

This needs to stop. It’s a toxic mentality and it cultivates a culture that doesn’t respect humanity. (Which is kind of funny to write as a furry blogger.)

If you work in the tech industry, scream very loudly about properly implementing human-respecting consent controls into your software.

Just because it’s widespread doesn’t mean it’s inevitable. Push back against it. Your less privileged, less technical neighbors deserve better.

## Addendum on NightShade

Ever since this was first posted, a few people expressed confusion or didn’t grasp the implications of the 3rd suggestion to Automattic above. So I will elaborate.

[Nightshade](https://nightshade.cs.uchicago.edu/whatis.html) is a technique for poisoning LLM data sets.

The idea is that, for blogs that have not opted into industrial plagiarism enabled by large-scale computing (“AI” for short), the WordPress.com CDN would serve Nightshade-poisoned images, instead of the standard ones provided by the blogger, to all readers.

Then you don’t even have to worry about detecting who’s scraping for AI projects or not. You just serve poison to everyone. If they’re using the images for AI, it ruins their model. If they aren’t, it causes no harm at all.

If OpenAI and Midjourney want non-poisoned images, then instead of scraping, they can abide by boundaries of the bloggers who didn’t opt in and only accept the data that Automattic provides to them without resorting to scraping.

This is a fairly obvious way to protect users from the AI machine.

Nightshade isn’t perfect, but no technology is. The end goal is to make the cost of bypassing Nightshade’s poison higher than respecting the decision of individual authors and artists.

Incentives rule everything around me.