<!--yml
category: 未分类
date: 2024-05-27 14:48:15
-->

# Releases · uktrade/pg-bulk-ingest · GitHub

> 来源：[https://github.com/uktrade/pg-bulk-ingest/releases](https://github.com/uktrade/pg-bulk-ingest/releases)

# Releases: uktrade/pg-bulk-ingest

Releases · uktrade/pg-bulk-ingest

## v0.0.46

## What's Changed

The main change in this release is the addition of support of ingesting to multiple tables per call to `ingest`, while being as stream-y as possible, and also in a single transaction per batch 🎉

*   feat: support multiple tables by [@michalc](https://github.com/michalc) in [#176](https://github.com/uktrade/pg-bulk-ingest/pull/176)
*   docs: avoid hinting that only a single table is supported by [@michalc](https://github.com/michalc) in [#178](https://github.com/uktrade/pg-bulk-ingest/pull/178)

**Full Changelog**: [`v0.0.45...v0.0.46`](https://github.com/uktrade/pg-bulk-ingest/compare/v0.0.45...v0.0.46)

## v0.0.45

## What's Changed

*   docs: add getting started with dagster section by [@JosefSmith](https://github.com/JosefSmith) in [#165](https://github.com/uktrade/pg-bulk-ingest/pull/165)
*   docs: adding some basic docs about using high watermarks by [@JosefSmith](https://github.com/JosefSmith) in [#166](https://github.com/uktrade/pg-bulk-ingest/pull/166)
*   build(deps): move to govuk-eleventy-plugin v6.0.3 by [@michalc](https://github.com/michalc) in [#167](https://github.com/uktrade/pg-bulk-ingest/pull/167)
*   docs: make logo a bit tighter by [@michalc](https://github.com/michalc) in [#168](https://github.com/uktrade/pg-bulk-ingest/pull/168)
*   build(deps): pin localtunnel to avoid axios vulnerability by [@niross](https://github.com/niross) in [#169](https://github.com/uktrade/pg-bulk-ingest/pull/169)
*   build(deps): add rollup-linux-x64-gnu as optional dependency by [@michalc](https://github.com/michalc) in [#170](https://github.com/uktrade/pg-bulk-ingest/pull/170)
*   build(deps): fix package-lock.json by [@michalc](https://github.com/michalc) in [#171](https://github.com/uktrade/pg-bulk-ingest/pull/171)
*   build(deps): fix package-lock.json again by [@michalc](https://github.com/michalc) in [#172](https://github.com/uktrade/pg-bulk-ingest/pull/172)
*   build(deps): fix package-lock.json for the third time by [@michalc](https://github.com/michalc) in [#173](https://github.com/uktrade/pg-bulk-ingest/pull/173)
*   build(deps): avoid axios vulnrability (again) by [@michalc](https://github.com/michalc) in [#174](https://github.com/uktrade/pg-bulk-ingest/pull/174)
*   refactor: use to-file-like-obj by [@michalc](https://github.com/michalc) in [#175](https://github.com/uktrade/pg-bulk-ingest/pull/175)

## New Contributors

**Full Changelog**: [`v0.0.44...v0.0.45`](https://github.com/uktrade/pg-bulk-ingest/compare/v0.0.44...v0.0.45)

## v0.0.43

## What's Changed

*   docs: add Prerequisites to Airflow guide by [@michalc](https://github.com/michalc) in [#145](https://github.com/uktrade/pg-bulk-ingest/pull/145)
*   docs: better setup instructions by [@michalc](https://github.com/michalc) in [#146](https://github.com/uktrade/pg-bulk-ingest/pull/146)
*   docs: tweak example instructions to be more consistent with Airflow instructions by [@michalc](https://github.com/michalc) in [#147](https://github.com/uktrade/pg-bulk-ingest/pull/147)
*   docs: improve flow and mention that the environment variable for the database will likely be different by [@michalc](https://github.com/michalc) in [#148](https://github.com/uktrade/pg-bulk-ingest/pull/148)
*   docs: tweak order, putting Airflow higher up by [@michalc](https://github.com/michalc) in [#149](https://github.com/uktrade/pg-bulk-ingest/pull/149)
*   docs: use DAG more the pipeline when talking about Airflow by [@michalc](https://github.com/michalc) in [#150](https://github.com/uktrade/pg-bulk-ingest/pull/150)
*   docs: slightly more concise by [@michalc](https://github.com/michalc) in [#151](https://github.com/uktrade/pg-bulk-ingest/pull/151)
*   tests: test for larger amounts of data by [@michalc](https://github.com/michalc) in [#152](https://github.com/uktrade/pg-bulk-ingest/pull/152)
*   refactor: comment out dead code (but don't remove it) by [@michalc](https://github.com/michalc) in [#153](https://github.com/uktrade/pg-bulk-ingest/pull/153)
*   tests: test that assert that multiple tables, for now, are not supported by [@michalc](https://github.com/michalc) in [#154](https://github.com/uktrade/pg-bulk-ingest/pull/154)
*   refactor: reduce a small amount of duplication in tests by [@michalc](https://github.com/michalc) in [#155](https://github.com/uktrade/pg-bulk-ingest/pull/155)
*   docs: more features on the front page by [@michalc](https://github.com/michalc) in [#156](https://github.com/uktrade/pg-bulk-ingest/pull/156)
*   docs: more features on front page by [@michalc](https://github.com/michalc) in [#157](https://github.com/uktrade/pg-bulk-ingest/pull/157)
*   docs: split up the get started page to make it a bit less scary by [@michalc](https://github.com/michalc) in [#158](https://github.com/uktrade/pg-bulk-ingest/pull/158)
*   docs: link to PostgreSQL docs by [@michalc](https://github.com/michalc) in [#159](https://github.com/uktrade/pg-bulk-ingest/pull/159)
*   docs: make it clearer that the Airflow guide is not directly after the regular Get started guide by [@michalc](https://github.com/michalc) in [#160](https://github.com/uktrade/pg-bulk-ingest/pull/160)
*   docs: fix missing space between words by [@michalc](https://github.com/michalc) in [#161](https://github.com/uktrade/pg-bulk-ingest/pull/161)
*   docs: fix link and improve clarity around the create_dag function by [@michalc](https://github.com/michalc) in [#162](https://github.com/uktrade/pg-bulk-ingest/pull/162)
*   deps: bump eleventy plugin and associated packages by [@michalc](https://github.com/michalc) in [#163](https://github.com/uktrade/pg-bulk-ingest/pull/163)
*   feat: add to_file_like_obj utility function to pg-bulk-ingest by [@JosefSmith](https://github.com/JosefSmith) in [#143](https://github.com/uktrade/pg-bulk-ingest/pull/143)

**Full Changelog**: [`v0.0.42...v0.0.43`](https://github.com/uktrade/pg-bulk-ingest/compare/v0.0.42...v0.0.43)