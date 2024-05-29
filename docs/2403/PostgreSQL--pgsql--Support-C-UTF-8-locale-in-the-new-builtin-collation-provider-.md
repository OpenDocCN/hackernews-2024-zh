<!--yml
category: 未分类
date: 2024-05-29 12:31:40
-->

# PostgreSQL: pgsql: Support C.UTF-8 locale in the new builtin collation provider.

> 来源：[https://www.postgresql.org/message-id/E1rmhxt-004ezT-OB%40gemulon.postgresql.org](https://www.postgresql.org/message-id/E1rmhxt-004ezT-OB%40gemulon.postgresql.org)

Support C.UTF-8 locale in the new builtin collation provider.

The builtin C.UTF-8 locale has similar semantics to the libc locale of
the same name. That is, code point sort order (fast, memcmp-based)
combined with Unicode semantics for character operations such as
pattern matching, regular expressions, and
LOWER()/INITCAP()/UPPER(). The character semantics are based on
Unicode simple case mappings.

The builtin provider's C.UTF-8 offers several important advantages
over libc:

* faster sorting -- benefits from additional optimizations such as
abbreviated keys and varstrfastcmp_c
* faster case conversion, e.g. LOWER(), at least compared with some
libc implementations
* available on all platforms with identical semantics, and the
semantics are stable, testable, and documentable within a given
Postgres major version

Being based on memcmp, the builtin C.UTF-8 locale does not offer
natural language sort order. But it is an improvement for most use
cases that might otherwise use libc's "C.UTF-8" locale, as well as
many use cases that use libc's "C" locale.

Discussion: [https://postgr.es/m/ff4c2f2f9c8fc7ca27c1c24ae37ecaeaeaff6b53.camel%40j-davis.com](https://postgr.es/m/ff4c2f2f9c8fc7ca27c1c24ae37ecaeaeaff6b53.camel@j-davis.com)
Reviewed-by: Daniel Vérité, Peter Eisentraut, Jeremy Schneider

Branch
------
master

Details
-------
[https://git.postgresql.org/pg/commitdiff/f69319f2f1fb16eda4b535bcccec90dff3a6795e](https://git.postgresql.org/pg/commitdiff/f69319f2f1fb16eda4b535bcccec90dff3a6795e)

Modified Files
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
17 files changed, 494 insertions(+), 26 deletions(-)