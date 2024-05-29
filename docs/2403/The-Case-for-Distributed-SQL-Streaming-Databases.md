<!--yml
category: 未分类
date: 2024-05-27 14:30:31
-->

# The Case for Distributed SQL Streaming Databases

> 来源：[https://risingwave.com/blog/the-case-for-distributed-sql-streaming-databases/](https://risingwave.com/blog/the-case-for-distributed-sql-streaming-databases/)

## Introduction

The statement above may seem provocative, but there is an element of truth in it. As businesses strives to leverage data for implementing new business models, it becomes clear that the existing data stack falls short. The traditional data stack was not designed for today’s ultra-low latency requirements. Before delving into the emerging requirements of new applications, it is important to step back and understand the expectations of data-driven organizations. Equally important is identifying common trends in data characteristics that are driving these new requirements.

Let's take a trip down memory lane to about a decade ago. During that time, the key trend in the data and analytics field was undoubtedly the 'Big Data' movement. Thought leaders had defined ‘Big Data’ by three V’s: Volume, Velocity and Variety.

In simple terms, big data refers to large and complex data sets, often from new sources. These data sets are too big for traditional software to handle, but they can be used to solve business problems that were previously impossible to address.

There was immense promise for businesses to extract meaningful information from massive amounts of data. However, this potential remained untapped due to a lack of tools for handling such large datasets. The introduction of technologies like Hadoop was expected to unlock this potential. However, it became clear that these Big Data technologies primarily focused on addressing the ***volume*** aspect. As a result, widespread adoption by enterprises was not possible, as the majority of users didn't see the need or value in these tools.

There are multiple reasons, but the primary one is the limited shelf life of data. Data practitioners faced challenges in accessing data in real-time, precisely when its intrinsic value is high. Simply storing raw data in a data lake resembled a data dump rather than leveraging a valuable resource.

Another significant reason was the state of data engineering practices. Even when data was accessible, it often proved inadequate for analysis in its original form. As a result, complex Extract-Transform-Load (ETL) processes became necessary to extract valuable information from the data. Data remained isolated in separate systems, known as 'siloes', and closely tied to specific applications. The integration of data sources only recently improved through mechanisms such as message queues and CDC connectors.

## 
Evolution in the Characteristics of Data

Traditionally, all data practitioners have cared about the following characteristics:

These characteristics have been addressed in relational database systems. Database systems' transaction management features support ACID properties to handle these characteristics.

*   Atomicity: Ensures **completeness** with all-or-nothing semantics.

*   Consistency: Enforces **data** **accuracy** through constraints.

*   Isolation: Provides guarantees for **data integrity** and **correctness**.

*   Durability: Ensures **reliability** of data based on immutable writes.

These characteristics have been effective in addressing various business requirements. Current data processing systems have ensured robust support for these characteristics in any data stack. As a result, businesses have been able to execute workloads that rely on static snapshots of data. While there has been significant effort to enhance the speed and real-time capabilities of these workloads through various optimizations, these improvements are no longer adequate.

Among industry observers, there is a growing consensus on the value of viewing data as a continuous unbounded stream, rather than a snapshot. Businesses are no longer content with knowing what happened in the past; instead, they are focused on predicting future outcomes, which necessitates analyzing data in "real-time". In this context, "real-time" is defined by data latency, rather than query latency. To gain a better understanding, we need to establish a new set of characteristics for defining data.

To address these characteristics, a new data processing paradigm is required. This paradigm will:

*   Handle discrete granular event data.

*   Process live data continuously.

*   Integrate multiple data streams for stateful processing.

## 
Early Stream Processing Solutions

A new data processing stack is necessary to support the new paradigm discussed in the previous section. This data stack should have the following features:

*   Event data semantics to maintain consistency of event data.

*   An incremental computation model for continuous updates on live data.

*   A familiar relational data model to treat streams as tables, enabling seamless integration of various data sources.

**First-generation stream processing systems**: Stream processing systems have been trying to meet these requirements for quite some time. The first-generation stream processing systems, such as Spark Streaming, Apache Heron, and early versions of Flink, have proven their value in certain aspects. For instance, they excelled in micro-batch processing and were well-suited for specific use cases. Spark Streaming, for instance, was a valuable addition for Spark users looking to incorporate stream processing into their existing workloads. In general, these systems inherited many benefits from the mature batch processing model.

However, they also inherited scheduling and coordination issues from the legacy batch model. They did not support true event time semantics, which is crucial for building applications in event-driven architectures. Additionally, these technologies only focused on the data processing aspect. The lack of a data store meant that a separate data store was needed for persistence, resulting in slower application performance and increased operational overhead. Furthermore, these systems were primarily designed for early adopters who were comfortable working with low-level APIs and interfaces. As a result, these technologies did not significantly advance the ability to quickly and easily build real-time applications.

## 
New Breed of Stream Processing Solutions

Early stream processing systems have traditionally been designed with a code-first approach. In order for stream processing to become more widely adopted, it is essential to incorporate SQL as the standard API. Additionally, the new system should include a built-in storage layer to efficiently handle data retrieval.

**Enter streaming databases**: They are designed to combine the incremental processing capabilities of stream-processing engines with the SQL-based analysis and persistence capabilities of traditional databases. The emergence of a new breed of streaming databases could improve operational inefficiencies compared to approaches that rely on separate platforms for stream and batch processing. Streaming databases, such as [RisingWave](https://www.risingwave.com/) and [Materialize](https://materialize.com/), are designed to continuously process streams of event data using SQL queries and real-time materialized views. They also persist historical event data for further analysis.

Unlike streaming compute engines that store data in an external database, streaming databases are specifically designed to offer built-in processing and persistence capabilities. This means that a single streaming database can serve as a viable alternative to using a combination of tools such as Apache Flink and Apache Cassandra. By doing so, it simplifies deployment, configuration, integration, and management tasks. With a streaming database, there is a paradigm shift towards moving the database functionality upstream, enabling real-time processing of data as it arrives and facilitating prompt data serving.

## 
Looking Ahead

We are making stream processing accessible to a wider range of users by combining the strengths of early stream processing systems with traditional database systems. This innovative approach revolutionizes real-time data processing and empowers more people to utilize it. By bridging the gap between stream processing agility and traditional database reliability, we create a more inclusive and user-friendly data processing landscape.

The implications of this convergence are profound. Businesses can leverage real-time data analysis to make informed decisions, predict outcomes, and gain a competitive edge. Continuous live data processing and integration of multiple data streams enable various use cases, including fraud detection, real-time personalization, supply chain optimization, and IoT analytics. Moreover, the democratization of stream processing allows data engineers, scientists, and analysts to tap into real-time data processing without requiring extensive technical expertise.

In conclusion, this integration addresses limitations and empowers businesses with real-time data analysis. Stream processing is becoming more accessible, efficient, and transformative for organizations across industries. The democratization of stream processing is now within reach, opening up possibilities for innovation and growth.