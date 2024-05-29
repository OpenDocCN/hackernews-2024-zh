<!--yml
category: 未分类
date: 2024-05-27 14:45:10
-->

# Starting a side project as a creative studio | Hellotime

> 来源：[https://www.hellotime.io/blog/starting-a-side-project-as-a-creative-studio](https://www.hellotime.io/blog/starting-a-side-project-as-a-creative-studio)

## Running side projects while doing client work is possible, fun, and valuable. Here are our lessons learned while building Hellotime, the resource capacity planning app born out of our studio.

We have worked with clients since 2012\. We’ve learned how to do that the hard way. We always loved working on products and always thought about starting our own.

Our first attempt was with [Done](https://www.mozestudio.com/work/done-to-do-app-for-minimalists/), a minimalist to-do app for Mac. We started it in 2019 to develop skills in Swift and Xcode. At that time, we realized that “unconventional” and “simple” on the surface meant it “hard” under the hood. Our second project was Hellotime, our team planner app for teams doing client work.

We’re sharing here our lessons learned and why we think side projects are good for creative firms. I won’t write about how to build a scalable and high-growth business. Many more qualified people have written about that. This is about to build something a team is proud of and that drives progress towards their goals.

Have fun, and I hope building a side project inspires you the way it did for us!

## What is a Side Project anyway?

First, let’s start with the definition of a side project.

> A side project [...] is an endeavor pursued alongside one’s main activities, often during free time. It can be a creative, entrepreneurial, or learning initiative that allows individuals to explore their interests, build new skills, and express their creativity.
> 
> [What is a side project?](https://www.perplexity.ai/search/what-is-a-BTqoeDxBQyyxlHnWpx4CiQ?s=c#0), Perplexity AI

For an agency, this means doing something alongside client work (the main activity). This could involve building a new product, a pet project, or conducting research. In other words, anything that is not paid by clients but drives new skills, profits, or even fun.

## Define Success

tl;dr: It’s not solely about the money.

With these premises, it’s clear that “success” isn’t binary. Do you aim for your team to develop new skills? That’s fine. When we launched Done, our focus was on learning new skills. Did Swift and Xcode become part of our studio’s skill set? Well, no. We hit the principles of the symmetry of ignorance. We were good at React, but native development was an entirely different challenge. Thanks to that side project, we decided to further invest in our existing React skills. Failure or a success? The definition of it is up to you, depending on your goals.

If you read between the lines of why Trello was started, it was about keeping the team motivated.

> Around the time of Fog Creek Software’s ten year anniversary, I started thinking that if we want to keep our employees excited and motivated for another ten years, we were going to need some new things to work on.
> 
> [Announcing Trello](https://www.joelonsoftware.com/2011/09/13/announcing-trello/), Joel on Software

Of course, Joel Spolsky didn’t just say, “you guys play around with the first thing that comes to your mind.” He provided direction, a method, and business ideas. However, note that he didn’t start the article with the common rationale of “making money.

As per Hellotime, we started because of three factors:

*   We were tired of existing resource capacity planning tools (frustration).
*   We always wanted to try building our own product after doing so for our clients (motivation, willingness to learn new skills).
*   We could invest some people’s time in an internal project because the start of another project was suddenly postponed (opportunity).

This time, we were mature enough to understand that building a resource capacity planning app wasn’t as simple as “investing three days here and there.”

That means, you should first ask yourself and share with the rest of the team the following:

*   Why are we doing this?
*   What kind of progress are we looking for? (e.g., money, motivation, new skills)
*   What will suggest that we’ll have to stop?

## Start from your own need

For side projects, the question of “why are we doing this” often relates to a challenge the team is encountering. It might be, for example, a recurring issue within the scope of their actual work. As many suggest, it’s easier to focus on a problem that’s already known. Diving into a new topic from scratch—a problem you haven’t faced—is a much larger undertaking.

From this perspective, frustration can actually be quite good.

> Basecamp was born back in February 2004\. It was conceived a few years earlier, inspired by a need that wasn’t being filled and a frustration with the tools of the trade.
> 
> [Basecamp: The origin story](https://medium.com/@jasonfried/basecamp-the-origin-story-f509fdd725f8), Jason Fried

For years we had refused to use planning tools, feeling they weren’t fit for studios like ours. We used Google Sheets for a few years, but it really became a pain to maintain and turned laggy. I remember once, speaking with Matteo on a video call about the idea of doing it from scratch as an app. We said: resource capacity planning is something we do daily, it’s crucial for our business, the current tools are bad. Let’s try to build something great. That’s how we started Hellotime.

For that reason, there’s a need to ask yourself a question like “what’s causing your frustration?” You already know. Reflect on it and ask yourself: how big is this problem? How often does it occur? As Des Traynor suggests, [be wary of solving a small, rare problem](https://www.youtube.com/watch?v=P6pQyB6ACrk).

## Define a process

Your experience may vary, depending on the type of project you plan to undertake. In our case, even a simple to-do app for MacOS proved to be a lot of hard work. At the beginning, it’s very common to underestimate the amount of effort needed to create something.

For Hellotime, we were prepared. We knew that making something simple and easy to use often requires a surprising amount of work. When talking to early users, we sometimes heard: “We were about to build something similar. Then we found Hellotime.” One very honest person added “We tried building our own tool, but it quickly became clear that we couldn’t.”

That’s why, when our development team hit a wall, I would say, “Good sign: it means not just anyone could do it in a short time.”

For Hellotime, we began with rough documents and sketches. These outlined:

*   Why are we doing the project?
*   Which problems do we want to solve and for whom?
*   What is the project’s scope and the use case scenarios?
*   What will the User Interface look like?
*   How are we going to promote our project?

We faced many challenges. First, the technological challenge of rendering an infinite timeline canvas with rich elements. Second, the go-to-market challenge of introducing a brand-new product to the world. So, we made a plan.

### Prove you can do it

The first step was the Proof of Concept. Our aim was to create an infinite timeline canvas where we could place projects and people. We set aside an initial six weeks to build the basics. By the end, we didn’t have the sign-up form ready, but the timeline was working. Then, we spent a few more weeks adding what was missing to start testing it ourselves.

### First, convince yourself

The second step focused on internal use. The goal here was to develop a working app for our studio’s use. The success criteria were whether the app could convince us, so we could then suggest others to try it out. This required some fixes, as what looked good on Figma didn’t always work well when built. In the end, we were happy with it; we used it ourselves and it met most real-world use cases.

### Find others like you

The third step was the Private Beta. We wanted to find out if others shared our need for a simple, easy-to-use, and sleek resource capacity planning app. Could we test our assumptions before building the product? Yes. And no. We began, of course, by discussing with other agencies about their use of resource capacity planning apps.

Meanwhile, we started by teasing our personal network on LinkedIn. Later, we posted on Reddit and BetaList. Our aim was to get 10 companies using our tool as their go-to for resource capacity planning.

We made an effort to talk to everyone we could. People who signed up, those who showed interest but didn’t follow through, and those who suddenly stopped using it. From this experience, we learned a lot. We began blending ideas from our personal preferences with user feedback.

Then comes the launch. People have shown their support into Hellotime. This encouragement is guiding us as we head towards our public launch, which is right where we are now.

## Speak out loud

When building something new, your existing network is your best ally. Everyone has their preferred platform. For some, it’s Twitter; for others, Reddit, Indie Hackers, or LinkedIn. The key is to get yourself in a network. This can be invaluable for validating your idea. A thoughtful post that shares what you’re working on, your learnings, and your drivers can be a good way to start.

### People are okay

From my experience, people are naturally curious about new things. If you capture their interest and they trust you, they’ll join a waitlist. With Hellotime, we gathered dozens of access requests. Even now, some sign up but forget to complete their workspace setup when invited. We tried to address this with a double opt-in and an initial screener to gauge real interest. We also made sure to focus on those who fit our Ideal Customer Profile (ICP).

### Try try try

Your call for beta testers may not always get noticed, depending on the channel. The response can vary. When seeking early users for Hellotime, my Reddit posts received mixed reactions. Sometimes interest, sometimes indifference. I still don’t know why; maybe because of the wrong subreddits, posting times, or other reasons. The advice is to persist. Crafting messages that resonate with your ICP will maintain a consistent outreach.

## Listen carefully

Starting a new project? Everyone’s going to tell you to “talk to people”. That’s why random strangers DM you on LinkedIn, asking you to jump on a call. The key concept you can get from reading excellent books like “The Mom Test” is to explore context first. Instead of just talking about your idea, focus on discussing their life.

### You are wasting your time

However, what I’ve come to believe is that user interviews represent one side of the coin. They help you gain more knowledge about the subject you’re interested in. However, it is often difficult to separate the signal from the noise. Two examples stand out: when we didn’t even have a basic version of Hellotime, I began speaking with agencies (our ICP). I wanted to understand how they managed their team’s planning on projects.

The first person I spoke to told me, “These planning tools are useless.” The second one said, “I don’t really care; we’ll stick with product X until the end of time.” The few people I interviewed when I didn’t have a product yet were generally cold.

### Maybe not

I began to realize that I would need a lot of time to interview many people. Meanwhile, we had a basic version of the timeline. We had few features but in line with our idea of a simple and beautiful UI. So we tried something different. We posted a screen recording of the app along with a Google Form for those interested. What happened was that many people signed up, and some said, “That is exactly what I was looking for.” I reached out to them, discussed their needs, and shared early access to our app.

What I've learned from this is that you can spend a lot of time searching for individuals one by one to see if they have the problem, or you can launch a product teaser to attract those who already face it. The latter is much faster.

### People sometimes don’t tell the whole story

Another curious thing happened. Once, I interviewed a person who signed up, played around, and then left. She told me, “Cool, but I was looking for something slightly different.” After further asking, it turned out to be about a minor difference in how we displayed data. It was strange feedback. That person had the problem and saw the value in the solution but dismissed it for an apparently trivial reason. I was confused.

After a few months, that team set up all their projects and people and began using our app daily. Later, speaking frankly, they told me, «We weren’t sure about using a tool from a competitor agency. Then we said, “why the hell not."»

People don’t always tell the whole story. And that’s totally okay. All of this made me think about how true it is that the only way to validate anything is to build it and see what happens. It’s not just “Build and they will come"; it’s more like “Build. The journey starts from there".

> If you want to see if something works, make it. The whole thing. The simplest version of the whole thing — that’s what version 1.0 is supposed to be. But make that, put it out there, and learn. If you want answers, you have to ask the question, and the question is: Market, what do you think of this completed version 1.0 of our product?
> 
> [Validation is a mirage](https://basecamp.com/articles/validation-is-a-mirage), Basecamp blog

## Iterate. Think. Go next.

If you’re building something others can use, the best thing can happen is that people reach out to you. They will tell you the product is what they were looking for, and they will share what is missing. Iterating helps increase adoption, creates momentum, and extends your reach.I

It’s also good to determine when to proceed further and when to stop. When running a side project, starting frugally allows you to manage at your own pace.

Sahil Lavingia is the founder of Gumroad and author of “The Minimal Entrepreneur.” In the book, he discusses avoiding common pitfalls such as running out of money or energy. There’s a part of the book that particularly caught my attention. That’s when Sahil mentions that projects don’t usually fail because there isn’t enough money or no market need. They fail because the founder runs out of energy. Energy for trying, to change, to adapt. In the spirit of the Minimal Entrepreneur, there isn’t a fixed direction. Rather, a flexible journey where the community’s input shapes the vision.

A project doesn’t have to be successful no matter what. This leads to asking yourself every now and then: “Am I still excited about this project? Does it make me happy, and am I adding value by sticking with it?". If something feels off, it’s good to reconsider the direction and make changes.

That’s what we’ve learned so far in our journey building Hellotime. We’ll keep sharing our lessons learned along the way. Feel free to sign up and never miss our updates.