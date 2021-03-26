---
title: "3. Alerting with Alertmanager"
weight: 1
sectionnumber: 1
---

## Installation

### Setup

The alertmanager is already installed on your system and can be controlled using systemctl:

```bash
# status
sudo systemctl status alertmanager

# start
sudo systemctl start alertmanager

# stop
sudo systemctl stop alertmanager

# restart
sudo systemctl restart alertmanager

# reload
sudo systemctl reload alertmanager
```

The configuration file of alertmanager is located here: `/etc/alertmanager/alertmanager.yml`

### Configuration

Alertmanager's configuration is done using a YAML config file and CLI flags. Take a look at the very basic configuration file at `/etc/alertmanager/alertmanager.yml`:

```yaml
global:
  resolve_timeout: 5m

route:
  group_by: ['alertname']
  group_wait: 10s
  group_interval: 10s
  repeat_interval: 1h
  receiver: 'web.hook'
receivers:
- name: 'web.hook'
  webhook_configs:
  - url: 'http://127.0.0.1:5001/'
inhibit_rules:
  - source_match:
      severity: 'critical'
    target_match:
      severity: 'warning'
    equal: ['alertname', 'dev', 'instance']
```

{{% alert title="Note" color="primary" %}}
For the moment we leave it with the default configuration and come back to it later in the course.
{{% /alert %}}

## Configuration in Alertmanager

There are two main sections for configuring how Alertmanager is dispatching alerts: receivers and routing.

### Receivers

With a [receiver](https://prometheus.io/docs/alerting/latest/configuration/#receiver), one or more notifications can be defined. There are different types of notifications types, e.g. mail, webhook, or one of the message platforms like Slack or PagerDuty.

### Routing

With [routing blocks](https://prometheus.io/docs/alerting/latest/configuration/#route), a tree of routes and child routes can be defined. Each routing block has a matcher which can match one or several labels of an alert. Per block, one receiver can be specified, or if empty, the default receiver is taken.

### amtool

As routing definitions might be very complex and hard to understand, [amtool](https://github.com/prometheus/alertmanager#examples) becomes handy as it helps to test the rules. It can also generate test alerts and has even more useful features. More about this in the labs.

### More (advanced) options

For more insights of the configuration options, study the following resources:

* Example configuration provided by [Alertmanager on GitHub](https://github.com/prometheus/alertmanager/blob/master/doc/examples/simple.yml)
* General overview of [Alertmanager](https://prometheus.io/docs/alerting/latest/alertmanager/)

Before we enable Alertmanager in Prometheus, let's do some [labs concerning the Alertmanager](labs/32).

## Enable Alertmanager in Prometheus

The Alertmanager instance we installed before must be configured in Prometheus. Open `prometheus.yml`, add the config below, and reload the Prometheus config with `sudo systemctl reload prometheus`.

```yaml
# Alertmanager configuration
alerting:
  alertmanagers:
  - static_configs:
    - targets:
       - localhost:9093
```

## Alert rules in Prometheus

[Prometheus alert rules](https://prometheus.io/docs/prometheus/latest/configuration/alerting_rules/) are configured very similarly to recording rules which you got to know [earlier in this training](/docs/02#recording-rules). The main difference is that the rule's expression contains a threshold (e.g., `query_expression >= 5`) and that an alert is sent to the Alertmanager in case the rule evaluation matches the threshold. An alert rule can be based on a recording rule or be a normal expression query.

Now it's time for the last [labs concerning alertrules and alerts](labs/33).

{{% alert title="Note" color="primary" %}}
Sometimes the community or the maintainer of your Prometheus exporter already provide generic Prometheus rules that can be adapted to your needs. For this reason, it makes sense to do some research before writing alerting rules from scratch. Before implementing such a rule, you should always understand and verify the rule. Here are some examples:

* MySQL: [mysqld-mixin](https://github.com/prometheus/mysqld_exporter/tree/master/mysqld-mixin)
* Kafka: [strimzi/strimzi-kafka-operator](https://github.com/strimzi/strimzi-kafka-operator/blob/master/examples/metrics/prometheus-install/prometheus-rules.yaml)
* Kubernetes: [kubernetes-mixin-ruleset](https://github.com/prometheus-operator/kube-prometheus/tree/main/manifests)
* General rules for various exporters: [samber/awesome-prometheus-alerts](https://github.com/samber/awesome-prometheus-alerts)

{{% /alert %}}
