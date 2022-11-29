---
title: "3.1 Tasks: Setup custom alerting rules"
weight: 2
sectionnumber: 3.1
onlyWhen baloise
---

### Task {{% param sectionnumber %}}.1: Enable Alertmanager in Prometheus

The Alertmanager instance we installed before must be configured in Prometheus. Open `/etc/prometheus/prometheus.yml`, add the config below, and reload the Prometheus config with `sudo systemctl reload prometheus.service`.

```yaml
...
# Alertmanager configuration
alerting:
  alertmanagers:
    - static_configs:
        - targets:
            - localhost:9093
...
```

### Task {{% param sectionnumber %}}.2: Add Alertmanager as monitoring target

{{% alert title="Note" color="primary" %}}
This setup is only suitable for our lab environment. In real life, you must consider how to monitor your monitoring infrastructure:
Having an Alertmanager instance as an Alertmanager AND as a target only in the same Prometheus is a bad idea!
{{% /alert %}}

This is repetition: The Alertmanager (`localhost:9093`) also exposes metrics which can be scraped by Prometheus.

Configure the metric endpoint of Alertmanager in Prometheus and check, if the target in Prometheus can be scraped.

{{% details title="Hints" mode-switcher="normalexpertmode" %}}

Configure a new job under `scrape_configs` in `/etc/prometheus/prometheus.yml`:
```yaml
  ...
  - job_name: "alertmanager"
    static_configs:
      - targets: ["localhost:9093"]
  ...
```

Reload Prometheus
```bash
sudo systemctl reload prometheus
```

Check in the [Prometheus web UI](http://LOCALHOST:9090/targets) if the target can be scraped.

{{% /details %}}

### Task {{% param sectionnumber %}}.3: Query an Alertmanager metric

After you add the Alertmanager metrics endpoint, you will have huge bunch of different values and identifiers.

Use curl to get a list of all available metrics and query any one from the Alertmanager.

{{% details title="Hints" mode-switcher="normalexpertmode" %}}

To find out which metrics are available for one service you might query its metrics endpoint with `curl`, e.g. for Alertmanager:

```bash
curl localhost:9093/metrics
```

Then you get all metrics as follows (shortened), and you can pick whatever you're interested in.

```promql
# HELP alertmanager_alerts How many alerts by state.
# TYPE alertmanager_alerts gauge
alertmanager_alerts{state="active"} 0
alertmanager_alerts{state="suppressed"} 0
# HELP alertmanager_alerts_invalid_total The total number of received alerts that were invalid.
# TYPE alertmanager_alerts_invalid_total counter
alertmanager_alerts_invalid_total{version="v1"} 0
alertmanager_alerts_invalid_total{version="v2"} 0
# HELP alertmanager_alerts_received_total The total number of received alerts.
# TYPE alertmanager_alerts_received_total counter
alertmanager_alerts_received_total{status="firing",version="v1"} 0
alertmanager_alerts_received_total{status="firing",version="v2"} 0
alertmanager_alerts_received_total{status="resolved",version="v1"} 0
alertmanager_alerts_received_total{status="resolved",version="v2"} 0
...
```

{{% /details %}}

### Task {{% param sectionnumber %}}.4: Get all Metrics from Alertmanager

After you successfully configured Prometheus to scrape the Alertmanager you can also query them using PromQL

Write a PromQL query, which selects all metrics exposed by the Alertmanager (`job="alertmanager"`).

{{% details title="Hints" mode-switcher="normalexpertmode" %}}

To do that, we can simply execute a query without a metrics name and only the job label filter `job="alertmanager"`.

```promql
{job="alertmanager"}
```

{{% /details %}}

