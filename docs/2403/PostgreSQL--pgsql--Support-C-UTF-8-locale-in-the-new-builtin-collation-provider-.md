<!--yml

类别：未分类

日期：2024-05-29 12:31:40

-->

# PostgreSQL：pgsql：在新的内置排序提供程序中支持C.UTF-8语言环境。

> 来源：[https://www.postgresql.org/message-id/E1rmhxt-004ezT-OB%40gemulon.postgresql.org](https://www.postgresql.org/message-id/E1rmhxt-004ezT-OB%40gemulon.postgresql.org)

在新的内置排序提供程序中支持C.UTF-8语言环境。

内置的C.UTF-8语言环境与libc的"C"语言环境类似。

相同的名称。也就是说，码点排序顺序（快速，基于memcmp）

结合Unicode语义进行字符操作，例如

模式匹配，正则表达式和

LOWER()/INITCAP()/UPPER()。字符语义基于

Unicode简单大小写映射。

内置提供程序的C.UTF-8提供了几个重要的优势

通过libc：

* 更快的排序--受益于诸如

简化键和varstrfastcmp_c

* 更快的大小写转换，例如LOWER()，至少与某些情况相比

libc实现

* 在所有平台上都具有相同的语义，并且

语义在给定的范围内是稳定的，可测试的，并且可以文档化

PostgreSQL主要版本

基于memcmp，内置的C.UTF-8语言环境不提供

内置的C.UTF-8语言环境在大多数使用情况下都是一种改进

否则可能使用libc的"C.UTF-8"语言环境的情况

许多使用libc的"C.UTF-8"语言环境的用例，以及

讨论：[https://postgr.es/m/ff4c2f2f9c8fc7ca27c1c24ae37ecaeaeaff6b53.camel%40j-davis.com](https://postgr.es/m/ff4c2f2f9c8fc7ca27c1c24ae37ecaeaeaff6b53.camel@j-davis.com)

评论：Daniel Vérité、Peter Eisentraut、Jeremy Schneider

分支

------

master

细节

-------

[https://git.postgresql.org/pg/commitdiff/f69319f2f1fb16eda4b535bcccec90dff3a6795e](https://git.postgresql.org/pg/commitdiff/f69319f2f1fb16eda4b535bcccec90dff3a6795e)

修改的文件

--------------

doc/src/sgml/charset.sgml | 27 +++++-

doc/src/sgml/ref/create_collation.sgml | 2 +-

doc/src/sgml/ref/create_database.sgml | 13 ++-

doc/src/sgml/ref/initdb.sgml | 5 +-

src/backend/regex/regc_pg_locale.c | 36 ++++++-

src/backend/utils/adt/formatting.c | 112 ++++++++++++++++++++++

src/backend/utils/adt/pg_locale.c | 52 +++++++---

src/bin/initdb/initdb.c | 16 +++-

src/bin/initdb/t/001_initdb.pl | 17 ++++

src/bin/pg_upgrade/t/002_pg_upgrade.pl | 2 +-

src/bin/scripts/t/020_createdb.pl | 18 ++++

src/include/catalog/catversion.h | 2 +-

src/include/catalog/pg_collation.dat | 3 +

src/test/regress/expected/collate.utf8.out | 136 +++++++++++++++++++++++++++

src/test/regress/expected/collate.utf8_1.out | 8 ++

src/test/regress/parallel_schedule | 4 +-

src/test/regress/sql/collate.utf8.sql | 67 +++++++++++++

更改了17个文件，增加了494个插入(+)，删除了26个(-)
