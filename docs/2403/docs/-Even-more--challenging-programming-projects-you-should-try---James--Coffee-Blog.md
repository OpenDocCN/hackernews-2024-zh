<!--yml
category: 未分类
date: 2024-05-27 14:29:54
-->

# (Even more) challenging programming projects you should try | James' Coffee Blog

> 来源：[https://jamesg.blog/2024/02/28/programming-projects/](https://jamesg.blog/2024/02/28/programming-projects/)

My knowledge of programming has been largely self-directed. When I get excited about an idea, I research what I need to do to solve that problem. For example, when I was interested in how search engines work, I learned about the computational efficiency of sets. I discovered the problem "how can I check if I have crawled this URL?" when I may have crawled thousands of URLs. To answer this question faster, I used a set, which has O(1) lookup time, instead of O(n) lookup time.

Learning what you need to solve a problem is exciting, but following your own path in coding also leaves you with gaps in your knowledge. My conclusion here is that if you keep challenging yourself, you will fill in the gaps as you go (Even if it takes longer than it would have if you pursured a course. Fun is an important motivator for making progress; pursue what excites you.)

The moment I really started to understand computational efficiency -- and build an appreciation for making my programs faster -- was tackling that one problem for that one search engine. Since then, I sometimes ask myself: what can I do next? What is the next challenge? This is highly specific to where you are; some ideas will make sense, others will seem out of reach. Thus is learning.

In the spirit of Austin Henley's "[challenging projects every programmer should try](https://austinhenley.com/blog/morechallengingprojects.html)" series, I wanted to offer my own list of projects that have sustained my joy of programming. Herein begins the list.

## Build a search crawler

A crawler is a bot that navigates web pages and saves their contents. Search engines use crawlers to explore the web. The contents of web pages are "indexed". This means that pages are saved somewhere that they can be looked up.

I decided to build a search engine for a small community I was a part of. There were ~1000 sites that were known to the community, as established by people who contributed to our shared wiki. I used that as the list of sites to crawl. You can make your own list of sites and create your own search engine. Maybe you make a search engine for your favourite anime blogs. Or Taylor Swift fan sites. Whatever you want, so long as it's on the web.

My biggest learning through this process was that the web is a wild west. You can never trust that someone's web page is going to be exactly how you want it to be. Buidling a search crawler is an exercise in learning how to retrieve as many web pages as possible, reliably. without taking someone's site down.

In building a search crawler, you will learn about:

1.  How to download a web page
2.  Standards for content crawling (robots.txt, adult meta tags)
3.  Rate limiting
4.  Exponential back-offs
5.  Knowing when to crawl and recrawl sites
6.  Content negotiation
7.  Etags
8.  And more

Indeed, the web is a wild west. But with that wild west comes many exciting technical challenges to solve. This challenge occupied my mind for months. While my search engine is no longer active, this project was what gave me confidence that I can do more than write small Python scripts.

## Build an autocomplete system

Imagine you have a blog post. How could you auto-complete a word based on a sequence of letters? Take this blog post. If I started a word with "exponent", how could you, efficiently, recommend "exponential" as the last word? Herein lies a challenge. I am not going to talk too much about the how behind this project, but there is a lot of fun to be had!

When you discover how to auto-complete words, there is another challenge: how to recommend *which* word to auto-complete. If I type "expone", how should an autocomplete know whether to recommend "exponent" or "exponential" (both used in this guide).

## Write a file compressor

There are many existing tools that compress files, but have you ever asked: how do they make files smaller?

Here's a challenge: download this web page as a HTML file. Write a program that creates a compressed version of the HTML file. Your program should then be able to reproduce the exact file. Spaces, captalisation: everything should be the same.

Here are some relevant topics that may guide your reading on this subject (although I recommend trying to write a compressor before reading too much about it if that excites you!):

*   Information theory (the parent field of compression)
*   Dictionary coding (a way of representing information)
*   Huffman coding (a commonly used compression algorithm)
*   Entropy (a measure of the amount of "information" in a file; "information" is intentionally in quotation marks)

## Implement BitCask

[BitCask](https://riak.com/assets/bitcask-intro.pdf) is a key-value store algorithm. Key-value stores map keys (names) to values (pieces of information). For example, I could store information about posts on this blog like so:

```
{"title": "(Even more) challenging programming projects you should try", "published": "2024-02-28"} 
```

Where "title" and "published" are names, which are associated with values.

BitCask works entirely using your file system. You store keys and values in files. Every file is append-only. You can delete keys, but this works by overriding an existing key rather than deleting the value explicitly. [BitCask is documented in a short, useful paper](https://riak.com/assets/bitcask-intro.pdf) that guided me when I was making it. Challenge yourself to implement:

1.  Creating a "cask" (a key-value store that follows the BitCask algorithm)
2.  Add items to a cask
3.  Retrieve items from a cask
4.  Remove items from a cask
5.  Close a cask

When you close a cask, a merge operation is performed to merge all cask files into one.

## Write a programming language

*Wow... slow down. A programming language? Only serious experts can make that!* That's what I thought, too. It turns out it isn't true! You, too, can make a programming language. And you don't even need to read hundreds of pages of literature on theory before you get started (although the more you read, the more information you'll have to design your language!).

Designing a programming language lets you decide how you want to write code. You can follow existing patterns or make up your own. You get to set the rules.

There are many types of programming language, but a good place to start is writing a language that doesn't need a compiler. I am biased because that's how I learned, but the thought of writing a compiler before I had a bit more experience with programming langauge theory felt daunting. Thus, I wrote an "interpreted" language. A file is read, parsed, then executed; no compiling.

An interpreted language has a few key pieces:

1.  A "grammar", which outlines the structure of a language;
2.  A "lexer" / "parser", which takes in arbitrary text (i.e. a script written in your language!) and turns it into an Abstract Syntax Tree (AST) using your grammar, and;
3.  The evaluation engine, which reads the syntax tree and does something with it.

All of the above three components are separate technical challenges. I recommend starting with writing a grammar and a parser. Your grammar will state how your language works (i.e. how to declare variables, which symbols are allowed and in which order, how things can nest, or not nest). You can use an existing lexer to interpret your grammer. I used Lark in Python. Your evaluation engine will take your syntax tree and enforce the logic of your program (i.e. maintain variables, perform mathematical computations, work with strings and booleans; whatever you want your language to do).

*P.S. Writing a Lisp language is a great place to start, too!*

## The list, and: What other ideas should be here?

The list above is not a checklist, rather inspiration for projects you can take on. Perhaps one of the items above will inspire you to make something. Perhaps none of them resonate. That is okay, too. If that's the case, I recommend saving this post in the back of your mind; there may come a day when one of the ideas above delights you. James five years ago would probably have found the list above out of reach. James one year ago would have yearned for this list.

I recommend [Austin Henley's writing](https://austinhenley.com/blog/morechallengingprojects.html) for more wonderful ideas. His post is the reason I wrote this one.

If you do implement anything as a result of this post, let me know! Send me an email at readers [at] jamesg [dot] blog. In addition, if you need any guidance with the ideas above, email me. I would love to help. I have completed all the challenges I documented above; they continue to cultivate my child-like excitement for coding. I hope they can do the same for you, too.

Now: what other ideas should be on this list? What have you made that you think would be a great challenge for other programmers? Share it with others! An idea for a project that pushed you to learn more about programming five years ago may be the thing that helps change someone's perception of programming like search engines did for me.

[Share this post on Hacker News](https://news.ycombinator.com/submitlink?u=https://jamesg.blog//2024/02/28/programming-projects/&t=(Even%20more)%20challenging%20programming%20projects%20you%20should%20try).

[Share this post on Lobste.rs](https://lobste.rs/stories/new?url=https://jamesg.blog//2024/02/28/programming-projects/&title=(Even%20more)%20challenging%20programming%20projects%20you%20should%20try).