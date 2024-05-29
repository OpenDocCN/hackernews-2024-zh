<!--yml
category: 未分类
date: 2024-05-27 14:55:54
-->

# Procedure for the creation of vector databases Benchmark tests and their results

> 来源：[https://www.adesso.de/en/news/blog/procedure-for-the-creation-of-vector-databases-benchmark-tests-and-their-results.jsp](https://www.adesso.de/en/news/blog/procedure-for-the-creation-of-vector-databases-benchmark-tests-and-their-results.jsp)

#### Vectors and Vector Databases

A vector is a mathematical object that can be represented as an ordered collection of numbers, called components, that correspond to the extent of the vector along different coordinate axes. In two-dimensional space (a plane), a vector can be represented as (x,y), where x and y are the components along the horizontal and vertical axes, respectively. In three-dimensional space, a vector can be represented as (x,y,z), with components along the x, y and z axes. There are many ways to define a concept of similarity between a pair of vectors (Euclidean distance and cosine similarity are two examples). Euclidean distance generalises our natural concept of the distance between two points on a plane (#pythagoreantheorem), while cosine similarity captures the idea that two vectors are similar if they point in roughly the same direction.

At their core, vector databases are designed to store collections of metadata-enriched vectors of the same size, and to perform fast approximate similarity searches over such a collection. The question they can answer efficiently in a short time is:

> “Given an arbitrary vector X, what are the 5 approximately closest vectors to X in a stored collection?”.

The power of vector databases is unlocked when the stored vectors are obtained as semantic embeddings of their associated metadata, and the similarity between two vectors captures the concept of semantic proximity of the metadata. If the metadata is natural language, this can be achieved using an embedding algorithm such as word2vec or a large language model such as Aleph Alpha's Luminous or the OpenAI GPT models.

From an application perspective, such a vectorised document store can serve as a knowledge store for the retrieval phase of a Retrieval Augmented Generation (RAG) architecture.

You can also find more information in this blog post on the topic: "[A clever way to retrieve information: conversational agents as a tool to access knowledge held at the company](https://www.adesso.de/en/news/blog/a-clever-way-to-retrieve-information-conversational-agents-as-a-tool-to-access-knowledge-held-at-the-company-2.jsp)".

#### Benchmarking Vector Databases

When you are building a data-intensive application (or any application for that matter), finding the sweet spot on a performance/cost scale for each component is critical. For a vector database, there are two important performance issues:

*   1\. How long does it take to insert a vector (including making it discoverable for similarity searches)?

*   2\. Given an input vector, how long does it take to find the N nearest vectors?

It is important to note that the answers to the second question should always be seen in the light of the accuracy of the approximation. After all, the worse an approximation is allowed to be, the easier it is to derive it quickly. When optimising the accuracy of the similarity search, one often has to carefully adjust parameters that can strongly influence the time consumption of both the upload and the search, and whose choice is highly dependent on the structure of the underlying dataset. Therefore, in order to keep the parameter space for our benchmark test manageable, we focus on investigating the above issues without optimising for similarity search accuracy.

#### The Databases

In our experiment, we benchmarked three open source contenders in the vector database race - Chroma, Qdrant and Weaviate.

##### Chroma

Chroma is an AI-native, open source embedding database. The company has raised an impressive $18 million seed round, led by Astasia Myers of Quiet Capital, underlining investors' belief in its potential. Chroma advertises itself as coming with batteries, allowing developers to spin up database instances quickly and conveniently, making development easier.

##### Weaviate

Weaviate is an open source search engine and database, developed by Amsterdam-based SeMI Technologies since 2019\. Weaviate's stated mission is to democratise search capabilities, which have previously been monopolised by a few major tech companies. The company's software gives customers the option to run it on their infrastructure, allowing them to maintain control of their data behind their firewalls. The company also offers a managed cloud service, Weaviate Cloud, which provides additional flexibility for organisations that prefer a cloud-based approach to data management.

##### Qdrant

Qdrant is an open source vector database that started more than two years ago with a mission to build a vector database using Rust as the system programming language. In early 2023, Qdrant expanded its offering with the launch of Qdrant Cloud, a managed vector database solution serving more than 1,000 Qdrant clusters. Qdrant is now expanding its offering with managed on-premises solutions for enterprise customers, further solidifying its position as a versatile and accessible vector database solution.

#### The Experiment

In our benchmark, we tested the performance of Chroma, Qdrant and Weaviate in terms of vector upload and retrieval performance using the GloVe 100 angular dataset. This dataset is a collection of 1.1 million 100-dimensional vectors obtained as a word representation by the GloVe algorithm.

You can find more information about glove100_angular [here](https://www.tensorflow.org/datasets/catalog/glove100_angular).

To test vector insertion performance, we loaded 1,183,514 100-dimensional vectors in batches of approximately 1000 vectors into each database on each of the test infrastructures. This tests the database's ability to ingest data quickly and efficiently, a key metric for use cases involving large amounts of data.

For vector retrieval, we performed 10,000 queries on the loaded data. This measures the search speed of the database, which is critical for applications that need to provide fast results to user queries.

The infrastructure for our tests included three sizes of Azure virtual machines (VMs) - small (S), medium (M) and large (L). Each virtual machine was tested with both a hard disk drive (HDD) and a solid state drive (SSD). The specifications for each VM size were: