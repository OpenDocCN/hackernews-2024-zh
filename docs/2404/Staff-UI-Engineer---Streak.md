<!--yml
category: 未分类
date: 2024-05-27 13:19:48
-->

# Staff UI Engineer | Streak

> 来源：[https://www.streak.com/careers/staff-ui-engineer](https://www.streak.com/careers/staff-ui-engineer)

‍

[Streak](https://www.streak.com) is a CRM built on Gmail. We’re a **remote-first** team of 35 people across North America. We’re growing and very **profitable,** and we have customers that love our product. We’re currently in the goldilocks zone of having product market fit with real revenue but also a really flat hierarchy where you can ship fast.

We want to accelerate product delivery (there’s so much to build!) so we’re looking to bring on **high agency** and **experienced frontend engineers** to work on high impact product features / frontend infrastructure. We’re building a small, focused, high performing frontend engineering team that works directly with the CEO leading the product. To help build this great team, we offer incredibly competitive compensation (based on Bay Area bands) and some interesting [benefits](https://www.streak.com/careers#benefits). We want you to do the best work of your career here, so you should expect high autonomy and ownership.

Lastly, we believe the traditional interview process isn’t well correlated with an engineers’ ability to ship product. Instead, our interview process mimics what it’s actually like to work here. It’s highly asynchronous, favors writing and **involves building something useful**. We think this will give you a really good insight into our team and company and of course lets us assess if you’ll thrive here.

### Your skills

1.  **Have high agency and urgency** → you can independently make progress and can navigate through a large codebase (don’t worry, our team loves helping if you ever get stuck). Having large competitors, our differentiator is our product velocity so your default preference is to urgently ship.
2.  **Have top notch async/written skills** → remote collaboration is especially important to us so you should have excellent written and live collaboration skills. You’ll excel here if you default to async first communication. We are often writing design docs, summarizing viewpoints before meetings, recording Loom videos of our code or debugging sessions.
3.  **Are a frontend specialist** → you should have 5+ years of experience with modern frontend tooling like react and typescript. You’ll be a great fit if you’ve shipped user facing features in a modern web app to a significant user base. It doesn’t have to be millions of users, but enough to see all the edge cases.
4.  **Put in the effort** → we don’t work heroic hours to get stuff done but we do aim for a consistent, 40 hours of focused work a week. That should be more than enough to do great work and push projects over the finish line. The focused bit is usually the difference between  “almost done” and “shipped”.
5.  **Design solutions** → there are no architects here. Every single pixel and program state is not specified. You should be able to design a solution that works well with our codebase. You should be able to fill in the gaps in product specs and make product decisions when needed.
6.  **Are pragmatic** → you should be able to balance new features vs refactors vs rewrites vs just getting something working. You also can recognize when it’s worth investing in new things. You look at AI as a tool to magnify our effort, not a competitor.
7.  **Level up our team** → you’re willing to share your expertise with other engineers by reviewing PRs, writing great documentation and demo’ing your work in progress. You’re generally an optimist and you enjoy working with others collaboratively.

### What you’ll work on

Product engineers at Streak build user facing features across the full stack with a focus on frontend technologies. We work on two products, [Streak](https://www.streak.com) and our **open source** [**InboxSDK**](https://www.inboxsdk.com) (developer tool to build apps inside of Gmail, which of course we ourselves use).

You will own projects from start to finish and be responsible for shipping high quality software that users love. Here are some examples of interesting recent and upcoming technical challenges:

*   **High performance spreadsheets** → our primary UI is a collaborative realtime spreadsheet. It needs to be highly performant (60 fps) and support millions of cells of varying types.
*   **First-party-looking Gmail integration** → we wrangle Gmail to make our Streak integration look deeply native. We integrate with Gmail throughout their UI - in compose windows, left nav, search box, thread toolbars, right sidebars, message views, etc.. You’ll contribute to our popular open source [InboxSDK library](https://github.com/InboxSDK/InboxSDK) and [others](https://github.com/StreakYC/page-parser-tree).
*   **Data sync architecture** → we want interactions in Streak to feel native. Lots of work here on local data models, caching, memory optimization, rendering optimization, etc. Especially tricky given the complexity of operating in a heavyweight app like Gmail.
*   **AI** → CRM is the perfect use case for AI. We use our enormous datasets (entire sales pipelines, every email inbox in a company, etc) + off the shelf LLMs to remove all the busy work for our users. We’ve shipped several AI powered features as well as homegrown infrastructure to iterate quickly.
*   **Component system** → a CRM has a large product surface area. We need to be able to create UI quickly and make sure it has a consistent look and feel and behavior with the rest of our app. Some interesting challenges here on when and how to consolidate different components of different generations.
*   **Dev, build, test and deploy tooling** → we have some unique and challenging requirements here given our application is an extension. We push hard on build/CI times, type checking, and our incremental rollout system. These are all complicated by extension specific technical and policy requirements from the browsers. We work closely with the Chrome Extension team at Google.
*   **Beautifully crafted data visualizations** → dashboards are no joke. How do you make beautiful data visualizations while still allowing users to customize their reports to their heart’s content, without rebuilding all the UI of the various BI tools? (hint: use a sprinkle of AI).
*   **Funnel Optimization** → we have a lot of $ flowing through Streak so optimizing various funnels and user flows is high impact. Goal is to have a better user experience while converting more to our higher tiered plans.

### How we work

On the engineering team, we follow the [ShapeUp](https://basecamp.com/shapeup) methodology. We break projects into *appetites*  of either 2 or 6 weeks. Instead of an estimate for how long something will take, these appetites are a strict boundary on how much time we’ll spend on a feature. If it doesn’t ship in that time, we don’t automatically keep working on it. This encourages us to build something minimal but high quality first so that at least something of value ships to users. As an engineer, you have the flexibility to cut scope to ensure something ships. This allows us to  consistently ship features to users every 6 weeks and avoid marathon projects. If you want to follow along on one of these projects, [I tweeted](https://twitter.com/aloo/status/1386777105405419521) one whole feature end to end.

Between cycles, we generally cool down and work on bugs, refactors, etc. We also use that time to plan the next cycle. Features specs have strict boundaries but malleable internals. This encourages engineers to change the scope while building to ensure we deliver something by the cycle end. We stay away from very specific long term roadmaps. Instead we have a general direction we want to head in, but every 6 weeks, we’ll re-evaluate what’s up next.

We’re the perfect size for engineers who want to build stuff and ship without distraction. We prefer smaller teams of independent engineers that love their craft, over larger inefficient teams with management overhead.

### More about Streak

Streak frees small and medium sized business users from switching between their email, where all their work actually gets done, and the tools they are forced to use to manage that work (like Salesforce). We recognize that sales, hiring, partnerships, fundraising all happens in your inbox - so we’re building a meta layer on top of email that lets your team push these processes forward.

Streak is a growing, but more importantly, profitable company. Because of this we have the best of both worlds - nimble enough to have growth opportunities like a startup. But we're not under the gun to raise a round in 9 months or die, so we can focus on building a foundation and a company that works for the long-term.

### Join us

We hope the detail in this doc gave you a good sense of who we are and what we’re working on. Our goal was to make it an efficient use of your time. If you found yourself nodding along while reading, we’d love to hear from you.

Our interview process is non-standard but, we believe, more enjoyable and much more effective. The majority of it will be an asynchronous and paid project. The **first step** is to hit the apply button above or [email the CEO](mailto:aleem@streak.com?subject=Staff%20UI%20Engineer%20application) directly (we’ll read either). Please include:

1.  a Loom video that goes over why you think Streak would be a great fit for you and vice versa
2.  a code sample of work you’ve done (open source contributions, features in launched commercial products, side projects, etc.)
3.  a list of products you’ve worked on or a resume or a LinkedIn url

### Compensation

The total compensation for this role is $175,000 (Senior) - $400,000 (Staff). This includes a 10% bonus component based on personal and company performance. Compensation (base and bonus targets) are revisited every 6 months coinciding with performance reviews. Bonus targets are aggressively ramped with tenure and it's not uncommon for your bonus target to become a large portion of your total compensation. Unlike stock based compensation, bonuses are highly liquid and paid out every 6 months.