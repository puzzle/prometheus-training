---
title: "3. Kubernetes Service Discovery"
weight: 1
sectionnumber: 3
---

## Collecting Application Metrics

When running applications in production, a fast feedback loop is a key factor. The following reasons show why it's essential to gather and combine all sorts of metrics when running an applications in production:

* To make sure that an application runs smoothly
* To be able to see production issues and send alerts
* To debug an application
* To take business and architectural decisions
* Metrics can also help to decide when to scale applications

There are a lot of frameworks and client libraries available, which integrate nicely into different application stacks.

The instrumented application provides Prometheus scrapable application metrics.

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

We also need to create a Service for the new application. Create a file with the name `~/work/service.yaml` with the following content:

{{< highlight yaml >}}{{< readfile file="content/en/docs/03/service.yaml" >}}{{< /highlight >}}

Create the Service with the following command:

```bash
kubectl apply -f ~/work/service.yaml -n application-metrics
```

This created a so-called [Kubernetes Service](https://kubernetes.io/docs/concepts/services-networking/service/)

```bash
kubectl -n application-metrics get services
```

Which gives you an output similar to this:

```bash
NAME                 TYPE       CLUSTER-IP       EXTERNAL-IP   PORT(S)          AGE
example-web-python   NodePort   10.101.249.125   <none>        5000:31626/TCP   2m9s
```

Our example application can now be reached on the port `31626`. This may be different in your setup.

We can now get the exposed url with the `minikube service` command:

```bash
minikube service example-web-python --url -n application-metrics
# http://192.168.49.2:31626
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

Since our newly deployed application now exposes metrics, the next thing we need to do, is to tell our Prometheus server to scrape metrics from the Kubernetes deployment. In a highly dynamic environment like Kubernetes this is done with so called Service Discovery.

## Service Discovery

When configuring Prometheus to scrape metrics from Containers deployed in a Kubernetes Cluster it doesn't really make sense to configure every single target manually. That would be far too static and wouldn't really work in a highly dynamic environment.

In fact, we tightly integrate Prometheus with Kubernetes and let Prometheus discover the targets, which need to be scraped automatically via the Kubernetes API.

The tight integration between Prometheus and Kubernetes can be configured with the [Kubernetes Service Discovery Config](https://prometheus.io/docs/prometheus/latest/configuration/configuration/#kubernetes_sd_config).

Now we instruct Prometheus to scrape our application metrics from the sample application by creating a ServiceMonitor.

ServiceMonitors are Kubernetes custom resources, which basically represent the scrape_config and look like this:

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

The Prometheus Operator watches namespaces for ServiceMonitor custom resources. It then updates the Service Discovery configuration of the Prometheus server(s) accordingly.

The selector part in the ServiceMonitor defines which Kubernetes Services will be scraped.

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

The `spec` section in the ServiceMonitor resource allows to further configure the targets.
In our case Prometheus will scrape:

* Every 30 seconds
* Look for a port with the name `http` (this must match the name in the Service resource)
* Scrape metrics from the path `/metrics` using `http`

### Task {{% param sectionnumber %}}.1: ServiceMonitors

**Task description**:

Create a ServiceMonitor  for the example application

* Create a ServiceMonitor, which will configure Prometheus to scrape metrics from the example-web-python application every 30 seconds.
  * hint: `kubectl -n application-metrics apply -f my_file.yaml` will create a resource in the Kubernetes namespace

For this to work, you need to ensure:

* The example-web-python Service is labeled correctly and matches the labels you've defined in your ServiceMonitor.
* The port name in your ServiceMonitor configuration matches the port name in the Service definition.
  * hint: check with `kubectl get service example-web-python -n application-metrics -o yaml`
* Verify the target in the Prometheus user interface

{{% alert title="Troubleshooting: Prometheus is not scrapping metrics" color="primary" %}}

Does the configuration of the ServiceMonitor appear in the Prometheus scrape config?

* Check if the label of your ServiceMonitor matches the label defined in the Prometheus custom resource
* Check the Prometheus operator logs for errors (Permission issues or invalid ServiceMonitors)

The Endpoint appears in the Prometheus scrape config but not under targets. The Service Discovery can't find the Endpoint.

* The namespaceSelector in the ServiceMonitor does not match the namespace of your app
* The label selector does not match the Service of your app
* The port name does not match the Service of your app

No data gets scraped

* The application does not provide metrics under the correct path and port
* Networking issues
* Authentication required, but not configured

{{% /alert %}}

{{% details title="Hints" mode-switcher="normalexpertmode" %}}

Create the following ServiceMonitor (`~/work/servicemonitor.yaml`) in the `application-metrics` namespace

{{< highlight yaml >}}{{< readfile file="content/en/docs/03/servicemonitor.yaml" >}}{{< /highlight >}}

Apply it using the following command:

```bash
kubectl -n application-metrics apply -f ~/work/servicemonitor.yaml
```

Verify that the target gets scraped in the [Prometheus user interface](http://LOCALHOST:19090/targets). Target name: `application-metrics/example-web-python-monitor/0`

{{% /details %}}

### Task {{% param sectionnumber %}}.2: Blackbox exporter in Kubernetes

Configuring a [multi-target exporter through relabel_configs](https://prometheus.io/docs/guides/multi-target-exporter/) can be a bit tricky to understand. The Prometheus operator brings us a so-called Probe custom resource, which allows us to define the targets for a black box exporter in a much simplified way.

**Task description**:

* Create a [Probe custom resource](https://github.com/prometheus-operator/prometheus-operator/blob/master/Documentation/design.md#probe) in the application-metrics namespace for the example-web-python application
* Use the Prometheus expression browser to check if the new metric is being scraped

{{% alert title="Note" color="primary" %}}

Use `kubectl describe crd probe | less` to describe the crd and get the available options.

{{% /alert %}}

{{% details title="Hints" mode-switcher="normalexpertmode" %}}

Create the following probe custom resource (`~/work/probe.yaml`) in the `application-metrics` namespace

{{< highlight yaml >}}{{< readfile file="content/en/docs/03/probe.yaml" >}}{{< /highlight >}}

Apply it using the following command:

```bash
kubectl -n application-metrics apply -f ~/work/probe.yaml
```

Verify that the target gets scraped in the [Prometheus user interface](http://LOCALHOST:19090/targets). Target name: `application-metrics/example-web-python-probe`

Check for the following metric in Prometheus:

```promql
{instance="example-web-python.application-metrics.svc:5000/health"}
```

{{% /details %}}
