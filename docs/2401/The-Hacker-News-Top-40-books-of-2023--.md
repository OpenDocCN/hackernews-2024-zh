<!--yml
category: 未分类
date: 2024-05-27 14:51:05
-->

# The Hacker News Top 40 books of 2023 -

> 来源：[https://hnreads.com/post/top40_2023/](https://hnreads.com/post/top40_2023/)

Hi, welcome to the brand new website, HN Reads. I enjoy reading [Hacker News](https://news.ycombinator.com/) and I love buying books (and reading), and I also love data, so what better than doing some processing of data about books to find some interesting results?! It also gives me the opportunity to write about books that I find interesting.

Here are the top 40 books recommended by HN readers in 2023.

How I generated this list:

*   get stories that contain the word “book”, but not “macbook”, “chromebook”, etc. That were posted in 2023, and also try to only get “ask HN” stories, by excluding stories that have a url in them.
*   get the comments for the stories
*   for each comment, use the OpenAI gpt3.5 chat API to extract a list of book details in json format:

```
 [
 	 	{"match":"xxx","title":"xxx", "author":"xxx", "link":"xxx"},
 	 	...
 	 ] 
```

*   standardise titles, e.g. “Gödel, Escher, Bach: An Eternal Golden Braid” is converted to “Gödel, Escher, Bach”. Any leading “the” is also removed. I end up with a title column and an orig_title column in my data. If there are that have the same title, but different subtitles, for example “Javascript: the good parts” and “Javascript: the Definitive Guide”, this isn’t too much of a problem, as the authors are different.
*   aggregate the resulting information. There was some manual adjustment to standardise the book titles and authors names, for example Gödel was sometimes spelled “Godel” and Douglas Hofstadter was sometimes returned as “Douglas R. Hofstadter”.

I also shared some details about this process at [my blog](https://blog.reyem.dev/post/extracting_hn_book_recommendations_with_chatgpt_vol_ii/).

It’s nice to see one of my favourite series appearing on this list, the Hyperion Cantos. I don’t usually go for books with horror elements, but the story is so good, and the science fiction elements are cool too.

A couple of controversial figures don’t rank in the top 40 of 2023: Elon Musk’s new biography wasn’t recommended once in an “Ask HN” about books, although there were plenty of stories on Hacker News about him and Isaacson’s book. Also Ayn Rand’s books weren’t mentioned often enough in 2023 to make the list.

Without further delay, here’s the list:

*See footnote for info on affiliate links*.

| # | Title | Author | Count | First Mention |
| --- | --- | --- | --- | --- |
| 1 | [Structure and Interpretation of Computer Programs](https://www.amazon.com/dp/0262510871?tag=reyemdev0f-20) | Harold Abelson, Gerald Jay Sussman, Julie Sussman | 26 | [34231811](https://news.ycombinator.com/item?id=34231811) |
| 2 | [Designing Data-Intensive Applications](https://www.amazon.com/dp/1449373321?tag=reyemdev0f-20) | Martin Kleppmann | 18 | [34231811](https://news.ycombinator.com/item?id=34231811) |
| 3 | [Gödel, Escher, Bach](https://www.amazon.com/dp/0465026567?tag=reyemdev0f-20) | Douglas Hofstadter | 18 | [34440923](https://news.ycombinator.com/item?id=34440923) |
| 4 | [The C Programming Language](https://www.amazon.com/dp/0131103628?tag=reyemdev0f-20) | Brian W. Kernighan, Dennis M. Ritchie | 17 | [35933845](https://news.ycombinator.com/item?id=35933845) |
| 5 | [How to Win Friends and Influence People](https://www.amazon.com/dp/0671027034?tag=reyemdev0f-20) | Dale Carnegie | 15 | [34310435](https://news.ycombinator.com/item?id=34310435) |
| 6 | [The Mythical Man-Month](https://www.amazon.com/dp/B00B8USS14?tag=reyemdev0f-20) | Frederick P. Brooks Jr. | 14 | [34333293](https://news.ycombinator.com/item?id=34333293) |
| 7 | [How to Solve It](https://www.amazon.com/dp/069111966X?tag=reyemdev0f-20) | George Pólya | 14 | [34441085](https://news.ycombinator.com/item?id=34441085) |
| 8 | [Principles of Mathematical Analysis](https://www.amazon.com/dp/0070856133?tag=reyemdev0f-20) | Walter Rudin | 13 | [34311378](https://news.ycombinator.com/item?id=34311378) |
| 9 | [The Elements of Computing Systems](https://www.amazon.com/dp/0262640686?tag=reyemdev0f-20) | Noam Nisan, Shimon Schocken | 13 | [34412400](https://news.ycombinator.com/item?id=34412400) |
| 10 | [Calculus](https://www.amazon.com/dp/0914098918?tag=reyemdev0f-20) | Michael Spivak | 13 | [34441489](https://news.ycombinator.com/item?id=34441489) |
| 11 | [Code: The Hidden Language of Computer Hardware and Software](https://www.amazon.com/dp/0735611319?tag=reyemdev0f-20) | Charles Petzold | 12 | [34476530](https://news.ycombinator.com/item?id=34476530) |
| 12 | [Crafting Interpreters](https://www.amazon.com/dp/0990582930?tag=reyemdev0f-20) | Robert Nystrom | 11 | [34231811](https://news.ycombinator.com/item?id=34231811) |
| 13 | [Peopleware](https://www.amazon.com/dp/0321934113?tag=reyemdev0f-20) | Tom DeMarco, Timothy Lister | 11 | [35930473](https://news.ycombinator.com/item?id=35930473) |
| 14 | [Eloquent JavaScript](https://www.amazon.com/dp/1593279507?tag=reyemdev0f-20) | Marijn Haverbeke | 10 | [34412355](https://news.ycombinator.com/item?id=34412355) |
| 15 | [Elements](https://www.amazon.com/dp/1888009187?tag=reyemdev0f-20) | Euclid | 10 | [34440894](https://news.ycombinator.com/item?id=34440894) |
| 16 | [The Art of Computer Programming](https://www.amazon.com/dp/020103803X?tag=reyemdev0f-20) | Donald E. Knuth | 10 | [34442163](https://news.ycombinator.com/item?id=34442163) |
| 17 | [Blindsight](https://www.amazon.com/dp/0765319640?tag=reyemdev0f-20) | Peter Watts | 10 | [35054030](https://news.ycombinator.com/item?id=35054030) |
| 18 | [The E-Myth Revisited](https://www.amazon.com/dp/0887307280?tag=reyemdev0f-20) | Michael E. Gerber | 10 | [35169069](https://news.ycombinator.com/item?id=35169069) |
| 19 | [The Pragmatic Programmer](https://www.amazon.com/dp/0135957052?tag=reyemdev0f-20) | Andrew Hunt, David Thomas | 10 | [35966675](https://news.ycombinator.com/item?id=35966675) |
| 20 | [Compilers: Principles, Techniques, and Tools](https://www.amazon.com/dp/8131721019?tag=reyemdev0f-20) | Alfred V. Aho, Monica S. Lam, Ravi Sethi, Jeffrey D. Ullman | 10 | [36146877](https://news.ycombinator.com/item?id=36146877) |
| 21 | [High Output Management](https://www.amazon.com/dp/0679762884?tag=reyemdev0f-20) | Andrew S. Grove | 9 | [34262064](https://news.ycombinator.com/item?id=34262064) |
| 22 | [Concrete Mathematics](https://www.amazon.com/dp/B00NPN0U86?tag=reyemdev0f-20) | Ronald L. Graham, Donald E. Knuth, Oren Patashnik | 9 | [34441399](https://news.ycombinator.com/item?id=34441399) |
| 23 | [Never Split the Difference](https://www.amazon.com/dp/0062407805?tag=reyemdev0f-20) | Chris Voss | 9 | [35171863](https://news.ycombinator.com/item?id=35171863) |
| 24 | [The Phoenix Project](https://www.amazon.com/dp/1942788290?tag=reyemdev0f-20) | Gene Kim, Kevin Behr, George Spafford | 8 | [34312721](https://news.ycombinator.com/item?id=34312721) |
| 25 | [The Art of Electronics](https://www.amazon.com/dp/0521809266?tag=reyemdev0f-20) | Paul Horowitz, Winfield Hill | 8 | [34475069](https://news.ycombinator.com/item?id=34475069) |
| 26 | [Dune](https://www.amazon.com/dp/0441013597?tag=reyemdev0f-20) | Frank Herbert | 8 | [35053380](https://news.ycombinator.com/item?id=35053380) |
| 27 | [Hyperion](https://www.amazon.com/dp/0553283685?tag=reyemdev0f-20) | Dan Simmons | 8 | [35054030](https://news.ycombinator.com/item?id=35054030) |
| 28 | [The Hitchhiker’s Guide to the Galaxy](https://www.amazon.com/Complete-Hitchhikers-Guide-Galaxy-Boxset/dp/1529044197?tag=reyemdev0f-20) | Douglas Adams | 8 | [35399641](https://news.ycombinator.com/item?id=35399641) |
| 29 | [Operating Systems: Three Easy Pieces](https://www.amazon.com/dp/198508659X?tag=reyemdev0f-20) | Remzi H. Arpaci-Dusseau, Andrea C. Arpaci-Dusseau | 7 | [34231811](https://news.ycombinator.com/item?id=34231811) |
| 30 | [Automate the Boring Stuff with Python](https://www.amazon.com/dp/1593275994?tag=reyemdev0f-20) | Al Sweigart | 7 | [34412667](https://news.ycombinator.com/item?id=34412667) |
| 31 | [The Design of Everyday Things](https://www.amazon.com/dp/0465050654?tag=reyemdev0f-20) | Don Norman | 7 | [34439374](https://news.ycombinator.com/item?id=34439374) |
| 32 | [Calculus Made Easy](https://www.amazon.com/dp/B09MYWWBDC?tag=reyemdev0f-20) | Silvanus P. Thompson | 7 | [34440860](https://news.ycombinator.com/item?id=34440860) |
| 33 | [Linear Algebra Done Right](https://www.amazon.com/dp/3031410254?tag=reyemdev0f-20) | Sheldon Axler | 7 | [34440972](https://news.ycombinator.com/item?id=34440972) |
| 34 | [The Three-Body Problem](https://www.amazon.com/dp/0765382032?tag=reyemdev0f-20) | Cixin Liu | 7 | [34606380](https://news.ycombinator.com/item?id=34606380) |
| 35 | [Children of Time](https://www.amazon.com/dp/B06ZXTHNSJ?tag=reyemdev0f-20) | Adrian Tchaikovsky | 7 | [34611775](https://news.ycombinator.com/item?id=34611775) |
| 36 | [The Five Dysfunctions of a Team](https://www.amazon.com/dp/8126522747?tag=reyemdev0f-20) | Patrick Lencioni | 7 | [34627776](https://news.ycombinator.com/item?id=34627776) |
| 37 | [Bobiverse](https://www.amazon.com/dp/1680680595?tag=reyemdev0f-20) | Dennis E. Taylor | 7 | [35053380](https://news.ycombinator.com/item?id=35053380) |
| 38 | [Founders at Work](https://www.amazon.com/dp/B092RC56HW?tag=reyemdev0f-20) | Jessica Livingston | 7 | [35170064](https://news.ycombinator.com/item?id=35170064) |
| 39 | [Neuromancer](https://www.amazon.com/Complete-Collection-William-Neuromancer-Overdrive/dp/1473233968?tag=reyemdev0f-20) | William Gibson | 7 | [35393643](https://news.ycombinator.com/item?id=35393643) |
| 40 | [Design Patterns](https://www.amazon.com/dp/0201633612?tag=reyemdev0f-20) | Erich Gamma, Richard Helm, Ralph Johnson, John Vlissides | 7 | [35930659](https://news.ycombinator.com/item?id=35930659) |

#### The Almost-Made-It list

Originally I wanted to publish a top 50, but there are too many ties. I chose to finish at 40 items, as that’s the cutoff between books that were mentioned 7 times and those that were mentioned 6 or fewer times. The following 29 books have been mentioned 5 or 6 times.

* * *