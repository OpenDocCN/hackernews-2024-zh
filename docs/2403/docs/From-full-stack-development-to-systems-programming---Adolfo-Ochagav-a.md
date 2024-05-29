<!--yml
category: 未分类
date: 2024-05-29 12:39:28
-->

# From full-stack development to systems programming | Adolfo Ochagavía

> 来源：[https://ochagavia.nl/blog/from-full-stack-development-to-systems-programming/](https://ochagavia.nl/blog/from-full-stack-development-to-systems-programming/)

 <content>Back when I was pursuing my Computer Science degree, one of the big questions that kept me busy was: what am I going to do after I graduate? I felt like I could work on pretty much anything, thanks to the solid foundations I received at the university and to my real-world experience contributing to Rust. So… what to choose?

## Lured into full-stack development

After reading [Never Eat Alone](https://www.goodreads.com/book/show/84699.Never_Eat_Alone) I decided to start meeting people in order to answer my question. I reached out to friends of friends who were into programming, talked to random people on the internet, frequented local programming meetups and even organized my own. This experience shifted my focus: instead of trying to find a specific area of technology to *work in*, I started looking for the kind of people I’d like to *work with*.

The answer became clear during the last year of my master’s degree: I decided to work for [Infi](https://infi.nl/), a boutique consultancy in Utrecht doing full-stack development for scale-ups. The first thing that drew me to the company was the atmosphere, which I jokingly described as a never-ending meetup. People were genuinely curious about computers^(, and I got to spend about 20% of my working time on personal development (ranging from side projects, to open source contributions, to doing a philosophy course out of the company’s pocket).)

Because full-stack development was somewhat new for me (everything was), I thoroughly enjoyed the challenge of learning the craft. This included the human side of things, like interfacing with stakeholders, working in an agile way that suited our team (not the dreaded top-down buzzword-driven farce), doing pair-programming sessions, helping new colleagues get up to speed, etc. It felt like the right place to work^(, even though my university and open source background had given me deeper technical roots than I was using on the job.)

## Self-employment

My time at Infi came to an end when I decided to spend less time working and more time volunteering. You can find the full account of that change [here](https://ochagavia.nl/blog/becoming-a-contractor/), but the bottom line is that I became self-employed and started working on whatever projects I could find.

At first I continued doing full-stack development, but in 2022 something unexpected happened: a company in San Francisco hired me to [implement the MySQL wire protocol](https://ochagavia.nl/blog/implementing-the-mysql-server-protocol-for-fun-and-profit/)! For the first time in my life I got paid to use my technical skills in full. I researched the intricacies of an underdocumented network protocol, came up with a robust implementation, and exposed it as a hard-to-misuse library.

I was surprised. This kind of work had always felt out of my reach, something reserved for a few lucky people. In fact, after this project things went back to “normal” development work, though this time with a stronger focus on the backend.

## Systems programming breakthrough

The year 2023 was special. A few of my Rust-related blog articles caught the attention of [Prefix.dev](https://prefix.dev), who develop programmer tooling for the Conda ecosystem. They hired me to [explore the rabbit hole of containers](https://ochagavia.nl/blog/crafting-container-images-without-dockerfiles/), [boost development of their open source libraries](https://ochagavia.nl/blog/the-birth-of-a-package-manager/) and even to [implement a new dependency solver](https://ochagavia.nl/blog/the-magic-of-dependency-resolution/).

That same year I was also engaged by [Stormshield](https://www.stormshield.com/) to enhance Rust’s main [QUIC](https://en.wikipedia.org/wiki/QUIC) implementation ^(and later by [ISRG](https://www.abetterinternet.org/) (known for [Let’s Encrypt](https://letsencrypt.org/)) to create a [continuous benchmarking setup for Rust’s main TLS library](https://ochagavia.nl/blog/continuous-benchmarking-for-rustls/).)

These days I’m back to low-level container work, now for [Outerbounds](https://outerbounds.com/), with the goal of significantly bringing down startup times of [Metaflow](https://github.com/Netflix/metaflow) “flows”. Spoiler alert: results are *very* promising and I hope to publish an article about the adventure soon^.

As you can imagine, after all these projects I see myself more and more as a [systems programmer](https://en.wikipedia.org/wiki/Systems_programming). I still do full-stack development once in a while, mostly as a hobby. I like the way you can make end-users happy with something simple. It also allows me to scratch my own itch and wrap the resulting programs in a pleasant interface. More importantly, it lets me keep in touch with the wider industry.

## What about the human side?

The kind of work I’m doing now is pretty research-heavy. It’s also remote. That means lots of text-based communication and no chance encounters with colleagues. This is very different from my beginnings as a full-stack developer in an agile team! Still, I enjoy it thoroughly and would dread going back to an office job in the short term (full-stack or not).

In these circumstances, cultivating the human side of work now depends mainly on my own initiative. I try to write in such a way that co-workers feel I care about the work and about them; I try to get to know the people who work with me through one-on-one videocalls; I regularly connect with this blog’s readers and other people on the internet.

Socializing based on personal interest feels much more in line with my character than the more “forced” style you end up having in an office^(. It has also allowed me to be in touch with *very* talented people in the field. I’m looking forward to more!)

#### Bonus: are you a specialist?

Recently my post on [the undercover generalist](https://ochagavia.nl/blog/the-undercover-generalist/) went viral. My thesis was that, even if you are a generalist, it sometimes makes sense to present yourself as a specialist (for marketing reasons). In the same article I referred to myself as an undercover generalist, but after writing this blog post, I’m not that sure anymore! Nowadays I really want to focus on systems programming… which makes me feel like a specialist. Interestingly, [someone wrote](https://gusvanhorn.blogspot.com/2024/02/marketing-isnt-only-reason-to-specialize.html) in response to “the undercover generalist” saying that *“Specialization isn’t just a way to market. It’s part of becoming a generalist, which has to start with one or more specific areas of expertise.”* Does that make me a generalist *and* a specialist? I guess time will tell.

*Comment on [HN](https://news.ycombinator.com/item?id=39817026).*</content>