<!--yml

类别：未分类

日期：2024-05-27 14:48:15

-->

# 发布 · uktrade/pg-bulk-ingest · GitHub

> 来源：[https://github.com/uktrade/pg-bulk-ingest/releases](https://github.com/uktrade/pg-bulk-ingest/releases)

# 发布：uktrade/pg-bulk-ingest

发布 · uktrade/pg-bulk-ingest

## v0.0.46

## 发生了什么变化

本次发布的主要更改是增加了对单批次每个表一次事务的流处理支持，使其尽可能记忆深刻，而且是在一次事务中完成的 🎉

+   特性：支持一次调用中对多个表进行摄入的新增功能，由[@michalc](https://github.com/michalc)在[#176](https://github.com/uktrade/pg-bulk-ingest/pull/176)中提交

+   文档：避免暗示仅支持单个表，由[@michalc](https://github.com/michalc)在[#178](https://github.com/uktrade/pg-bulk-ingest/pull/178)中提交

**完整变更记录**：[`v0.0.45...v0.0.46`](https://github.com/uktrade/pg-bulk-ingest/compare/v0.0.45...v0.0.46)

## v0.0.45

## 发生了什么变化

+   文档：添加有关使用dagster的入门部分，由[@JosefSmith](https://github.com/JosefSmith)在[#165](https://github.com/uktrade/pg-bulk-ingest/pull/165)中提交

+   文档：添加关于使用高水位标记的基本文档，由[@JosefSmith](https://github.com/JosefSmith)在[#166](https://github.com/uktrade/pg-bulk-ingest/pull/166)中提交

+   构建（依赖）：移至govuk-eleventy-plugin v6.0.3，由[@michalc](https://github.com/michalc)在[#167](https://github.com/uktrade/pg-bulk-ingest/pull/167)中提交

+   文档：使标志更加紧凑，由[@michalc](https://github.com/michalc)在[#168](https://github.com/uktrade/pg-bulk-ingest/pull/168)中提交

+   构建（依赖）：固定本地隧道以避免axios漏洞，由[@niross](https://github.com/niross)在[#169](https://github.com/uktrade/pg-bulk-ingest/pull/169)中提交

+   构建（依赖）：将rollup-linux-x64-gnu添加为可选依赖项，由[@michalc](https://github.com/michalc)在[#170](https://github.com/uktrade/pg-bulk-ingest/pull/170)中提交

+   构建（依赖）：修复package-lock.json，由[@michalc](https://github.com/michalc)在[#171](https://github.com/uktrade/pg-bulk-ingest/pull/171)中提交

+   构建（依赖）：再次修复package-lock.json，由[@michalc](https://github.com/michalc)在[#172](https://github.com/uktrade/pg-bulk-ingest/pull/172)中提交

+   构建（依赖）：第三次修复package-lock.json，由[@michalc](https://github.com/michalc)在[#173](https://github.com/uktrade/pg-bulk-ingest/pull/173)中提交

+   构建（依赖）：避免axios漏洞（再次），由[@michalc](https://github.com/michalc)在[#174](https://github.com/uktrade/pg-bulk-ingest/pull/174)中提交

+   重构：使用类似文件对象的to-file-like-obj，由[@michalc](https://github.com/michalc)在[#175](https://github.com/uktrade/pg-bulk-ingest/pull/175)中提交

## 新贡献者

**完整变更记录**：[`v0.0.44...v0.0.45`](https://github.com/uktrade/pg-bulk-ingest/compare/v0.0.44...v0.0.45)

## v0.0.43

## 发生了什么变化

+   文档：在Airflow指南中添加前提条件，由[@michalc](https://github.com/michalc)在[#145](https://github.com/uktrade/pg-bulk-ingest/pull/145)中提交

+   文档：更好的设置说明，来自[@michalc](https://github.com/michalc)的[#146](https://github.com/uktrade/pg-bulk-ingest/pull/146)

+   文档：调整示例说明，使其与Airflow指南更一致，来自[@michalc](https://github.com/michalc)的[#147](https://github.com/uktrade/pg-bulk-ingest/pull/147)

+   文档：改善流程并提到数据库的环境变量可能会不同，来自[@michalc](https://github.com/michalc)的[#148](https://github.com/uktrade/pg-bulk-ingest/pull/148)

+   文档：调整顺序，将Airflow放在更靠前的位置，来自[@michalc](https://github.com/michalc)的[#149](https://github.com/uktrade/pg-bulk-ingest/pull/149)

+   文档：在讨论Airflow时更多地使用DAG而不是pipeline，来自[@michalc](https://github.com/michalc)的[#150](https://github.com/uktrade/pg-bulk-ingest/pull/150)

+   文档：稍微简洁一些，来自[@michalc](https://github.com/michalc)的[#151](https://github.com/uktrade/pg-bulk-ingest/pull/151)

+   测试：测试更大量数据，来自[@michalc](https://github.com/michalc)的[#152](https://github.com/uktrade/pg-bulk-ingest/pull/152)

+   重构：注释掉死代码（但不要删除它），来自[@michalc](https://github.com/michalc)的[#153](https://github.com/uktrade/pg-bulk-ingest/pull/153)

+   测试：测试断言目前不支持多表格，来自[@michalc](https://github.com/michalc)的[#154](https://github.com/uktrade/pg-bulk-ingest/pull/154)

+   重构：减少测试中的少量重复代码，来自[@michalc](https://github.com/michalc)的[#155](https://github.com/uktrade/pg-bulk-ingest/pull/155)

+   文档：在首页上添加更多功能特性，来自[@michalc](https://github.com/michalc)的[#156](https://github.com/uktrade/pg-bulk-ingest/pull/156)

+   文档：在首页上添加更多功能特性，来自[@michalc](https://github.com/michalc)的[#157](https://github.com/uktrade/pg-bulk-ingest/pull/157)

+   文档：拆分入门页面，使其看起来不那么吓人，来自[@michalc](https://github.com/michalc)的[#158](https://github.com/uktrade/pg-bulk-ingest/pull/158)

+   文档：链接到PostgreSQL文档，来自[@michalc](https://github.com/michalc)的[#159](https://github.com/uktrade/pg-bulk-ingest/pull/159)

+   文档：更清楚地表明Airflow指南不是紧接着常规的入门指南，来自[@michalc](https://github.com/michalc)的[#160](https://github.com/uktrade/pg-bulk-ingest/pull/160)

+   文档：修复单词之间缺失的空格，来自[@michalc](https://github.com/michalc)的[#161](https://github.com/uktrade/pg-bulk-ingest/pull/161)

+   文档：修复链接并改善create_dag函数周围的清晰度，来自[@michalc](https://github.com/michalc)的[#162](https://github.com/uktrade/pg-bulk-ingest/pull/162)

+   依赖：升级eleventy插件及相关包，来自[@michalc](https://github.com/michalc)的[#163](https://github.com/uktrade/pg-bulk-ingest/pull/163)

+   feat: 添加了由[@JosefSmith](https://github.com/JosefSmith)贡献的`to_file_like_obj`实用函数到[#143](https://github.com/uktrade/pg-bulk-ingest/pull/143)

**完整变更日志**: [`v0.0.42...v0.0.43`](https://github.com/uktrade/pg-bulk-ingest/compare/v0.0.42...v0.0.43)
