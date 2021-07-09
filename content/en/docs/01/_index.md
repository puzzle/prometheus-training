---
title: "1. Install and configure"
weight: 1
sectionnumber: 1
---

## Prometheus

We will use [minikube](https://minikube.sigs.k8s.io/docs/start/) to start a minimal Kubernetes environment. If you are a novice in Kubernetes, you may want to use the [kubectl cheatsheet](https://kubernetes.io/docs/reference/kubectl/cheatsheet/)

{{% alert title="Minikube" color="primary" %}}
Minikube is already started and configured. When you restart your virtual machine, you might need to start it manually.

```bash
minikube start \
--kubernetes-version=v1.20.2 \
--memory=6g \
--cpus=4 \
--bootstrapper=kubeadm \
--extra-config=kubelet.authentication-token-webhook=true \
--extra-config=kubelet.authorization-mode=Webhook \
--extra-config=scheduler.address=0.0.0.0 \
--extra-config=controller-manager.address=0.0.0.0
```

{{% /alert %}}

Check if you can connect to the API and verify the minikube master node is in `ready` state.

```bash
kubectl get nodes
```

```bash
NAME       STATUS   ROLES                  AGE    VERSION
minikube   Ready    control-plane,master   2m2s   v1.20.2
```

Deploy the Prometheus operator stack, consisting of:

* Prometheus Operator [deployment](https://kubernetes.io/docs/concepts/workloads/controllers/deployment/)
* Prometheus Operator [ClusterRole and ClusterRoleBinding](https://kubernetes.io/docs/reference/access-authn-authz/rbac/#rolebinding-and-clusterrolebinding)
* CustomResourceDefinitions
  * [Prometheus](https://github.com/prometheus-operator/prometheus-operator/blob/master/Documentation/api.md#prometheus): Manage the Prometheus instances
  * [Alertmanager](https://github.com/prometheus-operator/prometheus-operator/blob/master/Documentation/api.md#alertmanager): Manage the Alertmanager instances
  * [ServiceMonitor](https://github.com/prometheus-operator/prometheus-operator/blob/master/Documentation/api.md#servicemonitor): Generate Kubernetes service discovery scrape configuration based on Kubernetes [service](https://kubernetes.io/docs/concepts/services-networking/service/) definitions
  * [PrometheusRule](https://github.com/prometheus-operator/prometheus-operator/blob/master/Documentation/api.md#prometheusrule): Manage the Prometheus rules of your Prometheus
  * [AlertmanagerConfig](https://github.com/prometheus-operator/prometheus-operator/blob/master/Documentation/api.md#alertmanagerconfig): Add additional receivers and routes to your existing Alertmanager configuration
  * [PodMonitor](https://github.com/prometheus-operator/prometheus-operator/blob/master/Documentation/api.md#podmonitor): Generate Kubernetes service discovery scrape configuration based on Kubernetes pod definitions
  * [Probe](https://github.com/prometheus-operator/prometheus-operator/blob/master/Documentation/api.md#probe): Custom resource to manage Prometheus blackbox exporter targets
  * [ThanosRuler](https://github.com/prometheus-operator/prometheus-operator/blob/master/Documentation/api.md#thanosruler): Manage [Thanos rulers](https://github.com/thanos-io/thanos/blob/main/docs/components/rule.md)

```bash
git clone https://github.com/prometheus-operator/kube-prometheus.git ~/work/kube-prometheus
cd ~/work/kube-prometheus
git checkout release-0.8
kubectl create -f manifests/setup
```

The manifest will deploy a complete monitoring stack consisting of:

* Two Prometheis
* Alertmanager cluster
* Grafana
* kube-state metrics exporter
* node_exporter
* blackbox exporter
* A set of default PrometheusRules
* A set of default dashboards

```bash
kubectl create -f manifests/
```

By default, Prometheus is only allowed to monitor the `default`, `monitoring` and `kube-system` namespaces. Therefore we will add the necessary ClusterRoleBinding to grant Prometheus access to cluster-wide resources. Also we will create the needed ingress definitions for you, which will expose the monitoring components.

```bash
kubectl -n monitoring apply -f \
https://raw.githubusercontent.com/puzzle/prometheus-training/main/content/en/docs/07/resources.yaml
```

Wait until all pods are running

```bash
watch kubectl -n monitoring get pods
```

Check if you can access the Prometheus web interface at <http://LOCALHOST:19090>

Check access to Alertmanager at <http://LOCALHOST:19093>

Check access to Grafana at <http://LOCALHOST:13000>
{{% alert title="Note" color="primary" %}}
Use the default Grafana logging credentials and change the password

* username: admin
* password: admin

{{% /alert %}}

### Task {{% param sectionnumber %}}.1: Prometheus web UI

Get a feel for how to use the Prometheus web UI. Open the [web UI](http://LOCALHOST:19090) and navigate to the **Graph** menu (right on top in the grey navigation bar next to Alerts).

{{% details title="Hints" mode-switcher="normalexpertmode" %}}

![Prometheus UI](prometheus-ui.png)

Let's start and find a memory related metric. The best way to start is by typing `node_memory` in the expression bar.

{{% alert title="Note" color="primary" %}}
As soon as you start typing a dropdown with matching metrics is shown.
{{% /alert %}}

You can also open the `Metrics Explorer` by clicking on the globe symbol next to the `Execute` button.

Select a metric such as `node_memory_MemFree_bytes` and click the `Execute` button.

The result of your first Query will be available under the two tabs:

1. Table
1. Graph

Explore those two views on your results. Shrink the time range in the Graph tab.

{{% /details %}}

### Task {{% param sectionnumber %}}.2: Metric Prometheus server version

Prometheus collects its own metrics, so information such as the current build version of your Prometheus server is displayed as a metric.

Let's find a metric that shows you the version of your Prometheus server.

{{% details title="Hints" mode-switcher="normalexpertmode" %}}

Start typing `prometheus_...` in the expression browser, choose the `prometheus_build_info` metric and click the `Execute` Button.

Something similar to the following will be displayed

```promql
prometheus_build_info{branch="HEAD", container="prometheus", endpoint="web", goversion="go1.16.2", instance="172.17.0.13:9090", job="prometheus-k8s", namespace="monitoring", pod="prometheus-k8s-1", revision="3cafc58827d1ebd1a67749f88be4218f0bab3d8d", service="prometheus-k8s", version="2.26.0"}
	1
prometheus_build_info{branch="HEAD", container="prometheus", endpoint="web", goversion="go1.16.2", instance="172.17.0.14:9090", job="prometheus-k8s", namespace="monitoring", pod="prometheus-k8s-0", revision="3cafc58827d1ebd1a67749f88be4218f0bab3d8d", service="prometheus-k8s", version="2.26.0"}
	1
```

The actual Version of your Prometheus Server will be available as label `version`
```promql
{version="2.26.0"}
```

{{% /details %}}

### Task {{% param sectionnumber %}}.3: Display pod metrics in Kubernetes Grafana

The Prometheus operator stack provides a few generic dashboards for your Kubernetes cluster deployment. These dashboards provide you with information about the resource usage of Kubernetes infrastructure components or your deployed apps. They also show you latency and availability of Kubernetes core components.

**Task description**:

* Navigate to your [Kubernetes Grafana](http://LOCALHOST:13000)
* Find a dashboard that displays compute resources per namespace and pod
* Take a look at the metrics from the `monitoring` namespace

{{% alert title="Note" color="primary" %}}
The initial password is `admin`. You need to change it after the first login.
{{% /alert %}}

{{% details title="Hints" mode-switcher="normalexpertmode" %}}

* Use the search function (magnifying glass) on the left side and hit `Search`
* The dashboard name is `Kubernetes / Compute Resources / Namespace (Pods)` and can be found in the `Default` directory
* Select `monitoring` in the namespace drop-down

You get usage metrics for CPU and memory as well as network statistics per pod in the namespace `Monitoring`.

{{% /details %}}

### Task {{% param sectionnumber %}}.4: Configure Prometheus Retention

By default, the Prometheus operator stack will set the retention of your metrics to `24h`.
Read about [retention operational-aspects](https://prometheus.io/docs/prometheus/latest/storage/#operational-aspects) for options to manage retention.

**Task description**:

* Set Prometheus retention time to two days and retention size to 9Gi

{{% alert title="Note" color="primary" %}}
Check [documentation](https://github.com/prometheus-operator/prometheus-operator/blob/master/Documentation/api.md#prometheusspec) for available options
{{% /alert %}}

{{% details title="Hints" mode-switcher="normalexpertmode" %}}

Set the following options

```bash
kubectl -n monitoring edit prometheus k8s
```

```bash
spec:
...
  retention: 2d
  retentionSize: 9GB
...
```

Check if the retention parameter is set in the running pod.

```bash
kubectl -n monitoring describe pods prometheus-k8s-0
...
Containers:
  prometheus:
    Args:
      --storage.tsdb.retention.size=9GB
      --storage.tsdb.retention.time=2d
...
```

{{% /details %}}