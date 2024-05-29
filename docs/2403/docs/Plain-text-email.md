<!--yml
category: 未分类
date: 2024-05-29 12:35:09
-->

# Plain text email

> 来源：[https://notes.ghed.in/posts/2024/plain-text-email/](https://notes.ghed.in/posts/2024/plain-text-email/)

22/3/2024

# Plain text email

In the mid-1990s, a war was waged in the email inboxes of those who were already online. It was at this time that HTML email arrived, creating heated discussions in BBSs, mailing lists, and IRC channels.

It’s very likely that most — maybe, you — don’t know what I’m talking about. Let’s go back, so we can all be on the same page.

Email messages can be sent in plain text (`text/plain`), just like files saved in Notepad, or in HTML (`text/html`). In this second format (or [“MIME type”](https://developer.mozilla.org/en-US/docs/Web/HTTP/Basics_of_HTTP/MIME_Types)), the messages are created as if they were pages of a website, which opens a Pandora’s box, I mean… many possibilities, such as rich formatting and images mixed with the text.

HTML email has [some obvious disadvantages](https://subversion.american.edu/aisaac/notes/htmlmail.htm), such as less security due to hiding links and loading remote media. An incidental problem is that, unlike web browsers, email clients/apps do not follow web standards — each one renders HTML differently, which makes the design of newsletter layouts, for example, a hellish endeavor.

Another problem with HTML is that messages in this format are heavier, because they have invisible parts (headers and the HTML code itself) and visible (images, in particular) that the pure text counterpart doesn’t have.

Nowadays, this may not be a problem, thanks to the ubiquitous fast internet connections. In the 1990s, with the very slow links of dial-up, it mattered.

[Mauricio Teixeira](https://hachyderm.io/@badnetmask), then an employee of an ISP in Belém (PA), was so angry that, with some friends, he idealized the “ASCII Ribbon Campaign”, a protest against the HTML email graved in the signature of all the messages they sent back then.

([ASCII](https://en.wikipedia.org/wiki/ASCII) is an old character encoding standard. Years ago it was surpassed by UTF-8, capable of handling characters from languages other than English.)

Although he lost the records of almost 30 years ago, one of his messages, dated June 17, 1998, was saved and [illustrates a web page](http://arc.pasp.de/) that retains the memory of the ASCII Ribbon Campaign. Obviously, the signature had the drawing of an actual ASCII ribbon, which I will try to reproduce here:

```
/"\
\ /  CAMPANHA DA FITA ASCII - CONTRA MAIL HTML
 X   ASCII RIBBON CAMPAIGN - AGAINST HTML MAIL
/ \
```

I spoke with Mauricio. Nowadays, he uses Gmail in his browser and sends HTML messages.

“I was someone else at that time, a more extremist guy,” he told me. “Not that I disagree with the basic idea of email in plain text. I do. I just wouldn’t do what I did again, the way I did.”

Mauricio was lucky to have his first contact with computers very early, in BBSs of universities, before the arrival of the commercial internet in Brazil. “I came from an environment where there was only plain text,” he said.

Working in an ISP (“I knew by memory all the [email] rules, the RFCs”), immersed in the Linux terminal all day long, and reading email in plain text led him to the ASCII Ribbon Campaign.

His biggest issue was with emails that did not come with a copy in plain text. HTML emails can be sent with a plain text copy, useful for cases such as Mauricio’s in the late 1990s, that is, people who use email clients unable to render HTML. “We were angry because of that,” he recalled.

Nowadays, he points out another problem of email in HTML: lower accessibility. HTML layouts or even complex formatting can make it difficult for people with motor or blind limitations to read.

The campaign did not have much repercussion, despite the records that resist to this day on obscure websites (and in [an entry on Wikipedia](https://en.wikipedia.org/wiki/ASCII_ribbon_campaign)). According to Mauricio, at that time it was more difficult to “spread ideas that were considered alternatives.”

When I asked if another factor for the campaign’s failure could have been the fact that it tackled a non-issue, he agreed: “For most people, their ability to create a prettier email was more important than the other crazy comrades, who kept complaining that that was wrong.”

Mauricio, who works at Red Hat in the US, replaced CLI email clients for the webmail when his professional demands took him out of the office, that is, when he needed access to email on the go. (That was before modern smartphones.) “Nowadays, much of my work is in the web browser,” he said. “I accepted that life is in the browser now.”

`***`

Despite the absolute dominance of HTML email, plain text resists. Some popular clients/apps, such as Gmail, Apple Mail, and Outlook, allow writing messages in plain text, but not reading when the HTML version is available.

In Thunderbird, you can live only in the plain text domain. An extension that enables the HTML version when there is no plain text or it’s messy, such as [Allow HTML Temp](https://addons.thunderbird.net/pt-BR/thunderbird/addon/allow-html-temp/), is recommended. Evolution, for Linux, also offers this experience.

Rare, there are still email clients that only work with plain text, such as Claws or The Bat. KMail, from the KDE project, even allows you to see messages in HTML, but the mandatory default is plain text. (The bar/button to convert messages into HTML is a visual atrocity.)

I’ve been writing emails in plain text for a few years. I’d love to be able to read them like that too. I can set up my favorite font and read without distractions.

In the real world, HTML enables bad email marketing and the deepest instincts of bad taste in people who can’t resist a text formatting row of buttons.

If you want to join the ASCII Ribbon gang, the [Use plaintext email](https://useplaintext.email/) website is a great starting point.