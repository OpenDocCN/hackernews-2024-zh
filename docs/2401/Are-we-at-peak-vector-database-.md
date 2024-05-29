<!--yml
category: 未分类
date: 2024-05-27 15:13:05
-->

# Are we at peak vector database?

> 来源：[https://softwaredoug.com/blog/2024/01/24/are-we-at-peak-vector-db](https://softwaredoug.com/blog/2024/01/24/are-we-at-peak-vector-db)

As both a search person and something of a veteran of the NoSQL days, I wonder to myself, often, “how can so many vector databases need to exist?”.

Even in our current AI age, where everyone and their mom is trying to build the next chat / AI powered whatever. Even today when it seems everyone is putting vectors somewhere to retrieve them…

I have to ask the tough question - when have we reached peak vector DB? Do we have enough choices for the specific task of storing and retrieving vectors?

Just on cursory listing, I can think of the following vector databases, libraries, whatever:

**Pure Vector DBs**

*   Pinecone
*   QDrant
*   Milvus / Zilliz
*   Weaviate
*   Turbopuffer
*   MyScale

**Open Source search engines**

*   Solr
*   Elasticsearch
*   OpenSearch
*   Vespa

**Libraries**

*   Annoy,
*   FAISS,
*   NMSLib
*   HNSWLib
*   Lucene
*   Chroma
*   (a million others)

**Open Source DBs**

*   Redis
*   PGVector
*   Cassandra, Mongo, etc etc (every DB seems to be getting its vector index :wink: )

**Cloud solutions**

*   Google Vertex
*   Azure AI Search

(more at [awesome vector search](https://github.com/currentslab/awesome-vector-search))

If we take vector search as one type of data store in the NoSQL paradigm, we might put it in its own category. We would say Mongo is a document database alongside CouchDB and pals. We would say Cassandra is a columnar data store, alongside the Scylla or HBase.

So in each category, we have a handful (2-3). Yet in vector search, we have dozens upon dozens of options. As a “customer” of such options, the field becomes overwhelming.

And vector retrieval increasingly isn’t the problem. The hard problems of solving real-world retrieval are not related to just getting vectors, it’s everything around it

Intent classification - given a “query” how do I know whether I can solve the problem or not (RAG) or how to route the query to the correct place Inference and reranking - given a “query”, and some candidate retrieved vectors / items, how do I perform inference on say, an arbitrary tensorflow model, to give the most relevant items? Diversity - given a “query” how do I broaden the candidate pool to more than just “similar to vectors” - to get at not just one intent, but all possible intents Lexical retrieval / hybrid search - given natural language, how do I use direct lexical signals (boring old BM25, just filtering, whatever) to give relevant results

OK and that’s just the lexical side. We’re inventing new ways of interacting with data. Nobody I talk to has really created a robust way to evaluate quality of context for RAG. There’s new UX paradigms out there - chat and chat-adjacent - that we’re experimenting with.

The challenge being, the world is wide open for experimentation, yet on first blush, all the money is being concentrated in one part of the stack. We’re not looking at the problem holistically.

## Why we’re not at peak vector DB

OK, that’s one argument, sure.

Here’s the other point of view.

There needs to be a place to focus on, and rethink, retrieval problems. In the same way NoSQL forced us to rethink databases. Capital and brainpower need a place to zero in on how to solve this next generation of retrieval + relevance problems.

So the old, curmudgeonly search person in me would say “well whatever, people realize they need search engines and use Solr / Elasticsearch anyway”.

But that’s not good enough. These search tools feel esoteric, in the average “AI Engineers” mind, they will think first of vector retrieval, then stumble into all the problems I list above. They’ll learn they need to care about all the things beyond ANN, but only after their app is stood up. In the same way the search engineers of yore backed into all kinds of problems only after comitting to a big Solr or Elasticsearch installation.

Additionally, I suspect, more and more surfaces will be driven by some retrieval-ish thing. Search-but-not-search. Real-time recommendations, but driven by vector (and other kinds of) retrieval that looks more like a search engine - not batch computed, nightly jobs common these days. I wrote about the coming revolution of “Relevance Driven Applications” in 2016, and now, its happening - not with boring old search engines as I once thought - but by reinventing our whole notion of the retrieval layer that drives user experiences.

So, I suspect the smart people at these companies will branch out beyond “making ANN better / more scalable” to building complete retrieval and ranking systems solving a tremendous array of problems. The money and effort will flow to where the customers see the problem.

In the end, like in NoSQL, we may end up with SQL again (but with all the NoSQL innovations). Or, in other words, we may end up with these vendors building a full blown “search engine”. Yet these new search engines will have many more batteries included the AI/Chat/RAG/whatever experiences people increasingly reach for.