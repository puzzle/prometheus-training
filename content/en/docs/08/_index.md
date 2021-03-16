---
title: "8. Prometheus in Kubernetes App Monitoring"
weight: 1
sectionnumber: 8
---


## Collecting Application Metrics

When running applications in production, a fast feedback loop is a key factor. The following reasons show why it's essential to gather and combine all sorts of metrics when running an application in production:

* To make sure that an application runs smoothly
* To be able to see production issues and send alerts
* To debug an application
* To take business and architectural decisions
* Metrics can also help to decide when to scale applications

As we saw in [lab 5 Instrumenting with client libraries](../05/) Application Metrics (e.g. Request Count on a specific URL, gc metrics, or even Custom Metrics and many more), application metrics are collected within the application. There are a lot of frameworks and client libraries available, which integrate nicely into different application stacks.

The instrumented application provides prometheus scrapable application metrics.

Create a namespace where the example application can be deployed to.

```bash
kubectl create namespace application-metrics
```

Deploy the Acend example Python application, which provides application metrics at `/metrics`:

```bash
kubectl -n application-metrics create deployment example-web-python \
--image=quay.io/acend/example-web-python
```

Use the following command to verify the deployment, that the pod `example-web-python` is Ready and Running. (use CTRL C to exit the command)

```bash
kubectl -n application-metrics get pod -w
```

We also need to create a service for the new application. Create a file with the name `~/work/service.yaml` with the following content:

{{< highlight yaml >}}{{< readfile file="content/en/docs/08/service.yaml" >}}{{< /highlight >}}

Create the service with the following command:

```bash
kubectl apply -f ~/work/service.yaml -n application-metrics
```

This created a so-called [Kubernetes Service](https://kubernetes.io/docs/concepts/services-networking/service/)

```bash
kubectl get services --namespace application-metrics
```

Which gives you an output similar to this:

```bash
NAME                 TYPE       CLUSTER-IP       EXTERNAL-IP   PORT(S)          AGE
example-web-python   NodePort   10.101.249.125   <none>        5000:31626/TCP   2m9s
```

Our example application can now be reached on the port `31626`. This may be different at your setup..

We can now get the exposed url with the `minikube service` command:

```bash
minikube service example-web-python --url -n application-metrics
```

```bash
http://192.168.49.2:31626
```

Use `curl` and verify the successful deployment of our example application:

```bash
curl $(minikube service example-web-python --url -n application-metrics)/metrics
```

Should result in something like:

```bash
# HELP python_gc_objects_collected_total Objects collected during gc
# TYPE python_gc_objects_collected_total counter
python_gc_objects_collected_total{generation="0"} 541.0
python_gc_objects_collected_total{generation="1"} 344.0
python_gc_objects_collected_total{generation="2"} 15.0
...
```

Since our newly deployed application now exposes metrics, the next thing we need to do, is to tell our prometheus server to scrape the kubernetes deployment. In a highly dynamic environment like Kubernetes this is done with so called Service Discovery.

## Service Discovery

When configuring Prometheus to scrape metrics from Services, Endpoints, Deployments and Pods deployed in a Kubernetes Cluster it doesn't really make sense to configure every single target manually. That would be way to static and won't really work in a highly dynamic environment. Instead it makes sense to use a similar concept, like we used in [lab 1 - file discovery](../01/).

In fact we actually integrate Prometheus with Kubernetes tightly and let Prometheus discover the targets, which need to be scraped automatically via the Kubernetes API.

The tight integration between Prometheus and Kubernetes can be configured with the [Kubernetes Service Discovery Config](https://prometheus.io/docs/prometheus/latest/configuration/configuration/#kubernetes_sd_config)

If we now want to tell prometheus to scrape our application metrics from the example application, we can create a so called Servicemonitor.

ServiceMonitors are custom Kubernetes resources, which basically represent the scrape_config and look like this:

```yaml
apiVersion: monitoring.coreos.com/v1
kind: ServiceMonitor
metadata:
  labels:
    app.kubernetes.io/name: example-web-python
  name: example-web-python-monitor
spec:
  endpoints:
  - interval: 30s
    port: http
    scheme: http
    path: /metrics
  selector:
    matchLabels:
      prometheus-monitoring: 'true'
```

### How does it work

The Prometheus Operator watches namespaces for ServiceMonitor custom resources. It then updates the Service Discovery configuration accordingly.

The selector part in the Service Monitor then defines which Kubernetes Services will be scraped.

```yaml
# servicemonitor.yaml
...
  selector:
    matchLabels:
      prometheus-monitoring: 'true'
...
```

And the corresponding Service

```yaml
apiVersion: v1
kind: Service
metadata:
  name: example-web-python
  labels:
    prometheus-monitoring: 'true'
...
```

This means Prometheus scrapes all Endpoints where the `prometheus-monitoring: 'true'` label is set.

The `spec` section in the ServiceMonitor resource allows us to further configure the targets Prometheus will scrape.
In our case Prometheus will scrape:

* Every 30 seconds
* Look for a port with the name `http` (this must match the name in the Service resource)
* Scrape metrics from the path `/metrics` using `http`

## Best practices

Use the common k8s labels <https://kubernetes.io/docs/concepts/overview/working-with-objects/common-labels/>

If possible, reduce the number of different ServiceMonitors for an application and thereby reducing complexity.

* Use the same `matchLables` on different Services for your application (e.g. Frontend Service, Backend Service, Database Service)
* Also make sure the ports of different Services have the same name
* Expose your metrics under the same path

Avoid relabeling and use standards or change the metric labels within the exporter.
