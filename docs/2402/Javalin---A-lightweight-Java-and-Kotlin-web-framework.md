<!--yml
category: 未分类
date: 2024-05-27 14:46:27
-->

# Javalin - A lightweight Java and Kotlin web framework

> 来源：[https://javalin.io/](https://javalin.io/)

## Simple

Unlike other Java and Kotlin web frameworks, Javalin has very few concepts that you need to learn. You never extend classes and you rarely implement interfaces.

## Lightweight

Javalin is just a few thousand lines of code on top of Jetty, and its performance is equivalent to raw Jetty code. Due to its size, it's very easy to reason about the source code.

## Interoperable

Other Java and Kotlin web frameworks usually offer one version for each language. Javalin is being made with inter-operability in mind, apps are built the same way in both Java and Kotlin.

## Flexible

Javalin is designed to be simple and blocking, as this is the easiest programming model to reason about. But, if you set a `Future` as a result, Javalin switches into asynchronous mode.

## Educational

Visit our [educators page](/for-educators) if you're teaching web programming and looking for a web framework which will get out of your way and let you focus on the core concepts of your curriculum.

## OpenAPI

Many lightweight Java and Kotlin web frameworks don't support OpenAPI, but Javalin does (including Swagger UI and ReDoc). Learn more at the [OpenAPI plugin page](/plugins/openapi).

## Jetty

Javalin runs on top of Jetty, one of the most used and stable web-servers on the JVM. You can configure the Jetty server fully, including SSL and HTTP3 and everything else that Jetty offers.