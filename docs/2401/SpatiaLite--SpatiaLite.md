<!--yml
category: 未分类
date: 2024-05-27 15:16:57
-->

# SpatiaLite: SpatiaLite

> 来源：[https://www.gaia-gis.it/fossil/libspatialite/index](https://www.gaia-gis.it/fossil/libspatialite/index)

SpatiaLite is an **open source** library intended to extend the [SQLite](http://www.sqlite.org/) core to support fully fledged Spatial SQL capabilities.
SQLite is intrinsically simple and lightweight:

*   a single lightweight library implementing the full SQL engine
*   standard SQL implementation: almost complete SQL-92
*   no complex client/server architecture
*   a whole database simply corresponds to a single monolithic file (no size limits)
*   any DB-file can be safely exchanged across different platforms, because the internal architecture is universally portable
*   no installation, no configuration

SpatiaLite is smoothly integrated into SQLite to provide a complete and powerful Spatial DBMS (mostly OGC-SFS compliant).
Using SQLite + SpatiaLite you can effectively deploy an alternative *open source* Spatial DBMS roughly equivalent to [PostgreSQL](http://www.postgresql.org/) + [PostGIS](http://postgis.refractions.net/).

SpatiaLite is licensed under the [MPL tri-license](http://www.mozilla.org/MPL/boilerplate-1.1/mpl-tri-license-html) terms; you are free to choose the best-fit license between: