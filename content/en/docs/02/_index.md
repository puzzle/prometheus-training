---
title: "2 Prometheus Basics Training - Metrics"
weight: 1
sectionnumber: 1
---

## Prometheus exposition format

{{% alert title="Note" color="primary" %}}
Prometheus consumes metrics in Prometheus text-based exposition format and plans to adopt the [OpenMetrics](https://openmetrics.io/) standard. <https://prometheus.io/docs/introduction/roadmap/#adopt-openmetrics>
{{% /alert %}}

[Prometheus Exposition Format](https://prometheus.io/docs/instrumenting/exposition_formats/)
```
metric_name [
  "{" label_name "=" `"` label_value `"` { "," label_name "=" `"` label_value `"` } [ "," ] "}"
] value [ timestamp ]
```

As an example check the metrics of your Prometheus server (<http://127.0.0.1:9090/metrics>)
```
...
# HELP prometheus_tsdb_head_samples_appended_total Total number of appended samples.
# TYPE prometheus_tsdb_head_samples_appended_total counter
prometheus_tsdb_head_samples_appended_total 463
# HELP prometheus_tsdb_head_series Total number of series in the head block.
# TYPE prometheus_tsdb_head_series gauge
prometheus_tsdb_head_series 463
....
```

{{% alert title="Note" color="primary" %}}
There are 4 different Metric types in Prometheus

* Counter
* Gauge
* Histogram
* Summary

[Prometheus Metric Types](https://prometheus.io/docs/concepts/metric_types/)
{{% /alert %}}


## Explore Prometheus metrics

Open your Prometheus Server and navigate to the `graph` menu. You can use the drop-down list to browse your metrics or start typing keywords in the expression field. Prometheus will try to find metrics that match your text.

* Using [Prithmetic Operators](https://prometheus.io/docs/prometheus/latest/querying/operators/)
* Using [Prometheus Functions](https://prometheus.io/docs/prometheus/latest/querying/functions/)

## Recording Rules

Prometheus [Recording Rules](https://prometheus.io/docs/prometheus/latest/configuration/recording_rules/) allow you to precompute queries at a defined interval (`global.evaluation_interval` or `interval` in  <rule_group> ) and save them to a new set of time series.
