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
# HELP <metric name> <info>
# TYPE <metric name> <metric type>
<metric name>{<label name>=<label value>, ...} <sample value>
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


### Task {{% param sectionnumber %}}.1: Container restart alerting rule

Navigate to the [Prometheus user interface](http://LOCALHOST:19090/rules) and take a look at the provided default Prometheus rules.

{{% alert title="Note" color="primary" %}}
Search for an Alerting rule with `CrashLooping` in its name
{{% /alert %}}

**Task description**:

* Investigate if there is a default Alerting rule configured to monitor container restarts
* Which exporter exposes the required metrics?

{{% details title="Hints" mode-switcher="normalexpertmode" %}}

The Alerting rule is called `KubePodCrashLooping` and the PromQL defined for the rule looks as follows:

```promql
rate(kube_pod_container_status_restarts_total{job="kube-state-metrics"}[10m]) * 60 * 5 > 0
```
If you take a look at the query, you will see that there is a filter that only uses metrics from the `kube-state-metrics` exporter.

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

There are different ways to archive this. You may sum() all running pods on your Kubernetes nodes

```promql
sum(kubelet_running_pods)
```

You can also query all running containers and group them by `pod` and `namespace`.

```promql
count(sum(kube_pod_container_status_running == 1) by (pod,namespace))
```

{{% /details %}}


### Task {{% param sectionnumber %}}.4: Rate Function

Use the `rate()` function to display the current CPU **idle** usage per CPU core of the Prometheus server in % based on data of the last 5 minutes.

{{% alert title="Hint" color="primary" %}}
Read the [documentation](https://prometheus.io/docs/prometheus/latest/querying/functions/) about the `rate()` function.
{{% /alert %}}

{{% details title="Hints" mode-switcher="normalexpertmode" %}}

The CPU metrics are collected and exposed by the `node_exporter` therefore the metric we're looking for is under the `node` namespace.

```promql
node_cpu_seconds_total
```

To get the `idle` CPU seconds, we add the label filter `{mode="idle"}`.

Since the `rate` function calculates the per-second average increase of the time series in a **range vector**, we have to pass a range vector to the function `node_cpu_seconds_total{mode="idle"}[5m]`

To get the idle usage in % we therefore have to multiply it with 100.

```promql
rate(
  node_cpu_seconds_total{mode="idle"}[5m]
  )
* 100
```

{{% /details %}}


### Task {{% param sectionnumber %}}.5 Create your first dashboard

**Task description**:

Navigate to the [Grafana server](http://LOCALHOST:13000), create a dashboard `my_dashboard` and add the panel `Memory Utilisation` with the metric `container_memory_working_set_bytes`.

{{% details title="Hints" mode-switcher="normalexpertmode" %}}

* Navigate to **+** (left navigation menu) > **Dashboard**
  * Select **Add an empty panel**
  * Select the **prometheus** datasource
  * Add the query `container_memory_working_set_bytes{pod=~"prometheus-k8s-.*", container="prometheus"}` in the **Metrics browser** field
  * Set the panel title to `Memory Utilisation` under **Panel options > Title** (you may need to open the options pane with the **<** button on the right hand side just below the **Apply** button)
* Save the dashboard and give it the name `my_dashboard`

{{% /details %}}


### Task {{% param sectionnumber %}}.6 Add a Gauge panel to the dashboard

**Task description**:

Add another panel to the existing `my_dashboard` with the panel name `Pod count`. Display the metric `kubelet_running_pods` and change the panel type to `Gauge`.


{{% details title="Hints" mode-switcher="normalexpertmode" %}}

* Hit **Add panel** (top navigation menu) **> Add an empty panel**
  * Select the **prometheus** datasource
  * Add the query `sum(kubelet_running_pods)` to the **Metrics browser** field
  * Set the panel title to `Pod count` under **Panel options > Title** (you may need to open the options pane with the **<** button on the right hand side just below the **Apply** button)
  * Choose **Gauge** in the **visualization** dropdown menu just below the **Apply** button
* Save the dashboard

{{% /details %}}
