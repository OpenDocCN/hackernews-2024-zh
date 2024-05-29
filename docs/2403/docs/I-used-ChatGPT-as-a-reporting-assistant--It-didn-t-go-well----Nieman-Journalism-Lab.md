<!--yml
category: 未分类
date: 2024-05-29 12:31:20
-->

# I used ChatGPT as a reporting assistant. It didn’t go well. | Nieman Journalism Lab

> 来源：[https://www.niemanlab.org/2024/03/i-used-chatgpt-as-a-reporting-assistant-it-didnt-go-well/](https://www.niemanlab.org/2024/03/i-used-chatgpt-as-a-reporting-assistant-it-didnt-go-well/)

When it comes to developments in artificial intelligence, things are moving fast. It’s been less than two years since the public release of AI tools like ChatGPT, Midjourney, Stable Diffusion and Meta’s LLaMA. Regulators, lawmakers, and businesses are all beginning to wrap their heads around the implications of the use of generative AI tools.

This includes

[news organizations](https://www.cjr.org/tow_center_reports/artificial-intelligence-in-the-news.php)

and journalists, who have already started

[experimenting](https://www.thecity.nyc/2024/02/29/chatgpt-map-stories-nyc/)

.

[Nieman Lab reported](https://www.niemanlab.org/2024/03/five-of-this-years-pulitzer-finalists-are-ai-powered/)

that 5 of the 45 unannounced finalists for this year’s Pulitzer prizes used AI for “researching, reporting, or telling their submissions.” At Investigative Reporters & Editors’ annual

[NICAR](https://www.ire.org/training/conferences/nicar-2024/)

data journalism conference in Baltimore last week, 14 of the

[200-plus sessions](https://schedules.ire.org/nicar-2024/)

were related to AI, discussing how the technology can help journalists with workflows, summarize dense documents, and debug their code.

I had one of my own, a session titled “[Using AI Tools for Data Journalism](https://bit.ly/nicar24-ai-tools)” in which I started out by reviewing the many tools available and highlighted [the](https://arxiv.org/abs/2403.00742) [many](https://www.theatlantic.com/technology/archive/2024/03/ai-water-climate-microsoft/677602/) [ethical](https://www.bloomberg.com/graphics/2024-openai-gpt-hiring-racial-discrimination/) [concerns](https://themarkup.org/hello-world/2023/11/11/meet-nightshade-a-tool-empowering-artists-to-fight-back-against-ai) about them.

Then I showed the results of my time-consuming experiment to use ChatGPT as an assistant for reporting on a story. Spoiler alert: It didn’t go so well!

The example story I used for this exercise was the [train derailment](https://www.ntsb.gov/investigations/Pages/RRD23MR005.aspx) in East Palestine, Ohio in February 2023, a major story that involved various kinds of data that I could ask ChatGPT to help analyze. To be clear, this wasn’t a story I had reported on, but I wanted to try using ChatGPT in a way that a data journalist might when covering it. I spent a *lot* of time chatting with ChatGPT as part of this exercise and, frankly, sometimes it was exhausting. You can read one of my chat sessions [here](https://chat.openai.com/share/c343795f-210f-4052-a506-7c662c6c78e5). The confidence that ChatGPT exudes when providing poorly sourced information (like Wikipedia) or imprecise locations can be misleading. At times I was able to get the chat agent to give me what I wanted, but I had to be very specific and I often had to scold it.

For example, when ChatGPT fulfilled my request to “generate a simple map centered on the location of the crash,” I immediately noticed that the pin on the map was far from any train tracks. When I asked where it got the coordinates for the crash location, it replied that these were “inferred based on general knowledge of the event’s location.” When I pressed it for a more specific citation, it could not provide one, and kept repeating that it was relying on “general knowledge.” I had to remind the tool that I had told it at the start of the chat that “it is crucial that you cite your sources, and always use the most authoritative sources.” Before it was finally able to mark the correct location, I had to remind it that it could obtain location coordinates from an authoritative document I had uploaded earlier in the chat, a PDF of the Federal Railroad Administration [incident report](https://safetydata.fra.dot.gov/Officeofsafety/Publicsite/FORM54/F54Report.aspx?RepType=SQL&txtf54key=NS15220720230203).

Extracting information and summarizing long documents is often cited as among the biggest strengths of tools like ChatGPT. My results were mixed. After some back and forth, I coaxed the agent into extracting the details and quantities of the hazardous chemicals that were released in the accident and format the information in a table listing the chemical name, the quantity released, what it is typically used for and its effects on human health. But it took a few tries. Where it did save time was in explaining specialized information that might have otherwise taken a time-consuming Google search to figure out—such as decoding railroad car numbers.

At times, the tool was too eager to please, so I asked it to tone it down a little: “You can skip the chit chat and pleasantries.” Users can instruct the bot to change the tone or style of their responses, but telling it that it is a lawyer [doesn’t make it more accurate](https://themarkup.org/hello-world/2024/01/06/what-happens-when-you-roleplay-with-chatgpt).

Overall, the sessions were a lot of work trying to figure out where the agent got its information, and redirecting it with precise instructions. It took a long time.

The company that makes ChatGPT, OpenAI, did not respond to a request for comment.

Based on my interactions, by far the most useful capabilities of ChatGPT are its ability to generate and debug programming code. (At one point during the East Palestine exercise, it generated some simple [Python](https://en.wikipedia.org/wiki/Python_(programming_language)) code for creating a map of the derailment.) When responding to a request to write code, it typically explains its approach (even though it may not be the best one), and shows its work, and you can redirect it to follow a different approach if you think its plan isn’t what you need. The ability to continually add to the features of your code while the AI agent retains the context and history of what you have been discussing can really save you a ton of time, avoiding painstaking searches for posts about a similar problem on StackOverflow (one of the largest online coding communities).

The NICAR exercise left me with concerns about using generative AI tools for the precise work of data journalism. The fact that a tool as powerful as ChatGPT can’t produce a “receipt” of exactly how it knows something goes against everything we are trained to do as journalists. Also I worry about small, understaffed newsrooms relying upon these tools too much as the news industry struggles with layoffs and closures. And when there is a lack of guidance from newsroom leadership regarding the use of these tools, it could lead to errors and inaccuracies.

Thankfully, many newsrooms have started to address some of these concerns by drafting AI policies to help their journalists and their readers understand how they plan on using AI in their work.

[The Markup](https://themarkup.org/) has followed the lead of [other](https://www.wired.com/about/generative-ai-policy/) [news](https://www.thomsonreuters.com/en/artificial-intelligence/ai-principles.html) [organizations](https://www.theguardian.com/help/insideguardian/2023/jun/16/the-guardians-approach-to-generative-ai), and last week we updated [our ethics policy](https://themarkup.org/ethics#ai-ethics) with a section detailing our rules for any use of AI in our work. In summary, it says:

*   We will not publish stories or artwork created by AI (unless it is part of a story about AI)
*   We will always label or disclose its use
*   We will always rigorously check our work, and this certainly applies to anything generated by AI
*   Going forward we will evaluate the security, privacy and ethical considerations of any new AI tools

Thanks for reading, and always double check everything that a chatbot tells you!