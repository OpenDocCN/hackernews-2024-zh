<!--yml
category: 未分类
date: 2024-05-29 12:35:03
-->

# HTML's readability, robustness, and intuitiveness | James' Coffee Blog

> 来源：[https://jamesg.blog/2024/03/22/html/](https://jamesg.blog/2024/03/22/html/)

This evening, I had a delightful conversation with [Tantek](https://tantek.com) and [Joe](https://artlung.com) about markup and programming languages. The crux of the discussion was complexity. We arrived at the topic of why HTML -- and more broadly the web -- won. Why HTML, at a time when the exact markup of the web was in its formative years, became the preferred standard.

Tantek pointed me to a quote from Sir. Tim Berners-Lee's "Weaving the Web" book, which according to [Tantek's review](https://tantek.com/log/2003/0813t1158.html) has many interesting insights about the web. One quote from the book that was relevant to our discussion was:

> But the human readability of HTML was an unexpected boon. To my surprise, people quickly became familiar with the tags and started writing their own HTML documents directly.

I find HTML readable. And it is explicit. After a bit of learning, I developed the mental mapping that `p` was a paragraph, `h1` was a top-level heading, etc. While my initial knowledge may have spanned only the foundational text elements, and less of the semantic layout elements available -- `main`, `section`, `header`, etc. -- I felt I could do a lot with a little knowledge. This feeling of empowerment feels significant: with a bit of studying, you can get a long way with HTML.

In comparison, markdown is a markup language that strives for readability, but has many syntactical gotcha-s that make authoring complex documents difficult. This is something one should keep in mind because many more complex documents start off simple. For instance, I wrote a poem and put it on my website, but I think my markdown parser messed up the line spacing. It looked okay in my text editor, but not when presented. I have never run into this issue with HTML (although I didn't want to write my poem in HTML because it is more verbose, I probably should have converted my markdown into HTML and verified the output!).

HTML is robust, too. If you miss a tag, or put a typo in a tag, or don't include a `DOCTYPE`, among many other things, HTML still works. It tries its best to render a page. This robustness is great for authorship, too. If something is broken, you can visualise the issue. There is a visual feedback loop. You can debug and see the results.

Joe pointed me to a relevant comment from [Adam Bosworth](https://artlung.com/blog/2020/02/19/adam-bosworths-talk-at-2005-mysql-users-conference/), who drove the team that worked on Internet Explorer 4.0 at Microsoft:

> The other issue was sloppy — HTML is really sloppy. In fact when we were building IE4 we had this very simple mantra. It doesn’t matter what you give us: we will render it. And every so often we sort of choke and we would stare at something we go “no no we will render this”– if table rows overlap table rows: we’d render if ‘s intersected with forms: we’d render. We didn’t care. Basically the general theory was it takes a licking and it keeps on ticking. And because we did that it meant that people could be very flexible and very sloppy about how they generated their content.

For a beginner to programming, a page that doesn't look quite right is more intuitive than a page with an error. Errors are intimidating. A page that is slightly off begets improvement, just as seeing a mark on a plate you just cleaned begets cleaning the plate a bit more to make it look just right. I am grateful for how gracefully HTML fails. Indeed, I cannot recall writing a HTML page that failed. To all the browser vendor makers who have worked on ensuring this property remains true, thank you!

We discussed how writing HTML is a joyful activity for some. There are in person communities like HTML Energy and online communities like the IndieWeb and 32 Bit Cafe where you can find many people who enjoy authoring HTML. Indeed, two weeks ago I helped someone who works in the film industry author their own website with HTML. They wanted to make their site with code, instead of using a site generator. The result, after a few hours of work, and occasional assistance from a community of people who love websites -- the IndieWeb -- was an amazing resume / CV web page that they wanted to continue improving. Seeing excitement over making a web page first-hand is wonderful.

Indeed, I feel excited when I write, and proud when I have written, a HTML document. The process is relaxing. I can think about what I want to represent -- lists, paragraphs, italicised text, and so on -- and explicitly state what I want and where I want it. Then, I can start thinking about CSS, but that is another kettle of fish entirely. At the end, I have a web page that shows my content that I can share with others. I really do love HTML. (Note to self: Look up the etymology of the expression "kettle of fish".)