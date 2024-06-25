<!--yml

类别：未分类

日期：2024年05月27日 15:16:57

-->

# SpatiaLite: SpatiaLite

> 来源：[https://www.gaia-gis.it/fossil/libspatialite/index](https://www.gaia-gis.it/fossil/libspatialite/index)

SpatiaLite 是一个 **开源** 库，旨在扩展 [SQLite](http://www.sqlite.org/) 核心以支持完整的空间 SQL 能力。

SQLite 本质上简单且轻量级：

+   单个轻量级库实现完整的 SQL 引擎

+   标准 SQL 实现：几乎完全符合 SQL-92

+   没有复杂的客户端/服务器架构

+   整个数据库简单地对应于单个庞大的文件（没有大小限制）

+   任何数据库文件都可以安全地在不同平台之间交换，因为内部架构是普遍可移植的

+   无需安装，无需配置

SpatiaLite 被平滑地整合到 SQLite 中，提供完整而强大的空间数据库管理系统（大部分符合 OGC-SFS 标准）。

使用 SQLite + SpatiaLite，你可以有效地部署一种替代的 *开源* 空间数据库管理系统，大致相当于 [PostgreSQL](http://www.postgresql.org/) + [PostGIS](http://postgis.refractions.net/)。

SpatiaLite 根据 [MPL tri-license](http://www.mozilla.org/MPL/boilerplate-1.1/mpl-tri-license-html) 条款授权；你可以在以下最适合的许可证中自由选择：
