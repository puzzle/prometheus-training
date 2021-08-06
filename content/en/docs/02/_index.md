---
title: "2. Query and Visualize"
weight: 1
sectionnumber: 2
---

## Query

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

