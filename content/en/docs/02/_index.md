---
title: "2. Query and Visualize"
weight: 1
sectionnumber: 2
---

## Query

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

There are different ways to archive this. You may sum() all running pods on your Kubernetes nodes

```promql
sum(kubelet_running_pods)
```

You can also query all running containers and group them by `pod` and `namespace`.

```promql
count(sum(kube_pod_container_status_running == 1) by (pod,namespace))
```

{{% /details %}}

## Visualize


### Task {{% param sectionnumber %}}.4 Create your first dashboard

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


### Task {{% param sectionnumber %}}.5 Add a Gauge panel to the dashboard

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
