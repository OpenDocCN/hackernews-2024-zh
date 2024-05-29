<!--yml
category: 未分类
date: 2024-05-27 14:47:38
-->

# hn-index

> 来源：[https://www.alexmolas.com/2024/02/12/hn-index.html](https://www.alexmolas.com/2024/02/12/hn-index.html)

<main>

# hn-index

 *February 12, 2024 · 5 mins · 1066 words

* * *

**TLDR** - I implemented an h-index calculator using HackerNews scores - [alexmolas.com/hn-index](https://www.alexmolas.com/hn-index).

* * *

I’m always looking for new blogs to follow, and a good way to find good quality blogs is HackerNews. I already have a curated list of blogs, but I’m sure I’m missing a lot of them. One of my problems is that a priori it’s difficult to know which profiles are worth following and which are not. To solve this problem I decided to use h-index, a metric used in science to measure the impact of publications. With this metric, I can score HN users, and depending on the score decide whether to follow them or not. As quick reminder, according to [Wikipedia](https://en.wikipedia.org/wiki/H-index), the h-index is defined as

> The h-index is defined as the maximum value of h such that the given author/journal has published at least h papers that have each been cited at least h times.

Fortunately, HackerNews exposes an open API to fetch the data of users and items, and with this information is more or less easy to compute this index. The API is documented in their GitHub ([HN API docs](https://github.com/HackerNews/API)).

In this post, I’ll explain how I’ve implemented this solution, and show how can you use it to compute your index.

# User data

User data can be fetched from `https://hacker-news.firebaseio.com/v0/user/USER_NAME.json?print=pretty`, and you’ll get something like

```
{  "about"  :  "This is a test",  "created"  :  1173923446,  "delay"  :  0,  "id"  :  "jl",  "karma"  :  2937,  "submitted"  :  [  8265435,  8168423,  8090946,  8090326,  7699907  ]  } 
```

# Item data

User data can be fetched from `https://hacker-news.firebaseio.com/v0/item/ITEM_ID.json?print=pretty`, and you’ll get something like

```
{  "by"  :  "dhouston",  "descendants"  :  71,  "id"  :  8863,  "kids"  :  [  8952,  9224,  8917,  8884,  8887  ],  "score"  :  111,  "time"  :  1175714200,  "title"  :  "My YC app: Dropbox - Throw away your USB drive",  "type"  :  "story",  "url"  :  "http://www.getdropbox.com/u/2/screencast.html"  } 
```

One of the problems I’ve faced here is that all the items are exposed in the same endpoint. This means that you can’t query specifically for stories. So you don’t know a priori if an item is a comment or a story. We’ll see later that this slows down quite a bit the computations.

# Putting everything together

With these two endpoints, we have all the information we need to compute the h-index of a user. To do so we only need to get all the submissions a user has made, and then for each submission get the score it has. As I said before, we can’t know a priori which submissions of a user are stories and which comments, so we need to fetch first the data and then filter by type, which makes the process slower.

I implemented the code to compute it in a [python script](https://github.com/alexmolas/alexmolas.github.io/blob/master/hn-index/script.py). The code is quite fast, and it takes around 20 seconds to compute the h-index for [pg](https://news.ycombinator.com/user?id=pg) which has more than 15k submissions, of which around 700 are stories. For my user, it takes around 1 second.

## Making it accessible

Then, I thought that if I wanted to make it accessible I should implement it directly in my site. And with some help from a well-known LLM I managed to do so. The functionality is now available at [HN-Index](https://www.alexmolas.com/hn-index/).

For some reasons unknown to me, the web version is much slower than the Python one, and I haven’t managed to make it faster yet. So if you know how to improve it please write me and I’ll change it. The JavaScript code to fetch and compute the index is [here](https://github.com/alexmolas/alexmolas.github.io/blob/master/hn-index/app.js). I’m not a JavaScript expert, don’t be too harsh judging the code.

# Some insights

Finally, let’s see who are the best HN users in terms of h-index. Since computing the index for all the users was unfeasible I decided to check it only for the [leaders](https://news.ycombinator.com/leaders). In the following table, we have the user with the highest h-index.

```
| username        |   number of submissions |   h index |   karma |
|:----------------|------------------------:|----------:|--------:|
| ingve           |                   14263 |       283 |  190555 |
| tosh            |                   18329 |       255 |  142664 |
| todsacerdoti    |                   10798 |       252 |  134184 |
| pseudolus       |                   15798 |       238 |  152021 |
| lxm             |                    9622 |       237 |  137389 |
| danso           |                    4871 |       235 |  162226 |
| Tomte           |                   22221 |       232 |  142563 |
| zdw             |                    4125 |       214 |  110034 |
| thunderbong     |                    7380 |       208 |   75116 |
| luu             |                    5309 |       200 |  103885 | 
```

On the other hand, it’s also interesting that some of the users in the leaderboard have a very low h-index, which means that almost all their karma comes from comments.

```
| username        |   number of submissions |   h index |   karma |
|:----------------|------------------------:|----------:|--------:|
| saagarjha       |                       0 |         0 |   53435 |
| Waterluvian     |                       7 |         3 |   44106 |
| Someone1234     |                       3 |         3 |   47266 |
| dragonwriter    |                      12 |         5 |  115744 |
| tyingq          |                      16 |         5 |   58391 |
| derefr          |                      11 |         6 |   52205 |
| Retric          |                      23 |         6 |   53454 |
| hn_throwaway_99 |                      15 |         8 |   62696 |
| paxys           |                      18 |         9 |   58494 |
| WalterBright    |                      22 |         9 |   68943 | 
```* </main>