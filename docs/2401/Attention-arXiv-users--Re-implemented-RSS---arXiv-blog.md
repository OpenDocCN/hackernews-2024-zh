<!--yml
category: 未分类
date: 2024-05-27 15:22:49
-->

# Attention arXiv users: Re-implemented RSS – arXiv blog

> 来源：[https://blog.arxiv.org/2024/01/31/attention-arxiv-users-re-implemented-rss/](https://blog.arxiv.org/2024/01/31/attention-arxiv-users-re-implemented-rss/)

arXiv has re-implemented our RSS feed to improve results for users. This is a completely rewritten version of arXiv’s RSS feed that runs in the cloud. It should create a more accurate list of each day’s articles. For technical details see [RSS Feeds – arXiv info](https://info.arxiv.org/help/rss.html).

Changes from the old RSS feed:

*   new base URL of rss.arxiv.org, instead of just arxiv.org (arxiv.org/rss will redirect to rss.arxiv.org)
*   status page at /feed/status
*   new content updated at midnight (Eastern US time, as usual)
*   can request multiple categories with + hep-lat+math.CO
*   limit of 2000 items
*   categorization and order of new, cross, replace and replace-cross now matches listings
*   An author list is now provided for each paper
*   The RSS output complies with the RSS 2.0 specification. RSS 0.91, and 1.0 are no longer supported. Atom format is still supported — for example http://rss.arxiv.org/atom/math.