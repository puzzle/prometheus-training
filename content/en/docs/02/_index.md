---
title: "2. Query and Visualize"
weight: 1
sectionnumber: 2
---

## Prometheus exposition format

{{% alert title="Note" color="primary" %}}
Prometheus consumes metrics in Prometheus text-based exposition format and plans to adopt the [OpenMetrics](https://openmetrics.io/) standard: <https://prometheus.io/docs/introduction/roadmap/#adopt-openmetrics>.
{{% /alert %}}

[Prometheus Exposition Format](https://prometheus.io/docs/instrumenting/exposition_formats/)
```
metric_name [
  "{" label_name "=" `"` label_value `"` { "," label_name "=" `"` label_value `"` } [ "," ] "}"
] value [ timestamp ]
```

As an example, check the metrics of your Prometheus server (<http://LOCALHOST:19090/metrics>).
```
...
# HELP prometheus_tsdb_head_samples_appended_total Total number of appended samples.
# TYPE prometheus_tsdb_head_samples_appended_total counter
prometheus_tsdb_head_samples_appended_total 463
# HELP prometheus_tsdb_head_series Total number of series in the head block.
# TYPE prometheus_tsdb_head_series gauge
prometheus_tsdb_head_series 463
...
```

{{% alert title="Note" color="primary" %}}
There are 4 different metric types in Prometheus

* Counter
* Gauge
* Histogram
* Summary

[Prometheus Metric Types](https://prometheus.io/docs/concepts/metric_types/)
{{% /alert %}}


## Explore Prometheus metrics

Open your Prometheus [web UI](http://LOCALHOST:19090) and navigate to the **Graph** menu. You can use the `insert metric at cursor` drop-down list (next to the `Execute` button) to browse your metrics or start typing keywords in the expression field. Prometheus will try to find metrics that match your text.

Learn more about:

* [Prometheus operators](https://prometheus.io/docs/prometheus/latest/querying/operators/)
* [Prometheus functions](https://prometheus.io/docs/prometheus/latest/querying/functions/)
* [PromLens](https://promlens.com/), the power tool for querying Prometheus

## Special labels

As you have already seen in several examples, a Prometheus metric is defined by one or more labels with the corresponding values. Two of those labels are special, because the Prometheus server will automatically generate them for every metric:


* instance

     The instance label describes the endpoint where Prometheus scraped the metric. This can be any application or exporter. In addition to the ip address or hostname, this label usually also contains the port number. Example: `10.0.0.25:9100`

* job

     This label contains the name of the scrape job as configured in the Prometheus configuration file. All instances configured in the same scrape job will share the same job label.


{{% alert title="Note" color="primary" %}}
Prometheus will append these labels dynamically before sample ingestion. Therefore you will not see these labels if you query the metrics endpoint directly (e.g. by using `curl`).

{{% /alert %}}


### Task {{% param sectionnumber %}}.1: Container restart alerting rule

Navigate to the [Prometheus user interface](http://LOCALHOST:19090/rules) and take a look at the provided default Prometheus rules.

{{% alert title="Note" color="primary" %}}
Search for an alert with `CrashLooping` in its name
{{% /alert %}}

**Task description**:

* Investigate if there is a default Alerting rule configured to monitor container restarts
* Which exporter exposes the required metrics?

{{% details title="Hints" mode-switcher="normalexpertmode" %}}

The Alerting rule is called `KubePodCrashLooping` and the PromQL defined for the rule looks as follows:

```promql
rate(kube_pod_container_status_restarts_total{job="kube-state-metrics"}[10m]) * 60 * 5 > 0
```

When you take a look at the query you will see, that there is filter to use just metrics from the `kube-state-metrics` exporter.

{{% /details %}}

### Task {{% param sectionnumber %}}.2: Memory usage of Prometheus

{{% alert title="Note" color="primary" %}}
Search for an metric with `memory_working_set` in its name
{{% /alert %}}

**Task description**:

* Display the memory usage of both Prometheus pods
* Use a filter to just display metrics from the `prometheus` containers

{{% details title="Hints" mode-switcher="normalexpertmode" %}}

```promql
container_memory_working_set_bytes{pod=~"prometheus-k8s-.*", container="prometheus"}
```

Your query returns two time series per Prometheus replica with the same value but different labels. One has additional information and allows you to filter the Docker ID.

```promql
# Docker container ID as an additional information
id="/docker/34750eee3d37c8cd3594fc46bb20ed166374ece7737c590b81ed1847aaa21d50/kubepods/burstable/pode2ab8dc5-ad8e-4b5c-9b0f-49c3bb5a8a34/dce2373557fb05e0bc9819b9f786f6e3bcc882280ee2eb9267edb7040886a55b"
# Kubernetes pod information
id="/kubepods/burstable/pode2ab8dc5-ad8e-4b5c-9b0f-49c3bb5a8a34/dce2373557fb05e0bc9819b9f786f6e3bcc882280ee2eb9267edb7040886a55b"
```

{{% /details %}}

### Task {{% param sectionnumber %}}.3: Kubernetes pod count

**Task description**:

* Display how many pods are currently running on your Kubernetes platform

{{% details title="Hints" mode-switcher="normalexpertmode" %}}

There are different ways to archive this. You can for example query all running containers and group them by `pod` and `namespace`.

```promql
count(sum(kube_pod_container_status_running == 1) by (pod,namespace))
```

You may also sum() all running pods on your Kubernetes nodes

```promql
sum(kubelet_running_pods)
```

{{% /details %}}


### Task {{% param sectionnumber %}}.4 Create your first dashboard

**Task description**:

In this task you're going to create your first own dashboard `my_dashboard`. You will add the panel `Memory Utilisation` with the metric `container_memory_working_set_bytes`.

{{% details title="Hints" mode-switcher="normalexpertmode" %}}

* Navigate to **+** (left navigation menu) > **Dashboard**
  * Select **Add an empty panel**
  * Add the query `container_memory_working_set_bytes{pod=~"prometheus-k8s-.*", container="prometheus"}` in the **Metrics browser** field
  * Set the panel title to `Memory Utilisation` under **Panel options > Title** (you may need to open the options pane with the **<** button on the right hand side just below the **Apply** button)
* Save the dashboard and give it the name `my_dashboard`

{{% /details %}}


### Task {{% param sectionnumber %}}.5 Add a Gauge panel to the dashboard

**Task description**:

Add another panel to the existing `my_dashboard` with the panel name `Pod count`. Display the metric `kubelet_running_pods` and change the panel type to `Gauge`.


{{% details title="Hints" mode-switcher="normalexpertmode" %}}

* Hit **Add panel** (top navigation menu) **> Add an empty panel**
  * Add the query `sum(kubelet_running_pods)` to the **Metrics browser** field
  * Set the panel title to `Pod count` under **Panel options > Title** (you may need to open the options pane with the **<** button on the right hand side just below the **Apply** button)
  * Choose **Gauge** in the dropdown menu just below the **Apply** button
* Save the dashboard

{{% /details %}}
