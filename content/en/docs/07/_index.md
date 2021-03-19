---
title: "7. Prometheus in Kubernetes"
weight: 1
sectionnumber: 1
---

## Prometheus in Kubernetes

{{% alert title="Note" color="primary" %}}
When running the [Vagrant](https://prometheus-training.puzzle.ch/setup/) setup, make sure you have at least 16Gi on your local machine to run the Prometheus Kubernetes setup.
{{% /alert %}}

We will use [minikube](https://minikube.sigs.k8s.io/docs/start/) to start a minimal Kubernetes environment. If you are a novice in Kubernetes, you may want to use the [kubectl cheatsheet](https://kubernetes.io/docs/reference/kubectl/cheatsheet/)

{{% alert title="Note" color="primary" %}}
Minikube is already started and configured. When you restart your virtual machine, you might need to start it manually.

```bash
minikube start \
--kubernetes-version=v1.19.0 \
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
NAME       STATUS   ROLES    AGE     VERSION
minikube   Ready    master   4m15s   v1.19.4
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

Check if you can access the Prometheus web interface at `localhost:19090`

Check access to Alertmanager at `localhost:19093`

Check access to Grafana at `localhost:13000`
{{% alert title="Note" color="primary" %}}
Use default Grafana loging credentials

* username: admin
* password: admin

{{% /alert %}}

{{% alert title="Note" color="primary" %}}
When you run the [Vagrant](https://prometheus-training.puzzle.ch/setup/) setup, you can make your monitoring component accessible through port-forward.

```bash
kubectl -n monitoring port-forward svc/prometheus-k8s 19090:9090 &
kubectl -n monitoring port-forward svc/alertmanager-main 19093:9093 &
kubectl -n monitoring port-forward svc/grafana 13000:3000 &
```

{{% /alert %}}
