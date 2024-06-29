<!--yml

category: 未分类

date: 2024-05-27 14:46:27

-->

# Javalin - 一个轻量级的Java和Kotlin Web框架

> 来源：[https://javalin.io/](https://javalin.io/)

## 简单的

与其他Java和Kotlin Web框架不同，Javalin需要学习的概念非常少。您从不扩展类，很少实现接口。

## 轻量级

Javalin只是在Jetty之上的几千行代码，其性能与原始的Jetty代码相当。由于其体积小，非常容易推理源代码。

## 可互操作的

其他Java和Kotlin Web框架通常为每种语言提供一个版本。Javalin考虑到互操作性，在Java和Kotlin中以相同的方式构建应用程序。

## 灵活的

Javalin被设计为简单且阻塞的，因为这是最容易推理的编程模型。但是，如果将`Future`设置为结果，Javalin将切换到异步模式。

## 教育性的

如果您正在教授Web编程并寻找一个能够脱离您的核心课程概念并让您专注于教学的Web框架，请访问我们的[教育者页面](/for-educators)。

## OpenAPI

许多轻量级的Java和Kotlin Web框架不支持OpenAPI，但Javalin支持（包括Swagger UI和ReDoc）。详细了解请访问[OpenAPI插件页面](/plugins/openapi)。

## Jetty

Javalin运行在JVM上使用最广泛且稳定的Web服务器之一Jetty之上。您可以完全配置Jetty服务器，包括SSL和HTTP3以及Jetty提供的所有其他功能。
