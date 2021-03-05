---
title: "7. Prometheus in Kubernetes"
weight: 1
sectionnumber: 1
#onlyWhen: promOnK8s
---

## Prometheus in Kubernetes

{{% alert title="Note" color="primary" %}}
When running the Vagrant setup, make sure you have at least 16Gi on your local machine to run the following Kubernetes setup.
{{% /alert %}}

We will use minikube to start a minimal Kubernetes environment. <https://github.com/prometheus-operator/kube-prometheus#minikube>

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

The kube-prometheus stack includes a resource metrics API server, so the metrics-server addon is not necessary. Ensure the metrics-server addon is disabled on minikube

```bash
minikube addons disable metrics-server
```

Check if you can connect to the API and see the minikube master node.
```bash
kubectl get nodes
NAME       STATUS   ROLES    AGE     VERSION
minikube   Ready    master   4m15s   v1.19.4
```

Deploy the Prometheus Operator stack consisting of:
* Prometheus Operator deployment
* Prometheus Operator clusterrole and clusterrolebindings
* Custom Resource Definitions
  * `prometheuses`
 
     Manage your Prometheus instances
 
  * `alertmanagers`
 
    Manage your alertmanager instances
 
  * `servicemonitors`
 
    Generates kubernetes service discovery scrape configuration based on Kubernetes service definitions
 
  * `prometheusrules`
 
    Add Prometheus rules to your prometheus
 
  * `alertmanagerconfigs`
 
    Add additional receivers and routes to your existing alertmanager configuration
 
  * `podmonitors`
 
    Generates kubernetes service discovery scrape configuration based on Kubernetes pod definitions
 
  * `probes`
 
    First class custom resource to manager blackbox targets
 
  * `thanosrulers`
 
    Manage thanos rulers

```bash
git clone https://github.com/prometheus-operator/kube-prometheus.git ~/work/kube-prometheus
cd ~/work/kube-prometheus
kubectl create -f manifests/setup
```

The manifest will deploy a complete monitoring stack consisting of:

* Two Prometheus replicas
* Alertmanager cluster
* Grafana
* kube-state metrics
* node_exporter
* A set of default PrometheusRules
* A set of default dashboards


```bash
kubectl create -f manifests/
```

Check if you can access the monitoring stack user interfaces


FIXME: nodeport
```bash
watch kubectl -n monitoring get pods
kubectl -n monitoring port-forward --address=0.0.0.0 svc/prometheus-k8s 9090
kubectl -n monitoring port-forward --address=0.0.0.0 svc/grafana 3000
kubectl -n monitoring port-forward --address=0.0.0.0 svc/alertmanager-main 9093
```
