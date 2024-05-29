<!--yml
category: 未分类
date: 2024-05-27 14:39:21
-->

# Optimizing Technical Docs for LLMs | kapa.ai docs

> 来源：[https://docs.kapa.ai/blog/optimizing-technical-documentation-for-llms](https://docs.kapa.ai/blog/optimizing-technical-documentation-for-llms)

We're seeing lots of **forward-thinking technical companies** like [OpenAI](https://docs.kapa.ai/examples#-openai), [CircleCI](https://docs.kapa.ai/examples#-circleci), [Temporal](https://docs.temporal.io/), [Mixpanel](https://docs.kapa.ai/examples#-mixpanel) and [Prisma](https://docs.kapa.ai/examples#-prisma) **adopt Large Language Models (LLMs) trained on their documentation to improve their developer experience**.

At [kapa.ai](https://www.kapa.ai) we have **worked with over 80 technical teams**, including those mentioned above, to implement these LLM-based systems for their developers. In the process, We've learned a lot about **how to structure documentation for LLMs** and **wanted to share some best practices** to consider for others considering this approach.

### 1\. Embrace Page Structure and Hierarchy[​](#1-embrace-page-structure-and-hierarchy "Direct link to 1\. Embrace Page Structure and Hierarchy")

LLMs excel at navigating structured content and rely on context hints to understand the broader picture. A **clear hierarchy of headings and subheadings on a page** helps LLMs understand the relationships between different sections of your documentation.

A great example of this is how **Temporal** structures its [documentation](https://docs.temporal.io/dev-guide/java/durable-execution#add-replay-test) for their SDKs. Take `Add a replay test` within the Java SDK, which is an important feature related to workflow execution. The hierarchy of the documentation is as follows:

```
- Development
 - Java SDK - Develop for durability - Add a replay test - ... 
```

This structure allows an LLM to more effectively navigate and understand the context when answering questions related to `replay tests` within the Java SDK. This is especially important as `replay tests` are also used in [other SDKs](https://docs.temporal.io/dev-guide/go/durable-execution#why-replay-test).

### 2\. Segment Documentation by Sub-products[​](#2-segment-documentation-by-sub-products "Direct link to 2\. Segment Documentation by Sub-products")

To avoid LLMs confusing similar offerings, such as cloud versus open-source versions, it's also helpful to ensure that good **documentation hierarchy extends to the product-level**. We've seen that **maintaining separate documentation for each sub-product** can significantly improve the LLM's understanding of the context and the user's intent.

A great example of this is how **Prisma** divides their [documentation](https://www.prisma.io/docs/) into **their three main offerings**:

*   `ORM`: A Node.js and TypeScript ORM (core product)
*   `Accelerate`: A Global database cache (newly released)
*   `Pulse`: Managed change data capture (early access)

Segmenting docs per product in some cases also **allows for deploying separate LLMs for each product**, which can be further optimized for the specific use case.

### 3\. Include Troubleshooting FAQs[​](#3-include-troubleshooting-faqs "Direct link to 3\. Include Troubleshooting FAQs")

Troubleshooting sections formatted as Q&A are an effective source for LLMs as they **mirror the questions users often ask**, making it easier for LLMs to understand and respond to similar questions.

**OpenAI's** documentation is a good example of this, particularly on their [capabilities pages](https://platform.openai.com/docs/guides/vision), where they have technical FAQs on the bottom of every page.

The format that works best for LLMs is a clear question followed by a concise answer. For instance, a **well-structured FAQ section** might look like this:

```
### [Common User Questions]   [Concise 1-2 Sentence Answer] 
```

When [looking at metrics](https://docs.kapa.ai/#how-does-kapa-work-) for how frequently specific sources are used in LLM responses, we've seen that **technical FAQs are often the most frequently used source**.

### 4\. Provide Self-contained Example Code Snippets[​](#4-provide-self-contained-example-code-snippets "Direct link to 4\. Provide Self-contained Example Code Snippets")

Including **small, self-standing code snippets** can be helpful, especially for products that rely on large and often complex SDKs or APIs.

**Mixpanel** for example uses code snippets effectively across their [documentation](https://docs.mixpanel.com/docs/tracking-methods/sdks/javascript#incrementing-numeric-properties), which contains lots of tracking and analytics implementation code. For example, to increment numeric properties, they provide the following code snippet to showcase the `mixpanel.people.increment` method:

Two other helpful tips for including code are to ensure that snippets (1) **have a brief description** above the code to clarify its purpose and usage, and (2) **comments within the code** to explain the logic and functionality. Both of these help LLMs further understand the context and purpose of the code snippet.

Although less related to the structure of your documentation, this guide would be incomplete without mentioning the importance of building a **community forum as a source for both developers and LLMs** to get help on **undocumented topics**.

For example, **CircleCI** has an [active and well maintained community forum](https://discuss.circleci.com/) where users can ask questions and get help from other users and CircleCI staff.

Similar to FAQs, a technical forum works well because it **mirrors the questions users often ask**. A forum also works well as an interim solution for questions not yet covered in your official docs.

Note that care should be taken when including forum content. Applying filters such as **only including responses marked** `resolved` or `accepted` can help ensure the relevancy of the content and **including links to the original forum thread** ensures authors are properly attributed.

### 6\. A Few More Practical Tips[​](#6-a-few-more-practical-tips "Direct link to 6\. A Few More Practical Tips")

In addition to above, here's a few tactical tips to solve common documentation-related issues we've seen with LLMs:

*   **Avoid storing docs in files:** Keep relevant content directly in your docs rather than in linked files such as PDFs, as LLMs have a harder time parsing these.
*   **Write text descriptions for images:** Ensure information conveyed through screenshots is also described in text, as LLMs parse text more efficiently.
*   **Provide OpenAPI specs for REST APIs:** Providing structured OpenAPI specifications makes it possible to leverage custom parsers, which can improve formatting for LLMs.
*   **Include example requests and responses:** Include these in your API descriptions to give LLMs concrete examples of how to use your APIs.
*   **Define specific acronyms and terms:** Clarify all acronyms and specialized terminology within your documentation to aid LLM comprehension.
*   **Include necessary imports in code examples:** This ensures code examples can run without additional context.

These tips can significantly improve LLMs' ability to understand and accurately respond to user queries.

* * *

By following these guidelines, you can significantly **enhance the usefulness of your technical documentation and sources for LLMs**, ultimately improving the developer experience.

If you're interested **in testing out an LLM on your technical sources then [sign up here for a quick demo](https://www.kapa.ai/early-access) on your content** or **[reach out to the kapa team](mailto:founders@kapa.ai)** if you have questions about how to **further optimize your technical documentation** for LLMs.