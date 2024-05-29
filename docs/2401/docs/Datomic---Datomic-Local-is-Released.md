<!--yml
category: 未分类
date: 2024-05-27 14:58:45
-->

# Datomic - Datomic Local is Released

> 来源：[https://blog.datomic.com/2023/08/datomic-local-is-released.html](https://blog.datomic.com/2023/08/datomic-local-is-released.html)

*16 August 2023*

The Datomic team is excited to announce the launch of [Datomic Local](https://docs.datomic.com/cloud/datomic-local.html) under the [Apache License 2.0](https://www.apache.org/licenses/LICENSE-2.0.html). Datomic Local is a library that is both embeddable and redistributable. It is a lightweight version of Datomic, designed to be easily embedded within applications and freely redistributed. Datomic Local supports the Client API, making it an excellent choice for small single-process applications and for testing systems that are based on the Datomic Client API.

### What is new?

Datomic Local is a lightweight version of Datomic backed by local storage that supports the [Client API](https://docs.datomic.com/cloud/client/client-api.html). The Datomic Local and supporting binaries are released under the [Apache License 2.0](https://www.apache.org/licenses/LICENSE-2.0.html) license.

### Use cases

The Datomic Local version of Datomic is indicated for:

*   People who want to write libraries that depend on Datomic

*   People who want an embedded DB

*   Cloud users who want to dev locally

*   People who already use dev-local

# FAQ

### What is Datomic Local?

Datomic Local is a lightweight version of Datomic backed by local storage which supports the Client API.

### What changed?

Datomic Local is a library, embeddable, and redistributable. Datomic Local replaces dev-local.

### How to get Datomic Local?

### What changes in terms of licensing?

The Datomic Local binary is available under the [Apache 2.0 License](https://www.apache.org/licenses/LICENSE-2.0.html), which means that it is possible to redistribute it.

### For whom is Datomic Local recommended?

*   People who want to write libraries that depend on Datomic

*   People who want an embedded DB

*   Cloud users who want to dev locally

*   People who already use dev-local

### What happened to Datomic Free?

If you are used to using Datomic Free, use [Datomic Pro](https://www.datomic.com/). If you want a redistributable, embeddable Datomic, use Datomic Local.

### What is the difference between Datomic Cloud, Datomic Local, and Datomic Pro?

[**Datomic Cloud**](https://docs.datomic.com/cloud/index.html) is tightly integrated with AWS. It supports only client access. Datomic Cloud has ions, which allows users to run entire applications on Datomic, with reproducible deployment, elastic autoscaling, and integration via AWS Lambda events and AWS API Gateway.

[**Datomic Pro**](https://docs.datomic.com/pro/overview/introduction.html) is a distributed database designed to enable scalable, flexible applications running on-premises or in the cloud. It is fully designed for use against multiple storages in large-scale, multi-user systems.

[**Datomic Local**](https://docs.datomic.com/cloud/datomic-local.html) is a lightweight version of Datomic that is embeddable, redistributable, and supports the Client API. It is well suited for single-process applications and for testing Client API-based Datomic systems.

### Is Datomic Local open source?

No. The *binaries* are Apache 2.0, as with other versions of Datomic.

* * *