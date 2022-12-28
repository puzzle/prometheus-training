---
title: "2. Metrics"
weight: 1
sectionnumber: 1
---

## Prometheus exposition format

{{% alert title="Note" color="primary" %}}
Prometheus consumes metrics in Prometheus text-based exposition format and plans to adopt the [OpenMetrics](https://openmetrics.io/) standard: <https://prometheus.io/docs/introduction/roadmap/#adopt-openmetrics>.
{{% /alert %}}

[Prometheus Exposition Format](https://prometheus.io/docs/instrumenting/exposition_formats/)
```promql
# HELP <metric name> <info>
# TYPE <metric name> <metric type>
<metric name>{<label name>=<label value>, ...} <sample value>
```

As an example, check the metrics of your Prometheus server (<http://{{% param replacePlaceholder.prometheus %}}/metrics>).

{{% onlyWhenNot baloise %}}
```promql
...
# HELP prometheus_tsdb_head_samples_appended_total Total number of appended samples.
# TYPE prometheus_tsdb_head_samples_appended_total counter
prometheus_tsdb_head_samples_appended_total 463
# HELP prometheus_tsdb_head_series Total number of series in the head block.
# TYPE prometheus_tsdb_head_series gauge
prometheus_tsdb_head_series 463
...
```
{{% /onlyWhenNot %}}

{{% onlyWhen baloise %}}
```promql
...
# HELP prometheus_tsdb_head_min_time_seconds Minimum time bound of the head block.
# TYPE prometheus_tsdb_head_min_time_seconds gauge
prometheus_tsdb_head_min_time_seconds 1.669622401e+09
# HELP prometheus_tsdb_head_samples_appended_total Total number of appended samples.
# TYPE prometheus_tsdb_head_samples_appended_total counter
prometheus_tsdb_head_samples_appended_total 2.5110946e+07
...
```
{{% /onlyWhen %}}


{{% alert title="Note" color="primary" %}}
There are 4 different metric types in Prometheus

* Counter
* Gauge
* Histogram
* Summary

[Prometheus Metric Types](https://prometheus.io/docs/concepts/metric_types/)
{{% /alert %}}


## Explore Prometheus metrics


Open your Prometheus [web UI](http://{{% param replacePlaceholder.prometheus %}}) and navigate to the **Graph** menu. You can use the `Open metrics explorer` icon (next to the `Execute` button) to browse your metrics or start typing keywords in the expression field. Prometheus will try to find metrics that match your text.

Learn more about:

* [Prometheus operators](https://prometheus.io/docs/prometheus/latest/querying/operators/)
* [Prometheus functions](https://prometheus.io/docs/prometheus/latest/querying/functions/)
* [PromLens](https://promlens.com/), the power tool for querying Prometheus

## Recording Rules

Prometheus [recording rules](https://prometheus.io/docs/prometheus/latest/configuration/recording_rules/) allow you to precompute queries at a defined interval (`global.evaluation_interval` or `interval` in `rule_group`) and save them to a new set of time series.

## Special labels

As you have already seen in several examples, a Prometheus metric is defined by one or more labels with the corresponding values. Two of those labels are special, because the Prometheus server will automatically generate them for every metric:


* instance

     The instance label describes the endpoint where Prometheus scraped the metric. This can be any application or exporter. In addition to the ip address or hostname, this label usually also contains the port number. Example: `10.0.0.25:9100`

* job

     This label contains the name of the scrape job as configured in the Prometheus configuration file. All instances configured in the same scrape job will share the same job label.


{{% alert title="Note" color="primary" %}}
Prometheus will append these labels dynamically before sample ingestion. Therefore you will not see these labels if you query the metrics endpoint directly (e.g. by using `curl`).

{{% /alert %}}

Let's take a look at the following scrape config (example, no need to change the Prometheus configuration on your lab VM):

```yaml
...
scrape_configs:
  ...
  - job_name: "node_exporter"
    static_configs:
      - targets:
        - "10.0.0.25:9100"
        - "10.0.0.26:9100"
        - "10.0.0.27:9100"
  ...
```

In the example above we configured a single scrape job with the name `node_exporter` and three targets. After ingestion into Prometheus, every metric scraped by this job will have the label: `job="node_exporter"`. In addition, metrics scraped by this job from the target `10.0.0.25` will have the label `instance="10.0.0.25:9100"`
