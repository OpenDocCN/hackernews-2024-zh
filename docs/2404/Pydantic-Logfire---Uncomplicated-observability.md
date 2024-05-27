<!--yml
category: 未分类
date: 2024-05-27 13:40:24
-->

# Pydantic Logfire | Uncomplicated observability

> 来源：[https://pydantic.dev/logfire](https://pydantic.dev/logfire)

## Structured Data & Direct SQL Access

Ensure your Python objects and structured data are query-ready. Use tools like Pandas, SQLAlchemy, or psql for querying, integrate seamlessly with BI software, and leverage AI for SQL generation.

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

## Manual Tracing

You can use the logfire library to create logs and traces directly — it’s kind of like standard logging in Python, with a more modern interface and some extra capabilities. And a lot less painful than using OpenTelemetry directly.

```
import logfire

logfire.info('Hello, {name}!', name='world')

advantages = 'timing', 'context', 'exception capturing' 
with logfire.span('spans provide: {advantages}', advantages=advantages):
     logfire.warn('the next line will raise an exception')
     1 / 0
```

## OpenTelemetry

[OpenTelemetry](https://opentelemetry.io/docs/what-is-opentelemetry/) (OTel) is an open source observability framework, it provides libraries for Python and every other popular language to let you collect traces, logs and metrics.

OTel is a powerful tool that increasing numbers of developers want to use, but it can be time-consuming to set up and limited in the kinds of data it can collect.

Pydantic Logfire takes the best of OTel (instrumentation for popular Python packages, open standard for data transmission) and makes it easier to use and more flexible.

By harnessing OpenTelemetry, Pydantic Logfire offers automatic instrumentation for popular Python packages, enables cross-language data integration, and supports data export to any OpenTelemetry-compatible backend or proxy.