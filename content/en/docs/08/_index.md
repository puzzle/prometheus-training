---
title: "8. Prometheus in Kubernetes App Monitoring"
weight: 8
sectionnumber: 8
---

{{% onlyWhen baloise %}}

## Executing oc commands

{{% alert title="Note" color="primary" %}}
Execute the following `oc` commands using one of those options:

* OpenShift Webconsole Terminal <http://{{% param replacePlaceholder.openshift_console %}}> right top menu `>_`
* On your local machine using the `oc` tool, make sure to login on your OpenShift Cluster first.

{{% /alert %}}
{{% /onlyWhen %}}


## Collecting Application Metrics

When running applications in production, a fast feedback loop is a key factor. The following reasons show why it's essential to gather and combine all sorts of metrics when running an application in production:

* To make sure that an application runs smoothly
* To be able to see production issues and send alerts
* To debug an application
* To take business and architectural decisions
* Metrics can also help to decide when to scale applications

As we saw in [Lab 5 - Instrumenting with client libraries](../05/) Application Metrics (e.g. Request Count on a specific URL, GC metrics, or even Custom Metrics and many more) are collected within the application. There are a lot of frameworks and client libraries available, which integrate nicely into different application stacks.

The instrumented application provides Prometheus scrapable application metrics.

{{% onlyWhenNot baloise %}}

Create a namespace where the example application can be deployed to.

```bash
{{% param cliToolName %}} create namespace application-metrics
```

Deploy the Acend example Python application, which provides application metrics at `/metrics`:

```bash
{{% param cliToolName %}} -n application-metrics create deployment example-web-python \
--image=quay.io/acend/example-web-python
```

Use the following command to verify the deployment, that the pod `example-web-python` is Ready and Running. (use CTRL C to exit the command)

```bash
{{% param cliToolName %}} -n application-metrics get pod -w
```

We also need to create a Service for the new application. Create a file with the name `~/work/service.yaml` with the following content:

{{< readfile file="/content/en/docs/08/service.yaml" code="true" lang="yaml" >}}

Create the Service with the following command:

```bash
{{% param cliToolName %}} apply -f ~/work/service.yaml -n application-metrics
```

This created a so-called [Kubernetes Service](https://kubernetes.io/docs/concepts/services-networking/service/)

```bash
{{% param cliToolName %}} -n application-metrics get services
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

{{% /onlyWhenNot %}}

{{% onlyWhen baloise %}}

{{% alert title="Note" color="primary" %}}
We will deploy an application for demonstration purposes in our monitoring namespace. This should never be done for production use cases. If you are familiar with deploying on OpenShift, you can complete this lab by deploying the application on our test cluster.
{{% /alert %}}

Create the following file `training_python-deployment.yaml` in your monitoring directory.

{{< readfile file="/content/en/docs/08/baloise_python-deployment.yaml" code="true" lang="yaml" >}}

Use the following command to verify that the pod of the deployment `example-web-python` is ready and running (use CTRL+C to exit the command).

```bash
team=<team>
{{% param cliToolName %}} -n $team-monitoring get pod -w -l app=example-web-python
```

We also need to create a Service for the new application. Create a file with the name `training_python-service.yaml` with the following content:

{{< readfile file="/content/en/docs/08/baloise_python-service.yaml" code="true" lang="yaml" >}}

This created a so-called [Kubernetes Service](https://kubernetes.io/docs/concepts/services-networking/service/)

```bash
team=<team>
{{% param cliToolName %}} -n $team-monitoring get svc -l app=example-web-python
```

Which gives you an output similar to this:

```bash
NAME                 TYPE       CLUSTER-IP       EXTERNAL-IP   PORT(S)          AGE
example-web-python                  ClusterIP   172.24.195.25    <none>        5000/TCP                     24s
```

Our example application can now be reached on port `5000`.

We can now make the application directly available on our machine using [port-forward](https://kubernetes.io/docs/tasks/access-application-cluster/port-forward-access-application-cluster/)

```bash
team=<team>
{{% param cliToolName %}} -n $team-monitoring port-forward svc/example-web-python 5000
```

Use `curl` and verify the successful deployment of our example application in a separate terminal:

```bash
curl localhost:5000/metrics
```
{{% /onlyWhen %}}

Should result in something like:

```promql
# HELP python_gc_objects_collected_total Objects collected during gc
# TYPE python_gc_objects_collected_total counter
python_gc_objects_collected_total{generation="0"} 541.0
python_gc_objects_collected_total{generation="1"} 344.0
python_gc_objects_collected_total{generation="2"} 15.0
...
```

Since our newly deployed application now exposes metrics, the next thing we need to do, is to tell our Prometheus server to scrape metrics from the Kubernetes deployment. In a highly dynamic environment like Kubernetes this is done with so called Service Discovery.


## Service Discovery

When configuring Prometheus to scrape metrics from Containers deployed in a Kubernetes Cluster it doesn't really make sense to configure every single target manually. That would be far too static and wouldn't really work in a highly dynamic environment. Instead it makes sense to use a similar concept, like we used in [Lab 1 - Dynamic configuration](../01/#dynamic-configuration).

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

## Best practices

Use the common k8s labels <https://kubernetes.io/docs/concepts/overview/working-with-objects/common-labels/>

If possible, reduce the number of different ServiceMonitors for an application and thereby reduce the overall complexity.

* Use the same `matchLabels` on different Services for your application (e.g. Frontend Service, Backend Service, Database Service)
* Also make sure the ports of different Services have the same name
* Expose your metrics under the same path

Avoid relabeling and use standards or change the metric labels within the exporter.
