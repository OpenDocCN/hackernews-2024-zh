<!--yml

category: 未分类

date: 2024-05-27 13:40:24

-->

# Pydantic Logfire | 简单的可观测性

> 来源：[https://pydantic.dev/logfire](https://pydantic.dev/logfire)

## 结构化数据与直接 SQL 访问

确保您的 Python 对象和结构化数据可以进行查询。使用像 Pandas、SQLAlchemy 或 psql 这样的工具进行查询，与 BI 软件无缝集成，并利用 AI 进行 SQL 生成。

```
`SELECT 
  attributes->>'campaign' as campaign, 
  count(distinct attributes->>'track_id') as clicks, 
  round(count(distinct attributes->>'track_id')::numeric/50*100, 2) as click_rate
FROM records
WHERE
  message ilike 'click%' and 
  attributes->>'campaign' ilike 'c%'
GROUP BY attributes->>'campaign'`
```

## 手动跟踪

您可以使用 logfire 库直接创建日志和跟踪 —— 这有点像 Python 中的标准日志记录，但具有更现代的界面和一些额外的功能。比直接使用 OpenTelemetry 要简单得多。

```
import logfire

logfire.info('Hello, {name}!', name='world')

advantages = 'timing', 'context', 'exception capturing' 
with logfire.span('spans provide: {advantages}', advantages=advantages):
     logfire.warn('the next line will raise an exception')
     1 / 0
```

## OpenTelemetry

[OpenTelemetry](https://opentelemetry.io/docs/what-is-opentelemetry/)（OTel）是一个开源的可观测性框架，它提供了 Python 和其他流行语言的库，让您可以收集跟踪、日志和指标。

OTel 是一种强大的工具，越来越多的开发人员希望使用，但设置起来可能耗时，并且在可以收集的数据种类方面存在限制。

Pydantic Logfire 继承了 OTel 的优点（流行 Python 包的仪器化，数据传输的开放标准），使其更易于使用和更灵活。

通过利用 OpenTelemetry，Pydantic Logfire 提供了流行 Python 包的自动仪器化，支持跨语言数据集成，并支持数据导出到任何兼容 OpenTelemetry 的后端或代理。
