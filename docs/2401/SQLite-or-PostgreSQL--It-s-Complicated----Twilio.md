<!--yml
category: 未分类
date: 2024-05-27 14:55:41
-->

# SQLite or PostgreSQL? It's Complicated! | Twilio

> 来源：[https://www.twilio.com/blog/sqlite-postgresql-complicated](https://www.twilio.com/blog/sqlite-postgresql-complicated)

The `run_test()` function launches one or more threads, according to the `num_threads` argument. All the threads are configured to run the `test()` function in parallel. This simulates load coming from multiple concurrent clients, each going through the list of URLs in their own random order, to create some unpredictability.

After all the threads have ended, `run_test()` prints the average request time for each request URL, and also an average of all the queries combined, which is the metric I decided to use for my analysis.

The command-line arguments allow me to pass the server root, the API key and the start and end dates to query and the concurrency. With these controls I have the ability to test a variety of scenarios.

The test script is now ready, so it’s time to get some metrics!

The development system that I work on is a Mac laptop with 6 hyperthreaded cores and 16GB of RAM. The production environment for this dashboard is a Linode virtual server with 1 vCPU and 2GB RAM.

From past experiences with benchmarking, I know that results from a fast system are not always the same as a slower one, so my end goal is to test the production system and make decisions based on the results I get on that platform.

But before doing that, I wanted to do a first round of “practice” tests on my laptop, both as a way to ensure that the test script was working correctly, and also, why not admit it, because I was curious about how these two databases perform on a fairly powerful platform.

The testing methodology that I decided to use is as follows: I would test the system running under the two databases, for queries with periods of a week, a month, a quarter and a year, with all queries having 2021-01-01 as the start date. I would also repeat the tests with 1, 2 and 4 concurrent clients. For each individual test I would run the script three times, and record the best run of the three. The metric I would use is the total average of all queries.

With this plan I ended up with 24 data points (2 databases x 4 query periods x 3 concurrency levels). The following chart shows the response time for PostgreSQL (blue) and SQLite (red), with a single client.