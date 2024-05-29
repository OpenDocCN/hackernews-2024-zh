<!--yml
category: 未分类
date: 2024-05-27 15:12:06
-->

# New email policies at Gmail, Yahoo et al will require DMARC enabled domains | easyDNS

> 来源：[https://easydns.com/blog/2024/01/19/new-email-policies-at-gmail-yahoo-et-al-will-require-dmarc-enabled-domains/](https://easydns.com/blog/2024/01/19/new-email-policies-at-gmail-yahoo-et-al-will-require-dmarc-enabled-domains/)

As of February 1st, Gmail and Yahoo will start enforcing a requirement for **D**omain-based **M**essage **A**uthentication, **R**eporting and **C**onformance (“**DMARC**“). DMARC is an IETF protocol as specified in [RFC7489](https://www.rfc-editor.org/rfc/rfc7489).

It will initially affect large senders, defined as domains originating more than 5K emails per day into the Gmail system (this is the shape of things to come however, it will be increasingly more difficult to get your email delivered in the absence of SPF, DKIM and DMARC).

It works with **SPF** and **DKIM **to signal to receiving mail servers how they should handle messages that *fail *SPF and DKIM validation.

You should be implementing SPF on all your mailer domains already – and DKIM is enabled by default on easyMail. If you’re running your own mail server your postmaster should have implemented it.

## Quick refresher on SPF and DKIM:

**SPF ([Sender Policy Framework](https://spfwizard.com/)):**  signals which mailservers can legitimately originate mail from your domain name, so phishes, fakes and spams can be better identified. *(Also note that if you run an email forwarder at your local installation with **[Sender Rewrite Scheme (SRS)](https://easydns.com/features/srs-enabled-email-forwarding)** enabled, then you need to have DMARC on your SRS domain as well.)*

**DKIM ([DomainKeys Identified Mail](https://easydns.com/features/dkim-domainkeys/)):** cryptographically signs the messages as they leave the originating mail server, so that intermediary hops cannot modify the message contents without you knowing it.

## What DMARC adds to the mix

While the final “all” parameter in SPF (-all, +all, ~all) signals suggested actions upon an SPF fail, it’s largely up to the receiving the server what to do with it. Some ignore the setting no matter what, some send it to spam, some will bounce an **-all **that fails, others will ignore it.

With DMARC, you signal to the receivers – everywhere – two things:

1.  What to do with messages that fail SPF or DKIM
2.  Who to report failures to

This second point is important. Within DMARC records (which are all just DNS TXT records, just like SPF and DKIM) – you can add an address that specifies and email that DMARC reports get sent back to.

## DMARC Reports flow back to postmaster

Using these reports, you can monitor your email origination – perhaps there’s a blog server, a CRM or a listserv somewhere that you’ve forgotten to include into your SPF – and all this time those messages were being largely routed to spam folders because they are failing SPF checks.  With DMARC enabled, you will  pick this up in the reports – and be able to adjust your SPF record accordingly.

DMARC contains a parameter that governs *how much *of the inbound email flow for your domain to quarantine if they fail a check. You can even start with “none” (p=none), and the recommended deployment method is to start with that value.

The reports flow back to the address you specify – and they come in xml format and often times .zip-ed. And once you enable DMARC, you’ll get a *lot *of them so using one of your own email boxes to receive them is a bad idea.

*   You’ll have to create a separate email box to receive this (we’re looking at bundling a postmaster@ box with easyMail domains as part of our implementation)
*   You will optionally want to use a [DMARC parsing service](https://partners.easydmarc.com/9sqbo9k9idd8) like **easydmarc** to make sense of all these reports (not kidding and they are not related to us, but we use them now and are partnering with them) . If you have the technical chops there are also open source DMARC analyzer packages like **[parsedmarc](https://domainaware.github.io/parsedmarc/)** (and if you’re a dev, consider pitching in with some open issues [on his Github](https://github.com/domainaware/parsedmarc/issues), one man show could use the help).

## The DMARC Quickstart:

Our recommended starting deployment for DMARC is to just add a TXT record to your zone like this: `_dmarc IN TXT "v=DMARC1;p=none;rua=mailto:dmarc@example.com;ruf=mailto:dmarc@example.com;rf=afrf"` 

And the absolute MVD (Minimum Viable DMARC) rec to put in just to satisfy the Google servers would be:

`_dmarc IN TXT "v=DMARC1;p=none;"` 

Note that where this record sits in your zone is slightly different from SPF:

*   your SPF TXT record is under “@”, your zone apex
*   the DMARC record is scoped record: “_dmarc”

Having the minimal record above in place (along with at *least *an SPF record for your zone) will bring you into compliance with the new policy and maintain your email deliverability to the likes of Gmail and Yahoo. It may even improve it, if you haven’t had an SPF record until now.

## DMARC Deployment within easyMail

As I mentioned, we’re going to be adding the postmaster@ account on the new stand-alone easyMail boxes (announcement on that soon).

We will be testing all easyMail domains for DMARC records over the next few weeks and notifying you if you’re flying without one – you *will *need this on come February lest your deliverability to Gmail (and places like Yahoo) plummet.

## Further Reading and Next Steps:

If you’ve never set up an SPF record, you should do that no matter what:

Watch this space for links to our DMARC documentation and rollout.