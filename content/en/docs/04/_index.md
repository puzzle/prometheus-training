---
title: "4. Additional Tasks"
weight: 1
sectionnumber: 4
---

## Optional Tasks

In this section you will find some optional tasks.


### Task {{% param sectionnumber %}}.1: Check if Alertmanager is running clustered (optional)

The Prometheus operator stack deployed three Alertmanagers. Normally there would be further configuration needed to make sure that these instances running clustered. But as we are running Alertmanager managed by Prometheus operator this should be done automatically.

**Task description**: Investigate if Alertmanger is clustered and which paramaters have been set by the operator

{{% details title="Hints" mode-switcher="normalexpertmode" %}}

The Alertmanager custom resource has 3 replicas configured

```bash
kubectl -n monitoring get alertmanager main -o yaml
...
spec:
  replicas: 3
...
```

The operator makes sure that the Alertmanagers know about each other and generates the necessary [configuration](https://github.com/prometheus/alertmanager#high-availability) to form a cluster. Let's review the pod definition:

```bash
kubectl -n monitoring get pods alertmanager-main-0 -o yaml
...
spec:
  containers:
  - args:
    - --cluster.listen-address=[$(POD_IP)]:9094
    - --cluster.peer=alertmanager-main-0.alertmanager-operated:9094
    - --cluster.peer=alertmanager-main-1.alertmanager-operated:9094
    - --cluster.peer=alertmanager-main-2.alertmanager-operated:9094
...
```

{{% /details %}}


### Task {{% param sectionnumber %}}.2: Add a custom Prometheus rule

The Prometheus operator allows you to extend the existing Prometheus rules with your own rules using the [PrometheusRule custom resource](https://github.com/prometheus-operator/prometheus-operator/blob/master/Documentation/api.md#prometheusrule). You can take a look at existing PrometheusRules or at this [example](https://github.com/prometheus-operator/prometheus-operator/blob/master/Documentation/user-guides/alerting.md#fire-alerts) about the format of PrometheusRules.

**Task description**:

* Add a custom Prometheus rule to the monitoring stack, wich checks if you have reached the defined retention size

{{% alert title="Note" color="primary" %}}
You can use the `prometheus_tsdb_size_retentions_total` metric
{{% /alert %}}

* Set the labels `prometheus: k8s` and `role: alert-rules` on your PrometheusRule to match the resource with your Prometheus configuration

See Prometheus custom resource as reference

```bash
kubectl -n monitoring edit prometheuses k8s
```

```yaml
...
spec:
  ruleSelector:
    matchLabels:
      prometheus: k8s
      role: alert-rules
...
```

{{% details title="Hints" mode-switcher="normalexpertmode" %}}

There are different ways to archive this. The first approach is to just check the number of times that blocks were deleted because the maximum number of bytes was exceeded.

{{< highlight yaml >}}{{< readfile file="content/en/docs/04/prometheus-custom-rule.yml" >}}{{< /highlight >}}

```bash
curl -o ~/work/prometheus-custom-rule.yml \
https://raw.githubusercontent.com/puzzle/prometheus-training/cloud-native-day/content/en/docs/04/prometheus-custom-rule.yml
kubectl -n monitoring create -f ~/work/prometheus-custom-rule.yml
```

Another approach is to alert, when the on-disk time series database size is greater than `prometheus_tsdb_retention_limit_bytes`. For example:

```promql
prometheus_tsdb_storage_blocks_bytes >= prometheus_tsdb_retention_limit_bytes
```

{{% /details %}}


### Task {{% param sectionnumber %}}.3: Configure additional Alertmanager receiver

We can manage the Kubernetes Alertmanager via several approaches. In this task, we will learn how to add an additional receiver using [alertmanagerConfig custom resource](https://github.com/prometheus-operator/prometheus-operator/blob/master/Documentation/user-guides/alerting.md#alertmanagerconfig-resource). First we need do define the alertmanagerConfigSelector label in the `Alertmanager`. This must match the labels defined in our alertmanagerConfig.

```bash
kubectl -n monitoring edit alertmanagers main
```

```bash
spec:
...
  alertmanagerConfigSelector:
    matchLabels:
      alertmanagerConfig: training
```

**Task description**:

* Configure Alertmanger to send all alerts from the monitoring namespace to [MailCatcher](http://LOCALHOST:1080)
* Create a `AlertmanagerConfig` custom resource. See [example](https://github.com/prometheus-operator/prometheus-operator/blob/master/Documentation/user-guides/alerting.md#alertmanagerconfig-resource) as reference
* Name the resource `mailcatcher`
* Define the following route and receiver

```yaml
  route:
    receiver: 'mailcatcher'
  receivers:
    - name: 'mailcatcher'
      emailConfigs:
        - to: 'alert@localhost'
          from: 'prometheus-operator@localhost'
          smarthost: '192.168.49.1:1025'
          requireTLS: false
```

{{% alert title="Note" color="primary" %}}
When you create an `AlertmanagerConfig`, it will only match alerts that have the namespace label set to the scope in which the `AlertmanagerConfig` is defined. In our case:

```bash
route
...
  routes:
  - receiver: monitoring-mailcatcher-mailcatcher
    match:
      namespace: monitoring
...
```

{{% /alert %}}

{{% details title="Hints" mode-switcher="normalexpertmode" %}}

Add the AlertmanagerConfig

{{< highlight yaml >}}{{< readfile file="content/en/docs/04/mailcatcher.yml" >}}{{< /highlight >}}

```bash
curl -o ~/work/mailcatcher.yml \
https://raw.githubusercontent.com/puzzle/prometheus-training/cloud-native-day/content/en/docs/04/mailcatcher.yml

kubectl -n monitoring create -f ~/work/mailcatcher.yml
```


**Optional**: You can add an alert to check your configuration using the amtool and check if the [MailCatcher](http://LOCALHOST:1080) received the mail. It can take up to 5 minutes as the alarms are grouped together based on the [group_interval](https://prometheus.io/docs/alerting/latest/configuration/#route). E.g.

```bash
kubectl -n monitoring exec alertmanager-main-0  -c alertmanager -- \
amtool alert add --alertmanager.url=http://localhost:9093 alertname=test namespace=monitoring severity=critical
```

{{% /details %}}

### Task {{% param sectionnumber %}}.4: Deploy a database and use a sidecar container to expose metrics

**Task description**:

As we've learned in [Lab 4 - Prometheus exporters](../../../04/) when applications do not expose metrics in the Prometheus format, there are a lot of exporters available to convert metrics into the correct format. In Kubernetes this is often done by deploying so called sidecar containers along with the actual application.

Use the following command to deploy a MariaDB database in the `application-metrics` namespace.

```bash
curl -o ~/work/mariadb.yaml \
https://raw.githubusercontent.com/puzzle/prometheus-training/cloud-native-day/content/en/docs/04/mariadb.yaml
kubectl -n application-metrics apply -f ~/work/mariadb.yaml
```

This will create a [Secret](https://kubernetes.io/docs/concepts/configuration/secret/) (username password to access the database), a [Service](https://kubernetes.io/docs/concepts/services-networking/service/) and the [Deployment](https://kubernetes.io/docs/concepts/workloads/controllers/deployment/).

* Deploy the [mariadb exporter](https://github.com/prometheus/mysqld_exporter) from <https://registry.hub.docker.com/r/prom/mysqld-exporter/> as a sidecar container
  * Alter the existing MariaDB deployment definition (~/work/mariadb.yaml) to contain the side car
  * Apply your changes to the cluster with `kubectl -n application-metrics apply -f ~/work/mariadb.yaml`
* Create a ServiceMonitor to instruct Prometheus to scrape the sidecar container

{{% details title="Hints" mode-switcher="normalexpertmode" %}}

First we need to alter the deployment of the MariaDB with adding the MariaDB exporter as a second container.
Then extend the service by adding a second port for the MariaDB exporter.

{{< highlight yaml >}}{{< readfile file="content/en/docs/04/mariadb-sidecar.yaml" >}}{{< /highlight >}}

We can apply the file above using:

```bash
kubectl -n application-metrics apply -f ~/work/mariadb.yaml
```

Then we also need to create a new ServiceMonitor `~/work/servicemonitor-sidecar.yaml`.

{{< highlight yaml >}}{{< readfile file="content/en/docs/04/servicemonitor-sidecar.yaml" >}}{{< /highlight >}}

```bash
kubectl -n application-metrics apply -f ~/work/servicemonitor-sidecar.yaml
```

Verify that the target gets scraped in the [Prometheus user interface](http://LOCALHOST:19090/targets). Target name: `application-metrics/mariadb/0`

{{% /details %}}
