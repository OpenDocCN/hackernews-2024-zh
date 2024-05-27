<!--yml
category: 未分类
date: 2024-05-27 13:35:27
-->

# karelvo | Self-hosting forms, the sane way

> 来源：[https://karelvo.com/blog/selfhosting-forms-the-sane-way](https://karelvo.com/blog/selfhosting-forms-the-sane-way)

I run a few (very) small websites, nothing serious: they’re not businesses, they’re hobbies. There is no risk involved if they would go down, apart from maybe disappointing a few hundred people that found a link to it on search engines. Because working on them is sometimes a fun challenge, but at the same time I am not a developer/sysadmin, there are two rules I follow:

*   Try to do as much as you can by yourself, and seek out the challenges.
*   …but seek alternatives when it gets too frustrating/scary. It’s a hobby, after all - no stress.

Some examples of how the combo of those two things translates to my setup:

*   I run all my sites by myself on a Linux server, **BUT** I rent the server (opening ports on my home network gives me more anxiety than I need).
*   I don’t use a CMS or “anything with a GUI”, **BUT** I do use static site builders (I can’t be bothered with raw-dogging plain HTML, CSS, and JS). My favorite is [Astro](https://astro.build).
*   I host a few services, such as site analytics, myself so that any data passes through as few hands as possible, **BUT** I run [Coolify](https://coolify.io/) to deploy my sites (and lately, manage my server).

In other words: DIY, unless there’s a great FOSS tool to help me out. The excuse I give myself is “this is more *hacky* than 95% of people doing the same thing, anyway”.

* * *

## DIY vs form services

* * *

On one of my sites, I needed to install a form with file upload capability. After some research, I found a few solutions to solve my need:

*   **Embedding a form that you build elsewhere.** Examples are [Tally](https://tally.so/) (which is a really inspiring company, by the way), [Typeform](https://www.typeform.com/), and [Jotform](https://www.jotform.com/). I don’t like the idea of embedding as it doesn’t give me control over what’s on my site.
*   **Form backends**, basically databases where you can send your form data to. Examples are [Formspree](https://formspree.io/), [FormSubmit](https://formsubmit.co/), [getform](https://getform.io/), and [Submit JSON](https://www.submitjson.com/). If my use-case was more professional, I’d choose this.
*   **DIY-ing it with PHP scripts**, a.k.a. the old school way. Easy but relatively insecure and prone to breaking (for someone at my skill level).

I didn’t like what I found: I wanted something

*   …where I **didn’t need to pay** for a service (or be crippled in forms/submits/styling if I didn’t), meaning options 1 and 2 are off the table.
*   …that **didn’t let other services process the form data**, so again 1 and 2 weren’t an option.
*   …that was **secure** and **wouldn’t give me a headache**, so number 3 was off as well.

* * *

## My solution: the middle ground

* * *

In the end, I decided to build something myself that adhered to the above points as much as possible. In summary, it looks like this:

Most people reading this will understand this chart without much further explanation. Before I dive into the details, here are the pros and cons of this setup:

*   Pro - I can do whatever the hell I want with my form: have infinite submissions, fields, and style it however I desire. The only bottleneck is the capacity of my server.
*   Pro - troubleshooting is a breeze; n8n as “central processor” is a gem to work with.
*   Pro/con - the data only passes the user’s browser, my server and (unfortunately) an email server. If you want to be super strict, you could only send it to a database you host yourself - or host the email server yourself, which is notoriously a pain in the ass.
*   Con - your server is a single point of failure. If it goes down, you lose everything apart from historical data you’ve received via email. Solution: host n8n on a different server, or do continuous offsite backups (which you should do anyway).

Here’s the step by step process. Yes, it’s really this simple:

* * *

### Form gets filled

* * *

*   Your website hosts a form that looks like this:

```
<form action="${webhook}" method="post">
 <label for="name">Name</label>
 <input name="name" type="text">
 <button data-umami-event="buttonname" type="submit">Submit</button>
</form> 
```

```
<form
 action="${webhook}"
 method="post"
 >
 <label for="name">Name</label>
 <input name="name" type="text">
 <button
 data-umami-event="buttonname"
 type="submit"
 >
 Submit
 </button>
</form> 
```

*   User fills it out, presses the “Submit” button and 2 things happen:

*   The form data is sent to the n8n webhook you declared in your form.
*   (Optional) The button click event is sent to your website analytics warehouse, [Umami](https://umami.is/docs/track-events) and event name `buttonname` in the example above.

* * *

### n8n processes

* * *

*   [n8n](https://n8n.io/) is a workflow automation tool (like [Zapier](https://zapier.com)) that [you can host yourself](https://docs.n8n.io/hosting).
*   n8n receives the form data via the webhook.
*   (Optional but recommended) Add a [Respond to Webhook](https://docs.n8n.io/integrations/builtin/core-nodes/n8n-nodes-base.respondtowebhook/) step in which you define which page the form should redirect after submitting.
*   (Optional) Add data cleanup steps to format the data you received in whichever way you need.
*   Add 2 subsequent, independent steps:

* * *

### NocoDB collects

* * *

*   [NocoDB](https://nocodb.com/) is a no-code database (like [Airtable](https://www.airtable.com/)) that [you can host yourself](https://docs.nocodb.com/getting-started/self-hosted/installation).
*   If you create a table with columns table correspond to the form fields, you can select “Auto-Map Input Data to Columns” in the NocoDB step in n8n. Works like a charm! If not, or if you want more extensive data like time, IP address, etc., you can define it for each column.
*   Use NocoDB as your warehouse for all forms ever submitted.

* * *

### Email notifies

* * *

*   I find it crucial to get notified of a form fill, because form fills are relatively rare on my sites. If you’re a business that gets multiple form fills a day, just syncing it to a CRM that gets checked daily if a more obvious choice, of course.
*   n8n has integrations with many services that can notify you (think of proprietary services [Discord](https://docs.n8n.io/integrations/builtin/app-nodes/n8n-nodes-base.discord/) or [Slack](https://docs.n8n.io/integrations/builtin/app-nodes/n8n-nodes-base.slack/), but also things like [Pushbullet](https://docs.n8n.io/integrations/builtin/app-nodes/n8n-nodes-base.pushbullet/) or [ntfy.sh](https://github.com/cryingpotat0/n8n-ntfy.sh))). I chose [email](https://docs.n8n.io/integrations/builtin/core-nodes/n8n-nodes-base.sendemail/), however.
*   My email is hosted via [Fastmail](https://fastmail.com/), which seamlessly and securely integrates with third-party applications via its [app passwords](https://www.fastmail.help/hc/en-us/articles/360058752854-App-passwords). Sending the email happens via SMTP.
*   I send an email containing the form data to myself via an alias.

And there you have it: one way to host forms yourself, without losing your mind.