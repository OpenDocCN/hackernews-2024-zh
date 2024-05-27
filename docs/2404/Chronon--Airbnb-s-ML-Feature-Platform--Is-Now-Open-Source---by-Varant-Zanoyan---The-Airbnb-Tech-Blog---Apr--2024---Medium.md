<!--yml
category: 未分类
date: 2024-05-27 13:03:39
-->

# Chronon, Airbnb’s ML Feature Platform, Is Now Open Source | by Varant Zanoyan | The Airbnb Tech Blog | Apr, 2024 | Medium

> 来源：[https://medium.com/airbnb-engineering/chronon-airbnbs-ml-feature-platform-is-now-open-source-d9c4dba859e8](https://medium.com/airbnb-engineering/chronon-airbnbs-ml-feature-platform-is-now-open-source-d9c4dba859e8)

# **Chronon, Airbnb’s ML Feature Platform, Is Now Open Source**

A feature platform that offers observability and management tools, allows ML practitioners to use a variety of data sources, while handling the complexity of data engineering, and provides low latency streaming.

By: [Varant Zanoyan](https://www.linkedin.com/in/vzanoyan/), [Nikhil Simha Raprolu](https://www.linkedin.com/in/nikhilsimha/)

*Chronon allows ML practitioners to use a variety of data sources as inputs to feature transformations. It handles the complexity of data plumbing, such as batch and streaming compute, provides low latency serving, and offers a host of observability and management tools.*

**Airbnb is happy to announce that** [**Chronon**](https://www.chronon.ai)**, our ML Feature Platform, is now open source. Join our** [**community Discord channel**](https://discord.gg/GbmGATNqqP) **to chat with us.**

**We’re excited to be making this announcement along with our partners at Stripe, who are early adopters and co-maintainers of the project.**

This blog post covers the main motivation and functionality of Chronon. For an overview of core concepts in Chronon, please see [this previous post](/airbnb-engineering/chronon-a-declarative-feature-engineering-framework-b7b8ce796e04).

# Background

We built Chronon to relieve a common pain point for ML practitioners: they were spending the majority of their time managing the data that powers their models rather than on modeling itself.

Prior to Chronon, practitioners would use one of the following two approaches:

1.  **Replicate offline-online:** ML practitioners train the model with data from the data warehouse, then figure out ways to replicate those features in the online environment. The benefit of this approach is that it allows practitioners to utilize the full data warehouse, both the data sources and powerful tools for large-scale data transformation. The downside is that this leaves no clear way to serve model features for online inference, resulting in inconsistencies and label leakage that severely affect model performance.
2.  **Log and wait:** ML practitioners start with the data that is available in the online serving environment from which the model inference will run. They log relevant features to the data warehouse. Once enough data has accumulated, they train the model on the logs, and serve with the same data. The benefit of this approach is that consistency is guaranteed and leakage is unlikely. However the major drawback is that it can result in long wait times, hindering the ability to respond quickly to changing user behavior.

The Chronon approach allows for the best of both worlds. Chronon requires ML practitioners to define their features only once, powering both offline flows for model training as well as online flows for model inference. Additionally, Chronon offers powerful tooling for feature chaining, observability and data quality, and feature sharing and management.

# How It Works

Below we explore the main components that power most of Chronon’s functionality using a simple example derived from the [quickstart guide](https://chronon.ai/getting_started/Tutorial.html). You can follow that guide to run this example.

Let’s assume that we’re a large online retailer, and we’ve detected a fraud vector based on users making purchases and later returning items. We want to train a model to predict whether a given transaction is likely to result in a fraudulent return. We will call this model each time a user starts the checkout flow.

## Defining Features

**Purchases Data:** We can aggregate the purchases log data to the user level to give us a view into this user’s previous activity on our platform. Specifically, we can compute SUMs, COUNTs and AVERAGEs of their previous purchase amounts over various time windows.

```
source = Source(
    events=EventSource(
        table="data.purchases", # This points to the log table in the warehouse with historical purchase events, updated in batch daily
        topic="events/purchases", # The streaming source topic
        query=Query(
            selects=select("user_id","purchase_price"), # Select the fields we care about
            time_column="ts") # The event time
    ))

window_sizes = [Window(length=day, timeUnit=TimeUnit.DAYS) for day in [3, 14, 30]] # Define some window sizes to use below

v1 = GroupBy(
    sources=[source],
    keys=["user_id"], # We are aggregating by user
    online=True,
    aggregations=[Aggregation(
            input_column="purchase_price",
            operation=Operation.SUM,
            windows=window_sizes
        ), # The sum of purchases prices in various windows
        Aggregation(
            input_column="purchase_price",
            operation=Operation.COUNT,
            windows=window_sizes
        ), # The count of purchases in various windows
        Aggregation(
            input_column="purchase_price",
            operation=Operation.AVERAGE,
            windows=window_sizes
        ), # The average purchases by user in various windows
        Aggregation(
            input_column="purchase_price",
            operation=Operation.LAST_K(10),
        ), # The last 10 purchase prices aggregated as a list
    ],
)
```

*This creates a `GroupBy` which transforms the `purchases` event data into useful features by aggregating various fields over various time windows, with `user_id` as a primary key.*

This transforms raw purchases log data into useful features at the user level.

**User Data:** Turning User data into features is a littler simpler, primarily because we don’t have to worry about performing aggregations. In this case, the primary key of the source data is the same as the primary key of the feature, so we can simply extract column values rather than perform aggregations over rows:

```
source = Source(
    entities=EntitySource(
        snapshotTable="data.users", # This points to a table that contains daily snapshots of all users
        query=Query(
            selects=select("user_id","account_created_ds","email_verified"), # Select the fields we care about
        )
    ))

v1 = GroupBy(
    sources=[source],
    keys=["user_id"], # Primary key is the same as the primary key for the source table
    aggregations=None, # In this case, there are no aggregations or windows to define
    online=True,
) 
```

*This creates a `GroupBy` which extracts dimensions from the `data.users` table for use as features, with `user_id` as a primary key.*

**Joining these features together:** Next, we need to combine the previously defined features into a single view that can be both backfilled for model training and served online as a complete vector for model inference. We can achieve this using the Join API.

For our use case, it’s very important that features are computed as of the correct timestamp. Because our model runs when the checkout flow begins, we want to use the corresponding timestamp in our backfill, such that feature values for model training logically match what the model will see in online inference.

Here’s what the definition would look like. Note that it combines our previously defined features in the right_parts portion of the API (along with another feature set called returns).

```
 source = Source(
    events=EventSource(
        table="data.checkouts", 
        query=Query(
            selects=select("user_id"), # The primary key used to join various GroupBys together
            time_column="ts",
            ) # The event time used to compute feature values as-of
    ))

v1 = Join(  
    left=source,
    right_parts=[JoinPart(group_by=group_by) for group_by in [purchases_v1, returns_v1, users]] # Include the three GroupBys
)
```

# Backfills/Offline Computation

The first thing that a user would likely do with the above Join definition is run a backfill with it to produce historical feature values for model training. Chronon performs this backfill with a few key benefits:

1.  **Point-in-time accuracy:** Notice the source that is used as the “left” side of the join above. It is built on top of the “data.checkouts” source, which includes a “ts” timestamp on each row that corresponds to the logical time of that particular checkout. Every feature computation is guaranteed to be window-accurate as of that timestamp. So for the one-month sum of previous user purchases, every row will be computed for the user as of the timestamp provided by the left-hand source.
2.  **Skew handling:** Chronon’s backfill algorithms are optimized for handling highly skewed datasets, avoiding frustrating OOMs and hanging jobs.
3.  **Computational efficiency optimizations:** Chronon is able to bake in a number of optimizations directly into the backend, reducing compute time and cost.

# Online Computation

Chronon abstracts away a lot of complexity for online feature computation. In the above examples, it would compute features based on whether the feature is a batch feature or a streaming feature.

**Batch features (for example, the User features above)**

Because the User features are built on top of a batch table, Chronon will simply run a daily batch job to compute the new feature values as new data lands in the batch data store and upload them to the online KV store for serving.

**Streaming features (for example, the Purchases features above)**

The Purchases features are built on a source that includes a streaming component, as indicated by the inclusion of a “topic” in the source. In this case, Chronon will still run a batch upload in addition to a streaming job for real time updates. The batch jobs is responsible for:

1.  **Seeding the values:** For long windows, it wouldn’t be practical to rewind the stream and play back all raw events.
2.  **Compressing “the middle of the window” and providing tail accuracy:** For precise window accuracy, we need raw events at both the head and the tail of the window.

The streaming job then writes updates to the KV store to keep feature values up to date at fetch time.

# Online Serving / Fetch API

Chronon offers an API to fetch features with low latency. We can either fetch values for individual GroupBys (i.e. the Users or Purchases features defined above) or for a Join. Here’s an example of what one such request and response for a Join would look like:

```
// Fetching all features for user=123
Map<String, String> keyMap = new HashMap<>();
keyMap.put("user", "123")
Fetcher.fetch_join(new Request("quickstart_training_set_v1", keyMap));
// Sample response (map of feature name to value)
'{"purchase_price_avg_3d":14.2341, "purchase_price_avg_14d":11.89352, ...}'
```

*Java code that fetches all features for user 123\. The return type is a map of feature name to feature value.*

The above example uses the Java client. There is also a Scala client and a Python CLI tool for easy testing and debugging:

```
run.py --mode=fetch -k '{"user_id":123}' -n quickstart/training_set -t join

> {"purchase_price_avg_3d":14.2341, "purchase_price_avg_14d":11.89352, ...}
```

*Utilizes the run.py CLI tool to make the same fetch request as the Java code above. run.py is a convenient way to quickly test Chronon workflows like fetching.*

Another option is to wrap these APIs into a service and make requests via a REST endpoint. This approach is used within Airbnb for fetching features in non-Java environments such as Ruby.

# Online-Offline Consistency

Chronon not only helps online-offline accuracy, it also offers a way to measure it. The measurement pipeline starts with the logs of the online fetch requests. These logs include the primary keys and timestamp of the request, along with the fetched feature values. Chronon then passes the keys and timestamps to a Join backfill as the left side, asking the compute engine to backfill the feature values. It then compares the backfilled values to actual fetched values to measure consistency.

# What’s Next?

Open source is just the first step in an exciting journey that we look forward to taking with our partners at Stripe and the broader community.

Our vision is to create a platform that enables ML practitioners to make the best possible decisions about how to leverage their data and makes enacting those decisions as easy as possible. Here are some questions that we’re currently using to inform our roadmap:

**How much further can we lower the cost of iteration and computation?**

Chronon is already built for the scale of data processed by large companies such as Airbnb and Stripe. However, there are always further optimizations that we can make to our compute engine, both to reduce the compute cost and the “time cost” of creating and experimenting with new features.

**How much easier can we make authoring a new feature?**

Feature engineering is the process by which humans express their domain knowledge to create signals that the model can leverage. Chronon could integrate NLP to allow ML practitioners to express these feature ideas in natural language and generate working feature definition code as a starting point for their iteration.

Lowering the technical bar to feature creation would in turn open the door to new kinds of collaboration between ML practitioners and partners who have valuable domain expertise.

**Can we improve the way models are maintained?**

Changing user behavior can cause shifts in model performance because the data that the model was trained on no longer applies to the current situation. We imagine a platform that can detect these shifts and create a strategy to address them early and proactively, either by retraining, adding new features, modifying existing features, or some combination of the above.

**Can the platform itself become an intelligent agent that helps ML practitioners build and deploy the best possible models?**

The more metadata that we gather into the platform layer, the more powerful it can become as a general ML assistant.

We mentioned the goal of creating a platform that can automatically run experiments with new data to identify ways to improve models. Such a platform might also help with data management by allowing ML practitioners to ask questions such as “What kinds of features tend to be most useful when modeling this use case?” or “What data sources might help me create features that capture signal about this target?” A platform that could answer these types of questions represents the next level of intelligent automation.

# Getting Started

Here are some resources to help you get started or to evaluate if Chronon is a good fit for your team.

Interested in this type of work? Check out our open roles [here](https://careers.airbnb.com/) — we’re hiring.

# Acknowledgements

**Sponsors:** [Henry Saputra](mailto:henry.saputra@airbnb.com) [Yi Li](mailto:yi.li@airbnb.com) [Jack Song](mailto:jack.song@airbnb.com)

**Contributors:** [Pengyu Hou](mailto:pengyu.hou@airbnb.com) [Cristian Figueroa](mailto:cristian.figueroa@airbnb.com) [Haozhen Ding](mailto:haozhen.ding@airbnb.com) [Sophie Wang](mailto:sophie.wang@airbnb.com) [Vamsee Yarlagadda](mailto:vamsee.y@airbnb.com) [Haichun Chen](mailto:haichun.chen@airbnb.com) [Donghan Zhang](mailto:donghan.zhang@airbnb.com) [Hao Cen](mailto:hao.cen@airbnb.com) [Yuli Han](mailto:yuli.han@airbnb.com) [Evgenii Shapiro](mailto:evgeny.shapiro@airbnb.com) [Atul Kale](http://atul.kale@airbnb.com) Patrick Yoon