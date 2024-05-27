<!--yml
category: 未分类
date: 2024-05-27 12:57:26
-->

# Tool use (function calling) - Anthropic

> 来源：[https://docs.anthropic.com/claude/docs/tool-use](https://docs.anthropic.com/claude/docs/tool-use)

Claude is capable of interacting with external client-side tools and functions, allowing you to equip Claude with your own custom tools to perform a wider variety of tasks.

**Tool use public beta**

We’re excited to announce that tool use is now in public beta! To access this feature, you’ll need to include the `anthropic-beta: tools-2024-05-16` header in your API requests.

We’ll be iterating on this open beta over the coming weeks, so we appreciate all your feedback. Please share your ideas and suggestions using this [form](https://forms.gle/BFnYc6iCkWoRzFgk7).

Here’s an example of how to provide tools to Claude using the Messages API: