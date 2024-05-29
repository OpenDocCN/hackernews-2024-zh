<!--yml
category: 未分类
date: 2024-05-27 15:14:16
-->

# Long Term Refactors - Max Chernyak

> 来源：[https://max.engineer/long-term-refactors](https://max.engineer/long-term-refactors)

Big (or “Long Term”) refactors are hard to pull off in a busy company. To succeed, we must:

*   Convince business that it’s worth the delay.
*   Decide what features will have to wait.
*   Produce regular status updates and ETAs.
*   Justify the refactor as we go. Is it the right approach?
*   Keep ourselves from burning out.
*   Allow time for the team to digest and review the huge diff.
*   Fix a bombardment of QA issues.

And we better do this all quickly, because god forbid original and refactored code coexist!

Is this really the only way? Feature freeze, a rush, a buggy rollout, and likely burnout?

## The Other Way

I have a theory that long refactors get a bad rap because most of them take far longer than we expect. The length leads to stress, an awkward codebase, a confused team, and often no end in sight. Instead, what if we *prepared* an intentional long term refactor? A few years ago, I began trying this method, and it has led to some surprisingly successful results:

*   We didn’t need to negotiate business timelines.
*   We didn’t need to compete against business priorities.
*   The team quickly understood and even took ownership of the refactor over time.
*   There was no increase in stress and risk of burnout.
*   PRs were easy to review, no huge diffs.
*   The refactor was consistently and collaboratively re-evaluated by the entire team.
*   We never wasted time refactoring code that didn’t need it.
*   Our feature development remained unblocked.
*   The team expanded their architectural knowledge.
*   The new engineers had a great source of first tasks.
*   We rolled out the refactor gradually, making it easier to QA, and reducing bugs.

Long-term refactors involve the whole team from the beginning, which is one of their most powerful aspects. So far, I’ve participated in ~10 big refactors using this method across 2 companies with at least 3 different teams, and I’ve yet to see it go wrong. Here was our approach.

## Prerequisites

To start, you should have the following:

1.  An experienced software engineer with a vision for the refactor.
2.  A team of software engineers at various levels of expertise.
3.  An internal knowledge base. (Any of Github Wiki, Notion, Confluence, Markdown files, etc)
4.  Less than ~5-10 long term refactors already in progress, depending on their scope.

## Process

Almost every big refactor I’ve encountered follows a semi-consistent pattern. What makes a refactor big is the sheer number of times you must apply the pattern. In an ideal world, this labor is divided. Unfortunately, the refactor often requires case-by-case decision making. My proposed process is centered around explaining the refactoring idea to your colleagues, so that they can also make decisions.

NOTE: The process is for the “experienced engineer” from prerequisite #1.

1.  **Identify code that should be refactored.**
2.  **Identify the refactoring pattern.**
    Explore the codebase to identify a common pattern of required changes. A rough idea is fine for now. It’s okay to ignore special cases and focus on the commonalities.
3.  **Implement an example of the refactor.**
    Find the smallest representative sample that you can apply your rough pattern to, and refactor it. This is where you want to be extremely diligent. Experiment and thoroughly refine your pattern. Don’t skimp on best practices. Follow [4 reasons to leave a code comment](https://max.engineer/reasons-to-leave-comment). Convey [how, what, and why](https://max.engineer/maintainable-code). Make it your best work, because it’s going to become the primary reference for the rest of your team. Submit a merge request.
4.  **Prepare the codebase for the refactor.**
    Now that you’ve tried the refactor yourself, made decisions, and fought friction, you have an idea of what your colleagues are going to be dealing with. Use your experience to pave a smoother path for them. Go through the codebase and do minor preparations: reshuffle code, fill gaps, rename things for clarity, resolve ambiguities, create new dir structures. Keep your changes minor. Don’t refactor everything yourself. It’s critical that the bulk of the work is shared. Submit a merge request.
5.  **Name your refactor.**
    Give your refactor a convenient name for use in discussions and docs. Make sure the name is concise, clear, and descriptive. For example, “Remove dependency on [package X]”.
6.  **Write up refactoring instructions.**
    Create a document in your internal knowledge base and title it with the name from step 5\. Some tips:
    *   State exactly what to do, and how to do it. Be brief and specific.
        *   *Do NOT tell stories*: “Over the years we’ve realized that the method we’ve been using …”
        *   *Do list specific steps*: “Find a class that has function X. Create new class named Y. Move function X into class Y.” The steps can’t all be plain of course, but challenge yourself to see how brief and specific you can make them.
    *   Link your example merge request from step 3\. People should see the code before and after.
    *   Finally, feel free to add some context at the end. Here, you’re welcome to provide the background, tell stories, link to relevant resources and discussions. That said, do hide this part under an expandable element, like [<details>](https://developer.mozilla.org/en-US/docs/Web/HTML/Element/details). We want the doc focused on the pattern itself, with an option to expand the context as needed.
7.  **Add this refactor to the list of long term refactors.**
    Make sure that there is a page in your knowledge base that lists all long term refactors. The document from step 6 should be added to that list.
8.  **Introduce this refactor to your team.**
    Use either a written announcement or a meeting. Explain how to pick a chunk that needs refactoring, walk them through your example. Don’t forget to link the instructions you wrote in step 6\. It’s very important that your engineering team is aware of every long term refactor currently in progress. That’s why you want to stick to just a few at a time, and properly introduce each one. Adding a new long term refactor should be a big deal.
9.  **Assign refactoring tasks.**
    Your refactor is now ready to be done gradually over time, but I advise against creating all the tasks up front in the task tracker. That would destroy one of the main benefits — not wasting time on unimportant or soon-to-be-deleted code. Instead, create tasks as they naturally come up in planning. “Hey, since you’re going to be changing that, maybe remove dependency on package X while you’re in there?”. Moreover, I advise keeping the whole umbrella-refactor away from the task tracker, or at least from the areas where business can see it. A successful long term refactor should be tracked by engineers, not the company management. As long as it’s written in the knowledge base, and is always present on engineers’ minds, you should be good. It shouldn’t matter to the business whether the refactor is completed, and how long it takes.
10.  **Stay aware of long term refactors.**
    Get every new engineer joining the team to read the list from step 7\. Make sure you have this step in your onboarding process. This is also a great source of first tasks for them, to help them understand both the existing code, and the new direction. It’s easy to refer back to the list anytime (remember, the list must remain short), but engineers also tend to remind each other of these refactors when planning.
11.  **Complete the refactor.**
    A long term refactor doesn’t need to be 100% completed. Instead, one day you will find that your doc is redundant, because the codebase already speaks for itself. If all the major parts are refactored, and there is no more confusion about your direction, feel free to mark it done. This creates space for the next one.

Having followed this process carefully, I’ve seen something awesome happen. The team got into the habit of self-assigning refactors as needed. When they had questions, they’d initiate discussions and meetings. This got everyone on the same page around decisions that might’ve been controversial if made alone. With each completed refactor task, we’d all gain new examples to draw from in upcoming tasks.

Compare that to working on your own for weeks or months, and blindsiding your team with a huge diff.

## Drawbacks

Here are some that I can think of.

*   Albeit rare, some big refactors don’t have a common pattern. It’s possible that you’re actually dealing with multiple refactors that shouldn’t be under the same umbrella. Try to split them instead.
*   You need patience to get through these refactors. They can span a year, two years, who knows. During that time, the old and the new code will coexist, and might cause some confusion if the list from step 7 is not on everyone’s mind. I personally haven’t encountered this drawback in practice, because the process constantly keeps everyone on the same page. Due to organization and communication, nobody is confused about where we’re coming from, and where we are headed.
*   Certain parts of the code may never get refactored. There’s probably a good reason why. It could be that this part is easy to maintain as is, and doesn’t need to change. Or perhaps this code is on its way out. Think of it as a win — you saved time and didn’t introduce bugs unnecessarily.
*   If you like doing everything alone, this ain’t it. This approach is designed to get everyone on the same page. You will have to agree on solutions and articulate your reasoning. If you don’t like doing that, you won’t like long term refactors.

Try it, let me know how it goes!