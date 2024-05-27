<!--yml
category: 未分类
date: 2024-05-27 13:34:24
-->

# The Rise of GQL: A New ISO Standard in Graph Query Language

> 来源：[https://www.tigergraph.com/blogs/gsql/the-rise-of-gql-a-new-iso-standard-in-graph-query-language/](https://www.tigergraph.com/blogs/gsql/the-rise-of-gql-a-new-iso-standard-in-graph-query-language/)

by [Mingxi Wu](https://www.linkedin.com/in/mingxi-wu-a1704817/)

We’re thrilled to celebrate a significant milestone in the world of database technology: the publication of the first version of the ISO GQL standard on April 12, 2024\. You can access the standard [here](https://www.iso.org/standard/76120.html). 

As an active contributor to the development of GQL standard version 1, I’ve witnessed the entire journey from its inception to its high-quality final publication. I can attest that this marks a significant milestone in the history of databases. Thanks to the GQL editor [Stefan Plantikow](https://www.linkedin.com/in/stefan-plantikow-49896637/) and [Stephen Cannan](https://www.linkedin.com/in/ACoAAAAtayUB9pzZK_nIKrdtC8hm6aoCRwnzxk8), editing tooling support from [Jim Melton](https://www.linkedin.com/in/ACoAAADNnC4BsfclEJ6S5Ux61kbqFFtkzTn8dA4), the convenor of WG3 [Keith Hare](https://www.linkedin.com/in/ACoAAAJRseUBq_q6BsMZ8gZhHW3T3StojwXC4X0), and  the original author of the GQL manifesto [Alastair Green](https://www.linkedin.com/in/ACoAABa6wGYB8jtCFp2w2BWnBEWhFrKNYNm9dzE). This elegant standard is developed with a strong international team including both academia and industry query language experts. A non-exhaustive contributor list can be found on these two SIGMOD papers [1](https://dl.acm.org/doi/abs/10.1145/3514221.3526057), [2](https://dl.acm.org/doi/10.1145/3183713.3190654).

In this blog, I’m going to share my personal enthusiasm on GQL and invite you to start learning it. 

### **What is GQL?**

GQL, or Graph Query Language, is an ISO standard defined for property graph databases. It stands as the first sibling database language emerging from the ISO standard committee for databases since the initial publication of the SQL standard in 1986.

GQL aims to become the de facto query language standard for property graph databases, addressing the growing demand for such databases.

Let’s revisit some background knowledge on property graph databases. 

Firstly, property graph databases utilize property graph data modeling, where vertices and edges serve as the fundamental units to represent data elements. In contrast, the traditional relational data model relies on tables for data representation. Intuitively, the property graph model offers greater flexibility and aligns more closely with real-world interactions between objects.

Secondly, there are various storage formats tailored to support the property graph data model. Native graph databases distinguish themselves from non-native counterparts primarily through their storage format. In a native graph database, vertices and edges are treated as first-class citizens within the storage layer, resulting in superior performance, particularly in path traversal and iterative graph algorithms. This performance boost stems from the native graph database’s inherent structure as a gigantic object index. During graph traversal, one navigates from an active vertex set to another via their connecting edges, circumventing the need for costly runtime joins (as relational databases do) to reconnect data elements. In my opinion, native graph database’s graph storage format is the key differentiator to other databases. TigerGraph falls in the category of native graph database. 

### **Why GQL and Why Now?**

As the graph database industry evolves, it becomes the third canonical database, alongside relational databases and key-value store databases. In the past decade, the burgeoning graph database industry has witnessed a plethora of vendors offering their own graph database products, each accompanied by their proprietary graph query language. Examples include Neo4j’s Cypher, TigerGraph’s GSQL, Oracle’s PGQ, LDBC’s G-core, and Gremlin, among others. For a comprehensive overview of the graph query language landscape, I recommend referring to Peter Boncz’s insightful [survey slide](https://homepages.cwi.nl/~boncz/gql-survey.pdf) from CWI.

Amidst this thriving ecosystem, GQL emerged to address the growing demand for a standardized graph query language. Its publication establishes a solid foundation and drives the prosperity of graph databases in the coming years, akin to what SQL did for relational databases

### **Scratching the GQL Surface**

At its core, GQL uses pattern matching syntax to declaratively ask queries against graph databases, similar to SQL for relational databases.

The building blocks of a pattern are (1) vertex pattern and (2) edge pattern, illustrated below.

*   **MATCH** (x:Account **WHERE** x.isBlocked=’no’)
    *   x binds to a vertex in the graph. () is a node pattern.
*   **MATCH** −[e:Transfer **WHERE** e.amount>5M]−>
    *   e binds to an edge in the graph. -[]-> is an edge pattern.

With the above building blocks, you can have a longer pattern by alternating the node and edge patterns. 

*   **MATCH** (x:Account)-[:SignInWithIP]->(y:IP)  -[:Associated]->(p:Phone)

Each matched pattern provides a table. You can filter, group by, aggregate, and project the columns of the table, as you do in SQL.

GQL has two syntax flavors: one is Cypher, and the other is SQL. However, both flavors share the same core pattern matching part.

If you’re coming from the SQL world, you’ll use something like this.

*   **SELECT** x.age, y.ip**FROM** g **MATCH** (x:Account)-[:SignInWithIP]-(y:IP)  **WHERE** x.name == “John”

*Note: TigerGraph is the main contributor and advocate for the SQL flavor in GQL because we firmly believe that SQL users will have a much easier time learning GQL with the SQL dialect. *

If you’re familiar with the Cypher world, you’ll use something like the example below.

*   **MATCH** (x:Account)-[:SignInWithIP]-(y:IP)  **WHERE** x.name == “John”  **RETURN** x.age, y.ip

Furthermore, GQL supports linear composition of pattern match statements, meaning that each MATCH statement’s generated result table columns can be used to drive the next MATCH statement. This linear composition can be illustrated by the two flavors:

{

**SELECT** expression_list  **FROM** g **MATCH** graph_pattern1  **WHERE** xxx

**NEXT**

**SELECT** expression_list  **FROM** g **MATCH** graph_pattern2  **WHERE** yyy

**NEXT**

**SELECT** expression_list  **FROM** g **MATCH** graph_pattern3  **WHERE** zzz

}

{

**USE** g  **MATCH** graph_pattern1  **WHERE** xxx  **YIELD** expression_list

**NEXT**

**USE** g**MATCH** graph_pattern2  **WHERE** yyy  **YIELD** expression_list    **NEXT**

**USE** g  **MATCH** graph_pattern2  **WHERE** zzz  **RETURN** expression_list  

}

 Besides the query syntax, GQL also supports a Linux file system-style directory hierarchy to host graph schemas and their catalog objects. This physical design of the catalog layout is to adapt to the flexibility of two-level property graph modeling, where a graph is a logical container and vertices and edges are the base containers. This design was initially informally discussed between TigerGraph and Neo4j, as we were first inspired by our multi-graph modeling product offering—multiple graphs can share a subset of the base vertex and edge containers. After multi-round constructive discussions, we agree that graph schemas will evolve freely under this catalog layout, especially for cross graph sharing and joining.

There are many other gems in the GQL standard; interested readers can purchase and study the standard.

### **How To Learn It?**

Many TigerGraph users have asked me how to learn GQL or have an early peek. Here is my suggested approach for a quick ramp-up:

1.  Study the core pattern matching design: get familiar with the Pattern Match syntax by reading [this](https://arxiv.org/abs/2112.06217) SIGMOD paper. It lays the foundation for understanding pattern matching nuances and design philosophy.
2.  Study existing LDBC-SNB BI benchmark queries written in [Cypher](https://github.com/ldbc/ldbc_snb_bi/tree/main/cypher/queries) and [GSQL](https://github.com/ldbc/ldbc_snb_bi/tree/main/tigergraph/queries_syntax_v3). They bear many similarities to GQL syntax.
3.  Dive into the Standard: read the 628-page ISO GQL standard (cost: 217 CHF) for a comprehensive understanding of the language’s specifications.

Last and the most important thing, start writing!

### **What’s Next for GQL?**

GQL version 1 marks just the beginning of an ongoing journey. As the industry evolves, there are still challenges to address, including graph view support, trigger support, enhanced graph algorithm capabilities, and join graphs and tables, among others. At TigerGraph, we’re actively implementing GQL to champion this standard, maintaining our unwavering commitment to both openCypher and GQL.

Reflecting on our journey, Alin Deutsch, Yu Xu, and I began inventing GSQL in the summer of 2015 in Mountain View, CA, drawing inspiration from both the SQL standard and pattern matching literature. Alin and I implemented the initial version of GSQL that summer, burning many night candles. Throughout this process, we’ve contributed our best practices and real-world experiences back to the ISO GQL standard. We’re continuing this journey with the ISO standard committee, actively shaping the future versions of GQL. Along the way, we’ve forged numerous friendships within the global graph community, collectively delivering this beautiful standard. We hope you’ll recognize the dedication and passion this community has poured into this endeavor and take the time to learn, appreciate the design, and, most importantly, start using it.

In conclusion, GQL represents a significant leap forward in standardizing graph query languages. Whether you’re a seasoned database professional or new to graph databases, now is the time to start learning GQL and embrace the future of graph data management. Stay tuned for exciting developments in the world of GQL!

If you want to learn more about TigerGraph and GQL contact us at [[email protected]](/cdn-cgi/l/email-protection).