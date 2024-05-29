<!--yml
category: 未分类
date: 2024-05-27 14:52:14
-->

# Demystifying the use of the Parquet file format for time series - SenX

> 来源：[https://blog.senx.io/demystifying-the-use-of-the-parquet-file-format-for-time-series/](https://blog.senx.io/demystifying-the-use-of-the-parquet-file-format-for-time-series/)

As the maker of [Warp 10](https://warp10.io/), the most advanced Time Series Platform, we are regularly in contact with people who have been storing time series data for quite some time. The diversity of technologies that have employed would surprise you. Among those technologies we often encounter the [Parquet](https://parquet.apache.org) file format which is usually chosen *because it is a columnar format and therefore it is performant for time series data*. When we ask further questions to better understand the schema adopted, how those files are used and what storage and access performance they allow, the answers often show the way the Parquet format works is not very well understood. This post aims at giving you a better understanding of the internals of the Parquet format and of its suitability for time series data.

## The Parquet file format

### History

In 2010 at the [VLDB Conference](https://vldb.org/), people from [Google](https://google.com/) presented a paper titled [Dremel: Interactive Analysis of Web-Scale Datasets](https://static.googleusercontent.com/media/research.google.com/en//pubs/archive/36632.pdf) which described an internal tool named [*Dremel*](https://en.wikipedia.org/wiki/Dremel_(software)) and the way it organized data.

In the summer of 2012, [Julien Le Dem](https://twitter.com/J_) who was then working at Twitter, [tweeted](https://twitter.com/J_/status/239247373581316096) that he had found an error in the Dremel paper, the reason was that he had started working on an open source implementation of the nested record format used by Dremel for use in Hadoop, a project that would a little later become known as [Parquet](https://web.archive.org/web/20130415111841/http://parquet.io/).

### Fundamentals

The novelty described in the *Dremel* paper was a clever way of enabling complex records to be stored in a columnar format.

The idea behind columnar formats is that by grouping data in *columns* instead or *rows*, similar values end up together and lead to better compression, and at retrieval time only the needed data can be read if not all columns are needed. Systems such as [HBase](https://hbbase.apache.org) had long made use of the columnar format paradigm.

But if the columnar format was well suited for *table like* data, it was not adapted to nested data structures which often would end up in a single column hence annihilating the benefit of the columnar format if only specific fields of such a structure are needed.

This is the issue that the Dremel paper addressed and solved with a clever mechanism for describing how fields are nested and repeated. We let you deep dive into section 4 of the original Dremel paper to learn the intrinsics of how records are split in columns.

### Implementation

The Parquet implementation uses the *flattening* principles described in the Dremel paper and organises a file in the following manner:

A *file* is divided in *row groups* which contain values for various columns stored in *column chunks*. The actual data is packed in *pages* within those column chunks. Pages are compressed using a mechanism adapted to the type of values they store. At the end of the file a *footer* contains various informations, such as the file schema, the index of the row groups and column chunks and information such as statistics for each column chunk.

When reading data from a Parquet file, the footer is read to identify the offsets where the column chunks can be read, the chunks matching the requested data are then accessed and the records reconstructed from the data for the individual columns.

A predicate pushdown mechanism in the Parquet read API can exploit the statistics in the file footer to only read the row groups containing values matching a predicate.

### Essential takeaway

The Parquet format is an intelligent *columnar* format with the ability to store complex nested or repeated data structures as columns. So in short, every field, or every element of repeated fields in the stored data structures end up in its own column. This is fundamental to understand and has important implications when storing time series in Parquet.

## Storing Time Series in Parquet

The motivation behind the selection of the Parquet format to store time series could be that those data need to be processed using a parallel processing framework such as [Spark](https://spark.apache.org/), that they need to be read by a query engine such as [Impala](https://impala.apache.org) or [Drill](https://drill.apache.org), or it could be that you are considering using Parquet for the storage layer of the [next rewrite](https://www.influxdata.com/blog/announcing-influxdb-iox/) of your time series database or of your [cloud service](https://docs.microsoft.com/en-us/azure/time-series-insights/concepts-storage#parquet-file-format-and-folder-structure).

### Data model

Before you can actually store your data in a Parquet file you need to have a clear idea of how you want to model your data as this model will directly reflect in the structure of the records which will be stored.

When working with time series you basically have two ways to model the data. The first one, the most flexible, is to consider series as individual entities which will be treated independently but may be manipulated jointly with other series. The second way, more naive, is to consider series as columns of a table. The records which are stored are therefore really rows which, at a given timestamp, contain values for various columns. This is model typical of IT monitoring use cases where each record is a photograph of several metrics. It is less suited for IoT use cases where sensors may differ from device to device and sampling frequency from sensor to sensor.

The *table* model of time series maps naturally to a Parquet schema with one field per series, one field for the timestamp and maybe a set of fields for metadata such as labels, *i.e.* key/value pairs which add context to a series. This model is sometimes also called the *observations* model since each record is just that, a set of observations at a given timestamp.

The *series* model differs in that each series, or a chunk of time of a series, lies in a row. The columns that Parquet will create would then be the occurrences of the timestamps and values. This approach is described in [a chapter](http://what-when-how.com/Tutorial/topic-406tdru0g/Time-Series-Databases-32.html) of the [Time Series Book](https://apprize.best/data/series/4.html) by [Ted Dunning](https://twitter.com/ted_dunning) and [Ellen Friedman](https://twitter.com/Ellen_Friedman).

We see that in both cases the number of columns may explode if you want to store many series (in the table model) or many data points (in the series model). In this latter case the compression would also most probably be rather poor since a given column (the i*th* tick or i*th* value) would not have anything in common with the same tick in the next series.

### Performance

There are two performance aspects to consider when using Parquet files. The first one is the efficiency of the compression of the data. As we have just seen, the *table* model leads to better compression that the *series* approach.

The second performance aspect to consider is the speed at which data can be retrieved. As we already mentioned, Parquet has the ability to do *predicate pushdowns* where predicates on data are used to determine what data to read. The Parquet format does so at the *column chunk* + *row group* level and considers columns independently of each other. That is to say a predicate on a column will use the columns statistics to determine if a given row group contains valid data for the column given the predicate. If multiple predicates are pushed down, the row groups matching all predicates will be selected. This could seem to be a victory for performance, but unfortunately it is not completely so, as the only guarantee is that the selected row groups contain data which match the predicate for various columns, but there is no guarantee that the matching values for those columns belong to the same rows. You might therefore read many data which you will end up discarding. Luckily Parquet will drop records before reassembling them when a column ultimately does not verify a pushed down predicate, but that will not prevent you from paying for the transfer of those useless data.

Predicates on values are seldom used for time series, but predicates on timestamp and on metadata (labels) are very common. Imagine you have stored the following data in your Parquet file:

| row group | timestamp | room | house | temperature | humidity |
| --- | --- | --- | --- | --- | --- |
| 0 | 10:00:00Z | `kitchen` | `main` | 20.0 | 75.0 |
| 0 | 10:00:00Z | `kitchen` | `vacation` | 15.0 | 50.1 |
| 1 | 10:01:00Z | `bathroom` | `main` | 20.1 | 80.0 |
| 1 | 10:01:10Z | `den` | `vacation` | 17.0 | 60.0 |

Imagine that you want the data from your vacation house between 10:00:00Z and 10:01:00Z, you see that the row which matches is the second one which is in row group `0`, but as per the independence of the columns, both row groups will match as they contain timestamps within the requested range **AND** an occurrence of `vacation` in the *house* column. So you will end up reading the second row group even though it does not contain any data matching your criteria.

Mitigating this would require creation of custom row groups, ideally with a single series being stored per row group to guarantee that row group selection would not lead to extraneous data being read. This would create small row groups in most cases which is opposite to the purpose of row groups. This is rather cumbersome, but unfortunately this is the only solution as the Parquet format lacks provision for indexing the content of Parquet files for efficient random access to specific rows. Note also that predicates cannot be applied to nested structures, so grouping labels in a *MAP* column would not work for being more efficient at selecting matching rows.

Another issue we have briefly mentioned is the explosion in the number of columns. It is not wise to have too many columns in a Parquet file as it makes the footer grow quite consequently and some problems may arise with clients doing code generation. This constraints both the *tables* and *series* approaches to either a limited number of series, or a limited size of time chunks.

*Read also [Build a Complete Application with Warp 10, from TCP Stream to Dashboard](https://blog.senx.io/complete-application-warp-10-tcp-stream-dashboard/)*

## Conclusion

While Parquet is a very good format, when it comes to Time Series data the columnar approach it provides may not be very well suited and the access performance advertised by Parquet aficionados might just not be there in some contexts. The need for a fixed schema in a Parquet file also limits the variety of time series that can be stored in a single file. Bear that in mind when choosing Parquet or a tool using it for accessing time series data in an interactive way.

At SenX we have come to that conclusion several years ago and we designed a way of storing time series in files which lead to extreme compression and speed of access. Those files are readable in tools such as [Spark](https://spark.apache.org) and [Flink](https://flink.apache.org/) but more importantly can be mounted in a Warp 10 instance so the data they contain can be `FETCH`*ed* almost like if they were stored in the Warp 10 storage engine. Those files can also be stored in a cloud object storage service.

In some cases we have experienced compression ratios approaching 1:40, with each timestamp and value occupying less than 0.4 bytes, and read performance of hundreds of millions of data points per second on a single thread.

This approach, named *History File Stores*, is also very well suited for distributing data sets. For example a complete yearly dataset of [AIS data](https://blog.senx.io/ais-data-made-easy/) from [AISHub](https://www.aishub.net/) fits in a single file less than 100Gb, including the [spatio-temporal indexing](https://blog.senx.io/spatio-temporal-indexing-in-warp-10/) of the ships movements, this single file can be served by a Warp 10 instance and applications can interact with that data with blazing fast performance.

The History File Store format is currently available via our *Technology Preview* program, don't hesitate to [reach out](mailto:contact@senx.io) if you are interested.