---
title: "3.1 Tasks: Setup custom alerting rules"
weight: 2
sectionnumber: 3.1
onlyWhen: baloise
---

### Task {{% param sectionnumber %}}.1: Add alerting rules

{{% alert title="Note" color="primary" %}}
Alertmanager will automatically send mails to the defined `responsible` email address in the teams root configuration when you set the label `severity=critical` in your PrometheusRule.
To change this behaviour and/or add Alerting to MS Teams, check the documentation [03 - Setup custom alerting rules](https://confluence.baloisenet.com/atlassian/display/BALMATE/03+-+Setup+custom+alerting+rules#id-03Setupcustomalertingrules-Alerting) in Confluence.
{{% /alert %}}

The Prometheus Operator allows you to configure Alerting Rules (PrometheusRules). This enables OpenShift users to configure and maintain alerting rules for their projects. Furthermore it is possible to treat Alerting Rules like any other Kubernetes resource and lets you manage them in Helm or Kustomize. A PrometheusRule has the following form:

```yaml
apiVersion: monitoring.coreos.com/v1
kind: PrometheusRule
metadata:
name: <resource-name>
spec:
  <rule definition>
```

See [the Alertmanager documentation](https://prometheus.io/docs/prometheus/latest/configuration/alerting_rules/) for `<rule definition>`

Example:
To add an Alerting rule you need to create a PrometheusRule resource in the monitoring folder of your CAASI Team Config Repository.

```yaml
apiVersion: monitoring.coreos.com/v1
kind: PrometheusRule
metadata:
  name: testrules
spec:
  groups:
    - name: testrulesgroup
      rules:
        - alert: kubePodCrashLooping # this will be the mail subject, enter which ever text you want
          expr: rate(kube_pod_container_status_restarts_total{job="kube-state-metrics",namespace="testnamespace"}[5m]) * 60 * 5 > 0
          for: 15m
          annotations:
            message: Pod {{ $labels.namespace }}/{{ $labels.pod }} ({{ $labels.container }}) is restarting {{ printf "%.2f" $value }} times / 5 minutes.
          labels:
            severity: critical
```

This will fire an alert, everytime the following query matches

```
rate(kube_pod_container_status_restarts_total{job="kube-state-metrics",namespace="testnamespace"}[5m]) * 60 * 5 > 0
```

You can build/verify your Query in your Thanos Querier UI. As soon, as you apply the PrometheusRule resource, you should be able to see the alert in your Thanos Ruler implementation.

### Task {{% param sectionnumber %}}.2: Send a test alert

In this task you can use the [amtool](https://github.com/prometheus/alertmanager#amtool) command to send a test alert.

{{% details title="Hints" mode-switcher="normalexpertmode" %}}

To send a test alert with the labels `alertname=Up` and `node=bar` you can simply execute the following command.

```bash
oc -n <namespace> exec -it sts/alertmanager-alertmanager -- sh
amtool alert add --alertmanager.url=http://localhost:9093 alertname=Up node=bar
```

Check in the [Alertmanger web UI](http://LOCALHOST:9093) if you see the test alert with the correct labels set.

### Task {{% param sectionnumber %}}.3: Show the routing tree

Show routing tree:

```bash
oc -n <namespace> exec -it sts/alertmanager-alertmanager -- sh
amtool config routes --config.file /etc/alertmanager/config/alertmanager.yml
```

Depending on the configured receivers your output might vary.

The routing tree of the monitoring stack in namespace `infra-config` is more complex than the one of the `examples-monitoring` namespace:

Namespace `config-caasi01-monitoring`:

```bash
$ oc -n config-caasi01-monitoring exec -it sts/alertmanager-alertmanager -- amtool config routes --config.file /etc/alertmanager/config/alertmanager.yaml
Routing tree:
.
└── default-route  receiver: default
    ├── {severity=~"^(?:critical|warning)$"}  continue: true  receiver: mail-critical
    ├── {alertname=~"^(?:DeadMansSwitch)$"}  receiver: deadmanswitch
    ├── {env="prod",severity="critical"}  receiver: teams-critical-prod
    ├── {env="prod",severity="warning"}  receiver: teams-warning-prod
    ├── {env="prod"}  receiver: teams-info-prod
    ├── {env!="prod",severity="critical"}  receiver: teams-critical-nonprod
    ├── {env!="prod",severity="warning"}  receiver: teams-warning-nonprod
    ├── {env!="prod",severity="info"}  receiver: teams-info-nonprod
    └── {env!="prod"}  receiver: teams-warning-prod
```

Namespace `examples-monitoring`:

```bash
$ oc -n examples-monitoring exec -it sts/alertmanager-alertmanager -- amtool config routes --config.file /etc/alertmanager/config/alertmanager.yaml
Routing tree:
.
└── default-route  receiver: default
    └── {severity=~"^(?:critical|warning)$"}  continue: true  receiver: mail-critical
```

### Task {{% param sectionnumber %}}.4: Test your alert receivers

Add a test alert and check if your defined target mailbox receives the mail. It can take up to 5 minutes as the alarms are grouped together based on the [group_interval](https://prometheus.io/docs/alerting/latest/configuration/#route).

![Alerting Mail](../alert-mail.png)

```bash
oc -n <namespace> exec -it sts/alertmanager-alertmanager -- sh
amtool alert add --alertmanager.url=http://localhost:9093 env=dev severity=critical
```

Example:

```bash
oc -n examples-monitoring exec -it sts/alertmanager-alertmanager -- sh
amtool alert add --alertmanager.url=http://localhost:9093 alert=test severity=critical
```

It is also advisable to validate the routing configuration against a test dataset to avoid unintended changes. With the option `--verify.receivers` the expected output can be specified:

```bash
oc -n examples-monitoring exec -it sts/alertmanager-alertmanager -- sh
amtool config routes test --config.file /etc/alertmanager/config/alertmanager.yaml --verify.receivers=mail-critical env=dev severity=info
```

```bash
default
WARNING: Expected receivers did not match resolved receivers.
```

```bash
oc -n examples-monitoring exec -it sts/alertmanager-alertmanager -- sh
amtool config routes test --config.file /etc/alertmanager/config/alertmanager.yaml --verify.receivers=mail-critical env=prod severity=critical
```

```bash
mail-critical
```

