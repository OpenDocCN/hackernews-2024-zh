<!--yml
category: 未分类
date: 2024-05-27 14:42:29
-->

# Founding Engineer at Nimbus | Y Combinator

> 来源：[https://www.ycombinator.com/companies/nimbus-3/jobs/TgQFIkz-founding-engineer](https://www.ycombinator.com/companies/nimbus-3/jobs/TgQFIkz-founding-engineer)

Nimbus is an intelligent control plane for modern observability. Our mission is to enable FAANG-like observability for every organization in the world. We're starting with an optimization engine to reduce datadog costs by an order of magnitude without manual effort from developers.

How it works: Update three lines in your datadog agent configuration to send data to Nimbus. Nimbus analyzes all incoming logs for common patterns while proxying data upstream. Based on the analysis, Nimbus comes up with optimizations that can be reviewed and deployed by a human in one click. We employ novel compression techniques to reduce billable volume by an order of magnitude without dropping data. You can watch a demo of this at [nimbus.dev](https://nimbus.dev/?utm_source=hm)

We're looking to hire our first founding engineer to help us expand beyond logging. Nimbus today can reduce log volume by over 90% without losing signal - our immediate priorities is to do the same for metrics and traces.

You'll be coming in at the ground level and working directly with our founder at all levels of the stack. We are growing quickly, have multiple years of runway, and expect to be profitable middle of this year.

## What You'll Do

You'll have autonomy to work across a wide set of challenging problem. A few things you might work on:

*   Derive new algorithms to optimize observability data without losing signal
*   Build a highly performant, always available, multi-cloud multi-tenant observability pipeline
*   Make low level tweaks in rust to speed up pipeline performance
*   Design and expand our [custom query language](https://docs.nimbus.dev/overview/ntl)
*   Collaborate with open source working groups on observability standards
*   Create load tests to benchmark pipeline performance across different network topologies and compute nodes
*   Help build out a team of world-class, highly collaborative, software engineers

## Who You Are

*   You are an exceptional senior engineer (6+ years of industry experience) who communicates clearly and has a strong desire to improve
*   You are comfortable working at all levels of the stack - from capturing wire traffic using wireshark to deploying CRUD apps
*   You have deep knowledge of working with AWS, Docker, and Kubernetes
*   You have deep proficiency in at least one of the following languages: Typescript, Python, Rust

## Bonus

*   have worked in the observability space
*   familiar with Datadog's infrastructure, APM, and logging services
*   deep understanding of observability pipeline (ideally vector)
*   deep understanding of prometheus and grafana
*   worked with the CDK

## Our Stack

### Code

*   Typescript for Backend (Express) and Frontend (React, Next, & Material)
*   Rust for the pipeline agent (specifically, [vector](https://vector.dev/))
*   Python for ML and Analytics Engine

### Infra

*   AWS
*   Kubernetes (EKS)
*   Grafana & Prometheus for monitoring
*   CDK and Kubectl for IaC