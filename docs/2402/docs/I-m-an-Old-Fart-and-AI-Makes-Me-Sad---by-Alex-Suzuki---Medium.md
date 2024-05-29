<!--yml
category: 未分类
date: 2024-05-27 14:55:12
-->

# I’m an Old Fart and AI Makes Me Sad | by Alex Suzuki | Medium

> 来源：[https://medium.com/@alex.suzuki/im-an-old-fart-and-ai-makes-me-sad-06003bfb6750](https://medium.com/@alex.suzuki/im-an-old-fart-and-ai-makes-me-sad-06003bfb6750)

# I’m an Old Fart and AI Makes Me Sad

Previous major advancements in Computer Science and its applications always felt empowering to me, AI feels different.

Dall-E 3, “A drawing of a software engineer overwhelmed by recent advances in Artificial Intelligence”

To find out why I feel this way, I had to look into the past couple of decades and compare the major technological advances to today’s movements.

## Personal Computers [80s]

PCs were my gateway drug into technology — they got me into gaming and eventually programming. I have fond memories using tools like MEMMAKER to squeeze more memory from my machine just so I could run a pirated copy of some new game on my DOS box.

PCs came in different shapes, sizes and architectures. They had many names and were sold by different vendors. You were mostly free to do what you wanted with these machines. You could swap out parts. You could run games, you could write software, or do something totally different with them.

From an engineer’s viewpoint, the inner workings of these machines were comprehensible. With some skill and dedication, you could even build your own.

## World Wide Web (a.k.a. The Internet) [90s]

Suddenly the bandwidth of your internet connection was more interesting than the clock rate of your computer. I spent hours in Internet Cafés chatting with strangers, who like me, were exploring this new and strange medium.

The early Internet felt accessible to me. It was built on open, interoperable protocols like TCP/IP, HTTP, SMTP etc. You could buy a book or visit SELFHTML, open an account at Geocities and create a website. If Geocities went down, you could move your website somewhere else, or even self-host. No single company or entity owned a significant chunk of the Internet.

## Smartphones [mid-2000s]

A new form factor and interaction paradigm is introduced. Even though we saw similarly sized devices and touch screens before, the iPhone was the first device that tied it all together, helped by ubiquitous mobile internet access.

Smartphones are, for the most part, accessible in the same way as other computers are. They are small, connected, programmable computers with a wide range of input sensors.

I understand how smartphones work, I can program them, I know their limitations.

# How AI is Different

Don’t get me wrong, I’m excited about AI and blown away by stuff like the just announced [Sora](https://openai.com/sora) text-to-video model. But at the same time, I feel left out.

## **AI is Opaque**

I want to understand how things work. AI feels like a black box to me. The amount of papers I’d have to read and mathematics that I’d have to ingest to really understand why a certain prompt X results in a certain output Y feels overwhelming. Even some top scientists in the field admit that we don’t really understand how AI works.

> “If we open up ChatGPT or a system like it and look inside, you just see millions of numbers flipping around a few hundred times a second,” says AI scientist [Sam Bowman](https://cims.nyu.edu/~sbowman/). “And we just have no idea what any of it means.”

To me as an engineer, that is just incredibly **unsatisfying**. Without understanding how something works, we are doomed to be just users.

## AI is Not Approachable

Sure, on the outside it is — anyone and their dog can open a ChatGPT session or throw some JSON at OpenAI’s APIs. What I’m talking about is gaining access to the core technological foundations that make AI possible.

Flipping all those numbers to get the result (inference), and especially determining those numbers in the first place (training), requires a vast amounts of resources, data and skill.

AI is not a shovel for little people.

## AI is Not Open

If you’re like me, you might have some ideas for cool stuff to build with AI. But in doing so, chances are you’ll end up building a *GPT Wrapper*.

What is a GPT Wrapper, you might ask? It’s any bit of software, SaaS, whatever you want to call it, that relies on someone else’s AI product that you can’t easily replicate or exchange (ChatGPT, for instance).

If I build an app that needs persistence, I might use Postgres and S3 for storing data. If those are no longer available, I’ll use another relational database, key-value store, distributed filesystem, whatever. But what if OpenAI decides to revoke access to that API feature I’m using? What if they change pricing and make it uneconomical to run? What if OpenAI extends their offering and makes my product redundant?

Old man yells at cloud (credits: [https://knowyourmeme.com/memes/old-man-yells-at-cloud](https://knowyourmeme.com/memes/old-man-yells-at-cloud))

So am I just like that angry old man shaking his fist at the Cloud in anger? I hope not.