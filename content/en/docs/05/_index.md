---
title: "5. Instrumenting with client libraries"
weight: 1
sectionnumber: 1
---

While an exporter is an adapter for your service to adapt a service specific value into a metric in the Prometheus format, it is also possible to export metric data programmatically in your application code.

## Client libraries

The Prometheus project provides [client libraries](https://prometheus.io/docs/instrumenting/clientlibs/) which are either official or maintained by third-parties. There are libraries for major languages like Java, Go, Python, PHP and even .NET/C#.

Even if you don't plan to provide your own metrics, those libraries already export some basic metrics based on the language. For [Java](https://github.com/prometheus/client_java#included-collectors), default metrics about memory management (Heap, garbage collection) and thread pools can be collected. The same applies to [Go](https://prometheus.io/docs/guides/go-application/).

{{% alert title="Note" color="primary" %}}

Just a short mention of the Spring Framework as it is very popular in application development. The framework also supports [exporting metrics](https://spring.io/blog/2018/03/16/micrometer-spring-boot-2-s-new-application-metrics-collector) in the Prometheus data format.
{{% /alert %}}

## Specifications and conventions

There are some guidelines and best practices how to name your own metrics. Of course, the [specifications of the datamodel](https://prometheus.io/docs/concepts/data_model/#metric-names-and-labels) must be followed and applying the [best practices about naming](https://prometheus.io/docs/practices/naming/) is not a bad idea. All those guidelines and best practices are now officially specified in [openmetrics.io](https://openmetrics.io).

Following these principles is not (yet) a must, but it helps to understand and interpret your metrics.

You can check your metrics by using the following `promtool` command: `curl -s http://localhost:8080/metrics | promtool check metrics`

## Best practices

Though implementing a metric is an easy task from a technical point of view, it is not so easy to define what and how to measure. If you follow your existing [log statements](https://prometheus.io/docs/practices/instrumentation/#logging) and if you define an error counter to count all [errors and exceptions](https://prometheus.io/docs/practices/instrumentation/#failures), then you already have a good base to see the internal state of your application.

### The four golden signals

Another approach to define metrics is based on [the four golden signals](https://sre.google/sre-book/monitoring-distributed-systems/):

* Latency
* Traffic
* Errors
* Saturation

There are other methods like [RED](https://www.weave.works/blog/the-red-method-key-metrics-for-microservices-architecture/) or [USE](http://www.brendangregg.com/usemethod.html) that go into the same direction.
