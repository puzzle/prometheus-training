---
title: "5. Instrumenting with client libraries
weight: 1
sectionnumber: 1
---

## Instrumenting your application code

Intro: explain difference to exporters

### Client libraries

* mention some libraries
* some libraries provide basic metrics (Java/Go: Heap, Threads/goroutines etc)

### Spring Framework

* When using spring framework, only

### Conventions and specification

* how to name metrics
* check metrics with `curl -s http://localhost:8080/metrics/invalid | promtool check metrics`

* https://prometheus.io/docs/concepts/data_model/#metric-names-and-labels
* https://prometheus.io/docs/practices/naming/

## The Four Golden Signals

https://sre.google/sre-book/monitoring-distributed-systems/
https://www.weave.works/blog/the-red-method-key-metrics-for-microservices-architecture/
http://www.brendangregg.com/usemethod.html
