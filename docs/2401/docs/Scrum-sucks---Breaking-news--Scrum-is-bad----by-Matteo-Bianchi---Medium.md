<!--yml
category: 未分类
date: 2024-05-27 14:48:16
-->

# Scrum sucks.. Breaking news: Scrum is bad. | by Matteo Bianchi | Medium

> 来源：[https://blog.mb-consulting.dev/scrum-sucks-9960011fc5cf](https://blog.mb-consulting.dev/scrum-sucks-9960011fc5cf)

# Scrum sucks.

## Breaking news: scrum is bad.

Any developer right before sprint review

If you are reading this post, you probably worked following (some form of) Scrum but in case never did: take a seat and be my guest.

Let’s start from the very beginning.

# What is scrum?

> Scrum is an Agile project management system that “helps people and teams deliver value incrementally in a collaborative way.”

Cit. [Scrum.org](https://www.scrum.org/resources/what-scrum-module)

As for Agile, if you never read its [Manifesto](https://agilemanifesto.org/) (2001), I define it as a **lean list of good practices** to follow in software development.

> Agile is not: the Holy Bible of Software Development, a dogmatic set of strict rules, Jira tickets or Agile Coaches running around your company.

Late disclaimer: definitions are flawed, by definition. (now read that again)

I will openly accept any criticism about the way I defined Scrum, Agile and any other term too.
I only ask you to **read the whole post** before angrily typing in the comments!

# Theory vs Practice

Agile is the opposite of Waterfall, which has been the old school way of crafting software until the ’90s.

Waterfall as pictured by Agile practitioners: a pile of sh*t

Agile is iterative, Waterfall is sequential.
Agile is lean, Waterfall is heavy.
Agile is fast, Waterfall is slow.
Agile is code-first, Waterfall is doc-first.

I could go on for ages but I will not, you got it.

Typical agile practitioner working on the same task for 4 sprints in a row.

Agile comes from a (very) good place and it was a revolution that transformed software development from a sequential nightmare in the discipline tons of engineers, including myself, happily practice today.

I would never question the authority held by the people who theorized it, since the **real issue is Scrum implementation** in our real not-so-ideal world.

## Ceremonies

Scrum’s most fundamental term is: **sprint**.
Sprints are basically **pre-defined time-slots** (typically lasting two-weeks). During this amount of time, aside from **development cycles**, the Scrum team is required to follow some **ceremonies**.

Midsommar, must watch, add it to your list!

*Sidenote: I always disliked this religious term used to define these kind of meetings, but ok.*

*   Sprint Planning — as simple as it seems, a long session (up to 4 hours) to plan items/user stories that need to be delivered during the following sprint, **involving the scrum team**.
*   Daily stand-up meetings — a daily, **SHORT**, meeting (around 15 minutes) where the ***small* development team** (like 3 to 7 people), communicates their progress and underlines blocking issues, both **without going into deep detail**.
    *This is the time where you can ask Alice to have a separated meeting about that server-side caching issue and to Bob about that date parameter being a string and not Unix time.*
*   Sprint Execution — People **autonomously pulling tasks**, then execute them and solve eventual issues through informal meetings, pair-programming sessions, code reviews and intensive copy-paste from ChatGPT.
    *Testing? Part of the execution.
    QA? Yep.
    Documentation? You know the answer.
    Operations / DevOps / Infrastructure tasks? Guess what.*
*   Sprint Review — a long meeting (between 1–2 hours) which aim is showcasing the sprint outcome to stakeholders and **getting feedback** from them.
*   Spring Retrospective — an internal analysis of what has been executed, with focus on impediments, in order to remove them for future sprints and overall **continuously improve** processes, practices and tooling for the next iteration.
*   Backlog Refinement — not really a ceremony but roughly 10% of the time that should be spent by the team, during a sprint, on backlog refinement to: break down, explain, find gaps, **provide a definition of done and collaboratively prioritize** the tasks.

## People

These are the individuals, or group of, identified by Scrum:

*Stakeholders* — They have a stake in the game, they want to be **informed, involved** and they benefit from the product, whatever “product” means for your company.
*Yes, everything is a product if you stretch the definition enough.*
They can be end users, other teams or even another business.

*Product owner* — should act as a **bridge between the stakeholders and product developers**, translating requirements into actionable items, understanding business both domain and business value, managing the backlog and maximizing the delivered outputs.

*Developers* — should be the people in charge of operational duties, including but **not limited to software development**, in a self-managed way, being able to **take technical decisions steering the product** on the right path.

*Scrum master*— should facilitate communication, monitor and track progress, determine and evaluate risks, **educate and coach** the self-managed team about scrum to render it effective.

Ahhh yes, the famous multitasking productivity of a Scrum Master

# Scrum is broken

*Wait. So these ceremonies and roles are so complex that a* [*Certified*](https://www.scrumalliance.org/get-certified/scrum-master-track/certified-scrummaster) *or* [*Professional*](https://www.scrum.org/professional-scrum-certifications/professional-scrum-master-assessments) *Scam Master is needed?*

Ahhh, sweet summer child.
The obvious answer is, in short: no.

Wanna know why?

> We do not live in an ideal world where theory happens exactly as written in the books.

You could have guessed it by the highlighted phrases in these last paragraphs, but in practice this is what *really* happens:

Another Jira ticket? Let’s GOOOOOOOOOO

Planning, regardless of the tooling, takes ages and you end up with pre-assigned tasks, measured in story points.
These points, that should be there to help **getting an idea of the complexity** of a certain task, are instead used to measure time.

Well done, Sherlock!
Your team is now focused on occupying time by filling their schedule, instead of focusing on delivering features and you cannot even measure their **velocity.** Velocity should be useful to understand, based on previous sprints, what is the relative amount of product progress your team can deliver.

Ok, but why are we not using a [Fibonacci scale](https://en.wikipedia.org/wiki/Fibonacci_scale_(agile)) again? Or what about a game of [planning poker](https://www.planningpoker.com/)?

Me, after you told me your company uses Fibonacci scales for stories

Wait, don’t resign yet. I have the definitive solution, I swear!
Just give each task a clothing size to simplify its measurement.

What a wonderful idea.
We simply need to discuss if that is that gRPC API is an L or an M tasks, meanwhile the only L we are getting is the time we lost on picking a method for this **unproductive millimetric measuring**.

Big L, micromanager!

Thinking about a possible scenario: Product Owner and Scrum Master are out of office.
One is on sick leave and the other has a well-deserved PTO or holiday.
Does your team still attend the stand-up meeting? I doubt it.

What? You set individual velocity as a goal to meet each sprint?
Be prepared for crappy code and incomplete, but still automated, testing.
Once you **set a static metric, people will gradually adapt to go around it**.
Oh, and say goodbye to pair-programming and collaboration, everyone is too focused earning those juicy story points.

A task is too difficult to break down and requires extensive analysis that may involve multiple teams and last a couple of days?
No one is going to self-assign that task as they **fear** blame and looking unproductive.

Writing tickets for 10-minutes tasks such as formatting some file, updating a URL and remove unused imports?
Welcome to **ticket-driven development** where even decluttering takes ages.

Time for a pretty common case: someone found a bug during the sprint.
Houston, we have a problem, let’s talk with… who?

Team Leader?
Scrum master?
Tech Lead?
Product Owner?
Service Owner/Manager?
Project Manager? Yes, I experienced having a PM in a team running Scrum, don’t ask why, it is very real and I have proof.

Who you gonna call?

What a **fractioned leadership** we have here!
So much for “self-managed team”, huh?

Imagine you aligned all these people about your issue and now guess the amount of time required for them to figure out their piece of responsibility and provide a solution.

You, leaving the room without a clue about the solution

Good luck, see you next sprint!

Scrum is often seen by management as an elixir of productivity and a cure for any under-performing team, but reality is pretty much the opposite.

> Scrum does not solve your team performance issues.

A slow team can become even slower and believe me, that **burnout chart** will be painful to watch.

The — ??? now I’m really burnout — chart

Whoops, I meant burndown, pun intended.

# Is there a way to make Scrum work?

I really believe there is one, but I have never seen a single engineer happy about their company Scrum implementation.

Bingo.

> **Way of work should be defined at a team level by its people, not by the company.**

The Agile prophecy is finally fulfilled!

*   Time between deliveries is finally short and it is **flexible** too!
    No need to call it “sprint” and make it look like an endless run against time.
    Quality of software produced will rise sensibly as any change to the scope is free of the constraints of rigid time slots.
*   Estimates are done as intended: intuitively **based on previous experience**, validated at a team level.
    In software development more than anything, time must be considered as a relative measurement.
*   Planning involves, management aside, people that are **actually executing** whatever is being planned so they can discuss it and find flaws when there is still time to handle them gracefully.
    Work is pulled by each individual contributor and not assigned by any leader/manager.
*   Execution is **empowering.** Each developer, based on experience, has a certain degree of technical freedom and feels business responsibility towards the product, their colleagues and stakeholders.
    Reduction of dependencies, makes your development team independent and, believe it or not, way faster.
    Daily updates finally reflect the status of the issue tracker, information flows and documentation gets written.
*   Daily **alignments are short** and really give info about the **work in progress**.
    No one feels the need of filling their todo list with minor filler tasks or lying about being stuck or ahead of schedule.
*   Blockages are solved **within the team, as a team,** without interacting with lasagna-layered management and by taking ownership of each decision, whatever the outcome may be.
*   **Results are shared** in complete transparency and really show the status even to the most difficult stakeholders (typically “top” customers). Objective is to look for real feedback, not for headpats.
*   Review of what has been executed is **honest and blameless** so people can stop manipulating data to make it look like they did a relatively good job and start focusing on solving the root causes of issues encountered.
*   Backlog is finally a **shared responsibility** **and a continuous effort**.
    You will be amazed by the impact of this simple (but not trivial) bullet.

# The fix

*   **Take the retrospective seriously**, as a moment to reflect with a clear mind, over what turned out to be a good strategy, how to iterate on that and praise the team for the job done.
    Yes, of course you also need to thinker about what went wrong, why and how to improve it.
*   **Ask for advice** on how to improve rather than feedback.
    People love to give advice (sometimes even without prompt but that’s a different story), it helps avoiding empty feedback as in “all good” or “all bad”, when you getting a good advice will be immediately actionable, asking someone for advice makes them feel more trustworthy than when they are asked to give “just” a feedback.
*   **Trust the team,** be honest and stop trying to micro-manage it.
    Try to delegate more and be flexible, you will see that giving freedom results in a healthier environment where is easier to learn how to be responsible, even when the cat (aka management) is away.
*   **Be aware that metrics will lie** at some point,if you don’t look give a critical look at them, don’t fall in the usual confirmation bias.
    Time for one of my favorite quotes: “if you torture the data long enough, it will confess to anything” — Ronald H. Coase.
*   **Encourage mistakes**, own them and fix them without fear of undergoing performance reviews. Stop scrutinize and question that dev that made a mistake during the previous sprint, it will only worsen their performance and create a toxic environment to work in.
    You should examine the mistake but never the person.
    You see a dev struggling? Try to ask them how they are doing, give them some time off and look at the results on the next iteration.

I know that some of you think that what I said is just bullshit.
*If you write that angry comment at this point, I won’t blame you.*

Let’s now read the Manifesto again:

> **Individuals and interactions** over processes and tools
> 
> **Working software** over comprehensive documentation
> 
> **Customer collaboration** over contract negotiation
> 
> **Responding to change** over following a plan

This time you cannot say that title was a clickbait:

> Scrum really sucks.
> 
> Unless you make it Agile.

Which means: flexible, adaptable, collaborative, built around people and focused on results rather than metrics.