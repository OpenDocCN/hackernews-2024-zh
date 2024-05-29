<!--yml
category: 未分类
date: 2024-05-29 12:41:27
-->

# The other hard retrieval problems

> 来源：[https://softwaredoug.com/blog/2024/03/24/other-hard-retrieval](https://softwaredoug.com/blog/2024/03/24/other-hard-retrieval)

In my darkest “old man yells at cloud” moments, in todays AI world, I get a little sad.

I’m happy that as practitioners we now see every *thing* as living in a vector space. Indeed that’s exciting!

We can store these vectors, query them, and find things most similar. The “vector database” (or really –retrieval system–) will become the center for user interactivity. I’m very enthusiastic about this future.

What makes me a bit sad though - we think that the only vector dimensions that matter come from dense embeddings.

In reality, to get to make step changes in retrieval systems, diverse, orthogonal features matter.

Embeddings “blur” what we look at. We squint at our picture of a squirell and we see a vague N-legged furry thing. We recall in our minds all the other things we know: like dogs, cats, maybe lizards. Maybe tables? Stuffed animals? Amazing. Vector search helps us broaden out in a way that once seemed magical.

But we also need the un-squinted, explicit information from the photo too! The high precision, engineered, domain-specific features. Features that tell us indeed, we can explicitly label a thing called a “squirrel”. That if our users notion of similarity has a more scientific-bent - that we must strongly weigh explicit labels, explicitly mapped to a family of animals called “rodents”. These are explicit, categorical labels, mapped to a hierarchy or knowledge graph.

In search, when you type a query, users may actually want to focus precisely on the text for that query. Exactly on that thing. Not just the fuzzified related stuff to that query.

But what’s most important here is the notion of ORTHOGONALITY.

If I want to build a good ranking system, I need to ADD information to my vector. I don’t want to append features heavily correlated to existing features. Embedding models will correlate with other embedding models. Adding other, completely different, perspectives on the search problem are more likely to ADD information than just another embedding thing.

Because, ranking models are notoriously non-linear. Embedding matches matter only IF the information is clearly not spam. Text matches only matter WHEN the embedding similarity is close enough. All of which may or may not matter for our specific users. The legal researcher needs are quite different from the lay-person seeking legal help – even over the same corpus!

This means a diverse array of data structures - from lexical to embedding similarity. From dynamic changing numerical values like recency or dynamic pricing. To explicit classification of items as spam. To all sorts of things that matter in unique and different ways.

This is why I see teams adopting Vespa and OpenSearch more and more - they explicitly build for hybrid retrieval, making all the perspectives on the problem first class. This is why teams invest heavily in indexing and query preprocessing stages. They know they need to extract and update all kinds of information.

So get excited about where we are today, but don’t remain on your isolated retrieval island. Step changes in require new, orthogonal information, new data structures – to broaden your mind to every perspective of retrieval.